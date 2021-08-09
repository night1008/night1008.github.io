---
layout: post
keywords: caddy, domain, https, website
description: 使用caddy快速构建包含https站点
title: 使用caddy快速构建包含https站点
comments: true
---

### 准备条件

1. 购买域名
2. 购买包含公网ip的机器
3. 设置 A type DNS 记录
4. 修改 cloudflare SSL/TLS encryption mode

我首先在 `godaddy` 上买个域名，然后转移到 `cloudflare` 上；
然后在 `AWS Lightsail` 上购买了一台机子；
然后把这台机子的`静态IP`绑定到 DNS 记录上；
然后用`caddy`起了个网站，查看是否可以正常访问域名。

使用`caddy`的原因是它配置简单，自动包含`https`设置，
具体代码可见 [mysite](https://github.com/night1008/mysite)

访问过程中发现，一直报`ERR_TOO_MANY_REDIRECTS`错误，
原来是因为没有设置`cloudflare SSL/TLS encryption mode`为`Full或Full (strict)`

具体可见 [Fix Too Many Redirect error using Caddy + Cloudflare](https://gist.github.com/lopezjurip/5314252970cc94970058320ac78f490a)

至此就可以正常访问域名了。