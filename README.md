# fount 白痴手册 (Fount Guide for Dummies)
一份基于我个人经验与现有教程的细化版本，旨在帮助新手更快地上手并入门fount，包含搭建教程，使用注意事项与报错自查，由于我本人并不了解程序和编程，所以会在教程下方补充黑话之类的扫盲，该手册在我脱坑前也会一直持续更新。**禁转类脑/方舟，转发时注明作者即可，其他随意**


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

### 1. 搭建教程
详情参照[搭建教程](https://github.com/steve02081504/fount/blob/master/docs/Readme.zh-CN.md#%E5%AE%89%E8%A3%85%E5%B0%86-fount-%E7%BC%96%E7%BB%87%E5%85%A5%E4%BD%A0%E7%9A%84%E4%B8%96%E7%95%8C--%E6%AF%AB%E4%B8%8D%E8%B4%B9%E5%8A%9B)

---
**以下为该搭建教程的~~不负责佛系扫盲~~**

***预演***：在真正执行一个从网上下载的脚本之前，先把它下载到本地的一个变量里，让用户有机会检查一下脚本内容，确认安全无误之后，再手动运行它。

***[CAUTION]***：警告的意思。在该搭建教程里是提醒你注意fount可以原生执行JavaScript脚本，即可以直接操作你的电脑

遇到任何问题，或仍然有疑问，**[欢迎加入我们的Discord群组！](https://discord.gg/sKdutkWQgt)**

---
### 2. 角色导入
详情参照[角色导入教程](https://discord.com/channels/1288934771153440768/1302581938657431622/1332720852223131729)

---
以下是我个人在此过程出现过的一些问题与解决方法

__**❓导入角色后没有显示/运行时终端爆红**__

首先刷新fount页面或重启fount程序，如果你也是win10，出现了以上情况（终端爆红）建议使用[Windows terminal](https://aka.ms/terminal)，这是因为老旧的终端（CMD/PowerShell）无法正确“翻译” fount 输出的漂亮的彩色文本和特殊符号，把它们误认为了错误，而terminal则没有这个问题，它完美兼容fount输出的字符与文本，所以后续我的所有操作包括运行fount也是使用terminal而不是电脑自带的cmd或powershell

__**❓打不开Microsoft Store下载Windows terminal**__

使用浏览器的地址栏打开这个[链接](ms-windows-store://pdp/?productId=9N0DX20HK701)
或者去terminal的github页面[下载](https://github.com/microsoft/terminal/releases)，找到最新的Release，下滑到Assets，找Microsoft.WindowsTerminal_<版本号>_8wekyb3d8bbwe.msixbundle下载（有的叫.msixbundle，点大一点的那个）

__**❓安装terminal时出现以下提示**__

去该页面[下载依赖包](https://www.nuget.org/packages/Microsoft.UI.Xaml/)，点进去之后，在右边找到 “Download package” 下载，下载下来的是一个`.nupkg`文件。他可以把这个文件的后缀名改成`.zip`，然后解压，在里面的`tools\AppX\`文件夹里找到对应他系统架构（一般是x64）的`.appx`文件，双击安装。

<img width="812" height="508" alt="image" src="https://github.com/user-attachments/assets/1064e58e-0ed8-4baf-8d04-c6594abf03cf" />




