## ⚠️ 安全警告

fount 的原生角色拥有**极高的系统权限**。它们运行在你的本地机器上，权限等同于你本人的用户权限。

这意味着一个恶意的角色卡可以：

- ❌ **删除你硬盘上的文件**
- ❌ **读取你的隐私照片/文档并上传**
- ❌ **安装病毒或挖矿软件**

**🛡️ 安全原则：**

1. **绝对不要**运行来源不明、未经验证的 fount 角色卡。
2. 只从你信任的作者或官方渠道下载角色。

---

## 🧐 什么是 fount？

简单来说，**fount 是一个超级自由的、开源的 AI 角色扮演平台。**

它最大的特点就是【**完全的自由**】！不像其他平台有各种各样的限制，在 fount 的世界里，你想创造什么样的角色、进行什么样的对话，都只取决于你自己的想象力。

- **绝对自由**：支持原生 JS 动画/无过滤 HTML 渲染。
- **高度兼容**：兼容 Risu 的 charX、SillyTavern 的大部分角色卡。
- **完全本地**：保护隐私，所有对话均在本地进行。
- **开源免费**：社区驱动，持续进化，完全免费。

---

## 🚀 如何开始？(极速上手指南)

### 1. 搭建教程

#### 🟢 给超级电脑小白 / 懒鬼

如果你什么都不会，可以通过 fount runner 运行它。
你只需要 [下载 exe 文件](https://github.com/steve02081504/fount/releases/download/runner-v0.0.0.1/fount.exe)，随后点击运行即可。

#### 🟠 给一般人

首先，和酒馆不同，fount 依赖于 **Deno** 而不是 Node.js。
fount 的启动脚本会在启动后未找到 deno 时自行安装 deno。

1. **准备环境**：如果需要，你可以参照 [deno 的官方文档](https://docs.deno.com/runtime/getting_started/installation/) 进行自定义安装。
2. **下载 fount**：
   - **会 Git 的人**：

     ```bash
     git clone https://github.com/steve02081504/fount/
     ```

   - **不会 Git 的人**：
     [点击这里下载最新版的 fount](https://github.com/steve02081504/fount/archive/refs/heads/master.zip)。下载后解压。

3. **启动**：
   在终端中运行启动脚本。fount 会自动安装 Git（如果需要）并在未来自动检查更新。

---

### 2. 首次运行与验证

进入后你会看到类似这样的页面，点击输入用户名和密码来创建一个账户。
**注意：所有内容都在本地，删了 fount 文件夹就啥都没了，记得备份！**

点击发送验证码后，你的 **fount 终端（就是那个黑框框）** 处会显示验证码内容，输入进去即可。
**要是没跳验证码那就不管它，直接进。**

<img width="923" height="728" alt="image" src="https://github.com/user-attachments/assets/3073d736-3ee5-418e-b191-55f652cfaa32" />

---

## 🧘 看不懂这些鸟语？

遇到 **Terminal**、**Git**、**Clone** 这些词头大？
别怕，我们准备了 **[👉 不负责佛系扫盲](./glossary.md)**，专门拯救电脑小白。

---

[返回主页](../README.md) | [下一页：基础配置 →](./configuration.md)
