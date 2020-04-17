##挂载主机目录

####挂载一个主机目录作为数据卷

使用 `--mount` 标记可以指定挂载一个本地主机的目录到容器中去。

```bash
[root@docker ~]# docker run -d -P \
> --name web \
> --mount type=bind,source=/src/webapp,target=/opt/webapp \
> training/webapp \
> python app.py
```

上面的命令加载主机的 `/src/webapp` 目录到容器的 `/opt/webapp`目录。这个功能在进行测试的时候十分方便，比如用户可以放置一些程序到本地目录中，来查看容器是否正常工作。本地目录的路径必须是绝对路径，以前使用 `-v` 参数时如果本地目录不存在 Docker 会自动为你创建一个文件夹，现在使用 `--mount` 参数时如果本地目录不存在，Docker 会报错。

```bash
docker: Error response from daemon: invalid mount config for type "bind": bind source path does not exist: /src/webapp.
```



Docker 挂载主机目录的默认权限是 `读写`，用户也可以通过增加 `readonly` 指定为 `只读`。

```bash
$ docker run -d -P \
  --name web \
  --mount type=bind,source=/src/webapp,target=/opt/webapp,readonly \
  training/webapp \
  python app.py
```

加了 `readonly` 之后，就挂载为 `只读` 了。如果你在容器内 `/opt/webapp` 目录新建文件，会显示如下错误

```bash
[root@docker ~]# docker exec -it 11a253e0372f bash
root@11a253e0372f:/opt/webapp# cd /webapp/
root@11a253e0372f:/webapp# ls
root@11a253e0372f:/webapp# touch 123
touch: cannot touch '123': Read-only file system
```

####挂载一个本地主机文件作为数据卷

`--mount` 标记也可以从主机挂载单个文件到容器中

```bash
docker run --rm -it \
--mount type=bind,source=$HOME/.bash_history,target=/root/.bash_history \
ubuntu:18.04 \
bash
```

```bash
root@d7e3e687a07d:/# history |head -10
    1   
    2  wget 106.54.179.232
```

这样就可以记录在容器输入过的命令了。