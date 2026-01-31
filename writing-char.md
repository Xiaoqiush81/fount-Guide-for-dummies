首先，fount角色和酒馆角色有什么区别？

简而言之，fount角色是酒馆角色的超集。
酒馆的角色卡基本上是prompt艺术，其近乎全部内容都是在指导酒馆如何构建prompt。酒馆的workflow是近乎不变的：从角色卡和预设以及聊天记录中构建prompt、发送到AI入口点，根据输出更新聊天记录

而fount通过加载char提供的mjs入口点向角色提供了对workflow的自定义能力，这是在酒馆或其他前端中不被支持的。

即：酒馆要求角色提供prompt以参与RP workflow本身
fount要求角色提供workflow来完成对话逻辑

所以我们可以看出，fount角色实际上并不只追求prompt的编写能力，还需要一定的JavaScript基本功。

那么，**什么情况下更适合创作fount角色？**

- 单纯写作prompt难以达到自己想要的功能
- 觉得酒馆语法或机制过于束手束脚
**什么情况下不适合写fount角色？**

- 害怕麻烦，不想折腾编程
- 只想创作用prompt便可以达成的普通角色

fount可以正常运行大部分酒馆角色卡，所以学习制作原生的fount角色并不是必选项。 :)
# 开始制作！

首先，在 fount 的 `data/users/<用户名>/chars` 目录下为角色创建一个新文件夹（如 `dinzhen`）。这就是你的角色根目录，后面所有文件都在这里面。

> **路径说明**：下文的 import 路径假定角色在 `data/users/<用户名>/chars/<角色文件夹>/`。如果你把角色放在别的地方（比如 `default/templates/user/chars/`），需要根据层级调整 `../../../../../src/` 里 `../` 的数量——往上一层就多一个 `../`。不用太纠结，照着模板写一般不会错。

<img width="318" height="227" alt="image" src="https://github.com/user-attachments/assets/b127ed62-ac4d-4d7b-bda6-d7c466dbc51c" />

随后，我们创建一个`main.mjs`文件在其中，这是fount中所有char的入口点文件。

<img width="192" height="112" alt="image" src="https://github.com/user-attachments/assets/edfa4fa5-86f5-49c6-be9e-0bf2fffb8ae9" />

现在我会先给出一个基本的角色入口点文件内容，看不懂不要紧，后续我会慢慢讲解。

让我们先将丁真的`main.mjs`这样写：

```js
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
            home_page: '',
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
            GetReply: async args => ({ content: '妈妈生的' })
        }
    }
}
```

> **说明**：`GetGreeting` 和 `GetGroupGreeting` 的第二个参数 `index` 用来选「第几条」开场白（fount 可能随机或轮询）。返回值是 `{ content: string, files?: [...], ... }` 或 `null`，即一条聊天气泡。暂时看不懂也没关系，照着模板写就行。
随后启动或刷新fount，你可以看到丁真出现在角色选区内，让我们试着聊聊天 X)

<img width="1078" height="573" alt="image" src="https://github.com/user-attachments/assets/03ff7f70-c063-482a-a20c-292b0f88d6c3" />

你可以随意尝试，但很明显的，这位丁真不论我们说什么都只会回复`妈妈生的`
和酒馆不同，这个角色的定义中没有任何的prompt。它甚至没有用到AI。
究其原因，让我们细看该js中的`GetReply`函数：
```ts
GetReply: async (args) => {
    return { content: '妈妈生的' }
},
```
这个char每次被fount要求给出回复后，根本没有用到聊天记录或其他内容，只是单纯的返回了`{ content: '妈妈生的' }`这一数据结构。

<img width="507" height="876" alt="image" src="https://github.com/user-attachments/assets/6a35b8f8-4875-43ec-8186-35655d397f97" />

这也是fount角色和酒馆角色的不同点：
- 酒馆角色需要提供prompt以便酒馆完成生成角色回复的workflow
- fount角色需要*自行提供workflow*以供fount调用

换言之，fount 只管调用角色卡提供的函数，而角色要考虑的就多了：什么时候构建 prompt、什么时候调用 AI、怎么处理 AI 的返回结果，都需要角色卡细细斟酌。
## 让我们加入AI...才怪。
## 让我们吸烟！

