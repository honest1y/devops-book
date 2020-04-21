## Ingress介绍

Kubernetes 暴露服务的有三种方式，分别为 LoadBlancer Service、NodePort Service、Ingress。官网对 Ingress 的定义为管理对外服务到集群内服务之间规则的集合，通俗点讲就是它定义规则来允许进入集群的请求被转发到集群中对应服务上，从来实现服务暴漏。 Ingress 能把集群内 Service 配置成外网能够访问的 URL，流量负载均衡，终止SSL，提供基于域名访问的虚拟主机等等。

### LoadBlancer Service

LoadBlancer Service 是 Kubernetes 结合云平台的组件，如国外 GCE、AWS、国内阿里云等等，使用它向使用的底层云平台申请创建负载均衡器来实现，有局限性，对于使用云平台的集群比较方便。

### NodePort Service

NodePort Service 是通过在节点上暴漏端口，然后通过将端口映射到具体某个服务上来实现服务暴漏，比较直观方便，但是对于集群来说，随着 Service 的不断增加，需要的端口越来越多，很容易出现端口冲突，而且不容易管理。当然对于小规模的集群服务，还是比较不错的。

### Ingress

Ingress 使用开源的反向代理负载均衡器来实现对外暴漏服务，比如 Nginx、Apache、Haproxy等。Nginx Ingress 一般有三个组件组成：
1）ingress是kubernetes的一个资源对象，用于编写定义规则。
2）反向代理负载均衡器，通常以Service的Port方式运行，接收并按照ingress定义的规则进行转发，通常为nginx，haproxy，traefik等，本文使用nginx。
3）ingress-controller，监听apiserver，获取服务新增，删除等变化，并结合ingress规则动态更新到反向代理负载均衡器上，并重载配置使其生效。
以上三者有机的协调配合起来，就可以完成 Kubernetes 集群服务的暴漏。

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/ingress.png)

### 组件说明

**externalLB** :  外部的4层负载均衡器

**<Service> ingress-nginx **: nodePort 类型的 service 为 <IngressController> ingress-nginx 的 pod 接入外部流量

**<IngressController> ingress-nginx** : ingress-nginx pod, 负责创建负载均衡
**<Ingress>** : Ingress 根据后端 Service 实时识别分类及 IP 把结果生成配置文件注入到 ingress-nginx pod 中
**<Service> site1** : Service 对后端的pod 进行分类(只起分类作用)



###YAML格式

- API 和 kind

```
apiVersion: extensions

kind: ingress
```

- ingress.spec

```
backend         # 后端POD
rules           # 调度规则
    host        # 虚拟主机
    http        # http 路径
```