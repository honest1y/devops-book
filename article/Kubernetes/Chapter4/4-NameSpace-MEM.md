如果在一个拥有默认内存限额的命名空间中创建一个容器，并且这个容器未指定它自己的内存限额， 它会被分配这个默认的内存限额值。

## 创建命名空间

创建一个命名空间，以便您在本练习中创建的资源与集群的其它部分相隔离。

```
$ kubectl create namespace default-mem-example
```

## 创建 LimitRange 和 Pod

以下是一个 LimitRange 对象的配置文件。该配置指定了默认的内存请求与默认的内存限额。

**memory-defaults.yaml**

```
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```

在 default-mem-example 命名空间中创建 LimitRange：

```
$ kubectl apply -f memory-defaults.yaml --namespace=default-mem-example
```

现在如果在这个 default-mem-example 命名空间中创建一个容器，并且该容器未指定它自己的内存请求与内存限额， 该容器会被赋予默认的内存请求值 256MiB 和默认的内存限额值 512MiB。

以下是一个 Pod 的配置文件，它含有一个容器。这个容器没有指定内存请求和限额。

**memory-defaults-pod.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: default-mem-demo
spec:
  containers:
  - name: default-mem-demo-ctr
    image: nginx
```

创建 Pod:

```
$ kubectl apply -f memory-defaults-pod.yaml --namespace=default-mem-example
```

查看关于该 Pod 的详细信息：

```
containers:
  - image: nginx
    imagePullPolicy: Always
    name: default-mem-demo-ctr
    resources:
      limits:
        memory: 512Mi
      requests:
        memory: 256Mi
```

删除 Pod:

```
kubectl delete pod default-mem-demo --namespace=default-mem-example
```

## 如果指定了容器的限额值，但未指定请求值，会发生什么？

以下是含有一个容器的 Pod 的配置文件。该容器指定了内存限额，但未指定请求：

**memory-defaults-pod-2.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: default-mem-demo-2
spec:
  containers:
  - name: defalt-mem-demo-2-ctr
    image: nginx
    resources:
      limits:
        memory: "1Gi"
```

创建Pod：

```
$ kubectl apply -f memory-defaults-pod-2.yaml -n default-mem-example
```

查看关于该 Pod 的详细信息：

```
$ kubectl get pod mem-limit-no-request --output=yaml --namespace=default-mem-example
```

输出显示该容器的内存请求值与它的限额值相等。注意该容器并未被赋予默认的内存请求值 256MiB。

```
containers:
  - image: nginx
    imagePullPolicy: Always
    name: defalt-mem-demo-2-ctr
    resources:
      limits:
        memory: 1Gi
      requests:
        memory: 1Gi
```

## 如果指定了容器的请求值，但未指定限额值，会发生什么？

以下是含有一个容器的 Pod 的配置文件。该容器指定了内存请求，但未指定限额：

**memory-defaults-pod-3.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: default-mem-demo-3
spec:
  containers:
  - name: default-mem-demo-3-ctr
    image: nginx
    resources:
      requests:
        memory: "128Mi"
```

创建该 Pod:

```
kubectl apply -f memory-defaults-pod-3.yaml --namespace=default-mem-example
```

查看该 Pod 的配置信息：

```
kubectl get pod default-mem-request-no-limit --output=yaml --namespace=default-mem-example
```

输出显示该容器的内存请求值被设置为该容器配置文件中指定的值。该容器的内存限额设置为 512Mi，这是该命名空间的默认内存限额值。

```
containers:
  - image: nginx
    imagePullPolicy: Always
    name: default-mem-demo-3-ctr
    resources:
      limits:
        memory: 512Mi
      requests:
        memory: 128Mi
```

## 默认内存限额与请求的动机

如果您的命名空间具有资源配额, 它为内存限额设置默认值是有意义的。 以下是资源配额对命名空间施加的两个限制：

- 在命名空间运行的每一个容器必须有它自己的内存限额。
- 在命名空间中所有的容器使用的内存总量不能超出指定的限额。
  如果一个容器没有指定它自己的内存限额，它将被赋予默认的限额值，然后它才可以在被配额限制的命名空间中运行。

## 练习环境的清理

通过删除名字空间即可完成环境的清理：

```
$ kubectl delete namespace default-mem-example
namespace "default-mem-example" deleted
```

