## 集群配置
##### 1、安装docker
```
yum install -y docker-ce-18.06.1.ce-3.el7

systemctl enable docker && systemctl start docker

docker –version

Docker version 18.06.1-ce, build e68fc7a
```
##### 2、安装kubeadm
```
yum install -y kubelet-1.14.3 kubeadm-1.14.3 kubectl-1.14.3
systemctl enable kubelet
```
Kubelet负责与其他节点集群通信，并进行本节点Pod和容器生命周期的管理。Kubeadm是Kubernetes的自动化部署工具，降低了部署难度，提高效率。Kubectl是Kubernetes集群管理工具。
##### 3、部署master 节点
###### 3.1 集群初始化
```yaml
kubeadm init --kubernetes-version=1.14.2 \

--apiserver-advertise-address=172.20.2.10 \

--image-repository registry.aliyuncs.com/google_containers \

--service-cidr=10.1.0.0/16 \

--pod-network-cidr=10.244.0.0/16
```
定义POD的网段为: `10.244.0.0/16`， api server地址就是master本机IP地址。

这一步很关键，由于kubeadm 默认从官网k8s.grc.io下载所需镜像，国内无法访问，因此需要通过–image-repository指定阿里云镜像仓库地址，很多新手初次部署都卡在此环节无法进行后续配置。

集群初始化成功后返回如下信息：

记录生成的最后部分内容，此内容需要在其它节点加入Kubernetes集群时执行。
```yaml
kubeadm join 172.20.2.10:6443 --token kekvgu.nw1n76h84f4camj6 \

--discovery-token-ca-cert-hash sha256:4ee74205227c78ca62f2d641635afa4d50e6634acfaa8291f28582c7e3b0e30e
```
###### 3.2 配置kubectl工具
```
mkdir -p /root/.kube

cp /etc/kubernetes/admin.conf /root/.kube/config

kubectl get nodes

kubectl get cs
```

###### 3.3 部署flannel网络
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml
```
##### 4、部署node节点
执行如下命令，使所有node节点加入Kubernetes集群
```
kubeadm join 172.20.2.10:6443 --token kekvgu.nw1n76h84f4camj6 \

--discovery-token-ca-cert-hash sha256:4ee74205227c78ca62f2d641635afa4d50e6634acfaa8291f28582c7e3b0e30e
```
此命令为集群初始化时（kubeadm init）返回结果中的内容
##### 5、集群状体检测
```
kubectl get nodes
NAME     STATUS   ROLES    AGE     VERSION
master   Ready    master   26m     v1.14.2
node1    Ready    <none>   3m10s   v1.14.2
node2    Ready    <none>   3m      v1.14.2
node3    Ready    <none>   3m      v1.14.2
```
