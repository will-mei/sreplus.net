---
Weight: 6
title: "06 添加节点与网络"
date: 2022-11-21T16:32:13+08:00
draft: false
---

# 添加 Node 节点

运行 k8s 初始化之后提示的节点添加命令

```shell
kubeadm join 192.168.129.3:6443 --token c0eb6t.4r67ceebwspvckdl \
        --discovery-token-ca-cert-hash sha256:52bec8a9ba3eda56d2065112b7cfea1bfe011e077d3cf324e6df8c6e636714e1
```

输出结果

```shell
[preflight] Running pre-flight checks
        [WARNING Hostname]: hostname "wkube5" could not be reached
        [WARNING Hostname]: hostname "wkube5": lookup wkube5 on 192.168.129.1:53: server misbehaving
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

```



# 添加 Master 节点

控制平面中的节点需要共享Kubernetes CA、Etcd CA和Kube-proxy CA等的证书信息和私钥信息.

做法有两种：

1. 手动将第一个控制平面上生成的证书及私钥文件添加到其他master节点
2. 借助kubeadm init phase命令

## 生成用于加入控制平面的 secret

在控制节点运行

```shell
kubeadm init phase upload-certs --upload-certs
```

输出结果

```
[upload-certs] Using certificate key:
9fc1e92d9c27f8f6673370e7279a7d6d3d5ad04f71299d7ecca5b7c499b46d10
```

该证书通常两小时以后即失效.



## 添加 master 节点

在 k8s 初始化之后的添加 master 节点命令后面添加上 secret 信息. 使用 `--certificated-key` 选项. 然后将生成的 secret 添加在后面.

```shell
  kubeadm join 192.168.129.3:6443 --token c0eb6t.4r67ceebwspvckdl \
        --discovery-token-ca-cert-hash sha256:52bec8a9ba3eda56d2065112b7cfea1bfe011e077d3cf324e6df8c6e636714e1 \
        --control-plane --certificate-key 9fc1e92d9c27f8f6673370e7279a7d6d3d5ad04f71299d7ecca5b7c499b46d10
```

输出结果

```

This node has joined the cluster and a new control plane instance was created:

* Certificate signing request was sent to apiserver and approval was received.
* The Kubelet was informed of the new secure connection details.
* Control plane label and taint were applied to the new node.
* The Kubernetes control plane instances scaled up.
* A new etcd member was added to the local/stacked etcd cluster.

To start administering your cluster from this node, you need to run the following as a regular user:

        mkdir -p $HOME/.kube
        sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config

Run 'kubectl get nodes' to see this node join the cluster.

```

参照上面的提示, 初始化 kuebectl 配置

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

查看节点状态

```
# kubectl get node -o wide
NAME     STATUS   ROLES           AGE     VERSION   INTERNAL-IP       EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION               CONTAINER-RUNTIME
wkube1   Ready    control-plane   23h     v1.25.4   192.168.129.197   <none>        CentOS Linux 7 (Core)   5.19.3-1.el7.elrepo.x86_64   containerd://1.6.10
wkube2   Ready    control-plane   4m19s   v1.25.4   192.168.129.196   <none>        CentOS Linux 7 (Core)   5.19.3-1.el7.elrepo.x86_64   containerd://1.6.10
wkube3   Ready    control-plane   73s     v1.25.4   192.168.129.151   <none>        CentOS Linux 7 (Core)   5.19.3-1.el7.elrepo.x86_64   containerd://1.6.10
wkube4   Ready    <none>          38m     v1.25.4   192.168.129.191   <none>        CentOS Linux 7 (Core)   5.19.3-1.el7.elrepo.x86_64   containerd://1.6.10
wkube5   Ready    <none>          30m     v1.25.4   192.168.129.129   <none>        CentOS Linux 7 (Core)   5.19.3-1.el7.elrepo.x86_64   containerd://1.6.10
```



# 添加网络插件

为了配置扩展和功能考虑, 建议使用 calico 插件.



下载 calico 插件

```shell
curl https://raw.githubusercontent.com/projectcalico/calico/v3.24.5/manifests/calico.yaml -O

# or get latest

wget https://docs.projectcalico.org/manifests/calico.yaml
```



修改网络配置 `CALICO_IPV4POOL_CIDR` 应当跟部署时指定的 pod 网络 CIDR 一致. 

```yaml
4542             # Set MTU for the Wireguard tunnel device.
4543             - name: FELIX_WIREGUARDMTU
4544               valueFrom:
4545                 configMapKeyRef:
4546                   name: calico-config
4547                   key: veth_mtu
4548             # The default IPv4 pool to create on startup if none exists. Pod IPs will be
4549             # chosen from this range. Changing this value after installation will have
4550             # no effect. This should fall within `--cluster-cidr`.
4551             - name: CALICO_IPV4POOL_CIDR
4552               value: "10.100.0.0/16"
4553             # Disable file logging so `kubectl logs` works.
4554             - name: CALICO_DISABLE_FILE_LOGGING
4555               value: "true"
```

修改过程中请务必注意保持 yaml 格式的对齐, 不允许使用 tab 键代替空格.

输出结果

```
# kubectl apply -f calico.yaml
poddisruptionbudget.policy/calico-kube-controllers configured
serviceaccount/calico-kube-controllers unchanged
serviceaccount/calico-node unchanged
configmap/calico-config unchanged
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org configured
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org configured
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org configured
customresourcedefinition.apiextensions.k8s.io/caliconodestatuses.crd.projectcalico.org configured
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org configured
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org configured
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org configured
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org configured
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org configured
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org configured
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org configured
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org configured
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org configured
customresourcedefinition.apiextensions.k8s.io/ipreservations.crd.projectcalico.org configured
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org configured
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org configured
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org configured
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers unchanged
clusterrole.rbac.authorization.k8s.io/calico-node unchanged
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers unchanged
clusterrolebinding.rbac.authorization.k8s.io/calico-node unchanged
daemonset.apps/calico-node created
deployment.apps/calico-kube-controllers created
```

