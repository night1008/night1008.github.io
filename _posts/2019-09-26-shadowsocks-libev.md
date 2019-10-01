---
layout: post
keywords: install shadesocks-libev on ubuntu
description: 如何在ubuntu上安装shadesocks-libev
title: 如何在ubuntu上安装shadesocks-libev
comments: true
---

```
sudo apt update

sudo apt install shadowsocks-libev

sudo vim /etc/shadowsocks-libev/config.json

sudo systemctl start shadowsocks-libev

systemctl status shadowsocks-libev

systemctl restart shadowsocks-libev
```

config.json具体内容

```json
{
    "server":"0.0.0.0",
    "server_port":3307,
    "local_port":1080,
    "password":"123456",
    "timeout":60,
    "method":"aes-128-gcm"
}
```