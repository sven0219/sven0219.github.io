---
title: k8s 教程学习记录
tags:
  - k8s
  - 记录
id: '12392'
categories:
  - - skills
date: 2021-06-01 11:07:42
---

# 1\. 前面

教程地址： https://www.bilibili.com/video/BV1GT4y1A756

## 1.1 前置知识

1.  Linux 操作系统
2.  Docker

# 2\. K8S 概述和特性

## 2.1 K8S 概述
<!--more-->
```
K8S 是谷歌在 2014 年开源的容器化集群管理系统
使用 K8S 进行容器化应用部署
使用 K8S 利于应用扩展
K8S 目标实施让部署容器化应用更加简洁和高效
```

## 2.2 K8S 优势

```
 自动装箱
 自我修复
 水平扩展
 服务发现
 滚动更新
 版本回退
 密码和配置管理
 存储编排
 批处理
```

## 2.3 K8S 集群架构组件

Master（主控节点）和 node（工作节点） Master 组件：

```
 apiserver 集群统一入口，以 restful 方式，交给 etcd 存储
 scheduler 节点调度，选择 node 节点应用部署
 controller-manager 处理集群中常规的后台任务，一个资源对应一个控制器
 etcd 存储系统，用于保存集群相关的数据
```

node 节点：

```
kubelet master 在 node 节点中的代理，管理本机容器
kube-proxy 提供网络代理，实现负载均衡等操作
```

## 2.4 K8S 核心概念（概述）

1.  pod

*   最小部署单元
*   一组容器的集合
*   共享网络
*   生命周期是短暂的

2.  controller

*   确保预期的 pod 副本数量
*   无状态应用部署
*   有状态应用部署
*   确保所有的 node 运行同一个 pod
*   一次性任务和定时任务

3.  Service

*   定义一组pod 的访问规则

# 3\. 从零搭建 k8s 集群

## 3.1 搭建步骤

