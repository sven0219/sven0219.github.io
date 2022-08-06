---
title: Kubernetes 监控及日志采集方案
tags:
  - k8s
  - 运维
id: '12484'
categories:
  - - skills
date: 2021-08-04 11:04:50
---

# 1\. 监控方案

## 1.1 介绍

kubernetes 的监控分为三个维度： 主机、容器、应用 我这里采用的是 Promethues 和 Grafana 方案，通过各种 exporter 采集不同维度的监控指标，并通过 promethues 支持的数据格式暴露出来，最后使用 Grafana 展示，异常情况使用 AlertManager 告警。 [![](https://i.loli.net/2021/08/04/MkxNCQco3XwDAfn.jpg)](https://i.loli.net/2021/08/04/MkxNCQco3XwDAfn.jpg)

监控指标

实现方式

举例<!--more-->

Pod 性能

cAdvisor

容器 CPU、内存利用率

Node 性能

node-exporter

节点 CPU，内存利用率

K8S 资源对象

kube-state-metrics

Pod/Deployment/Service

# 2\. 部署

## 2.1 k8s 中部署 promethues

### 1\. 部署守护进程

文件：node-exporter.yaml

```yaml
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: kube-system
  labels:
    k8s-app: node-exporter
spec:
  selector:
    matchLabels:
        k8s-app: node-exporter
  template:
    metadata:
      labels:
        k8s-app: node-exporter
    spec:
      containers:
      - image: prom/node-exporter
        name: node-exporter
        ports:
        - containerPort: 9100
          protocol: TCP
          name: http
---
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: node-exporter
  name: node-exporter
  namespace: kube-system
spec:
  ports:
  - name: http
    port: 9100
    nodePort: 31672
    protocol: TCP
  type: NodePort
  selector:
    k8s-app: node-exporter
```

执行

```bash
kubectl create -f node-exporter.yaml
```

### 2\. 部署 promethues 相关

rbac-setup.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups: [""]
  resources:
  - nodes
  - nodes/proxy
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups:
  - extensions
  resources:
  - ingresses
  verbs: ["get", "list", "watch"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: kube-system
```

configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: kube-system
data:
  prometheus.yml: 
    global:
      scrape_interval:     15s
      evaluation_interval: 15s
    scrape_configs:

    - job_name: 'kubernetes-apiservers'
      kubernetes_sd_configs:
      - role: endpoints
      scheme: https
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      relabel_configs:
      - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
        action: keep
        regex: default;kubernetes;https

    - job_name: 'kubernetes-nodes'
      kubernetes_sd_configs:
      - role: node
      scheme: https
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - target_label: __address__
        replacement: kubernetes.default.svc:443
      - source_labels: [__meta_kubernetes_node_name]
        regex: (.+)
        target_label: __metrics_path__
        replacement: /api/v1/nodes/${1}/proxy/metrics

    - job_name: 'kubernetes-cadvisor'
      kubernetes_sd_configs:
      - role: node
      scheme: https
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - target_label: __address__
        replacement: kubernetes.default.svc:443
      - source_labels: [__meta_kubernetes_node_name]
        regex: (.+)
        target_label: __metrics_path__
        replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor

    - job_name: 'kubernetes-service-endpoints'
      kubernetes_sd_configs:
      - role: endpoints
      relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scheme]
        action: replace
        target_label: __scheme__
        regex: (https?)
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
        action: replace
        target_label: __address__
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_service_name]
        action: replace
        target_label: kubernetes_name

    - job_name: 'kubernetes-services'
      kubernetes_sd_configs:
      - role: service
      metrics_path: /probe
      params:
        module: [http_2xx]
      relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_probe]
        action: keep
        regex: true
      - source_labels: [__address__]
        target_label: __param_target
      - target_label: __address__
        replacement: blackbox-exporter.example.com:9115
      - source_labels: [__param_target]
        target_label: instance
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_service_name]
        target_label: kubernetes_name

    - job_name: 'kubernetes-ingresses'
      kubernetes_sd_configs:
      - role: ingress
      relabel_configs:
      - source_labels: [__meta_kubernetes_ingress_annotation_prometheus_io_probe]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_ingress_scheme,__address__,__meta_kubernetes_ingress_path]
        regex: (.+);(.+);(.+)
        replacement: ${1}://${2}${3}
        target_label: __param_target
      - target_label: __address__
        replacement: blackbox-exporter.example.com:9115
      - source_labels: [__param_target]
        target_label: instance
      - action: labelmap
        regex: __meta_kubernetes_ingress_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_ingress_name]
        target_label: kubernetes_name

    - job_name: 'kubernetes-pods'
      kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      - action: labelmap
        regex: __meta_kubernetes_pod_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: kubernetes_pod_name
```

prometheus.deploy.yml

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    name: prometheus-deployment
  name: prometheus
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - image: prom/prometheus:v2.0.0
        name: prometheus
        command:
        - "/bin/prometheus"
        args:
        - "--config.file=/etc/prometheus/prometheus.yml"
        - "--storage.tsdb.path=/prometheus"
        - "--storage.tsdb.retention=24h"
        ports:
        - containerPort: 9090
          protocol: TCP
        volumeMounts:
        - mountPath: "/prometheus"
          name: data
        - mountPath: "/etc/prometheus"
          name: config-volume
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
          limits:
            cpu: 500m
            memory: 2500Mi
      serviceAccountName: prometheus
      volumes:
      - name: data
        emptyDir: {}
      - name: config-volume
        configMap:
          name: prometheus-config
```

prometheus.svc.yml

```yaml
---
kind: Service
apiVersion: v1
metadata:
  labels:
    app: prometheus
  name: prometheus
  namespace: kube-system
spec:
  type: NodePort
  ports:
  - port: 9090
    targetPort: 9090
    nodePort: 30003
  selector:
    app: prometheus
```

依次执行

```bash
kubectl create -f rbac-setup.yaml
kubectl create -f configmap.yaml
kubectl create -f prometheus.deploy.yml
kubectl create -f prometheus.svc.yml

```

## 2.2 部署 grafana

grafana-deploy.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana-core
  namespace: kube-system
  labels:
    app: grafana
    component: core
spec:
  selector:
    matchLabels:
        app: grafana
        component: core
  replicas: 1
  template:
    metadata:
      labels:
        app: grafana
        component: core
    spec:
      containers:
      - image: grafana/grafana:4.2.0
        name: grafana-core
        imagePullPolicy: IfNotPresent
        # env:
        resources:
          # keep request = limit to keep this container in guaranteed class
          limits:
            cpu: 100m
            memory: 100Mi
          requests:
            cpu: 100m
            memory: 100Mi
        env:
          # The following env variables set up basic auth twith the default admin user and admin password.
          - name: GF_AUTH_BASIC_ENABLED
            value: "true"
          - name: GF_AUTH_ANONYMOUS_ENABLED
            value: "false"
          # - name: GF_AUTH_ANONYMOUS_ORG_ROLE
          #   value: Admin
          # does not really work, because of template variables in exported dashboards:
          # - name: GF_DASHBOARDS_JSON_ENABLED
          #   value: "true"
        readinessProbe:
          httpGet:
            path: /login
            port: 3000
          # initialDelaySeconds: 30
          # timeoutSeconds: 1
        volumeMounts:
        - name: grafana-persistent-storage
          mountPath: /var
      volumes:
      - name: grafana-persistent-storage
        emptyDir: {}
```

grafana-ing.yaml

```yam
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
   name: grafana
   namespace: kube-system
spec:
   rules:
   - host: k8s.grafana
     http:
       paths:
       - path: /
         backend:
          serviceName: grafana
          servicePort: 3000
```

grafana-svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: kube-system
  labels:
    app: grafana
    component: core
spec:
  type: NodePort
  ports:
    - port: 3000
  selector:
    app: grafana
    component: core
```

依次执行

```bash
kubectl create -f  grafana-deploy.yaml
kubectl create -f  grafana-svc.yaml
kubectl create -f  grafana-ing.yaml
```

## 2.3 配置

查看端口

```bash
 kubectl get svc -n kube-system -o wide
```

[![](https://i.loli.net/2021/08/31/i5VrehEnCOu6HfY.jpg)](https://i.loli.net/2021/08/31/i5VrehEnCOu6HfY.jpg) 访问 [![](https://i.loli.net/2021/08/31/MqXiPKL9RFxnaQD.jpg)](https://i.loli.net/2021/08/31/MqXiPKL9RFxnaQD.jpg) 默认账号密码都是 admin 登录之后 [![](https://i.loli.net/2021/08/31/lNcfRrEJFaIgzUp.jpg)](https://i.loli.net/2021/08/31/lNcfRrEJFaIgzUp.jpg) 配置数据源 [![](https://i.loli.net/2021/08/31/LsVISfJW8bAcXdu.jpg)](https://i.loli.net/2021/08/31/LsVISfJW8bAcXdu.jpg) [![](https://i.loli.net/2021/08/31/ELT9xsYCfvSIVGw.jpg)](https://i.loli.net/2021/08/31/ELT9xsYCfvSIVGw.jpg) 配置dashboard [![](https://i.loli.net/2021/08/31/MDIYCZbmcvsKqf1.jpg)](https://i.loli.net/2021/08/31/MDIYCZbmcvsKqf1.jpg) [![](https://i.loli.net/2021/08/31/xASjMuET9aiyNvs.jpg)](https://i.loli.net/2021/08/31/xASjMuET9aiyNvs.jpg) [![](https://i.loli.net/2021/08/31/8cTaqIfWN5SnH3A.jpg)](https://i.loli.net/2021/08/31/8cTaqIfWN5SnH3A.jpg) 结果 [![](https://i.loli.net/2021/08/31/PpnEYCa7UB1otls.jpg)](https://i.loli.net/2021/08/31/PpnEYCa7UB1otls.jpg)