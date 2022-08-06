---
title: elk搭建实践
tags:
  - ELK
  - 教程
  - 软件安装
  - 运维
id: '12235'
categories:
  - - skills
date: 2020-11-05 11:51:09
---

# 背景

新公司生产环境 ELK 突然丢失了几天的日志，因为不是很了解 ELK 具体的架构所以花了很长时间才排查出问题，所以准备花点时间自己部署一套系统以便了解 elk 详细点。
<!--more-->
# 介绍

ELK是三个开源软件的缩写，分别表示：Elasticsearch , Logstash, Kibana , 它们都是开源软件。新增了一个FileBeat，它是一个轻量级的日志收集处理工具(Agent)，Filebeat占用资源少，适合于在各个服务器上搜集日志后传输给Logstash，官方也推荐此工具。 Elasticsearch是个开源分布式搜索引擎，提供搜集、分析、存储数据三大功能。它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。 Logstash 主要是用来日志的搜集、分析、过滤日志的工具，支持大量的数据获取方式。一般工作方式为c/s架构，client端安装在需要收集日志的主机上，server端负责将收到的各节点日志进行过滤、修改等操作在一并发往elasticsearch上去。 Kibana 也是一个开源和免费的工具，Kibana可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以帮助汇总、分析和搜索重要数据日志。 Filebeat隶属于Beats。目前Beats包含四种工具：

*   Packetbeat（搜集网络流量数据）
*   Topbeat（搜集系统、进程和文件系统级别的 CPU 和内存使用情况等数据）
*   Filebeat（搜集文件数据）
*   Winlogbeat（搜集 Windows 事件日志数据）

