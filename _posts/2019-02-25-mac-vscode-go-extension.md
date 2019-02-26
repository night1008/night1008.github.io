---
layout: post
keywords: mac,vscode,go extension
description: mac下安装vscode的go插件
title: mac下安装vscode的go插件
comments: true
---

由于github被墙的原因，导致不能直接从vscode内安装go插件。

本机环境为mac，通过 `brew install go` 安装的go开发环境。

使用 `go env` 查看$GOPATH，本机为 `GOPATH="/Users/night/go"`

github下载速度较慢时可以通过修改DNS改进。

通过http://tool.chinaz.com/dns查询github.com的dns ip地址

采用海外DNS，更改hosts文件加入ip地址与域名的对应。

编辑hosts文件：`sudo vi /etc/hosts`

加入：

192.30.255.113 github.com

192.30.255.112 github.com

刷新DNS：`sudo killall -HUP mDNSResponder`


```
cd $GOPATH/src
mkdir -p golang.org/x
cd golang.org/x
git clone https://github.com/golang/tools.git
git clone https://github.com/golang/lint.git
```

手动执行 go install 安装失败包

```
cd $GOPATH
go install github.com/ramya-rao-a/go-outline
go install github.com/acroca/go-symbols
go install golang.org/x/tools/cmd/guru
go install golang.org/x/tools/cmd/gorename
go install github.com/sqs/goreturns
go install github.com/golang/lint/golint
```

不过我本机安装最后一个`github.com/golang/lint/golint`，提示
```
can't load package: package github.com/golang/lint/golint: cannot find package "github.com/golang/lint/golint" in any of:
    /usr/local/Cellar/go/1.11.2/libexec/src/github.com/golang/lint/golint (from $GOROOT)
    /Users/night/go/src/github.com/golang/lint/golint (from $GOPATH)
```

在`$GOPATH/src/github.com`目录下执行，`cp -R ../golang.org/x/lint .`即可。

重启vscode即可看到go插件安装成功。
