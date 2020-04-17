## 创建 redis-slave Pod和服务

接下来我们继续完成 redis-slave 服务的创建过程，在本次实验中， redis-slave 会启动两个副本，每个副本的 Redis 进程都与redis-master所对应的Redis进程进行数据同步，3个Redis实例组成了一个具备读写分离能力的Redis集群。

留言板的PHP程序通过访问 redis-slave 服务来获取已保存的留言数据。与之前的 redis-master 服务的创建过程一样，首先创建一个名为 redis-slave 的RC定义文件：

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis-slave
  labels:
    name: redis-slave
spec:
  replicas: 2
  selector:
    name: redis-slave
  template:
    metadata:
      labels:
        name: redis-slave
    spec:
      containers:
      - name: master
        image: kubeguide/guestbook-redis-slave
        env:
        - name: GET_HOSTS_FROM
          value: env
        ports:
        - containerPort: 6379
```

运行 kubectl create 命令：

```yaml
[root@rancher-0 ~]# kubectl create -f redis-slave.yaml 
replicationcontroller/redis-slave created
```

运行 kubectl get 命令查看 RC：

```yaml
[root@rancher-0 ~]# kubectl get rc
NAME           DESIRED   CURRENT   READY   AGE
redis-master   1         1         1       4h19m
redis-slave    2         2         2       27s
```

运行 kubectl get 命令查看 Pod：

```yaml
[root@rancher-0 ~]# kubectl get pods
NAME                 READY   STATUS    RESTARTS   AGE
redis-master-xtsdr   1/1     Running   0          4h19m
redis-slave-g7dvv    1/1     Running   0          47s
redis-slave-pk5g8    1/1     Running   0          47s
```

然后创建 redis-slave 服务，内容如下：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
  labels:
    name: redis-slave
spec:
  ports:
  - port: 6379
  selector:
    name: redis-slave
```

运行 kubectl 创建 Service：

```yaml
[root@rancher-0 ~]# kubectl create -f redis-slave-svc.yaml 
service/redis-slave created
```

运行 kubectl get 查看创建的 Service：

```yaml
[root@rancher-0 ~]# kubectl get svc
NAME           TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
kubernetes     ClusterIP   10.1.0.1      <none>        443/TCP    47h
redis-master   ClusterIP   10.1.43.109   <none>        6379/TCP   4h14m
redis-slave    ClusterIP   10.1.197.43   <none>        6379/TCP   22s
[root@rancher-0 ~]# 
```
