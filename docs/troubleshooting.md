# 报错自查

遇到问题先别慌，这里汇总了常见问题和解决方案。

## 🔧 基础排查三板斧

1. **刷新页面 / 重启 fount**：80% 的问题（如卡顿、显示异常）可以通过重启解决。
2. **检查网络**：AI 没反应？通常是梯子/网络问题。
3. **使用调试面板**：很多小白不知道右上角有调试按钮。点击右上角这个图标：  
   <img width="77" height="61" alt="image" src="https://github.com/user-attachments/assets/c9b2e51f-82cb-4567-bb09-d5871a101a63" />  
   然后点击 **“打开调试面板”**。在这里你可以查看当前版本、网络状态以及自检。  
   <img width="268" height="89" alt="image" src="https://github.com/user-attachments/assets/24da81ac-473b-474f-a1d1-e7f82264243e" />

---

## 💻 终端显示与启动问题

### 终端里全是乱码或红字？

**原因**：Windows 自带的 `cmd` 或老版 `PowerShell` 对彩色字符支持不好。
**解决**：推荐使用 [Windows Terminal](https://github.com/microsoft/terminal/releases) (微软官方终端)。

<img width="2483" height="1254" alt="image" src="https://github.com/user-attachments/assets/ad62e748-189b-4335-b0f1-276ce7edb106" />

- **下载**：在 GitHub Releases 页面下载 `Microsoft.WindowsTerminal_..._msixbundle` 安装。
- **缺少依赖**：如果安装时提示缺少 `Microsoft.UI.Xaml.2.8`，请去 [NuGet](https://www.nuget.org/packages/Microsoft.UI.Xaml/) 下载对应包安装。  
   <img width="812" height="508" alt="image" src="https://github.com/user-attachments/assets/1064e58e-0ed8-4baf-8d04-c6594abf03cf" />

### 启动后报错退出？

- 检查端口是否被占用。
- 确保 `data` 目录有读写权限。

---

## 🌐 网络与 AI 调用问题

### 报错 400 / 403 / Network Error

**原因**：通常是网络连通性问题，或者 API Key 无效。
**解决**：

- **检查 Key**：确认你的 API Key 没有欠费或过期。
- **检查梯子**：
  - **开启全局模式** 或 **TUN 模式** (虚拟网卡)。
  - 更换节点 (Gemini 建议用美国、日本、新加坡节点)。
  - **Clash 用户注意**：老版 Clash for Windows 可能有兼容性问题，推荐使用 [Clash Verge Rev](https://github.com/clash-verge-rev/clash-verge-rev)。

<img width="1468" height="464" alt="image" src="https://github.com/user-attachments/assets/ce7623f9-9a4f-4259-bfb8-7ae3f3e1e1fd" />  
<img width="949" height="184" alt="image" src="https://github.com/user-attachments/assets/b97e581c-51e8-4021-87d9-63b8e7c89008" />

### Bot 一直卡在 "Typing..."

**原因**：AI 请求发送出去了，但没收到回音。  
**解决**：同上，大概率是网络超时。尝试刷新页面重试。

---

## 🐛 角色与显示问题

### 角色界面显示错位/报错

**原因**：可能是网络加载资源失败，或角色卡文件损坏。  
**解决**：

1. 强制刷新网页 (Ctrl + F5)。
2. 如果是特定角色报错，尝试删除该角色重新导入。

---

[返回主页](../README.md)
