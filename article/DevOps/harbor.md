# CentOS安装Harbor-v1.10.1

Harbor是一个开源的可信云本地注册表项目，用于存储、签名和扫描内容。Harbor扩展了开源Docker发行版，增加了用户通常需要的功能，比如安全性、身份和管理。
Harbor经常作为Docker私有云端仓库被企业使用。

## 一、安装docker-compose

Harbor是通过docker-compose来管理镜像的。
所以在Harbor主机安装docker-compose是必须的首要的一步。

```
[root@etcd-0 ~]# curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
[root@etcd-0 ~]# chmod +x /usr/local/bin/docker-compose
[root@etcd-0 ~]# ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
[root@etcd-0 ~]# docker-compose --version
docker-compose version 1.25.4, build 8d51620a
```

## 二、Harbor的域名

如果没有域名的话，可以自己定义一个域名，并在Harbor主机和Docker主机通过向`/etc/hosts`文件添加条目完成自定义域名与Harbor主机IP的映射关系。本文中自定义的域名是`harbor.cn`，配置如下：

```
[root@etcd-0 ~]# cat /etc/hosts | grep harbor
172.20.2.10 etcd-0 harbor.cn
```

## 三、生成自签证书

Docker默认通过HTTPS与Harbor通信的，虽然可以改为HTTP方式，但需要修改的配置项会很多，而且也不安全。

```
[root@etcd-0 ~]# mkdir /home/cert
[root@etcd-0 ~]# cd /home/cert
```

Step1 - 生成根证书私钥（无加密）：

```
openssl genrsa -out ca.key 4096
```

Step2 - 生成自签名证书（使用已有私钥ca.key自行签发根证书）生成ca.crt：

**添加-subj参数可以免去交互过程**

```
openssl req -x509 -new -nodes -sha512 -days 3650 \
    -subj "/C=CN/ST=Beijing/L=Beijing/O=ccx/OU=plat/CN=172.20.2.10" \
    -key ca.key \
    -out ca.crt
```

Step3 - 生成服务器端自己域名的key：

```
openssl genrsa -out harbor.cn.key 4096
```


Step4 - 生成服务器端自己域名的CSR签名请求：

```
openssl req -sha512 -new \
    -subj "/C=CN/ST=Beijing/L=Beijing/O=ccx/OU=plat/CN=172.20.2.10" \
    -key harbor.cn.key \
    -out harbor.cn.csr
```

Step5 - 生成一个 openssl 命令需要的外部配置文件 `externalfile.ext`。
这个文件可以随意命名，但是要记住，后面对的命令还要用到。
文件内容中主要是**subjectAltName**这一项
如果配IP就写`IP.1=172.20.2.10`
如果配域名就写 `DNS.1=xxx.xxx.com`

```
[root@etcd-0 ~]# cat externalfile.ext
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth 
subjectAltName = @alt_names
[alt_names]
DNS.1=harbor.cn
```

Step6 - 通过外部配置文件 `externalfile.ext` 和 csr 生成 crt：

```
openssl x509 -req -sha512 -days 3650 -extfile externalfile.ext \
    -CA ca.crt \
    -CAkey ca.key \
    -CAcreateserial \
    -in harbor.cn.csr \
    -out harbor.cn.crt
```


Step7 - 将服务端的 crt 转换成客户端用的 cert：

```
openssl x509 -inform PEM -in harbor.cn.crt -out harbor.cn.cert
```


至此，所有证书文件就创建好了

```
[root@etcd-0 cert]# ll
total 32
-rw-r--r-- 1 root root 2004 May 27 10:40 ca.crt
-rw-r--r-- 1 root root 3243 May 27 10:39 ca.key
-rw-r--r-- 1 root root   17 May 27 10:41 ca.srl
-rw-r--r-- 1 root root  228 May 27 10:41 externalfile.ext
-rw-r--r-- 1 root root 2033 May 27 10:44 harbor.cn.cert
-rw-r--r-- 1 root root 2033 May 27 10:41 harbor.cn.crt
-rw-r--r-- 1 root root 1691 May 27 10:40 harbor.cn.csr
-rw-r--r-- 1 root root 3243 May 27 10:40 harbor.cn.key
```

## 四、为各个docker客户端分发证书

