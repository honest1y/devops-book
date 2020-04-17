## 4.3、安装Rancher

#### 1、主机名配置

因为K8S的规定，主机名只支持包含 **-** 和 **.**(中横线和点)两种特殊符号，并且主机名不能出现重复。

#### 2、Hosts

配置每台主机的**hosts(/etc/hosts)**,添加**host_ip $hostname**到**/etc/hosts**文件中。

#### 3、关闭SELinux

```bash
sudo sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

#### 4、关闭防火墙

- CentOS

```bash
systemctl stop firewalld.service && systemctl disable firewalld.service
```

- Ubuntu

```bash
ufw disable
```

#### 5、配置主机时间、时区、系统语言

- 查看时区

```bash
date -R
或者
timedatectl
```

- 修改时区

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

- 修改系统语言环境

```bash
sudo echo 'LANG="en_US.UTF-8"' >> /etc/profile;source /etc/profile
```

#### 6、Kernel性能调优

```bash
cat >> /etc/sysctl.conf<<EOF
net.ipv4.ip_forward=1
net.bridge.bridge-nf-call-iptables=1
net.ipv4.neigh.default.gc_thresh1=4096
net.ipv4.neigh.default.gc_thresh2=6144
net.ipv4.neigh.default.gc_thresh3=8192
EOF
```

最后执行保存配置

```bash
sysctl -p
```

####7、ETCD集群容错表

建议在ETCD集群中使用`奇数`个成员,通过添加额外成员可以获得更高的失败容错。

| 集群大小 | MAJORITY | 失败容错 |
| :------: | :------: | :------: |
|    1     |    1     |    0     |
|    2     |    2     |    0     |
|    3     |    2     |    1     |
|    4     |    3     |    1     |
|    5     |    3     |    2     |
|    6     |    4     |    2     |
|    7     |    4     |    3     |
|    8     |    5     |    3     |
|    9     |    5     |    4     |

#### 8、Docker安装

参考[Docker安装](https://k8s.cloudcared.cn/article/Docker/Chapter3/3-1-install.html) 



####9、安装Rancher

要想在主机上安装Rancher,需要先登录到主机上，接着进行以下步骤:

通过shell工具(例如PuTTy或远程终端连接)登录到主机

```bash
docker run -d --restart=unless-stopped -p 80:80 -p 443:443 -v /data/rancher:/var/lib/rancher/ -v /root/var/log/auditlog:/var/log/auditlog -e CATTLE_SYSTEM_CATALOG=bundled -e AUDIT_LEVEL=3 rancher/rancher:stable
```

