---
title: 运维提升——Kubernetes（03）
tags:
  - k8s
  - 运维
id: '12035'
categories:
  - - skills
date: 2020-03-31 21:18:14
---

# Kubernetes 安装与配置

## 系统要求

软硬件

最低配置

推荐配置
<!--more-->
CPU 和内存

Master：至少 2core 和 4GB 内存  
Node:至少 4core 和 16GB

Master：4core 和 16GB。  
Node：应根据需要运行的容器数量进行配置

Linux 操作系统

基于 x86\_64 架构的各种 Linux 发行版本

Centos 7  
Red Hat Linux 7

Docker

1.9 版本以上

1.12 版本

etcd

2.0 版本及以上

3.0 版本

**这里因为没有那么多机器，所以准备采购阿里云较低配置 ECS 进行配置。** 机器信息如下： ![image.png](https://i.loli.net/2020/03/31/24mRXLSVMTjczHp.png)

角色

IP

Master

172.26.182.146

Node

172.26.170.206

Node

172.26.170.205

## 安装

本文采用 kubernetes.io 官方推荐的 kubeadm 工具安装 kubernetes 集群。

#### 检查 centos / hostname

```
# 在 Master 节点和 Node 节点都要执行

# 此处 hostname 的输出将会是该机器在 Kubernetes 集群中的节点名字
# 不能使用 localhost 作为节点的名字
hostname

# 请使用 lscpu 命令，核对 CPU 信息
# Architecture: x86_64    本安装文档不支持 arm 架构
# CPU(s):       2         CPU 内核数量不能低于 2
lscpu
```

#### 安装 Docker / kubelet

使用 root 身份在所有节点执行如下代码：

```
# 在 master 节点和 Node 节点都要执行

curl -sSL https://kuboard.cn/install-script/v1.16.0/install-kubelet.sh  sh
```

安装结果 ![image.png](https://i.loli.net/2020/03/31/bISAl3szVXfpmwZ.png)

#### 初始化 Master 节点

```bash
# 只在 master 节点执行
# 替换 172.26.182.146 为 master 节点实际 IP（请使用内网 IP）
# export 命令只在当前 shell 会话中有效，开启新的 shell 窗口后，如果要继续安装过程，请重新执行此处的 export 命令
export MASTER_IP=172.26.182.146
# 替换 apiserver.demo 为 您想要的 dnsName (不建议使用 master 的 hostname 作为 APISERVER_NAME)
export APISERVER_NAME=apiserver.demo
# Kubernetes 容器组所在的网段，该网段安装完成后，由 kubernetes 创建，事先并不存在于您的物理网络中
export POD_SUBNET=10.100.0.1/20
echo "${MASTER_IP}    ${APISERVER_NAME}" >> /etc/hosts
curl -sSL https://kuboard.cn/install-script/v1.16.0/init-master.sh  sh
```

检查 Master 初始化结果

```bash
# 只在 master 节点执行

# 执行如下命令，等待 3-10 分钟，直到所有的容器组处于 Running 状态
watch kubectl get pod -n kube-system -o wide

```

执行结果 ![image.png](https://i.loli.net/2020/04/01/nhRvUIE9yeDBzFH.png)

```bash
# 查看 master 节点初始化结果
kubectl get nodes -o wide
```

执行结果 ![image.png](https://i.loli.net/2020/04/01/JI34QPLemANaTwM.png)

#### 初始化 Node

在 master 上执行

```bash
# 只在 master 节点执行
kubeadm token create --print-join-command
```

执行结果

```
# 上条命令输出
kubeadm join apiserver.demo:6443 --token em5532.75zpnql598hyysgv     --discovery-token-ca-cert-hash sha256:db74610b703df55095ae326845f0e4add727fa4e554cab980119d9d1b3ca464f 
```

在所有 node 上执行

```bash
# 只在 worker 节点执行
export MASTER_IP=172.26.182.146
# 替换 apiserver.demo 为 您想要的 dnsName (不建议使用 master 的 hostname 作为 APISERVER_NAME)
export APISERVER_NAME=apiserver.demo

echo "${MASTER_IP}    ${APISERVER_NAME}" >> /etc/hosts

# 替换为 master 节点上 kubeadm token create 命令的输出
kubeadm join apiserver.demo:6443 --token em5532.75zpnql598hyysgv     --discovery-token-ca-cert-hash sha256:db74610b703df55095ae326845f0e4add727fa4e554cab980119d9d1b3ca464f 
```

##### 检查初始化结果

在 master 上执行

```bash
# 只在 master 节点执行
kubectl get nodes -o wide
```

结果 ![image.png](https://i.loli.net/2020/04/01/6SCK8lsu4nDkjUq.png) 至此，完成集群配置