将Harbor主机上带域名的`.cert`和`.key`证书文件拷贝到docker客户端所在主机的`/etc/docker/certs.d/xxx.xxx.com/`目录下。
下面以`172.20.2.11`这台docker客户端主机上的操作为例进行介绍。

Step1 - 在Docker主机上执行：

```
[root@etcd-0 docker]# mkdir -p /etc/docker/certs.d/harbor.cn/
```

Step2、在Harbor主机，执行：

```
[root@etcd-0 docker]# scp ./harbor.cn.cert ./harbor.cn.key root@172.20.2.11:/etc/docker/certs.d/harbor.cn/
```

Step3、在Docker主机修改 `/etc/docker/daemon.json`，主要是增加"insecure-registries":["http://harbor.cn"] ：

```
[root@etcd-0 docker]# cat /etc/docker/daemon.json 
{
  "registry-mirrors": ["https://*****.mirror.aliyuncs.com"],
  "insecure-registries":["http://harbor.cn"]
}
```


Step4、重启Docker：

```
[root@etcd-0 docker]# systemctl daemon-reload
[root@etcd-0 docker]# systemctl restart docker
```

## 五、安装Harbor

下载&解压：

```
wget https://github.com/goharbor/harbor/releases/download/v1.10.1/harbor-offline-installer-v1.10.1.tgz
mkdir -p /home/harbor
tar -zxvf harbor-offline-installer-v1.10.1.tgz -C /home/harbor/
```

```
[root@etcd-0 harbor]# ll
total 662128
drwxr-xr-x 3 root root        20 Jun  1 11:14 common
-rw-r--r-- 1 root root      3398 Feb 10 14:18 common.sh
-rw-r--r-- 1 root root      5344 Jun  1 11:17 docker-compose.yml
-rw-r--r-- 1 root root 677974489 Feb 10 14:19 harbor.v1.10.1.tar.gz
-rw-r--r-- 1 root root      5879 Jun  1 11:13 harbor.yml
-rwxr-xr-x 1 root root      2284 Feb 10 14:18 install.sh
-rw-r--r-- 1 root root     11347 Feb 10 14:18 LICENSE
-rwxr-xr-x 1 root root      1749 Feb 10 14:18 prepare
```

修改配置文件`harbor.yml`：

```
hostname: harbor.cn
http:
	port: 80
https:
	port: 443
	certificate: /home/cert/harbor.cn.crt # 这里是证书信息
	private_key: /home/cert/harbor.cn.key # 这里是证书信息
harbor_admin_password: Harbor12345
database:
	password: root123
data_volumn: /data
log:
    level: info
    location: /var/log/harbor # harbor日志存放路径
```

更新参数：

```
./prepare
```

进行安装：

```
[root@etcd-0 harbor]# ./install.sh 

[Step 0]: checking if docker is installed ...

Note: docker version: 19.03.9

[Step 1]: checking docker-compose is installed ...

Note: docker-compose version: 1.25.4

[Step 2]: loading Harbor images ...
Loaded image: goharbor/harbor-core:v1.10.1
a2ffdaaa3434: Loading layer [==================================================>]  63.56MB/63.56MB
5745ac9e0297: Loading layer [==================================================>]  54.44MB/54.44MB
dc1d24cbb1d5: Loading layer [==================================================>]  5.632kB/5.632kB
4280f2f98340: Loading layer [==================================================>]  2.048kB/2.048kB
0f9279b20eec: Loading layer [==================================================>]   2.56kB/2.56kB
8b9162d25131: Loading layer [==================================================>]   2.56kB/2.56kB
50ad7481e5af: Loading layer [==================================================>]   2.56kB/2.56kB
54b809bfb5ec: Loading layer [==================================================>]  10.24kB/10.24kB
Loaded image: goharbor/harbor-db:v1.10.1


[Step 3]: preparing environment ...

[Step 4]: preparing harbor configs ...
prepare base dir is set to /home/harbor/harbor
Clearing the configuration file: /config/log/logrotate.conf
Clearing the configuration file: /config/log/rsyslog_docker.conf
...
Generated configuration file: /config/jobservice/env
Generated configuration file: /config/jobservice/config.yml
loaded secret from file: /secret/keys/secretkey
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir



[Step 5]: starting Harbor ...
Creating network "harbor_harbor" with the default driver
Creating harbor-log ... done
Creating registryctl   ... done
Creating redis         ... done
Creating harbor-db     ... done
Creating registry      ... done
Creating harbor-portal ... done
Creating harbor-core   ... done
Creating harbor-jobservice ... done
Creating nginx             ... done
✔ ----Harbor has been installed and started successfully.----
```

