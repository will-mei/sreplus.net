---
weight: 3
title: "02 环境准备"
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



## ipvs install
yum -y install ipvsadm ipset

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
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sysctl --system



##swap off
swapoff -a
sed -i "/swap/s/^/#/" /etc/fstab
#sed -i 's?^/dev/mapper/centos-swap*?#&?' /etc/fstab

grep -no 'vm.swappiness=0' /etc/sysctl.d/k8s.conf || echo "vm.swappiness=0" >> /etc/sysctl.d/k8s.conf

sysctl -p /etc/sysctl.d/k8s.conf



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

