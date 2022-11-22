---
weight: 2
title: "02 运行 Demo 应用"
date: 2022-11-23T00:52:18+08:00
draft: false
---

# 创建命名空间

创建一个名为 demo 的命名空间

```shell
kubectl create ns dmeo
```



# 创建 nginx demo

## 准备模版配置

下面是一份 nginx 的应用的 yaml 文件, 复制内容并保存为 `nginx.yaml` 作为部署配置文件.

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    app: nginx-001-deployment_label
  name: nginx-001-deployment
  namespace: demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-001-selector_label
  template:
    metadata:
      labels:
        app: nginx-001-selector_label
    spec:
      containers:
      - name: nginx-001-container
        image: nginx:stable
        imagePullPolicy: Always
        ports:
        - containerPort: 80
          protocol: TCP
          name: http
        - containerPort: 443
          protocol: TCP
          name: https
        env:
        - name: "password"
          value: "123456"
        - name: "age"
          value: "20"
        resources:
          limits:
            cpu: 2
            memory: 2Gi
          requests:
            cpu: 1
            memory: 1Gi

---
kind: Service
apiVersion: v1
metadata:
  labels:
    app: nginx-001-service_label
  name: nginx-001-service
  namespace: demo
spec:
  type: NodePort
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30090
  - name: https
    port: 443
    protocol: TCP
    targetPort: 443
    nodePort: 30091
  selector:
    app: nginx-001-selector_label
```



## 执行部署

```
kubectl apply -f nginx.yaml
```



查看 deployment 状态

```shell
# kubectl get deployment -n demo
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
nginx-001-deployment   3/3     3            3           18m
```

查看 service 状态

```shell
# kubectl get service -n demo
NAME                TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
nginx-001-service   NodePort   10.200.241.99   <none>        80:30090/TCP,443:30091/TCP   22m
```

查看 pod 状态

```shell
# kubectl get pod -n demo
NAME                                   READY   STATUS    RESTARTS   AGE
nginx-001-deployment-dd7d4cd4f-jv7hb   1/1     Running   0          20m
nginx-001-deployment-dd7d4cd4f-q9jtb   1/1     Running   0          20m
nginx-001-deployment-dd7d4cd4f-tqsrn   1/1     Running   0          20m
```



## 测试访问

测试访问 pod IP 端口

```shell
# #查看 pod IP 
# kubectl describe pod nginx-001-deployment-dd7d4cd4f-jv7hb -n demo |grep IP
                  cni.projectcalico.org/podIP: 10.100.238.193/32
                  cni.projectcalico.org/podIPs: 10.100.238.193/32
IP:               10.100.238.193
IPs:
  IP:           10.100.238.193

# #访问 IP 端口
# curl 10.100.238.193:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```



测试访问 service IP 端口

```shell
# curl 10.200.241.99:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```



测试访问 node 端口

```shell
# curl wkube3:30090
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```



一切访问正常, 将 node 端口添加到负载均衡, demo 应用就可以对外提供服务了.