---
title: cisco面试记录
tags:
  - 记录
  - 面试
id: '12069'
categories:
  - - Essay
date: 2020-04-15 20:42:41
---

# 思科面试记录

面试的是合肥的思科，由于在上海，所以采取的是远程面试，使用的是思科自己的 Webex Meet

# 面试题目

#### 1\. Linux 开机加载顺序

1 加载BIOS硬件信息, 并获取第一个启动设备的代号; 2 读取第一个启动设备的MBR的引导加载程序的启动信息; 3 加载核心操作系统的核心信息, 核心开始解压缩, 并且尝试驱动所有的硬件设备; 4 核心执行init程序并获取运行信息; 5 init执行/etc/init.d的脚本; 6 启动核心的外挂模块/etc/modeprobe.d/中的脚本; 7 init执行运行各个批处理脚本; 8 init执行/etc/init.d/rc.local文件; 9 执行/bin/login程序, 等待用户登录; 10 登录之后开始以shell控制主机.
<!--more-->
#### 2\. 用户登录加载的配置文件顺序

登录式shell加载配置文件过程 `/etc/profile --> /etc/profile.d/*.sh --> ~/.bash_profile --> ~/.bashrc --> /etc/bashrc` 非登录式shell加载配置文件过程 `~/.bashrc --> /etc/bashrc --> /etc/profile.d/*.sh`

#### 3\. 安装软件的三种方式

*   在线安装（yum install）
*   离线安装（rpm 包）
*   编译安装

#### 4\. linux 查看进程打开的文件（lsof 使用）

```
# lsof (list open file)
 -p pid 列出进程pid所打开的所有文件

 -d FD_pattern 列出所有已经打开的FD值为FD_pattern的文件FD的值有整数，TXT,MEM等等

 -a lsof后可以加多个匹配条件，默认为or连接，此参数将多个条件变成and关系

 -i [46] [proto] [@hostnameip][:serviceport] 用来选择占用某个端口的进程

  +d/+D dir 非递归或递归的显示打开dir下文件的进程

  -c string   显示进程的command中包含string的进程所打开的文件

  -u username 显示属于user的进程所打开的文件
```

#### 5\. linux 定时任务（cron 表达式）

分钟 小时 日期 月份 星期

#### 6\. 卸载文件系统

umount

#### 7\. git pull request

有一个仓库，叫Repo A。你如果要往里贡献代码，首先要Fork这个Repo，于是在你的Github账号下有了一个Repo A2。 然后你在这个A2下工作，Commit，push等。然后你希望原始仓库Repo A合并你的工作，你可以在Github上发起一个Pull Request，意思是请求Repo A的所有者从你的A2合并分支。 如果被审核通过并正式合并，这样你就为项目A做贡献了。

#### 8\. jenkins 等持续集成工具的使用

基础流程说一下

#### 9\. shell 脚本监控磁盘使用率（占用高发邮件）

