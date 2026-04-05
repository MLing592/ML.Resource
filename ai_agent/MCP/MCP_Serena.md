# Serena 在 VS Code Codex 插件中的安装与使用教程

本文记录我在 Windows 环境下，为 VS Code 的 Codex 插件安装并使用 Serena MCP 的完整过程。

目标：

- 安装 `uv`
- 在 Codex 插件中接入 Serena
- 主动提示 Codex 优先使用 Serena 读取文件
- 证明某次文件读取来自 Serena，而不是 PowerShell 的 `Get-Content`

## 一、适用环境

- 操作系统：Windows
- IDE：VS Code
- 插件：Codex 插件

## 二、安装 uv

在 PowerShell 中执行：

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

安装成功后，终端会提示：

```text
installing to C:\Users\Administrator\.local\bin
  uv.exe
  uvx.exe
```

如果当前终端还没刷新 `PATH`，继续执行：

```powershell
$env:Path = "C:\Users\Administrator\.local\bin;$env:Path"
```

然后验证：

```powershell
uv --version
uvx --version
```

如果能看到版本号，说明 `uv` 安装完成。

## 三、在 Codex 中配置 Serena

编辑文件：

```text
C:\Users\Administrator\.codex\config.toml
```

在文件末尾加入：

```toml
[mcp_servers.serena]
startup_timeout_sec = 25
command = "uvx"
args = ["-p", "3.13", "--from", "git+https://github.com/oraios/serena", "serena", "start-mcp-server", "--project-from-cwd", "--context", "codex"]
```

说明：

- `command = "uvx"` 表示通过 `uvx` 启动 Serena
- `--project-from-cwd` 表示按当前工作目录自动识别项目
- `--context codex` 表示这是给 Codex 用的上下文，不是 VS Code 原生 Agent 的 `ide`

配置完成后，完全重启一次 VS Code。

## 四、确认 Codex 已识别 Serena

在 PowerShell 中执行：

```powershell
codex mcp list
```

如果看到类似输出：

```text
Name    Command  Args  Status   Auth
serena  uvx      ...   enabled  Unsupported
```

说明：

- `enabled` 代表 Codex 已经识别并启用了 Serena
- `Unsupported` 在这里不是报错，表示这个本地 `stdio` MCP 不需要走 OAuth 登录

还可以查看单个服务器详情：

```powershell
codex mcp get serena
```

## 五、在 Codex 插件中主动使用 Serena

在 VS Code 的 Codex 对话里，可以直接用中文提示词明确要求它优先走 Serena。

推荐这样写：

```text
请优先使用 Serena MCP 处理仓库导航、文件发现和文件读取。
对于项目文件，不要使用 `Get-Content` 之类的 shell 读文件命令。
如果 Serena 当前不可用，请先告诉我。
```

如果要读取某个具体文件，可以这样说：

```text
请使用 Serena MCP 读取工作区根目录 `abc.md` 的前 5 行。
不要使用 `Get-Content` 等 shell 读文件命令。
只返回这 5 行内容。
```

如果要强调工具选择，也可以这样说：

```text
请使用 Serena 的 `find_file` 和 `search_for_pattern` 检查 `abc.md`。
不要使用 PowerShell 的 `Get-Content`。
```

注意：

- 你不能保证模型一定调用一个名字正好叫 `readfile` 的工具。
- 重点不是工具名，而是这次读取是不是通过 `server = serena` 的 MCP 工具完成的。

## 六、如何验证这次读取来自 Serena

分两行执行：

```powershell
codex exec --json "请使用 Serena MCP 读取工作区根目录 abc.md 的前 5 行，不要使用 shell 读文件命令。" > .\trace.jsonl
Get-Content .\trace.jsonl | Select-String 'mcp_tool_call|command_execution|serena|Get-Content|abc\.md'
```

分一行执行:

```powershell
codex exec --json "请使用 Serena MCP 读取工作区根目录 abc.md 的前 5 行，不要使用 shell 读文件命令。" > .\trace.jsonl; Get-Content .\trace.jsonl | Select-String 'mcp_tool_call|command_execution|serena|Get-Content|abc\.md'
```

判断方法：

- 如果看到 `mcp_tool_call` 且 `server":"serena"`，说明这次读取用了 Serena。
- 如果看到 `command_execution` 且命令里有 `Get-Content`，说明这次读取走的是 PowerShell。

