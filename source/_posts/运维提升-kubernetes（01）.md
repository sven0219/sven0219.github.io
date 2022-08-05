---
title: 运维提升——Kubernetes（01）
tags:
  - k8s
  - 教程
  - 运维
id: '11968'
categories:
  - - skills
date: 2020-03-20 16:49:57
---

# 前面说的话

工作中发现 k8s 在实际生产环境中使用越来越频繁了，所以准备开始学习 k8s，坚持一周写一篇博客 学习途径： 1. 书籍：《Kubernetes权威指南-从 Docker 到 Kubernetes 实践全接触》 2. [Kubernetes中文社区](https://www.kubernetes.org.cn/k8s "Kubernetes中文社区")
<!--more-->
# 背景

`Kubernetes`这个名字起源于古希腊，舵手的意思。谷歌采用这个名字一层深意就是：`Docker`自己定义为驮着集装箱在大海自在遨游的鲸鱼，那么谷歌就要以`Kubernetes`掌舵大航海时代的话语权。 `Kubernetes`诞生的时间并不早，第一个正式版本`Kubernetes 1.0`在 2015 年 7 月才发布，但是很快就吸引了IBM、微软、红帽、Vmware 等众多业界巨头纷纷加入。

# Kubernetes 是什么

`Kubernetes`是一个全新的基于容器技术的分布式架构领先方案。`Kubernetes`吸取了谷歌一个久负盛名的内部使用的大规模集群管理系统`Borg`过去十年的经验与教训，所以`Kubernetes`一经开源就一鸣惊人，并迅速称霸了容器技术领域。

# 为什么要用 Kubernetes

1.  减少开发周期，Kubernetes已经做了多多事情
2.  使用 Kubernetes 就是全面拥抱微服务架构
3.  系统可以随时随地整体‘搬迁’到公有云上
4.  Kubernetes 具有超强的横向扩容能力