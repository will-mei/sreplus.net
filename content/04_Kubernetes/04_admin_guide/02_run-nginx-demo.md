---
weight: 2
title: "02 运行 Demo 应用"
date: 2022-11-23T00:52:18+08:00
draft: false
---

# 创建 nginx demo 应用

下面是一份 nginx 的应用的 yaml 文件, 执行 `kubectl apply -f nginx.yaml` 即可完成该 demo 的部署.

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

