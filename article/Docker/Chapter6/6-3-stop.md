## 终止容器

可以使用 `docker stop` 来终止一个运行中的容器。

```bash
[root@docker ~]# docker stop 27ee12a35471
27ee12a35471
```

此外，当 Docker 容器中指定的应用终结时，容器也自动终止。

例如对于上一章节中只启动了一个终端的容器，用户通过 `exit` 命令或 `Ctrl+d` 来退出终端时，所创建的容器立刻终止。

终止状态的容器可以用 `docker ps -a` 命令看到。例如

```bash
[root@docker ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                       PORTS               NAMES
27ee12a35471        ubuntu:18.04        "/bin/sh -c 'while t…"   37 seconds ago      Exited (137) 8 seconds ago                       competent_albattani                         backstabbing_pike
```

处于终止状态的容器，可以通过 `docker start` 命令来重新启动。

此外，`docker restart` 命令会将一个运行态的容器终止，然后再重新启动它。