```bash
#!/bin/bash
#用途：监控磁盘的使用情况。
#把脚本名字存在变量l-name
l_name=`echo $0  awk -F '/' 'print $NF'`
#定义收件人的邮箱
mail_user=admin@admin.com

#定义检查磁盘的空间使用率函数
chk_sp()
{
    df -m  sed '1d'  awk -F '%  +' '$5>90 {print $7,$5}'>/tmp/chk_sp.log
    n=`wc -l /tmp/chk_sp.log  awk 'print $1'`
    if [ $n -gt 0 ]
    then 
      tag=1
      for d in `awk '{print $1}' /tmp/chk_sp.log`
      do 
           find $d -type d  sed '1d'  xargs du -sm  sort -nr  head -3
      done >/tmp/most_sp.txt
   fi

}

#定义检查inode使用率函数

chk_in()
{
  df -i  sed `1d`  awk -F '%  +' '$5>90 {print $7,$5}'>/tmp/chk_in.log
    n=`wc -l /tmp/chk_in.log  awk '{print $1}'`
    if [ $n -gt 0 ]
    then 
        tag=2
    fi
 }

#定义告警函数

m_mail（）{
    log=$1
    t_s=`date +%s`
    t_s2=`data -d "1 hours ago" +%s`
    if [ ! -f /tmp/$log ]
    then
        #创建$log文件
        touch /tmp/$log
        #增加a权限，只允许追加内容，不允许更改或删除
        chattr +a /tmp/$log
        #第一次告警，可以直接写入1小时以前的时间戳
        echo $t_s2 >> /tmp/$log
     fi
    #无论#log文件是否刚刚创建，都需要查看最后一行的时间戳
    t_s2=`tail -l /tmp/$log  awk '{print $1}'`
    # 取出最后一行及上次告警的时间戳，立即写入当期的时间戳
    echo $t_s >>/tmp/$log
    #取两次时间戳差值
    v=$[ $t_s-$t_s2 ]
    #如果差值超过100，立即发送邮件。
    if [ $v -gt 1800 ]
    then
      #发邮件，其中$2为mail函数的第二个函数，这里为一个文件
      python  mail.py $mail_user "磁盘使用率超过90%"
      #定义技数器临时文件，并写入0
      echo "0" > /tmp/$log.count
     else
      #如果技数器临时文件不存在，需要创建并写入0
      if [ ! -f /tmp/$log.count }
      then
         echo "0" > /tmp/$log.count 

      fi

      nu=`cat /tmp/$log.count`
      #30分钟内每发生1次告警，计算器加1
      nu2=$[ $nu+1 ]
      echo $nu2>/tmp/$log.count
      #当告警次数超过30次，需要再次发油件
      if [ $nu2 -gt 30 ]
      then 
          python mail.py $mail_user "磁盘使用率90%持续30分钟了" "`cat $2`" 2>/dev/null
          #第二次告警后，将计算器再次从0开始
          echo "0" > /tmp/$log.count
      fi
 fi 
}
#把进程数大于0.则说明上次的脚本还未执行完
if [ $p_n -gt 0 ]
then
    exit

fi


chk_sp
chk_in

if [ $tag == 1 ]
then
    m_mail chk_sp /tmp/most_sp.txt
elif [ $tag == 2 ]
then
    m_mail chk_in /tmp/chk_in.log
fi
```

#### 10\. Python 基本的脚本

基本模块

#### 11\. mysql 主从复制原理

(1)首先，mysql主库在事务提交时会把数据库变更作为事件Events记录在二进制文件binlog中；mysql主库上的sys\_binlog控制binlog日志刷新到磁盘。 (2)主库推送二进制文件binlog中的事件到从库的中继日志relay log,之后从库根据中继日志重做数据库变更操作。通过逻辑复制，以此来达到数据一致。 Mysql通过3个线程来完成主从库之间的数据复制：其中BinLog Dump线程跑在主库上，I/O线程和SQl线程跑在从库上。当从库启动复制（start slave）时，首先创建I/O线程连接主库，主库随后创建Binlog Dump线程读取数据库事件并发给I/O线程，I/O线程获取到数据库事件更新到从库的中继日志Realy log中去，之后从库上的SQl线程读取中继日志relay log 中更新的数据库事件并应用。

#### 12\. mysql 主从延迟的原因（优化方法）

原因1：主库的从库太多，导致复制延迟 从库数量一般 3—5个为宜，要复制的节点过多，导致复制延迟。 原因2：从库硬件配置比主库差，导致延迟 查看Master和Slave的配置，可能因为配置不当导致复制的延迟 原因3：慢SQL语句过多 假如一条语句执行时间超过2秒， 就需要进行优化进行调整 原因4：主从复制设计问题 主从复制单线程，如果主库的写入并发太大，来不及传送到从库，就会导致延迟，更高版本的MySQL可以支持多线程复制，门户网站则会自己 开发多线程同步功能。 原因5：主从库之间的网络延迟 主从库网卡、网线、连接的交换机等网络设备都可能成为复制的瓶颈，导致复制延迟，另外跨公网主从复制很容易导致主从复制延迟。 原因6：主库读写压力大，导致复制延迟 主库硬件要好一些，架构前端要加buffer缓存层。

