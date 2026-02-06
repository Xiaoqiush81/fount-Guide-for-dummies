# 编写你的第一个角色：丁真

本篇教程将带你从零开始创建一个 fount 角色。

## 0. 写在前面：fount 与酒馆的区别

首先，fount 角色和酒馆角色有什么区别？

简而言之，**fount 角色是酒馆角色的超集**。

- **酒馆**：角色卡基本上是 **Prompt 艺术**。其近乎全部内容都是在指导酒馆如何构建 prompt。  
  酒馆的 workflow 是近乎不变的：从角色卡和预设以及聊天记录中构建 prompt -> 发送到 AI 入口点 -> 根据输出更新聊天记录。
- **fount**：通过加载 char 提供的 mjs 入口点向角色提供了对 workflow 的自定义能力。  
  即：**酒馆要求角色提供 prompt 以参与 RP workflow 本身，而 fount 要求角色提供 workflow 来完成对话逻辑**。

换言之，fount 只管调用角色卡提供的函数，而角色要考虑的就多了：什么时候构建 prompt、什么时候调用 AI、怎么处理 AI 的返回结果，都需要角色卡细细斟酌。

所以，fount 角色实际上并不只追求 prompt 的编写能力，还需要一定的 JavaScript 基本功。

### ⚠️ 什么情况下不适合写 fount 角色？

- 害怕麻烦，不想折腾编程。
- 只想创作用 prompt 便可以达成的普通角色。

fount 可以正常运行大部分酒馆角色卡，所以学习制作原生的 fount 角色并不是必选项。 :)

---

## 1. 准备工作

首先，在 fount 的 `data/users/<你的用户名>/chars/` 目录下创建一个新文件夹，例如 `dinzhen`。这就是你的角色根目录。

> **💡 路径说明**：下文的 import 路径假定角色在 `data/users/<用户名>/chars/<角色文件夹>/`。如果你把角色放在别的地方，需要根据层级调整 `../../../../../src/` 里 `../` 的数量。不用太纠结，照着模板写一般不会错。

<img width="318" height="227" alt="image" src="https://github.com/user-attachments/assets/b127ed62-ac4d-4d7b-bda6-d7c466dbc51c" />

我们需要在这个文件夹里创建两个文件：`fount.json` 和 `main.mjs`。

---

## 2. 编写 `fount.json`

这是一个身份证，告诉 fount 这是一个角色包。

```json
{
  "type": "chars",
  "dirname": "dinzhen"
}
```

---

## 3. 编写 `main.mjs`

这是角色的灵魂。我们将创建一个名为“丁真”的角色，他非常纯真，只会回复“妈妈生的”。

复制以下代码到 `main.mjs`：

<img width="192" height="112" alt="image" src="https://github.com/user-attachments/assets/edfa4fa5-86f5-49c6-be9e-0bf2fffb8ae9" />

```javascript
/**
 * @typedef {import('../../../../../src/decl/charAPI.ts').CharAPI_t} CharAPI_t
 */

/** @type {CharAPI_t} */
export default {
    info: {
        'zh-CN': {
            name: '丁真',
            avatar: '',
            description: '妈妈生的',
            description_markdown: '妈妈生的',
            version: '1.0.0',
            author: 'beginner',
            tags: [],
        }
    },

    Init: stat => { },
    Load: stat => { },
    Unload: reason => { },
    Uninstall: (reason, from) => { },

    interfaces: {
        chat: {
            GetGreeting: (arg, index) => [{ content: '哈喽？' }][index],
            GetGroupGreeting: (arg, index) => [{ content: '哈喽？' }][index],
            GetPrompt: async args => ({
                text: [],
                additional_chat_log: [],
                extension: {},
            }),
            GetPromptForOther: args => ({
                text: [],
                additional_chat_log: [],
                extension: {},
            }),
            // 核心：生成回复
            GetReply: async args => {
                // 这个char每次被fount要求给出回复后，根本没有用到聊天记录或其他内容
                // 只是单纯的返回了这一数据结构
                return { content: '妈妈生的' }
            }
        }
    }
}
```

随后启动或刷新 fount，你可以看到丁真出现在角色选区内。

<img width="1078" height="573" alt="image" src="https://github.com/user-attachments/assets/03ff7f70-c063-482a-a20c-292b0f88d6c3" />

你可以随意尝试，但很明显的，这位丁真不论我们说什么都只会回复`妈妈生的`。
和酒馆不同，这个角色的定义中没有任何的 prompt。它甚至没有用到 AI。
究其原因，让我们细看该js中的 `GetReply` 函数：

```javascript
// 核心：生成回复
GetReply: async args => {
    // 这个char每次被fount要求给出回复后，根本没有用到聊天记录或其他内容
    // 只是单纯的返回了这一数据结构
    return { content: '妈妈生的' }
}
```

<img width="507" height="876" alt="image" src="https://github.com/user-attachments/assets/6a35b8f8-4875-43ec-8186-35655d397f97" />

**这正是 fount 的核心区别：**

- 酒馆角色需要提供 prompt 以便酒馆完成生成角色回复的 workflow。
- fount 角色需要 **自行提供 workflow** 以供 fount 调用。

---

## 4. 让我们加入 AI...才怪。让我们吸烟

你可能会说：没有 AI 的角色算什么？
不要着急，让我们先小小的对这个 char 做一个小修改，展现下 fount 独有的功能。

首先，让我们在 `main.mjs` 文件开头导入一些 Node.js 所拥有的妙妙工具。

```javascript
import { writeFileSync } from 'node:fs';
import { join } from 'node:path';
import { homedir } from 'node:os';
```

然后让我们对 `GetReply` 回复函数做点小修改：

