---
layout: post
keywords: docker alpine go unknown timezone
description: docker alpine 运行 go 程序报 unknown timezone 错误
title: docker alpine 运行 go 程序报 unknown timezone 错误
comments: true
---

### 现象

本地环境运行 go 程序好好的，通过 docker 部署之后报以下错误。

```
unknown time zone Asia/Shanghai
```

### 原因

原来是 docker alpine 中没有 Go 所需的 timezone 配置导致的。

### 解决方法

所以解决办法就是在镜像中安装这个 `tzdata` 包，在 Dockerfile 中加入以下这一段即可

```
FROM golang:1.14.3-alpine

FROM alpine

RUN apk update && apk add tzdata
```

