在使用 OpenAI Codex CLI 配置第三方中转服务（如 AnyRouter、OneAPI 等）时，常遇到如下报错：

```text
ERROR: unexpected status 401 Unauthorized: 未提供令牌 (request id: ...), url: https://<provider-domain>/v1/responses
```

明明在 `~/.codex/auth.json` 中配置了 API Key，为什么请求依然报未提供令牌？

---

## 核心原因

1. **服务商拦截**：中转网关在收到请求时，要求必须携带 `Authorization: Bearer <API_KEY>` 请求头，缺失时直接返回 401「未提供令牌」。
2. **Codex 凭证隔离安全机制**：
   Codex CLI 对自定义 Provider（非官方 `openai`）默认开启凭证隔离（`requires_openai_auth = false`）。**Codex 出于安全考量，绝不会自动将 `auth.json` 中的 `OPENAI_API_KEY` 发给未经显式授权的第三方域名**。
   若配置中未指明凭证传递方式，Codex 会发起一个不包含认证请求头的裸请求，导致 401 拦截。

---

## 解决方案

编辑 `~/.codex/config.toml`，在对应 Provider 配置段中添加鉴权声明（三选一）：

```toml
model = "gpt-5-codex"
model_provider = "anyrouter"
preferred_auth_method = "apikey"
```

### 推荐做法 1：显式配置 `experimental_bearer_token`

直接为该 Provider 绑定专属 Token（最稳定）：

```toml
[model_providers.anyrouter]
name = "Any Router"
base_url = "https://anyrouter.top/v1"
wire_api = "responses"
# 添加专属 Bearer Token
experimental_bearer_token = "sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

---

### 推荐做法 2：声明复用 `auth.json` (`requires_openai_auth = true`)

继续复用 `~/.codex/auth.json` 中统一维护的密钥：

```toml
[model_providers.anyrouter]
name = "Any Router"
base_url = "https://anyrouter.top/v1"
wire_api = "responses"
# 授权 Codex 将 auth.json 中的凭证发送至该 Provider
requires_openai_auth = true
```

---

### 推荐做法 3：通过环境变量传递 (`env_key`)

避免在配置文件中写入明文密钥：

```toml
[model_providers.anyrouter]
name = "Any Router"
base_url = "https://anyrouter.top/v1"
wire_api = "responses"
# 指定环境变量名称
env_key = "ANYROUTER_API_KEY"
```

在 Shell 中注入该变量：
```bash
export ANYROUTER_API_KEY="sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

---

## 验证与排查技巧

配置完成后，运行 Codex 内置体检工具验证：

```bash
codex doctor
```

- 检查 `auth` 与 `reachability mode` 项：若显示 `API key auth` 且路由探测成功，说明令牌已正确附加。
- 若 401 消失但提示 `We’re currently experiencing high demand`（响应码 500），说明凭证已生效，当前属于中转端上游对应模型渠道繁忙或配额耗尽。
