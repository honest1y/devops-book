# 目录

* [序言](README.md)
* Docker
  * [1、Docker介绍](article/Docker/docker.md) 
  * 2、基本概念
    * [2.1 镜像](article/Docker/Chapter2/2-1-image.md)
    * [2.2 容器](article/Docker/Chapter2/2-2-containor.md)
    * [2.3 仓库](article/Docker/Chapter2/2-3-repository.md)
  * 3、安装Docker
    * [3.1 Centos](article/Docker/Chapter3/3-1-install.md)
    * 3.2 Ubuntu
    * 3.3 MacOS
  * 4、使用镜像
    * [4.1 获取镜像](article/Docker/Chapter4/4-1-get-image.md)
    * [4.2 列出镜像](article/Docker/Chapter4/4-2-list-image.md)
    * [4.3 删除本地镜像](article/Docker/Chapter4/4-3-delete-image.md)
    * [4.4 commit 修改镜像](article/Docker/Chapter4/4-4-commit-image.md)
  * 5、Dockerfile
    * [5.1 FROM 指定基础镜像](article/Docker/Chapter5/5-1-from.md)
    * [5.2 RUN 执行命令](article/Docker/Chapter5/5-2-run.md)
    * [5.3 构建镜像](article/Docker/Chapter5/5-3-build.md)
    * [5.4 镜像构建上下文（Context）](article/Docker/Chapter5/5-4-context.md)
    * [5.5 COPY 复制文件](article/Docker/Chapter5/5-5-copy.md)
    * [5.6 ADD 高级复制文件](article/Docker/Chapter5/5-6-add.md)
    * [5.7 CMD 容器启动命令](article/Docker/Chapter5/5-7-cmd.md)
    * [5.8 ENTRYPOINT 入口点](article/Docker/Chapter5/5-8-entrypoint.md)
    * [5.9 ENV 设置环境变量](article/Docker/Chapter5/5-9-env.md)
    * [5.10 ARG 构建参数](article/Docker/Chapter5/5-10-arg.md)
    * [5.11 VOLUME 定义匿名卷](article/Docker/Chapter5/5-11-volume.md)
    * [5.12 EXPOSE 暴露端口](article/Docker/Chapter5/5-12-expose.md)
    * [5.13 WORKDIR 指定工作目录](article/Docker/Chapter5/5-13-workdir.md)
    * [5.14 USER 指定当前用户](article/Docker/Chapter5/5-14-user.md)
    * [5.15 HEALTHCHECK 健康检查](article/Docker/Chapter5/5-15-healthcheck.md)
    * [5.16 SAVE/LOAD](article/Docker/Chapter5/5-16-saveload.md)
  * 6、容器
    * [6.1 启动](article/Docker/Chapter6/6-1-start.md)  
    * [6.2 后台运行](article/Docker/Chapter6/6-2-backend.md)  
    * [6.3 终止](article/Docker/Chapter6/6-3-stop.md)  
    * [6.4 进入容器](article/Docker/Chapter6/6-4-exec.md)  
    * [6.5 导出和导入](article/Docker/Chapter6/6-5-export.md)  
    * [6.6 删除](article/Docker/Chapter6/6-6-delete.md)  
  * 7、仓库
    * [7.1 仓库](article/Docker/Chapter7/7-1-repository.md)  
    * [7.2 Docker Hub](article/Docker/Chapter7/7-2-dockerhub.md)  
    * [7.3 Registry](article/Docker/Chapter7/7-3-registry.md)  
  * 8、数据管理
    * [8.1 数据卷](article/Docker/Chapter8/8-1-volume.md)  
    * [8.2 挂载主机目录](article/Docker/Chapter8/8-2-mount.md)  
  * 9、网络管理
    * [9.1 外部访问容器](article/Docker/Chapter9/9-1-external.md)  
    * [9.2 容器互联](article/Docker/Chapter9/9-2-interconnection.md)  
    * [9.3 配置DNS](article/Docker/Chapter9/9-3-dns.md)  
  * 10、Docker三驾马车-Compose
    * [10.1 简介](article/Docker/Chapter10/10-1-base.md)  
    * [10.2 安装与卸载](article/Docker/Chapter10/10-2-install.md)  
    * [10.3 使用](article/Docker/Chapter10/10-3-use.md)  
    * [10.4 命令说明](article/Docker/Chapter10/10-4-command.md)  
    * [10.5 Django](article/Docker/Chapter10/10-5-django.md)  
    * [10.6 WordPress](article/Docker/Chapter10/10-6-wordpress.md)  
  * 11、Docker三驾马车-Machine
    * [11.1 简介](article/Docker/Chapter11/11-1-base.md)  
    * [11.2 安装与卸载](article/Docker/Chapter11/11-2-install.md)  
    * [11.3 使用](article/Docker/Chapter11/11-3-use.md)  
  * 12、Docker三驾马车-Swarm
    * [12.1 简介](article/Docker/Chapter12/12-1-base.md)  
    * [12.2 安装与卸载](article/Docker/Chapter12/12-2-install.md)  
    * [12.3 使用](article/Docker/Chapter12/12-3-use.md)  
