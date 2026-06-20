## 用自己的域名做邮箱，告别 Gmail 锁死

一个域名 + Cloudflare + Resend，免费实现：

- **一域多号**：`shop@你的域名`、`bank@你的域名`，每个用途一个地址
- **追踪泄露源**：哪个地址开始收垃圾邮件，就知道是哪个网站卖了你
- **一键拉黑**：删掉那条转发规则，垃圾邮件源头直接断流
- **不被服务商绑架**：以后想换 Gmail 换别的邮箱，对外身份不用变

### 原理

```
收信：发件人 → Cloudflare Email Routing → 转发到 Gmail
发信：Gmail 界面 → Resend SMTP → 以 me@你的域名 发出
```

域名是你的根，Gmail 只是临时管道，随时可换。

### 配置步骤

**第一步：Cloudflare 收信**

1. Cloudflare Dashboard → 你的域名 → Email Routing
2. 添加规则：`me@你的域名` → 转发到 Gmail
3. 去 Gmail 点确认验证邮件

**第二步：Resend 发信**

1. 注册 [resend.com](https://resend.com/)（免费：每天收 100 封，每月发 3000 封）
2. 创建 API Key
3. 在 Settings → SMTP 拿到服务器地址、账号、密码

**第三步：设置 Gmail**

1. Gmail → 设置 → 账号和导入 → 用其他地址发送邮件 → 添加 `me@你的域名`
2. 填入 Resend 的 SMTP 信息
3. 验证码会发到 `me@你的域名`，因为第一步配好了转发，会自动转进 Gmail 收件箱
4. 验证通过后，写邮件时"发件人"选 `me@你的域名` 即可

### 用法建议

- 不同网站用不同的本地部分（`shop-jd@`、`sub-weekly@`），方便事后追溯
- 开启 catch-all，注册新网站时现想现填，不用提前在后台建规则
- 银行、政府类账号用固定、不外传的独立地址，别用"随手发放"的那批