#### 13\. mysql 性能监控（几方面，脚本）

*   查询吞吐量
*   查询执行性能
*   连接情况
*   缓冲池使用情况

#### 14\. Nginx 调度算法有哪几种

```
1、轮询

按时间顺序逐一分配到不同的后端服务器。

    upstream lb_demo {
        server 172.16.255.194:9001;
        server 172.16.255.195:9001;
    }
2、加权轮询

可在配置的server后面加个weight=number，number值越高，分配的概率越大。

upstream lb_demo {
        server 172.16.255.194:9001 weight=10;
        server 172.16.255.195:9001 weight=20;
    }


3、ip_hash

每个请求按访问IP的hash分配，这样来自同一IP固定访问一个后台服务器。

upstream lb_demo {
        ip_hash;
        server 172.16.255.194:9001;
        server 172.16.255.195:9001;
    }
4、least_hash

最少链接数，哪个机器连接数少就发分发给哪个机器。

upstream lb_demo {
        least_conn;
        server 172.16.255.194:9001;
        server 172.16.255.195:9001;
    } 
5、url_hash
按访问的url的hash结果分配请求，是每个url定向到同一后端服务器上。
upstream lb_demo {
        url_hash;
        server 172.16.255.194:9001;
        server 172.16.255.195:9001;
    }
6、hash关键值
hash自定义的key。
注：调度算法在设置upstream中配置，例如在此大括号里面写入ip_hash表示使用ip_hash的方式分配
轮询只是简单实现请求的顺序转发，并没有考虑不同服务器的性能差异；
加权轮询设置了初始时服务器的权重，但是没有考虑运行过程中的服务器状态；
IP Hash保证同一个客户端请求转发到同一个后台服务器实现了session保存，然而当某一后台服务器发生故障时，某些客户端将访问失败；\
最少连接数只是考虑了后端服务器的连接数情况，并没有完全考虑服务器的整体性能。
```

#### 15\. Docker 中 cmd 和 entrypoint 的区别

　1. Dockerfile必须至少指定CMD或者ENTRYPOINT其中的一个。 　2. ENTRYPOINT应该用作容器的主执行程序。 　3. CMD应该用于定义ENTRYPOINT的默认参数，或者为容器执行一个ad-hoc命令。 　4. 当启动容器时使用交互时的参数时，CMD命令会被覆盖。

#### 16\. Docker 中 namesapce 资源隔离的方式

**namespace（命名空间）可以隔离哪些** 文件系统需要是被隔离的 网络也是需要被隔离的 进程间的通信也要被隔离 针对权限，用户和用户组也需要隔离 进程内的PID也需要与宿主机中的PID进行隔离 容器也要有自己的主机名 有了以上的隔离，我们认为一个容器可以与宿主机和其他容器是隔离开的。 **使用Namespace进行容器的隔离有什么缺点呢？** 　　最大的缺点就是隔离不彻底 　　1）容器知识运行在宿主机上的一种特殊的进程，那么多个容器之间使用的就还是同一个宿主机的操作系统内核 　　2）在Linux内核中，有很多资源和对象是不能被Namespace化的，最典型的例子是：时间即如果某个容器修改了时间，那整个宿主机的时间都会随之修改 　　3）容器给应用暴露出来的攻击面比较大，在生产环境中，没有人敢把运行在物理机上的Linux容器暴露在公网上

#### 17\. Linux 如何跟踪网络请求

traceroute

#### 18.Nginx 和 Apache 的区别

**nginx 相对 apache 的优点：** 轻量级，同样起web 服务，比apache 占用更少的内存及资源 抗并发，nginx 处理请求是异步非阻塞的，而apache 则是阻塞型的，在高并发下nginx 能保持低资源低消耗高性能 高度模块化的设计，编写模块相对简单 社区活跃，各种高性能模块出品迅速啊 **apache 相对nginx 的优点：** rewrite ，比nginx 的rewrite 强大 模块超多，基本想到的都可以找到 少bug ，nginx 的bug 相对较多 超稳定 # 结果 问的太难了，很多都不会，一轮就结束了