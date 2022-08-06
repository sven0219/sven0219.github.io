---
title: 基于 kubernetes 的分布式构建平台
tags:
  - k8s
  - 运维
id: '12316'
categories:
  - - skills
date: 2021-01-18 10:20:29
---

## 分布式构建系统架构

一个是`Jenkins`，一个流行的持续集成/发布的工具，另一个是`Kubernetes`，一个流行的容器编排引擎。持续构建与发布是我们日常工作中必不可少的一个步骤，目前大多公司都采用 Jenkins 集群来搭建符合需求的 CI/CD 流程，然而传统的 Jenkins Slave 一主多从方式会存在一些痛点: 1. 主 Master 发生单点故障时，整个流程都不可用了； 1. 每个 Slave 的配置环境不一样，来完成不同语言的编译打包等操作，但是这些差异化的配置导致管理起来非常不方便，维护起来也是比较费劲； 1. 资源分配不均衡，有的 Slave要运行的job出现排队等待，而有Slave处于空闲状态；最后资源有浪费，每台 Slave 可能是实体机或者VM，当Slave处于空闲状态时，也不会完全释放掉资源。 针对这些痛点，我们引入了`kubernetes-plugin插件`
<!--more-->
#### kubernetes-plugin插件使用

![image](https://www.kubernetes.org.cn/img/2017/10/20171025111340.jpg)

1.  **服务高可用**，当 Jenkins Master 出现故障时，Kubernetes 会自动创建一个新的 Jenkins Master 容器，并且将 Volume 分配给新创建的容器，保证数据不丢失，从而达到集群服务高可用。
2.  **动态伸缩，合理使用资源**，每次运行 Job 时，会自动创建一个 Jenkins Slave，Job 完成后，Slave 自动注销并删除容器，资源自动释放，而且 Kubernetes 会根据每个资源的使用情况，动态分配 Slave 到空闲的节点上创建，降低出现因某节点资源利用率高，还排队等待在该节点的情况。
3.  **扩展性好**，当 Kubernetes 集群的资源严重不足而导致 Job 排队等待时，可以很容易的添加一个 Kubernetes Node 到集群中，从而实现扩展。

## 搭建Kuberntes集群

kubeadm进行搭建

## 搭建Jenkins集群

kubernetes集群搭建好了以后，可以通过kubectl进行远程安装。具体的安装的`yaml`文件如下：

```
--- yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
 name: jenkins-dev
spec:
 replicas: 1
 template:
  metadata:
   labels:
    app: jenkins-dev-test
  spec:
   securityContext:
     fsGroup: 1000
     runAsUser: 0
   containers:
   - name: jenkins
     imagePullPolicy: IfNotPresent
     image: jenkins/jenkins:lts
     ports:
     - containerPort: 8080
     volumeMounts:  
       - mountPath: /var/jenkins_home
         name: jenkins-home 
   volumes: 
     - name: jenkins-home
       persistentVolumeClaim: 
         claimName: jenkins-claim

---
apiVersion: v1
kind: Service
metadata:
  name: jenkins-dev
spec:
  # type: LoadBalancer
  selector:
    app: jenkins-dev-test
  # ensure the client ip is propagated to avoid the invalid crumb issue when using LoadBalancer (k8s >=1.7)
  #externalTrafficPolicy: Local
  ports:
    - port: 8080
      name: web
      targetPort: 8080
    - port: 50000
      name: agent
      targetPort: 50000
```

把jenkins的工作目录`/var/jenkins_home`挂载到了openebs管理的硬盘上。

## 构建镜像环境准备

Maven的DockerInDocker镜像dockerfile

```
FROM cnych/jenkins:jnlp6
MAINTAINER Stanley Huang <xinyu.huang02@liulishuo.com>

USER root
RUN apt-get update && apt-get install -y libltdl7.*
RUN apt-get install maven -y
```

Gradle的DockerInDockerDockerfile

```
FROM cnych/jenkins:jnlp6
MAINTAINER Stanley Huang <xinyu.huang02@liulishuo.com>
ARG GRADLE_VERSION=5.6.1

USER root
RUN apt-get update && apt-get install -y libltdl7.*
RUN apt-get install maven -y

WORKDIR /usr/bin

RUN \
  curl -sLO https://services.gradle.org/distributions/gradle-${GRADLE_VERSION}-all.zip && \
  unzip gradle-${GRADLE_VERSION}-all.zip && \
  ln -s gradle-${GRADLE_VERSION} gradle && \
  rm gradle-${GRADLE_VERSION}-all.zip

ENV GRADLE_HOME /usr/bin/gradle
ENV PATH $PATH:$GRADLE_HOME/bin
```

## 配置Ingress和域名

Ingress是授权入站连接到达集群服务的规则集合。

1.  从外部流量调度到nodeprot上的service
2.  从service调度到ingress-controller
3.  ingress-controller根据ingress中的定义（虚拟主机或者后端的url）
4.  根据虚拟主机名调度到后端的一组pod中

安装`ingress-nginx`, 官方文档如下，可以直接在k8s集群安装：

```
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
      serviceAccountName: nginx-ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.25.0
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
            # www-data -> 33
            runAsUser: 33
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
            - name: https
              containerPort: 443
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

---

```

在安装一个service，通过nodeport方式对外暴露端口，之后都通过该端口和容器内的服务进行通讯：

```
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  type: NodePort
  ports:
    - name: http
      port: 80
      targetPort: 80
      nodePort: 80
      protocol: TCP
    - name: https
      port: 443
      targetPort: 443
      protocol: TCP
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---

```

> 这里暴露80端口

创建一个ingress, 绑定后端服务，并且设置域名。

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-myapp
  namespace: default
spec:
  rules:
  - host: dev-jenkins-k8s.liulishuo.work
    http:
      paths:
      - path:
        backend:
          serviceName: myapp
          servicePort: 80
```

如上所示，这里以`dev-jenkins-k8s.liulishuo.work`为例，在ingress成功创建以后，找到sa配置下域名即可完成。

## OpenEBS存储服务

通过如下指令进行安装

```
kubectl apply -f https://openebs.github.io/charts/openebs-operator-1.1.0.yaml
```

基于LocalPV的hostPath方式，管理员可以创建一个个性化的`StorageClass`，通过下面的相同配置

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: openebs-hostpath
  annotations:
    openebs.io/cas-type: local
    cas.openebs.io/config: 
      - name: BasePath
        value: "/var/openebs/local"
      - name: StorageType
        value: "hostpath"
provisioner: openebs.io/local
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

当需要使用存储的时候, 可以创建一个pvc进行请求存储操作

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: jenkins-dev-aws-claim
  namespace: jenkins-dev
  annotations:
    volume.beta.kubernetes.io/storage-class: openebs-hostpath
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 15G
```

上面通过`openebs-hostpath`方式，申请了15G的内存进行使用, 并且采用`ReadWriteOnce`方式进行。pvc使用以后，通过`get pvc`指令可以查看到以及bound。 并且告诉你一个VOLUME目录，例如下面的`pvc-36743f52-9f11-47d9-97f8-dedc22046a9e`。

```bash
xinyu:~ xinyuhuang$  kubectl --kubeconfig ~/.kube/kubeconfig get pvc -n jenkins-dev
NAME                     STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       AGE
jenkins-hostpath-claim   Bound    pvc-36743f52-9f11-47d9-97f8-dedc22046a9e   15G        RWO            openebs-hostpath   16h
jenkins-pv-claim         Bound    jenkins-pv-volume                          10Gi       RWO            manual             15d
```

openebs的hostPath方式默认将volume数据存储在`/var/openebs/local`下，可以对其进行备份。

## 数据备份

## 搭建过程中遇到的问题

*   **jnlp agent 连接不上master**

遇到这个问题，需要注意下面几点，看看是不是设置错了 1. `Command to run` 不填内容 1. `Arguments to pass to the command`不填内容 1. `Container Template`必须设置为`jnlp` 1. `Jenkins tunnel`设置为`服务名：50000（端口号）`

*   **如何让ingress controller对外暴露80端口**

默认情况下，强行开放80端口，会爆出下面的错误

```
The Service "ingress-nginx" is invalid: spec.ports[0].nodePort: Invalid value: 80: provided port is not in the valid range. The range of valid ports is 30000-32767
```

我们需要修改下`kube-apiserver`的yaml配置文件。修改`/etc/kubernetes/manifests/kube-apiserver.yaml`(有些版本也可能是json)文件，修改其中的`- --service-node-port-range=80-32767` 将range从30000-32767修改为80-32767。如果没有这句话，则按照格式添加一句。

*   **Gradle Build速度缓慢**

对`~/.gradle`和`/home/jenkins/workspace`进行hostpath挂载，相当于添加了缓存。

*   **Kubernetes-plugin连接测试Jenkins失败**

会爆出下面的错误

```
deployments.apps is forbidden: User “system:serviceaccount:default:default” cannot create deployments.apps in the namespace
```

可以通过使得有权限访问pod api

```
kubectl create clusterrolebinding serviceaccounts-cluster-admin \
  --clusterrole=cluster-admin \
  --group=system:serviceaccounts
```

*   **jenkins异常重启导致用户名和密码改变**

[配置文件修改密码](https://www.jianshu.com/p/5e8b53baeff5)

## 快速恢复和迁移

jenkins可能经常会有环境的迁移，由于Jenkins的插件下载由于各种原因下载很缓慢，因此能过快速copy配置环境很重要。我们在Master节点都会通过openebs对`/var/jenkins_home`配置文件进行挂载。openebs的路径为`/var/openebs/local`。可以对下面的volume进行拷贝备份，迁移和快速恢复时，再通过将备份的配置文件复制到新的jenkins挂载的volume中，即可快速迁移和恢复。具体步骤如下：

*   将备份的jenkins配置文件拷贝到新创建的jenkins挂载目录下。所有文件拷贝过去即可。
*   重启Jenkins，重启只要删除相应jenkins的pod即可，deployment会自动创建新的pod。