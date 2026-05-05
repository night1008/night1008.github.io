最好通过自建 AI 中转站，不然每个渠道的消息格式可能不一样。

### [Cloudfare Workers AI](https://developers.cloudflare.com/workers-ai/) 
> 需要再经过自建中转站
> 每天有一万的免费用量 (Daily usage (resets at 00:00 UTC) - Neurons used today: 0/10k)
> 需要经过中转站处理消息格式问题

### [Google AI Studio](https://aistudio.google.com/api-keys)
> 需要再经过自建中转站
> [Gemini API 速率限制](https://aistudio.google.com/rate-limit)

### [Groq](https://console.groq.com)
> 需要再经过自建中转站
> [Rate Limits](https://console.groq.com/docs/rate-limits)
> 虽然能通，但几乎用不了，触发 TPM 或 max_completion_tokens 限制错误

### [BAI](https://b.ai)
> 需要再经过自建中转站
> 注册即送 50w token
> 支持的 [模型列表](https://docs.b.ai/zh-Hans/llmservice/quick-start/) 也都不错

### [OpenCode](https://opencode.ai/)
> 不需要再经过自建中转站
> 每天有几个免费模型可以用
> 直接安装 `brew install anomalyco/tap/opencode` 即可使用

### [Anyrouter](https://anyrouter.top)
> 不需要再经过自建中转站
> 可以使用最新的 claude 模型，不过工作日服务经常不可用
