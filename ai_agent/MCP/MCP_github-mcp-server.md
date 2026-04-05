# GitHub MCP 使用说明

## 目标

- 在 `VS Code` 中使用 `Codex` 扩展
- 让 `Codex` 通过 `GitHub MCP` 搜索 GitHub 上是否有符合需求的库

## 二、安装前准备

建议先确认以下条件：

- 已安装 `VS Code`
- 已安装并能正常使用 `Codex` 扩展
- 本机可以执行 `codex` 命令
- 准备一个可用的 GitHub Token

如果你的主要需求是“让 Codex 自动去 GitHub 搜索合适的库”，直接给 `Codex` 配 GitHub MCP 就够了。

## 三、给 Codex 配置 GitHub MCP

### 方式 1：使用命令直接添加

先准备一个环境变量名，下面示例使用：

```text
GITHUB_MCP_PAT
```

然后执行：

```powershell
codex mcp add github --url https://api.githubcopilot.com/mcp/ --bearer-token-env-var GITHUB_MCP_PAT
```

这条命令会把一个名为 `github` 的 MCP Server 加到 Codex 配置里，并让 Codex 从环境变量 `GITHUB_MCP_PAT` 读取 Bearer Token。

添加后可以检查：

```powershell
codex mcp list
```

```powershell
codex mcp get github
```

### 方式 2：直接写入 `config.toml`

如果你不想用命令，也可以直接在下面这个文件里手动写：

```text
C:\Users\Administrator\.codex\config.toml
```

加入：

```toml
[mcp_servers.github]
url = "https://api.githubcopilot.com/mcp/"
bearer_token_env_var = "GITHUB_MCP_PAT"
```

这和 `codex mcp add github --url ... --bearer-token-env-var ...` 是等价的。

如果你已经习惯直接维护 `config.toml`，这种方式也完全可以。

## 四、GitHub Token 如何获取

GitHub 官方目前支持两类 Personal Access Token：

- `Fine-grained personal access token`
- `Personal access token (classic)`

GitHub 官方文档当前更推荐优先使用 `fine-grained personal access token`。

### 推荐获取方式

1. 打开 GitHub 右上角头像
2. 进入 `Settings`
3. 打开 `Developer settings`
4. 进入 `Personal access tokens`
5. 选择 `Fine-grained tokens`
6. 点击 `Generate new token`
7. 填写名称、过期时间、资源所有者和仓库访问范围
8. 生成后复制这个 Token

如果你只是想让 Codex 搜索公开仓库，通常不需要给很大的权限范围。
GitHub 官方文档说明，fine-grained token 默认就包含对所有公开仓库的只读访问。

如果你还要让 Codex 访问你的私有仓库，再按需要增加对应仓库权限。

## 七、如何把 Token 设置到环境变量

### 方式 1：只在当前 PowerShell 会话里临时生效

```powershell
$env:GITHUB_MCP_PAT = "你的 GitHub Token"
```

这种方式只对当前终端会话有效，关掉终端后就失效。

### 方式 2：写入当前用户环境变量

```powershell
setx GITHUB_MCP_PAT "你的 GitHub Token"
```

执行后需要注意两点：

- 它不会自动刷新已经打开的 VS Code 进程
- 你需要关闭并重新打开 VS Code，让 Codex 扩展重新读取环境变量

### 方式 3：用 Windows 图形界面设置

1. 打开“编辑系统环境变量”
2. 进入“环境变量”
3. 在“用户变量”里新建：

```text
变量名：GITHUB_MCP_PAT
变量值：你的 GitHub Token
```

4. 保存后，重新打开 VS Code

如果你希望配置更稳，不依赖当前终端状态，推荐使用 `方式 2` 或 `方式 3`。

## 八、在 VS Code 的 Codex 扩展中使用

配置完成后，建议按下面顺序操作：

1. 关闭并重新打开 VS Code
2. 打开 Codex 扩展或新建一个 Codex 会话
3. 先确认当前可用的 MCP

例如可以直接对 Codex 说：

```text
请先检查当前是否已经连接了名为 github 的 MCP server。
```

确认可用后，再继续发搜索需求。

### 适合直接发给 Codex 的提示词

```text
请使用 GitHub MCP 搜索适合做 .NET 事件总线的开源库，优先看 C#、star 较高、最近仍活跃维护的项目。
```

```text
请使用 GitHub MCP 搜索 GitHub 上支持 MCP 协议的 .NET 库，先给候选仓库，再说明筛选原因。
```

```text
请先用 search_repositories 做粗筛，再用 search_code 验证候选项目是否真的支持我需要的功能。
```

## 九、GitHub MCP 对“找库”最有用的能力

如果你的目标是“自动搜索有没有满足我需求的库”，最常用的是这些工具能力：

- `search_repositories`
- `search_code`
- `search_issues`
- `get_file_contents`

通常可以这样理解：

- `search_repositories`：先找候选仓库
- `search_code`：再验证仓库里是否真的有目标能力
- `search_issues`：辅助判断社区活跃度和常见问题
- `get_file_contents`：读取 `README`、示例代码、配置文件

## 十、如何验证 Codex 是否真的用了 GitHub MCP

最直接的检查方式有两个。

第一种，看 Codex 当前配置：

```powershell
codex mcp list
```

如果列表里已经有：

```text
github
```

说明配置至少已经存在。

还可以进一步查看：

```powershell
codex mcp get github
```

你应当能看到类似：

```text
transport: streamable_http
url: https://api.githubcopilot.com/mcp/
```

第二种，直接让 Codex 明确使用它：

```text
请使用 GitHub MCP 搜索 GitHub 上满足我需求的库，不要先用普通网页搜索。
```

如果 Codex 能进一步读取某个仓库的 `README`、代码路径、issue，而不是只给泛搜索结果，通常就说明它确实在使用 GitHub MCP。

## 十一、常见问题

- `codex mcp list` 里没有 `github`：说明 GitHub MCP 还没加进去，重新执行 `codex mcp add ...` 或检查 `config.toml`
- `codex mcp get github` 能看到配置，但会话里用不上：通常需要重开 VS Code 或新开一个 Codex 会话
- Token 无效：检查环境变量名是否和 `bearer_token_env_var` 一致
- 刚执行了 `setx` 还是不生效：关闭并重新打开 VS Code
- 手工写了 `config.toml` 但不生效：检查节名是不是 `[mcp_servers.github]`
- 搜索公开仓库却仍提示权限问题：优先确认 Token 是否复制完整，环境变量里是否有多余空格
- 结果不理想：在提示词里明确要求先搜仓库，再验代码，再评估活跃度

## 十二、结论

对于 `VS Code + Codex`，最简使用路径就是：

1. 在 GitHub 里生成一个 Token
2. 在 Windows 中把它设置到 `GITHUB_MCP_PAT`
3. 用下面任一方式配置 Codex：

```powershell
codex mcp add github --url https://api.githubcopilot.com/mcp/ --bearer-token-env-var GITHUB_MCP_PAT
```

或：

```toml
[mcp_servers.github]
url = "https://api.githubcopilot.com/mcp/"
bearer_token_env_var = "GITHUB_MCP_PAT"
```

4. 重开 VS Code
5. 直接让 Codex 使用 GitHub MCP 按需求搜索仓库

## 参考

- GitHub 官方 MCP Server: https://github.com/github/github-mcp-server
- GitHub Docs - Managing your personal access tokens: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens
