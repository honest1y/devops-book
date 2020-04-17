##外部访问容器

容器中可以运行一些网络应用，要让外部也可以访问这些应用，可以通过 `-P` 或 `-p` 参数来指定端口映射。

当使用 `-P` 标记时，Docker 会随机映射一个的端口到内部容器开放的网络端口。

使用 `docker ps` 可以看到，本地主机的 32773 被映射到了容器的 5000 端口。此时访问本机的 32773 端口即可访问容器内 web 应用提供的界面。

```bash
[root@docker ~]# docker run -d -P training/webapp python app.py
6b7c9e7ffb7fa7c58c1c005d89aeb6a627faa7cc60afffc956d76f1e27244855
[root@docker ~]# docker ps 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                     NAMES
6b7c9e7ffb7f        training/webapp     "python app.py"     6 seconds ago       Up 5 seconds        0.0.0.0:32773->5000/tcp   cranky_bardeen
```

同样的，可以通过 `docker logs` 命令来查看应用的信息。

```bash
[root@docker ~]# docker logs -f 6b7c9e7ffb7f
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
123.139.156.117 - - [31/Mar/2020 03:13:50] "GET / HTTP/1.1" 200 -
123.139.156.117 - - [31/Mar/2020 03:13:50] "GET /favicon.ico HTTP/1.1" 404 -

```

`-p` 则可以指定要映射的端口，并且，在一个指定端口上只可以绑定一个容器。支持的格式有 `ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort`



####映射所有接口地址

使用 `hostPort:containerPort` 格式本地的 5000 端口映射到容器的 5000 端口，可以执行

```bash
[root@docker ~]# docker run -d -p 5000:5000 training/webapp python app.py
5b6cd08abfedc7700a79d036609771aa308810e02d9b787f44077c2ad4257dfb
[root@docker ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
5b6cd08abfed        training/webapp     "python app.py"     1 second ago        Up 1 second         0.0.0.0:5000->5000/tcp   epic_pascal
```

此时默认会绑定本地所有接口上的所有地址。



####映射到指定地址的指定端口

可以使用 `ip:hostPort:containerPort` 格式指定映射使用一个特定地址，比如 localhost 地址 127.0.0.1

```bash
[root@docker ~]# docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py
```



####映射到指定地址的任意端口

使用 `ip::containerPort` 绑定 localhost 的任意端口到容器的 5000 端口，本地主机会自动分配一个端口。

```bash
[root@docker ~]# docker run -d -p 127.0.0.1::5000 training/webapp python app.py
```

还可以使用 `udp` 标记来指定 `udp` 端口

```bash
[root@docker ~]# docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py
```



####查看映射端口配置

使用 `docker port` 来查看当前映射的端口配置，也可以查看到绑定的地址

```bash
[root@docker ~]# docker port epic_pascal 5000
0.0.0.0:5000
```

注意：

- 容器有自己的内部网络和 ip 地址（使用 `docker inspect` 可以获取所有的变量，Docker 还可以有一个可变的网络配置。）
- `-p` 标记可以多次使用来绑定多个端口

例如

```bash
[root@docker ~]# docker run -d \
    -p 5000:5000 \
    -p 3000:80 \
    training/webapp \
    python app.py
```