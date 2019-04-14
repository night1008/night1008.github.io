---
layout: post
keywords: shadesocks vpn command
description: 如何搭建shadesocks vpn
title: 搭建shadesocks vpn
comments: true
---


```
wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh

chmod +x shadowsocks-all.sh

./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log
```
