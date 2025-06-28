业务上需要对外提供一些 sdk 包方便相关人员使用，
会把一些常量定义到 `api` 包中，供 `client` 引用，如下所示，

```
go-sdk-test
  - api
     api.go
     go.mod
  - client
     client.go
     go.mod
```

在 `client` 使用时，直接下载的包版本是 `v0.0.0-20250628093558-4e219645ad46` 这种格式的，
```sh
go get github.com/night1008/go-sdk-test/api
```
> go: downloading github.com/night1008/go-sdk-test/api v0.0.0-20250628093558-4e219645ad46
> go: added github.com/night1008/go-sdk-test/api v0.0.0-20250628093558-4e219645ad46


即使给仓库打上 `v0.1.0` 仍然无法指定到具体版本，如下所示
```sh
go get github.com/night1008/go-sdk-test/api@v0.1.0
```
> go: module github.com/night1008/go-sdk-test@v0.1.0 found, but does not contain package github.com/night1008/go-sdk-test/api

原来打标签的时候还要多指定下子包路径才可以正常下载
```sh
git tag api/v0.1.0
```