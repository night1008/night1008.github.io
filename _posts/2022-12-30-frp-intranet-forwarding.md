---
layout: post
keywords: frp intranet forwarding
description: 如何通过 frp 进行内网穿透
title: 如何通过 frp 进行内网穿透
comments: true
---

最近几天居家办公。我这边实现的功能需要暴露 http 接口给前端联调，
如何直接把代码部署到公网机器的话会比较麻烦，所以打算直接用 frp 进行我本地的内网穿透。
那么怎么做呢。

## 准备条件
1. 内网主机 (假设地址为: intranet-machine)
2. 公网主机 (假设地址为: aliyun-machine)

> 公网主机可以使用阿里云的 ECS 抢占式实例，比较便宜，然后记得开放安全组的端口。

### 1. 公网主机上安装 frps

我安装的系统是 Ubuntu22。

假设该主机公网 IP 地址为 8.134.56.47

```sh
wget https://github.com/fatedier/frp/releases/download/v0.46.0/frp_0.46.0_linux_amd64.tar.gz

tar -zxvf frp_0.46.0_linux_amd64.tar.gz

cd frp_0.46.0_linux_amd64
```

然后修改 `frps.ini` 文件，内容如下

```
[common]
bind_port = 7000
vhost_http_port = 8080
```

正常直接执行 `./frps -c ./frps.ini` 就行了，不过我们如果想自动重启的话，可以加到 systemctl service 中。

```sh
cd /etc/systemd/system

touch frps.service
```

内容如下

```
[Unit]
Description=frps service
After=network.target syslog.target
Wants=network.target

[Service]
Type=simple
ExecStart=/root/frp_0.46.0_linux_amd64/frps -c /root/frp_0.46.0_linux_amd64/frps.ini

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl start frps
sudo systemctl enable frps
systemctl status frps
```

这样 frps 就算启动完成了。

### 2. 自己的开发机上安装 frpc

我的电脑是 mac。


```sh
wget https://github.com/fatedier/frp/releases/download/v0.46.0/frp_0.46.0_darwin_amd64.tar.gz

tar -zxvf frp_0.46.0_darwin_amd64.tar.gz

cd frp_0.22.0_darwin_amd64
```

然后修改 `frpc.ini` 文件，内容如下

```
[common]
server_addr = 8.134.56.47 (公网机器IP)
server_port = 7000

[web]
type = http
local_port = 8080 (需要暴露的本地端口)
custom_domains = 8.134.56.47 (域名的话需要备案)
```

然后启动 `frpc`，别人就可以远程访问你的接口了。

```sh
./frpc -c ./frpc.ini
```

---

最终效果：其他地方的主机可以通过 `8.134.56.47:8080` 访问我本地的开发环境接口了。