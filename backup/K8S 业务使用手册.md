### 概念

Pod 是由一个或多个为了管理和联网而绑定在一起的容器构成的组。

Deployment 用于管理运行一个应用负载的一组 Pod，通常适用于不保持状态的负载。
> 一个 Deployment 为 Pod 和 ReplicaSet 提供声明式的更新能力。

Service 是公开 Pod 的虚拟网络外部访问。
> --type=LoadBalancer 参数表明 Service 暴露到集群外部。

[Helm](https://helm.sh/) 是一种管理预配置 Kubernetes 资源包的工具。

将多个资源归入同一个文件（在 YAML 中使用 --- 分隔）可以简化对多个资源的管理。
资源会按照在清单中出现的顺序创建。 因此，最好先指定 Service，这样可以确保调度器能在控制器（如 Deployment）创建 Pod 时对 Service 相关的 Pod 作分布。

---