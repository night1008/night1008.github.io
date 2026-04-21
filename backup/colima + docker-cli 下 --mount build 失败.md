
### 系统环境
```sh
Client: Docker Engine - Community
 Version:           29.3.0
 API version:       1.53 (downgraded from 1.54)
 Go version:        go1.26.1
 Git commit:        5927d80c76
 Built:             Thu Mar  5 14:22:32 2026
 OS/Arch:           darwin/amd64
 Context:           colima

Server: Docker Engine - Community
 Engine:
  Version:          29.2.1
  API version:      1.53 (minimum version 1.44)
  Go version:       go1.25.6
  Git commit:       6bc6209
  Built:            Mon Feb  2 17:17:26 2026
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          v2.2.1
  GitCommit:        dea7da592f5d1d2b7755e3a161be07f43fad8f75
 runc:
  Version:          1.3.4
  GitCommit:        v1.3.4-0-gd6d73eb8
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

### 错误情况
```sh
build image image:latest
DEPRECATED: The legacy builder is deprecated and will be removed in a future release.
            Install the buildx component to build images with BuildKit:
            https://docs.docker.com/go/buildx/

Sending build context to Docker daemon  538.1kB
Step 1/8 : FROM golang:1.25-trixie AS build
 ---> 091e6099f4a5
Step 2/8 : COPY . /ai-agent-manager/
 ---> Using cache
 ---> abff4fdc98bb
Step 3/8 : RUN --mount=type=cache,target=/root/.cache/go-build     --mount=type=cache,target=/go     cd /ai-agent-manager &&     go build . &&     go build ./internal/cmd/shim
the --mount option requires BuildKit. Refer to https://docs.docker.com/go/buildkit/ to learn how to build images with BuildKit enabled
```

```sh
docker buildx version
docker: unknown command: docker buildx

Run 'docker --help' for more information
```

### 解决方法

```
brew install docker-buildx
```
