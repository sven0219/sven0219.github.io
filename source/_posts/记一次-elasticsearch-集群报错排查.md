---
title: 记一次 elasticsearch 集群报错排查
tags:
  - Elasticsearch
id: '12373'
categories:
  - - skills
date: 2021-04-07 16:59:47
---

# 问题现象

1.  Kibana 页面报错链接不是 elasticsearch 集群
2.  elasticsearch 索引信息状态为 red
3.  日志报错 all shards failed
<!--more-->
# 排查记录

1.  查看所有索引 `curl 'localhost:9288/_cat/indices?v`如下 [![](https://i.loli.net/2021/04/07/1obGtlNdk2vIUV8.jpg)](https://i.loli.net/2021/04/07/1obGtlNdk2vIUV8.jpg)

# 修复

修复所有节点上索引

```bash
 curl -XPOST "http://127.0.0.1:9288/_cluster/reroute?retry_failed=true"
```

再次查看

```
curl  'localhost:9288/_cat/indices?v
```

[![](https://i.loli.net/2021/04/07/h9MTXQNW2Fg45nL.jpg)](https://i.loli.net/2021/04/07/h9MTXQNW2Fg45nL.jpg) 打开 kibana 页面恢复正常。

# 故障原因

前段时间迁移了部分服务到容器中，将机器删除了，导致很多索引都失败了。