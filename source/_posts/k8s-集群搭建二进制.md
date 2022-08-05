---
title: K8S 集群搭建(二进制)
tags:
  - k8s
  - 学习
  - 教程
  - 记录
id: '12425'
categories:
  - - skills
date: 2021-06-07 16:35:27
---

# 1\. 准备

本次实验为手动使用二进制方式安装`kubernetes`,为了学习和了解`kubernetes`的部署流程及相关原理。

## 1.1 实验安装的版本

*   Kubernetes v1.18.18
*   Etcd v3.2.9
*   Calico v2.6.2
*   Docker v20.10.0-ce
<!--more-->
## 1.2 机器准备

ip

role

cpu

memory

os

10.162.0.9

master

2

4

centos 7.6

10.162.0.11

node1

2

4

centos 7.6

10.162.0.12

node2

2

4

centos 7.6

**以上机器为虚拟机，均采用 root 用户进行操作（生产环境不推荐）** 下面操作如果没有特别说明在 master 还是 node 上操作默认都是所有机器上操作。

## 1.3 环境准备

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

# 添加 hosts
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

# 在 master 上按照CFSSL 工具，用来建立 TLS certificates
export CFSSL_URL="https://pkg.cfssl.org/R1.2"
wget "${CFSSL_URL}/cfssl_linux-amd64" -O /usr/local/bin/cfssl
wget "${CFSSL_URL}/cfssljson_linux-amd64" -O /usr/local/bin/cfssljson
chmod +x /usr/local/bin/cfssl /usr/local/bin/cfssljson
```

## 1.4 安装 Docker

所有节点都要安装 docker 引擎

```bash
# 安装，时间较长的话可使用其它方式安装
curl -fsSL "https://get.docker.com/"  sh
# 启动并设置开机自启
systemctl enable docker && systemctl start docker
```

```bash
# 编辑/lib/systemd/system/docker.service，在ExecStart=..上一行加入：
ExecStartPost=/sbin/iptables -I FORWARD -s 0.0.0.0/0 -j ACCEPT
# 重启 docker 服务
systemctl daemon-reload && systemctl restart docker
```

# 2\. Etcd 安装

Etcd 是 Kubernetes 最重要的一部分，Kubernetes 会将大部分信息储存于 Etcd 上，来提供给其他节点索取，以确保整个集群运作与沟通正常。

## 2.1 创建集群 CA 与 Certificates

创建 client 与 server 的各组件 certificates，并且替 Kubernetes admin user 产生 client 证书。 这部分可参考：[证书](https://kubernetes.io/zh/docs/tasks/administer-cluster/certificates/#cfssl "证书")

```bash
# 在 master 上创建 ssl 目录并完成操作
mkdir -p /opt/etcd/ssl && cd /opt/etcd/ssl
export PKI_URL="https://kairen.github.io/files/manual-v1.8/pki"
# 生成 ca-config.json
cat > ca-config.json<< EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": [
          "signing",
          "key encipherment",
          "server auth",
          "client auth"
        ],
        "expiry": "8760h"
      }
    }
  }
}
EOF
# 生成ca-csr.json
cat > ca-csr.json<< EOF 
{
    "CN": "etcd CA",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [{
        "C": "CN",
        "L": "ShangHai",
        "ST": "ShangHai"
    }]
}
EOF

# 生成 ca 秘钥
cfssl gencert -initca ca-csr.json  cfssljson -bare ca

# 创建证书申请文件,需要修改 ip 为你自己的 ip
cat > server-csr.json<< EOF 
{
    "CN": "etcd",
    "hosts": ["10.162.0.9", "10.162.0.11", "10.162.0.12"],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [{
        "C": "CN",
        "L": "ShangHai",
        "ST": "ShangHai"
    }]
}
EOF
#生成证书
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  server-csr.json  cfssljson -bare server
 # 删除 json 文件
 rm -f *.json
```

完成后如下 [![](https://i.loli.net/2021/06/07/ptgJ1zU6GIjylVM.jpg)](https://i.loli.net/2021/06/07/ptgJ1zU6GIjylVM.jpg)

## 2.2 Etcd 的安装与设置

在 master 节点上下载安装 我这里使用的是 3.2.9，新版本可以在 [**github**](https://github.com/etcd-io/etcd/releases "github")下载

```bash
mkdir -p  /opt/etcd/{bin,ssl}
cd /opt
export  ETCD_VER=v3.2.9
export GOOGLE_URL=https://storage.googleapis.com/etcd
export GITHUB_URL=https://github.com/etcd-io/etcd/releases/download
# 任选一个下载地址,我这边 google 比较快就选了 google 的地址
export DOWNLOAD_URL=${GOOGLE_URL}
wget  ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz
tar -zxvf etcd-v3.2.9-linux-amd64.tar.gz
mv etcd-v3.2.9-linux-amd64/etcd* /opt/etcd/bin && rm -rf etcd-v3.2.9-linux-amd64
```

将 master 上创建的文件拷贝到其它两个节点上

```bash
scp -r /opt/etcd/ root@k8s-node1:/opt/
scp -r /opt/etcd/ root@k8s-node2:/opt/ 