## 七、在仓库里固定“优先 Serena”

可以在仓库根目录放一个 `AGENTS.md`，把“优先 Serena”的规则写进去。重新开新会话后，Codex 一般会自动读取它。

本仓库已经加入了这类规则。如果你想让首轮行为更稳，第一句话还可以补一句：

```text
请优先使用 Serena 处理仓库导航、文件读取、符号检索和编辑；如果 Serena 做不到，再先说明原因后回退。
```

## 八、常见问题

- `uvx` 找不到：先重启 VS Code；如果还是不行，用 `Get-Command uvx | Select-Object -ExpandProperty Source` 找到绝对路径，再写回 `config.toml`。
- 偶尔仍看到 shell 命令：这是正常现象。Serena 是“优先使用”，不是“彻底禁用 shell”；想提高命中率，就配合 `AGENTS.md` 和首句提示词一起用。
- 中文乱码：通常是终端编码问题，可以用 `Get-Content -Encoding UTF8 .\serena.md` 检查，也可以在 VS Code 右下角确认文件编码。

## 九、让 Serena 少重复启动

默认 `stdio` 模式会随一次性任务结束而退出，所以看起来像“问一次开一次”。如果你想常驻运行，可以改成 HTTP 模式。

先把 `C:\Users\Administrator\.codex\config.toml` 改成：

```toml
[mcp_servers.serena]
url = "http://127.0.0.1:9121/mcp"
```

然后单独开一个 PowerShell 窗口启动 Serena。常见有 3 种方式：

### 方案 1：固定绑定单个项目

适合长时间只做一个项目。

```powershell
Set-Location E:\MyGit\ML.MemoryBus
uvx -p 3.13 --from git+https://github.com/oraios/serena serena start-mcp-server --transport streamable-http --port 9121 --project "E:\MyGit\ML.MemoryBus" --context codex
```

特点：

- 服务固定绑定到指定项目
- 不需要每次再激活项目
- 不适合所有项目共用

### 方案 2：按启动时当前目录识别项目

适合每次都先 `cd` 到目标项目，再启动 Serena。

```powershell
Set-Location E:\MyGit\ML.MemoryBus
uvx -p 3.13 --from git+https://github.com/oraios/serena serena start-mcp-server --transport streamable-http --port 9121 --project-from-cwd --context codex
```

特点：

- 启动时会按当前目录识别项目
- 比固定项目灵活
- 但服务启动后还是只认这一次启动时的目录，不会随着后面切换终端目录自动变化

### 方案 3：全项目通用，不预先绑定项目

适合经常切换不同项目，希望一个常驻 Serena 服务通用复用。

```powershell
uvx -p 3.13 --from git+https://github.com/oraios/serena serena start-mcp-server --transport streamable-http --port 9121 --context codex
```

特点：

- 启动时不绑定任何项目
- 更适合多个项目轮流使用
- 每次进入新项目时，最好先让 Codex 用 Serena 激活当前项目

建议在新项目会话开头补一句：

```text
请先使用 Serena 将当前工作区根目录激活为项目，然后优先使用 Serena 处理仓库导航、文件读取、符号检索和编辑。
```

### 该选哪一种

- 如果长期只做一个项目，选“方案 1”
- 如果每次启动前都会先进入目标目录，选“方案 2”
- 如果想所有项目共用一个常驻 Serena 服务，选“方案 3”

无论哪种方式，启动后都要保持这个 PowerShell 窗口不关闭，然后再重启 VS Code。

## 十、建议的固定提示词

平时使用：

```text
请优先使用 Serena MCP 处理仓库导航、文件读取、符号检索和编辑。
对于项目文件，不要使用 shell 读文件命令。
如果 Serena 做不到，请先说明原因，再决定是否回退。
```

取证使用：

```text
请使用 Serena MCP 读取工作区根目录 `abc.md` 的前 5 行。
不要使用 `Get-Content` 或其他 shell 读文件命令。
只返回这 5 行内容。
```

## 十一、结论

- 日常使用时，优先靠 `config.toml` + `AGENTS.md` + 中文提示词 三者配合。
- 想证明某次读取是否真的用了 Serena，就看 `codex exec --json` 事件流里有没有 `mcp_tool_call` 和 `server":"serena"`。
- 想减少“问一次开一次”，就改成 HTTP 常驻模式。
