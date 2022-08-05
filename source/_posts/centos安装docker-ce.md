---
title: Centos安装Docker CE
tags:
  - centos
  - 运维
id: '11800'
categories:
  - - skills
date: 2019-10-02 16:30:32
---

# Centos安装Docker CE

Docker分为CE和EE两大版本.CE免费,支持周期7个月,EE付费,支持周期24个月.

## 系统要求

```
Docker CE支持64位版本CentOS7,并且要求内核版本不低于3.10。
```
<!--more-->

根据官方文档：https://docs.docker.com/install/linux/docker-ce/centos/搭建docker

##### 1.卸载docker旧版本：

```bash
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

##### 2.安装相关工具类：

```bash
sudo yum install -y yum-utils  device-mapper-persistent-data  lvm2
```

##### 3.配置docker仓库：

```bash
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

出现以下内容则表示docker仓库配置成功：

```
Loaded plugins: fastestmirror
adding repo from: http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
grabbing file http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
```

##### 4.安装docker

```bash
sudo yum install docker-ce
```

```
Installed:
  docker-ce.x86_64 0:18.03.0.ce-1.el7.centos

Dependency Installed:
  audit-libs-python.x86_64 0:2.7.6-3.el7 checkpolicy.x86_64 0:2.5-4.el7   container-selinux.noarch 2:2.42-1.gitad8f0f7.el7 libcgroup.x86_64 0
  libtool-ltdl.x86_64 0:2.4.2-22.el7_3   pigz.x86_64 0:2.3.3-1.el7.centos policycoreutils-python.x86_64 0:2.5-17.1.el7     python-IPy.noarch

Complete!
```

##### 5.验证docker安装成功：

启动docker：

```bash
sudo systemctl enable docker
sudo systemctl start docker 
```

验证docker:

```bash
sudo docker run hello-world
```

出现以下内容则表示安装成功：

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
9bb5a5d4561a: Pull complete
Digest: sha256:f5233545e43561214ca4891fd1157e1c3c563316ed8e237750d59bde73361e77
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
```

##### 6\. 建立docker用户组

默认情况下, docker 命令会使用 Unix socket 与 Docker 引擎通讯。而只有 root 用户和 docker 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑,一般 Linux 系统 上不会直接使用 root 用户。因此,更好地做法是将需要使用 docker 的用户加入 docker  
用户组。 建立docker组:

```
sudo groupadd docker
```

将当前用户加入docker组:

```
sudo usermod -aG docker $USER
```