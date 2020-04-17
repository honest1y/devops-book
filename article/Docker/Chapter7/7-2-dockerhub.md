##Docker Hub

目前 Docker 官方维护了一个公共仓库 [Docker Hub](https://hub.docker.com/)。

大部分需求都可以通过在 Docker Hub 中直接下载镜像来实现。

#### 注册

在 [https://hub.docker.com](https://hub.docker.com/) 免费注册一个 Docker 账号。

####登录

登录需要输入用户名和密码，登录成功后，我们就可以从 docker hub 上拉取自己账号下的全部镜像。

```bash
[root@docker ~]# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: testuser
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

####退出

退出 docker hub 可以使用以下命令：

```bash
[root@docker ~]# docker logout
Removing login credentials for https://index.docker.io/v1/
```



#### 拉取镜像

你可以通过 docker search 命令来查找官方仓库中的镜像，并利用 docker pull 命令来将它下载到本地。

以 ubuntu 为关键词进行搜索：

```bash
[root@docker ~]# docker search ubuntu
NAME                                                      DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
ubuntu                                                    Ubuntu is a Debian-based Linux operating sys…   10693               [OK]                
dorowu/ubuntu-desktop-lxde-vnc                            Docker image to provide HTML5 VNC interface …   410                                     [OK]
rastasheep/ubuntu-sshd                                    Dockerized SSH service, built on top of offi…   245                                     [OK]
consol/ubuntu-xfce-vnc                                    Ubuntu container with "headless" VNC session…   212                                     [OK]
ubuntu-upstart                                            Upstart is an event-based replacement for th…   107                 [OK]                
ansible/ubuntu14.04-ansible                               Ubuntu 14.04 LTS with ansible                   98                                      [OK]
neurodebian                                               NeuroDebian provides neuroscience research s…   67                  [OK]                
1and1internet/ubuntu-16-nginx-php-phpmyadmin-mysql-5      ubuntu-16-nginx-php-phpmyadmin-mysql-5          50                                      [OK]
```

使用 docker pull 将官方 ubuntu 镜像下载到本地：

```bash
[root@docker ~]# docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
Digest: sha256:bec5a2727be7fff3d308193cfde3491f8fba1a2ba392b7546b43a051853a341d
Status: Image is up to date for ubuntu:latest
docker.io/library/ubuntu:latest
```

####推送镜像

用户登录后，可以通过 docker push 命令将自己的镜像推送到 Docker Hub。

以下命令中的 username 请替换为你的 Docker 账号用户名。

```bash
[root@docker ~]# docker tag ubuntu:18.04 username/ubuntu:18.04
[root@docker ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
rclone              v1                  6642ba8323a4        17 hours ago        399MB
test/ubuntu         v1.0                0bbbaf44fab0        18 hours ago        64.2MB
username/ubuntu     18.04               4e5021d210f6        10 days ago         64.2MB
ubuntu              18.04               4e5021d210f6        10 days ago         64.2MB
ubuntu              latest              4e5021d210f6        10 days ago         64.2MB
rclone/rclone       latest              077820eefe46        8 weeks ago         40.2MB
centos              7                   5e35e350aded        4 months ago        203MB
[root@docker ~]# docker push username/ubuntu:18.04
The push refers to repository [docker.io/username/ubuntu]
16542a8fc3be: Layer already exists 
6597da2e2e52: Layer already exists 
977183d4e999: Layer already exists 
c8be1b8f4d60: Layer already exists 
18.04: digest: sha256:e5dd9dbb37df5b003359f6f126958c9c928f937bec69836320 size: 1152
```

