---
title: K8S 集群搭建(kubeadm)
tags:
  - k8s
  - 教程
  - 记录
id: '12401'
categories:
  - - skills
date: 2021-06-02 15:00:12
---

# 1\. 搭建 k8s 环境平台规划

单 master 集群 [![](https://i.loli.net/2021/06/02/DpF3i9LPKOBrRfa.jpg)](https://i.loli.net/2021/06/02/DpF3i9LPKOBrRfa.jpg) 多 master 集群 [![](https://i.loli.net/2021/06/02/yviN3Qut594kBrI.jpg)](https://i.loli.net/2021/06/02/yviN3Qut594kBrI.jpg)

# 2\. 准备

## 2.1 机器准备

测试环境

角色

cpu

内存

硬盘

master

2 核

4g

20g

node

4 核

8g

40g

生产环境

角色

cpu

内存

硬盘

master

4 核

8g

20g

node

8 核

16g

100g

实验机器准备： [![](https://i.loli.net/2021/06/03/jbTXZfdApIomyQn.jpg)](https://i.loli.net/2021/06/03/jbTXZfdApIomyQn.jpg)

## 2.2 操作系统初始化

```bash
# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

# 关闭 selinux
sed -i 's/enforcing/disable/' /etc/selinux/config
setenforce 0

# 关闭 swap
swapoff -a
sed -ri 's/.*swap.*/#&/' /etc/fstab

# 根据规划设置主机名
hostnamectl set-hostname <hostname>

# 在 master 添加 hosts
cat >> /etc/hosts <<EOF
10.162.0.9  k8s-master
10.162.0.11 k8s-node2
10.162.0.12 k8s-node1
EOF

# 将桥接的 IPv4 流量传递到 iptables 的链
# 要求iptables不对bridge的数据进行处理
cat > /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-ip6tables =1
net.bridge.bridge-nf-call-iptables =1
EOF

sysctl --system

# 同步时间
yum install ntpdate -y 
ntpdate time.windows.com
```

# 3\. kubeadm搭建

## 3.1 所有节点安装 Docker/kubeadm/kubelet

kubernetes 默认 CRI（容器运行时）为 Docker，因此先安装 Docker

### 3.1.1 安装 Docker

```bash
yum install wget -y 
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
yum install docker-ce-18.06.1.ce-3.el7
systemctl enable docker && systemctl start docker
docker -version #检查安装是否成功
```

修改仓库地址

```bash
mkdir -p /etc/docker
cat > /etc/docker/daemon.json <<EOF
{
    "registry-mirrors":["https://registry.docker-cn.com"]
}
EOF
```

启动

```bash
systemctl   restart docker
systemctl  enable docker # 开机自启
```

### 3.1.2 添加阿里云 yum 软件源

```bash
cat >/etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enable=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

### 3.1.3 安装 kubeadm,kubelet 和 kubectl

```bash
yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0
systemctl enable kubelet
```

## 3.2 部署 Kubernetes Master

在 Master 上执行

```bash
kubeadm init 
    --apiserver-advertise-address=10.162.0.9 
    --image-repository registry.aliyuncs.com/google_containers 
    --kubernetes-version v1.18.0 
    --service-cidr=10.96.0.0/12 
    --pod-network-cidr=10.244.0.0/16


# 这里会执行三到四分钟
```

成功后打印如下 [![](https://i.loli.net/2021/06/03/ienv6f7Wr2wNc4a.jpg)](https://i.loli.net/2021/06/03/ienv6f7Wr2wNc4a.jpg) 指定镜像仓库地址为阿里云

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin/conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## 3.3 加入 Kubernetes Node

在 node 上执行

```bash
# init 成功之后打印出的语句
kubeadm join 10.162.0.9:6443 --token x644b0.j0k1jplo1lej0ujk 
    --discovery-token-ca-cert-hash sha256:e74a03c6bca17e9815bb59e79eba9060c64e65efb6841f4ccee63ab49ecf60ae
```

默认token 有效期 24 小时，后期需要重新生成执行以下

```bash
kubeadm token create --print-join-command
```

加入成功在 master 上执行`kubectl get nodes`显示 [![](https://i.loli.net/2021/06/03/K8pwZDs6jVLBve9.jpg)](https://i.loli.net/2021/06/03/K8pwZDs6jVLBve9.jpg)

## 3.4 部署 CNI 网络插件

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl get pods -n kube-system
```

完成如下 [![](https://i.loli.net/2021/06/03/I49zfNjnpF6GUPQ.jpg)](https://i.loli.net/2021/06/03/I49zfNjnpF6GUPQ.jpg) 这时再执行`kubectl get nodes`节点的 status 就会变为 Ready [![](https://i.loli.net/2021/06/03/ague9wEJ8hTyBP6.jpg)](https://i.loli.net/2021/06/03/ague9wEJ8hTyBP6.jpg)

## 3.5 测试 kubernetes 集群

部署 nginx 进行测试

```bash
# 拉取镜像运行
kubectl create deployment nginx --image=nginx
# 暴露端口
kubectl expose deployment nginx --port=80 --type=NodePort
# 查看
kubectl get services
```

[![](https://i.loli.net/2021/06/03/t1SVzUrd6MvQFX2.jpg)](https://i.loli.net/2021/06/03/t1SVzUrd6MvQFX2.jpg) 访问 `nodeip:port`显示 nginx 页面即为成功 [![](https://i.loli.net/2021/06/03/Sg3MErRw1QZVH27.jpg)](https://i.loli.net/2021/06/03/Sg3MErRw1QZVH27.jpg)