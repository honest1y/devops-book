##数据卷

数据卷是一个可供一个或多个容器使用的特殊目录，它绕过 UFS，可以提供很多有用的特性：

-  可以在容器之间共享和重用
- 对 数据卷的修改会立马生效
- 对 数据卷 的更新，不会影响镜像
- 数据卷 默认会一直存在，即使容器被删除

####创建一个数据卷

```bash
[root@docker ~]# docker volume create my-vol
my-vol
```

####查看所有的 数据卷

```bash
[root@docker ~]# docker volume ls
DRIVER              VOLUME NAME
local               9186dbebe36c0273686adcf4519a2e99706546d0425f4f9999f61d14aaaf566e
local               my-vol
```

#### 查看指定卷的详情信息

```bash
[root@docker ~]# docker volume inspect my-vol
[
    {
        "CreatedAt": "2020-03-31T10:24:06+08:00",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
        "Name": "my-vol",
        "Options": {},
        "Scope": "local"
    }
]
```

####启动一个挂载数据卷的容器

在用 `docker run` 命令的时候，使用 `--mount` 标记来将 `数据卷` 挂载到容器里。在一次 `docker run` 中可以挂载多个 `数据卷`。

下面创建一个名为 `web` 的容器，并加载一个 `数据卷` 到容器的 `/webapp` 目录。

```bash
[root@docker ~]# docker run -d -P \
> --name web \
> --mount source=my-vol,target=/webapp \
> training/webapp \
> python app.py
```

数据卷信息在 "Mounts" Key 下面

```bash
[root@docker ~]# docker inspect web
 "Mounts": [
            {
                "Type": "volume",
                "Name": "my-vol",
                "Source": "/var/lib/docker/volumes/my-vol/_data",
                "Destination": "/webapp",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
        ],
```

####删除数据卷

```bash
[root@docker ~]# docker volume rm my-vol
my-vol
```

数据卷 是被设计用来持久化数据的，它的生命周期独立于容器，Docker 不会在容器被删除后自动删除 `数据卷`，并且也不存在垃圾回收这样的机制来处理没有任何容器引用的 `数据卷`。如果需要在删除容器的同时移除数据卷。可以在删除容器的时候使用 `docker rm -v` 这个命令。

无主的数据卷可能会占据很多空间，要清理请使用以下命令

```bash
$ docker volume prune
```