##1.2.2、Kubernetes简述

Kubernetes已成为容器集群管理标准，通过YAML文件来管理配置应用程序容器和其他资源。Kubernetes执行诸如调度，扩展，服务发现，健康检查，密文管理和配置管理等功能。

一个Kubernetes集群由多个节点组成:

- **etcd database**
  通常在一个节点上运行一个etcd实例服务，但生产环境上，建议通过3个或5个(奇数)以上的节点来创建ETCD HA配置。
- **Master nodes**
  主节点是无状态的，用于运行API Server，调度服务和控制器服务。
- **Worker nodes**
  工作负载在工作节点上运行。

默认情况下Master节点也会有工作负载调度上去， 可通过命令设置其不加入调度