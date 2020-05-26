## etcd简介 + 使用 + 部署

### 1、etcd简介

etcd是CoreOS团队于2013年6月发起的开源项目，它的目标是构建一个高可用的分布式键值(key-value)数据库。etcd内部采用`raft`协议作为一致性算法，etcd基于Go语言实现。



### 2、etcd特点

- 简单：基于HTTP+JSON的API让你用curl就可以轻松使用。
- 安全：可选SSL客户认证机制。
- 快速：每个实例每秒支持一千次写操作。
- 可信：使用Raft算法充分实现了分布式。



### 3、etcd使用场景

#### 场景一 服务发现（Service Discovery）

服务发现就是想要了解集群中是否有进程在监听udp或tcp端口，并且通过名字就可以查找和连接。

1. **一个强一致性、高可用的服务存储目录**。基于Raft算法的etcd天生就是这样一个强一致性高可用的服务存储目录。
2. **一种注册服务和监控服务健康状态的机制**。用户可以在etcd中注册服务，并且对注册的服务设置`key TTL`，定时保持服务的心跳以达到监控健康状态的效果。
3. **一种查找和连接服务的机制**。通过在etcd指定的主题下注册的服务也能在对应的主题下查找到。为了确保连接，我们可以在每个服务机器上都部署一个Proxy模式的etcd，这样就可以确保能访问etcd集群的服务都能互相连接。

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/05/0129001.jpg)

- **微服务协同工作架构中，服务动态添加**。

  通过服务发现机制，在etcd中注册某个服务名字的目录，在该目录下存储可用的服务节点的IP。在使用服务的过程中，只要从服务目录下查找可用的服务节点去使用即可。

![img](https://cdn.cloudcared.cn/wp-content/uploads/2020/05/0129002.jpg)

- **PaaS平台中应用多实例与实例故障重启透明化**。

  某个实例随时都有可能故障重启，这时就需要动态的配置域名解析（路由）中的信息。通过etcd的服务发现功能就可以轻松解决这个动态配置的问题。

![img](https://cdn.cloudcared.cn/wp-content/uploads/2020/05/0129003.jpg)



#### 场景二：消息发布与订阅

在分布式系统中，构建一个配置共享中心，数据提供者在这个配置中心发布消息，而消息使用者则订阅他们关心的主题，一旦主题有消息发布，就会实时通知订阅者。通过这种方式可以做到分布式系统配置的集中式管理与动态更新。

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/05/0129004.jpg)



#### 场景三：负载均衡

为了保证服务的高可用以及数据的一致性，通常都会把数据和服务部署多份，以此达到对等服务，即使其中的某一个服务失效了，也不影响使用。由此带来的坏处是数据写入性能下降，而好处则是数据访问时的负载均衡。因为每个对等服务节点上都存有完整的数据，所以用户的访问流量就可以分流到不同的机器上。

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/05/0129005.jpg)



#### 场景四：分布式通知与协调

不同系统都在etcd上对同一个目录进行注册，同时设置Watcher观测该目录的变化（如果对子目录的变化也有需要，可以设置递归模式），当某个系统更新了etcd的目录，那么设置了Watcher的系统就会收到通知，并作出相应处理。

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/05/0129006.jpg)



#### 场景五：分布式锁

保证在多个节点同时去创建某个目录时，只有一个成功。而创建成功的用户就可以认为是获得了锁。

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/05/0130000.jpg)



#### 场景六：分布式队列

在/queue这个目录中另外建立一个/queue/condition节点，condition可以**表示队列大小**。比如一个大的任务需要很多小任务就绪的情况下才能执行，每次有一个小任务就绪，就给这个condition数字加1，直到达到大任务规定的数字，再开始执行队列里的一系列小任务，最终执行大任务。

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/05/0129008.jpg)



#### 场景七：集群监控与Leader竞选

通过etcd来进行监控实现起来非常简单并且实时性强。

1. 前面几个场景已经提到Watcher机制，当某个节点消失或有变动时，Watcher会第一时间发现并告知用户。
2. 节点可以设置`TTL key`，比如每隔30s发送一次心跳使代表该机器存活的节点继续存在，否则节点消失。

这样就可以第一时间检测到各节点的健康状态，以完成集群的监控要求。

另外，使用分布式锁，可以完成Leader竞选。这种场景通常是一些长时间CPU计算或者使用IO操作的机器，只需要竞选出的Leader计算或处理一次，就可以把结果复制给其他的Follower。从而避免重复劳动，节省计算资源。

