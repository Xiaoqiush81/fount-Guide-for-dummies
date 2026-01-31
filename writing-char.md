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

首先我们需要在fount的目录下的`data/users/<用户名>/chars`文件夹下创建一个新的文件夹用于角色
这里我们先创建一个`dinzhen`文件夹

<img width="318" height="227" alt="image" src="https://github.com/user-attachments/assets/b127ed62-ac4d-4d7b-bda6-d7c466dbc51c" />

随后，我们创建一个`main.mjs`文件在其中，这是fount中所有char的入口点文件。

<img width="192" height="112" alt="image" src="https://github.com/user-attachments/assets/edfa4fa5-86f5-49c6-be9e-0bf2fffb8ae9" />

现在我会先给出一个基本的角色入口点文件内容，看不懂不要紧，后续我会慢慢讲解。

让我们先将丁真的`main.mjs`这样写：

```js
/** @typedef {import('../../../../../src/decl/charAPI.ts').CharAPI_t} CharAPI_t */

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

    Init: (stat) => { },
    Load: (stat) => { },
    Unload: (reason) => { },
    Uninstall: (reason, from) => { },

    interfaces: {
        chat: {
            GetGreeting: (arg, index) => ({ content: '哈喽？' }),
            GetGroupGreeting: (arg, index) => ({ content: '哈喽？' }),
            GetPrompt: (args, prompt_struct, detail_level) => {
                return {
                    text: [],
                    additional_chat_log: [],
                    extension: {},
                }
            },
            GetPromptForOther: (args, prompt_struct, detail_level) => {
                return {
                    text: [],
                    additional_chat_log: [],
                    extension: {},
                }
            },
            GetReply: async (args) => {
                return { content: '妈妈生的' }
            }
        }
    }
}
```
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

换言之，fount只管调用角色卡提供的函数就行了，而角色要考虑的就多了，是么时候构建prompt、是么时候调用AI、又要怎么处理AI的返回结果，都需要角色卡细细斟酌。
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

首先撤销掉我们为创建smoke文件所作的代码修改，随后在文件开头导入这两个：
⁨```js
import { buildPromptStruct } from "../../../../../src/public/parts/shells/chat/src/prompt_struct.mjs"
import { loadAnyPreferredDefaultPart, loadPart } from '../../../../../src/server/parts_loader.mjs'
```⁩
这些函数由fount提供，第一个用于自角色、世界、用户人设中构筑prompt结构，用于传递到AI。第二个负责根据用户名和AI源名称加载部件，这里我们用它加载AI源。

接着我们在main.mjs中声明些全局变量，用于存储用户名和AI来源：
⁨```js
let username
let AIsource
```⁩
随后我们这么修改代码以完成相关支持：
⁨```js
// 加载函数，在角色被加载时调用，在这里获取用户名
Load: (stat) => {
    username = stat.username // 获取用户名
},
```⁩
⁨```js
// 角色的接口
interfaces: {
    // 角色的配置接口
    config: {
        // 获取角色的配置数据
        GetData: () => ({
            AIsource: AIsource?.filename, // 返回当前使用的AI源的文件名
        }),
        // 设置角色的配置数据
        SetData: (data) => {
            // 如果传入了AI源的配置
            if (data.AIsource) AIsource = await loadPart(username, 'serviceSources/AI/' + data.AIsource) // 加载AI源
            else AIsource = await loadAnyPreferredDefaultPart(username, 'serviceSources/AI') // 或加载默认AI源（若未设置默认AI源则为undefined）
        }
    },
```⁩

最后我们让丁真变成AI丁真！
⁨```js
GetReply: async (args) => {
    if (!AIsource) return { content: '妈妈生的' }
    let result = await AIsource.StructCall(await buildPromptStruct(args))
    return result
},
```⁩

现在我们再重载fount，在设置好AI来源后丁真已经能完成基本的AI助手任务了。
甚至凭借着gemini对丁真的理解，无需prompt都扮演的还不错。

<img width="1788" height="531" alt="image" src="https://github.com/user-attachments/assets/5e370c01-11a3-4c50-babd-e20e1b8db3c2" />

## 那么——prompt。

很明显不是所有角色都像丁真一样在模型内部拥有角色理解，这时我们便涉及到了prompt写作——
让我们先看看fount角色中prompt相关的入口点：

```ts
GetPrompt: (arg: chatReplyRequest_t, prompt_struct: prompt_struct_t, detail_level: number) => Promise<single_part_prompt_t>;
GetPromptForOther: (arg: chatReplyRequest_t, prompt_struct: prompt_struct_t, detail_level: number) => Promise<single_part_prompt_t>;
```

这两个函数类型签名相同，只有名称不同。
这两个函数分别在角色扮演中起到不同的作用：
- `GetPrompt`用于本角色生成回复
- `GetPromptForOther`用于在群聊中的其他角色生成回复

