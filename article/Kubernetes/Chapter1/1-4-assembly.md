## Master 组件

```
kube-scheduler             # 调度 pod
kuber-controller-manager   # 管理 pod
kube-apiserver             # 接收请求
etcd                       # 集群状态存储，集群所有的组件的状态都保存在这里
```

### kube-apiserver

[kube-apiserver](https://kubernetes.io/docs/admin/kube-apiserver)用于暴露Kubernetes API。任何的资源请求/调用操作都是通过kube-apiserver提供的接口进行

### ETCD

[etcd](https://kubernetes.io/docs/admin/etcd)是Kubernetes提供默认的存储系统，保存所有集群数据，使用时需要为etcd数据提供备份计划。

### kube-controller-manager

[kube-controller-manager](https://kubernetes.io/docs/admin/kube-controller-manager)运行管理控制器，它们是集群中处理常规任务的后台线程。逻辑上，每个控制器是一个单独的进程，但为了降低复杂性，它们都被编译成单个二进制文件，并在单个进程中运行。

这些控制器包括：

- [节点（Node）控制器](https://www.bookstack.cn/read/Kubernetes-zh/32.md)。
- 副本（Replication）控制器：负责维护系统中每个副本中的pod。
- 端点（Endpoints）控制器：填充Endpoints对象（即连接Services＆Pods）。
- [Service Account](https://www.bookstack.cn/read/Kubernetes-zh/134.md)和Token控制器：为新的[Namespace](https://www.bookstack.cn/read/Kubernetes-zh/22.md) 创建默认帐户访问API Token。

### cloud-controller-manager

云控制器管理器负责与底层云提供商的平台交互。云控制器管理器是Kubernetes版本1.6中引入的，目前还是Alpha的功能。

云控制器管理器仅运行云提供商特定的（controller loops）控制器循环。可以通过将`—cloud-provider` flag设置为external启动kube-controller-manager ，来禁用控制器循环。

cloud-controller-manager 具体功能：

- 节点（Node）控制器
- 路由（Route）控制器
- Service控制器
- 卷（Volume）控制器

### kube-scheduler

kube-scheduler 监视新创建没有分配到[Node](https://www.bookstack.cn/read/Kubernetes-zh/32.md)的[Pod](https://www.bookstack.cn/read/Kubernetes-zh/34.md)，为Pod选择一个Node。

### 插件 

- addons

插件（addon）是实现集群pod和Services功能的 。Pod由[Deployments](https://www.bookstack.cn/read/Kubernetes-zh/45.md)，ReplicationController等进行管理。Namespace 插件对象是在kube-system Namespace中创建。

- DNS

虽然不严格要求使用插件，但Kubernetes集群都应该具有集群 DNS。

群集 DNS是一个DNS服务器，能够为 Kubernetes services提供 DNS记录。

由Kubernetes启动的容器自动将这个DNS服务器包含在他们的DNS searches中。

- 用户界面

kube-ui提供集群状态基础信息查看。更多详细信息，请参阅[使用HTTP代理访问Kubernetes API](https://kubernetes.io/docs/tasks/access-kubernetes-api/http-proxy-access-api/)

- Cluster-level Logging

负责保存容器日志，搜索/查看日志。



## Node组件

```
kubelet                    # 节点/pod管理
kube-proxy                 # 通过在主机上维护网络规则并执行连接转发来实现Kubernetes服务抽象
docker                     # 容器运行需要
```

### kubelet

[kubelet](https://kubernetes.io/docs/admin/kubelet)是主要的节点代理，它会监视已分配给节点的pod，具体功能：

- 安装Pod所需的volume。
- 下载Pod的Secrets。
- Pod中运行的 docker（或experimentally，rkt）容器。
- 定期执行容器健康检查。
- Reports the status of the pod back to the rest of the system, by creating a *mirror pod* if necessary.
- Reports the status of the node back to the rest of the system.

### kube-proxy

[kube-proxy](https://kubernetes.io/docs/admin/kube-proxy)通过在主机上维护网络规则并执行连接转发来实现Kubernetes服务抽象。

### docker

docker用于运行容器。