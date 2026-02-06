# 角色开发进阶

本章节将深入 fount 的核心能力：**流式输出**、**工具调用**、**逻辑匹配**以及**自定义 API**。
我们将以社区中的知名角色 **Gentian (龙胆)** 和 **Saira** 为例，解析她们是如何实现的。

<img width="729" height="342" alt="image" src="https://github.com/user-attachments/assets/9a476938-b14f-4160-9bdd-9f839aa40fa0" />

---

## 1. 关键词匹配机制

龙胆实现了一个强大的“不依赖 AI 的设定触发器”。她不调用 LLM，而是直接扫描聊天记录，这在性能和成本上都非常划算。

以下是 `match.mjs` 的完整源码解析：

```javascript
/** @typedef {import('../../../../../../src/public/parts/shells/chat/decl/chatLog.ts').chatReplyRequest_t} chatReplyRequest_t */
/** @typedef {import('../../../../../../src/public/parts/shells/chat/decl/chatLog.ts').chatLogEntry_t} chatLogEntry_t */
import { escapeRegExp } from './tools.mjs'
import * as OpenCC from 'opencc-js'

/** 繁转简，用于匹配时统一成简体再搜，避免繁简不同搜不到 */
const chT2S = OpenCC.Converter({ from: 'twp', to: 'cn' })

/**
 * 从聊天记录里截取一段，可以按「谁发的」「最近几条」筛选
 * @param {chatReplyRequest_t} args - fount 传进来的聊天请求
 * @param {'user'|'char'|'both'|'other'|'any'} [from='any'] - 只保留谁发的
 * @param {number} [depth=4] - 取最近几条
 * @returns {chatLogEntry_t[]}
 */
export function getScopedChatLog(args, from = 'any', depth = 4) {
    let chat_log = args.chat_log.slice(-depth)
    switch (from) {
        case 'user':
            chat_log = chat_log.filter(x => x.name == args.UserCharname)
            break
        case 'char':
            chat_log = chat_log.filter(x => x.name == args.Charname)
            break
        case 'both':
            chat_log = chat_log.filter(x => x.name == args.UserCharname || x.name == args.Charname)
            break
        case 'other':
            chat_log = chat_log.filter(x => x.name != args.UserCharname && x.name != args.Charname)
            break
    }
    return chat_log
}

/**
 * 在指定范围的聊天记录里搜关键词，返回命中数量
 * @param {chatReplyRequest_t} args - 聊天请求
 * @param {(string|RegExp)[]} keys - 关键词列表
 * @param {'any'|'user'|'char'|'other'} [from='any'] - 搜哪段记录
 * @param {number} [depth=4] - 搜最近几条
 * @param {(content:string, reg_keys:RegExp[]) => number} [matcher] - 自定义匹配逻辑
 * @returns {Promise<number>}
 */
export async function match_keys(args, keys, from = 'any', depth = 4,
    matcher = (content, reg_keys) => reg_keys.filter(key => content.match(key)).length
) {
    const chat_log = getScopedChatLog(args, from, depth)
    // 自动处理正则转义和中文分词边界
    keys = keys.map(key =>
        key instanceof RegExp ? key :
        new RegExp(/\p{Unified_Ideograph}/u.test(key) ? escapeRegExp(key) : `\\b${escapeRegExp(key)}\\b`, 'ugi'))

    // 统一转简体进行匹配
    const content = chat_log.map(x => x.extension?.SimplifiedContent ??= chT2S(x.content)).join('\n')
    return matcher(content, keys)
}

/** 只有所有 key 都命中时才返回真，适合「必须同时提到 A 和 B」的场景 */
export async function match_keys_all(args, keys, from = 'any', depth = 4) {
    return match_keys(args, keys, from, depth,
        (content, reg_keys) => reg_keys.every(key => content.match(key)))
}
```

### 使用示例：扩展 Prompt 边界

既然我们有了 `match_keys`，我们甚至可以让 Javascript 运行结果直接进入 Prompt。
比如，告诉 AI 当前时间：

```javascript
// 如果聊到了时间相关的词
if (await match_keys(args, ['什么日子', '几点了', 'date', 'time'], 'any')) {
    const locale = args.locales?.[0] || 'zh-CN'
    const now = new Date().toLocaleString(locale);
    // 直接把时间塞进 Prompt
    prompt += `\n[System] 当前时间：${now}`;
}
```

---

## 2. 工具调用与代码执行

fount 角色可以执行任何代码。
龙胆通过让 AI 输出 XML 标签（如 `<run-pwsh>`）来识别工具调用。

### 实现思路

