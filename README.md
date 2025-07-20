# fount 白痴手册 (Fount Guide for Dummies)
一份基于我个人经验与现有教程的细化版本，旨在帮助新手更快地上手并入门fount，包含基础的搭建教程，使用注意事项与一些报错自查，由于我本人并不了解程序和编程，所以会在教程下方补充黑话之类的扫盲，该手册在我脱坑前也会一直持续更新。**禁转类脑/方舟，转发时注明作者即可，其他随意**


---

## 🧐 什么是 fount？

简单来说，**fount 是一个超级自由的、开源的 AI 角色扮演平台。**

它最大的特点就是【**完全的自由**】！不像其他平台有各种各样的限制，在 fount 的世界里，你想创造什么样的角色、进行什么样的对话，都只取决于你自己的想象力。
详情请查看项目地址[fount](https://github.com/steve02081504/fount)

## ✨ 为什么选择 fount？

*   **绝对自由**: 支持原生JS动画/无过滤html渲染：fount将这些不安全的渲染选项作为默认选项进行支持而无需安装插件，真正的无限制，你的创意是唯一的边界。
*   **高度兼容**: 支持多种主流的人物卡格式，兼容risu的charX，兼容SillyTavern的大部分不依赖插件和ST脚本的卡，可以轻松把其他地方的角色带过来玩。
*   **完全本地**: 保护你的隐私，所有对话都发生在你的电脑上。
*   **开源免费**: 社区驱动，不断进化，而且完全免费！

## 🚀 如何开始？(极速上手指南)

<details>
 <summary><b>1. 搭建教程</b></summary>

* **给超级电脑小白/懒鬼：**
如果你什么都不会，可以通过fount runner运行它。
你只需要[下载exe文件]( https://github.com/steve02081504/fount/releases/download/runner-v0.0.0.1/fount.exe )，随后点击运行即可。

* **给一般人：**
首先，和酒馆不同，fount依赖于deno而不是nodejs，所以fount的启动脚本会在启动后未找到deno时自行安装deno

如果需要，你可以参照[deno的官方文档]( https://docs.deno.com/runtime/getting_started/installation/ )进行deno的自定义安装

完成安装deno后，如果你会git，则可以考虑使用git来clone [fount的repo](https://github.com/steve02081504/fount/)
如果你不会，[点击这里下载最新版的fount]( https://github.com/steve02081504/fount/archive/refs/heads/master.zip )，fount会自动安装git并在未来每次启动时自动检查和下载更新

如果你懒得用git，在终端中输入命令也可以运行fount，详见[终端搭建教程](https://github.com/steve02081504/fount/blob/master/docs/Readme.zh-CN.md#%E5%AE%89%E8%A3%85%E5%B0%86-fount-%E7%BC%96%E7%BB%87%E5%85%A5%E4%BD%A0%E7%9A%84%E4%B8%96%E7%95%8C--%E6%AF%AB%E4%B8%8D%E8%B4%B9%E5%8A%9B)

进入后你会看到类似这样的页面，点击输入用户名和密码来创建一个账户，**其所有内容都存储在本地，数据会随着fount的删除而丢失**。
点击发送验证码后你的fount终端（就是那个黑框框）处会显示验证码内容，输入进去即可。
没有验证码就不用输入。
<img width="923" height="728" alt="image" src="https://github.com/user-attachments/assets/3073d736-3ee5-418e-b191-55f652cfaa32" />

---
**【以下为该搭建教程的~~不负责佛系扫盲~~】**

* ***预演***：在真正执行一个从网上下载的脚本之前，先把它下载到本地的一个变量里，让用户有机会检查一下脚本内容，确认安全无误之后，再手动运行它。

* ***[CAUTION]***：警告的意思。在该搭建教程里是提醒你注意fount可以原生执行JavaScript脚本，即可以直接操作你的电脑

* ***deno或nodejs***：代码的“运行环境”。它们是能让电脑读懂并运行特定程序（比如fount）的必要软件。可以理解为，fount是游戏，deno就是运行这个游戏所必须的平台或启动器。

* ***git***：一个专业的“文件历史记录工具”。它能追踪一个项目里所有文件的每一次修改，方便开发者随时查看历史版本、撤销错误操作或与他人合作。

* ***repo***：repository（仓库）的简称。在GitHub这类网站上，一个repo就代表一个项目的完整文件夹，里面存放着该项目的所有代码和文件。

* ***git clone***：一个命令，作用是“完整地复制一个repo”，clone即克隆、复制的意思。执行这个命令，就会把服务器上整个项目的文件夹（包括所有历史记录）下载到你自己的电脑里。


遇到任何问题，或仍然有疑问，**[欢迎加入我们的Discord群组！](https://discord.gg/sKdutkWQgt)**
</details>

---
<details>
 <summary><b>2. 角色导入</b></summary>

当我们进入fount后你会看到类似这样的角色选择页面，刚开始是没有角色的，莫慌，看到右上角的那个菜单按钮了吗？点它就会出现功能界面！
<img width="1919" height="1022" alt="image" src="https://github.com/user-attachments/assets/40111044-27f9-4800-a038-26dc61b37f65" />

在导入中我们有两种导入方式，一种是上传已下载的fount角色（zip档案）或酒馆角色（png，json档案）
另一种是右边的文字导入，我们可以粘贴角色网址（github网址、chub网址或risurlm网址），每行一个，随后批量导入！
<img width="1919" height="1024" alt="image" src="https://github.com/user-attachments/assets/df584df6-68c1-4a73-bd13-03cb17e3e2d2" />

---
以下是我个人在此过程出现过的一些问题与解决方法
 
 * **❓导入角色后没有显示/运行时终端爆红**

 首先尝试刷新fount页面或重启fount程序，如果你也是win10，出现了以上情况（终端爆红）建议使用[Windows terminal](https://aka.ms/terminal)，这是因为老旧的终端（CMD/PowerShell）无法正确“翻译” fount 输出的漂亮的彩色文本和特殊符号，把它们误认为了错误，而terminal则没有这个问题，它完美兼容fount输出的字符与文本，所以后续我的所有操作包括运行fount也是使用terminal而不是电脑自带的cmd或powershell

* **❓导入后角色界面出现类似下图的显示异常并伴随报错**

首先尝试重启fount，重启一般能解决百分之八十的问题，其次就是VPN或者网络的问题，尝试换个网络或VPN开启全局和虚拟网卡模式。
如果你使用的是clash或其他第三方VPN软件时出现此情况，我建议你使用[Clash rev](https://github.com/clash-verge-rev/clash-verge-rev/releases)，原因是现有的clash for Windows基本已经停止维护和更新，所以可能会出现兼容和连接问题
<img width="2483" height="1254" alt="image" src="https://github.com/user-attachments/assets/ad62e748-189b-4335-b0f1-276ce7edb106" />

* **❓打不开Microsoft Store下载Windows terminal**

使用浏览器的地址栏打开这个[链接](ms-windows-store://pdp/?productId=9N0DX20HK701)
或者去terminal的[github页面下载](https://github.com/microsoft/terminal/releases)，找到最新的Release，下滑到Assets，找Microsoft.WindowsTerminal_<版本号>_8wekyb3d8bbwe.msixbundle下载（有的叫.msixbundle，点大一点的那个）

* **❓安装Terminal后提示缺少Microsoft.UI.Xaml.2.8的应用包**

去该页面[下载依赖包](https://www.nuget.org/packages/Microsoft.UI.Xaml/)，点进去之后，在右边找到 “Download package” 下载，下载下来的是一个`.nupkg`文件。他可以把这个文件的后缀名改成`.zip`，然后解压，在里面的`tools\AppX\`文件夹里找到对应他系统架构（一般是x64）的`.appx`文件，双击安装。

<img width="812" height="508" alt="image" src="https://github.com/user-attachments/assets/1064e58e-0ed8-4baf-8d04-c6594abf03cf" />
</details>

---
<details>
 <summary><b>3. 配置AI源</b></summary>
 
到此为止我们完成了用户的创建和角色的导入，现在我们假设你有了一个**需要AI源**来运行的角色。
让我们试着配置AI源！
fount目前原生支持的AI源有[gemini](aistudio.google.com)和[cohere](cohere.com)，如果你没有他们的API Key，可以直接点击链接跳转获取，如果你不清楚Gemini的API Key的获取流程，可以参阅[哈基米API Key纯宝宝教程](https://github.com/Xiaoqiush81/Gemini-API-Key-guide)，如果你有这两个模型的官方API key，可以通过以下方式来添加它，现在我以Gemini举例：
* 在主页右上角选择管理ai源
<img width="395" height="290" alt="image" src="https://github.com/user-attachments/assets/dc165e64-0805-4dc2-9469-475409212f35" />

然后点击"+"新建一个AI源，生成器选择"gemini"，此时系统会提示你新建一个AI源名称，这个名称无特定标准，自行填写即可，继续以我这里填写的“Gemini”为例，**(后续系统自动把Gemini变成gemini了，所以这里的AI源名称实际是gemini)**，记得不要包含后缀
 <img width="832" height="209" alt="image" src="https://github.com/user-attachments/assets/9dc1a6b9-701d-4463-9f2b-1277384c08fc" />
 
* 然后在左侧的json文件中，将你的API Key填写在第3行的apikey，在第4行的model填写正确的模型名，例如我使用的是gemini-2.5-pro，不清楚模型名的请自行查询（注意模型名一定要确认正确填写噢），第一行的`name`与你实际调用的模型无关，随意填写即可，它只会影响在后台出现的调用AI源的名称，其他的地方在无特殊要求和情况下保持默认即可
<img width="1251" height="735" alt="image" src="https://github.com/user-attachments/assets/df529196-22f2-4e0b-8746-807b281ac2a8" />

这样一个新的AI来源便创建好了！
如果你的AI不是来自cohere或gemini也不要慌，fount也支持任何Open AI格式的自定义来源，我们只需要在"选择生成器"里更换成proxy

<img width="232" height="359" alt="image" src="https://github.com/user-attachments/assets/ea7716fa-cd08-4d99-86d1-46e288c80e38" />

这里我以deepseek举例，在“选择生成器”里选择proxy，然后在左侧的json文件按照如图填写，如果你也想配置deepseek，url和model可以直接复制我的，然后把`apikey`替换成你自己的key
。此处顺便附赠链接[获取deepseek的API](platform.deepseek.com)
<img width="949" height="550" alt="image" src="https://github.com/user-attachments/assets/8960d234-0f43-4198-8124-d344b7c1bc45" />

* 同理，因为fount的proxy生成器支持任何Open AI格式的自定义来源，所以如果你想把在SillyTavern中使用的轮询、反代等"兼容OpenAI"的API放入fount中使用，只需把你的URL地址和密钥填入相应的位置即可，但是需要注意的是如果你想使用proxy接口连接Gemini的轮询，**fount的proxy生成器只支持接收文件而不支持发送文件**，所以可能会出现AI识图功能失灵的情况（除此之外没有任何影响），此时我们就可以选择使用fount内置的polling轮询，接下来我会以配置Gemini的polling轮询进行示范
  
---
* **配置Gemini的polling轮询**

首先参照先前的步骤新建一个AI源，然后生成器选择"polling"，此时你左侧的json文件大概是这样的内容
<img width="1246" height="740" alt="image" src="https://github.com/user-attachments/assets/43dbc31e-7c87-4340-b0c3-b82dc0cc5eb1" />

**【此处为~~不负责佛系~~扫盲】**
  
* ***选择生成器 (polling)***:这个是“团队的工作模式”。咱们图里选的这个 polling（轮询），就像是排队模式。您给团队一个任务，团队长（也就是fount）会按照您定好的名单，从第一个AI开始问：“你能干这个活吗？”。如果它不行或者在忙，就立刻去问名单上的第二个，以此类推，直到找到一个能完成任务的AI为止。

* ***name (名称)***：这个最简单啦，就是给这个“AI工作团队”起个好记的名字，只会影响它在你后台出现的调用AI源的名字

* ***provider (服务商)***：这是指这个AI服务是由谁提供的，比如是OpenAI还是Google。因为 polling 本身只是一个“团队工作模式”，只是一个fount的AI源生成器，而不是一个具体的AI，所以这里显示"unknown"是正常的，fount是根据"generator"来决定调用谁的。

* ***sources (来源列表)***：这就是那个“团队成员名单”啦！我们可以在这个名单里，写上所有您想让它去轮流询问的AI。您可以直接写上之前配置好的另一个AI源的名字（比如我们之前新建的AI源名称是"gemini"，那么可以直接填入在此处），也可以像图里那样，把一个AI的完整配置信息直接写在这里。

* ***generator (生成器类型)***:这个是名单里某个成员的“具体身份”，比如它是OpenAI家的AI，还是Ollama家的AI，我们在这里配置的是Gemini，所以填写"gemini"即可。

* ***config (详细配置)***：这是这个成员的“个人档案”，里面记录了它的详细信息：

* ***model_name(模型名称)***：它的具体“型号”，比如是gpt-4还是claude-3。
 
* ***other_datas (其他数据)***：其他各种杂项设置，比如它的“通行证”(API Key)或者“家庭住址”(API地址)之类的，都可以塞在这里。

---
因为 polling 本身只是一个“团队工作模式”，只是一个fount的AI源生成器，而不是某一个具体的AI，所以这里provider填写"unknown"是正常的，fount是根据"generator"来决定调用谁的。

那么根据以上的了解，我们就可以直接把Gemini相关的配置格式替换进去：
```
"name": "pooling array（或者换成任意你喜欢的名字）",
	"provider": "unknown",
	"sources": [
		{
			"generator": "gemini",
			"config": {
				"apikey": "你的API Key",
				"model": "具体的模型名称（比如gemini-2.5-pro）"
```
复制以上的格式，然后无脑复制粘贴即可，你有多少个key就复制多少个

或者你仍然一头雾水，也可以直接参照下图我的polling配置，打码的位置替换你自己的API Key，如果保存失败或报错，务必检查自己的json文件格式是否正确，有没有漏括号之类的！
<img width="1282" height="751" alt="image" src="https://github.com/user-attachments/assets/c8d39bae-fdf9-426d-8799-101105350db3" />

不想遵循OpenAI的API格式？也没问题！fount支持使用main.js文件自定义来源生成器，你可以参照文件夹中的`fount/src/public/AIsourceGenerators`来写好自己的来源生成器，而无需在fount外运行一个单独的服务器来中转请求
如果你对你的作品有自信，欢迎pr到[fount](https://github.com/steve02081504/fount)使其成为fount的一部分，如果你想自己用，可以考虑将其放置在`fount/data/users/<用户名>/AIsourceGenerators`中 :)

---
**【扫盲区】**

* ***pr***：即pull request，你可以把你fork的项目修改过后的版本提交给项目的负责人供其采纳
* ***fork***：相当于把原项目完整复制到自己的仓库里，无论怎样随意修改和删除都不会影响原项目的文件，后续也可以选择将修改后的版本通过pr提交给原项目的负责人
</details>

---
<details>
 <summary><b>4. 配置角色的AI源</b></summary>
 
现在你已经创建好了一个可用的AI源，接下来就是把它接入char使其工作起来！
和酒馆不同，在fount中AI设置不是全局的，而是**每一个角色可以配置一个或多个AI源**

**【为什么要这么做？】**

或许A模型可能擅长解答问题，B模型没有NSFW屏蔽，那么我们可能会想让角色在一般情况下使用A模型，在R18情节中使用B模型
又或者C模型免费但是有点蠢，适合做一些简单的判断，D模型适合生成回复的正文，角色可以开放多个来源接口来适当调用不同用途的模型。

---
将AI来源配置到角色也很简单，首先点击角色界面上的配置按钮（有的角色可能无需配置，那么你可以直接开玩！）

<img width="129" height="104" alt="image" src="https://github.com/user-attachments/assets/a51c1cc3-830b-4263-bddd-3dd009c9c696" />

在角色配置中的"AIsources"或"AIsource"位置填写你配置好的AI源名称，不同的角色可能数据结构不同，具体请看该角色的"部件配置"。
这里我以龙胆举例，在对应的位置填写你之前创建的AI源的名字，**这里的AI源名称是指你新建源时取的名字，不是json文件中的"name"。**
比如在上一步中我新建的AI源名称是"gemini"，那么就对应图中的gemini，记得替换自己的AI源，然后在你希望这个AI源应用的地方填写对应的名称即可，比如我希望调用grok来回复nsfw的内容，调用gemini来回复sfw内容
<img width="1711" height="825" alt="image" src="https://github.com/user-attachments/assets/60471c75-0382-4c97-a1e0-2c863346be6c" />

## 到此你已经成功为角色配置好了AI源，可以直接开始和他们聊天啦！
</details>




