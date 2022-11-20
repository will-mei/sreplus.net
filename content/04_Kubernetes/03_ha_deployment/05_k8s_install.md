---
title: "04_k8s_install"
date: 2022-09-08T16:28:58+08:00
draft: false
---

# 下载镜像

kubeadm 自动下载

```shell
kubeadm config images pull
```



查看需要下载的镜像

```shell
kubeadm config images list --kubernetes-version v1.25.4
```

输出参考

```
registry.k8s.io/kube-apiserver:v1.25.4
registry.k8s.io/kube-controller-manager:v1.25.4
registry.k8s.io/kube-scheduler:v1.25.4
registry.k8s.io/kube-proxy:v1.25.4
registry.k8s.io/pause:3.8
registry.k8s.io/etcd:3.5.5-0
registry.k8s.io/coredns/coredns:v1.9.3
```

手动拉取镜像

```bash
#!/bin/bash

image_repository=registry.aliyuncs.com/google_containers
kube_version=v1.25.4

for img in $(kubeadm config images list --kubernetes-version $kube_version); do
    nerdctl pull $image_repository/${img##*/}
done
```



# 初始化集群

集群的初始化主要有命令行初始化和配置文件初始化两种.



## kubeadm 单机

### 命令行方式

```shell
kubeadm init \
--apiserver-bind-port=6443 --kubernetes-version=v1.25.4 \
--apiserver-advertise-address=192.168.129.197 \
--pod-network-cidr=10.100.0.0/16 \
--service-cidr=10.200.0.0/16 \
--service-dns-domain=cluster.local \
--image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers \
--ignore-preflight-errors=swap
```

这里所有指定的网络不要和你现有的网络冲突, `apiserver-advertise-address` 使用本机地址.

Pod 网络是一个私网的网段, 尽量分配一个相对较大的网段, 以便容纳足够多的 pod.



### 配置文件方式

将默认初始化配置输出到文件, 然后进行修改

```shell
kubeadm config print init-defaults > kubeadm-init.yaml
```

示例

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.129.197
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock
  imagePullPolicy: IfNotPresent
  name: 192.168.129.197
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: 1.25.4
networking:
  dnsDomain: cluster.local
  podSubnet: 10.100.0.0/16
  serviceSubnet: 10.200.0.0/16
scheduler: {}
```

执行配置文件

```shell
kubeadm init --config kubeadm-init.yaml

# mkdir -p $HOME/.kube
# sudo cp -r /etc/kubernetes/admin.conf $HOME/.kube/config
# sudo chown ${uid}:${gid} $HOME/.kube/config
```



## kubeadm 高可用

### 命令行方式

```shell
kubeadm init \
--apiserver-bind-port=6443 --kubernetes-version=v1.25.4 \
--apiserver-advertise-address=192.168.129.197 \
--control-plane-endpoint=192.168.129.3 \
--pod-network-cidr=10.100.0.0/16 \
--service-cidr=10.200.0.0/16 \
--service-dns-domain=pinkcloud.local \
--image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers \
--ignore-preflight-errors=swap
```

与单机的主要不同点在于 `control-plane-endpoint` 使用的是 `VIP` .

负载均衡需要另外用 `Keepalived` 和 `haproxy` 配置好, 公有云购买 SLB 也可以.



配置文件方式

```yaml
# kubeadm config print init-defaults > kubeadm-init.yaml
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.129.197
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock
  imagePullPolicy: IfNotPresent
  name: 192.168.129.197  # 当前节点
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: 192.168.129.3:6443  # VIP地址
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers  # 阿里
kind: ClusterConfiguration
kubernetesVersion: 1.25.4
networking:
  dnsDomain: cluster.local
  podSubnet: 10.100.0.0/16      # pod
  serviceSubnet: 10.200.0.0/16  # service
scheduler: {}
```

执行配置文件

```shell
kubeadm init --config kubeadm-init.yaml

# mkdir -p $HOME/.kube
# sudo cp -r /etc/kubernetes/admin.conf $HOME/.kube/config
# sudo chown ${uid}:${gid} $HOME/.kube/config
```



## 执行结果

初始化成功后, 输出内容参考

```
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:
                                                                                                                                                                                        export KUBECONFIG=/etc/kubernetes/admin.conf                                                                                                                                                                                                                                                                                                                              You should now deploy a pod network to the cluster.                                                                                                                                   Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:                                                                                                             https://kubernetes.io/docs/concepts/cluster-administration/addons/                                                                                                                                                                                                                                                                                                        You can now join any number of control-plane nodes by copying certificate authorities                                                                                                 and service account keys on each node and then running the following as root:
                                                                                                                                                                                        kubeadm join 192.168.129.3:6443 --token c0eb6t.4r67ceebwspvckdl \                                                                                                                           --discovery-token-ca-cert-hash sha256:52bec8a9ba3eda56d2065112b7cfea1bfe011e077d3cf324e6df8c6e636714e1 \                                                                              --control-plane                                                                                                                                                                                                                                                                                                                                                     Then you can join any number of worker nodes by running the following on each as root:                                                                                                                                                                                                                                                                                      kubeadm join 192.168.129.3:6443 --token c0eb6t.4r67ceebwspvckdl \                                                                                                                             --discovery-token-ca-cert-hash sha256:52bec8a9ba3eda56d2065112b7cfea1bfe011e077d3cf324e6df8c6e636714e1