1. **Prompt 约定**：告诉 AI，如果想执行命令，请输出 `<run-pwsh>Write-Host "Hello"</run-pwsh>`。
2. **ReplyHandler**：在 `GetReply` 拿到 AI 回复后，用正则提取标签内容。
3. **Execution**：调用 Node.js 的 `child_process` 执行命令。
4. **Regen**：把执行结果塞回聊天记录，让 AI 重新生成回复（或者继续对话）。

### 代码示例 (龙胆的 coderunner)

```javascript
import { exec } from 'node:child_process';
import { promisify } from 'node:util';
const execPromise = promisify(exec);

export async function coderunner(result, { AddLongTimeLog }) {
    // 1. 尝试匹配 <run-pwsh> 标签
    const pwshCode = result.content.match(/<run-pwsh>([\s\S]*?)<\/run-pwsh>/)?.[1];
    
    if (pwshCode) {
        // 2. 将 AI 的请求记录下来 (让用户看到)
        console.log('AI 请求运行 PowerShell:', pwshCode);

        // 3. 将AI的请求作为消息塞入聊天记录
        AddLongTimeLog({
            name: '龙胆', role: 'char',
            content: `<run-pwsh>\n${pwshCode}\n</run-pwsh>`,
            files: [], extension: {}
        });

        // 4. 执行代码
        let output;
        try {
            const { stdout, stderr } = await execPromise(pwshCode, { shell: 'pwsh.exe' });
            output = `STDOUT:\n${stdout}\nSTDERR:\n${stderr}`;
        } catch (err) {
            output = `Error:\n${err.message}`;
        }

        // 5. 将结果作为 System 消息塞回去
        AddLongTimeLog({
            name: 'system', role: 'system',
            content: `代码执行结果：\n${output}`,
            files: [], extension: {}
        });

        // 6. 返回 true，告诉主循环 "我处理了工具调用，请让 AI 再生成一次回复"
        return true; 
    }
    return false;
}
```

---

## 3. 网页交互与 Router

Saira (赛拉) 拥有一个独立的网页界面“记忆宫殿”。
这是通过 fount 的 Router 功能实现的。

### A. 挂载 Express 路由

在 `Load` 函数中，我们可以往 `stat.router` 上挂载自定义 API。

```javascript
Load: async stat => {
    const charname = 'Saira';
    // 路由路径必须与 /parts/chars:<角色名> 对应
    // Express 中 : 是特殊字符，必须转义为 \\:
    const apiUrl = `/api/parts/chars\\:${charname}`;

    stat.router.post(`${apiUrl}/palace_of_loci/chat`, async (req, res) => {
        const { history, user_input } = req.body;
        // ... 这里调用 GetReply 生成回复 ...
        res.json({ content: 'AI 的回复' });
    });
}
```

### B. 提供静态网页

在角色目录下创建 `public/palace_of_loci/index.html`。
用户可以通过浏览器访问：`http://localhost:8000/parts/chars:Saira/palace_of_loci/`

### C. 网页反向控制角色 (`window.trigger`)

Saira 的网页不仅能显示，还能控制 fount。
她在网页中通过 `fetch` 调用上面注册的 API。

```javascript
// 前端代码 (public/index.html 中的 script)
const apiBase = `/api/parts/chars:Saira`;

async function sendMessage(text) {
    const res = await fetch(`${apiBase}/palace_of_loci/chat`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ user_input: text })
    });
    const data = await res.json();
    console.log("AI 回复:", data.content);
}
```

更高级的玩法是，AI 可以在回复中输出 `<execute-js>window.trigger('event_name')</execute-js>`，前端网页监听到后执行特定动画。

---

## 5. 流式输出与预览优化

如果你的角色使用了 `<run-pwsh>...</run-pwsh>` 这类 XML 标签，流式生成时 AI 会先慢慢吐出 `<run-pwsh>`，再吐代码，最后才是 `</run-pwsh>`。中途用户会看到半成品，体验不好。

用 `defineToolUseBlocks` 可以把这些未闭合的标签先藏起来，等完整了再显示：

```javascript
import { defineToolUseBlocks } from '../../../../../src/public/parts/shells/chat/src/stream.mjs'

// 在 GetReply 中构建 replyPreviewUpdater 管线
let replyPreviewUpdater = (args, r) => oriReplyPreviewUpdater?.(r)
for (const GetReplyPreviewUpdater of [
    defineToolUseBlocks([
        { start: '<run-pwsh>', end: '</run-pwsh>' },
        { start: /<generate-char[^>]*>/, end: '</generate-char>' },
        // ... 你的工具标签对
    ]),
    ...Object.values(args.plugins || {}).map(p => p.interfaces?.chat?.GetReplyPreviewUpdater)
].filter(Boolean))
    replyPreviewUpdater = GetReplyPreviewUpdater(replyPreviewUpdater)

args.generation_options.replyPreviewUpdater = r => replyPreviewUpdater(args, r)
```

