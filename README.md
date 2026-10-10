# DocRoute MCP

**Route your AI coding agent to the RIGHT official docs — full text, not snippets.**

Give your AI a documentation librarian: it asks DocRoute what the official docs say, gets back a curated menu of the most relevant pages plus their complete original text, and writes code against *actual* documentation instead of stale training-data memories.

> 中文说明见下方 [中文](#中文) 部分。

- **88 tech stacks**: React, Vue, Next.js, FastAPI, Django, Fastify, K8s, Docker, Terraform, Redis, PostgreSQL-ecosystem ORM, Stripe, OpenAI / Anthropic / Gemini APIs, LangChain, Playwright… grouped into 9 categories.
- **Remote MCP — zero install**: point your MCP client at one URL. Works with ZCode, Claude Desktop (via `mcp-remote`), Cursor, Cherry Studio, and any MCP-compatible agent.
- **Judge-based matching**: instead of keyword/embedding search, DocRoute broadcasts your question to a judgment model that reads each doc unit's profile and scores actual relevance — so "how do I deploy Fastify to production" returns the deployment guide, not the plugins page.
- **Full-text delivery**: `get_doc` returns the *entire* page (with sha256 fingerprint to catch drift), ready for the model to read.

## Quick start

### ZCode

Add to `~/.zcode/cli/config.json` under `mcp.servers`, then restart:

```json
"docroute": {
  "type": "http",
  "url": "https://docroute.songm.dpdns.org/mcp",
  "timeoutMs": 600000
}
```

### Cursor

Settings → MCP → Add Server:

```json
{
  "docroute": {
    "url": "https://docroute.songm.dpdns.org/mcp"
  }
}
```

### Claude Desktop / Claude Code (remote MCP via bridge)

Requires Node.js. Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "docroute": {
      "command": "npx",
      "args": ["mcp-remote", "https://docroute.songm.dpdns.org/mcp"]
    }
  }
}
```

### Any other MCP client

Use **Streamable HTTP** transport with endpoint `https://docroute.songm.dpdns.org/mcp`, no auth required during the open test. Set timeouts ≥ 300s (large stacks judge their whole corpus per query, which takes 1–5 min).

## Tools

| Tool | Input | What you get |
|---|---|---|
| `list_stacks` | — | All 88 stacks, grouped by category (frontend / backend / ops / data / AI / tooling / testing / languages / payments) |
| `search_docs` | `stack`, `query` | A ranked menu of matching docs with score + one-line gist. Cheap on tokens — read the top 1–2, fetch more only if needed |
| `get_doc` | `stack`, `path` | The **complete original text** of one doc, with drift detection |

Result quality signal: `hit` = direct match; `weak` / `no_evidence` = thin coverage, try rephrasing; `judge_error` = upstream judge hiccup, retry.

## How to ask (works much better)

1. **Check docs before writing code** — model memory drifts.
2. **One atomic topic per query.** "How do I define routes" ✔ — "routes, middleware and deployment" ✘ (split it).
3. Use **technical keywords + intent**: `"options reference"`, `"how to deploy"`.

## Example

> **Q:** `search_docs(stack="fastify", query="部署到云服务器生产环境")`
>
> **A:** menu → `deployment` (0.78), `logging` (0.62) … then `get_doc` fetches the full deployment page.

## Status & notes

- 🧪 **Open test**: free, no auth. Rate limits apply per client IP (search 60/h, get_doc 600/h, list 120/h). Be kind.
- The corpus (4.4 GB across 88 stacks) and routing engine are **not** in this repo — this repo is the access manual. The service is the product.
- Endpoint is a permanent domain behind a Cloudflare named tunnel. Test-period availability is best-effort.

---

<a id="中文"></a>
# 中文说明

**给你的 AI 编码助手接一个"官方文档管理员"：先查原文，再写代码。**

- **88 个技术栈**：前端 / 后端 / 运维 / 数据 / AI / 工具 / 测试 / 语言 / 支付 九大类。
- **远程 MCP，零安装**：MCP 客户端里填一个 URL 即可（ZCode / Claude Desktop / Cursor / Cherry Studio 均可）。
- **判定式匹配**：不是关键词也不是向量检索——把你的问题广播给判定模型逐单元打分，命中真正的相关文档。
- **交付完整原文**：`get_doc` 直接返回整页文档（带 sha256 指纹防过期）。

### 接入（三选一）

**ZCode**（`~/.zcode/cli/config.json` 的 `mcp.servers` 下）：

```json
"docroute": { "type": "http", "url": "https://docroute.songm.dpdns.org/mcp", "timeoutMs": 600000 }
```

**Cursor**（Settings → MCP → Add Server）：

```json
{ "docroute": { "url": "https://docroute.songm.dpdns.org/mcp" } }
```

**Claude Desktop**（需 Node.js，`claude_desktop_config.json`）：

```json
{ "mcpServers": { "docroute": { "command": "npx", "args": ["mcp-remote", "https://docroute.songm.dpdns.org/mcp"] } } }
```

### 三个工具

| 工具 | 作用 |
|---|---|
| `list_stacks` | 列出全部 88 个技术栈（按类分组） |
| `search_docs` | 提问 → 判定 → 返回相关文档菜单（带匹配分和一句话简介） |
| `get_doc` | 取某份文档的完整原文（带指纹校验） |

### 用法规矩（照做效果最好）

1. 先查再写代码——模型记忆里的 API 可能过时
2. 一次问一个原子主题："怎么定义路由" ✓；"路由、中间件和部署怎么做" ✗
3. 措辞带技术关键词 + 意图："options reference"、"how to deploy"
4. search 返回的是菜单（省 token），先取前 1~2 份原文读，不够再续取

### 测试期说明

- 免费开放，无需鉴权；按 IP 限速（search 60 次/时、get_doc 600 次/时、list 120 次/时），请勿滥用
- 语料与路由引擎**不在本仓库**——本仓库只是接入说明书，服务本身才是产品
- 大栈全库判定需 1~5 分钟，客户端超时建议 ≥300 秒
- 域名为 Cloudflare 命名隧道永久域名，测试期可用性尽力保障

## License

MIT (this repository only; docs are served from the service, not distributed here).
