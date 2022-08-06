---
title: Kubernetes 源码：Kubernetes 目录结构
tags:
  - Kubernetes源码
id: '12529'
categories:
  - - skills
date: 2022-02-22 17:42:47
---

# Kubernetes 源码阅读

version:v1.24 目录结构： ![file](https://www.52ynn.top/wp-content/uploads/2022/02/6214aa9a32e73.png)

# 目录结构分析

根据功能主要分成以下几类：

1.  文档类：（`api 、docs 、logo`）
2.  工具类：（`build、cluster、Godeps、hack、staging、translations`）
3.  代码类:（`cmd、pkg、plugin、test、third_party`）

工具类主要用到的build目录下的文件，自己动手编译的时候会用到；核心代码集中在cmd和pkg中。 cmd内部包含各个组件的入口，具体核心的实现部分在pkg目录下，分别如图：
<!--more-->
plugin目录之前的版本包括scheduler部分的代码，当前版本（应该是在1.10之后）已经将scheduler部分代码移到和其他组件一致的pkg目录，所以目前plugin主要包含的是认证与鉴权部分的代码。