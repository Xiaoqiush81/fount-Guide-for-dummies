# 拓展功能

这些功能是选修课，让你的 fount 角色走出 fount，在其他平台活跃。

## 1. Discord Bot

让角色成为 Discord 里的聊天机器人。

### 第一步：创建 Discord Application

1. 前往 [Discord Developer Portal](https://discord.com/developers/applications)。
2. 点击 **New Application**，输入 bot 的名字，点击 Create。
   <img width="767" height="428" alt="image" src="https://github.com/user-attachments/assets/dd2d51e9-3a00-4198-8769-9a0b21a100b7" />
   _(注意：这个名字会作为你的 bot 的名字和用户名)_
3. 你可以在这里上传头像，设置描述等。

### 第二步：配置 Bot 权限 (非常重要！) 🔴

1. 在左侧菜单点击 **Bot**。
2. 点击 **Reset Token** 获取你的 Bot Token，保存好它。
   <img width="1920" height="667" alt="image" src="https://github.com/user-attachments/assets/87b9c820-2de6-4750-8577-8ae860099674" />
3. **【关键步骤】** 向下滚动，找到 **Privileged Gateway Intents**。
4. **把下面的三个开关全部打开！** (Presence, Server Members, Message Content)。
   <img width="1906" height="875" alt="image" src="https://github.com/user-attachments/assets/911e1943-84ec-436c-8884-c947a6ea442e" />
   > **⚠️ 注意**：如果不打开这三个开关，Bot 无法读取消息，Fount 会报错或无响应。

### 第三步：配置 fount

1. 回到 fount，右上角菜单 -> **Bot** -> **管理 Discord bot**。
2. 新建一个 Bot，填入刚才获取的 Token。
3. 在下方 JSON 中配置 `OwnerUserName` (你的 Discord 用户名) 等信息。
   <img width="669" height="247" alt="image" src="https://github.com/user-attachments/assets/ff8ed41a-d8ee-4504-b9b0-dabafad18991" />

### 第四步：邀请 Bot 进入服务器

1. 在 Discord 开发者页面左侧选 **OAuth2** -> **URL Generator**。
2. 勾选 `bot`。
   <img width="1625" height="365" alt="image" src="https://github.com/user-attachments/assets/b7be9ec2-4ec0-4dff-9fda-761540740694" />
3. 在下方权限 (Bot Permissions) 中，勾选需要的权限（勾选 `Administrator` 最省事）。
   <img width="1901" height="930" alt="image" src="https://github.com/user-attachments/assets/775ba6fb-28d9-4288-aa2b-ee7531ec6a4a" />
4. 复制生成的 URL，在浏览器打开，邀请 Bot 进入你的服务器。
   <img width="1590" height="283" alt="image" src="https://github.com/user-attachments/assets/41971530-e3e0-4db2-a109-fb5b83153839" />

邀请成功后，你应该能在服务器侧边栏看到你的 Bot：
<img width="565" height="850" alt="image" src="https://github.com/user-attachments/assets/5c6002b7-132f-41aa-8f37-58fa16859e07" />

---

## 2. Telegram Bot

1. **准备 Bot Token**：
   - 在 Telegram 私聊[机爸](https://t.me/BotFather)。
   - 发送 `/newbot`，按提示设置名字和 ID，获取 Token (打码位置)。
     <img width="690" height="695" alt="image" src="https://github.com/user-attachments/assets/dad55e2c-2bd2-491a-814b-9b52d67a41f4" />
   - 搜索 `@userinfobot` 获取你自己的 User ID。
2. **配置 fount**：
   - 菜单 -> **Bot** -> **管理 Telegram bot**。
   - 新建 Bot。
     <img width="612" height="230" alt="image" src="https://github.com/user-attachments/assets/69bd4c14-44b1-4fa1-bb6a-940f3f10ab3b" />
   - 填入 Token 和你的 User ID (作为管理员)。
     <img width="1350" height="807" alt="image" src="https://github.com/user-attachments/assets/576b220f-e5ca-4ce2-9789-12ad8d39fd18" />

---

## 3. fount-pwsh (命令行助手)

在 PowerShell 终端里召唤角色作为助手，或者纯粹在终端聊天。

1. **安装模块**：
   在 PowerShell 中运行：

   ```powershell
   Install-Module fount-pwsh
   ```

   <img width="570" height="40" alt="image" src="https://github.com/user-attachments/assets/04205186-eb11-4957-9b44-37a45cd5f8dc" />

2. **配置助手**：

   ```powershell
   Install-FountAssist <你的fount用户名> <角色名>
   ```

   <img width="817" height="48" alt="image" src="https://github.com/user-attachments/assets/06942e51-ffb2-48c2-91e1-c9beab98ee1c" />

3. **使用**：
   - `f 1`: 召唤助手。
   - `f 0`: 解散助手。

**助手预览：**
当命令执行失败时，助手会自动介入：
<img width="1328" height="303" alt="image" src="https://github.com/user-attachments/assets/6f9ea890-2c1e-42f2-9064-985edf020dd4" />

在终端里闲聊的效果：
<img width="1426" height="285" alt="image" src="https://github.com/user-attachments/assets/03ee6d4f-91bd-4e1a-9f2e-1e4025b7239b" />
<img width="1451" height="349" alt="image" src="https://github.com/user-attachments/assets/98fef666-e085-4b50-a7b1-47df15aef09e" />

---

## 4. 浏览器集成

### 效果展示

- **发送弹幕**：
  <img width="842" height="144" alt="image" src="https://github.com/user-attachments/assets/18d5c5d6-6941-454e-a64a-362605aef894" />

- **播放视频**：
  <img width="992" height="854" alt="image" src="https://github.com/user-attachments/assets/426a7037-7f7b-4815-8ed8-c3d436bc21b8" />

### 第一步：安装脚本管理器

你需要安装 **Tampermonkey (油猴)** 扩展。

### 第二步：配置浏览器权限 (关键！)

**这一步不做，插件无法读取本地 fount 接口！**

1. 打开浏览器的 **扩展程序管理** 页面 (`chrome://extensions/`)。
2. 找到 Tampermonkey，点击 **“详情”**。  
   <img width="412" height="330" alt="image" src="https://github.com/user-attachments/assets/402ea8cd-5bc6-4114-91e0-f8272d80f7a6" />
3. 向下滚动，找到 **“允许访问文件地址” (Allow access to file URLs)**，务必**开启**它。  
   <img width="854" height="69" alt="image" src="https://github.com/user-attachments/assets/bb2443c2-8c6b-4208-8853-6206f891ae57" />

### 第三步：安装与连接

1. 进入 fount 界面 -> 菜单 -> **集成** -> **浏览器集成**。
2. 根据提示安装 **CSP 禁用脚本** 和 **fount 用户脚本**。
3. 打开任意网页，如果 Tampermonkey 图标变为彩色，说明连接成功。  
   <img width="911" height="593" alt="image" src="https://github.com/user-attachments/assets/3758960e-1cfb-4925-83d8-84f3b38c5728" />

---

[返回主页](../README.md)