## 六、使用Harbor

### 6.1、访问Harbor WebUI

使用浏览器，通过`https://域名`或`https://ip:port`两种方式都可以访问Harbor的WebUI。
因为是自签CA证书，浏览器会拦截，需要添加信任即可。

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/06/harbor.png)

### 6.2、push镜像

要想将镜像push到Harbor仓库中，必须先要在Harbor中创建自己的项目，即project，当然也可以使用Harbor自带的项目：library
下面看看如何做才能吧nginx镜像推送到Harbor镜像中去。

Step1、docker拉取一个镜像并修改tag：

```
docker pull nginx
docker tag nginx:latest harbor.cn/library/nginx:latest
```


Step2、docker login 登录Harbor：

```
docker login -u<harbor_user_name> -p<harbor_password> <harbor_domain>
```


执行命令，及输出：

```
[root@etcd-1 ~]# docker login -uadmin -pHarbor12345 harbor.cn
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```


当看到Login Succeeded时，就说明登陆成功了。

下面可以查看docker中保存的登录信息：

```
[root@etcd-1 ~]# cat ~/.docker/config.json 
{
	"auths": {
		"harbor.cn": {
			"auth": "YWRtaW46SGFyYm9yMTIzNDU="
		}
	},
	"HttpHeaders": {
		"User-Agent": "Docker-Client/19.03.9 (linux)"
	}
}
```



Step3、docker推送镜像到Harbor：

```
docker push <harbor_domain>/<project_name>/<image_name>:<image_tag>
```


执行命令，及输出：

```
[root@etcd-1 ~]# docker push harbor.cn/library/nginx:latest
The push refers to repository [harbor.cn/library/nginx]
6c7de695ede3: Pushed 
2f4accd375d9: Pushed 
ffc9b21953f4: Pushed 
latest: digest: sha256:8269a7352a7dad1f8b3dc83284f195bac72027dd50279422d363d49311ab7d9b size: 948
```


Step4、登录Harbor查看镜像

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/06/image-harbor.png)

### 6.3、pull镜像： docker <== harbor
Docker想从Harbor拉取镜像，只需要：

```
docker login 登录harbor
docker pull时，在镜像名称前加上Harbor的域名，就像这样：
docker pull harbor.cn/library/nginx:latest
```

## 七、维护时常用命令
**查看harbor**

```
[root@etcd-0 harbor]# cd /home/harbor/harbor
[root@etcd-0 harbor]# docker-compose ps
      Name                     Command                  State                          Ports                   
---------------------------------------------------------------------------------------------------------------
harbor-core         /harbor/harbor_core              Up (healthy)                                              
harbor-db           /docker-entrypoint.sh            Up (healthy)   5432/tcp                                   
harbor-jobservice   /harbor/harbor_jobservice  ...   Up (healthy)                                              
harbor-log          /bin/sh -c /usr/local/bin/ ...   Up (healthy)   127.0.0.1:1514->10514/tcp                  
harbor-portal       nginx -g daemon off;             Up (healthy)   8080/tcp                                   
nginx               nginx -g daemon off;             Up (healthy)   0.0.0.0:80->8080/tcp, 0.0.0.0:443->8443/tcp
redis               redis-server /etc/redis.conf     Up (healthy)   6379/tcp                                   
registry            /home/harbor/entrypoint.sh       Up (healthy)   5000/tcp                                   
registryctl         /home/harbor/start.sh            Exit 137                          
```



**停止&开启命令**

```
docker-compose stop
docker-compose start
```

**修改harbor配置**

```
docker-compose down -v
vim harbor.yml
./prepare
docker-compose up -d
```

**删除harbors的镜像保留数据库和镜像数据**

```
docker-compose down -v
```


**删除harbor的数据库和数据，相当于重装**

```
docker-compose down -v
```