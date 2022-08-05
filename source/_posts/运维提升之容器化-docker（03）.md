---
title: 运维提升之容器化——Docker（03）
tags:
  - Docker
  - 教程
  - 运维
id: '11893'
categories:
  - - skills
date: 2019-12-05 09:55:00
---

# 镜像和容器的基本操作

## 获取镜像

Docker 官方提供了一个公共的镜像仓库：[Docker Hub](https://hub.docker.com/explore/ "Docker Hub")，我们就可以从这上面获取镜像，获取镜像的命令：docker pull，格式为：

```bash
$ docker pull [选项] [Docker Registry 地址[:端口]/]仓库名[:标签]
```

*   Docker 镜像仓库地址：地址的格式一般是 <域名/IP>\[:端口号\]，默认地址是 Docker Hub。
*   仓库名：这里的仓库名是两段式名称，即 <用户名>/<软件名>。对于 Docker Hub，如果不给出用户名，则默认为 library，也就是官方镜像。比如：

```
$ docker pull mysql:5.7  
5.7: Pulling from library/mysql
68ced04f60ab: Pull complete 
f9748e016a5c: Pull complete 
da54b038fed1: Pull complete 
6895ec5eb2c0: Pull complete 
111ba0647b87: Pull complete 
c1dce60f2f1a: Pull complete 
702ec598d0af: Pull complete 
63cca87a5d4d: Pull complete 
ec05b7b1c5c7: Pull complete 
834b1d9f49b0: Pull complete 
8ded6a30c87c: Pull complete 
Digest: sha256:f4a5f5be3d94b4f4d3aef00fbc276ce7c08e62f2e1f28867d930deb73a314c58
Status: Downloaded newer image for mysql:5.7

```

上面的命令中没有给出 Docker 镜像仓库地址，因此将会从 Docker Hub 获取镜像。而镜像名称是 mysql:5.7 ，因此将会获取官方镜像 library/mysql 仓库中标签为 5.7 的镜像。 从下载过程中可以看到镜像是由多层存储所构成。下载也是一层层的去下载，并非单一文件。下载过程中给出了每一层的 ID 的前 12 位。并且下载结束后，给出该镜像完整的sha256的摘要，以确保下载一致性。 在国内官方仓库可能速度没有这么快，可以使用[阿里云镜像加速](https://help.aliyun.com/document_detail/60750.html?spm=a2c4g.11186623.6.551.24ae42c7Wu47Yz "阿里云镜像加速")

## 运行

有了镜像后，我们就能够以这个镜像为基础启动并运行一个容器。以上面的 mysql:5.7 为例，如果我们打算启动里面的 bash 并且进行交互式操作的话，可以执行下面的命令。

```
 docker run -it --rm mysql:5.7 /bin/bash
 # 进入容器执行命令
 root@9075ee7db42e:/#  mysql -V
 mysql  Ver 14.14 Distrib 5.7.29, for Linux (x86_64) using  EditLine wrapper
```

*   `docker run`就是运行容器的命令
*   `-it`：这是两个参数，一个是 -i：交互式操作，一个是 -t 终端。我们这里打算进入 bash 执行一些命令并查看返回结果，因此我们需要交互式终端。
*   `--rm`：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 docker rm。
*   `mysql:5.7`：这是指用 `mysql:5.7` 镜像为基础来启动容器。
*   `bash`：放在镜像名后的是命令，这里我们希望有个交互式 `Shell`，因此用的是 `bash`。 进入容器后，我们可以在 `Shell` 下操作，执行任何所需的命令。这里，我们执行了`mysql -V`，这是查看`mysql` 版本的命令，从返回的结果可以看到容器内是 `mysql` 版本 `Ver 14.14 Distrib 5.7.29`。

## 其它命令

#### 列出镜像

```
docker images

REPOSITORY                                                    TAG                 IMAGE ID            CREATED             SIZE
mysql                                                         5.7                 84164b03fa2e        8 hours ago         456MB
```

#### 镜像大小

```
docker system df

TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              17                  7                   4.612GB             3.702GB (80%)
Containers          10                  0                   2.155GB             2.155GB (100%)
Local Volumes       1                   1                   206.9MB             0B (0%)
Build Cache         0                   0                   0B                  0B
```

#### 后台运行容器

```
docker run -d  --name mysql mysql:5.7 /bin/sh -c "while true; do echo hello world; sleep 1; done"

05801216d88b750051fcdff2733bf86e44e358d57454f06342a6009c29251b09

# 使用 docker ps 命令可以查看所有已运行的的容器
docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                 NAMES
05801216d88b        mysql:5.7           "docker-entrypoint.s…"   3 seconds ago       Up 2 seconds        3306/tcp, 33060/tcp   mysql

```

#### 终止容器

```
docker stop mysql
# 停止后会打印容器名
mysql

```

#### 启动已终止的容器

```
docker start mysql
# 启动会打印容器名
mysql
```

#### 进入已启动的容器

```
docker exec -it mysql /bin/bash
# 进入容器中的终端
root@05801216d88b:/# 

```

#### 删除容器

```
# 删除已停止的容器
docker rm mysql 
# 删除正在运行的容器
docker rm -f mysql
```

#### 删除本地镜像

```
# 删除镜像前必须先删除该镜像启动的容器
docker rm mysql 
# 删除镜像
docker rmi mysql:5.7
```