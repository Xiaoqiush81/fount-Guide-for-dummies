# 基础配置

本章节将介绍如何导入角色以及最核心的 **AI 源配置**。

---

## 1. 导入角色

进入 fount 后，你会看到角色选择页面。刚开始是没有角色的，莫慌，点击右上角的 **菜单按钮 (☰)** 就会出现功能界面！

<img width="1919" height="1022" alt="image" src="https://github.com/user-attachments/assets/40111044-27f9-4800-a038-26dc61b37f65" />

首先点击“组件”——“导入”

<img width="1242" height="805" alt="image" src="https://github.com/user-attachments/assets/1597ec4c-abbb-4ce7-9ac6-aedde133e5ec" />

在导入中我们有两种导入方式：

1. **文件导入**：上传已下载的 fount 角色（zip 档案）或酒馆角色（png，json 档案）。
2. **链接导入**：在右边的文本框粘贴角色网址（github 网址、chub 网址或 risurlm 网址），每行一个，随后点击批量导入！

<img width="1919" height="1024" alt="image" src="https://github.com/user-attachments/assets/df584df6-68c1-4a73-bd13-03cb17e3e2d2" />

---

## 2. 配置 AI 源

这是最重要的一步。假设你已经有了一个需要 AI 源的角色。

### 进入配置界面

1. 在主页右上角菜单，选择 **“管理服务源”**。
 
 <img width="293" height="253" alt="image" src="https://github.com/user-attachments/assets/1688a1b4-f848-45e7-a909-9f1c6e14bbdb" />

2. 点击 **“+”** 新建一个 AI 源。
3. **生成器** 选择 `gemini` (或其他)。
4. **名称**：给它起个名字（例如 `gemini`），**这个名称无特定标准，自行填写即可，记得不要包含后缀**。

   <img width="832" height="209" alt="image" src="https://github.com/user-attachments/assets/9dc1a6b9-701d-4463-9f2b-1277384c08fc" />

### A. 配置 Gemini (官方 API)

如果你不清楚 Gemini 的 API Key 的获取流程，可以参阅 [哈基米 API Key 纯宝宝教程](https://github.com/Xiaoqiush81/Gemini-API-Key-guide)。

在左侧的 JSON 文件中，将你的 API Key 填写在第 3 行的 `apikey`，在第 4 行的 `model` 填写正确的模型名（如 `gemini-1.5-pro`）。

<img width="1251" height="735" alt="image" src="https://github.com/user-attachments/assets/df529196-22f2-4e0b-8746-807b281ac2a8" />

### B. 配置 OpenAI / DeepSeek

如果你的 AI 不是来自 cohere 或 gemini，fount 也支持任何 Open AI 格式的自定义来源。
在 "选择生成器" 里更换成 `proxy`：

<img width="232" height="359" alt="image" src="https://github.com/user-attachments/assets/ea7716fa-cd08-4d99-86d1-46e288c80e38" />

然后在左侧 json 文件按照如图填写。如果你也想配置 DeepSeek，url 和 model 可以直接复制我的，然后把 `apikey` 替换成你自己的 key。
此处附上 [DeepSeek API 获取链接](https://platform.deepseek.com/sign_in)。

<img width="949" height="550" alt="image" src="https://github.com/user-attachments/assets/8960d234-0f43-4198-8124-d344b7c1bc45" />

### C. 配置 Polling (轮询/负载均衡)

fount 内置了 polling 轮询功能。

**【此处为~~不负责佛系~~扫盲】**

- **选择生成器**：这个是“团队的工作模式”。就像是排队模式。您给团队一个任务，团队长（也就是 fount）会按照您定好的名单，从第一个 AI 开始问：“你能干这个活吗？”。如果它不行或者在忙，就立刻去问名单上的第二个，以此类推，直到找到一个能完成任务的 AI 为止。

- **选择生成器**：`polling`
- **JSON 配置**：

<img width="1246" height="740" alt="image" src="https://github.com/user-attachments/assets/43dbc31e-7c87-4340-b0c3-b82dc0cc5eb1" />

你可以直接把你配置好的其他 AI 源的名字填入 `sources` 列表，也可以像下图那样，把一个 AI 的完整配置信息直接写在这里。

<img width="1282" height="751" alt="image" src="https://github.com/user-attachments/assets/c8d39bae-fdf9-426d-8799-101105350db3" />

---

## 3. 为角色分配 AI

配置好 AI 源后，需要把它接入 char 使其工作起来！

和酒馆不同，在 fount 中 AI 设置不是全局的，而是 **每一个角色可以配置一个或多个 AI 源**。

**【为什么要这么做？】**

或许 **A 模型** 可能擅长解答问题，**B 模型** 没有 NSFW 屏蔽。
那么我们可能会想让角色：

- 在一般情况下使用 **A 模型**。
- 在 **R18 / NSFW 情节** 中使用 **B 模型**。
- 或者 **C 模型** 免费但是有点蠢，适合做一些简单的判断。

### 配置步骤

1. 首先点击角色，然后点击界面上的配置按钮（那个齿轮！）。  
   <img width="357" height="384" alt="image" src="https://github.com/user-attachments/assets/8a095eba-2d05-4f70-bc05-7d656e040d5f" />

2. 进入后左边选择 **chars**，右方直接选择你要配置的角色名字。  
   <img width="1892" height="649" alt="image" src="https://github.com/user-attachments/assets/1a698b6f-27db-4ca9-a87f-edd3e2a5145e" />

3. 下滑找到 **配置文件** 中的 `AIsources` 或 `AIsource`。  
   填入你配置好的 AI 源名称（例如 `gemini`）。  
   <img width="1711" height="825" alt="image" src="https://github.com/user-attachments/assets/60471c75-0382-4c97-a1e0-2c863346be6c" />

> **💡 懒人福利**：  
> 可以在 AI 源列表给你想设置的源打上勾，设为默认源。这样新角色会自动调用这个源，不需要挨个配置。  
> <img width="424" height="609" alt="image" src="https://github.com/user-attachments/assets/21ceca98-8366-487a-90dd-7f1e12e3e837" />

---

[返回主页](../README.md)
