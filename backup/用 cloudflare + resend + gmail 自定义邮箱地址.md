一个邮箱如何做到
1. 一域多号，每个用途一个独立地址
2. 追踪信息泄露源头
3. 一键"拉黑"某个泄露源
4. 摆脱对单一邮箱服务商的依赖

可以使用 cloudflare 的邮箱转发，再配合 Resend 的 SMTP 配置，就可以获得一个自有域名的且独一无二的邮箱地址。

配置很简单，如下：

第一步：设置 Cloudflare Email Routing（Compute > Email Service > Email Routing）

Cloudflare Dashboard → 【你的域名】 → 电子邮件路由
添加规则：me@【你的域名】 → 转发到你的 Gmail
去 Gmail 确认转发验证邮件

第二步：设置 Gmail 转发 (设置 > 账号和导入 > 添加其他电子邮件地址)

Gmail 支持"以其他地址发送"，但需要一个 SMTP 服务来验证你对 【你的域名】 的所有权。

可以使用 [http://resend.com](Resend)（免费额度够个人用）：每天免费收100封，每月免费发3000封

创建一个 API Key，然后在 Resend 里找 SMTP 凭据（Settings → SMTP）

Gmail → 设置 → 账号和导入 → 用其他地址发送邮件 → 

添加 me@【你的域名】

SMTP 服务器填 Resend 提供的地址，用户名/密码填 Resend 的 SMTP 凭据

Gmail 会发验证码到 me@【你的域名】，因为第一步已经配好转发，验证码会转到你 Gmail 收件箱

配完之后，写邮件时在"发件人"下拉选 me@【你的域名】 就行了。

收发链路的流程图如下，
<img width="2720" height="2480" alt="Image" src="https://github.com/user-attachments/assets/06ddd490-6f2e-4135-84aa-c9817cc03618" />