这个的经典场景是**搜索系统中建立全量索引**。如果每个机器都进行一遍索引的建立，不但耗时而且建立索引的一致性不能保证。通过在etcd的CAS机制同时创建一个节点，创建成功的机器作为Leader，进行索引计算，然后把计算结果分发到其它节点。

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/05/0129009.jpg)



### 4、etcd单机部署



安装epel源

```bash
wget http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -ivh epel-release-latest-7.noarch.rpm
```

安装etcd

```bash
yum install etcd -y
```

配置hosts

```
vim /etc/hosts
172.20.2.10 etcd-0
```

配置etcd文件

```
[root@etcd-0 ~]# grep -Ev "^#|^$" /etc/etcd/etcd.conf
ETCD_DATA_DIR="/var/lib/etcd/etcd-cluster"
ETCD_LISTEN_PEER_URLS="http://localhost:2380"
ETCD_LISTEN_CLIENT_URLS="http://localhost:2379"
ETCD_NAME="etcd-0"
ETCD_ADVERTISE_CLIENT_URLS="http://localhost:2379"
```

启动/查看etcd

```bash
systemctl restart etcd
systemctl status etcd 
```

查看etcd数据目录

```
[root@etcd-0 ~]# tree /var/lib/etcd/etcd-cluster/
/var/lib/etcd/etcd-cluster/
└── member
    ├── snap
    │   └── db
    └── wal
        ├── 0000000000000000-0000000000000000.wal
        └── 0.tmp

3 directories, 3 files
```

etcd单机搭建完成



### etcd集群搭建

环境

| IP地址      | 节点角色 | CPU  | 内存 |
| ----------- | -------- | ---- | ---- |
| 172.20.2.10 | etcd-0   | 4    | 4    |
| 172.20.2.11 | etcd-1   | 4    | 4    |
| 172.20.2.12 | etcd-2   | 4    | 4    |

安装epel源

```bash
wget http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -ivh epel-release-latest-7.noarch.rpm
```

安装etcd

```bash
yum install etcd -y
```

配置hosts

```bash
vim /etc/hosts
  172.20.2.10 etcd-0
  172.20.2.11 etcd-1
  172.20.2.12 etcd-2
```

修改配置

**172.20.2.10配置**

```
[root@etcd-0 ~]# grep -Ev "^#" /etc/etcd/etcd.conf
ETCD_DATA_DIR="/var/lib/etcd/etcd-cluster"
ETCD_LISTEN_PEER_URLS="http://172.20.2.10:2380"
ETCD_LISTEN_CLIENT_URLS="http://127.0.0.1:2379,http://172.20.2.10:2379"
ETCD_NAME="etcd-0"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://172.20.2.10:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://127.0.0.1:2379,http://172.20.2.10:2379"
ETCD_INITIAL_CLUSTER="etcd-0=http://172.20.2.10:2380,etcd-1=http://172.20.2.11:2380,etcd-2=http://172.20.2.12:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
```



**172.20.2.11配置**

```
[root@etcd-1 ~]# grep -Ev "^#" /etc/etcd/etcd.conf
ETCD_DATA_DIR="/var/lib/etcd/etcd-cluster"
ETCD_LISTEN_PEER_URLS="http://172.20.2.11:2380"
ETCD_LISTEN_CLIENT_URLS="http://127.0.0.1:2379,http://172.20.2.11:2379"
ETCD_NAME="etcd-1"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://172.20.2.11:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://127.0.0.1:2379,http://172.20.2.11:2379"
ETCD_INITIAL_CLUSTER="etcd-0=http://172.20.2.10:2380,etcd-1=http://172.20.2.11:2380,etcd-2=http://172.20.2.12:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
```



**172.20.2.12配置**

```
[root@etcd-2 ~]# grep -Ev "^#" /etc/etcd/etcd.conf
ETCD_DATA_DIR="/var/lib/etcd/etcd-cluster"
ETCD_LISTEN_PEER_URLS="http://172.20.2.12:2380"
ETCD_LISTEN_CLIENT_URLS="http://127.0.0.1:2379,http://172.20.2.12:2379"
ETCD_NAME="etcd-2"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://172.20.2.12:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://127.0.0.1:2379,http://172.20.2.12:2379"
ETCD_INITIAL_CLUSTER="etcd-0=http://172.20.2.10:2380,etcd-1=http://172.20.2.11:2380,etcd-2=http://172.20.2.12:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
```



查看etcd集群节点

```
[root@etcd-0 ~]# etcdctl member list
1ab5ad355dec8053: name=etcd-2 peerURLs=http://172.20.2.12:2380 clientURLs=http://127.0.0.1:2379,http://172.20.2.12:2379 isLeader=true
9e60b949250d7a55: name=etcd-1 peerURLs=http://172.20.2.11:2380 clientURLs=http://127.0.0.1:2379,http://172.20.2.11:2379 isLeader=false
dd92788c60397cc1: name=etcd-0 peerURLs=http://172.20.2.10:2380 clientURLs=http://127.0.0.1:2379,http://172.20.2.10:2379 isLeader=false
```



