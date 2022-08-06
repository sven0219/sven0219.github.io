---
title: Python 自动化运维 （01）
tags:
  - Python
  - 自动化
  - 运维
id: '11981'
categories:
  - - skills
date: 2020-03-21 23:03:46
---

# 前面的话

> 学习记录，记录中再次学习 所有代码基本上都是实际测试过 操作系统 macOS Catalina Python版本：3.7 所有代码可在 [GitHub](https://github.com/Swq1227/devops "GitHub") 获取
<!--more-->
# 系统性能监控模块 psutil

## 介绍

`psutil`是一个跨平台库，能实现获取系统运行的进程和系统利用率（包括 CPU、内存、磁盘、网络等）信息。主要应用于系统监控，分析和限制系统资源及进程的管理。 `psutil`是运维自动化中常用的一个模块。

## 安装

介绍两种比较简单的方法 方法一： 打开命令行，输入下面命令安装即可

```
pip install psutil
```

方法二： 使用 `Pycharm` 开发的话，`file->settings ->project interpreter`(mac系统的话选择`pychram->Preferences->setting->project interpreter`) ![image.png](https://i.loli.net/2020/03/21/jdZpi2DcIxMWvUq.png) 点击+搜索`psutil`点击 `install package` 即可 ![image.png](https://i.loli.net/2020/03/21/y9bg2odzqHBcawj.png)

## 使用

### CPU 信息

运维中，一般 CPU 需要知道以下几个部分

*   User Time，执行用户进程的时间百分比
*   System Time，执行内核进程和中断的时间百分比
*   Wait IO，由于 IO 等待而使 CPU 处于空闲状态的时间比
*   Idle，CPU 处于空闲状态的时间比

使用`psutil.cpu_times()`方法可以很容易获取到：

```python
import psutil
# 获取 cpu 完整信息，需要显示所有逻辑 cpu 信息
cpu_time=psutil.cpu_times()
print("cpu_time is:",cpu_time)
# 获取单项数据信息，如用户的 CPU 时间比
cpu_user=psutil.cpu_times().user
print("cpu_user is:",cpu_user)
# 获取 CPU 的逻辑个数,默认 logical=True
cpu_count=psutil.cpu_count()
print("count is :",cpu_count)
# 获取 CPU 的物理个数，logical=False
cpu_count2=psutil.cpu_count(logical=False)
print("count2 is :",cpu_count2)
```

运行结果 ![image.png](https://i.loli.net/2020/03/21/ZtUsnVqJRvhaB85.png)

### 内存信息

内存利用率信息设计 total（内存总数）、used（已使用的内存数）、free（空闲内存数）、buffers（缓冲使用数）、cache（缓存使用数）、swap（交换分区使用数）等，可分别使用`psutil.virtual_memory()`与`psutil.swap_memory()`方法获取这些信息

```python
import psutil

# 使用psutil.virtual_memory()获取内存完整信息
mem = psutil.virtual_memory()
print("memory info is", mem)
# 获取内存总数
print("mem total is :", mem.total)
# 获取空闲内存数
print("mem free is :", mem.free)
# 获取 swap 分区信息
swap = psutil.swap_memory()
print("swap is :", swap)

```

运行结果 ![image.png](https://i.loli.net/2020/03/21/nwSDsphkHWCQTdy.png)

### 磁盘信息

在磁盘信息中，关注的比较多的是磁盘的利用率及 IO 信息，其中磁盘利用率使用`psutil.disk_usage`方法获取。IO 信息可以使用`psutil_disk_io_counters()`获取

```python
import psutil

# 获取磁盘完整信息
disk_info = psutil.disk_partitions()
print("info:", disk_info)
# 获取分区使用情况,需要传一个 path 参数，这里我们取/路径
disk_usage = psutil.disk_usage('/')
print("usage info:", disk_usage)
# 获取磁盘总的 IO 个数、读写信息
io_counts = psutil.disk_io_counters()
print('io_counts:', io_counts)
# 获取单个分区的 IO 个数
io_counts2 = psutil.disk_io_counters(perdisk=True)
print(io_counts2)

```

运行结果 ![image.png](https://i.loli.net/2020/03/21/EmSM7bhaHunYOqr.png)

### 网络信息

网络信息与 IO 信息类似，涉及几个关键点，包括发送字节数、接收字节数、发送数据包数、接收数据包数。这些信息可以使用`psutil.net_io_counters()`方法获取

```python
import psutil

# 获取网络 IO 信息
net_io = psutil.net_io_counters()
print('net io :', net_io)
# 获取每个网络接口的 io 信息
net_io2 = psutil.net_io_counters(pernic=True)
print('every io:', net_io2)

```

运行结果 ![image.png](https://i.loli.net/2020/03/21/4OkmrvhIfe5ysuD.png)

### 其它系统信息

除了上面 cpu、内存、网络、磁盘等信息的获取，`psutil`模块还可以获取一些运维工作中也需要用到的信息，如用户登录、开机时间等

```python
import datetime

import psutil

# 获取当前用户信息
user = psutil.users()
print('now logined user info :', user)
# 获取开机时间，并格式化
startTime = psutil.boot_time()
print('boot time is :', startTime)
startTime = datetime.datetime.fromtimestamp(startTime).strftime("%Y-%m-%d")
print('format boot time is :', startTime)

```

运行结果 ![image.png](https://i.loli.net/2020/03/21/hvG5inmCfgcTaOl.png)

## 后面的话

psutil 还有很多在运维中经常用到的模块，可以查看该模块有哪些方法可使用到工作中