---
title: promethues 的联邦和 K8S 集群的监控方案
tags:
  - k8s
  - Promethues
  - 监控
id: '12493'
categories:
  - - skills
date: 2021-09-01 15:45:33
---

# 联邦集群介绍

对于大部分监控规模来说，只需要在一个数据中心（云服务可用区，k8s 集群）部署一个Promethues Server 实例就可以在各个数据中心处理上千规模的集群。同时将 Promethue Server 部署到不同的数据中心可以避免网络配置的复杂性。 [![](https://i.loli.net/2021/09/01/K8YTrJ5h2ae93ZL.jpg)](https://i.loli.net/2021/09/01/K8YTrJ5h2ae93ZL.jpg) 如上图所示，在每个数据中心部署单独的Prometheus Server，用于采集当前数据中心监控数据。并由一个中心的Prometheus Server负责聚合多个数据中心的监控数据。这一特性在Promthues中称为联邦集群。 联邦集群的核心在于每一个Prometheus Server都包含一个用于获取当前实例中监控样本的接口/federate。对于中心Prometheus Server而言，无论是从其他的Prometheus实例还是Exporter实例中获取数据实际上并没有任何差异。
<!--more-->
```yaml
scrape_configs:
  - job_name: 'federate'
    scrape_interval: 15s
    honor_labels: true
    metrics_path: '/federate'
    params:
      'match[]':
        - '{job="prometheus"}'
        - '{__name__=~"job:.*"}'
        - '{__name__=~"node.*"}'
    static_configs:
      - targets:
        - '192.168.77.11:9090'
        - '192.168.77.12:9090'

```

为了有效的减少不必要的时间序列，通过params参数可以用于指定只获取某些时间序列的样本数据，例如

```
"http://192.168.77.11:9090/federate?match[]={job%3D"prometheus"}&match[]={__name__%3D~"job%3A.*"}&match[]={__name__%3D~"node.*"}"
```

通过URL中的match\[\]参数指定我们可以指定需要获取的时间序列。match\[\]参数必须是一个瞬时向量选择器，例如up或者{job="api-server"}。配置多个match\[\]参数，用于获取多组时间序列的监控数据。

# Promethues 高可用

Prometheus的本地存储给Prometheus带来了简单高效的使用体验，可以让Promthues在单节点的情况下满足大部分用户的监控需求。但是本地存储也同时限制了Prometheus的可扩展性，带来了数据持久化等一系列的问题。通过Prometheus的Remote Storage特性可以解决这一系列问题，包括Promthues的动态扩展，以及历史数据的存储。 而除了数据持久化问题以外，影响Promthues性能表现的另外一个重要因素就是数据采集任务量，以及单台Promthues能够处理的时间序列数。因此当监控规模大到Promthues单台无法有效处理的情况下，可以选择利用Promthues的联邦集群的特性，将Promthues的监控任务划分到不同的实例当中。

## 基本HA：服务可用性

由于Promthues的Pull机制的设计，为了确保Promthues服务的可用性，用户只需要部署多套Prometheus Server实例，并且采集相同的Exporter目标即可。 [![](https://i.loli.net/2021/09/01/QjLZJInClGbYyDg.jpg)](https://i.loli.net/2021/09/01/QjLZJInClGbYyDg.jpg) 基本的HA模式只能确保Promthues服务的可用性问题，但是不解决Prometheus Server之间的数据一致性问题以及持久化问题(数据丢失后无法恢复)，也无法进行动态的扩展。因此这种部署方式适合监控规模不大，Promthues Server也不会频繁发生迁移的情况，并且只需要保存短周期监控数据的场景。

## 基本HA + 远程存储

在基本HA模式的基础上通过添加Remote Storage存储支持，将监控数据保存在第三方存储服务上。 [![](https://i.loli.net/2021/09/01/TxZwACltDE1PaVp.jpg)](https://i.loli.net/2021/09/01/TxZwACltDE1PaVp.jpg) 在解决了Promthues服务可用性的基础上，同时确保了数据的持久化，当Promthues Server发生宕机或者数据丢失的情况下，可以快速的恢复。 同时Promthues Server可能很好的进行迁移。因此，该方案适用于用户监控规模不大，但是希望能够将监控数据持久化，同时能够确保Promthues Server的可迁移性的场景。

## 基本HA + 远程存储 + 联邦集群

当单台Promthues Server无法处理大量的采集任务时，用户可以考虑基于Prometheus联邦集群的方式将监控采集任务划分到不同的Promthues实例当中即在任务级别功能分区。 [![](https://i.loli.net/2021/09/01/R2NqcyouOUQmZzI.jpg)](https://i.loli.net/2021/09/01/R2NqcyouOUQmZzI.jpg) 这种部署方式一般适用于两种场景： 场景一：单数据中心 + 大量的采集任务 这种场景下Promthues的性能瓶颈主要在于大量的采集任务，因此用户需要利用Prometheus联邦集群的特性，将不同类型的采集任务划分到不同的Promthues子服务中，从而实现功能分区。例如一个Promthues Server负责采集基础设施相关的监控指标，另外一个Prometheus Server负责采集应用监控指标。再有上层Prometheus Server实现对数据的汇聚。 场景二：多数据中心 这种模式也适合与多数据中心的情况，当Promthues Server无法直接与数据中心中的Exporter进行通讯时，在每一个数据中部署一个单独的Promthues Server负责当前数据中心的采集任务是一个不错的方式。这样可以避免用户进行大量的网络配置，只需要确保主Promthues Server实例能够与当前数据中心的Prometheus Server通讯即可。 中心Promthues Server负责实现对多数据中心数据的聚合。

## 高可用方案选择

上面的部分，根据不同的场景演示了3种不同的高可用部署方案。当然对于Promthues部署方案需要用户根据监控规模以及自身的需求进行动态调整，下表展示了Promthues和高可用有关3个选项各自解决的问题，用户可以根据自己的需求灵活选择 [![](https://i.loli.net/2021/09/01/q129eHPwEoaL8gG.jpg)](https://i.loli.net/2021/09/01/q129eHPwEoaL8gG.jpg)

# 在K8S里面的监控方案

[![](https://i.loli.net/2021/09/01/3Ja9lGSOVyxoHLQ.jpg)](https://i.loli.net/2021/09/01/3Ja9lGSOVyxoHLQ.jpg)