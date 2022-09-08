---
weight: 4
title: "03 CRI 安装"
date: 2022-09-08T16:23:50+08:00
draft: false啊运行时与OS
---



### 容器运行时与OS

containerd 与 cri-o 运行环境安装, 系统环境 CentOS 或 Rocky



#### containerd 

配置 yum 源

```shell
yum install -y yum-utils
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
yum makecache
```



安装 containerd.io 软件包, 启用服务

```shell
# 检查软件源中的 containerd 版本
yum list containerd.io

# 安装 containerd.io
yum -y install containerd.io

# 开机启动
systemctl enable containerd
# 启动
systemctl start containerd
# 检查服务状态
systemctl status containerd
# 检查客户端版本
ctr version 
```



调整 containerd 服务配置文件

```shell
# 生成新的系统配置文件
containerd config default > /etc/containerd/config.toml

# 启用 systemd 作为 cgroup driver
sed -i "/systemd_cgroup/s/false/true/" /etc/containerd/config.toml
# 配置阿里云镜像仓库加速下载
sed -i "/sandbox_image/s#k8s.gcr.io#registry.aliyuncs.com/google_containers#" /etc/containerd/config.toml
```



参考文档 

[阿里巴巴开源镜像站-OPSX镜像站-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/mirror/)

[docker-ce镜像_docker-ce下载地址_docker-ce安装教程-阿里巴巴开源镜像站 (aliyun.com)](https://developer.aliyun.com/mirror/docker-ce?spm=a2c6h.13651102.0.0.3e221b11t6Y9Wk)





#### CRI-O

确定 `OS` 系统版本 和 `VERSION` 软件版本, 然后在命令行下载软件源配置文件, 再使用包管理器安装即可.



**系统版本**

| Operating system | $OS               |
| :--------------- | :---------------- |
| Centos 8 /Rocky9 | `CentOS_8`        |
| Centos 8 Stream  | `CentOS_8_Stream` |
| Centos 7         | `CentOS_7`        |

**软件版本号**

到 [官方软件源](https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/) 或 [Releases · cri-o/cri-o (github.com)](https://github.com/cri-o/cri-o/releases) 查看最新的稳定版本, 记下主版本和次版本号.

若最新版本为 `1.24.2` 我们需要使用 `1.24` 作为版本号即可.



安装软件包

```shell
# 设定路径中的系统和版本号变量
OS=CentOS_7
VERSION=1.24

# 下载软件源配置文件
curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable.repo https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/devel:kubic:libcontainers:stable.repo
curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo

yum install cri-o
```

其他发行版参考官方文档 [cri-o](https://cri-o.io/)
