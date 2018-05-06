---
layout: post
keywords: python, shadowsocks, 代理
description: install python shadowsocks on server, 安装Python Shadowsocks
title: 安装Python Shadowsocks
comments: true
---

{{ page.title }}
<p class="meta">06 May 2018</p>
<hr>


承接上一篇文章，为了使用`维基百科`的接口，因为国内封锁了相关域名，导致无法使用，

又不想使用`百度百科`的相关链接，感觉不够简要，所以就使用自己搭建的VPN进行代理

Python环境：Python 3.6.1

```
pip install shadowsocks
```

### 创建配置文件/etc/shadowsocks.json

```
$ cat <<EOF > /etc/shadowsocks.json
{
    "server":"your_server_ip",      #ss服务器IP
    "server_port":your_server_port, #服务器端口
    "local_address": "127.0.0.1",   #本地IP
    "local_port":1080,              #本地端口
    "password":"your_server_passwd",#连接ss密码
    "timeout":300,                  #等待超时
    "method":"aes-256-cfb",         #加密方式
    "fast_open": false,             #true或false。如果你的服务器Linux内核在3.7+，可以开启fast_open以降低延迟。开启方法：echo 3 > /proc/sys/net/ipv4/tcp_fastopen 开启之后，将fast_open的配置设置为 true 即可
    "workers": 1                    # 工作线程数
}
EOF
```

### 启动ShadowSocks客户端

```
nohup sslocal -c /etc/shadowsocks.json /dev/null 2>&1 &
```

### 将Shadowsocks客户端加入开机自启动

```
echo "nohup sslocal -c /etc/shadowsocks.json /dev/null 2>&1 &" >> /etc/rc.local
```

### 测试代理

```
curl --socks5 127.0.0.1:1080 http://httpbin.org/ip

{
  "origin": "xx.xx.xx.xx" #如果这个IP是你Shadowsocks服务器的IP就OK了。
}
```
