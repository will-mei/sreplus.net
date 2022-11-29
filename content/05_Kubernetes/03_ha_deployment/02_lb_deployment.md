---
weight: 2
title: "02 LB 部署"
date: 2022-09-08T16:24:16+08:00
draft: false
---



### LB服务器配置

多个负载均衡器之间, keepalived 服务配置需要按节点调整部分配置, haproxy 服务配置各个节点完全一样.



案例的 VIP 是`192.168.129.3` 负载均衡端口是 `6443` 



防火墙配置, 参考下面的命令添加放行规则

```shell
# 放行 vrrp 协议
firewall-cmd --add-protocol=vrrp --permanent
# api-server 6443端口
firewall-cmd --add-port=6443/tcp --permanent

# haproxy web界面(可选)
firewall-cmd --add-port=1080/tcp --permanent

# 重新加载
firewall-cmd --reload
```

> 如果是有外部防火墙的情况可以关闭



内核参数调整, 允许绑定非本地 IP, 否则 keepalived 集群 BACKUP 节点无法监听VIP

编辑 `/etc/sysctl.conf` 增加以下配置

```ini
net.ipv4.ip_nonlocal_bind = 1
net.ipv4.ip_forward = 1
```



软件安装

```shell
yum -y  install keepalived haproxy
systemctl enable keepalived haproxy
```



Keepalived  最精简配置文件示例

```ini
! Configuration File for keepalived

global_defs {
   router_id LVS_DEVEL_WKUBE228 # id for this node
   vrrp_skip_check_adv_addr
   #vrrp_strict                 # disable on unicast mode
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_WKUBE {       # instance name(same for both node)
    state MASTER                # role for this node
    interface eth0
    virtual_router_id 3        # instance id(same for both node)
    priority 101                # priority for this node
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass wkube1
    }
    virtual_ipaddress {
        192.168.129.3
    }
}
```

HAproxy 配置文件示例

```ini
#---------------------------------------------------------------------
# Example configuration for a possible web application.  See the
# full configuration options online.
#
#   http://haproxy.1wt.eu/download/1.4/doc/configuration.txt
#
#---------------------------------------------------------------------

#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    tcp
    log                     global
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout check           10s
    maxconn                 3000

#---------------------------------------------------------------------
# main frontend which proxys to the backends
#---------------------------------------------------------------------
frontend  kube-api
    bind                        192.168.129.3:6443
    mode                        tcp
    log                         global
    default_backend             kube-masters

#---------------------------------------------------------------------
# kube master nodes backend
#---------------------------------------------------------------------
backend kube-masters
    balance     source
    server      wkube1 192.168.129.228:6443 check inter 2000 fall 2
    server      wkube2 192.168.129.211:6443 check inter 2000 fall 2
    server      wkube3 192.168.129.146:6443 check inter 2000 fall 2

listen stats
    mode  http
    bind  0.0.0.0:1080
    stats enable
    stats hide-version
    stats uri /haproxyadmin?stats
    stats realm Haproxy\ Statistics
    stats auth admin:admin
    stats admin if TRUE

```

> 示例中 web页面运行在1080 端口, 记得在防火墙放行与之一致的端口
>
> 页面地址 http://<VIP>:1080/haproxyadmin?stats
>
> 默认用户名密码 admin:admin



配置完成后启动服务

```shell
systemctl start keepalived haproxy
systemctl status keepalived haproxy
```





**负载均衡配置文档参考**

keepalived [Keepalived 安装和配置详解_Direct_的博客-CSDN博客_keepalived安装与配置](https://blog.csdn.net/D1179869625/article/details/126198495)

haproxy     [kubernetes的api-server高可用配置_激情燃烧的岁月的技术博客_51CTO博客](https://blog.51cto.com/liuzhengwei521/2377280)

[3.7. Turning on Packet Forwarding and Nonlocal Binding Red Hat Enterprise Linux 7 | Red Hat Customer Portal](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/load_balancer_administration/s1-initial-setup-forwarding-vsa#:~:text=Load%20balancing%20in%20HAProxy%20and%20Keepalived%20at%20the,an%20IP%20that%20is%20not%20local%20for%20failover.)

> 资源不足时, 也可以只配置 HA 省去负载均衡, 将 keeplived 安装到 master 节点, 可使用更少的节点完成部署.
>
> 注意初始化时指定节点的 `--apiserver-advertise-address` 参数.
