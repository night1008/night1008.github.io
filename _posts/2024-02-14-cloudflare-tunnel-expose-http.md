---
layout: post
keywords: Cloudflare Tunnels, http, web
description: 如何使用 Cloudflare Tunnels 提供内网 http 服务
title: 如何使用 Cloudflare Tunnels 提供内网 http 服务
comments: true
---

最近在推特上看到可以使用 Cloudflare Tunnels 提供内网服务，于是尝试了一下，在此作下记录。

#### 准备条件
1. 主机上安装了 Cloudflare Zero Trust
2. Cloudflare 上购买了域名，比如 example.cloudflare.com

我的电脑为 mac 系统，其他系统请自行选择安装方式。
### 安装 cloudflared
```
brew install cloudflared

sudo cloudflared service install xxx
```

### 设置 Public hostnames
```
tunnel.example.cloudflare.com

映射到

http://localhost:8080
```

### 启动内网服务

比如使用 go gin 启动一个 http 服务，端口为 8080。
```go
package main

import (
	"net/http"

	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()

	r.GET("/ping", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"message": "pong",
		})
	})

	r.Run() // listen and serve on 0.0.0.0:8080 (for windows "localhost:8080")
}
```

### 结果
```
curl http://tunnel.example.cloudflare.com/ping
```

可以用手机直接访问以上地址，即使手机上没有安装 Cloudflare Zero Trust。

更详细的可以参考官方文档，[# Set up a tunnel through the dashboard](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-remote-tunnel/)