### 概念

Pod 是由一个或多个为了管理和联网而绑定在一起的容器构成的组。

Deployment 用于管理运行一个应用负载的一组 Pod，通常适用于不保持状态的负载。

Service 是公开 Pod 的虚拟网络外部访问。
> --type=LoadBalancer 参数表明 Service 暴露到集群外部。

---