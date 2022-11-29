---
weight: 4
title: "04 CRI 安装"
date: 2022-09-08T16:23:50+08:00
draft: false
---



容器运行时 containerd 与 cri-o 运行环境安装, 系统环境 CentOS 或 Rocky



# containerd 

1. 节点安装 containerd
2. 客户端工具 nerdctl
3. 部署 runc
4. 部署 CNI
5. 启动容器测试



## 软件源安装

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



## 二进制安装

1. 从 [containerd release](https://github.com/containerd/containerd/releases) 下载对应平台的二进制包. 二进制包文件名: containerd-<版本号>-linux-amd64.tar.gz

2. 解压二进制包, 将可执行文件拷贝到 `/usr/local/bin/` 目录下

   ```shell
   tar -xf containerd-1.6.10-linux-amd64.tar.gz
   ```

   解压后的目录结构

   ```
   # tree bin
   bin/
   ├── containerd
   ├── containerd-shim
   ├── containerd-shim-runc-v1
   ├── containerd-shim-runc-v2
   ├── containerd-stress
   └── ctr
   
   0 directories, 6 files
   ```

   拷贝可执行文件

   ```shell
   cp bin/* /usr/local/bin/
   ```

   测试运行可执行文件

   ```shell
   containerd --version
   ```

3. 创建 containerd 服务 service 文件, 参考官方的配置文件, 注意可执行文件的路径是否匹配.

   CentOS 默认路径 `/usr/lib/systemd/system/containerd.service` 

   ```ini
   # Copyright The containerd Authors.
   #
   # Licensed under the Apache License, Version 2.0 (the "License");
   # you may not use this file except in compliance with the License.
   # You may obtain a copy of the License at
   #
   #     http://www.apache.org/licenses/LICENSE-2.0
   #
   # Unless required by applicable law or agreed to in writing, software
   # distributed under the License is distributed on an "AS IS" BASIS,
   # WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   # See the License for the specific language governing permissions and
   # limitations under the License.
   
   [Unit]
   Description=containerd container runtime
   Documentation=https://containerd.io
   After=network.target local-fs.target
   
   [Service]
   ExecStartPre=-/sbin/modprobe overlay
   ExecStart=/usr/bin/containerd
   
   Type=notify
   Delegate=yes
   KillMode=process
   Restart=always
   RestartSec=5
   # Having non-zero Limit*s causes performance problems due to accounting overhead
   # in the kernel. We recommend using cgroups to do container-local accounting.
   LimitNPROC=infinity
   LimitCORE=infinity
   LimitNOFILE=infinity
   # Comment TasksMax if your systemd version does not supports it.
   # Only systemd 226 and above support this version.
   TasksMax=infinity
   OOMScoreAdjust=-999
   
   [Install]
   WantedBy=multi-user.target
   ```

4. 调整 containerd 配置文件 `/etc/containerd/config.toml` 

   生成一份默认配置文件

   ```shell
   # mkdir /etc/containerd
   
   containerd config defautl > /etc/containerd/config.toml
   ```

   修改 sandbox 镜像地址, 该镜像用于为 pod 提供底层网络.

   ```toml
   # 约 61 行
   sandbox_image = "registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.8"
   ```

   修改基础镜像仓库 "docker.io", "k8s.gcr.io" 到国内镜像仓库

   ```toml
   # 约 153 行
   [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
     [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
       endpoint = ["https://dockerhub.mirrors.nwafu.edu.cn"]
     [plugins."io.containerd.grpc.v1.cri".registry.mirrors."k8s.gcr.io"]
       endpoint = ["https://registry.aliyuncs.com/k8sxio"]
   ```

   阿里云用户, 打开 [官方镜像加速](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors) 可以使用阿里云为用户提供的镜像加速服务.

5. 安装 nerdctl 工具, 从 [nerdctl release](https://github.com/containerd/nerdctl/releases) 下载

   ```shell
   wget https://github.com/containerd/nerdctl/releases/download/v1.0.0/nerdctl-1.0.0-linux-amd64.tar.gz
   ```

   配置 nerdctl 默认 namespace 为 "k8s.io", 以管理 k8s pod

   ```shell
   mkdir /etc/nerdctl/
   
   cat >/etc/nerdctl/nerdctl.toml <<EOF
   namespace         = "k8s.io"
   debug             = false
   debug_full        = false
   insecure_registry = true
   EOF
   ```

6. 安装 runc, 下载地址: [runc release](https://github.com/opencontainers/runc/releases) 

   部署可执行文件, 注意目标文件名必须是 `runc`

   ```shell
   # 目录和文件名是默认的
   cp runc.amd64 /usr/bin/runc
   ```

7. 部署 cni , 下载地址: [containernetworking plugins release](https://github.com/containernetworking/plugins/releases) 

   ```shell
   wget https://github.com/containernetworking/plugins/releases/download/v1.1.1/cni-plugins-linux-amd64-v1.1.1.tgz
   
   mkdir /opt/cni/
   tar -xvf cni-plugins-linux-amd64-v1.1.1.tgz -C /opt/cni/
   ```

   创建配置文件(可选)

   ```shell
   mkdir -p /etc/cni/net.d
   
   cat > /etc/cni/net.d/10-flannel.conflist << EOF
   {
     "name": "cbr0",
     "plugins": [
       {
         "type": "flannel",
         "delegate": {
           "hairpinMode": true,
           "isDefaultGateway": true
         }
       },
       {
         "type": "portmap",
         "capabilities": {
           "portMappings": true
         }
       }
     ]
   }
   EOF
   ```

8. 测试容器创建

   注意, `-d` 参数现在还不能与 `-i` `-t` 连用,  后续版本可能会修复, 但是不影响我们使用.

   ```shell
   nerdctl run -d -p 80:80 --name test1 nginx:stable
   ```

   访问宿主机 80 端口查看到 nginx 欢迎页即为容器创建成功.

   测试完成后将容器删除即可

   ```shell
   nerdctl rm -fv test1
   ```



<br>

# CRI-O

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
