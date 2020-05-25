## 创建 namespace

创建一个单独的名字空间，以便于隔离您在本练习中创建的资源与集群的其他资源。

```
 $ kubectl create namespace quota-mem-cpu-example
```

## 创建ResourceQuota对象

以下展示了ResourceQuota对象的配置文件内容：

**quota-mem-cpu.yaml**

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-demo
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```

下面，首先创建ResourceQuota对象

```
$ kubectl apply -f quota-mem-cpu.yaml --namespace=quota-mem-cpu-example
```

然后可以通过以下命令查看ResourceQuota对象的详细信息：

```
$ kubectl get resourcequota mem-cpu-demo --namespace=quota-mem-cpu-example --output=yaml
```

以上刚创建的ResourceQuota对象将在quota-mem-cpu-example名字空间中添加以下限制：

- 每个容器必须设置内存请求（memory request），内存限额（memory limit），cpu请求（cpu request）和cpu限额（cpu limit）。
- 所有容器的内存请求总额不得超过1 GiB。
- 所有容器的内存限额总额不得超过2 GiB。
- 所有容器的CPU请求总额不得超过1 CPU。
- 所有容器的CPU限额总额不得超过2 CPU。

## 创建一个Pod

以下展示了一个Pod的配置文件内容：

**quota-mem-cpu-pod.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: quota-mem-cpu-demo
spec:
  containers:
  - name: quota-mem-cpu-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "800Mi"
        cpu: "800m" 
      requests:
        memory: "600Mi"
        cpu: "400m"
```

通过以下命令创建这个Pod：

```
$ kubectl apply -f quota-mem-cpu-pod.yaml --namespace=quota-mem-cpu-example
```

运行以下命令验证这个Pod的容器已经运行：

```
$ kubectl get pod quota-mem-cpu-demo --namespace=quota-mem-cpu-example
NAME                 READY   STATUS    RESTARTS   AGE
quota-mem-cpu-demo   1/1     Running   0          31
```

然后再次查看ResourceQuota对象的详细信息：

```
$ kubectl get resourcequota mem-cpu-demo --namespace=quota-mem-cpu-example --output=yaml
```

除了配额本身信息外，上述命令还显示了目前配额中有多少已经被使用。可以看到，刚才创建的Pod的内存以及 CPU的请求和限额并没有超出配额。

```
status:
  hard:
    limits.cpu: "2"
    limits.memory: 2Gi
    requests.cpu: "1"
    requests.memory: 1Gi
  used:
    limits.cpu: 800m
    limits.memory: 800Mi
    requests.cpu: 400m
    requests.memory: 600Mi
```

## 尝试创建第二个Pod

第二个Pod的配置文件如下所示：

**quota-mem-cpu-pod-2.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: quota-mem-cpu-demo-2
spec:
  containers:
  - name: quota-mem-cpu-demo-2-ctr
    image: redis
    resources:
      limits:
        memory: "1Gi"
        cpu: "800m"      
      requests:
        memory: "700Mi"
        cpu: "400m"
```

在配置文件中，您可以看到第二个Pod的内存请求是700 MiB。可以注意到，如果创建第二个Pod, 目前的内存使用量加上新的内存请求已经超出了当前名字空间的内存请求配额。即600 MiB + 700 MiB > 1 GiB。

下面尝试创建第二个Pod：

```
$ kubectl apply -f quota-mem-cpu-pod-2.yaml -n quota-mem-cpu-example
```

以下命令输出显示第二个Pod并没有创建成功。错误信息说明了如果创建第二个Pod，内存请求总额将超出名字空间的内存请求配额。

```
Error from server (Forbidden): error when creating "quota-mem-cpu-pod-2.yaml": pods "quota-mem-cpu-demo-2" is forbidden: exceeded quota: mem-cpu-demo, requested: requests.memory=700Mi, used: requests.memory=600Mi, limited: requests.memory=1Gi
```

## 练习环境的清理

通过删除名字空间即可完成环境清理：

```
$ kubectl delete namespace quota-mem-cpu-example
```