---
title: Python 自动化运维（02）
tags:
  - Python
  - 自动化
  - 运维
id: '12018'
categories:
  - - skills
date: 2020-03-24 21:04:04
---

> 操作系统 macOS Catalina Python版本：3.7 所有代码可在 [GitHub](https://github.com/Swq1227/devops "GitHub") 获取
<!--more-->
# IP 地址处理模块 IPy

IP 地址规划是网络设计中的一个重要环节，IPy 模块可以很好的辅助我们高效的完成 IP 的规划工作。 安装过程就不再赘述了，和安装 psutil 一样的方式即可。

### IP 地址、网段的基本处理

IPy 模块包含 IP 类,可以输出指定的网段的 IP 个数及所有的 IP 地址清单。

```python
from IPy import IP

ip = IP('192.168.0.0/16')
# 输出个数
print(ip.len())
# 打印所有 ip 地址清单
for x in ip:
    print(x)
```

运行结果： ![image.png](https://i.loli.net/2020/03/24/y4smxnTJjouh7B3.png) IP 方法也支持网络地址的转换，例如根据 IP 地址与掩码生产网段格式

```python
from IPy import IP

print(IP('192.168.1.0').make_net('255.255.255.0'))
print(IP('192.168.1.0/255.255.255.0', make_net=True))
print(IP('192.168.1.0-192.168.1.255', make_net=True))

```

运行结果 ![image.png](https://i.loli.net/2020/03/24/fcZxjQC3hWRua65.png)

### 多网络计算方法

有时候我么想比较两个网段是否存在包含、重叠关系。IPy 支持类似于述职型数据的比较，以帮助 IP 对象进行比较

```python
>>> IP('192.168.0.0/23').overlaps('192.168.0.0/24')
1 #返回 1 代表存在重叠
>>>  IP('192.168.0.0/24').overlaps('192.168.2.0')
0 # 返回 0 代表不存在重叠
```

下面写个例子，根据输入的 IP 或子网返回网络、掩码、广播、反向解析、子网数、IP 类型等信息

```python
from IPy import IP

# 接受用户输入的 IP 地址或网段地址
ip_s = input('Please input an IP or net-range:')
ips = IP(ip_s)
# 如果长度大于 1 那就是一个网段地址
if len(ips) > 1:
    print('net:', ips.net())  # 输出网络地址
    print('netmask:', ips.netmask())  # 输出网络掩码地址
    print('broadcast:', ips.broadcast())  # 输出网络广播地址
    print('reverse address:', ips.reverseName()[0])  # 输出地址反向解析
    print('subnet:', len(ips))  # 输出网络子网数
else:
    print('reverse address:', ips.reverseName()[0])  # 输出地址反向解析

print('hexadecimal:', ips.strHex())  # 输出十六进制地址
print('binary ip', ips.strBin())  # 输出二进制地址
print('ipType:', ips.iptype())  # 输出地址类型

```

先尝试输入网段，运行结果如下 ![image.png](https://i.loli.net/2020/03/24/TaYiIyFgzWGDtZM.png) 输入 ip 地址，运行结果如下 ![image.png](https://i.loli.net/2020/03/24/AfkJTG2OWbKUp9q.png)