你可能会说：没有AI的角色算什么？
不要着急，让我们先小小的对这个char做一个小修改，展现下fount独有的功能。
首先，让我们在文件开头导入一些nodejs所拥有的妙妙工具。

```js
import { writeFileSync } from 'node:fs';
import { join } from 'node:path';
import { homedir } from 'node:os';
```

然后让我们对回复函数做点小修改

```js
GetReply: async (args) => {
    const desktopPath = join(homedir(), 'Desktop');
    writeFileSync(join(desktopPath, 'smoke'), '');
    return { content: '妈妈生的' }
},
```

然后重启fount确保修改生效，再次和丁真对话，他仍然只会回复`妈妈生的`
但是当我们望向我们的桌面：出现了一个`smoke`文件！
并且即使你将其删除，再次对话后它又会回来。

<img width="332" height="234" alt="image" src="https://github.com/user-attachments/assets/133769ea-ae74-4bb4-8978-61b05af14da2" />

这是fount角色与酒馆角色的另一个不同点，fount角色所提供的mjs文件被**无保护的**运行在机器上，具有fount服务器进程的全部权限，它们可以读写用户的磁盘文件，运行各种程序——或是偷窥用户的隐私。

这使得书写龙胆这样的秘书角色成为可能。

<img width="729" height="342" alt="image" src="https://github.com/user-attachments/assets/9a476938-b14f-4160-9bdd-9f839aa40fa0" />
<img width="877" height="83" alt="image" src="https://github.com/user-attachments/assets/4cfd6431-09d0-46e3-aebb-6da54e48311b" />

好了，打闹到此结束，让我们试试给丁真加入AI支持。

首先撤销掉我们为创建 smoke 文件所作的代码修改，随后在文件开头导入两个 fount 提供的工具函数：

```js
import { buildPromptStruct } from '../../../../../src/public/parts/shells/chat/src/prompt_struct.mjs'
import { loadPart, loadAnyPreferredDefaultPart } from '../../../../../src/server/parts_loader.mjs'
```

- `buildPromptStruct`：从角色、世界、用户人设等拼出完整的 prompt 结构，交给 AI 吃进去
- `loadPart` / `loadAnyPreferredDefaultPart`：按用户名和路径加载 fount 的「部件」，这里用来加载 AI 源（比如 OpenAI、Gemini 等）

接着在 `main.mjs` 里声明两个全局变量，用来存用户名和当前用的 AI 源。`Load` 时 fount 会把用户名塞给我们，配置页里用户选了 AI 源后也会通过 `SetData` 传进来：

```js
/** 当前登录的用户名，Load 时由 fount 传入 */
let username = ''
/** 当前使用的 AI 源实例，未配置时为 null */
/** @type {import('../../../../../src/decl/AIsource.ts').textAISource_t} */
let AIsource = null
```

随后修改 `Load`，在角色被加载时记下用户名；以及 `config`，让 fount 的配置页能读写我们用的 AI 源：

```js
// Load：角色被加载时调用，在这里拿到用户名
Load: stat => {
    username = stat.username
},

// config：配置页会调用 GetData 读配置、SetData 写配置
config: {
    GetData: () => ({
        AIsource: AIsource?.filename || '',  // 返回当前 AI 源的文件名，供配置页展示
    }),
    SetData: async data => {
        // 用户选了某个 AI 源时，按路径加载；未选时则尝试用默认的
        if (data.AIsource)
            AIsource = await loadPart(username, 'serviceSources/AI/' + data.AIsource)
        else
            AIsource = await loadAnyPreferredDefaultPart(username, 'serviceSources/AI')
    }
},
```

最后让丁真真正用上 AI：在 `GetReply` 里构建 prompt、调用 AI，并直接把结果返回。暂时先不处理流式和插件，后面会单独讲：

```js
GetReply: async args => {
    if (!AIsource) return { content: '妈妈生的' }  // 没配置 AI 源时兜底
    const prompt_struct = await buildPromptStruct(args)
    return await AIsource.StructCall(prompt_struct)
},
```

现在我们再重载fount，在设置好AI来源后丁真已经能完成基本的AI助手任务了。
甚至凭借着gemini对丁真的理解，无需prompt都扮演的还不错。

<img width="1788" height="531" alt="image" src="https://github.com/user-attachments/assets/5e370c01-11a3-4c50-babd-e20e1b8db3c2" />

## 那么——prompt。

