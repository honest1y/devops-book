## Ingress 代理HTTP

- 后端服务

```
apiVersion: v1
kind: Service
metadata:
  name: service-myapp
  namespace: default
spec:
  selector:
    app: myapp
    release: canary
  ports:
    - name: http
      port: 80
      targetPort: 80

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: default
spec:
  replicas: 4
  selector:
    matchLabels:
      app: myapp
      release: canary
  template:
    metadata:
      labels:
        app: myapp
        release: canary
    spec:
      containers:
        - name: myapp
          image: ikubernetes/myapp:v2
          ports:
            - name: http
              containerPort: 80
```

- 创建 ingress-nginx

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.21.0/deploy/mandatory.yaml
```

- 让 ingress-nginx 在集群外部访问

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.21.0/deploy/provider/baremetal/service-nodeport.yaml
```

- 创建 ingress 对象，它能将 ingress-nginx 与 service 关联，从而在 service 后主机发生变动的时候，反应在 ingress-nginx 这个容器的配置文件中

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-deploy-myapp
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
    - host: myapp.cloudcared.cn                       # 基于主机名的访问
      http:
        paths:
          - path:                                   # 空的时候代表根，访问根的时候映射到 backend
            backend:                                # 后端的 service 的配置
              serviceName: service-myapp            # 关联 service 从而获取到后端主机的变动
              servicePort: 80                       # 关联 service 的地址
```

- 修改`ingress-nginx`svc的默认端口

```
kubectl edit svc ingress-nginx -n ingress-nginx
```

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/ingress-port.png)

```
[root@master ~]# kubectl get svc -n ingress-nginx               
NAME            TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx   NodePort   10.1.100.211   <none>        80:30080/TCP,443:30443/TCP   107m
```

```
[root@master ~]# kubectl create -f myapp.yaml 
service/service-ingress-myapp created
deployment.apps/myapp created
[root@master ~]# kubectl create -f in.yaml 
ingress.extensions/ingress-deploy-myapp created
[root@master ~]# 
```

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/04/image-20200421140949184.png)