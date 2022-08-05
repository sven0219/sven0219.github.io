---
title: 运维提升——Kubernetes（02）
tags:
  - k8s
  - 教程
  - 运维
id: '12000'
categories:
  - - skills
date: 2020-03-23 21:50:58
---

# Kubernetes 基本概念和术语

> 晚上刚下班，学习下 Kubernetes 的基本概念和术语，先了解基本概念，后面不至于学的时候一脸懵逼

## Master

`Kubernetes`里面 Master指的是集群控制节点，每个 Kubernetes 集群里面都需要有一个Master节点负责整个集群的管理和控制，基本上 Kubernetes 的所有控制指令都发送给它，它负责具体的执行过程。Master节点一般占据一个独立的服务器（高可用部署建议使用 3 台服务器），如果 Master节点宕机或者不可以，那么集群内容器应用的管理都将失效（不像 zookeeper 一样可以推选出一个新的 master 节点） Master节点上运行着一组关键进程：
<!--more-->
*   Kubernetes API Servre (kube-apiserver): 提供了 HTTP Rest 接口的关键服务进程， Kubernetes 里所有资源的增删改查等操作的唯一入口，也是集群控制的入口进程。
*   Kubernetes Controller Manager（kube-controller-manager）: Kubernetes 里面所有资源对象的自动化控制中心，可以理解为资源对象的‘大总管’。
*   Kuberneter Scheduler (kube-scheduler): 负责资源调度（Pod 调度）的进程，相当于公司的‘调度室’。

另外，`Matser` 节点上还需要启动一个`etcd`服务，因为`Kubernetes`里面所有资源对象的数据都是保存在`etcd`中的。

## Node

和其它高可用集群类似， Kubernetes 集群中其他机器被称为 Node 节点。和 Master 一样，Node 节点可以是一台物理主机也可以是一台虚拟机。Node 节点是 Kubernetes 集群中的工作负载节点，每个 Node 都会被 Master 分配一些工作负载（Docker 容器），当某个 Node 宕机时，其上的工作负载就会被 Master 节点自动转移到其它节点上去。 每个 Node 节点上都运行着一组关键进程：

*   kubelet: 负载 Pod 对应容器的创建、启停等任务，同事与 Master 节点密切协作，实现集群管理的基本功能。
*   kube-proxy：实现 Kubernetes Serveice 的通信和负载均衡机制的重要组件
*   Docker Engine（Docker）：Docker 引擎，负责本机的容器创建和管理工作

在默认情况下，安装配置启动好上述关键进程的 Node 节点会自动想 Master 节点上注册自己，Node 节点可以在运行期间动态增加到 Kubernetes 集群中。kubelet 进程会定时想 Master 节点汇报自身的情报，如操作系统、Docker 版本、机器的 CPU 和内存情况，当某个 Node 节点超过指定时间不上报信息时，Master 会判定该节点‘失联’，Node 的状态会被标记为‘Not ready’，随后 Master 会触发‘工作负载大转移’的自动流程。

## Pod

Pod 是 Kubernetes 的最重要也是最基本的概念。是最小部署单元，不是一个程序/进程，而是一个环境(包括容器、存储、网络`ip:port`、容器配置)，其中可以运行1个或多个container（docker或其他容器），在一个pod内部的container共享所有资源，包括共享pod的ip:port和磁盘。 Kubernetes 为每个 Pod 分配了**唯一的** IP 地址，称之为 Pod IP，一个 Pod 容器中共享 Pod IP。 Kubernetes 集群中任意两个 Pod 之间支持 TCP/IP 直接通信。

## Label（标签）

