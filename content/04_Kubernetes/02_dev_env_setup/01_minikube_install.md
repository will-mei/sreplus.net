---
title: "01 minikube install"
date: 2022-09-02T16:25:20+08:00
draft: false
---

### Intro

[minikube](https://minikube.sigs.k8s.io/) 是一款基于 Kubernetes 的定位于快速验证功能的小型容器编排环境配置工具.



### container engine install

这里使用 `podman` 容器引擎作为 `hypervisor`. 

查询其他受支持的虚拟化 `driver` 可以参考 [minikube start | minikube (k8s.io)](https://minikube.sigs.k8s.io/docs/start/#what-youll-need)



centos8 /rocky9 以上, 直接使用 yum 安装

```shell
yum -y install podman
```

查看 podman 安装情况

```shell
# podman version
Client:       Podman Engine
Version:      4.1.1
API Version:  4.1.1
Go Version:   go1.17.12
Built:        Wed Aug 10 00:43:56 2022
OS/Arch:      linux/amd64
```

如果是centos7 系统, 官方软件源中的版本可能较低, 需要高版本podman 需要自己手动编译安装相对麻烦, 建议直接使用高版本操作系统更加便捷.



### cri install

centos8/rocky8 以上

```shell
# 设定路径中的系统和版本号变量
OS=CentOS_8
VERSION=1.24

# 下载软件源配置文件
curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable.repo https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/devel:kubic:libcontainers:stable.repo
curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo

yum -y install cri-o
```



### minikube install

centos/rocky Linux示例

```shell
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

其他发行版可以参考 [minikube start | minikube (k8s.io)](https://minikube.sigs.k8s.io/docs/start/)



### start cluster

minikube 安装完既可使用 minikube 启动一个 k8s 集群.



#### minikub 用户配置

使用 minikube 启动集群 需要先切换到一个可以无密码执行 `sudo` 的管理员用户.

方法一, 特定用户免密 `sudo`, 编辑 `/etc/sudoers`, 或执行 `visudo` 

```ini
# 下面的 user_name 是需要配置的用户名
# 添加这一行就能免密 sudo 了
user_name ALL=(ALL:ALL) NOPASSWD: ALL
```

方法二, 管理员组免密 `sudo`, 执行 `visudo` 

```ini
%wheel        ALL=(ALL)       NOPASSWD: ALL
```

将用户添加到 `wheel` 组

```shell
useradd -G wheel user
```



#### 启动参数

按照上面的环境配置, podman + cri-o, 我们的启动命令如下

```bash
#!/bin/bash

minikube start \
    --driver=podman \
    --container-runtime=cri-o \
    --cpus=4 \
    --memory=6g \
    --kubernetes-version=v1.24.3 \
    --image-mirror-country=cn \
    --image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers'
```



如果你使用 docker + containerd, 可以修改成这样

```bash
minikube start \
    --driver=docker \
    --container-runtime=containerd \
    --cpus=4 \
    --memory=4g
    --kubernetes-version=v1.24.3 \
    --image-mirror-country='cn' \
    --image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers' 
```



> 因为 minikube 通常是单节点, 在部分情况下, 节点污点会影响服务部署, 可以查看 pod 状态, 并使用 `kubectl edit node <node name>` 进行修复.
>
> 其他更常见的是运行权限, 镜像拉取, 与运行时问题, 需要修改引擎的运行权限, 提前拉取镜像, 检查运行时安装来解决.

更多信息参考 [minikube start | minikube (k8s.io)](https://minikube.sigs.k8s.io/docs/start/)



#### 节点恢复

如果机器重启, 你并不需要重新创建, 只要配置文件还在, 你可以直接执行 `minikube start` 即可恢复.



### minikube 常用管理操作

状态查看

```shell
# minikube 版本
minikube version

# kubectl 版本
minikube kubectl version

# 节点信息
minikube node list
```



#### 插件管理

插件状态

```shell
 minikube addons list
```

启用插件

```shell
minikube addons enable dashboard
```

停用插件

```shell
 minikube addons enable dashboard
```



#### 服务暴露

查看所有服务

```shell
minikube service list
```

获取 k8s 服务的本地访问地址

```shell
minikube service <服务名>
```

将服务转发到宿主机节点IP

```shell
kubectl port-forward redis-master 6379:6379
```

将本地 6379 端口的访问转发到宿主机节点的 6379, 这样可以使用宿主机 IP 访问服务, 方便调试.
