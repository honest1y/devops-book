Kubernetes可以使用Namespaces（命名空间）创建多个虚拟集群。

## 何时使用多个Namespace

当团队或项目中具有许多用户时，可以考虑使用Namespace来区分，a如果是少量用户集群，可以不需要考虑使用Namespace，如果需要它们提供特殊性质时，可以开始使用Namespace。

Namespace为名称提供了一个范围。资源的Names在Namespace中具有唯一性。

Namespace是一种将集群资源划分为多个用途(通过 [resource quota](https://kubernetes.io/docs/concepts/policy/resource-quotas/))的方法。

在未来的Kubernetes版本中，默认情况下，相同Namespace中的对象将具有相同的访问控制策略。

对于稍微不同的资源没必要使用多个Namespace来划分，例如同意软件的不同版本，可以使用[labels(标签)](https://www.bookstack.cn/read/Kubernetes-zh/29.md)来区分同一Namespace中的资源。

## 使用 Namespace

Namespace的创建、删除和查看。

### 创建

```
(1) 命令行直接创建
$ kubectl create namespace new-namespace
 
(2) 通过文件创建
$ cat my-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: new-namespace
 
$ kubectl create -f ./my-namespace.yaml
```

### 删除

```
$ kubectl delete namespace new-namespace
```

注意：

- 删除一个namespace会自动删除所有属于该namespace的资源。
- default和kube-system命名空间不可删除。
- PersistentVolumes是不属于任何namespace的，但PersistentVolumeClaim是属于某个特定namespace的。
- Events是否属于namespace取决于产生events的对象。

### 查看 Namespaces

使用以下命令列出群集中的当前的Namespace：

```
$ kubectl get namespace
NAME          STATUS    AGEdefault       Active    1dkube-system   Active    1d
```

Kubernetes从两个初始的Namespace开始：

- default
- kube-system 由Kubernetes系统创建的对象的Namespace



## 所有对象都在Namespace中?

大多数Kubernetes资源（例如pod、services、replication controllers或其他）都在某些Namespace中，但Namespace资源本身并不在Namespace中。而低级别资源（如[Node](https://www.bookstack.cn/read/Kubernetes-zh/32.md)和persistentVolumes）不在任何Namespace中。[Events](https://www.kubernetes.org.cn/1031.html)是一个例外：它们可能有也可能没有Namespace，具体取决于[Events](https://www.kubernetes.org.cn/1031.html)的对象。