简易架构图： [![](https://i.loli.net/2020/11/05/oAkCymFtwscbg82.jpg)](https://i.loli.net/2020/11/05/oAkCymFtwscbg82.jpg) 这是最简单的一种ELK架构方式。优点是搭建简单，易于上手。缺点是Logstash耗资源较大，运行占用CPU和内存高。另外没有消息队列缓存，存在数据丢失隐患。 此架构由Logstash分布于各个节点上搜集相关日志、数据，并经过分析、过滤后发送给远端服务器上的Elasticsearch进行存储。Elasticsearch将数据以分片的形式压缩存储并提供多种API供用户查询，操作。用户亦可以更直观的通过配置Kibana Web方便的对日志查询，并根据数据生成报表。 下面我将搭建该架构进行测试。

# 准备

## 机器准备

在阿里云上临时采购了一台测试机器 [![](https://i.loli.net/2020/11/05/PnSicY5BrgtuZ9l.jpg)](https://i.loli.net/2020/11/05/PnSicY5BrgtuZ9l.jpg) 本地测试也可以使用本地虚拟机，我这边为了方便直接就使用了阿里云机器。

## 软件准备

下载地址：https://www.elastic.co/downloads 下载如图三个： [![](https://i.loli.net/2020/11/05/DeMOySr5G9mvAuC.jpg)](https://i.loli.net/2020/11/05/DeMOySr5G9mvAuC.jpg) 下载好如图： [![](https://i.loli.net/2020/11/05/fM3H6jBGDAXTwCs.jpg)](https://i.loli.net/2020/11/05/fM3H6jBGDAXTwCs.jpg)

## 安装目录准备

```bash
mkdir -p /data/soft
```

# 部署配置

## elasticsearch 部署配置

### 安装 JDK

```bash
yum install java
```

检验

```bash
java -version
```

出现如下就说明 jdk 安装成功 [![](https://i.loli.net/2020/11/05/2kymIHZocdYTDjO.jpg)](https://i.loli.net/2020/11/05/2kymIHZocdYTDjO.jpg)

### 4.1.2 部署elasticsearch

**将压缩包解压至安装目录**

```bash
# 解压
tar -zxvf /root/elasticsearch-7.9.3-linux-x86_64.tar.gz -C /data/soft/
cd /data/soft
# 重命名
mv elasticsearch-7.9.3/ elasticsearch
```

**修改配置文件**

```bash
# 进入文件夹
cd /data/soft/elasticsearch
# 修改配置文件
vim config/elasticsearch.yml

# 修改以下几项：
node.name: node-1 # 设置节点名
http.port: 9200
network.host: 0.0.0.0 # 允许外部 ip 访问
cluster.initial_master_nodes: ["node-1"] # 设置集群初始主节点

# 修改系统安全配置
vim /etc/security/limits.conf
# 添加以下四行
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4096

vim /etc/sysctl.conf
# 添加下面一行
vm.max_map_count=655360
# 执行命令
sysctl -p

```

**新建用户并赋权** ES 为了安全考虑不允许使用 root 用户启动，所以需要新建一个普通用户启动程序

```bash
# 添加用户 es
useradd es
# 设置用户 es 的密码（需要输入两遍密码）
# （如果设置密码过于简单可能会提示 BAD PASSWORD: XXX ，如果是用 root 用户操作可忽略提示继续输入第二遍密码强制设置密码）
passwd es
# 将对应文件夹的权限赋予给 es
chown -R es:es /data/soft/elasticsearch
```

赋权后显示如下则说明成功 [![](https://i.loli.net/2020/11/05/4TQHWzrk1nsoIRB.jpg)](https://i.loli.net/2020/11/05/4TQHWzrk1nsoIRB.jpg) **切换用户启动 Elasticsearch**

```bash
# 切换用户
su es
# 进入安装目录
cd /data/soft/elasticsearch
# 启动服务（-d 表示后台启动）
./bin/elasticsearch -d
```

**验证** 开放防火墙9200 端口，我这边使用的是阿里云机器测试，将安全组全部放开了 [![](https://i.loli.net/2020/11/05/Ll8Q97ayNXxHfMS.jpg)](https://i.loli.net/2020/11/05/Ll8Q97ayNXxHfMS.jpg) 访问页面如下就说明成功 [![](https://i.loli.net/2020/11/05/kqMcVCNTaFoEiB1.jpg)](https://i.loli.net/2020/11/05/kqMcVCNTaFoEiB1.jpg)

## Kibana 部署配置

### 解压并移动到安装目录

```bash
# 解压前先退出 es 用户,切换至 root 用户
exit
# 解压至安装目录
 tar -zxvf  /root/kibana-7.9.3-linux-x86_64.tar.gz  -C /data/soft/
# 重命名
cd /data/soft
mv kibana-7.9.3-linux-x86_64/ kibana
```

### 修改配置文件

```bash
cd /data/soft/kibana
vim ./config/kibana.yml

# 修改如下
# 服务端口
server.port: 5601
# 服务器ip  本机
server.host: "0.0.0.0"
# Elasticsearch 服务地址
elasticsearch.hosts: ["http://localhost:9200"]
# 设置语言为中文
i18n.locale: "zh-CN"
```

### 授权并切换用户

```bash
# 授权
chown -R es:es /data/soft/kibana
# 切换用户
su es
```

### 启动 Kibana

注意： 启动 Kibana 之前要启动 Elasticsearch

```bash
nohup ./bin/kibana &
```

### 访问验证

[![](https://i.loli.net/2020/11/05/rL75RYbmNtKDl9n.jpg)](https://i.loli.net/2020/11/05/rL75RYbmNtKDl9n.jpg)

## Logstash 部署配置

### 解压并移动到指定目录

```bash
# 退出 es 用户
exit
# 解压至安装目录
tar -zxvf /root/logstash-7.9.3.tar.gz -C /data/soft
# 重命名
cd /data/soft
mv logstash-7.9.3/ logstash
```

### 安装插件

由于国内无法访问默认的gem source，需要将gem source改为国内的源。

```bash
vim Gemfile
# 将source这一行改成如下所示：
source "https://gems.ruby-china.com/"
# 安装
./bin/logstash-plugin install logstash-codec-json_lines
```

安装成功如下： [![](https://i.loli.net/2020/11/05/ujD7yediPAcVYmL.jpg)](https://i.loli.net/2020/11/05/ujD7yediPAcVYmL.jpg)

# 配置 NGINX 日志

## 安装 NGINX

```bash
yum install -y nginx 
```

## 修改 NGINX 日志格式

```bash
vim /etc/nginx/nginx.conf
```

```bash
log_format main '{"@timestamp":"$time_iso8601",'
'"host":"$server_addr",'
'"clientip":"$remote_addr",'
'"request":"$request",'
'"size":$body_bytes_sent,'
'"responsetime":$request_time,'
'"upstreamtime":"$upstream_response_time",'
'"upstreamhost":"$upstream_addr",'
'"http_host":"$host",'
'"url":"$uri",'
'"referer":"$http_referer",'
'"agent":"$http_user_agent",'
'"status":"$status"}';

access_log /var/log/nginx/access_test.log  main;
```

## 配置 logstash

```bash
vim /data/soft/logstash/config/logstash-nginx.conf

#如下
input {
   file {
        type => "nginx-access-log"
        path => "/var/log/nginx/access_test.log"
        start_position => "beginning"
        stat_interval => "2"
        codec => json
   }

}
filter {}
output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    #index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
    index => "logstash-nginx-access-log-%{+YYYY.MM.dd}"
    #user => "elastic"
    #password => "changeme"
  }
  stdout {
        codec => json_lines
  }
}
```

```bash
 nohup ./bin/logstash -f ./config/logstash-nginx.conf
```

## 配置索引

访问 kibana [![](https://i.loli.net/2020/11/05/NzWQAhOTp8s2jyE.jpg)](https://i.loli.net/2020/11/05/NzWQAhOTp8s2jyE.jpg) 创建索引 [![](https://i.loli.net/2020/11/05/6xUSnPEdDmBj3HK.jpg)](https://i.loli.net/2020/11/05/6xUSnPEdDmBj3HK.jpg) [![](https://i.loli.net/2020/11/05/Viwj3PEpOIJzXfN.jpg)](https://i.loli.net/2020/11/05/Viwj3PEpOIJzXfN.jpg)

## 验证

访问 NGINX 页面 [![](https://i.loli.net/2020/11/05/QxAkX1EPnWKUtO9.jpg)](https://i.loli.net/2020/11/05/QxAkX1EPnWKUtO9.jpg) 查看日志文件 [![](https://i.loli.net/2020/11/05/2rtPleQNvIiH19T.jpg)](https://i.loli.net/2020/11/05/2rtPleQNvIiH19T.jpg) 查看 Kibana 页面 [![](https://i.loli.net/2020/11/05/d5SI31Z7gK6eEnY.jpg)](https://i.loli.net/2020/11/05/d5SI31Z7gK6eEnY.jpg) 出现日志即完成