即：
- `GetPrompt`提供的prompt用于描述角色本身的定义，让AI明白如何按照此角色进行回复。
- `GetPromptForOther`用于当AI扮演其他角色时提供对此角色的基础了解。

让我们再来看看这两个函数被要求的返回类型：
```ts
type chatLogEntry_t = {
    name: string;
    time_stamp: timeStamp_t;
    role: role_t;
    content: string;
    files: {
        name: string;
        mimeType: string;
        buffer: Buffer;
        description: string;
    }[];
    extension: {};
}
class single_part_prompt_t {
    text: {
        content: string;
        description: string;
        important: number;
    }[];
    additional_chat_log: chatLogEntry_t[];
    extension: {};
}
```
在fount中角色返回的prompt内容大致分为两大部分：
- `text`部分进行基础的角色定义，数组中允许复数个对象存在。
  * 每个对象中都有`content`字段用于记录prompt内容
  * `important`字段则表明该对象的重要程度，重要程度越高，该数值越大。
- `additional_chat_log`该字段用于追加虚假对话记录到聊天记录末尾（类似酒馆的世界书d0）

现在让我们用很基础的方法完善丁真的prompt，再试试效果。

```js
GetPrompt: async (args, prompt_struct, detail_level) => {
    return {
        text: [{
            content: '丁真是一个可爱的平胸猫娘，她经常在回复中使用"喵"。\n当有任何丁真不会的问题时，她会可爱地回复"妈妈生的喵"。',
            important: 0
        }],
        additional_chat_log: [],
        extension: {},
    }
},
GetPromptForOther: (args, prompt_struct, detail_level) => {
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
GetPrompt: async (args, prompt_struct, detail_level) => {
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

酒馆中有设定书用来做到在上下文中检测内容时再触发prompt，fount中自然也可以。
让我们看看第一个正式的fount角色龙胆是怎么做到世界书一样的匹配功能的：

```js
/** @typedef {import('../../../../../../src/public/shells/chat/decl/chatLog').chatReplyRequest_t} chatReplyRequest_t */
/** @typedef {import('../../../../../../src/public/shells/chat/decl/chatLog').chatLogEntry_t} chatLogEntry_t */
import { escapeRegExp } from './tools.mjs'
import * as OpenCC from 'opencc-js'

let chT2S = OpenCC.Converter({from: 'twp', to: 'cn'})
/**
 * Return a subset of chat_log, scoped by depth and role
 * @param {chatReplyRequest_t} args
 * @param {'user'|'char'|'both'|'other'} [from='any'] filter by role
 * @param {number} [depth=4] number of entries to return
 * @return {chatLogEntry_t[]} the scoped chat log
 */