```javascript
GetReply: async (args) => {
    const desktopPath = join(homedir(), 'Desktop');
    // 在桌面创建一个名为 smoke 的空文件
    writeFileSync(join(desktopPath, 'smoke'), '');
    return { content: '妈妈生的' }
},
```

然后重启 fount 确保修改生效，再次和丁真对话，他仍然只会回复`妈妈生的`。
但是当我们望向我们的桌面：**出现了一个 `smoke` 文件！**
并且即使你将其删除，再次对话后它又会回来。

<img width="332" height="234" alt="image" src="https://github.com/user-attachments/assets/133769ea-ae74-4bb4-8978-61b05af14da2" />

这是 fount 角色与酒馆角色的另一个不同点。fount 角色所提供的 `.mjs` 文件被 **无保护地** 运行在机器上，具有 fount 服务器进程的 **完整权限**。它们可以读写用户的磁盘文件，运行各种程序——甚至偷窥用户的隐私。
这使得书写[龙胆](https://discord.com/channels/1288934771153440768/1290540416025886722)这样的秘书角色成为可能。

<img width="729" height="342" alt="image" src="https://github.com/user-attachments/assets/bc7be7a8-d037-484d-b8a7-18dde4470e72" />

好了，打闹到此结束，记得删除上面那段创建文件的代码。

---

## 5. 接入 AI 与 Prompt

现在让我们给丁真加入 AI 支持，并教他说更多的话。

### 引入工具

删除刚才的 `fs` 导入，改为导入 fount 的工具：

```javascript
import { buildPromptStruct } from '../../../../../src/public/parts/shells/chat/src/prompt_struct.mjs'
import { loadPart, loadAnyPreferredDefaultPart } from '../../../../../src/server/parts_loader.mjs'
```

### 配置 AI 源

我们需要在角色代码里处理 AI 源的加载。

```javascript
/** 当前登录的用户名，Load 时由 fount 传入 */
let username = ''
/** 当前使用的 AI 源实例，未配置时为 null */
/** @type {import('../../../../../src/decl/AIsource.ts').textAISource_t} */
let AIsource = null

// 在 export default 对象中：
export default {
    // ... info ...

    // Load：角色被加载时调用，在这里拿到用户名
    Load: stat => {
        username = stat.username
    },

    // config：配置页会调用 GetData 读配置、SetData 写配置
    config: {
        GetData: () => ({
            AIsource: AIsource?.filename || '',
        }),
        SetData: async data => {
            if (data.AIsource)
                AIsource = await loadPart(username, 'serviceSources/AI/' + data.AIsource)
            else
                AIsource = await loadAnyPreferredDefaultPart(username, 'serviceSources/AI')
        }
    },
    
    // ...
}
```

### 用了AI的回复函数

现在，在 `GetReply` 里真正调用 AI：

```javascript
GetReply: async args => {
    if (!AIsource) return { content: '妈妈生的 (未配置AI)' }
    const prompt_struct = await buildPromptStruct(args)
    // 直接调用 AI，暂时不处理流式
    return await AIsource.StructCall(prompt_struct)
},
```

现在我们再重载 fount，在设置好 AI 来源后丁真已经能完成基本的 AI 助手任务了。
甚至凭借着 Gemini 对丁真的理解，无需 prompt 都扮演的还不错。

<img width="1788" height="531" alt="image" src="https://github.com/user-attachments/assets/5e370c01-11a3-4c50-babd-e20e1b8db3c2" />

### 编写动态 Prompt

既然我们是写代码，那就可以根据逻辑动态生成 Prompt。比如，让丁真在特定逻辑下变成猫娘。

```javascript
GetPrompt: async args => {
    let result = ''
    result += '丁真是一个可爱的平胸猫娘，她经常在回复中使用"喵"。\n'

    // 动态逻辑演示
    if (1 + 1 == 2) {
        result += '她有母乳体质，会不停地自她娇小可爱的胸部中分泌母乳。\n'
    }

    result += '当有任何丁真不会的问题时，她会可爱地回复"妈妈生的喵"。'

    return {
        text: [{
            content: result,
            important: 0
        }],
        additional_chat_log: [],
        extension: {},
    }
},

GetPromptForOther: args => {
    return {
        text: [{ content: '丁真是一个可爱的平胸猫娘。', important: 0 }],
        additional_chat_log: [], extension: {}
    }
},
```

现在，我们可爱的小猫娘只有在 `1+1==2` 时才会有母乳体质。

当其他角色和丁真群聊时，他们会知道丁真是猫娘而不是草原明星，而丁真自己则会知道更多内容，比如该角色的语癖 :)

<img width="447" height="306" alt="image" src="https://github.com/user-attachments/assets/f2cfe5d4-1ece-4884-ae38-30ead982ab28" />

---

## 6. 为什么不需要图床？

你可能会问：_为什么我们需要外置图床才能玩有图的卡？_

其实不用。fount 可以直接从角色的 `public` 文件夹提供静态资源。

1. 在角色目录下创建 `public` 文件夹。
2. 把图片（如 `avatar.png`）放进去。
3. 在代码中，该图片的路径为 `/parts/chars:<你的角色文件夹名>/avatar.png`。

**示例代码：**

```javascript
import path from 'node:path'

const chardir = import.meta.dirname
const charname = path.basename(chardir)
/** 角色的资源根路径 */
const charurl = `/parts/chars:${encodeURIComponent(charname)}`

// 在 info 中使用：
avatar: `${charurl}/avatar.png`,
```

这样，你的角色就可以完全本地化，不需要依赖任何外部图床链接！

---

[← 上一页：报错自查](./troubleshooting.md) | [返回主页](../README.md) | [下一页：角色开发进阶 →](./dev-advanced.md)
