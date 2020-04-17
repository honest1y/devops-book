## 基于 Rancher 运行 VUE 程序

#### DEMO地址

**https://github.com/honest1y/demo-frontend**

#### Dockerfile

```bash
FROM node:10 as demo-frontend-build
COPY ./ /demo-frontend
WORKDIR /demo-frontend
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' >/etc/timezone
RUN npm install -g cnpm --registry=https://registry.npm.taobao.org && cnpm install && npm run build

FROM nginx:latest
RUN mkdir /demo-frontend-html
RUN rm -fr /etc/nginx/conf.d/default.conf
COPY --from=demo-frontend-build /demo-frontend/dist /demo-frontend-html
COPY demo-frontend.conf /etc/nginx/conf.d
```



#### demo-frontend.conf

```bash
server {
  listen 80;
  server_name localhost;
  location / {
    index      index.html;
    root       /demo-frontend-html;
    try_files $uri $uri/ /index.html;
    client_body_buffer_size 200m;
    charset utf-8;
  }
  gzip  on;
  gzip_vary on;
  gzip_min_length 1k; 
  gzip_buffers 4 16k;
  gzip_comp_level 6; 
  gzip_types  text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png image/x-icon;
}
```



#### 1、拉取代码

```bash
git clone https://github.com/honest1y/demo-frontend.git
```



#### 2、构建镜像

```bash
docker build -t registry.cn-shanghai.aliyuncs.com/cloud-devops/demo-frontend:v1 .
```

`registry.cn-shanghai.aliyuncs.com` 为[阿里云容器镜像服务](https://cr.console.aliyun.com/cn-shanghai/instances/repositories)，即私有化仓库



#### 3、推送镜像

```bash
docker push registry.cn-shanghai.aliyuncs.com/cloud-devops/demo-frontend:v1
```



#### 4、Rancher配置

1、点击 “资源” ->  "密文" -> "镜像库凭证列表"，添加凭证，填写镜像仓库相关信息

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/WX01.png)

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/WX02.png)

2、点击“资源” -> “工作负载”，部署服务

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/WX03.png)

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/WX04.png)

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/WX05.png)



#### 5、访问测试

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/WX06.png)