查看etcd集群状态

```
[root@etcd-0 ~]# etcdctl cluster-health
member 1ab5ad355dec8053 is healthy: got healthy result from http://127.0.0.1:2379
member 9e60b949250d7a55 is healthy: got healthy result from http://127.0.0.1:2379
member dd92788c60397cc1 is healthy: got healthy result from http://127.0.0.1:2379
cluster is healthy
```



测试:

```
[root@etcd-0 etcd-cluster]# etcdctl mkdir testdir
[root@etcd-0 etcd-cluster]# etcdctl ls
/testdir
[root@etcd-1 ~]# etcdctl ls
/testdir
[root@etcd-2 ~]# etcdctl ls
/testdir
```

集群搭建完毕

### etcd使用入门

etcdctl是一个命令行客户端，它能提供一些简洁的命令，供用户直接跟etcd服务打交道，而无需基于 HTTP API方式。可以方便我们在对服务进行测试或者手动修改数据库内容。建议刚刚接触etcd时通过etdctl来熟悉相关操作。这些操作跟HTTP API基本上是对应的。

etcd项目二进制发行包中已经包含了etcdctl工具，etcdctl支持的命令大体上分为数据库操作和非数据库操作两类。

```
$ etcd --version
etcd Version: 3.3.11
Git SHA: 2cf9e51
Go Version: go1.10.3
Go OS/Arch: linux/amd64

$ etcdctl -h
NAME:
   etcdctl - A simple command line client for etcd.

WARNING:
   Environment variable ETCDCTL_API is not set; defaults to etcdctl v2.
   Set environment variable ETCDCTL_API=3 to use v3 API or ETCDCTL_API=2 to use v2 API.

USAGE:
   etcdctl [global options] command [command options] [arguments...]
   
VERSION:
   3.3.11
   
COMMANDS:
     backup          backup an etcd directory
     cluster-health  check the health of the etcd cluster
     mk              make a new key with a given value
     mkdir           make a new directory
     rm              remove a key or a directory
     rmdir           removes the key if it is an empty directory or a key-value pair
     get             retrieve the value of a key
     ls              retrieve a directory
     set             set the value of a key
     setdir          create a new directory or update an existing directory TTL
     update          update an existing key with a given value
     updatedir       update an existing directory
     watch           watch a key for changes
     exec-watch      watch a key for changes and exec an executable
     member          member add, remove and list subcommands
     user            user add, grant and revoke subcommands
     role            role add, grant and revoke subcommands
     auth            overall auth controls
     help, h         Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --debug                          output cURL commands which can be used to reproduce the request
   --no-sync                        don't synchronize cluster information before sending request
   --output simple, -o simple       output response in the given format (simple, `extended` or `json`) (default: "simple")
   --discovery-srv value, -D value  domain name to query for SRV records describing cluster endpoints
   --insecure-discovery             accept insecure SRV records describing cluster endpoints
   --peers value, -C value          DEPRECATED - "--endpoints" should be used instead
   --endpoint value                 DEPRECATED - "--endpoints" should be used instead
   --endpoints value                a comma-delimited list of machine addresses in the cluster (default: "http://127.0.0.1:2379,http://127.0.0.1:4001")
   --cert-file value                identify HTTPS client using this SSL certificate file
   --key-file value                 identify HTTPS client using this SSL key file
   --ca-file value                  verify certificates of HTTPS-enabled servers using this CA bundle
   --username value, -u value       provide username[:password] and prompt if password is not supplied.
   --timeout value                  connection timeout per request (default: 2s)
   --total-timeout value            timeout for the command execution (except watch) (default: 5s)
   --help, -h                       show help
   --version, -v                    print the version
```

常用命令选项：

```
--debug 输出CURL命令，显示执行命令的时候发起的请求
--no-sync 发出请求之前不同步集群信息
--output, -o 'simple' 输出内容的格式(simple 为原始信息，json 为进行json格式解码，易读性好一些)
--peers, -C 指定集群中的同伴信息，用逗号隔开(默认为: "127.0.0.1:4001")
--cert-file HTTPS下客户端使用的SSL证书文件
--key-file HTTPS下客户端使用的SSL密钥文件
--ca-file 服务端使用HTTPS时，使用CA文件进行验证
--help, -h 显示帮助命令信息
--version, -v 打印版本信息
```



- set

指定某个键的值。例如：