```

systemd 管理 Etcd(三台都要执行)

```bash
# ip 改为当前执行机器 ip
INTERNAL_IP="10.162.0.9"
ETCD_NAME=$(hostname -s)

# 生成 etcd.service 的 systemd 配置文件
cat <<EOF  sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/opt/etcd/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/opt/etcd/ssl/server.pem \\
  --key-file=/opt/etcd/ssl/server-key.pem \\
  --peer-cert-file=/opt/etcd/ssl/server.pem \\
  --peer-key-file=/opt/etcd/ssl/server-key.pem \\
  --trusted-ca-file=/opt/etcd/ssl/ca.pem \\
  --peer-trusted-ca-file=/opt/etcd/ssl/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster k8s-master=https://10.162.0.9:2380,k8s-node2=https://10.162.0.11:2380,k8s-node1=https://10.162.0.12:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

启动并设置开机自启

```bash
systemctl daemon-reload 
systemctl start etcd 
systemctl enable etcd
```

检查集群是否成功部署

```bash
/opt/etcd/bin/etcdctl --ca-file=/opt/etcd/ssl/ca.pem --cert-file=/opt/etcd/ssl/server.pem --key-file=/opt/etcd/ssl/server-key.pem --endpoints="https://10.162.0.9:2379,https://10.162.0.11:2379,https://10.162.0.12:2379"  cluster-health
```

