## 创建 Redis-master Pod和服务
首先为 redis-master 服务创建一个名为 redis-master 的RC定义文件：redis-master.yaml
<br>
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: redis-master
  labels:
    name: redis-master
spec:
  replicas: 1
  selector:
    name: redis-master
  template:
    metadata:
      labels:
        name: redis-master
    spec:
      containers:
      - name: master
        image: kubeguide/redis-master
        ports:
        - containerPort: 6379
```
<br>
其中，kind字段的值为“ReplicationController”，表示这是一个RC；spec.selector是RC的Pod选择器，即监控和管理拥有这些标签（Label）的Pod实例，确保当前集群上始终有且仅有 replicas 个 Pod 实例在运行，这里我们设置 replicas=1，表示只能运行一个名为redis-master的Pod实例，当集群中运行的Pod数量小于replicas时，RC会根据 spec.template段定义的Pod模版来生成一个新的Pod实例，labels属性指定了该Pod的标签，注意，这里的labels必须匹配RC的 spec.selector，否则此 RC 就会陷入 “只为他人做嫁衣”的悲惨世界中，永无翻身之时。
</br>
创建好后，可以在 Master 节点执行命令：`kubectl create -f redis-master.yaml`，将它发布到集群中，就完成了 redis-master 的创建过程。
<br>
```yaml
[root@rancher-0 ~]# kubectl create -f redis-master.yaml 
replicationcontroller/redis-master created

[root@rancher-0 ~]# kubectl get rc
NAME           DESIRED   CURRENT   READY   AGE
redis-master   1         1         1       25s
```
<br>
接下来运行 `kubectl get pods`命令来查看当前系统中的Pod列表信息，我们看到一个名为redis-master-xxx的Pod实例，这是Kubernetes根据redis-master这个RC定义自动创建的Pod。由于Pod调度和创建需要花费一定的时间，比如需要一定的时间来确定调度到哪个节点上，以及下载Pod的相关镜像，所以一开始我们看到Pod状态为`Pending`。当Pod成功创建完成以后，状态会更新为 `Running`。
<br>
```yaml
[root@rancher-0 ~]# kubectl get pods
NAME                 READY   STATUS    RESTARTS   AGE
redis-master-xtsdr   1/1     Running   0          2m44s
```
<br>
提供Redis服务的Pod已经创建好了，接下来创建一个与之关联的Service（服务），redis-master的定义文件名为：redis-master-svc.yaml，内容如下：
<br>
```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-master
  labels:
    name: redis-master
spec:
  ports:
  - port: 6379
    targetPort: 6379
  selector:
    name: redis-master
```
<br>
其中 metadata.name 是 Service 的服务名（ServiceName），spec.selector 确定了哪些Pod对应到本服务，这里的定义表明拥有 redis-master 标签的 Pod 属于 redis-master 服务。另外，ports部分中的 targetPort 属性用来确定提供该服务的容器所暴露（EXPOSE）的端口号，即具体的服务进程在容器内的 targetPort 上提供服务，而 port 属性则定义了 Service 的虚端口。
<br>
运行 kubectl，创建 Service：
<br>
```yaml
[root@rancher-0 ~]# kubectl create -f redis-master-svc.yaml 
service/redis-master created
[root@rancher-0 ~]# kubectl get svc            
NAME           TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
kubernetes     ClusterIP   10.1.0.1      <none>        443/TCP    42h
redis-master   ClusterIP   10.1.43.109   <none>        6379/TCP   34s
```
<br>
注意到 redis-master 服务被分配了一个值为 10.1.43.109 的IP地址，随后，Kubernetes集群中其他创建的 Pod 就可以通过这个IP地址+端口6379来访问它了。
</br>
但由于IP地址是在服务创建后由Kubernetes系统自动分配的，在其他Pod中无法预先知道某个Service的虚拟IP地址，因此需要一个机制来找到这个服务。为此，Kubernetes使用了`Linux环境变量`，在每个Pod的容器里都增加了一组Service相关的环境变量，用来记录从服务名到虚拟IP地址的映射关系。以 redis-master 为例，在容器中的环境变量会增加下面两条记录：
<br>  
```yaml
REDIS_MASTER_SERVICE_HOST=10.1.43.109
REDIS_MASTER_SERVICE_PORT=6379
```
<br>  
于是，redis-slave 和 frontend 等 Pod 中的应用程序就可以通过环境变量 `REDIS_MASTER_SERVICE_HOST` 得到 redis-master 服务的虚拟IP地址，通过环境变量 `REDIS_MASTER_SERVICE_PORT`得到 redis-master 服务的端口号，完成了对服务地址的查询功能。