Label 是 Kubernetes 系统中另一个核心概念。一个 Label 是一个 key = Value 的键值对，其中 key 和 value 由用户自己指定。Label 可以附加到各种资源对象上，例如 Node、Pod、Service、RC 等，一个资源可以定义任意数量的 Label，同一个 Label 也可以被添加到任意数量的资源对象上去。 Label 相当于我们熟悉的“标签”，给某个资源对象定义一个 Label，就相当于给它打了一个标签，随后可以通过 Label Selector（标签选择器）查询和筛选拥有某些 Label 的资源对象， Kubernetes 通过这种方式实现了类似 SQL 的简单又通用的对象查询机制。相当于本篇博文的标签为 [k8s](https://www.52ynn.top/index.php/tag/k8s/ "k8s") 一样，可以直接筛选同类的文章 ## Replication Controller RC 是 Kubernetes 系统中的核心概念之奕，简单的说，他其实是定义了一个期望的场景，即声明某种 Pod 的副本数量在任意时刻都符合某个预期值，所以 RC 的定义包括如下几个部分

*   Pod 期待的副本数（replicas）
*   用于筛选目标的 Pod 的 Label Selector
*   当 Pod 的数量小于预期数量时，用于创建新 Pod 的 Pod 模板（template）

这个就有点像阿里云的弹性伸缩

## Deployment

Deployment 是 Kubernetes v1.2 引入的新概念，引入的目的是为例更好的解决 Pod 的编排问题。 Deployment 相对于 RC 的一个最大的升级是我们可以随时知道当前 Pod 的‘部署’进度。

## Horizontal Pod Autoscaler

HPA 与 RC、Deployment 一样，也属于一种 Kubernetes 资源对象。通过追踪分析 RC 控制所有目标 Pod 的负载变化情况，来确定是否需要针对性的调整目标 Pod 的副本数，这是 HPA 的实现原理。

## StatefulSet

Kubernetes 中，Pod 的管理对象 RC、Deployment、DaemonSet 和 Job 都是面向无状态的服务。但显示中有很多服务是有状态的。为了能够在其它节点史昂恢复某个失败的节点，这种集群中的 Pod 需要挂接某种共享存储，为了解决这个问题，在 v1.4 版本中引入这个新的额资源对象。

## Service（服务）

Service 也是 Kubernetes 里面最核心的资源对象之奕， Kubernetes 里的每个 Service 其实就是我们经常提起的微服务结构中的一个‘微服务’。 这个概念比较重要，后续实践中慢慢学习。

## Volume（存储卷）

Volume 是 Pod 中能够被多个容器访问的共享目录。Kubernetes 中的 Volume 的概念、用途和目的与 Docker 中的 Volume 比较类似，但不相同。

## Persistent Volume

PV 可以理解成 Kubernetes 集群中的某个网络存储对应的一块存储，与 Volume 类似，但有一些区别：

*   PV 只能是网络存储，不属于任何 Node，但可以在每个 Node 上访问
*   PV 并不是定义在 Pod 上的，而是独立于 Pod 职位的定义。
*   PV 目前支持的类型包括： gcePersistentDisk、AWSElasticBlockStore、AzureFile、NFS、iSCSI、RBD（Rados BlockDevice）等

## Namespace（命名空间）

Namespace（命名空间）是 Kubernetes 中用于实现多租户的资源隔离。Namespace 通过将集群内部的资源对象‘分配’到不同的 Namespace 中，形成逻辑上分组的不同项目、小组或用户组，便于不同分组在共享使用整个集群的资源的同时还能被分别管理。 Kubernetes 集群在启动后，会创建一个名为‘default’的 Namespace。

## Annotation（注解）

Annotation 与 Label 类似，也使用 key/value 键值对的形式进行定义。不同的是 Label 具有严格的命名规则，它定义的是 Kubernetes 对象的元数据（Metadata），并且用户 Label Selector。而 Annotation 则是用户任意定义的“附加”信息，以便于外部工具进行查找，很多时候，Kubernetes 的模块自身会通过 Annotation 的方式标记资源对象的一些特殊信息

## 写在后面

Kubernetes 中的概念术语相对于 Docker 而言还是比较多的，而且也比较复杂，今天只是简单的过了一遍，让自己的脑子中有一个基本的印象，等后续学习中遇到相关的概念再进行一个加深的理解。