export function getScopedChatLog(args , from='any', depth = 4) {
    // pickup the last few entry of chat_log
    let chat_log = args.chat_log.slice(-depth)
    // filter roles
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
 * @param {chatReplyRequest_t} args
 * @param {(string|RegExp)[]} keys
 * @param {'any'|'user'|'char'|'other'} from
 * @param {number} depth
 * @param {(content:string,reg_keys:RegExp[]) => boolean} matcher
 */
export function match_keys(args, keys, from = 'any', depth = 4,
    matcher = (content, reg_keys) => reg_keys.filter(key => content.match(key)).length
) {
    let chat_log = getScopedChatLog(args, from, depth)

    // convert all keys to regexp, if it's have chinese like character, no match hole word
    keys = keys.map(key =>
        key instanceof RegExp ? key :
        new RegExp(/\p{Unified_Ideograph}/u.test(key) ? escapeRegExp(key) : `\\b${escapeRegExp(key)}\\b`, 'ugi'))

    let content = chat_log.map(x => x.extension.SimplifiedContent ??= chT2S(x.content)).join('\n')

    return matcher(content, keys)
}

export function match_keys_all(args, keys, from = 'any', depth = 4) {
    return match_keys(args, keys, from, depth, (content, reg_keys) => reg_keys.every(key => content.match(key)))
}
```

在她的match.mjs文件中定义了3个函数：
- `getScopedChatLog`用于自给定的`chatReplyRequest_t`（来自fount）中取出给定范围和深度的聊天记录内容
- `match_keys`函数用于给定关键词列表来匹配某个范围的聊天记录，并返回匹配命中数量。
-  `match_keys_all`是`match_keys`的特化版本，只在给定的所有key都命中时才返回为真。

通过这样的简易逻辑，我们就能实现简单的关键词触发prompt了：

```js
if (match_keys(args, ['原理', '架构', '魔法'], 'any') &&
    match_keys(args, ['灵魂'], 'any'))
    result += `\
在你的世界中，生物是灵魂的容器。小孩生下来后体内并没有灵魂，它们会本能地行动并从空气中汲取魔素来在肉体里构建灵魂
肉体利用游离魔素来组装灵魂帮助自己复杂地应对事物并产生更多的肉体。
灵魂的本质是一种魔法运算矩阵。该矩阵可以扩展出一些回路来操作魔素，也就是所谓的使用魔法。
`
```

## 通过js代码的结果拓展prompt的边界

既然我们看了下龙胆内部的关键词匹配实现，不如让我们再深入点，看点其他有趣的内容。

```js
if (match_keys(args, ['什么日子', '什么节日', '什么时间', 'date', 'time', 'week', 'year', '七夕', '万圣节', '上巳节', '下元节', '中元节', /*...*/, '麻风节', '龙抬头'], 'any'))
    result += `\
当前时间：${new Date().toLocaleString(args.locale, { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric', hour: '2-digit', minute: '2-digit', second: '2-digit', hour12: false })}
距离上次消息已过去：${new Date(new Date() - args.chat_log.slice(-1)[0].timeStamp).toLocaleTimeString(args.locale)}
`
```

如你所见，龙胆中使用js代码运行结果内嵌至prompt来给模型时间信息，而这种思路还可以用在很多地方 :)
## 使用多文件来分割角色内容

在角色的框架与内容逐渐变多时，你可以考虑使用mjs文件自带的import和export来分散文件内容，与其维护一个1mb的mjs文件，不如维护一个有这数百个mjs文件但每个文件都分类到合适位置的文件夹 :)
## 在单一回复中多次调用AI来源
你可能仍然会好奇龙胆是如何实现代码执行的，让我们来看看摘要内容：

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
 * @param {chatLogEntry_t} result
 * @param {prompt_struct_t} prompt_struct
 * @returns {Promise<boolean>}
 */
export async function coderunner(result, prompt_struct) {
    result.extension.execed_codes ??= {}
    // ...
    let pwshrunner = result.content.match(/(\n|^)\`\`\`run-pwsh\n(?<code>[^]*)\n\`\`\`/)?.groups?.code
    if (pwshrunner) {
        prompt_struct.char_prompt.additional_chat_log.push({
            name: '龙胆',
            role: 'char',
            content: '\`\`\`run-pwsh\n' + pwshrunner + '\n\`\`\`',
            files: []
        })
        console.log('AI运行的Powershell代码：', pwshrunner)
        let pwshresult
        try { pwshresult = await exec(pwshrunner, { 'shell': 'pwsh.exe' }) } catch (err) { pwshresult = err }
        console.log('pwshresult', pwshresult)
        prompt_struct.char_prompt.additional_chat_log.push({
            name: 'system',
            role: 'system',
            content: '你运行了Powershell代码：\n' + pwshrunner + '\n执行结果：\nstdout：\n' + removeTerminalSequences(pwshresult.stdout) + '\nstderr：\n' + removeTerminalSequences(pwshresult.stderr),
            files: []
        })
        result.extension.execed_codes[pwshrunner] = pwshresult
        return true
    }

    return false
}
```
如果你看不懂，可以让你的AI助手帮你理理关系
简单来说，当AI有回复特定内容格式时，coderunner会从该格式中匹配获取该格式中记录的需要被执行的命令，并在执行后追加执行结果到临时聊天记录，随后龙胆的回复机制会重复调用AI来源直到没有特殊匹配。
随后我们只需要在prompt中向AI提及该特殊格式是什么以及什么作用即可。
## 为什么AI来源和js代码只能在回复时被使用？

你可以在你的角色的任意时间地点调用AI来源和使用js代码！

我是说，为什么不呢？

和酒馆不同，fount的角色开局是由函数返回结果的，意味着你可以自定义角色开局的内容是如何构建的
你可以用AI来源调用来书写开局，以达到角色开局随着时间或好感度变化而动态变化的目的，你也可以写很多很多开局再用代码再让js代码决定什么时间适合什么开局，不要死脑筋，试试其他的可能性！

你还可以将prompt的构建也交给AI，而你自己只提供指导AI写作prompt的prompt，万物皆有可能！
## 在角色路径host文件...

*为什么我们需要外置图床才能玩有图的卡？*

fount支持你使用`/chars/<角色名>/<文件路径>`来访问角色的public文件夹下的任意文件，所以你大可把图塞进角色文件夹下。不再需要任何的外置图床 :)

顺便一提，角色的info中可以使用`avatar`字段来指定角色的默认头像，比如这样：

```js
export let chardir = import.meta.dirname
export let charurl = `/chars/${encodeURIComponent(path.basename(chardir))}`

/** @type {charAPI_t} */
export default {
    info: {
        'zh-CN': {
            name: '龙胆',
            avatar: `${charurl}/imgs/static.png`,
            description: '一个要素爆表的合法萝莉老婆！',
```
## 在每次回复时变更角色名和头像