这样，用户就不会看到那一堆乱七八糟的代码生成过程，直到工具执行完毕显示结果。

---

## 6. 打破思维定势：不仅限于回复

其实 **AI 来源和 JS 代码不是只能用在回复时** —— 你可以在角色的 **任意地方** 调用 AI 和跑 JS！

和酒馆不同，fount 的开场白、prompt 都是函数返回的，你可以自己决定怎么拼。比如：

- **Init/Load 阶段搞事**：在角色加载时就扫描用户硬盘，根据用户的文件生成不同的开场白（请务必注意道德和隐私！）。
- **AI 生成开场白**：在 `GetGreeting` 里调用 AI，让开局随着时间、好感度甚至当天的天气新闻变化。
- **Meta-Prompt**：把 prompt 的构建也交给 AI，你只写一个“教 AI 怎么写 prompt”的 meta-prompt。

不要被“只能回复时用”限制住，尽情发挥想象。你的角色是一个活生生的程序，而不仅是一张卡片。

---

## 7. 动态修改名字与头像

还可以玩得更花：除了 `content` 和 `files`，回复对象里还能加 `name` 或 `avatar`，这会改变 **这条回复** 显示的角色名和头像。

你可以做“随着回复内容换表情”的角色、百变怪、模仿者…… 发挥想象就好。

```javascript
GetReply: async args => {
    // ...
    return {
        content: '我是新的角色！',
        name: '变身后的名字',
        avatar: 'https://example.com/new_avatar.png' 
    }
}
```

---

## 8. 制作安装包

想把角色分享给别人？fount 的安装包其实就是个 7z/zip，把角色文件夹打进去就行。
记得在根目录放一个 `fount.json`，告诉 fount 这是角色包、文件夹叫什么名字：

```json
{
    "type": "chars",
    "dirname": "<角色文件夹名称>"
}
```

然后把整个角色文件夹（含 `fount.json`、`main.mjs` 等）打成 7z/zip，别人就可以在 fount 里导入这个安装包了。

---

## 9. 完整模板代码

下面是一个可以直接复制用的模板。包含流式支持和插件支持。

