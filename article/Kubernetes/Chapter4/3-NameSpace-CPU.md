一个Kubernetes集群能细分为不同的命名空间。如果在一个拥有默认CPU限额的命名空间中创建一个容器，则这个容器不需要指定它自己的CPU限额， 它会被分配这个默认的CPU限额值。

## 创建一个命名空间

创建一个命名空间为了使你在本练习中创建的资源与集群的其它部分相隔离。

```
kubectl create namespace default-cpu-example
```

## 创建一个LimitRange和一个Pod

以下是一个LimitRange对象的配置文件。这个配置中指定了一个默认的CPU请求和一个默认的CPU限额。

```
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: 1
    defaultRequest:
      cpu: 0.5
    type: Container
```



```
[root@master ~]# kubectl apply -f cpu-defaults.yaml -n default-cpu-example
limitrange/cpu-limit-range created
[root@master ~]# kubectl get limitrange -n default-cpu-example
NAME              CREATED AT
cpu-limit-range   2020-05-25T08:09:50Z
```

现在如果在这个defaule-cpu-example命名空间中创建一个容器，则该容器不需要指定它自己的CPU请求和CPU限额， 该容器会被赋予一个默认的CPU请求值0.5和一个默认的CPU限额值1。

**cpu-defaults-pod.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: default-cpu-demo
spec:
  containers:
  - name: default-cpu-demo-ctr
    image: nginx
```

创建Pod

```
$ kubectl apply -f cpu-defaults-pod.yaml --namespace=default-cpu-example
```

查看该Pod的配置：

```
$ kubectl get pod default-cpu-demo --output=yaml --namespace=default-cpu-example
```

输出显示该Pod的容器含有一个CPU请求值500m和一个CPU限额值1。 这些是由LimitRange指定的默认值。

```
 containers:
  - image: nginx
    imagePullPolicy: Always
    name: default-cpu-demo-ctr
    resources:
      limits:
        cpu: "1"
      requests:
        cpu: 500m
```

## 如果指定了一个容器的限额值，但未指定请求值，会发生什么？

以下是一个含有一个容器的Pod的配置文件。该容器指定了一个CPU限额，但未指定请求：

**cpu-defaults-pod-2.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: default-cpu-demo-2
spec:
  containers:
  - name: default-cpu-demo-2-ctr
    image: nginx
    resources:
      limits:
        cpu: "1"
```

创建该Pod:

```
kubectl apply -f cpu-defaults-pod-2.yaml -n default-cpu-example
```

查看该Pod的配置：

```
$ kubectl get pod default-cpu-demo-2 --output=yaml --namespace=default-cpu-example
```

输出展示该容器的CPU请求值与它的限额值相等。注意该容器并未被赋予这个默认的CPU请求值0.5。

```
containers:
  - image: nginx
    imagePullPolicy: Always
    name: default-cpu-demo-2-ctr
    resources:
      limits:
        cpu: "1"
      requests:
        cpu: "1"
```

## 如果指定了一个容器的请求值，未指定限额值，会发生什么？

以下是含有一个容器的Pod的配置文件。该容器指定了一个CPU请求，但未指定一个限额：

**cpu-defaults-pod-3.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: default-cpu-demo-3
spec:
  containers:
  - name: default-cpu-demo-3-ctr
    image: nginx
    resources:
      requests:
        cpu: "0.75"
```

创建该Pod

```
$ kubectl apply -f cpu-defaults-pod-3.yaml -n default-cpu-example
```

查看该Pod的配置：

```
$ kubectl get pod default-cpu-demo-3 --output=yaml --namespace=default-cpu-example
```

```
 containers:
  - image: nginx
    imagePullPolicy: Always
    name: default-cpu-demo-3-ctr
    resources:
      limits:
        cpu: "1"
      requests:
        cpu: 750m
```

输出显示该容器的CPU请求值被设置为该容器配置文件中指定的值。该容器的CPU限额设置为1，这是该命名空间的默认

## 默认CPU限额和请求的动机

如果你的命名空间含有[资源配额](https://kubernetes.io/docs/tasks/administer-cluster/cpu-default-namespace/), 它是有帮助的对于设置一个CPU限额的默认值。 以下是资源配额对命名空间施加的两个限制：

- 在命名空间运行的每一个容器必须含有它自己的CPU限额。
- 在命名空间中所有容器使用的CPU总量不能超出指定的限额。
  如果一个容器没有指定它自己的CPU限额，它将被赋予默认的限额值，然后它可以在被配额限制的命名空间中运行。

## 练习环境的清理

通过删除名字空间即可完成环境的清理：

```
$ kubectl delete namespace default-cpu-example
namespace "default-cpu-example" deleted
```