[集群搭建（kubeadm）](https://www.52ynn.top/index.php/2021/06/02/k8s-集群搭建/ "集群搭建") [集群搭建（二进制）](https://www.52ynn.top/index.php/2021/06/07/k8s-集群搭建二进制/ "二进制")

# 4\. k8s 基础

## 4.1 命令行工具 kubectl

`kubectl` 是`Kubernetes`集群的命令行工具，通过`kubectl`能够对集群本身进行管理，并能够在集群上进行容器化应用的安装

### 4.1.1 kubectl 语法格式

```bash
kubectl [command] [TYPE] [NAME] [flages]
```

基本用法 [![](https://i.loli.net/2021/06/15/L4Ix19MiRGlD8ju.jpg)](https://i.loli.net/2021/06/15/L4Ix19MiRGlD8ju.jpg)

## 帮助命令

```bash
kubectl --help
```

[![](https://i.loli.net/2021/06/15/GJusFYaWLUDKRkm.jpg)](https://i.loli.net/2021/06/15/GJusFYaWLUDKRkm.jpg)

### 4.1.2 子命令分类

基础命令 [![](https://i.loli.net/2021/06/15/wOqWUHhvgQc9YX5.jpg)](https://i.loli.net/2021/06/15/wOqWUHhvgQc9YX5.jpg) 部署和集群管理命令 [![](https://i.loli.net/2021/06/15/KSZTkwM3dhNER28.jpg)](https://i.loli.net/2021/06/15/KSZTkwM3dhNER28.jpg) 故障和调试命令 [![](https://i.loli.net/2021/06/15/vak9GKRb5gpId4f.jpg)](https://i.loli.net/2021/06/15/vak9GKRb5gpId4f.jpg) 其他命令 [![](https://i.loli.net/2021/06/15/sg4nKhaWI2S5lVB.jpg)](https://i.loli.net/2021/06/15/sg4nKhaWI2S5lVB.jpg)

## 4.2 YAML 文件详解

### 4.2.1 YAML 文件概述

k8s 集群中对资源管理和资源对象编排部署都可以通过声明样式（YAML）文件来解决，也就是可以把需要对资源对象操作编辑到 YAML 格式文件中，我们把这种文件叫做资源清单文件，通过 kubectl 命令直接使用资源清单文件就可以实现对大量的资源对象进行编排部署 了。

### 4.2.2 YAML 文件书写格式

*   仍是一种标记语言
*   通过缩进表示层级关系
*   不能使用 Tab 进行缩进，只能使用空格
*   一般开头缩进两个空格
*   字符后缩进一个空格，比如冒号，逗号后面
*   使用---表示新的 yaml 文件开始
*   使用 # 开头代表注释

### 4.2.3 yaml 文件组成部分

1.  控制器定义
2.  被控制对象

### 4.2.4 如何快速编写 yaml 文件

1.  使用 kubectl create 命令生成 yaml 文件 [![](https://i.loli.net/2021/06/15/diNYF7jfv4U1wBM.jpg)](https://i.loli.net/2021/06/15/diNYF7jfv4U1wBM.jpg)
2.  使用 kubectl get 命令导出 yaml 文件 [![](https://i.loli.net/2021/06/15/8Q6Kx1RDFCvY4yq.jpg)](https://i.loli.net/2021/06/15/8Q6Kx1RDFCvY4yq.jpg)

# 5\. Kubernetes 核心技术-Pod

## 5.1 Pod 概述

1.  kubernetes 中最小的部署单元
2.  kubernetes 不会直接处理容器，而是 Pod ，Pod 可以包含一个或者多个容器
3.  一个 pod 中的容器是共享同一个网络命令空间
4.  pod 是短暂的，生命周期短
5.  每一个 Pod 都有一个特殊的被称为”根容器“的 Pause 容器

## 5.2 Pod 存在意义

1.  创建容器时使用 docker，一个 docker 对应一个容器，一个容器有进程，一个容器运行一个应用程序
2.  Pod 是多进程设计，可以运行多个应用程序
3.  Pode 存在为了亲密性应用（两个应用之间要进行交互，网络之间调用）

## 5.3 Pod 实现机制

1.  共享网络 通过 Pause 容器，把其它业务容器加入到 Pause 容器里面，让所有业务容器在同一个名称空间中，实现网络共享
    
2.  共享存储 引入数据卷概念 Vloumn，使用数据卷进行持久化存储
    

## 5.4 Pod 镜像拉取策略

yaml 中 imagePullPolicy 属性 IfNotPresent：默认值，镜像在宿主机上不存在时才拉取 Always：每次创建 Pod 都会拉取一次镜像 Never：Pod 永远不会主动拉取镜像

## 5.5 Pod 资源限制

[![](https://i.loli.net/2021/06/16/4xUq2XZShwOsMBL.jpg)](https://i.loli.net/2021/06/16/4xUq2XZShwOsMBL.jpg)

## 5.6 Pod 重启机制

[![](https://i.loli.net/2021/06/16/y8dcBeCrGsbHSLh.jpg)](https://i.loli.net/2021/06/16/y8dcBeCrGsbHSLh.jpg)

## 5.7 Pod 健康检查

**livenessProbe(存活检查)**：如果检查失败，将杀死容器，根据 pod 的 restartPolicy 来操作 **readinessProbe（就绪检查）**：如果检查失败，kubernetes 会把 Pod 从 service endpoints 中剔除 Probe 支持以下三种检查方法： **httpGet**：发送 HTTP 请求，返回 200-400 范围的状态码为成功 **exec**：执行 shell 命令返回状态码为 0 为成功 **tcpsocket**：发起 TCP Socket 建立成功

## 5.8 Pod 调度策略

1.  Pod 资源限制对 Pod 调用产生影响
2.  节点选择器标签影响 Pod 调用
3.  节点亲和性对 pod 调用的影响
4.  污点与污点容忍

# 6\. Kubernetes 核心技术-Controller

# 6\. Kubernetes 核心技术-Controller

## 6.1 Controller 概述

在集群上管理和运行管理的对象

## 6.2 Pod 和 Controller 的关系

Pod 是通过 Controller 实现应用的运维，比如伸缩、滚动升级等等 Pod 和 Controller 之间通过 label 建立关系 [![](https://i.loli.net/2021/06/17/kImqYaBJXCWOpVR.jpg)](https://i.loli.net/2021/06/17/kImqYaBJXCWOpVR.jpg)

## 6.3 Deployment 应用场景

*   部署无状态应用
*   管理 Pod 和 ReplicaSet
*   部署，滚动升级等功能

应用场景：web 服务，微服务

## 6.4 使用 Deployment 部署应用（yaml）

### 6.4.1 部署 nginx

```bash
# 导出 yaml 文件
kubectl create deployment nginx --image=nginx --dry-run="client" -o yaml > nginx.yaml
```

文件内容

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

使用文件部署

```bash
kubectl apply -f nginx.yaml
```

部署完成后检查

```bash
kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
nginx-f89759699-rbqtq   1/1     Running   0          116s
```

#### 6.4.2 对外发布（暴露端口号）

```bash
# 导出 yaml 文件
kubectl expose deployment nginx --port=80 --type=NodePort --target-port=80 --name=web-expose -o yaml >web-expose.yaml
```

web-expose.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2021-06-22T07:37:26Z"
  labels:
    app: nginx
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:labels:
          .: {}
          f:app: {}
      f:spec:
        f:externalTrafficPolicy: {}
        f:ports:
          .: {}
          k:{"port":80,"protocol":"TCP"}:
            .: {}
            f:port: {}
            f:protocol: {}
            f:targetPort: {}
        f:selector:
          .: {}
          f:app: {}
        f:sessionAffinity: {}
        f:type: {}
    manager: kubectl
    operation: Update
    time: "2021-06-22T07:37:26Z"
  name: web-expose
  namespace: default
  resourceVersion: "1409104"
  selfLink: /api/v1/namespaces/default/services/web-expose
  uid: 0a065654-df89-4e36-babb-166b27429ebc
spec:
  clusterIP: 10.105.48.202
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 32386
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
```

执行

```bash
kubectl apply -f web-expose.yaml
```

## 6.5 应用升级回滚和弹性伸缩

### 6.5.1 应用升级

删除之前的 Deployment

```bash
kubectl delete deployment nginx 
```

先部署一个低版本的 nginx,修改 nginx.yaml 文件（或者另外创建一个）

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.14
        name: nginx
        resources: {}
status: {}
```

```bash
kubectl apply -f nginx.yaml
```

部署完成后

```bash
[root@k8s-master ~]# kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
nginx-6b48675d6f-wpjnc   1/1     Running   0          5m12s
nginx-6b48675d6f-zs9gv   1/1     Running   0          5m12s
```

升级操作

```bash
kubectl set image deployment nginx nginx=nginx:1.16
```

升级过程

```bash
[root@k8s-master ~]# kubectl get pods
NAME                         READY   STATUS              RESTARTS   AGE
nginx-6b48675d6f-wpjnc   1/1     Running             0          6m51s
nginx-6b48675d6f-zs9gv   1/1     Running             0          6m51s
nginx-6d84fbc5f9-l8qhp   0/1     ContainerCreating   0          15s
```

升级完成

```bash
[root@k8s-master ~]# kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
nginx-6d84fbc5f9-l8qhp   1/1     Running   0          3m3s
nginx-6d84fbc5f9-rzs5c   1/1     Running   0          38s
```

查看升级结果

```bash
[root@k8s-master ~]# kubectl rollout status deployment nginx
deployment "nginx" successfully rolled out
```

### 6.5.2 回滚操作

查看历史版本

```bash
[root@k8s-master ~]# kubectl rollout history deployment nginx
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

回滚至上一个版本

```bash
[root@k8s-master ~]# kubectl rollout undo deployment nginx
deployment.apps/nginx rolled back
```

回滚至指定版本

```bash
[root@k8s-master ~]# kubectl rollout undo deployment nginx --to-revision=2
deployment.apps/nginx rolled back
```

### 6.5.3 弹性伸缩

```bash
# 伸缩至副本至 20 
kubectl scale deployment nginx 1.14 --replicas=20
```

## 6.6 部署有状态应用

### 6.6.1 无状态和有状态的区别

> 听说过一个通俗的说法可以辅助记忆下：无状态就是你养了一群鸭，死了一只，再买一只补充进去进行了，有状态就是你养了一只宠物鸭，死了的话你再买一只宠物的话那也不是和你之前有感情的宠物了

**无状态**：

1.  任务 pod 都是一样的
2.  没有顺序要求
3.  不用考虑在哪个 node 运行
4.  随意进行伸缩和扩展

**有状态**：

1.  每个 pod 都是独立的，保持 pod 启动顺序和唯一性
2.  唯一的网络标识符，持久存储
3.  有序，比如 mysql 主从

### 6.6.2 部署有状态应用

CLusterIP:none **SatefulSet 部署有状态应用** 创建 sts.yaml 文件：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx

---

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-statefulset
  namespace: default
spec:
  serviceName: nginx
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80

```

创建应用

```bash
[root@k8s-master ~]# kubectl apply -f sts.yaml
service/nginx created
statefulset.apps/nginx-statefulset created
```

创建完成后

```bash
[root@k8s-master ~]# kubectl get pods
NAME                         READY   STATUS              RESTARTS   AGE
nginx-statefulset-0          1/1     Running             0          3m14s
nginx-statefulset-1          1/1     Running             0          1m12s
nginx-statefulset-2          1/1     Running             0          12s
```

## 6.7 部署守护进程

所有 pod 运行在一个 node 中，类似于日志采集需要固定到每个节点上的场景需要使用 创建 ds.yaml

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-test 
  labels:
    app: filebeat
spec:
  selector:
    matchLabels:
      app: filebeat
  template:
    metadata:
      labels:
        app: filebeat
    spec:
      containers:
      - name: logs
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: varlog
          mountPath: /tmp/log
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

创建应用

```bash
kubectl apply -f ds.yaml
```

查看

```bash
[root@k8s-master ~]# kubectl get pods
NAME            READY   STATUS    RESTARTS   AGE
ds-test-gzjpl   1/1     Running   0          25s
ds-test-hhzxt   1/1     Running   0          25s
```

## 6.8 一次任务和定时任务

一次性 job.yaml

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

定时任务 cronjob.yaml

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure

```

# 7\. Kubernetes 核心技术-Service

## 7.1 用途

1.  防止 Pod 失联（服务发现）: Pod的生命周期短，Pod 在创建或者销毁的时候会注册到 service 中，使调用者能调用到 pod 的真实 ip [![](https://i.loli.net/2021/06/22/L6WJARXPSerQ7Db.jpg)](https://i.loli.net/2021/06/22/L6WJARXPSerQ7Db.jpg)
2.  定义一组 Pod 的访问策略（负载均衡） [![](https://i.loli.net/2021/06/22/XiJE8jDWend6fV5.jpg)](https://i.loli.net/2021/06/22/XiJE8jDWend6fV5.jpg)

## 7.2 Pod 和 Service 之间的关系

根据 label 和 selector 标签简历关联的 [![](https://i.loli.net/2021/06/22/dCFOhrwmTNxv3fb.jpg)](https://i.loli.net/2021/06/22/dCFOhrwmTNxv3fb.jpg)

## 7.3 常见的 Service 类型

**ClusterIP：**集群内部进行使用，例如服务间相互访问，不对外提供，类型为ClusterIP的service，这个service有一个Cluster-IP，其实就一个VIP。具体实现原理依靠kubeproxy组件，通过iptables或是ipvs实现。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-python
spec:
  ports:
  - port: 3000
    protocol: TCP
    targetPort: 443
  selector:
    run: pod-python
  type: ClusterIP
```

[![](https://i.loli.net/2021/06/22/CWMlRJnjPSY7OhL.jpg)](https://i.loli.net/2021/06/22/CWMlRJnjPSY7OhL.jpg) **NodePort：**我们的场景不全是集群内访问，也需要集群外业务访问。那么ClusterIP就满足不了了。NodePort当然是其中的一种实现方案。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-python
spec:
  ports:
  - port: 3000
    protocol: TCP
    targetPort: 443
    nodePort: 30080
  selector:
    run: pod-python
  type: NodePort
```

[![](https://i.loli.net/2021/06/22/AdcNXpLl3W48gKP.jpg)](https://i.loli.net/2021/06/22/AdcNXpLl3W48gKP.jpg) **LoadBalance：**LoadBalancer类型的service 是可以实现集群外部访问服务的另外一种解决方案。不过并不是所有的k8s集群都会支持，大多是在公有云托管集群中会支持该类型。负载均衡器是异步创建的，关于被提供的负载均衡器的信息将会通过Service的status.loadBalancer字段被发布出去。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-python
spec:
  ports:
  - port: 3000
    protocol: TCP
    targetPort: 443
    nodePort: 30080
  selector:
    run: pod-python
  type: LoadBalancer
```

# 8\. kubernetes 核心技术-配置管理

> 这一部分视频讲的并不好，所以没有记详细的笔记，贴了中文文档作参考

## 8.1 Secret

作用：`Secret` 对象类型用来保存敏感信息，例如密码、OAuth 令牌和 SSH 密钥。 将这些信息放在 `secret` 中比放在 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 的定义或者 [容器镜像](https://kubernetes.io/zh/docs/reference/glossary/?all=true#term-image) 中来说更加安全和灵活，加密数据存在 etcd 中，让 pod 容器以挂载 Volume 方式进行访问 场景：凭证 中文文档：https://kubernetes.io/zh/docs/concepts/configuration/secret/

## 8.2 ConfigMap

作用：ConfigMap 是一种 API 对象，用来将非机密性的数据保存到键值对中。使用时， [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 可以将其用作环境变量、命令行参数或者存储卷中的配置文件。 场景：ConfigMap 将您的环境配置信息和 [容器镜像](https://kubernetes.io/zh/docs/reference/glossary/?all=true#term-image) 解耦，便于应用配置的修改。 中文文档：https://kubernetes.io/zh/docs/concepts/configuration/configmap/

# 9\. Kubernetes 核心技术-集群安全机制

## 9.1 概述

1.  访问 k8s 集群时候，需要经过三个步骤完成具体操作

```text
第一步认证 传输安全
* 传输安全： 对外不暴露 8080 端口，只能内部访问，对外使用 6443
* 认证 ： https 证书认证，基于 ca 证书；https token 认证，通过 token 识别用户；http 基本认证，用户名+密码

第二步 鉴权
* 基于 RBAC 进行鉴权操作
* 基于角色访问控制

第三步 准入控制
* 准入控制器列表，如果列表有请求内容，则通过
```

2.  进行访问时候，过程中都需要经过 apiserver，apiserver 做统一协调，比如门卫

## 9.2 RBAC 介绍

> https://kubernetes.io/zh/docs/reference/access-authn-authz/rbac/

基于角色的访问控制 RBAC API 声明了四种 Kubernetes 对象：_Role_、_ClusterRole_、_RoleBinding_ 和 _ClusterRoleBinding_。你可以像使用其他 Kubernetes 对象一样， 通过类似 `kubectl` 这类工具 [描述对象](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/kubernetes-objects/#understanding-kubernetes-objects), 或修补对象。

### Role 和 ClusterRole

RBAC 的 _Role_ 或 _ClusterRole_ 中包含一组代表相关权限的规则。 这些权限是纯粹累加的（不存在拒绝某操作的规则）。 Role 总是用来在某个[名字空间](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/namespaces/) 内设置访问权限；在你创建 Role 时，你必须指定该 Role 所属的名字空间。 与之相对，ClusterRole 则是一个集群作用域的资源。这两种资源的名字不同（Role 和 ClusterRole）是因为 Kubernetes 对象要么是名字空间作用域的，要么是集群作用域的， 不可两者兼具。 ClusterRole 有若干用法。你可以用它来：

1.  定义对某名字空间域对象的访问权限，并将在各个名字空间内完成授权；
2.  为名字空间作用域的对象设置访问权限，并跨所有名字空间执行授权；
3.  为集群作用域的资源定义访问权限。

如果你希望在名字空间内定义角色，应该使用 Role； 如果你希望定义集群范围的角色，应该使用 ClusterRole。

### RoleBinding 和 ClusterRoleBinding

角色绑定（Role Binding）是将角色中定义的权限赋予一个或者一组用户。 它包含若干 **主体**（用户、组或服务账户）的列表和对这些主体所获得的角色的引用。 RoleBinding 在指定的名字空间中执行授权，而 ClusterRoleBinding 在集群范围执行授权。 一个 RoleBinding 可以引用同一的名字空间中的任何 Role。 或者，一个 RoleBinding 可以引用某 ClusterRole 并将该 ClusterRole 绑定到 RoleBinding 所在的名字空间。 如果你希望将某 ClusterRole 绑定到集群中所有名字空间，你要使用 ClusterRoleBinding。 RoleBinding 或 ClusterRoleBinding 对象的名称必须是合法的 [路径区段名称](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/names#path-segment-names)。

# 10\. Kubernetes 核心技术 - Ingress

Ingress 是对集群中服务的外部访问进行管理的 API 对象，典型的访问方式是 HTTP。 Ingress 可以提供负载均衡、SSL 终结和基于名称的虚拟托管。

## 10.1 Ingress 和 pod 关系

1.  pod 和 ingress 通过 service 关联
    
2.  ingress 作为统一入口，由 service 关联一组 pod [![](https://i.loli.net/2021/06/30/1pXqsIkrczWSayi.jpg)](https://i.loli.net/2021/06/30/1pXqsIkrczWSayi.jpg)
    

## 10.2 ingress 工作流程

[![](https://i.loli.net/2021/06/30/8eXd3GmfgPjlMzS.jpg)](https://i.loli.net/2021/06/30/8eXd3GmfgPjlMzS.jpg) [![](https://i.loli.net/2021/06/30/EMrtvoNmVsdPbGA.jpg)](https://i.loli.net/2021/06/30/EMrtvoNmVsdPbGA.jpg)

## 10.3 ingress 使用

### 创建需要暴露端口的应用

```bash
# 创建 Deployment
kubectl create deployment web --image=nginx
# 创建 service
kubectl expose deployment web --port=80 --target-port=80 --type=NodePort
# 此时可以通过 nodeIp:port 访问服务
```

### 部署 Ingress Controller

```yaml
# 这里部署官方的维护的 nginx ingress 控制器
# 创建 ingress-Controller.yaml 文件
apiVersion: v1
kind: Namespace
metadata:
  name: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---

kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
kind: ConfigMap
apiVersion: v1
metadata:
  name: tcp-services
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
kind: ConfigMap
apiVersion: v1
metadata:
  name: udp-services
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: nginx-ingress-clusterrole
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - endpoints
      - nodes
      - pods
      - secrets
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - services
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - "extensions"
      - "networking.k8s.io"
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - "extensions"
      - "networking.k8s.io"
    resources:
      - ingresses/status
    verbs:
      - update

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
  name: nginx-ingress-role
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - pods
      - secrets
      - namespaces
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - configmaps
    resourceNames:
      # Defaults to "<election-id>-<ingress-class>"
      # Here: "<ingress-controller-leader>-<nginx>"
      # This has to be adapted if you change either parameter
      # when launching the nginx-ingress-controller.
      - "ingress-controller-leader-nginx"
    verbs:
      - get
      - update
  - apiGroups:
      - ""
    resources:
      - configmaps
    verbs:
      - create
  - apiGroups:
      - ""
    resources:
      - endpoints
    verbs:
      - get

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: nginx-ingress-role-nisa-binding
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: nginx-ingress-role
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: ingress-nginx

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: nginx-ingress-clusterrole-nisa-binding
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: nginx-ingress-clusterrole
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: ingress-nginx

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
      annotations:
        prometheus.io/port: "10254"
        prometheus.io/scrape: "true"
    spec:
      hostNetwork: true
      # wait up to five minutes for the drain of connections
      terminationGracePeriodSeconds: 300
      serviceAccountName: nginx-ingress-serviceaccount
      nodeSelector:
        kubernetes.io/os: linux
      containers:
        - name: nginx-ingress-controller
          image: lizhenliang/nginx-ingress-controller:0.30.0
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
            - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
            - --publish-service=$(POD_NAMESPACE)/ingress-nginx
            - --annotations-prefix=nginx.ingress.kubernetes.io
          securityContext:
            allowPrivilegeEscalation: true
            capabilities:
              drop:
                - ALL
              add:
                - NET_BIND_SERVICE
            # www-data -> 101
            runAsUser: 101
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
            - name: https
              containerPort: 443
              protocol: TCP
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 10
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 10
          lifecycle:
            preStop:
              exec:
                command:
                  - /wait-shutdown

---

apiVersion: v1
kind: LimitRange
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  limits:
  - min:
      memory: 90Mi
      cpu: 100m
    type: Container

```

### 创建 ingress 规则

创建 ingress-path.yaml,其中 host 写自己的域名，也可以写测试域名本地加解析

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: ingress.52ynn.top
    http:
      paths:
      - path: /
        backend:
          serviceName: web
          servicePort: 80
```

生成规则

```bash
kubectl apply -f  ingress-path.yaml
```

### 访问验证

添加域名解析后访问域名，或者本地修改 hosts 文件将域名指向对应 ip [![](https://i.loli.net/2021/06/30/gGI2ojHlaNxP9sw.jpg)](https://i.loli.net/2021/06/30/gGI2ojHlaNxP9sw.jpg)

# 11\. kubernetes 核心技术 - helm

## 11.1 helm 介绍

**是什么？**

1.  [Helm](http://helm.sh/) 是一个 Kubernetes 应用的包管理工具，用来管理 [chart](https://github.com/helm/charts)——预先配置好的安装包资源，有点类似于 Ubuntu 的 APT 和 CentOS 中的 YUM。

**可以解决哪些问题？**

1.  使用 helm 可以将众多 yaml 作为一个整体管理
2.  实现 yaml 文件高效复用
3.  实现应用级别的版本管理

**主要概念哪些？**

1.  Helm： 命令行工具，主要用于 kubernetes 应用 chart 的创建、打包、发布和管理
2.  Chart： 把 yaml 打包，一系列用于描述 kubernetes 资源相关文件的集合
3.  Release： 基于 Chart 的部署实体，一个 chart 被 helm 运行后会生成对应的一个 release；将在 k8s 中创建出真实运行的资源对象

**V3 版本变化？**

1.  V3 版本中删除 Tiller
2.  Release 在 V3 版本中可以在不同的命名空间中重用
3.  支持将 chart 推送到 docker 镜像仓库

## 11.2 helm 的安装和配置仓库

### 安装

安装文档：https://helm.sh/zh/docs/intro/install/

```bash
# 这里使用的是安装脚本自动安装
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3  bash
```

### 配置 helm 仓库

添加仓库

```bash
# 添加
helm repo add ali-stable https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
# 查看
helm repo list
# 更新
helm repo update
```

### 11.3 使用 helm 快速部署应用

**使用命令搜索应用**

```bash
[root@k8s-master ~]# helm search repo weave
NAME                CHART VERSION   APP VERSION DESCRIPTION
stable/weave-cloud  0.3.8           1.4.0       Weave Cloud is a add-on to Kubernetes which pro...
stable/weave-scope  1.1.11          1.12.0      A Helm chart for the Weave Scope cluster visual...
```

**根据搜索内容选择安装**

```bash
[root@k8s-master ~]# helm install ui stable/weave-scope
NAME: ui
LAST DEPLOYED: Thu Jul  1 16:50:16 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
You should now be able to access the Scope frontend in your web browser, by
using kubectl port-forward:

kubectl -n default port-forward $(kubectl -n default get endpoints \
ui-weave-scope -o jsonpath='{.subsets[0].addresses[0].targetRef.name}') 8080:4040

then browsing to http://localhost:8080/.
For more details on using Weave Scope, see the Weave Scope documentation:

https://www.weave.works/docs/scope/latest/introducing/
```

此时 service 默认模式为 ClusterIP，可以修改为 NodePort 方便测试

```bash
kubectl edit svc ui-weave-scope
```

修改 spec.type 为 NodePort 后保存退出,查看

```bash
[root@k8s-master ~]# kubectl get svc
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP        28d
ui-weave-scope   NodePort    10.109.92.86     <none>        80:32207/TCP   54s
web              NodePort    10.107.210.102   <none>        80:30591/TCP   24h
```

**浏览器访问** [![](https://i.loli.net/2021/07/01/GU3tZ5xusDXrEI6.jpg)](https://i.loli.net/2021/07/01/GU3tZ5xusDXrEI6.jpg)

## 11.4 自定义 chart

### 使用命令创建 chart

```bash
helm create myChart # 会生成一个名称为 myChart 的目录
```

目录结构为

```yaml
myChart/
├── charts
├── Chart.yaml   # 当前 chart 属性配置信息
├── templates  # 存放自定义 yaml 文件
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml  # 定义 yaml 文件中可以使用的全局变量

```

### 在 templates 中创建 yaml 文件

```bash
# 先删除 templates 中的示例文件
rm -rf myChart/templates/*
# 进入 templates 
cd myChart/templates
# 生成 Deployment.yaml 文件
kubectl create deployment web1 --image=nginx --dry-run=client -o yaml > deployment.yaml
# 生成 service.yaml 文件
kubectl create service nodeport web1 --tcp=80:80 --dry-run=client -o yaml  > service.yaml

```

# 12 kubernetes 核心技术 持久化存储

## 12.1 NFS 网络存储

```bash
步骤略

```

## 12.2 PV 和 PVC

https://support.huaweicloud.com/basics-cce/kubernetes\_0030.html

# 13 kubernetes 集群资源监控

https://www.52ynn.top/index.php/2021/08/04/kubernetes-%e7%9b%91%e6%8e%a7%e5%8f%8a%e6%97%a5%e5%bf%97%e9%87%87%e9%9b%86%e6%96%b9%e6%a1%88/