```javascript
/**
 * @typedef {import('../../../../../src/decl/charAPI.ts').CharAPI_t} CharAPI_t
 */

import { buildPromptStruct } from '../../../../../src/public/parts/shells/chat/src/prompt_struct.mjs'
import { loadPart, loadAnyPreferredDefaultPart } from '../../../../../src/server/parts_loader.mjs'

/** 当前登录用户名，Load 时由 fount 传入 */
let username = ''
/** 当前使用的 AI 源，未配置时为 null */
/** @type {import('../../../../../src/decl/AIsource.ts').textAISource_t} */
let AIsource = null

/** @type {CharAPI_t} */
export default {
	info: {
		'zh-CN': {
			name: '<角色名>',
			avatar: '<头像 url，可留空或用 /parts/chars:角色名/xxx 格式>',
			description: '<一句话介绍，显示在角色列表>',
			description_markdown: '<完整介绍，支持 Markdown>',
			version: '<版本号>',
			author: '<作者名>',
			home_page: '<主页>',
			tags: ['<标签1>', '<标签2>'],
		}
	},

	// 初始化函数，在角色被启用时调用，可留空
	Init: (stat) => { },

	// 安装卸载函数，在角色被安装/卸载时调用，可留空
	Uninstall: (reason, from) => { },

	// 加载函数，在角色被加载时调用，在这里获取用户名
	Load: (stat) => {
		username = stat.username // 获取用户名
	},

	// 卸载函数，在角色被卸载时调用，可留空
	Unload: (reason) => { },

	// 角色的接口
	interfaces: {
		// 角色的配置接口
		config: {
			// 获取角色的配置数据
			GetData: () => ({
				AIsource: AIsource?.filename || '', // 返回当前使用的AI源的文件名
			}),
			// 设置角色的配置数据
			SetData: async data => {
				if (data.AIsource)
					AIsource = await loadPart(username, 'serviceSources/AI/' + data.AIsource)
				else
					AIsource = await loadAnyPreferredDefaultPart(username, 'serviceSources/AI')
			}
		},
		// 角色的聊天接口
		chat: {
			// 获取角色的开场白
			GetGreeting: (arg, index) => [{ content: '<角色的开场白>' }, { content: '<可以多个>' },][index],
			// 获取角色在群组中的问好
			GetGroupGreeting: (arg, index) => [{ content: '<群组中角色加入时的问好>' }, { content: '<可以多个>' },][index],
			// 获取角色的提示词
			GetPrompt: async args => ({
				text: [{ content: '<角色的设定内容>', important: 0 }],
				additional_chat_log: [],
				extension: {},
			}),
			GetPromptForOther: args => ({
				text: [{ content: '<其他角色看到的该角色的设定，群聊时生效>', important: 0 }],
				additional_chat_log: [],
				extension: {},
			}),
			// 获取角色的回复
			GetReply: async (args) => {
				// 如果没有设置AI源，返回默认回复
				if (!AIsource) return { content: '<未设置角色的AI来源时角色的对话回复>' }
				// 用fount提供的工具构建提示词结构
				const prompt_struct = await buildPromptStruct(args)
				// 创建回复容器
				/** @type {import("../../../../../src/public/parts/shells/chat/decl/chatLog.ts").chatReply_t} */
				const result = {
					content: '',
					logContextBefore: [],
					logContextAfter: [],
					files: [],
					extension: {},
				}
				// AddLongTimeLog：工具执行后把结果塞进 prompt，供 AI 下一轮参考
				function AddLongTimeLog(entry) {
					entry.charVisibility = [args.char_id]
					result?.logContextBefore?.push?.(entry)
					prompt_struct.char_prompt.additional_chat_log.push(entry)
				}

				// regen 循环：AI 输出 -> 若有 ReplyHandler 处理了工具调用，再生成一轮
				regen: while (true) {
					args.generation_options ??= {}
					args.generation_options.base_result = result
					const requestResult = await AIsource.StructCall(prompt_struct, args.generation_options)
					result.content = requestResult.content
					result.files = result.files.concat(requestResult.files || [])
					let continue_regen = false
					for (const handler of [...Object.values(args.plugins || {})]
						.map(p => p?.interfaces?.chat?.ReplyHandler).filter(Boolean))
						if (await handler(result, { ...args, prompt_struct, AddLongTimeLog }))
							continue_regen = true
					if (continue_regen) continue regen
					break
				}
				// 返回构建好的回复
				return result
			}
		}
	}
}
```

---

## 10. 持续集成 (CI) 自动化测试

