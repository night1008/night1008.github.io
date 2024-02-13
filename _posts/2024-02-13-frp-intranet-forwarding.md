---
layout: post
keywords: frp intranet forwarding
description: 如何通过 frp 进行内网穿透
title: 如何通过 frp 进行内网穿透
comments: true
---

### 准备条件
1. 内网主机 (假设地址为: intranet-machine)
2. 公网主机 (假设地址为: aliyun-machine)

## 1. 公网主机上安装 frps

我安装的系统是 Ubuntu22。

假设该主机公网 IP 地址为 8.134.56.47

```sh
wget https://github.com/fatedier/frp/releases/download/v0.54.0/frp_0.54.0_linux_amd64.tar.gz

tar -zxvf frp_0.54.0_linux_amd64.tar.gz

cd frp_0.54.0_linux_amd64.tar.gz
```

然后修改 `frps.toml` 文件，内容如下

```
# frps.toml
bindPort = 7000
```

正常直接执行 `./frps -c ./frps.toml` 就行了，不过我们如果想自动重启的话，可以加到 systemctl service 中。

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
ExecStart=/root/frp_0.54.0_linux_amd64.tar.gz/frps -c /root/frp_0.54.0_linux_amd64.tar.gz/frps.ini

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl start frps
sudo systemctl enable frps
systemctl status frps
```

这样 frps 就算启动完成了。

## 2. 内网主机上安装 frpc

我的电脑是 mac。


```sh
wget https://github.com/fatedier/frp/releases/download/v0.54.0/frp_0.54.0_darwin_amd64.tar.gz

tar -zxvf frp_0.54.0_darwin_amd64.tar.gz

cd frp_0.54.0_darwin_amd64.tar.gz
```

然后修改 `frpc.toml` 文件，内容如下

```
# frpc.toml
serverAddr = 8.134.56.47 (公网主机IP)
serverPort = 7000

[ssh]
name = "ssh"
type = "tcp"
localIP = "127.0.0.1"
localPort = 22
remotePort = 6000
```

然后启动 `frpc`，别人才可以远程访问你的接口了。

```sh
./frpc -c ./frpc.toml
```

## 3. 内网主机生成 ssh 密钥

内网主机上生成公钥和私钥，私钥用来给本机访问
> 假设输入名称 aliyun-ecs-jumpssh-internal
```sh
ssh-keygen
```

```sh
ssh -i ~/Secrets/aliyun-ecs.cer root@aliyun-machine 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/aliyun-ecs-jumpssh-internal.pub
```

## 4. 修改公网主机 ssh 配置

正常在阿里云上可以直接下载 aliyun-ecs.cer 私钥进行登录。

```sh
vim /etc/ssh/sshd_config
```

```
GatewayPorts yes
```

```sh
sudo service sshd restart
```

---

流程如下，

```
                           ┌──────────────────────────────────┐
                           │  aliyun-machine-ssh-public-key   │
                           └──────────────────────────────────┘
                                     ┌──────────────────┐
                                     │                  │
                 ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─▶│  aliyun-machine  │─ ─ ─ ─ ─ ─ ─ ─ ─
                                     │                  │                 │
                 │                   └──────────────────┘                    ┌──────────────────────────────────┐
                                                                          │  │  aliyun-machine-ssh-private-key  │
                 │                                                        ▼  └──────────────────────────────────┘
        ┌────────────────┐                                     ┌─────────────────────┐
        │                │                                     │                     │
        │  local-machine │                       ┌────────────▶│   intranet-machine  │
        │                │                       │             │                     │
        └────────────────┘                       │             └─────────────────────┘
┌──────────────────────────────────┐             │       ┌─────────────────────────────────┐
│ intranet-machine-ssh-private-key │─────────────┘       │ intranet-machine-ssh-public-key │
└──────────────────────────────────┘                     └─────────────────────────────────┘
```

---

最终访问命令：
```sh
ssh -oPort=6000 -i aliyun-ecs-jumpssh-internal intranet-machine-username@8.134.56.47
```
