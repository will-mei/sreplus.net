---
weight: 3
title: "03 环境准备"
date: 2022-09-02T16:24:49+08:00
draft: false
---

### 基础集群环境配置

服务集群通常需要一些基础服务准备

#### 配置清单

1. 网络连通性
2. 系统登陆配置
3. 防火墙规则, SELinux(按需)
4. 时区时间同步
5. DNS 集群节点的主机名解析 (DNS服务解析/本地解析)
6. 软件源配置 (软件包,镜像仓库)
7. 操作系统升级 (系统软件,内核5.14+)
8. 内核启动参数优化(模块加载, 网络配置, 句柄限制, CPU burst)
9. 禁用 swap
10. 其他



#### 快速操作

以下是一份 CentOS/Rocky 的配置脚本示例

```shell
#!/bin/bash

## node env
grep -q "LC_ALL=" /etc/profile || echo "export LC_ALL=en_US.UTF-8"  >>  /etc/profile
source /etc/profile

# firewalld
systemctl disable firewalld ; systemctl stop firewalld
# selinux
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config


# yum -y install bash-completion conntrack jq nfs-common ceph-common rsync socat

## ipvs install
yum -y install ipvsadm ipset

# 内核模块加载
#cat > /etc/smodules-load.d/ipvs.modules <<EOF
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
modprobe -- br_netfilter
EOF

chmod 755 /etc/sysconfig/modules/ipvs.modules
bash /etc/sysconfig/modules/ipvs.modules

lsmod | grep -e ip_vs -e nf_conntrack_ipv4
lsmod | grep br_netfilter || modprobe br_netfilter


## net settings for k8s
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-arptables = 1
net.ipv4.tcp_tw_reuse = 0
net.core.somaxconn = 32768
net.netfilter.nf_conntrack_max = 1000000
vm.swappiness = 0
vm.max_map_count = 655360
fs.file-max = 6553600
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_intvl = 30
net.ipv4.tcp_keepalive_probes = 10
EOF


#sysctl -p /etc/sysctl.d/k8s.conf
sysctl --system

##swap off
swapoff -a
sed -i "/swap/s/^/#/" /etc/fstab
#sed -i 's?^/dev/mapper/centos-swap*?#&?' /etc/fstab


# limits 资源限制
# vim /etc/security/limits.conf
cat >> /etc/security/limits.conf <<EOF
root        soft    core        ulimited
root        hard    core        ulimited
root        soft    nproc       1000000
root        hard    nproc       1000000
root        soft    nofile      1000000
root        hard    nofile      1000000
root        soft    memlock     32000
root        hard    memlock     32000
root        soft    msgqueue    8192000
root        hard    msgqueue    8192000
EOF


## check init
python -c "print('{0}|firewalld|{1}'.format('-'*40, '-'*40))"
systemctl status firewalld |grep Active

python -c "print('{0}|selinux|{1}'.format('-'*40, '-'*40))"
getenforce

python -c "print('{0}|ipvs|{1}'.format('-'*40, '-'*40))"
lsmod | grep -e ip_vs -e nf_conntrack_ipv4

python -c "print('{0}|br netfilter|{1}'.format('-'*40, '-'*40))"
lsmod | grep br_netfilter

python -c "print('{0}|swap stat|{1}'.format('-'*40, '-'*40))"
free -m

python -c "print('{0}|k8s.conf|{1}'.format('-'*40, '-'*40))"
cat /etc/sysctl.d/k8s.conf

```

重启服务器, 检查配置生效状况.