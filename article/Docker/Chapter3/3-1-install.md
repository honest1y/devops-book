## 安装

#### step 1: 安装必要的一些系统工具
```bash
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

#### Step 2: 添加软件源信息
```bash
$ sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

#### Step 3: 更新并安装 Docker-CE
```bash
sudo yum makecache fast
sudo yum -y install docker-ce
```

#### Step 4: 开启Docker服务
```bash
$ sudo systemctl enable docker
$ sudo systemctl start docker
```

#### Step 5: 安装校验
```bash
[root@dts-test ~]# docker version
Client: Docker Engine - Community
 Version:           19.03.2
 API version:       1.40
 Go version:        go1.12.8
 Git commit:        6a30dfc
 Built:             Thu Aug 29 05:28:55 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.2
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.8
  Git commit:       6a30dfc
  Built:            Thu Aug 29 05:27:34 2019
  OS/Arch:          linux/amd64
  Experimental:     false
```

#### 配置加速器

针对Docker客户端版本大于 1.10.0 的用户

可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://su9ppkb0.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
