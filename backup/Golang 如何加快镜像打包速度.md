原来镜像打包过程中可以指定挂载缓存和编译结果路径

```sh
FROM golang:1.23.9-alpine3.21 AS builder

WORKDIR /app

COPY . /app/

ARG BUILD_VERSION

ENV BUILD_VERSION=${BUILD_VERSION:-develop} \
    GOPROXY=https://goproxy.cn,direct

RUN --mount=type=cache,target=/root/.cache/go-build \
    --mount=type=cache,target=/go \
    go mod download \
    && go build -o demo-web --ldflags="-X 'main.buildVersion=$BUILD_VERSION'" .

FROM alpine:3.20

# 解决 Go 时区加载 unknown time zone XXX 的问题
RUN apk update && apk add tzdata

COPY --from=builder /app/demo-web /usr/bin/demo-web
```

参考链接
- [Optimize cache usage in builds](https://docs.docker.com/build/cache/optimize/)
- [BuildKit](https://yeasy.gitbook.io/docker_practice/buildx/buildkit)