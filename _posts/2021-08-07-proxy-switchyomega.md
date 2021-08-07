---
layout: post
keywords: Proxy SwitchyOmega
description: Proxy SwitchyOmega设置
title: Proxy SwitchyOmega设置
comments: true
---

目前的代理是直接使用`ShadowsocksX-NG`应用，发现通过浏览器访问时，
我们自定义的PAC规则不起作用，因此需要手动切换`PAC自动模式`和`全局模式`，很是麻烦。
因此使用`Proxy SwitchyOmega`的 chrome 扩展程序。

下面记录下如何配置。

#### 设置代理服务器

![proxy.jpg](/assets/img/2021-08-07/proxy.jpg)

#### 设置自动切换规则

![auto-switch.jpg](/assets/img/2021-08-07/auto-switch.jpg)

规则如何配置可以查看 [autoproxy](https://github.com/aglent/autoproxy)

被墙PAC地址为：https://raw.githubusercontent.com/aglent/autoproxy/master/gfwlist.pac

#### 设置`Shadowsocks`为`手动模式`