当角色的功能越来越复杂——工具调用、文件操作、记忆系统、定时器等——手动验证每个功能既费时又容易遗漏。[fount-charCI](https://github.com/steve02081504/fount-charCI) 是专门为 fount 角色设计的 CI 工具，能在代码推送后自动运行测试，保证角色在发布前的基本可用性。

> **注意**：fount-charCI 测试的是**程序逻辑的正确性**（如工具是否正常执行、文件是否写入成功），而非 AI 生成内容的质量。LLM 输出具有随机性，质量评估需要人工或其他手段。

### A. 三步快速上手

**第一步：创建 GitHub Actions 工作流**

在角色项目根目录创建 `.github/workflows/CI.yaml`（或 `CI.yml`）：

```yaml
name: Test Running

permissions:
  contents: read
  actions: write

on:
  workflow_dispatch:           # 允许手动触发
  push:
    paths:
      - '**.mjs'               # 仅 .mjs 变更时触发
    tags-ignore:
      - '*'                    # 忽略 tag 推送（避免发版时重复跑）
    branches:
      - '*'

jobs:
  test-running:
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]   # 可选：多平台
    runs-on: ${{ matrix.os }}
    steps:
      - uses: steve02081504/fount-charCI@master
        with:
          CI-filepath: .github/workflows/CI.mjs
```

**第二步：编写测试脚本**

创建 `.github/workflows/CI.mjs`，`fountCharCI` 会自动注入到全局，可直接使用：

```javascript
/* global fountCharCI */
const CI = fountCharCI

// 测试 1：无 AI 源时的兜底
await CI.test('noAI Fallback', async () => {
  await CI.char.interfaces.config.SetData({ AIsources: {} })
  await CI.runOutput()
})

// 测试 2：配置 AI 源并跑一次对话
await CI.test('Setup AI Source', async () => {
  await CI.char.interfaces.config.SetData({
    AIsources: { CI: 'CI' },
    disable_idle_event: true
  })
})
```

推送后，每次 `.mjs` 文件变更都会自动运行 CI。

### B. 核心 API 速查

| API                         | 用途                                         |
| --------------------------- | -------------------------------------------- |
| `CI.test(name, asyncFn)`    | 定义测试块，支持嵌套                         |
| `CI.runOutput(output)`      | 模拟 AI 输出，测试 ReplyHandler / 工具调用   |
| `CI.runInput(input)`        | 模拟用户输入，走完整「用户 → AI → 角色」流程 |
| `CI.assert(cond, msg)`      | 断言，失败时抛出带 `msg` 的错误              |
| `CI.context.workSpace.path` | 当前测试的隔离工作目录                       |
| `CI.context.http`           | `{ router, url, root }`，用于 mock 网页等    |
| `CI.char`                   | 当前加载的角色实例                           |
| `CI.wait(fn, timeout)`      | 轮询直到 `fn()` 为真或超时                   |

### C. 典型测试场景（参考龙胆）

**1. 测试工具调用**

用 `CI.runOutput` 传入 AI 的「输出」，触发 ReplyHandler，再检查 `logContextBefore` 中的 `role === 'tool'` 日志：

```javascript
CI.test('File Operations', async () => {
  CI.test('<view-file>', async () => {
    const testFilePath = path.join(CI.context.workSpace.path, 'view_test.txt')
    fs.writeFileSync(testFilePath, 'Hello from <view-file>!', 'utf-8')

    const result = await CI.runOutput([
      `<view-file>${testFilePath}</view-file>`,
      'File content is: Hello from <view-file>!'
    ])
    const systemLog = result.logContextBefore.find(log => log.role === 'tool')
    CI.assert(systemLog?.content.includes('Hello from <view-file>!'),
      `<view-file> 未正确读取文件内容`)
  })
})
```

**2. 测试代码执行（run-pwsh / inline-js 等）**

根据平台选择 `<run-pwsh>` / `<run-bash>` 或 `<inline-js>`：

```javascript
CI.test('Code Runner', () => {
  if (process.platform === 'win32') {
    CI.test('<run-pwsh>', async () => {
      const testDir = path.join(CI.context.workSpace.path, 'pwsh_test_dir')
      await CI.runOutput([`<run-pwsh>mkdir ${testDir}</run-pwsh>`, 'Directory created.'])
      CI.assert(fs.existsSync(testDir), `<run-pwsh> 未成功创建目录`)
    })
    CI.test('<inline-pwsh>', async () => {
      const result = await CI.runOutput('The result is <inline-pwsh>echo "hello"</inline-pwsh>.')
      CI.assert(result.content === 'The result is hello.', `<inline-pwsh> 未正确替换`)
    })
  }
  // ...
})
```

**3. 测试需要中间步校验的流程**

`runOutput` 的第二个参数可以是**函数**，用于在每一步后做断言并决定下一步 AI「输出」：

```javascript
const result = await CI.runOutput([
  `<web-browse><url>${url}</url><question>段落写了什么？</question></web-browse>`,
  result => {
    CI.assert(result.prompt_single.includes('This is a test paragraph'),
      'web-browse 未将网页内容放入 prompt')
    return 'The paragraph says: This is a test paragraph.'  // 模拟 AI 下一步回复
  },
  'Web browse test complete.'
])
```

**4. 测试异步回调（定时器、callback 等）**

用 `CI.wait` 等待异步结果：

```javascript
CI.test('Timer', async () => {
  await CI.runOutput([
    '<set-timer><item><time>1s</time><reason>CI_Test_Timer</reason></item></set-timer>',
    'Timer test complete.',
    '<run-js>globalThis.timerCallbacked = true;</run-js>',
    'Done.'
  ])
  await CI.wait(() => globalThis.timerCallbacked, 10000)
  CI.assert(globalThis.timerCallbacked, '定时器回调未执行')
})
```

**5. 测试特殊回复标记**

```javascript
CI.test('<-<null>-> (AI Skip)', async () => {
  const result = await CI.runOutput('<-<null>->')
  CI.assert(result === null, `<-<null>-> 应返回 null，实际: ${JSON.stringify(result)}`)
})
```

### D. 测试隔离与注意事项

- 每个测试拥有**独立的 `workSpace.path`** 和 **`http.router`**，互不干扰。
- 文件操作请使用 `CI.context.workSpace.path`，测试结束后会自动清理。
- 使用 `CI.beforeAll` / `CI.afterEach` 等钩子做共享资源的准备与回收。
- 不写 `await` 的 `CI.test` 会并发执行，需要顺序时务必加 `await`。

更完整的示例可参考龙胆的 [CI.mjs](https://github.com/steve02081504/GentianAphrodite/blob/master/.github/workflows/CI.mjs)。

---

[← 上一页：编写第一个角色](./dev-basic.md) | [返回主页](../README.md) | [下一页：从 Chatbot 到 Agent →](./dev-agent.md)
