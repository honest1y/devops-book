## 导出和导入容器

#### 导出容器

如果要导出本地某个容器，可以使用 `docker export` 命令。

```bash
[root@docker ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                    PORTS               NAMES
c0b2edc0c5ee        ubuntu:18.04        "/bin/bash"         36 hours ago        Exited (0) 21 hours ago                       test
[root@docker ~]# docker export c0b2edc0c5ee > ubuntu.tar
```

这样将导出容器快照到本地文件。

#### 导入容器快照

可以使用 `docker import` 从容器快照文件中再导入为镜像，例如

```bash
[root@docker ~]# cat ubuntu.tar | docker import - test/ubuntu:v1.0
sha256:0bbbaf44fab08a3302ff00199176f1d1d5ecc0a16d6e01ea203c281a82095298

[root@docker ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
test/ubuntu         v1.0                0bbbaf44fab0        2 seconds ago       64.2MB
```

此外，也可以通过指定 URL 或者某个目录来导入，例如

```bash
$ docker import http://example.com/exampleimage.tgz example/imagerepo
```