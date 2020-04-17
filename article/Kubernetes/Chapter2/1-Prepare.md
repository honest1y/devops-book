## 环境准备
Kubernetes支持在物理服务器或虚拟机中运行，本次使用虚拟机准备测试环境，硬件配置信息如表所示：

IP地址 | 节点角色 | CPU | 内存 | Hostname 
---|---|---|---|---
172.20.2.10 | master | 4 | 4 | master
172.20.2.11 | node   | 4 | 4 | node-1
172.20.2.12 | node   | 4 | 4 | node-2
172.20.2.13 | node   | 4 | 4 | node-3


##### 1、设置主机名
设置主机名hostname，管理节点设置主机名为 master
```
hostnamectl set-hostname master
```
需要设置其他主机名称时，可将 master 替换为正确的主机名node1、node2即可。
##### 2、添加解析
编辑 /etc/hosts 文件，添加域名解析
```
cat <<EOF >>/etc/hosts
172.20.2.10 master
172.20.2.11 node-1
172.20.2.12 node-2
172.20.2.13 node-3
EOF
```
##### 3、关闭防火墙、selinux和swap
```
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
swapoff -a
sed -i 's/.*swap.*/#&/' /etc/fstab
```
##### 4、配置内核参数，将桥接的IPv4流量传递到iptables的链
```
cat > /etc/sysctl.d/k8s.conf <<EOF

net.bridge.bridge-nf-call-ip6tables = 1

net.bridge.bridge-nf-call-iptables = 1

EOF

sysctl --system
```
##### 5、配置国内yum源
```
yum install -y wget

mkdir /etc/yum.repos.d/bak && mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/bak

wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.cloud.tencent.com/repo/centos7_base.repo

wget -O /etc/yum.repos.d/epel.repo http://mirrors.cloud.tencent.com/repo/epel-7.repo

yum clean all && yum makecache
```

##### 6、配置国内Kubernetes源
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo

[kubernetes]

name=Kubernetes

baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/

enabled=1

gpgcheck=1

repo_gpgcheck=1

gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg

EOF
```
##### 7、配置 docker 源
```
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
```
