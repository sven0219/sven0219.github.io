---
title: Etcd集群安装-3.4.3
tags:
  - Etcd
  - k8s
  - 运维
id: '12320'
categories:
  - - skills
date: 2021-01-18 09:27:11
---

## 集群机器列表

IP地址

hostname

10.158.1.9

k8s-master-03

10.158.1.8

k8s-master-02

10.158.1.7

k8s-master-01

分别在三台机器上，执行如下操作

## 1.配置CA证书

```
sudo mkdir /etc/etcd /var/lib/etcd
sudo mv ca.pem kubernetes.pem kubernetes-key.pem /etc/etcd
```

> sudo rm -rf /etc/etcd /var/lib/etcd

## 2.下载Etcd二进制安装包

我们安装的`Etcd`版本是`v3.4.300`

```
wget https://github.com/etcd-io/etcd/releases/download/v3.4.3/etcd-v3.4.3-linux-amd64.tar.gz
```

解压并把`Etcd`二进制包拷贝到`/usr/local/bin`目录下

```
tar xvzf etcd-v3.4.3-linux-amd64.tar.gz
sudo mv etcd-v3.4.3-linux-amd64/etcd* /usr/local/bin/
```

## 3.创建etcd.service

分别在`k8s-master-03, k8s-master-02, k8s-master-01`三台机器上生成systemd配置文件

```
INTERNAL_IP=10.158.1.9
ETCD_NAME=k8s-master-03

INTERNAL_IP=10.158.1.8
ETCD_NAME=k8s-master-02

INTERNAL_IP=10.158.1.7
ETCD_NAME=k8s-master-01

cat <<EOF  sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster k8s-master-03=https://10.158.1.9:2380,k8s-master-02=https://10.158.1.8:2380,k8s-master-01=https://10.158.1.7:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

不需要认证的`etcd`

```
INTERNAL_IP=10.158.1.7
ETCD_NAME=k8s-master-01

INTERNAL_IP=10.158.1.8
ETCD_NAME=k8s-master-02

INTERNAL_IP=10.158.1.9
ETCD_NAME=k8s-master-03

cat <<EOF  sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd key-value store
Documentation=https://github.com/coreos/etcd

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \\
 --name ${ETCD_NAME} \\
 --data-dir /var/lib/etcd \\
 --initial-advertise-peer-urls http://${INTERNAL_IP}:2380 \\
 --listen-peer-urls http://${INTERNAL_IP}:2380 \\
 --listen-client-urls http://${INTERNAL_IP}:2379,http://127.0.0.1:2379,http://${INTERNAL_IP}:4001 \\
 --advertise-client-urls http://${INTERNAL_IP}:2379,http://${INTERNAL_IP}:4001 \\
 --initial-cluster-token etcd-cluster-0 \\
 --initial-cluster k8s-master-03=https://10.158.1.9:2380,k8s-master-02=https://10.158.1.8:2380,k8s-master-01=https://10.158.1.7:2380 \\
 --initial-cluster-state new \\
 --heartbeat-interval 1000 \\
 --election-timeout 5000
Restart=on-failure
RestartSec=5
LimitNOFILE=40000

[Install]
WantedBy=multi-user.target
EOF
```

## 4.启动和检测Etcd服务是否正常

启动服务

```
sudo systemctl daemon-reload
sudo systemctl enable etcd
sudo systemctl start etcd
sudo systemctl status etcd  
```

查看集群节点

```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```

插入一个值看看能否成功

```
sudo ETCDCTL_API=3 etcdctl put /dev/key v1 \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```

```
sudo ETCDCTL_API=3 etcdctl cluster-health \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```

## 常见安装问题