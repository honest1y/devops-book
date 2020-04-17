## 进入容器

在使用 `-d` 参数时，容器启动后会进入后台。

某些时候需要进入容器进行操作，包括使用 `docker attach` 命令或 `docker exec` 命令，推荐大家使用 `docker exec` 命令，原因会在下面说明。

#### attach命令

下面示例如何使用 `docker attach` 命令。

```bash
[root@docker ~]# docker run -dit ubuntu:18.04
d1e04dbec84bd378b3223dcb22a29ed2f57fe226d0a140e2fc5b94b7391ad298

[root@docker ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
d1e04dbec84b        ubuntu:18.04        "/bin/bash"         7 seconds ago       Up 6 seconds                            friendly_proskuriakova

[root@docker ~]# docker attach d1e04dbec84b
root@d1e04dbec84b:/# 
```

*注意：* 如果从这个 stdin 中 exit，会导致容器的停止。

#### exec 命令

`-i -t 参数`

`docker exec` 后边可以跟多个参数，这里主要说明 `-i` `-t` 参数。

只用 `-i` 参数时，由于没有分配伪终端，界面没有我们熟悉的 Linux 命令提示符，但命令执行结果仍然可以返回。

当 `-i` `-t` 参数一起使用时，则可以看到我们熟悉的 Linux 命令提示符。

```bash
[root@docker ~]# docker run -dit ubuntu:18.04
c0b2edc0c5eeb38866e8ca1555e86c3bd0d4e17a93f7f9de497434a13aa14371

[root@docker ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
c0b2edc0c5ee        ubuntu:18.04        "/bin/bash"         15 seconds ago      Up 14 seconds                           nice_proskuriakova

[root@docker ~]# docker exec -i c0b2edc0c5ee bash 
ls
bin
boot
dev
etc
home
lib
...

[root@docker ~]# docker exec -it c0b2edc0c5ee bash
root@c0b2edc0c5ee:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin
```

如果从这个 stdin 中 exit，不会导致容器的停止。这就是为什么推荐大家使用 `docker exec` 的原因