```



查看集群初始化配置文件

```shell
kubectl -n kube-system get cm kubeadm-config -o yaml
```

结果参考

```yaml
# kubectl -n kube-system get cm kubeadm-config -o yaml
apiVersion: v1
data:
  ClusterConfiguration: |
    apiServer:
      extraArgs:
        authorization-mode: Node,RBAC
      timeoutForControlPlane: 4m0s
    apiVersion: kubeadm.k8s.io/v1beta3
    certificatesDir: /etc/kubernetes/pki
    clusterName: kubernetes
    controlPlaneEndpoint: 192.168.129.3:6443
    controllerManager: {}
    dns: {}
    etcd:
      local:
        dataDir: /var/lib/etcd
    imageRepository: registry.aliyuncs.com/google_containers
    kind: ClusterConfiguration
    kubernetesVersion: v1.25.4
    networking:
      dnsDomain: cluster.local
      podSubnet: 10.100.0.0/16
      serviceSubnet: 10.200.0.0/16
    scheduler: {}
kind: ConfigMap
metadata:
  creationTimestamp: "2022-11-20T16:39:46Z"
  name: kubeadm-config
  namespace: kube-system
  resourceVersion: "200"
  uid: 9d231fb0-574f-4be5-9cd7-7774508ad48b
```



## 初始化问题排查

初始化过程的日志可以在系统日志查看到. 

如果初始化异常, 通常需要仔细检查

1. kubelet 服务状态, 并仔细核对配置文件中的信息
2. 远端仓库信息, containerd 仓库配置信息, 初始化配置或命令中的仓库配置信息, 域名,路径, 名称, 端口
3. 远端仓库连接状态, 镜像拉取状态, 执行手动拉取测试.
4. 所有的网络配置, IP LB 状态

排除问题后可以使用 `kubeadm reset` 清理, 然后重新初始化



## 故障参考

若 pod 启动异常, kubelet 无法从 endpoint 访问到查询到节点信息, 会一直报 `err="node \"xxx\" not found"` 错误.

> 例如将 `rgoogle_containers` 写成了 `google-containers` 结果即使镜像已经成功被拉取到本地也始终无法启动服务, 提示 `failed to resolve reference` 镜像 not found.

```
Nov 20 11:36:13 wkube1 kubelet: E1120 11:36:13.304482    3443 kubelet.go:2448] "Error getting node" err="node \"wkube1\" not found"
Nov 20 11:36:13 wkube1 kubelet: E1120 11:36:13.405334    3443 kubelet.go:2448] "Error getting node" err="node \"wkube1\" not found"
Nov 20 11:36:13 wkube1 kubelet: E1120 11:36:13.506364    3443 kubelet.go:2448] "Error getting node" err="node \"wkube1\" not found"
Nov 20 11:36:13 wkube1 containerd: time="2022-11-20T11:36:13.555870635-05:00" level=info msg="trying next host - response was http.StatusNotFound" host=registry.aliyuncs.com
Nov 20 11:36:13 wkube1 containerd: time="2022-11-20T11:36:13.558600202-05:00" level=error msg="RunPodSandbox for &PodSandboxMetadata{Name:kube-scheduler-wkube1,Uid:11830119453a26c0a0
9e087b96f2b32a,Namespace:kube-system,Attempt:0,} failed, error" error="rpc error: code = NotFound desc = failed to get sandbox image \"registry.aliyuncs.com/google-containers/pause:3
.8\": failed to pull image \"registry.aliyuncs.com/google-containers/pause:3.8\": failed to pull and unpack image \"registry.aliyuncs.com/google-containers/pause:3.8\": failed to res
olve reference \"registry.aliyuncs.com/google-containers/pause:3.8\": registry.aliyuncs.com/google-containers/pause:3.8: not found"
Nov 20 11:36:13 wkube1 kubelet: E1120 11:36:13.559160    3443 remote_runtime.go:222] "RunPodSandbox from runtime service failed" err="rpc error: code = NotFound desc = failed to get
sandbox image \"registry.aliyuncs.com/google-containers/pause:3.8\": failed to pull image \"registry.aliyuncs.com/google-containers/pause:3.8\": failed to pull and unpack image \"reg
istry.aliyuncs.com/google-containers/pause:3.8\": failed to resolve reference \"registry.aliyuncs.com/google-containers/pause:3.8\": registry.aliyuncs.com/google-containers/pause:3.8
: not found"
```

处理办法, 修改 containerd 镜像仓库信息, 并核对初始化仓库信息都无误后, 重新执行初始化问题解决.