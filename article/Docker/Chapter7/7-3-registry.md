## 私有仓库

有时候自己部门内部有一些镜像要共享时，如果直接导出镜像拿给别人又比较麻烦，使用像Docker Hub这样的公共仓库又不是很方便，这时候我们可以自己搭建属于自己的私有仓库服务，用于存储和分布我们的镜像。

Docker官方提供了registry这个镜像，可以用于搭建私有仓库服务，我们把镜像拉到本地之后，用下面命令创建该镜像的容器便可以搭建一个仓库服务，如下：

```bash
[root@docker ~]# docker run -d -p 5000:5000 --restart=always --name registry registry
Unable to find image 'registry:latest' locally
latest: Pulling from library/registry
486039affc0a: Pull complete 
ba51a3b098e6: Pull complete 
8bb4c43d6c8e: Pull complete 
6f5f453e5f2d: Pull complete 
42bc10b72f42: Pull complete 
Digest: sha256:7d081088e4bfd632a88e3f3bcd9e007ef44a796fddfe3261407a3f9f04abe1e7
Status: Downloaded newer image for registry:latest
d6fb06b351e82ad6fd16a41d4a5b8969b016cd237732d22addf7beabe82e8c9e
```

#### 推送镜像

```bash
[root@docker ~]# docker tag ubuntu:18.04 127.0.0.1:5000/ubuntu:v1 
[root@docker ~]# docker push 127.0.0.1:5000/ubuntu:v1            
The push refers to repository [127.0.0.1:5000/ubuntu]
16542a8fc3be: Pushed 
6597da2e2e52: Pushed 
977183d4e999: Pushed 
c8be1b8f4d60: Pushed 
v1: digest: sha256:e5dd9dbb37df5b731a6688fa49f4003359f6f126958c9c928f937bec69836320 size: 1152
```

#### 拉取镜像

```bash
[root@docker ~]# docker pull 127.0.0.1:5000/ubuntu:v1
v1: Pulling from ubuntu
5bed26d33875: Pull complete 
f11b29a9c730: Pull complete 
930bda195c84: Pull complete 
78bf9a5ad49e: Pull complete 
Digest: sha256:e5dd9dbb37df5b731a6688fa49f4003359f6f126958c9c928f937bec69836320
Status: Downloaded newer image for 127.0.0.1:5000/ubuntu:v1
127.0.0.1:5000/ubuntu:v1
```

