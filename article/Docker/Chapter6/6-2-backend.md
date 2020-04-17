## 后台运行

更多的时候，需要让 Docker 在后台运行而不是直接把执行命令的结果输出在当前宿主机下。此时，可以通过添加 `-d` 参数来实现。

下面举两个例子来说明一下。

如果不使用 `-d` 参数运行容器。

```bash
[root@docker ~]# docker run ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
hello world
hello world
hello world
hello world
```

容器会把输出的结果 (STDOUT) 打印到宿主机上面

如果使用了 `-d` 参数运行容器。

```bash
[root@docker ~]# docker run -d ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
26e3a0f3c836da943f7ad1677bfede3ae87b7ba8b1a680b1997636c3a5552c84
```

此时容器会在后台运行并不会把输出的结果 (STDOUT) 打印到宿主机上面(输出结果可以用 `docker logs` 查看)。

**注：** 容器是否会长久运行，是和 `docker run` 指定的命令有关，和 `-d` 参数无关。

使用 `-d` 参数启动后会返回一个唯一的 id，也可以通过 `docker ps` 命令来查看容器信息。

```bash
[root@docker ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
26e3a0f3c836        ubuntu:18.04        "/bin/sh -c 'while t…"   12 seconds ago      Up 11 seconds                           frosty_johnson
```

要获取容器的输出信息，可以通过 `docker container logs` 命令。

```bash
[root@docker ~]# docker container logs 26e3a0f3c836
hello world
hello world
hello world
hello world
hello world
hello world
hello world
hello world
. . .
```

