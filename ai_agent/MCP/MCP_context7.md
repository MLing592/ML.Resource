# Context7 MCP 使用说明

`Context7` 用来给 `Codex` 补最新库文档。适合问：

- 框架 / SDK 最新用法
- 安装和配置步骤
- 某个版本的 API 示例

## 一、怎么调用

最简单的方式是在提示词里加：

```text
use context7
```

例如：

```text
请根据 Next.js 15 最新文档解释 middleware 的 matcher 写法。use context7
```

```text
请给我一个 Supabase 邮箱密码注册的最小示例。use context7
```

如果你已经知道库 ID，直接这样写更准：

```text
请使用 library /supabase/supabase，给我邮箱密码注册示例。
```

Context7 常见会用到这两个工具：

- `resolve-library-id`
- `get-library-docs`

## 二、给 Codex 配置 Context7

编辑：

```text
C:\Users\Administrator\.codex\config.toml
```

### 方式 1：远程方式，不带 Key

```toml
[mcp_servers.context7]
url = "https://mcp.context7.com/mcp"
```

适合先直接用起来。

### 方式 2：远程方式，带 Key

```toml
[mcp_servers.context7]
url = "https://mcp.context7.com/mcp"
http_headers = { "CONTEXT7_API_KEY" = "YOUR_API_KEY" }
```

需要更高限流或私有仓库相关能力时再加。

### 方式 3：本地方式，不带 Key

要求本机可用 `Node.js` 和 `npx`：

```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]
startup_timeout_ms = 20_000
```

### 方式 4：本地方式，带 Key

```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp", "--api-key", "YOUR_API_KEY"]
startup_timeout_ms = 20_000
```

如果本地启动超时，可以把两种本地方式都改成：

```toml
startup_timeout_ms = 40_000
```

## 三、API Key

获取地址：

```text
https://context7.com/dashboard
```

按 Context7 文档，API Key 是可选的，不配也能用。

配置后有这些好处：

- 更高的限流
- 可用私有仓库相关能力

## 四、检查是否接入成功

```powershell
codex mcp list
```

确认列表里有：

```text
context7
```

再看详情：

```powershell
codex mcp get context7
```

远程模式通常会看到：

```text
transport: streamable_http
url: https://mcp.context7.com/mcp
```

## 五、验证是否真的调用了 Context7

```powershell
codex exec --json "请根据 Next.js 15 最新文档解释 middleware 的 matcher 写法。use context7" > .\trace.jsonl;
Get-Content .\trace.jsonl -Encoding UTF8 | Select-String 'mcp_tool_call|context7|resolve-library-id|get-library-docs|query-docs'
```

如果输出里出现：

- `mcp_tool_call`
- `context7`
- `resolve-library-id`
- `get-library-docs`

通常就说明这次请求走了 `Context7 MCP`。

## 六、建议提示词

```text
请根据最新官方文档回答, use context7
```

```text
请先用 context7 找到对应库，再给我最新示例代码。
```

```text
请使用 library /vercel/next.js，读取 middleware 相关文档并给示例。
```

## 七、常见问题

- `codex mcp list` 里没有 `context7`：检查节名是不是 `[mcp_servers.context7]`
- 本地模式报 `npx` 找不到：先确认已安装 `Node.js`
- 本地模式超时：把 `startup_timeout_ms` 调大到 `40_000`
- 远程模式连不上：先检查网络和 API Key
- 回答还是偏旧：在提示词里明确加 `use context7`
- 已知具体库时不够准：直接写 `use library /xxx/yyy`

## 参考

- Context7 MCP README: https://www.npmjs.com/package/@upstash/context7-mcp
- Context7 Docs: https://context7.com/docs/resources/all-clients
- OpenAI Codex MCP Docs: https://developers.openai.com/codex/mcp/
