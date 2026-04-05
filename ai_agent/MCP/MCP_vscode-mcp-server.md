# vscode-mcp-server 使用说明

这份文档介绍如何在 VS Code 中安装 `vscode-mcp-server`，并分别接入 `Codex` 和 `Roo Code`。

它的核心作用是：把 VS Code 的文件、编辑、符号、诊断、终端能力，通过 MCP 暴露给 AI 使用。

## 一、适用场景

适合以下情况：

- 希望 AI 优先通过 VS Code 读取和编辑工作区文件
- 希望 AI 使用 VS Code 的符号检索和诊断能力
- 希望减少单纯依赖 shell 读文件

## 二、安装扩展

在 VS Code 扩展市场搜索：

```text
vscode-mcp-server
```

Marketplace：

```text
https://marketplace.visualstudio.com/items?itemName=JuehangQin.vscode-mcp-server
```

GitHub：

```text
https://github.com/juehang/vscode-mcp-server
```

## 三、VS Code 推荐设置

建议在 VS Code `settings.json` 中加入：

如果当前项目还没有 `.vscode` 目录，建议不要用命令行手动建，直接在 VS Code 里创建工作区设置文件：

1. 用 VS Code 打开整个项目文件夹，不要只打开单个文件
2. 按 `Ctrl+Shift+P`
3. 执行 `Preferences: Open Workspace Settings (JSON)`
4. 把下面配置贴进去并保存
5. 保存后，VS Code 会自动创建 `.vscode/settings.json`

如果你想所有项目通用，而不是每个项目都建 `.vscode`，可以改为执行：

```text
Preferences: Open User Settings (JSON)
```

这样会写入 VS Code 全局用户设置。

```json
{
  "vscode-mcp-server.host": "127.0.0.1",
  "vscode-mcp-server.port": 3000,
  "vscode-mcp-server.defaultEnabled": true,
  "vscode-mcp-server.enabledTools": {
    "file": true,
    "edit": true,
    "shell": false,
    "diagnostics": true,
    "symbol": true
  }
}
```

说明：

- `defaultEnabled: true` 表示 VS Code 启动后自动启用服务
- `shell: false` 更安全，建议先关掉
- 默认地址就是：

```text
http://127.0.0.1:3000/mcp
```

## 四、在 Codex 中使用

编辑：

```text
C:\Users\Administrator\.codex\config.toml
```

加入：

```toml
[mcp_servers.vscode]
url = "http://127.0.0.1:3000/mcp"
```

然后：

1. 确保 VS Code 中 `vscode-mcp-server` 已启用
2. 重启 VS Code
3. 新开一个 Codex 会话

验证：

```powershell
codex mcp get vscode
```

如果看到：

```text
transport: streamable_http
url: http://127.0.0.1:3000/mcp
```

说明 Codex 已接入成功。

## 五、在 Roo Code 中使用

### 方式 1：全局配置

编辑：

```text
C:\Users\Administrator\AppData\Roaming\Code\User\globalStorage\rooveterinaryinc.roo-cline\settings\mcp_settings.json
```

加入：

```json
{
  "mcpServers": {
    "vscode": {
      "type": "streamable-http",
      "url": "http://127.0.0.1:3000/mcp"
    }
  }
}
```

### 方式 2：项目级配置

在项目根目录创建或编辑：

```text
.roo/mcp.json
```

内容：

```json
{
  "mcpServers": {
    "vscode": {
      "type": "streamable-http",
      "url": "http://127.0.0.1:3000/mcp"
    }
  }
}
```

说明：

- 全局配置适合所有项目通用
- 项目级配置适合只给某个仓库启用
- 如果两者同时存在，通常优先看项目级配置

使用前还要确认 Roo Code 设置中已经开启 `Enable MCP Servers`。

改完后建议重启 VS Code。

## 六、常用能力

这个 MCP 的常见能力主要有：

- 文件读取与列目录
- 文件编辑
- 符号搜索
- 诊断信息
- 终端命令执行

