## 创建名字空间

创建一个单独的名字空间，以便于隔离您在本练习中创建的资源与集群的其他资源。

```
kubectl create namespace quota-pod-example
```

## 创建ResourceQuota对象

以下展示了ResourceQuota对象的配置文件内容：

**quota-pod.yaml**

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: pod-demo
  namespace: quota-pod-example
spec:
  hard:
    pods: "2"
```

下面，首先创建ResourceQuota对象

```
$ kubectl apply -f quota-pod.yaml 
resourcequota/pod-demo created
```

然后可以通过以下命令查看ResourceQuota对象的详细信息：

```
kubectl get resourcequota pod-demo --namespace=quota-pod-example --output=yaml
```

命令输出显示了这个名字空间的Pod配额是2，由于目前没有Pod运行，所有配额并没有被使用。

```
spec:
  hard:
    pods: "2"
status:
  hard:
    pods: "2"
  used:
    pods: "2"
```

下面展示的是一个Deployment的配置文件：

**quota-pod-deployment.yaml**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-quota-demo
  namespace: quota-pod-example
  labels:
    app: pod-quota-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: pod-quota-demo 
  template:
    metadata:
      labels:
        app: pod-quota-demo
    spec:
      containers:
      - name: pod-quota-demo
        image: nginx
```

从配置文件可以看到，replicas: 3将令Kubernetes尝试创建3个Pod，所有的Pod实例都将运行同样的应用程序。

接下来尝试创建这个Deployment：

```
[root@master ~]# kubectl apply -f quota-pod-deployment.yaml
deployment.apps/pod-quota-demo created
```

并通过以下命令查看Deployment的详细信息：

```
kubectl get deployment pod-quota-demo --namespace=quota-pod-example --output=yaml
```

从命令输出可以看到尽管在Deployment中我们设置了需要启动3个Pod实例，但由于配额的存在，只有两个Pod被成功创建。

```
status:
  availableReplicas: 2
  ...
    lastUpdateTime: "2020-05-25T07:54:49Z"
    message: 'pods "pod-quota-demo-74cb949d49-vmldb" is forbidden: exceeded quota:
      pod-demo, requested: pods=1, used: pods=2, limited: pods=2'
```

## 练习环境的清理

通过删除名字空间即可完成环境的清理：

```
$ kubectl delete namespace quota-pod-example
```