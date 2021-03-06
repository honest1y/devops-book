本文展示了如何为 namespace 中运行的容器设置内存的最小和最大值。您可以设置 [LimitRange](https://k8smeetup.github.io/docs/api-reference/v1.8/#limitrange-v1-core) 对象中内存的最小和最大值。如果 Pod 没有符合 LimitRange 施加的限制，那么它就不能在 namespace 中创建。

## 创建一个 namespace

请创建一个 namespace，这样您在本练习中创建的资源就可以和集群中其余资源相互隔离。

```
$ kubectl create namespace constraints-mem-example
```

## 创建一个 LimitRange 和一个 Pod

这是 LimitRange 的配置文件：

**memory-constraints.yaml**

```
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-min-max-demo-lr
spec:
  limits:
  - max:
      memory: 1Gi
    min:
      memory: 500Mi
    type: Container
```

创建 LimitRange:

```
kubectl apply -f memory-constraints.yaml --namespace=constraints-mem-example
```

查看 LimitRange 的详细信息：

```
kubectl get limitrange mem-min-max-demo-lr --namespace=constraints-mem-example --output=yaml
```

输出显示了符合预期的最小和最大内存限制。但请注意，即使您没有在配置文件中为 LimitRange 指定默认值，它们也会被自动创建。

```
spec:
  limits:
  - default:
      memory: 1Gi
    defaultRequest:
      memory: 1Gi
    max:
      memory: 1Gi
    min:
      memory: 500Mi
    type: Container
```

现在，每当在 constraints-mem-example namespace 中创建一个容器时，Kubernetes 都会执行下列步骤：

- 如果容器没有指定自己的内存请求（request）和限制（limit），系统将会为其分配默认值。
- 验证容器的内存请求大于等于 500 MiB。
- 验证容器的内存限制小于等于 1 GiB。
  这是一份包含一个容器的 Pod 的配置文件。这个容器的配置清单指定了 600 MiB 的内存请求和 800 MiB 的内存限制。这些配置符合 LimitRange 施加的最小和最大内存限制。



### 验证

**memory-constraints-pod.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: constraints-mem-demo
spec:
  containers:
  - name: constraints-mem-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "800Mi"
      requests:
        memory: "600Mi"
```

创建Pod：

```
$ kubectl apply -f memory-constraints-pod.yaml -n constraints-mem-example
```

验证 Pod 的容器是否运行正常：

```
$ kubectl get pod constraints-mem-demo --namespace=constraints-mem-example
NAME                   READY   STATUS    RESTARTS   AGE
constraints-mem-demo   1/1     Running   0          23s
```

查看关于 Pod 的详细信息：

```
$ kubectl get pod constraints-mem-demo --output=yaml --namespace=constraints-mem-example
```

输出显示了容器的内存请求为 600 MiB，内存限制为 800 MiB。这符合 LimitRange 施加的限制。

```
 containers:
  - image: nginx
    imagePullPolicy: Always
    name: constraints-mem-demo-ctr
    resources:
      limits:
        memory: 800Mi
      requests:
        memory: 600Mi
```

删除 Pod：

```
$ kubectl delete pod constraints-mem-demo --namespace=constraints-mem-example
```

## 尝试创建一个超过最大内存限制的 Pod

这是一份包含一个容器的 Pod 的配置文件。这个容器的配置清单指定了 800 MiB 的内存请求和 1.5 GiB 的内存限制。

```
apiVersion: v1
kind: Pod
metadata:
  name: constraints-mem-demo-2
spec:
  containers:
  - name: constraints-mem-demo-2-ctr
    image: nginx
    resources:
      limits:
        memory: "1.5Gi"
      requests:
        memory: "800Mi"
```

尝试创建 Pod：

```
$ kubectl apply -f memory-constraints-pod-2.yaml --namespace=constraints-mem-example
```

输出显示 Pod 没有能够成功创建，因为容器指定的内存限制值太大：

```
Error from server (Forbidden): error when creating "memory-constraints-pod-2.yaml": pods "constraints-mem-demo-2" is forbidden: maximum memory usage per Container is 1Gi, but limit is 1536Mi
```

## 尝试创建一个不符合最小内存请求的 Pod

这是一份包含一个容器的 Pod 的配置文件。这个容器的配置清单指定了 200 MiB 的内存请求和 800 MiB 的内存限制。

**memory-constraints-pod-3.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: constraints-mem-demo-3
spec:
  containers:
  - name: constraints-mem-demo-3-ctr
    image: nginx
    resources:
      limits:
        memory: "800Mi"
      requests:
        memory: "100Mi"
```

尝试创建 Pod：

```
$ kubectl apply -f memory-constraints-pod-3.yaml --namespace=constraints-mem-example 
```

输出显示 Pod 没有能够成功创建，因为容器指定的内存请求值太小：

```
Error from server (Forbidden): error when creating "memory-constraints-pod-3.yaml": pods "constraints-mem-demo-3" is forbidden: minimum memory usage per Container is 500Mi, but request is 100Mi
```

## 创建一个没有指定任何内存请求和限制的 Pod

这是一份包含一个容器的 Pod 的配置文件。这个容器没有指定内存请求，也没有指定内存限制。

**memory-constraints-pod-4.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: constraints-mem-demo-4
spec:
  containers:
  - name: constraints-mem-demo-4-ctr
    image: nginx
```

创建 Pod：

```
$ kubectl apply -f memory-constraints-pod-4.yaml --namespace=constraints-mem-example
```

查看关于 Pod 的细节信息：

```
$ kubectl get pod constraints-mem-demo-4 --namespace=constraints-mem-example --output=yaml
```

输出显示 Pod 的容器具有 1 GiB 的内存请求和 1 GiB 的内存限制。容器是如何获取这些值的呢？

```
 containers:
  - image: nginx
    imagePullPolicy: Always
    name: constraints-mem-demo-4-ctr
    resources:
      limits:
        memory: 1Gi
      requests:
        memory: 1Gi
```

因为您的容器没有指定自己的内存请求和限制，它将从 LimitRange 获取 [默认的内存请求和限制值](https://k8smeetup.github.io/docs/tasks/administer-cluster/memory-default-namespace/)。

到目前为止，您的容器可能在运行，也可能没有运行。回想起来，有一个先决条件就是节点必须拥有至少 1 GiB 内存。如果每个节点都只有 1 GiB 内存，那么任何一个节点上都没有足够的内存来容纳 1 GiB 的内存请求。如果碰巧使用的节点拥有 2 GiB 内存，那么它可能会有足够的内存来容纳 1 GiB 的内存请求。

删除 Pod：

```
$ kubectl delete pod constraints-mem-demo-4 --namespace=constraints-mem-example
```

## 应用最小和最大内存限制

LimitRange 在 namespace 中施加的最小和最大内存限制只有在创建和更新 Pod 时才会被应用。改变 LimitRange 不会对之前创建的 Pod 造成影响。

## 最小和最大内存限制的动因

作为一个集群管理员，您可能希望为 Pod 能够使用的内存数量施加限制。例如：

- 集群中每个节点拥有 2 GB 内存。您不希望任何 Pod 请求超过 2 GB 的内存，因为集群中没有节点能支持这个请求。
- 集群被生产部门和开发部门共享。 您希望生产负载最多使用 8 GB 的内存而将开发负载限制为 512 MB。这种情况下，您可以为生产环境和开发环境创建单独的 namespace，并对每个 namespace 应用内存限制。

## 清理

删除 namespace：

```
$ kubectl delete namespace constraints-mem-example
```

