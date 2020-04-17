##1.2.3、Rancher架构

大多数Rancher2.0软件运行在Rancher Server节点上,Rancher Server包括用于管理整个Rancher部署的所有组件。

下图说明了Rancher2.0的运行架构。该图描绘了管理两个Kubernetes集群的Rancher server安装:一个由RKE创建，另一个由GKE创建。

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/rancher-ar.png)

#### Rancher API服务器

Rancher API server建立在嵌入式Kubernetes API服务器和etcd数据库之上。它实现了以下功能:

- **Rancher API服务器**
  Rancher API server管理与外部身份验证提供程序(如Active Directory或GitHub)对应的用户身份
- **认证授权**
  Rancher API server管理访问控制和安全策略
- **项目**
  项目是集群中的一组多个命名空间和访问控制策略的集合
- **节点**
  Rancher API server跟踪所有集群中所有节点的标识。



#### 集群控制和Agent

集群控制器和集群代理实现管理Kubernetes集群所需的业务逻辑:

- **集群控制器实现Rancher安装所需的全局逻辑。它执行以下操作:**
  - 为集群和项目配置访问控制策略
  - 通过调用以下方式配置集群:
    - 所需的Docker machine驱动程序
    - 像RKE和GKE这样的Kubernetes引擎
- **单独的集群代理实例实现相应集群所需的逻辑。它执行以下活动:**
  - 工作负载管理，例如每个集群中的pod创建和部署
  - 绑定并应用每个集群全局策略中定义的角色
  - 集群与Rancher Server之间的通信:事件，统计信息，节点信息和运行状况



#### 认证代理

该认证代理转发所有Kubernetes API调用。它集成了身份验证服务，如本地身份验证，Active Directory和GitHub。在每个Kubernetes API调用中，身份验证代理会对调用方进行身份验证，并在将调用转发给Kubernetes主服务器之前设置正确的Kubernetes模拟标头。Rancher使用服务帐户与Kubernetes集群通信。

