---
title: "02 实验环境配置"
date: 2021-05-31T02:51:07-04:00
Weight: 2
draft: false
---

## Intro

官方文档： [Installation | Prometheus](https://prometheus.io/docs/prometheus/latest/installation/)

Prometheus 的部署十分方便, 下载二进制包然后直接指定配置文件启动服务即可.

Prometheus 对于容器运行环境也十分友好, 为配置端口映射, 配置文件和持久化卷即可.

  <br>

## 服务部署

1. 二进制方式
2. 容器方式

<br>

### 二进制

1. 下载二进制包, [Download | Prometheus](https://prometheus.io/download/#prometheus)

2. 编写配置文件

   ```yaml
   global:
     scrape_interval: 15s
   
   scrape_configs:
   - job_name: node
     static_configs:
     - targets: ['localhost:9100']
   ```

   

3. 运行服务进程

   ```shell
   ./prometheus --config.file=prometheus.yml
   ```

<br>

### 容器部署

**环境准备**

关于 Docker 的安装， 请参考： [Install Docker Engine on CentOS | Docker Documentation](https://docs.docker.com/engine/install/centos/) 

**启动容器**

关于 Prometheus 容器的配置以及容器的启动参数示例

```shell
# useradd -rs /sbin/nologin prometheus
# id -u prometheus
# id -g prometheus
# mkdir /data/prometheus
# chown prometheus:prometheus /data/prometheus

# mkdir -p /etc/prometheus
# edit /etc/prometheus/prometheus.yml


docker run \
    -d \
    -p 9090:9090 \
    --user 997:993 \
    --net=host \
    --restart=always \
    --name=mon1 \
    -v /etc/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
    -v /data/prometheus:/data/prometheus \
    prom/prometheus \
    --config.file="/etc/prometheus/prometheus.yml" \
    --storage.tsdb.path="/data/prometheus"
```

参考文档:

- 参考1 流程清晰，但是部分参数实验测试有问题 [How To Install Prometheus with Docker on Ubuntu 18.04 – devconnected](https://devconnected.com/how-to-install-prometheus-with-docker-on-ubuntu-18-04/)

  > 其中挂载参数和配置文件在 CentOS下测试不通过， 需要写完整， 不然会出现容器内的权限错误，类似于:
  >
  > [打开查询日志文件时出错”file=/prometheus/queries.active err="open /prometheus/queries.active: 权限被拒绝](https://www.likecs.com/ask-1297108.html?sc=552) 

- 参考2 [Install Prometheus using Docker(2022) | TechGeekNxt >> (techgeeknext.com)](https://www.techgeeknext.com/tools/docker/install-prometheus-using-docker)

- 参考3 更加简洁清晰 [Docker运行Prometheus(普罗米修斯),grafana,springboot整合_Andy_Health的博客-CSDN博客](https://blog.csdn.net/Andy_Health/article/details/123475186)

- 参考4 （关于容器数据卷配置更加详细）[docker方式快速部署 Prometheus - 进_进 - 博客园 (cnblogs.com)](https://www.cnblogs.com/augus007/articles/9225431.html)

<br>

## 数据源

不同类型的数据源需要不同类型的 Exporter 采集程序, 常见服务的 Exportor 都可以从网络获取, 以节点数据源 Node_Exporter 为例:

1. 下载二进制包, 下载地址: [Download | Prometheus](https://prometheus.io/download/#node_exporter)

2. 解压后运行, 并指定运行端口 

   示例:

   ```shell
   ./node_exporter --web.listen-address 127.0.0.1:8080
   
   # 不加端口则默认运行在 9100 端口
   ./node_exporter
   ```

3. 通过 web 访问检查 exporter 运行状态, 地址: http://<服务器 IP>:<服务端口>, 点击页面的 Metrics 链接可查看监控的指标数据.

   如: `http://localhost:9100` 即可查看到 Node Exporter 页面与 Metrics 链接地址, 页面内容类似下面这样

   # Node Exporter

   [Metrics](http://192.168.124.167:9100/metrics)

4. metrics 页面可以查看到所有的指标已经以 `#` 开头的帮助信息.

<br>

## 配置服务端拉取数据源

修改配置文件, 示例

```yaml
# A scrape configuration scraping a Node Exporter and the Prometheus server
global:
  scrape_interval: 15s
  evaluation_interval: 15s

# itself.
scrape_configs:
  # Scrape Prometheus itself every 10 seconds.
  - job_name: 'prometheus'
    scrape_interval: 10s
    static_configs:
      - targets: ['localhost:9090']
  # node exporter
  - job_name: 'node'
    static_configs:
      - targets: ['192.168.124.167:9100']
```

重启服务端进程或容器

```shell
./prometheus --config.file=prometheus.yml

# docker
docker restart mon1
```



<br>

## 查询监控数据

登录服务端 web 地址, 如 `http://localhost:9090` 

1. 查询状态数据

   尝试查询 `node_network_up` , 可以在搜索框输入 `k_u` 即可从下拉选单中改指标.

   点击 execute 或者回车, 如果看到来自数据源节点 `... instance="192.168.124.167:9100" ...` 的网卡信息即查询成功.

2. 查询使用量数据

   查询 `node_memory_Active_bytes`, 并点击 Graph 标签查看数据图

   查询 `node_memory_Active_bytes/1024`, 执行并查看查询项加入运算表达式的结果

   类型地, 可以继续尝试 `node_load1` 等, 查看负载



<br>

## 小结

这里入门完成, 我们部署完成了服务端与数据源, 从 Prometheus UI 实时获取了机器的网卡, 内存, 负载等信息.



