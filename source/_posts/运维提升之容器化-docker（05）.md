---
title: 运维提升之容器化——Docker（05）
tags:
  - Docker
  - 教程
id: '11966'
categories:
  - - skills
date: 2020-01-02 16:31:04
---

# 私有镜像仓库

## Dockerhub

目前 Docker 官方维护了一个公共仓库Docker Hub，大部分需求都可以通过在 Docker Hub 中直接下载镜像来实现。如果你觉得拉取 Docker Hub 的镜像比较慢的话，我们可以配置一个镜像加速器：[http://docker-cn.com/](http://docker-cn.com/ "http://docker-cn.com/")，当然国内大部分云厂商都提供了相应的加速器，简单配置即可。

> [阿里云镜像加速](https://help.aliyun.com/document_detail/60750.html " 阿里云镜像加速")

<!--more-->
## 私有仓库

有时候使用 Docker Hub 这样的公共仓库可能不方便，用户可以创建一个本地仓库供私人使用。 docker-registry是官方提供的工具，可以用于构建私有的镜像仓库。本文内容基于 docker-registry v2.x 版本。你可以通过获取官方 registry 镜像来运行。

```
$ docker run -d -p 5000:5000 --restart=always --name registry registry
```

这将使用官方的registry镜像来启动私有仓库。默认情况下，仓库会被创建在容器的/var/lib/registry目录下。你可以通过 -v 参数来将镜像文件存放在本地的指定路径。例如下面的例子将上传的镜像放到本地的 /opt/data/registry 目录。

```
$ docker run -d \
    -p 5000:5000 \
    -v /opt/data/registry:/var/lib/registry \
    registry
```

## 在私有仓库上传、搜索、下载镜像

创建好私有仓库之后，就可以使用docker tag来标记一个镜像，然后推送它到仓库。例如私有仓库地址为 127.0.0.1:5000。先在本机查看已有的镜像。

```
$ docker image ls
REPOSITORY                        TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu                            latest              ba5877dc9bec        6 weeks ago  
```

使用docker tag将 ubuntu:latest 这个镜像标记为 127.0.0.1:5000/ubuntu:latest。 格式为 docker tag IMAGE\[:TAG\] \[REGISTRY\_HOST\[:REGISTRY\_PORT\]/\]REPOSITORY\[:TAG\]

```
$ docker tag ubuntu:latest 127.0.0.1:5000/ubuntu:latest
$ docker image ls
REPOSITORY                        TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu                            latest              ba5877dc9bec        6 weeks ago         192.7 MB
127.0.0.1:5000/ubuntu:latest      latest              ba5877dc9bec        6 weeks ago         192.7 MB
```

使用docker push上传标记的镜像。

```
$ docker push 127.0.0.1:5000/ubuntu:latest
The push refers to repository [127.0.0.1:5000/ubuntu]
373a30c24545: Pushed
a9148f5200b0: Pushed
cdd3de0940ab: Pushedfc56279bbb33: Pushed
b38367233d37: Pushed
2aebd096e0e2: Pushed
latest: digest: sha256:fe4277621f10b5026266932ddf760f5a756d2facd505a94d2da12f4f52f71f5a size: 1568
```

用curl查看仓库中的镜像。

```
$ curl 127.0.0.1:5000/v2/_catalog
{"repositories":["ubuntu"]}
```

这里可以看到 {"repositories":\["ubuntu"\]}，表明镜像已经被成功上传了。 先删除已有镜像，再尝试从私有仓库中下载这个镜像。

```
$ docker image rm 127.0.0.1:5000/ubuntu:latest

$ docker pull 127.0.0.1:5000/ubuntu:latest
Pulling repository 127.0.0.1:5000/ubuntu:latest
ba5877dc9bec: Download complete
511136ea3c5a: Download complete
9bad880da3d2: Download complete
25f11f5fb0cb: Download complete
ebc34468f71d: Download complete
2318d26665ef: Download complete

$ docker image ls
REPOSITORY                         TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
127.0.0.1:5000/ubuntu:latest       latest              ba5877dc9bec        6 weeks ago         192.7 MB
```

## harbor

官方提供的 registry 比较简单，且没有 ui 界面，可以使用 harbor 搭建私有仓库 [harbor 搭建教程](https://www.52ynn.top/index.php/2020/02/19/harbor%e7%a7%81%e6%9c%89%e9%95%9c%e5%83%8f%e4%bb%93%e5%ba%93%e6%90%ad%e5%bb%ba/ "harbor 搭建教程")