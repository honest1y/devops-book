## 创建 frontend Pod和服务

类似的，定义 frontend 的 RC 配置文件，内容如下：

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: frontend
  labels:
    name: frontend
spec:
  replicas: 3
  selector:
    name: frontend
  template:
    metadata:
      labels:
        name: frontend
    spec:
      containers:
      - name: frontend
        image: kubeguide/guestbook-php-frontend
        env:
        - name: GET_HOSTS_FROM
          value: env
        ports:
        - containerPort: 80
```

<br>

运行 kubectl create 命令创建 RC：

```yaml
[root@rancher-0 ~]# kubectl create -f frontend.yaml 
replicationcontroller/frontend created
```

查看已创建的 RC：

```yaml
[root@rancher-0 ~]# kubectl get rc
NAME           DESIRED   CURRENT   READY   AGE
frontend       3         3         1       19s
redis-master   1         1         1       6h45m
redis-slave    2         2         2       146m
```

再查看生成的 Pod：

```yaml
[root@rancher-0 ~]# kubectl get pods
NAME                 READY   STATUS    RESTARTS   AGE
frontend-62n5t       1/1     Running   0          47s
frontend-tk9tb       1/1     Running   0          47s
frontend-zmbw6       1/1     Running   0          47s
redis-master-xtsdr   1/1     Running   0          6h45m
redis-slave-g7dvv    1/1     Running   0          147m
redis-slave-pk5g8    1/1     Running   0          147m
```

最后创建 frontend Service， 主要目的是使用 Service 的 NodePort 给 Kubernetes 集群中的 Service 映射一个外网可以访问的端口，这样一来，外部网络就可以通过 NodeIP + NodePort的方式访问集群中的服务了。

<br>

服务定义文件如下：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    name: frontend
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30001
  selector:
    name: frontend
```

这里的关键点是设置 `type = NodePort`，并指定一个 NodePort 的值，表示使用 Node 上的物理机端口提供对外访问的能力。需要注意的是， spec.ports.NodePort 的端口号定义有范围限制，默认为 `30000 ~ 32767`，如果配置为范围外的端口号，则创建 Service 失败。

运行 kubectl 创建 Service：

```yaml
[root@rancher-0 ~]# kubectl create -f frontend-svc.yaml 
service/frontend created
```

查看已创建的 Service：

```yaml
[root@rancher-0 ~]# kubectl get svc
NAME           TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
frontend       NodePort    10.1.162.83   <none>        80:30001/TCP   23s
kubernetes     ClusterIP   10.1.0.1      <none>        443/TCP        2d1h
redis-master   ClusterIP   10.1.43.109   <none>        6379/TCP       6h42m
redis-slave    ClusterIP   10.1.197.43   <none>        6379/TCP       148m
```


