## 安装Dashboard插件

Kubernetes Dashboard 是 k8s集群的一个 WEB UI管理工具，代码托管在 github 上，地址：https://github.com/kubernetes/dashboard



## 安装

直接使用官方的配置文件安装即可：

```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.1/aio/deploy/recommended.yaml
```

为了测试方便，我们将`Service`改成`NodePort`类型，注意 YAML 中最下面的 Service 部分新增一个`type=NodePort`：

```yaml
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 443
      targetPort: 8443
  type: NodePort
  selector:
    k8s-app: kubernetes-dashboard
```

然后直接部署新版本的`dashboard`即可：

```shell
$ kubectl create -f kubernetes-dashboard.yaml
```

然后我们可以查看 dashboard 的外网访问端口：

```shell
$ kubectl get svc -n kubernetes-dashboard    
NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP   10.1.200.162   <none>        8000/TCP        62s
kubernetes-dashboard        NodePort    10.1.38.183    <none>        443:30796/TCP   62s
```

然后直接访问集群中的任何一个节点 IP 加上上面的**30796**端口即可打开 dashboard 页面了

默认 dashboard 会跳转到登录页面，我们可以看到 dashboard 提供了`Kubeconfig`和`token`两种登录方式，我们可以直接跳过或者使用本地的`Kubeconfig`文件进行登录，可以看到会跳转到如下页面：

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/05/kube-dashborad.png)

## 身份认证

登录 dashboard 的时候支持 Kubeconfig 和token 两种认证方式，Kubeconfig 中也依赖token 字段，所以生成token 这一步是必不可少的。

### 生成token

我们创建一个admin用户并授予admin 角色绑定，使用下面的yaml文件创建admin用户并赋予他管理员权限，然后就可以通过token 登陆dashbaord，这种认证方式本质实际上是通过Service Account 的身份认证加上Bearer token请求 API server 的方式实现。

```
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: admin
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: admin
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile
```

上面的`admin`用户创建完成后我们就可以获取到该用户对应的`token`了，如下命令：

```shell
[root@master ~]# kubectl get secret -n kube-system|grep admin-token                                      
admin-token-wvzz4                                kubernetes.io/service-account-token   3      2m28s
[root@master ~]# kubectl get secret admin-token-wvzz4 -o jsonpath={.data.token} -n kube-system |base64 -d     
eyJhbGciOiJSUzI1NiIsImtpZCI6ImJNall2T0xpcUVFdDVGNm9EdTM2UE9wckRSSVN6MktveXZhUm9jQ3pUX3MifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi10b2tlbi13dnp6NCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJhZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjVhNzc0MmVlLTA2ZjgtNDFlNS04YTFlLTQ2NDgyZWVhMGQ4YyIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTphZG1pbiJ9.oxvEHYZ7w8KO4juHYVKjMt7N5zmYJ0ed4vOKH2u4L5n4W9BfRvP-fB9zpP2tT76-rPnQDgK9PfWAV_Z11YjlUu1xonuoQD4TMKmenIwSnyRGUlcP4_NTV19HtSoFZwKFY-B8DKOOJbxhLEdK87dbnybgJPKaoyrnUvY9ukEEy8pBONN7uiuQlIZwFAFbEF9ihdvK3OvVGmfFhRr8Zo20lNLgORizg1E7EhyYnsVXIo7KMWHhBkcpcQDGhEAhKpveMuP77azW-TR-Hd88ouBOlyT3efJGIHiT3rb3YfzqefTX
```

然后在 dashboard 登录页面上直接使用上面得到的 token 字符串即可登录，这样就可以拥有管理员权限操作整个 kubernetes 集群的对象，当然你也可以为你的登录用户新建一个指定操作权限的用户。

![](https://cdn.cloudcared.cn/wp-content/uploads/2020/05/kube-dashboard-login.png)