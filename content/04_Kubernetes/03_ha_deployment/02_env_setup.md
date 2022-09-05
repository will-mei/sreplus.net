---
title: "02 环境准备"
date: 2022-09-02T16:24:49+08:00
draft: false
---

### 基础集群环境配置

服务集群通常需要一些基础服务准备

1. 网络连通性
2. 防火墙规则
3. 时区时间同步
4. DNS 集群节点的主机名解析 (DNS服务解析/本地解析)
5. 软件源配置 (软件包,镜像仓库)
6. 操作系统升级 (系统软件,内核)
7. 其他



LB服务器配置

keepalived [Keepalived 安装和配置详解_Direct_的博客-CSDN博客_keepalived安装与配置](https://blog.csdn.net/D1179869625/article/details/126198495)

haproxy     [kubernetes的api-server高可用配置_激情燃烧的岁月的技术博客_51CTO博客](https://blog.51cto.com/liuzhengwei521/2377280)

> 非生产环境中, 如果只需要高可用而不在乎负载均衡也可以只安装 keepalived.



### 运行时安装

containerd 安装 (CentOS/Rocky)



配置 yum 源

```shell
yum install -y yum-utils
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
yum makecache
```
参考文档 

[阿里巴巴开源镜像站-OPSX镜像站-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/mirror/)

[docker-ce镜像_docker-ce下载地址_docker-ce安装教程-阿里巴巴开源镜像站 (aliyun.com)](https://developer.aliyun.com/mirror/docker-ce?spm=a2c6h.13651102.0.0.3e221b11t6Y9Wk)



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