如果你主要想让 AI 更像“通过 VS Code 读文件”，重点打开：

- `file`
- `edit`
- `symbol`
- `diagnostics`

## 七、建议提示词

### 给 Codex 或 Roo Code 的通用提示词

```text
请优先使用 vscode-mcp-server 的 VS Code 工具处理文件读取、编辑、符号检索和诊断。
对于项目文件，不要优先使用 shell 读文件命令。
```

### 读取文件时

```text
请优先使用 vscode-mcp-server 读取当前工作区中的文件，不要先使用 `Get-Content`。
```

### 查符号时

```text
请优先使用 vscode-mcp-server 的符号工具搜索定义和文档结构。
```

## 八、如何验证是否真的用了它

对 Codex，可以执行：

```powershell
codex exec --json "请优先使用 vscode-mcp-server 读取当前工作区中的 README.md，并简要总结内容。" > .\trace.jsonl;
Get-Content .\trace.jsonl -Encoding UTF8 | Select-String 'mcp_tool_call|command_execution|vscode|read_file_code|Get-Content'
```

判断方法：

- 出现 `mcp_tool_call` 且 `server":"vscode"`，说明用了 `vscode-mcp-server`
- 出现 `command_execution` 且命令里有 `Get-Content`，说明走了 shell

## 九、常见问题

- 连不上：先确认 VS Code 里的服务已经启动，端口是不是 `3000`
- 如果报错：

```text
Transport channel closed
UnexpectedContentType
missing-content-type
```

通常不是 `config.toml` 写错，而是 `vscode-mcp-server` 服务根本没有真正启动。

这时按下面顺序检查：

1. 在 VS Code 状态栏确认 `vscode-mcp-server` 是否已启用
2. 执行命令面板里的：

```text
vscode-mcp-server: Toggle MCP Server
vscode-mcp-server: Show MCP Server Information
```

3. 如果还是不行，执行：

```text
Developer: Reload Window
```

4. 在 PowerShell 中检查端口：

```powershell
Test-NetConnection 127.0.0.1 -Port 3000
```

如果 `TcpTestSucceeded` 不是 `True`，说明服务还没跑起来
- 即使设置了 `vscode-mcp-server.defaultEnabled = true`，第一次安装或窗口状态异常时，也可能仍需要手动执行一次 `Toggle MCP Server`
- 当前项目没有 `.vscode`：直接在 VS Code 里执行 `Preferences: Open Workspace Settings (JSON)`，保存时会自动创建 `.vscode/settings.json`
- 只打开了单个文件而不是项目文件夹：这时不适合配工作区设置，建议先用 `File > Open Folder`
- 这个扩展要求 VS Code 版本至少为 `1.99.0`
- Roo Code 没反应：确认已经开启 `Enable MCP Servers`
- 没有工具：检查 `enabledTools` 是否把对应分类关掉了
- 安全性：建议只监听 `127.0.0.1`，并优先关闭 `shell`
- 多工作区：这个扩展目前更适合单工作区使用
- `trace.jsonl` 中文乱码：读取时加上 `-Encoding UTF8`

```powershell
Get-Content .\trace.jsonl -Encoding UTF8
```

## 十、结论

最简配置方式就是：

1. 在 VS Code 安装 `vscode-mcp-server`
2. 打开 `defaultEnabled`
3. 在 Codex 里配置：

```toml
[mcp_servers.vscode]
url = "http://127.0.0.1:3000/mcp"
```

4. 在 Roo Code 里配置：

```json
{
  "mcpServers": {
    "vscode": {
      "type": "streamable-http",
      "url": "http://127.0.0.1:3000/mcp"
    }
  }
}
```

这样 `Codex` 和 `Roo Code` 都可以通过同一个 VS Code MCP 服务工作。

## 参考

- GitHub: https://github.com/juehang/vscode-mcp-server
- Marketplace: https://marketplace.visualstudio.com/items?itemName=JuehangQin.vscode-mcp-server
- Roo Code Docs: https://docs.roocode.com/