出现如下内容即为成功 [![](https://i.loli.net/2021/06/08/yiKIVRNm1zOFopL.jpg)](https://i.loli.net/2021/06/08/yiKIVRNm1zOFopL.jpg)

# 3\. 部署 kubernetes Master

Master 是 Kubernetes 的大总管，主要创建apiserver、Controller manager与Scheduler来组件管理所有 Node。本步骤将下载 Kubernetes 并安装至 master1上，然后产生相关 TLS Cert 与 CA 密钥，提供给集群组件认证使用。

## 3.1 下载 kubernetes 组件

在 Master 节点上下载

```bash
# Download Kubernetes
export KUBE_URL="https://storage.googleapis.com/kubernetes-release/release/v1.8.2/bin/linux/amd64"
wget "${KUBE_URL}/kubelet" -O /usr/local/bin/kubelet
wget "${KUBE_URL}/kubectl" -O /usr/local/bin/kubectl
chmod +x /usr/local/bin/kubelet /usr/local/bin/kubectl

# Download CNI
mkdir -p /opt/cni/bin && cd /opt/cni/bin
export CNI_URL="https://github.com/containernetworking/plugins/releases/download"
wget  "${CNI_URL}/v0.6.0/cni-plugins-amd64-v0.6.0.tgz"
tar -zxvf cni-plugins-amd64-v0.6.0.tgz
```

## 3.2 创建集群 CA 与 Certificates

在 master 上执行

```bash
# 创建目录
mkdir -p /opt/kubernetes/pki && cd /opt/kubernetes/pki
# 自签证书颁发机构 （CA）
cat > ca-config.json<< EOF
{
    "signing": {
        "default": {
            "expiry": "87600h"
        },
        "profiles": {
            "kubernetes": {
                "expiry": "87600h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                    ]
            }
        }
    }
}
EOF

cat > ca-csr.json<< EOF
{
    "CN": "kubernetes",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [{
        "C": "CN",
        "L": "ShangHai",
        "ST": "ShangHai",
        "O": "k8s",
        "OU": "System"
    }]
}
EOF
# 生成证书
cfssl gencert -initca ca-csr.json  cfssljson -bare ca -

#使用自签 CA 签发 kube-apiserver HTTPS 证书
cat > server-csr.json<< EOF 
{
    "CN": "kubernetes",
    "hosts": [
        "10.0.0.1", 
        "127.0.0.1", 
        "10.162.0.9", 
        "10.162.0.11", 
        "10.162.0.12", 
         "kubernetes", 
        "kubernetes.default",
        "kubernetes.default.svc",
        "kubernetes.default.svc.cluster",
        "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [{
        "C": "CN",
        "L": "ShangHai",
        "ST": "ShangHai",
        "O": "k8s",
        "OU": "System"
    }]
}
EOF

# 生成证书
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes server-csr.json  cfssljson -bare server
```

## 3.3 下载二进制文件

下载地址：[kubernetes v1.18.18](https://dl.k8s.io/v1.18.18/kubernetes-server-linux-amd64.tar.gz "kubernetes v1.18.18") 其它版本：[github](https://github.com/kubernetes/kubernetes/releases)

```bash
# 创建目录
mkdir -p /opt/kubernetes/{bin,cfg,ssl,logs,package}&&cd /opt/kubernetes/package
# 下载
wget https://dl.k8s.io/v1.18.18/kubernetes-server-linux-amd64.tar.gz
# 解压
tar zxvf kubernetes-server-linux-amd64.tar.gz
# 拷贝至安装目录
cd kubernetes/server/bin
cp kube-apiserver kube-scheduler kube-controller-manager /opt/kubernetes/bin
cp kubectl /usr/bin/
```

## 3.4 部署 kube-apiserver

创建配置文件

```bash
cat > /opt/kubernetes/cfg/kube-apiserver.conf << EOF
KUBE_APISERVER_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/opt/kubernetes/logs \\
--etcd-servers=https://10.162.0.9:2379,https://10.162.0.11:2379,https://10.162.0.12:2379 \\ 
--bind-address=10.162.0.9 \\ 
--secure-port=6443 \\
--advertise-address=10.162.0.9 \\
--allow-privileged=true \\
--service-cluster-ip-range=10.0.0.0/24 \\
--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,ResourceQuota,NodeRestriction \\
--authorization-mode=RBAC,Node \\
--enable-bootstrap-token-auth=true \\
--token-auth-file=/opt/kubernetes/cfg/token.csv \\
--service-node-port-range=30000-32767 \\
--kubelet-client-certificate=/opt/kubernetes/ssl/server.pem \\
--kubelet-client-key=/opt/kubernetes/ssl/server-key.pem \\
--tls-cert-file=/opt/kubernetes/ssl/server.pem \\
--tls-private-key-file=/opt/kubernetes/ssl/server-key.pem \\
--client-ca-file=/opt/kubernetes/ssl/ca.pem \\
--service-account-key-file=/opt/kubernetes/ssl/ca-key.pem \\
--etcd-cafile=/opt/etcd/ssl/ca.pem \\
--etcd-certfile=/opt/etcd/ssl/server.pem \\
--etcd-keyfile=/opt/etcd/ssl/server-key.pem \\
--audit-log-maxage=30 \\
--audit-log-maxbackup=3 \\
--audit-log-maxsize=100 \\
--audit-log-path=/opt/kubernetes/logs/k8s-audit.log"
EOF
```

```text
–logtostderr：启用日志
—v：日志等级
–log-dir：日志目录
–etcd-servers：etcd 集群地址
–bind-address：监听地址
–secure-port：https 安全端口
–advertise-address：集群通告地址
–allow-privileged：启用授权
–service-cluster-ip-range：Service 虚拟 IP 地址段
–enable-admission-plugins：准入控制模块
–authorization-mode：认证授权，启用 RBAC 授权和节点自管理
–enable-bootstrap-token-auth：启用 TLS bootstrap 机制
–token-auth-file：bootstrap token 文件
–service-node-port-range：Service nodeport 类型默认分配端口范围
–kubelet-client-xxx：apiserver 访问 kubelet 客户端证书
–tls-xxx-file：apiserver https 证书
–etcd-xxxfile：连接 Etcd 集群证书
–audit-log-xxx：审计日志
```

拷贝证书

```bash
cp /opt/kubernetes/pki/*pem /opt/kubernetes/ssl/
```

## 3.5 启用 TLS Bootstrapping 机制

TLS Bootstraping：Master apiserver 启用 TLS 认证后，Node 节点 kubelet 和 kube- proxy 要与 kube-apiserver 进行通信，必须使用 CA 签发的有效证书才可以，当 Node 节点很多时，这种客户端证书颁发需要大量工作，同样也会增加集群扩展复杂度。为了 简化流程，Kubernetes 引入了 TLS bootstraping 机制来自动颁发客户端证书，kubelet 会以一个低权限用户自动向 apiserver 申请证书，kubelet 的证书由 apiserver 动态签署。 所以强烈建议在 Node 上使用这种方式，目前主要用于 kubelet，kube-proxy 还是由我 们统一颁发一个证书。 TLS bootstraping 工作流程： [![](https://i.loli.net/2021/06/08/3NDhpqL9VEobrJI.jpg)](https://i.loli.net/2021/06/08/3NDhpqL9VEobrJI.jpg) 创建上述配置文件中 token 文件：

```bash
cat > /opt/kubernetes/cfg/token.csv << EOF
c47ffb939f5ca36231d9e3121a252940,kubelet-bootstrap,10001,system:node-bootstrapper
EOF
```

格式：token，用户名，UID，用户组

## 3.6 systemd 管理 apiserver

```bash
cat > /usr/lib/systemd/system/kube-apiserver.service << EOF
[Unit]
Description=Kubernetes API Server Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=/opt/kubernetes/cfg/kube-apiserver.conf
ExecStart=/opt/kubernetes/bin/kube-apiserver \$KUBE_APISERVER_OPTS
Restart=on-failure
[Install]
WantedBy=multi-user.target
EOF
```

启动并设置开机自启

```bash
systemctl daemon-reload
systemctl start kube-apiserver
systemctl enable kube-apiserver
```

出现报错：

```bash
E0608 19:41:30.601867    1912 controller.go:152] Unable to remove old endpoints from kubernetes service: StorageError: key not found, Code: 1, Key: /registry/masterleases/10.162.0.9, ResourceVersion: 0, AdditionalErrorMsg:
```

花了一下午时间还没有解决这个报错，有点小崩溃