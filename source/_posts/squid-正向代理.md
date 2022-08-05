---
title: squid 正向代理
tags:
  - 正向代理
  - 软件安装
  - 运维
id: '11835'
categories:
  - - skills
date: 2019-12-06 15:39:06
---

# 正向代理 squid 安装配置

> 之前一个香港客户连接我们公司的 sass 服务（部署在阿里云上海）一直网络延迟很大，于是就在阿里云香港购买了一个ECS 实例搭建了正向代理服务，由于在这之前从未接触过正向代理，所以当时随便在网上查了查，使用了 squid 进行搭建。 最初很简单的，直接使用 squid 默认配置，也没有限制 ip 网段和配置用户认证，导致某天看日志发现很多其它 ip 使用了这个代理。 今天工作不是很忙，正好整理下这个文档，以备后面还需要使用到

操作系统： Centos7.6 文章中所有命令都是 root 用户执行的，如使用管理员在命令前加上`sudo`

## 1\. 安装

Centos 7默认软件安装源是有 squid 的，所以安装很简单，直接使用命令即可

```bash
yum install aquid
```

## 2\. 配置

```
vim /etc/squid/squid.conf
```

修改配置文件如下

```properties
http_port 3712
cache_mem 64 MB
maximum_object_size 4 MB
cache_dir ufs /var/spool/squid 100 16 256
access_log /var/log/squid/access.log
http_access allow all
visible_hostname squid
```

## 3\. 初始化

在第一次启动之前或者修改了[cache](https://www.centos.bz/tag/cache/)路径之后，需要重新初始化cache目录。

```bash
squid -z
```

## 4\. 启动

```bash
systemctl start squid
# 设置开机自启
systemctl enable squid
```

## 5\. 使用

在浏览器中修改代理配置即可。 在[windows](https://www.centos.bz/category/other-system/windows/)中：

*   Internet选项 -> 连接 -> 局域网连接 -> 代理服务器

在macOSX中：

*   Safari -> 偏好设置 -> 代理 -> Web代理

在 linux 中：

```bash
# 1、临时生效
set"http_proxy=http://[user]:[pass]@host:port/"
# 或
export"http_proxy=http://[user]:[pass]@host:port/"
# 2. 永久生效
创建$HOME/.wgetrc文件，加入以下内容：
http_proxy=代理主机IP:端口
```

## 6\. 测试

首先打开百度，然后搜索ip。如果出来的是你代理的那台机器的ip，就是设置成功了

## 7.配置用户认证

Squid实现用户名密码，使用HTTPBasicAuth 的方式。 需要htpasswd工具来创建passwd文件 （安装Apache软件，此工具会附带安装， 或者使用 `yum install http-tools`的方式安装此工具）

1.  创建用户密码

```bash
htpasswd  -c /etc/squid/passwd proxy_username
```

输入相应的密码后，生成 文件 /etc/squid/passwd

2.  将下述代码添加到/etc/squid/squid.conf 中即配置实用验证的功能

```bash
auth_param basic program /usr/lib64/squid/basic_ncsa_auth /etc/squid/passwd
acl auth_user proxy_auth REQUIRED
http_access allow auth_user
```

3.  重启服务

```bash
   service squid restart
```