---
layout: post
keywords: minikube, docker
description: minikube 替换 docker 桌面应用
title: minikube 替换 docker 桌面应用
comments: true
---

发现两个问题，

1. Docker-for-mac 占用很大的电脑空间
2. minikube 使用 docker-desktop 无法正常访问 service

因此选择卸载 Docker-for-mac，使用 minikube 作为替换。

替换步骤如下，

```sh
brew install hyperkit

brew install docker

brew install docker-compose

brew install minikube

brew install kubectl
```

启动 minikube 脚本 `minikube_start.sh`

```sh
#!/bin/bash
set -xeu

minikube start \
    --driver hyperkit \
    --cpus 2 --memory 2g \
    --addons dashboard \
    --addons metrics-server \
    --addons ingress \
    --addons ingress-dns \
    --mount --mount-string="$HOME:$HOME" \
    --kubernetes-version v1.24.1

sudo mkdir -p /etc/resolver

cat <<EOF | sudo tee /etc/resolver/minikube-test
domain test
nameserver $(minikube ip)
search_order 1
timeout 5
EOF

sudo sed -i.bak "s/.*minikube.test/$(minikube ip) minikube.test/g" /etc/hosts
```

启动后再执行以下命令，新建窗口时就不用手动执行 `minikube docker-env` 了。

```sh
eval $(minikube ip minikube docker-env) >> ~/.zshrc
```

按照完成后可以使用 [hello-minikube](https://kubernetes.io/docs/tutorials/hello-minikube/) 教程进行检验。