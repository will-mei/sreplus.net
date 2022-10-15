---
title: "02 实验环境配置"
date: 2021-05-31T02:51:07-04:00
Weight: 2
draft: false
---

## 容器化部署Prometheus

官方文档： [Installation | Prometheus](https://prometheus.io/docs/prometheus/latest/installation/)

  <br>

### 环境准备

关于 Docker 的安装， 请参考： [Install Docker Engine on CentOS | Docker Documentation](https://docs.docker.com/engine/install/centos/) 

  <br>

### 容器部署：

关于 Prometheus 容器的配置以及容器的启动参数

-  参考1 流程清晰，但是部分参数实验测试有问题 [How To Install Prometheus with Docker on Ubuntu 18.04 – devconnected](https://devconnected.com/how-to-install-prometheus-with-docker-on-ubuntu-18-04/)

> 其中挂载参数和配置文件在 CentOS下测试不通过， 需要写完整， 不然会出现容器内的权限错误，类似于 [打开查询日志文件时出错”file=/prometheus/queries.active err="open /prometheus/queries.active: 权限被拒绝](https://www.likecs.com/ask-1297108.html?sc=552) 

- 参考2 [Install Prometheus using Docker(2022) | TechGeekNxt >> (techgeeknext.com)](https://www.techgeeknext.com/tools/docker/install-prometheus-using-docker)

- 参考3 更加简洁清晰 [Docker运行Prometheus(普罗米修斯),grafana,springboot整合_Andy_Health的博客-CSDN博客_docker启动prometheus](https://blog.csdn.net/Andy_Health/article/details/123475186)

- 参考4 （关于容器数据卷配置更加详细）[docker方式快速部署 Prometheus - 进_进 - 博客园 (cnblogs.com)](https://www.cnblogs.com/augus007/articles/9225431.html)