* Rancher
  * 1、概述
    * [1.1 什么是Rancher](article/Rancher/Chapter1/1-1-base.md)
    * 1.2 Rancher架构
      * [1.2.1 Docker简述](article/Rancher/Chapter1/1-2-1-docker.md)
      * [1.2.2 Kubernetes简述](article/Rancher/Chapter1/1-2-2-kubernetes.md)
      * [1.2.3 Rancher架构](article/Rancher/Chapter1/1-2-3-rancher.md)
  * 2、相关术语
    * [2.1 全局层](article/Rancher/Chapter2/2-1-global.md)
    * [2.2 集群层](article/Rancher/Chapter2/2-2-colony.md)
    * [2.3 项目层](article/Rancher/Chapter2/2-3-project.md)
    * [2.4 其他](article/Rancher/Chapter2/2-4-other.md)
  * 3、功能列表
    * [3.1 K8S集群管理](article/Rancher/Chapter3/3-1-k8s.md)
    * [3.2 多租户功能](article/Rancher/Chapter3/3-2-tenant.md)
    * [3.3 容器主机管理](article/Rancher/Chapter3/3-3-dockerhost.md)
    * [3.4 容器调度与管理](article/Rancher/Chapter3/3-4-dispatch.md)
    * [3.5 容器应用管理](article/Rancher/Chapter3/3-5-dockerapp.md)
    * [3.6 容器管理](article/Rancher/Chapter3/3-6-docker.md)
    * [3.7 容器网络管理](article/Rancher/Chapter3/3-7-network.md)
    * [3.8 负载均衡服务](article/Rancher/Chapter3/3-8-lb.md)
    * [3.9 容器存储服务](article/Rancher/Chapter3/3-9-storage.md)
    * [3.10 系统监控及日志](article/Rancher/Chapter3/3-10-log.md)
    * [3.11 应用商店管理](article/Rancher/Chapter3/3-11-appstore.md)
    * [3.12 系统管理及安全](article/Rancher/Chapter3/3-12-manager.md)
    * [3.13 镜像库功能](article/Rancher/Chapter3/3-13-image.md)
    * [3.14 系统集成支持](article/Rancher/Chapter3/3-14-system.md)
    * [3.15 CI/CD功能](article/Rancher/Chapter3/3-15-cicd.md)
  * 4、快速入门
    * [4.1 入门须知](article/Rancher/Chapter4/4-1-base.md)
    * 4.2 配置Linux主机
      * [4.2.1 系统需求](article/Rancher/Chapter4/4-2-1-system.md)
      * [4.2.2 硬件需求](article/Rancher/Chapter4/4-2-2-handware.md)
      * [4.2.3 软件需求](article/Rancher/Chapter4/4-2-3-software.md)
    * [4.3 安装Rancher](article/Rancher/Chapter4/4-3-install.md)
    * [4.4 登陆Rancher](article/Rancher/Chapter4/4-4-login.md)
    * [4.5 创建K8S集群](article/Rancher/Chapter4/4-5-createk8s.md)
    * [4.6 部署工作负载](article/Rancher/Chapter4/4-6-deploy.md)
  * 5、Rancher实战
    * [5.1 基于 Rancher 运行 Django 程序](article/Rancher/Chapter5/5-1-django.md)
    * [5.2 基于 Rancher 运行 VUE 程序](article/Rancher/Chapter5/5-2-vue.md)
  * [6、参考文档](article/Rancher/document.md)
* Kubernetes
  * 一、入门
    * [1.1 什么是Kubernetes?](article/Kubernetes/Chapter1/1-Introduction.md)
    * [1.2 为什么要用Kubernetes](article/Kubernetes/Chapter1/2-Whykubernetes.md)
    * [1.3 GuestBook举例](article/Kubernetes/Chapter1/3-Guestbook.md)
      * [1.3.1 创建redis-master Pod和服务](article/Kubernetes/Chapter1/3-1-redis-master.md)
      * [1.3.2 创建redis-slave Pod和服务](article/Kubernetes/Chapter1/3-2-redis-slave.md)
      * [1.3.3 创建frontend Pod和服务](article/Kubernetes/Chapter1/3-3-frontend.md)
      * [1.3.4 通过浏览器访问网页](article/Kubernetes/Chapter1/3-4-test.md)
  * 二、集群搭建
    * [2.1 环境准备](article/Kubernetes/Chapter2/1-Prepare.md)
    * [2.2 集群配置](article/Kubernetes/Chapter2/2-Install.md)
  * 三、Ingress
    * [3.1 Ingress介绍](article/Kubernetes/Chapter3/1-ingress.md)
    * [3.2 Ingress 代理HTTP](article/Kubernetes/Chapter3/2-ingress-http.md)
    * [3.3 Ingress 代理HTTPS](article/Kubernetes/Chapter3/3-ingress-https.md)
* Shell
  * [1、什么是Shell](article/Shell/Chapter1/1-1-shell.md)
  * 2、Shell 变量
      * [2.1 普通变量](article/Shell/Chapter2/2-1-normal.md)
      * [2.2 特殊变量](article/Shell/Chapter2/2-2-special.md)
  * 3、字符串和数组
      * [3.1 字符串](article/Shell/Chapter3/3-1-string.md)
      * [3.2 数组](article/Shell/Chapter3/3-2-array.md)
