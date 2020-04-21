##Ingress 代理 HTTPS

- 后端 service 和 pods

```
apiVersion: v1
kind: Service
metadata:
  name: service-tomcat
  namespace: default
spec:
  selector:
    app: tomcat
    release: canary
  ports:
    - name: http
      port: 8080
      targetPort: http
    - name: ajp
      port: 8009
      targetPort: ajp

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-tomcat
  namespace: default
spec:
  replicas: 4
  selector:
    matchLabels:
      app: tomcat
      release: canary
  template:
    metadata:
      labels:
        app: tomcat
        release: canary
    spec:
      containers:
        - name: tomcat
          image: tomcat:8.5.32-jre8-alpine
          ports:
            - name: http
              containerPort: 8080
            - name: ajp
              containerPort: 8009
```

- 制作自签名证书，让 ingress-nginx 带有证书来访问

```
# 生成 key
[root@master ~]# openssl genrsa -out tls.key 2048

# 生成自签证书，CN=域名必须要与自己的域名完全一致
[root@master ~]# openssl req -new -x509 -key tls.key -out tls.crt -subj /C=CN/ST=Beijing/L=Beijing/O=DevOps/CN=tomcat.cloudcared.cn
```

- 创建 secret 证书对象

```
[root@master ~]# kubectl create secret tls tomcat-ingress-secret --cert=tls.crt --key=tls.key
[root@master ~]# kubectl get secret
NAME                    TYPE                                  DATA   AGE
default-token-n5xhn     kubernetes.io/service-account-token   3      130m
tomcat-ingress-secret   kubernetes.io/tls                     2      2m59s
```

- 创建带证书的 ingress 对象，它能将 ingress-tomcat 与 service 关联，从而在 service 后主机发生变动的时候，反应在 ingress-tomcat 这个容器的配置文件中

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-deploy-tomcat-tls
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  tls:
    - hosts:
      - tomcat.kaliarch.com
      secretName: tomcat-ingress-secret
  rules:
    - host: tomcat.cloudcared.cn
      http:
        paths:
          - path:
            backend:
              serviceName: service-tomcat
              servicePort: 8080
```

- 查看 ingress-nginx 对外暴露的端口

```
[root@master ~]# kubectl get service -n ingress-nginx
NAME            TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx   NodePort   10.1.100.211   <none>        80:30080/TCP,443:30443/TCP   121m
```

- 使用 nodeip + ingress-nginx 暴露端口访问

```
https://tomcat.cloudcared.cn:30443
```

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/tomcat-ingress.jpg)