```
[root@etcd-0 ~]# etcdctl set /testdir/testkey "Hello world"
Hello world
```



- get

获取指定键的值。例如：

```
[root@etcd-0 ~]# etcdctl get /testdir/testkey
Hello world
```

当键不存在时，则会报错。例如：

```
[root@etcd-0 ~]# etcdctl get /testdir/testkey2
Error:  100: Key not found (/testdir/testkey2) [11]
```



- update

当键存在时，更新值内容。例如：

```
[root@etcd-0 ~]# etcdctl update /testdir/testkey "Hello"
Hello
```



- rm

删除某个键值。例如:

```
[root@etcd-0 ~]# etcdctl rm /testdir/testkey
PrevNode.Value: Hello
```



- mk

如果给定的键不存在，则创建一个新的键值。例如:

```
[root@etcd-0 ~]# etcdctl mk /testdir/testkey "Hello world"
Hello world
```

当键存在的时候，执行该命令会报错，例如:

```
[root@etcd-0 ~]# etcdctl mk /testdir/testkey "Hello world"
Error:  105: Key already exists (/testdir/testkey) [8]
```



- mkdir

如果给定的键目录不存在，则创建一个新的键目录。例如：

```
[root@etcd-0 ~]# etcdctl mkdir testdir2
```

当键目录存在的时候，执行该命令会报错，例如：

```
[root@etcd-0 ~]# etcdctl mkdir testdir2
Error:  105: Key already exists (/testdir2) [9]
```



- setdir

创建一个键目录。如果目录不存在就创建，如果目录存在更新目录TTL。

```
[root@etcd-0 ~]# etcdctl setdir testdir3
```



- updatedir

更新一个已经存在的目录。

```
[root@etcd-0 ~]# etcdctl updatedir testdir2
```



- rmdir

删除一个空目录，或者键值对。

```
[root@etcd-0 ~]# etcdctl setdir dir1
[root@etcd-0 ~]# etcdctl rmdir dir1
```

若目录不空，会报错:

```
[root@etcd-0 ~]# etcdctl set /dir/testkey hi
hi
[root@etcd-0 ~]# etcdctl rmdir /dir
Error:  108: Directory not empty (/dir) [17]
```



- ls

列出目录(默认为根目录)下的键或者子目录，默认不显示子目录中内容。

例如：

```
[root@etcd-0 ~]# etcdctl ls
/testdir
/testdir2
/dir

[root@etcd-0 ~]# etcdctl ls dir
/dir/testkey
```



**非数据库操作**

- backup

备份etcd的数据。

```
[root@etcd-0 ~]# etcdctl backup --data-dir /var/lib/etcd  --backup-dir /home/etcd_backup
```



- watch

监测一个键值的变化，一旦键值发生更新，就会输出最新的值并退出。

例如：用户更新testkey键值为Hello watch。



```
[root@etcd-0 ~]# etcdctl get /testdir/testkey
Hello world
[root@etcd-0 ~]# etcdctl set /testdir/testkey "Hello watch"
Hello watch
[root@etcd-0 ~]# etcdctl watch testdir/testkey
Hello watch
```



- exec-watch

监测一个键值的变化，一旦键值发生更新，就执行给定命令。

例如：用户更新testkey键值。

```
[root@etcd-0 ~]# etcdctl exec-watch testdir/testkey -- sh -c 'ls'
config  Documentation  etcd  etcdctl  README-etcdctl.md  README.md  READMEv2-etcdctl.md
```



- member

通过`list`、`add`、`remove`命令列出、添加、删除etcd实例到etcd集群中。

查看集群中存在的节点

```
[root@etcd-0 ~]# etcdctl member list
1ab5ad355dec8053: name=etcd-2 peerURLs=http://172.20.2.12:2380 clientURLs=http://127.0.0.1:2379,http://172.20.2.12:2379 isLeader=true
9e60b949250d7a55: name=etcd-1 peerURLs=http://172.20.2.11:2380 clientURLs=http://127.0.0.1:2379,http://172.20.2.11:2379 isLeader=false
dd92788c60397cc1: name=etcd-0 peerURLs=http://172.20.2.10:2380 clientURLs=http://127.0.0.1:2379,http://172.20.2.10:2379 isLeader=false
```

删除集群中存在的节点

```csharp
[root@etcd-0 ~]# etcdctl member remove 1ab5ad355dec8053
Removed member 1ab5ad355dec8053 from cluster
```

向集群中新加节点

```csharp
[root@etcd-0 ~]# etcdctl member add etcd-2 http://172.20.2.12:2380
Added member named etcd-2 with ID 37aebb4d126502d2 to cluster
```