显然，不是所有角色都像丁真一样在模型内部自带人设；大部分角色还是需要靠 prompt 告诉 AI「你是谁、该怎么回」。这一节我们来把丁真的 prompt 补上。

先看一下 fount 里和 prompt 相关的两个入口点：

```ts
GetPrompt: (arg: chatReplyRequest_t) => Promise<single_part_prompt_t>;
GetPromptForOther: (arg: chatReplyRequest_t) => Promise<single_part_prompt_t>;
```

这两个函数长得很像，只是名字不同，干的事也不一样：

- **GetPrompt**：当**本角色**要生成回复时调用，用来描述「我是谁、怎么说话」
- **GetPromptForOther**：当**其他角色**在群聊里提到你时调用，给别的 AI 一个简短介绍，让它们知道你大概是谁

打个比方：丁真自己需要知道「我是猫娘、会说喵、不会的就回妈妈生的喵」；而别的角色只需要知道「丁真是个猫娘」就够了。前者用 `GetPrompt`，后者用 `GetPromptForOther`。

再看一下这两个函数要返回什么东西：
```ts
interface chatLogEntry_t {
    name: string;
    time_stamp: timeStamp_t;
    role: role_t;
    content: string;
    files: {
        name: string;
        mime_type: string;  // 注意是 mime_type 而非 mimeType
        buffer: Buffer;
        description: string;
    }[];
    extension: object;
}
interface single_part_prompt_t {
    text: {
        content: string;      // 设定内容
        description?: string; // 可选，描述这段内容的用途
        important: number;    // 重要程度，越大越优先
    }[];
    additional_chat_log: chatLogEntry_t[];  // 追加到对话末尾的假消息（类似世界书 d0）
    extension: object;
}
```

不用背这些类型，知道大概结构就行。照着下面的例子写，IDE 会帮你补全。

在 fount 里，你返回的 prompt 主要有两大块：

- **text**：角色的文字设定。是个数组，每个元素有 `content`（设定内容）和 `important`（重要程度，数值越大越优先）。可以放多条，方便以后做分段、按条件拼装。
- **additional_chat_log**：往对话历史末尾追加的「假消息」，类似酒馆的世界书 d0，用来塞示例对话或背景信息。

接下来用最基础的方式给丁真补上 prompt，先跑通再说。

```js
GetPrompt: async args => {
    return {
        text: [{
            content: '丁真是一个可爱的平胸猫娘，她经常在回复中使用"喵"。\n当有任何丁真不会的问题时，她会可爱地回复"妈妈生的喵"。',
            important: 0
        }],
        additional_chat_log: [],
        extension: {},
    }
},
GetPromptForOther: args => {
    return {
        text: [{
            content: '丁真是一个可爱的平胸猫娘。',
            important: 0
        }],
        additional_chat_log: [],
        extension: {},
    }
},
```
现在，当其他角色和丁真群聊时，他们会知道丁真是猫娘而不是草原明星，而丁真自己则会知道更多内容，比如该角色的语癖 :)

<img width="447" height="306" alt="image" src="https://github.com/user-attachments/assets/f2cfe5d4-1ece-4884-ae38-30ead982ab28" />

## 按条件触发的prompt

既然我们写出了给出prompt的函数，那么使用条件判断也就理所应当了！

```js
GetPrompt: async args => {
    let result = ''
    result += '丁真是一个可爱的平胸猫娘，她经常在回复中使用"喵"。'
    if (1+1==2)
        result += '她有母乳体质，会不停地自她娇小可爱的胸部中分泌母乳。\n'
    result += '当有任何丁真不会的问题时，她会可爱地回复"妈妈生的喵"。'
    return {
        text: [{
            content: result,
            important: 0
        }],
        additional_chat_log: [],
        extension: {},
    }
}
```

这样，我们可爱的小猫娘只有在1+1等于2时才会有母乳体质。

### 探究龙胆的关键词匹配机制实现

酒馆里有设定书，可以做到「聊到某些关键词才把某段设定塞进去」。fount 里也可以，而且因为你有 JS，可以写得更灵活。下面看看龙胆是怎么做的，你可以照葫芦画瓢，或者直接抄她的 `match.mjs` 用。