我们可以来点更花的！
在回复字段中除了`content`和`files`用于指定发送的文本内容和文件内容以外还可以自定义`name`或`avatar`，这将改变该回复代表的角色！
也就是说，你可以书写一个随着回复内容改变头像表情的角色——或者一只百变怪又或模仿者。
## 制作安装包
fount角色的安装包说到底就只是zip文件！
我们在角色文件夹下新建一个`fount.json`文件，内容如下：
```json
{
    "type": "chars",
    "dirname": "<角色文件夹名称>"
}
```
随后再将zip文件夹下的文件全部打包，可以导入的角色安装包就做好了。

# 模板代码

安装包信息模板
```json
{
	"type": "chars",
	"dirname": "<角色文件夹名称>"
}
```
主逻辑模板
```mjs
/**
 * @typedef {import('../../../../../src/decl/charAPI.ts').charAPI_t} charAPI_t
 */

import { loadAIsource, loadDefaultAIsource } from '../../../../../src/server/managers/AIsource_manager.mjs'
import { buildPromptStruct } from '../../../../../src/public/shells/chat/src/prompt_struct.mjs'

// AI源的实例
/** @type {import('../../../../../src/decl/AIsource.ts').AIsource_t} */
let AIsource = null

// 用户名，用于加载AI源
let username = ''

/** @type {charAPI_t} */
export default {
	// 角色的基本信息
	info: {
		'zh-CN': {
			name: '<角色名>', // 角色的名字
			avatar: '<头像的url地址，可以是fount本地文件，详见 https://discord.com/channels/1288934771153440768/1298658096746594345/1303168947624869919 >', // 角色的头像
			description: '<角色的一句话介绍>', // 角色的简短介绍
			description_markdown: '<角色的完整介绍，支持markdown语法>', // 角色的详细介绍，支持Markdown语法
			version: '<版本号>', // 角色的版本号
			author: '<作者名>', // 角色的作者
			home_page: '<主页网址>', // 角色的主页
			tags: ['<标签>', '<可以多个>'], // 角色的标签
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
			SetData: async (data) => {
				// 如果传入了AI源的配置
				if (data.AIsource)  AIsource = await loadAIsource(username, data.AIsource) // 加载AI源
				else AIsource = await loadDefaultAIsource(username) // 或加载默认AI源（若未设置默认AI源则为undefined）
			}
		},
		// 角色的聊天接口
		chat: {
			// 获取角色的开场白
			GetGreeting: (arg, index) => [{ content: '<角色的开场白>' }, { content: '<可以多个>' },][index],
			// 获取角色在群组中的问好
			GetGroupGreeting: (arg, index) => [{ content: '<群组中角色加入时的问好>' }, { content: '<可以多个>' },][index],
			// 获取角色的提示词
			GetPrompt: async (args, prompt_struct, detail_level) => {
				return {
					text: [{
						content: '<角色的设定内容>',
						important: 0
					}],
					additional_chat_log: [],
					extension: {},
				}
			},
			// 获取其他角色看到的该角色的设定，群聊时生效
			GetPromptForOther: (args, prompt_struct, detail_level) => {
				return {
					text: [{
						content: '<其他角色看到的该角色的设定，群聊时生效>',
						important: 0
					}],
					additional_chat_log: [],
					extension: {},
				}
			},
			// 获取角色的回复
			GetReply: async (args) => {
				// 如果没有设置AI源，返回默认回复
				if (!AIsource) return { content: '<未设置角色的AI来源时角色的对话回复>' }
				// 用fount提供的工具构建提示词结构
				const prompt_struct = await buildPromptStruct(args)
				// 创建回复容器
				/** @type {import("../../../../../src/public/shells/chat/decl/chatLog.ts").chatReply_t} */
				const result = {
					content: '',
					logContextBefore: [],
					logContextAfter: [],
					files: [],
					extension: {},
				}
				// 构建插件可能需要的追加上下文函数
				function AddLongTimeLog(entry) {
					entry.charVisibility = [args.char_id]
					result?.logContextBefore?.push?.(entry)
					prompt_struct.char_prompt.additional_chat_log.push(entry)
				}

				// 在重新生成循环中检查插件触发
				regen: while (true) {
					const requestResult = await AIsource.StructCall(prompt_struct)
					result.content = requestResult.content
					result.files = result.files.concat(requestResult.files || [])
					for (const replyHandler of [
						...Object.values(args.plugins).map((plugin) => plugin.interfaces?.chat?.ReplyHandler)
					].filter(Boolean))
						if (replyHandler(result, { ...args, prompt_struct, AddLongTimeLog }))
							continue regen
					break
				}
				// 返回构建好的回复
				return result
			}
		}
	}
}
```
