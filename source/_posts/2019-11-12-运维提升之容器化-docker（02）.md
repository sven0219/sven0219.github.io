---
title: 运维提升之容器化——Docker（02）
tags:
  - Docker
  - 教程
  - 运维
id: '11875'
categories:
  - - skills
date: 2019-11-12 14:44:16
---

# Docker 安装

直接前往[官方文档](https://docs.docker.com/install/ "官方文档")选择合适的平台安装即可，比如我们这里想要在centos系统上安装 Docker，这前往[地址](https://docs.docker.com/install/linux/docker-ce/centos/ "地址")根据提示安装即可。 安装依赖包
<!--more-->
```bash
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

添加软件仓库，我们这里使用稳定版 Docker，执行下面命令添加 yum 仓库地址：

```bash
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

然后直接安装即可：

```bash
$ sudo yum install docker-ce
```

启动 Docker ：

```bash
#设置开机自启
$ sudo systemctl enable docker
#启动 Docker
$ sudo systemctl start docker
```

测试是否成功: 验证docker:

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