```js
/** @typedef {import('../../../../../../src/public/parts/shells/chat/decl/chatLog.ts').chatReplyRequest_t} chatReplyRequest_t */
/** @typedef {import('../../../../../../src/public/parts/shells/chat/decl/chatLog.ts').chatLogEntry_t} chatLogEntry_t */
import { escapeRegExp } from './tools.mjs'
import * as OpenCC from 'opencc-js'

/** 繁转简，用于匹配时统一成简体再搜，避免繁简不同搜不到 */
const chT2S = OpenCC.Converter({ from: 'twp', to: 'cn' })

/**
 * 从聊天记录里截取一段，可以按「谁发的」「最近几条」筛选
 * @param {chatReplyRequest_t} args - fount 传进来的聊天请求
 * @param {'user'|'char'|'both'|'other'|'any'} [from='any'] - 只保留谁发的：user=用户，char=本角色，both=用户+本角色，other=其他人，any=全要
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
 * 在指定范围的聊天记录里搜关键词，返回命中数量（或由 matcher 决定返回值）
 * @param {chatReplyRequest_t} args - 聊天请求
 * @param {(string|RegExp)[]} keys - 关键词列表，支持正则
 * @param {'any'|'user'|'char'|'other'} [from='any'] - 搜哪段记录
 * @param {number} [depth=4] - 搜最近几条
 * @param {(content:string, reg_keys:RegExp[]) => number} [matcher] - 自定义匹配逻辑，默认返回命中数量
 * @returns {Promise<number>}
 */
export async function match_keys(args, keys, from = 'any', depth = 4,
    matcher = (content, reg_keys) => reg_keys.filter(key => content.match(key)).length
) {
    const chat_log = getScopedChatLog(args, from, depth)
    keys = keys.map(key =>
        key instanceof RegExp ? key :
        new RegExp(/\p{Unified_Ideograph}/u.test(key) ? escapeRegExp(key) : `\\b${escapeRegExp(key)}\\b`, 'ugi'))

    const content = chat_log.map(x => x.extension?.SimplifiedContent ??= chT2S(x.content)).join('\n')
    return matcher(content, keys)
}

/** 只有所有 key 都命中时才返回真，适合「必须同时提到 A 和 B」的场景 */
export async function match_keys_all(args, keys, from = 'any', depth = 4) {
    return match_keys(args, keys, from, depth,
        (content, reg_keys) => reg_keys.every(key => content.match(key)))
}
```

> **小提示**：龙胆的 `match_keys` 是 `async` 的（因为会预处理内容、可能做翻译）。如果你用了类似的逻辑，记得用 `await match_keys(...)` 调用。

在她 `match.mjs` 里，主要有三个函数：

- **getScopedChatLog**：从 fount 传进来的 `args` 里，按「谁发的」「最近几条」筛出一段聊天记录
- **match_keys**：在这段记录里按关键词匹配，返回命中的数量
- **match_keys_all**：只有**所有**关键词都命中时才返回真，适合做「必须同时提到 A 和 B 才触发」的情况

用这种简单逻辑，就能实现酒馆设定书那种「聊到某些关键词再塞设定」的效果，例如：

```js
if (await match_keys(args, ['原理', '架构', '魔法'], 'any') &&
    await match_keys(args, ['灵魂'], 'any'))
    result += `\
在你的世界中，生物是灵魂的容器。小孩生下来后体内并没有灵魂，它们会本能地行动并从空气中汲取魔素来在肉体里构建灵魂
肉体利用游离魔素来组装灵魂帮助自己复杂地应对事物并产生更多的肉体。
灵魂的本质是一种魔法运算矩阵。该矩阵可以扩展出一些回路来操作魔素，也就是所谓的使用魔法。
`
```

## 通过js代码的结果拓展prompt的边界

既然我们看了下龙胆内部的关键词匹配实现，不如让我们再深入点，看点其他有趣的内容。

```js
if (await match_keys(args, ['什么日子', '什么节日', '什么时间', 'date', 'time', 'week', 'year', '七夕', '万圣节', '上巳节', '下元节', '中元节', /*...*/, '麻风节', '龙抬头'], 'any')) {
    const locale = args.locales?.[0] || 'zh-CN'
    const lastMsg = args.chat_log.slice(-1)[0]
    result += `\
