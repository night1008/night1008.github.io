## 原理

pi 本身不内置 MCP 客户端，`mcp.json` 只是静态配置。`pi-mcp-adapter` 负责读取配置、启动 MCP server（JSON-RPC over stdio）、把工具注册给 pi：

```
mcp.json ──> pi-mcp-adapter ──> @sentry/mcp-server ──> Sentry
```

## 安装

```bash
pi install npm:pi-mcp-adapter
```

安装后**重启 pi**。

## 配置

编辑 `~/.pi/agent/mcp.json`：

```json
{
  "mcpServers": {
    "sentry": {
      "command": "npx",
      "args": ["@sentry/mcp-server"],
      "env": {
        "SENTRY_ACCESS_TOKEN": "<TOKEN>",
        "SENTRY_HOST": "sentry.example.com"
      }
    }
  }
}
```

要点：

- `SENTRY_ACCESS_TOKEN`：Sentry Auth Token（`sntryu_` 开头）
- `SENTRY_HOST`：自建实例必须显式指定，否则默认连 sentry.io
- **不要加 `--agent`**：该模式只暴露 `use_sentry`，且依赖 LLM API key，未配置时不可用

## 使用

adapter 注册一个紧凑的 `mcp` 代理工具，两步调用：

```js
// 1. 发现工具
mcp({ search: "sentry issue" })

// 2. 调用（工具名形如 sentry__<原名>）
mcp({ tool: "sentry__get_sentry_resource", args: {
  resourceType: "issue",
  resourceId: "<ISSUE_ID>",
  organizationSlug: "<ORG_SLUG>"
} })
```

其他命令：`/mcp` 查看状态与工具，`/mcp status` 快速状态，`/mcp setup` 首次引导。

## 工具清单

| 工具 | 用途 |
|---|---|
| `search_issues` | 搜索 issue 列表 |
| `get_sentry_resource` | 按 URL 或类型+ID 拉取 issue/event |
| `search_events` | 搜索事件与统计 |
| `analyze_issue_with_seer` | Seer 根因分析 |
| `update_issue` | 更新状态 / 指派 |
| `find_organizations` / `find_projects` | 查组织 / 项目 |

## 验证

```bash
# 绕过 pi，直接验证 server
export SENTRY_ACCESS_TOKEN="<TOKEN>"
export SENTRY_HOST="sentry.example.com"
npx @sentry/mcp-server
```

| 现象 | 排查 |
|---|---|
| `/mcp` 看不到 sentry | 检查 mcp.json；重启 pi；`pi list` 确认已装 |
| 认证失败 | 重新生成 token |
| 连到 sentry.io | 漏配 `SENTRY_HOST` |
| 首次调用慢 | lazy 连接 + npx 首次下载，正常；可设 `lifecycle: "keep-alive"` |

## 备选

- **sentry-cli**：适合批量改状态（resolve/mute），但 `issues` 子命令查不了单 issue 堆栈
- **curl**：适合临时脚本

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "$HOST/api/0/organizations/<ORG>/issues/<ID>/events/latest/"
```
