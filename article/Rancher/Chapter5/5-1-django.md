##基于 Rancher 运行 Django 程序



#### DEMO地址

**https://github.com/honest1y/demo-backend**

#### Dockerfile

```bash
FROM python:latest
ENV BASE_ROOT="/data"
WORKDIR /data
ADD . ${BASE_ROOT}
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' >/etc/timezone
RUN pip config set global.index-url http://mirrors.aliyun.com/pypi/simple && pip config set install.trusted-host mirrors.aliyun.com
RUN pip install --upgrade pip && pip install --default-timeout=100 -r ${BASE_ROOT}/requirements.txt \
&& chmod +x ${BASE_ROOT}/entrypoint.sh && ln -s ${BASE_ROOT}/entrypoint.sh /bin/entrypoint.sh
EXPOSE 8000/tcp
ENTRYPOINT ["./entrypoint.sh"]
```

#### 1、拉取代码

```bash
git clone https://github.com/honest1y/demo-backend.git
```



#### 2、构建镜像

```bash
cd demo-backend
docker build -t registry.cn-shanghai.aliyuncs.com/cloud-devops/demo-backendend:v1 .
```

`registry.cn-shanghai.aliyuncs.com` 为[阿里云容器镜像服务](https://cr.console.aliyun.com/cn-shanghai/instances/repositories)，即私有化仓库



#### 3、推送镜像

```bash
docker push registry.cn-shanghai.aliyuncs.com/cloud-devops/demo-backendend:v1
```



#### 4、Rancher配置

1、点击 “资源” ->  "密文" -> "镜像库凭证列表"，添加凭证，填写镜像仓库相关信息

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/WX01.png)

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/WX02.png)

2、点击“资源” -> “工作负载”，部署服务

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/WX07.png)

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/WX08.png)

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/WX09.png)



#### 5、访问测试

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/WX10.png)