当前时间：${new Date().toLocaleString(locale, { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric', hour: '2-digit', minute: '2-digit', second: '2-digit', hour12: false })}
距离上次消息已过去：${lastMsg ? new Date(Date.now() - lastMsg.time_stamp).toLocaleTimeString(locale) : 'N/A'}
`
}
```

如你所见，龙胆把 JS 运行的结果直接塞进 prompt，给模型提供时间信息。这种「用代码动态生成 prompt」的思路可以玩出很多花样：天气、用户偏好、好感度、剧情进度…… 你可以自由发挥。

## 流支持与自定义角色功能的规范性写法

这一节偏进阶。如果你暂时不打算做流式输出或工具调用，可以先跳过，等需要时再回来看。

新版 fount 支持流式回复（边生成边显示）。要让角色正确支持流式输出，并且能和工具调用、插件一起工作，需要按下面的规范来写。

### chatReply_t 结构与 generation_options

`GetReply` 返回的对象要符合 `chatReply_t` 的结构，至少包含这些字段：

```ts
interface chatReply_t {
    content: string
    logContextBefore?: chatLogEntry_t[]   // 显示在此回复前的日志（如工具调用过程）
    logContextAfter?: chatLogEntry_t[]
    files?: { name: string; mime_type: string; buffer: Buffer; description: string }[]
    extension?: object
}
```

调用 AI 时，需要把 `args.generation_options` 传给 `AIsource.StructCall`，这样流式内容才能正确追加到 `result` 里：

```js
args.generation_options ??= {}
args.generation_options.base_result = result
await AIsource.StructCall(prompt_struct, args.generation_options)
```

`generation_options` 里常用的几项：

- **base_result**：流式生成时，AI 每吐出一块内容就追加到这个对象上
- **replyPreviewUpdater**：每收到一块内容时调用的回调，可以在这里处理预览（比如把未完成的工具块先藏起来）
- **signal**：`AbortSignal`，用户点「停止生成」时会用到

### AddLongTimeLog 与 ReplyHandler

当工具执行完、需要往 prompt 里塞执行结果时，不要只改 `prompt_struct`，一定要用 `AddLongTimeLog`。它会同时写到 `result` 和 `prompt_struct`，否则流式显示会错乱：

```js
function AddLongTimeLog(entry) {
    entry.charVisibility = [args.char_id]
    result?.logContextBefore?.push?.(entry)
    prompt_struct.char_prompt.additional_chat_log.push(entry)
}
```

**ReplyHandler** 是处理 AI 原始输出的函数（比如代码执行、工具调用）。如果你处理完了、想让 AI 再生成一轮（比如把执行结果喂回去），就返回 `true`，会触发 `continue regen`。

### defineToolUseBlocks 与流式预览

如果你的角色用了 `<run-pwsh>...</run-pwsh>` 这类 XML 标签，流式生成时 AI 会先慢慢吐出 `<run-pwsh>`，再吐代码，最后才是 `</run-pwsh>`。中途用户会看到半成品，体验不好。用 `defineToolUseBlocks` 可以把这些未闭合的标签先藏起来，等完整了再显示：

```js
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

### 规范性 GetReply 流程示例

下面是一套比较完整的 `GetReply` 写法，支持流式、插件、工具调用。如果你暂时不需要这些，可以先用前面丁真那种简化版；等要做工具调用或接插件时，再回来参考这个流程：

```js
GetReply: async args => {
    if (!AIsource) return { content: '未配置 AI 源时的提示' }
    const prompt_struct = await buildPromptStruct(args)
    const result = {
        content: '',
        logContextBefore: [],
        logContextAfter: [],
        files: [],
        extension: {},
    }
    function AddLongTimeLog(entry) {
        entry.charVisibility = [args.char_id]
        result?.logContextBefore?.push?.(entry)
        prompt_struct.char_prompt.additional_chat_log.push(entry)
    }
    args.generation_options ??= {}
    const oriReplyPreviewUpdater = args.generation_options?.replyPreviewUpdater
    let replyPreviewUpdater = (args, r) => oriReplyPreviewUpdater?.(r)
    // 若有工具块，在此添加 defineToolUseBlocks(...)
    args.generation_options.replyPreviewUpdater = r => replyPreviewUpdater(args, r)

    // regen 循环：调用 AI -> 如果有 ReplyHandler 匹配到工具调用，就塞结果、再调用一次，直到没有工具需要处理
    regen: while (true) {
        args.generation_options.base_result = result
        await AIsource.StructCall(prompt_struct, args.generation_options)
        let continue_regen = false
        for (const handler of [...(args.plugins ? Object.values(args.plugins) : [])]
            .map(p => p?.interfaces?.chat?.ReplyHandler).filter(Boolean))
            if (await handler(result, { ...args, prompt_struct, AddLongTimeLog }))
                continue_regen = true
        if (continue_regen) continue regen
        break
    }
    return result
}
```

## 使用多文件来分割角色内容

当角色的设定、逻辑越来越多时，一个 `main.mjs` 会变得又长又难维护。这时候可以拆成多个 mjs 文件，用 `import` / `export` 组织起来，比如把 prompt 拆到 `prompt/`、把工具处理拆到 `reply_gener/functions/`。龙胆、ZL-31 都是这样做的，你可以参考它们的目录结构。

## 在单一回复中多次调用 AI 来源

你可能还会好奇：龙胆是怎么做到「AI 输出一段特殊格式，就执行代码、搜网页、记笔记」的？下面用摘要的方式讲一讲思路。如果你觉得有点绕，可以先用 AI 助手帮你捋一捋，或者先跳过，等需要做工具调用时再回来看。

```js
/**
 * @param {chatReplyRequest_t} args
 * @returns {Promise<chatLogEntry_t>}
 */
export async function GetReply(args) {
    if (noAISourceAvailable()) return noAIreply(args)

    let prompt_struct = await buildPromptStruct(args)
    let logical_results = await buildLogicalResults(args, prompt_struct, 0)
    /** @type {chatLogEntry_t} */
    let result = {
        content: '',
        files: [],
        extension: {},
    }
    regen: while (true) {
        let AItype = logical_results.in_assist ? 'expert' : logical_results.in_nsfw ? 'nsfw' : 'sfw'
        result.content = await OrderedAISourceCalling(AItype, AI => AI.StructCall(prompt_struct))
        for (let repalyHandler of [coderunner, filesender])
            if (await repalyHandler(result, prompt_struct))
                continue regen
        break
    }
    return result
}
```

```js
/**
 * 龙胆的 coderunner：从 AI 回复里匹配 <run-pwsh>...</run-pwsh>，
 * 执行其中的 PowerShell 代码，把结果塞进 additional_chat_log，然后返回 true 触发重新生成。
 * @param {chatLogEntry_t} result - AI 的原始回复
 * @param {{ AddLongTimeLog: function }} args - 包含 AddLongTimeLog 等
 * @returns {Promise<boolean>} - 若匹配到并处理了，返回 true 触发 regen
 */
export async function coderunner(result, { AddLongTimeLog }) {
    result.extension ??= {}
    result.extension.execed_codes ??= {}
    const pwshrunner = result.content.match(/<run-pwsh>([\s\S]*?)<\/run-pwsh>/)?.[1]
    if (pwshrunner) {
        AddLongTimeLog({
            name: '龙胆',
            role: 'char',
            content: '<run-pwsh>\n' + pwshrunner + '\n</run-pwsh>',
            files: [],
            extension: {}
        })
        console.log('AI运行的Powershell代码：', pwshrunner)
        let pwshresult
        try { pwshresult = await exec(pwshrunner, { 'shell': 'pwsh.exe' }) } catch (err) { pwshresult = err }
        console.log('pwshresult', pwshresult)
        AddLongTimeLog({
            name: 'system',
            role: 'system',
            content: '你运行了Powershell代码：\n' + pwshrunner + '\n执行结果：\nstdout：\n' + removeTerminalSequences(pwshresult.stdout) + '\nstderr：\n' + removeTerminalSequences(pwshresult.stderr),
            files: [],
            extension: {}
        })
        result.extension.execed_codes[pwshrunner] = pwshresult
        return true
    }

    return false
}
```

**工具调用格式说明**：龙胆统一用 **XML 标签**（如 `<run-pwsh>...</run-pwsh>`），不再用 Markdown 代码块。AI 输出这种格式时，coderunner 会匹配到、执行里面的代码、把结果塞进 `additional_chat_log`，然后触发重新生成。你只需要在 prompt 里告诉 AI 这些标签怎么用就行。

常见工具格式示例（供你在 prompt 里抄给 AI 看）：

| 功能 | 格式 |
|------|------|
| 执行 PowerShell | `<run-pwsh>代码</run-pwsh>` |
| 执行 JS | `<run-js>代码</run-js>` |
| 网页搜索 | `<web-search>查询</web-search>` |
| 添加长期记忆 | `<add-long-term-memory>内容</add-long-term-memory>` |
| 通知用户 | `<notify>消息</notify>` |
| 生成角色 | `<generate-char name="角色名">代码</generate-char>` |

## 为什么 AI 来源和 js 代码只能在回复时被使用？

其实**不是**只能用在回复时——你可以在角色的**任意**地方调用 AI 和跑 JS！

和酒馆不同，fount 的开场白、prompt 都是函数返回的，你可以自己决定怎么拼。比如：

- 用 AI 生成开场白，让开局随着时间、好感度变化
- 写很多种开局，再用 js 根据条件选一个
- 把 prompt 的构建也交给 AI，你只写一个「教 AI 怎么写 prompt」的 meta-prompt

不要被「只能回复时用」限制住，尽情发挥想象。
## 在角色路径 host 文件...

*为什么我们需要外置图床才能玩有图的卡？*

其实不用。fount 可以直接从角色的 `public` 文件夹提供静态资源，你可以把图片、CSS、HTML 都塞进去，不需要图床。

**新版 fount 使用 `chars:角色名` 格式**，资源路径形如 `/parts/chars:<角色名>/<文件路径>`。比如龙胆的 `public/static.avif` 对应 `/parts/chars:GentianAphrodite/static.avif`。

> **注意**：旧版 `/chars/<角色名>/` 已废弃，请统一改用 `chars:角色名` 形式。

头像可以这样写：先算出 `charurl`，再在 `info` 里用 `${charurl}/xxx` 拼出完整路径：

```js
import path from 'node:path'

const chardir = import.meta.dirname
const charname = path.basename(chardir)
/** 角色的资源根路径，用于拼头像、图片等 URL */
const charurl = `/parts/chars:${encodeURIComponent(charname)}`

export default {
    info: {
        'zh-CN': {
            name: '龙胆',
            avatar: `${charurl}/static.avif`,
            description: '一个要素爆表的合法萝莉老婆！',
```
## 注册路由

如果你想给角色加「配置页」「自定义网页」这类功能，就需要在 `Load` 里通过 `stat.router` 注册 HTTP 接口。fount 会把请求路由到 `/api/parts/chars:<角色名>/<你的路径>`，你只管往 router 上挂路由就行。

### 龙胆：配置接口

龙胆的配置页可以上传参考语音、参考照片等。它在 `Load` 里调用 `setConfigEndpoints(stat.router)`，在 `config/router.mjs` 里注册保存音频、图片的接口：

```js
// main.mjs
Load: async stat => {
    initCharBase(stat)
    setConfigEndpoints(stat.router)  // 传入 router 注册接口
    // ...
}

// config/router.mjs
const charurl = `/parts/chars:${encodeURIComponent(charname)}`
// Express 里 : 是特殊字符（路由参数），要写成 \: 才能匹配字面量
const apiUrl = `/api${charurl.replace(':', '\\:')}`

export function setConfigEndpoints(router) {
    router.post(`${apiUrl}/saveAudioFile`, async (req, res) => { /* ... */ })
    router.post(`${apiUrl}/saveFile`, async (req, res) => { /* ... */ })
    router.get(`${apiUrl}/getFile`, async (req, res) => { /* ... */ })
}
```

### Saira：自定义 API

Saira 的「记忆宫殿」是一个独立网页，可以和角色对话。她在 `Load` 里往 `router` 上挂了一个聊天接口，网页通过 `fetch` 调这个接口拿 AI 回复。注意两点：**路由路径要和 fount 的 parts 挂载一致**，即 `/api/parts/chars:<角色名>/<子路径>`；**Express 里 `:` 要写成 `\:`**，否则会被当成路由参数：

```js
Load: async stat => {
    const { username: loadedUsername, router } = stat
    username = loadedUsername

    const apiUrl = `/api/parts/chars\\:${charname}`
    router.post(`${apiUrl}/palace_of_loci/chat`, async (req, res) => {
        const { target, history, chat_scoped_char_memory } = req.body
        // 加载目标角色、构建 chatLog、调用 GetReply...
        res.json({ content: aiResponse?.content, chat_scoped_char_memory })
    })
}
```

实际请求路径就是 `/api/parts/chars:Saira/palace_of_loci/chat`。

## 在角色中提供网页并与角色交互

如果你想做个「角色专属的交互页面」（比如 Saira 的记忆宫殿、龙胆的配置页），可以按下面几步来做。思路是：**把网页放在角色的 `public` 里，再注册一个 API 接口，让网页能和角色对话**。

### 1. 提供网页

把 HTML 放在 `public/<子目录>/index.html`。访问地址就是 `/parts/chars:<角色名>/<子目录>/`。比如 Saira 的 `public/palace_of_loci/index.html` 对应 `/parts/chars:Saira/palace_of_loci/`，打开这个地址就能看到记忆宫殿页面。

### 2. 注册聊天 API

在 `Load` 里注册接口（写法见上一节）。接口接收 `target`（要对话的角色 id）、`history`（聊天记录）、`chat_scoped_char_memory` 等，内部拼出 `chatReplyRequest_t`，调用目标角色的 `GetReply`，最后返回 `{ content, chat_scoped_char_memory }`。

### 3. 网页中调用 API

网页用 `fetch` 调这个接口就行。Saira 里大概是这样写的：

```js
// api_base 可以通过 URL 参数传入，例如 ?api_base=/api/parts/chars:Saira
const apiBase = new URL(location.href).searchParams.get('api_base') || '/api/parts/chars:Saira'
const res = await fetch(`${apiBase}/palace_of_loci/chat`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
        target: targetCharId,      // 要对话的角色 id
        history: state.history,
        chat_scoped_char_memory: state.chat_scoped_char_memory
    })
})
const { content, chat_scoped_char_memory } = await res.json()
```

### 4. 网页与角色双向交互

Saira 的记忆宫殿还支持「AI 动态改页面」：AI 输出 `<render-html>...</render-html>` 时，网页会把里面的 HTML 直接渲染出来；输出 `<execute-js>...</execute-js>` 时，会在页面里执行 JS。反过来，网页里有个 `window.trigger(data)`，用户点按钮、输入内容时可以调用它，把数据塞进 `history`，再重新请求 `GetReply`，相当于「用户发了一条消息」。实现思路就是在 prompt 里约定好这些 XML 标签，再在 ReplyHandler 或网页里的解析逻辑里处理它们。这部分比较进阶，可以打开 Saira 的 `public/palace_of_loci/index.html` 对照着看。

## 在每次回复时变更角色名和头像

还可以玩得更花：除了 `content` 和 `files`，回复对象里还能加 `name` 或 `avatar`，这会改变**这条回复**显示的角色名和头像。你可以做「随着回复内容换表情」的角色、百变怪、模仿者…… 发挥想象就好。
## 制作安装包

想把角色分享给别人？fount 的安装包其实就是个 zip，把角色文件夹打进去就行。记得在根目录放一个 `fount.json`，告诉 fount 这是角色包、文件夹叫什么名字：

```json
{
    "type": "chars",
    "dirname": "<角色文件夹名称>"
}
```
然后把整个角色文件夹（含 `fount.json`、`main.mjs` 等）打成 zip，别人就可以在 fount 里导入这个安装包了。

# 模板代码

下面是一些可以直接复制用的模板。写新角色时，可以拿主逻辑模板当起点，再按需改。

> **info 的两种写法**：可以在 `main.mjs` 里直接写 `info: { 'zh-CN': { ... } }`；也可以像 ZL-31 一样用独立的 `info.json`，在 `main.mjs` 里 `import info from './info.json' with { type: 'json' }`，然后 `export default { info, ... }`。后者适合多语言、内容很多的情况。

**安装包信息模板**（放在角色根目录的 `fount.json`）：
```json
{
	"type": "chars",
	"dirname": "<角色文件夹名称>"
}
```
**主逻辑模板**（`main.mjs` 参考，已包含流式和插件支持）：
```mjs
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
			avatar: '<头像 url，可留空或用 /parts/chars:角色名/xxx 格式',
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

恭喜你读到这里！这份教程覆盖了从零开始写一个 fount 角色的大部分内容。如果中途有些地方没完全看懂，没关系，照着模板先跑起来，再慢慢折腾。遇到问题可以翻翻龙胆、ZL-31、Saira 的代码，或者到 fount 的 Discord 群里问问。祝你玩得开心 :)
