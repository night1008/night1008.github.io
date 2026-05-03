前两天通过 [new-api](https://github.com/QuantumNous/new-api) 自建了中转站，同时看到 [cloudfare workers ai](https://developers.cloudflare.com/workers-ai/) 每天有一万的免费用量，
目前本地主要使用 [pi](https://github.com/badlogic/pi-mono) 进行 ai coding，于是尝试如何进行接入。

### 在 Cloudflare上创建 [workers](https://developers.cloudflare.com/workers/)
> 代码如下

```js
export default {
  async fetch(request, env, ctx) {
    if (request.method === 'OPTIONS') {
      return new Response(null, { status: 204 });
    }

    const url = new URL(request.url);
    const targetUrl = 'https://api.cloudflare.com' + url.pathname + url.search;

    let body = await request.json();

    console.log(body)

    // 把 developer role 转换成 system
    if (body.messages) {
      body.messages = body.messages.map(msg => {
        // 转换 role
        const role = msg.role === 'developer' ? 'system' : msg.role;
        
        // 转换 content 数组为字符串
        let content = msg.content;
        if (Array.isArray(content)) {
          content = content
            .filter(c => c.type === 'text')
            .map(c => c.text)
            .join('\n');
        }

        return { role, content };
      });
    }

    const response = await fetch(targetUrl, {
      method: request.method,
      headers: request.headers,
      body: JSON.stringify(body),
    });

    return response;
  }
};
```

为什么需要创建 workers 呢？直接使用过程中发现，
1. pi 发的系统提示词是：`{"role": "developer", "content": "You are a helpful assistant."}`
CF AI  期望的是：`{"role": "system", "content": "You are a helpful assistant."}`
CF AI  不认识 developer role，返回了错误。

2. pi 发的是：`{"role": "user", "content": [{"type": "text", "text": "hi"}]}`
但 CF AI 期望的是：`{"role": "user", "content": "你好"}`
messages content 是数组格式，CF AI 不支持，只支持字符串格式。

因此需要 cloudfare workers 进行请求参数转换。
可以直接通过请求进行测试
```
curl https://aiproxy.example.com/v1/chat/completions \
  -H "Authorization: Bearer sk-xxx" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "@cf/meta/llama-3.1-8b-instruct",
    "messages": [
      {"role": "system", "content": "You are a helpful assistant."},
      {"role": "user", "content": "你好"}
    ]
  }'
```

### 在 new-api 上配置 Cloudflare 渠道
> 注意 API地址 要填 Cloudflare Worker 地址，如下图所示，
<img width="585" height="538" alt="Image" src="https://github.com/user-attachments/assets/beb85af8-9379-49fc-8c28-723445ba9d02" />

### 在 pi 中加入 providers 配置
>  编辑 .pi/agent/models.json
>  路径上记得带 v1
```json
{
    "providers": {
        "myproxy": {
            "baseUrl": "https://aiproxy.example.com/v1",
            "apiKey": "sk-xxx",
            "api": "openai-completions",
            "models": [
                {
                    "id": "@hf/nousresearch/hermes-2-pro-mistral-7b",
                    "name": "@hf/nousresearch/hermes-2-pro-mistral-7b"
                }
            ]
        }
    }
}
```

注意不要使用 `@cf/meta/llama-3.1-8b-instruct` 之类的模型，这个只支持纯文本，
命令行工具的调用可以使用 `@hf/nousresearch/hermes-2-pro-mistral-7b` 进行测试。


