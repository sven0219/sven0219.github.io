---
title: MySQL 高可用方案
tags:
  - Linux
  - MySQL
  - 软件安装
  - 运维
id: '11973'
categories:
  - - skills
date: 2020-03-21 13:26:31
---

# MySQL 高可用方案

## 0.介绍

Keepalived+mysql双主来实现MySQL-HA，我们必须保证两台MySQL数据库的数据完全一样，基本思路是两台MySQL互为主从关系，通过Keepalived配置虚拟IP，实现当其中的一台MySQL数据库宕机后，应用能够自动切换到另外一台MySQL数据库，保证系统的高可用。

## 1\. 环境准备

角色
<!--more-->
ip

master1

192.168.53.199

master2

192.168.53.191

Vip（虚拟 ip）

192.168.53.100

关闭服务器防火墙或开启 3306 端口

```bash
# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld
# 开启 3306 端口
firewalld-cmd --permanent -add-port=3306/tcp
firewall-cmd --reload
```

安装所需工具

```bash
yum install wget vim kernel-devel openssl-devel poptdevel  -y
```

## 2\. 在两台机器上分别安装 mysql5.7

**两台机器都要执行以下操作** 安装 mysql

```bash
wget http://dl.qiyuesuo.com/private/mysql/mysql57-community-release-el7-11.noarch.rpm
rpm -Uvh mysql57-community-release-el7-11.noarch.rpm
yum install mysql-community-server
# 启动服务
service mysqld start
```

查看初始密码

```bash
grep 'temporary password' /var/log/mysqld.log
# 2019-09-17T01:46:58.792184Z 1 [Note] A temporary password is generated for root@localhost: )ecv9hQMVpkO
# 该初始密码为 )ecv9hQMVpkO

```

修改初始密码

```mysql
mysql -uroot -p
# 输入初始密码 )ecv9hQMVpkO
# 修改初始密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Admin#1234';
```

## 3\. 配置两台 mysql 主主同步

修改 master1 配置文件如下

```shell
cat /etc/my.cnf


datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
server_id=1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
log-bin=mysql-bin
binlog-format=mixed
relay-log=relay-bin
relay-log-index=slave-relay-bin.index
auto-increment-increment=2
auto-increment-offset=1
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

# 重启
service mysqld restart
```

修改 master2 配置文件如下

```shell
cat /etc/my.cnf


datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
server_id=2
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
log-bin=mysql-bin
binlog-format=mixed
relay-log=relay-bin
relay-log-index=slave-relay-bin.index
auto-increment-increment=2
auto-increment-offset=2
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

# 重启
service mysqld restart
```

部分解释

```text
binlog_format= mixed：指定mysql的binlog日志的格式，mixed是混合模式。

relay-log：开启中继日志功能

relay-log-index：中继日志清单

auto-increment-increment= 2：表示自增长字段每次递增的量，其默认值是1。它的值应设为整个结构中服务器的总数，本案例用到两台服务器，所以值设为2。

auto-increment-offset= 2：用来设定数据库中自动增长的起点(即初始值)，因为这两能服务器都设定了一次自动增长值2，所以它们的起点必须得不同，这样才能避免两台服务器数据同步时出现主键冲突。

注：另外还可以在my.cnf配置文件中，添加“binlog_do_db=数据库名”配置项（可以添加多个）来指定要同步的数据库。如果配置了这个配置项，如果没添加在该配置项后面的数据库，则binlog不记录它的事件。
```

#### 3.1 将 master1 设为 master2 的主服务器

在 master1 上创建授权账户，允许 master2 主机上连接

```bash
# 登陆 mysql
mysql -uroot -pAdmin#1234
# 创建用户
mysql> grant replication slave on *.* to 'qiyuesuo_bak'@'192.168.53.191' identified by "Qiyuesuo@2019";
# 查看当前 binlog 状态
mysql> show master status
    -> ;
+------------------+----------+--------------+------------------+-------------------+
 File              Position  Binlog_Do_DB  Binlog_Ignore_DB  Executed_Gtid_Set 
+------------------+----------+--------------+------------------+-------------------+
 mysql-bin.000001       459                                                    
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

在 master2 上将 master1 设置为自己的主服务器并开启 slave 功能

```bash
# 登陆 mysql
mysql -uroot -pAdmin#1234
# 设置
mysql> change master to master_host='192.168.53.199',master_user='qiyuesuo_bak',master_password='Qiyuesuo@2019',master_log_file='mysql-bin.000001',master_log_pos=459;
# 开启
mysql>start slave;
# 查看状态
mysql> Show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.53.199
                  Master_User: qiyuesuo_bak
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 459
               Relay_Log_File: relay-bin.000002
                Relay_Log_Pos: 672
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 459
              Relay_Log_Space: 873
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
                  Master_UUID: 0963db0f-d8ed-11e9-85fd-001c427d374f
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)
```

#### 3.2 将 master2 设为 master1 的主服务器

在 master2 上创建授权账户，允许 master1 主机上连接

```bash
# 登陆 mysql
mysql -uroot -pAdmin#1234
# 创建用户
mysql> grant replication slave on *.* to 'qiyuesuo_bak'@'192.168.53.199' identified by "Qiyuesuo@2019";
# 查看当前 binlog 状态
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
 File              Position  Binlog_Do_DB  Binlog_Ignore_DB  Executed_Gtid_Set 
+------------------+----------+--------------+------------------+-------------------+
 mysql-bin.000001       459                                                    
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

在 master1 上将 master2 设置为自己的主服务器并开启 slave 功能

```bash
# 登陆 mysql
mysql -uroot -pAdmin#1234
# 设置
mysql> change master to master_host='192.168.53.191',master_user='qiyuesuo_bak',master_password='Qiyuesuo@2019',master_log_file='mysql-bin.000001',master_log_pos=459;
# 开启
mysql> start slave;
Query OK, 0 rows affected (0.00 sec)
# 查看状态
mysql> show slave status \G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.53.191
                  Master_User: qiyuesuo_bak
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 459
               Relay_Log_File: relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 459
              Relay_Log_Space: 521
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 2
                  Master_UUID: 0524b2b4-d8ed-11e9-8681-001c42223706
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)
```

#### 3.3 测试主主同步

在 master1 上创建要同步的数据库，并在数据库中创建一张测试表

```mysql
mysql> create database  qiyuesuo_test;
Query OK, 1 row affected (0.00 sec)

mysql> use qiyuesuo_test;
Database changed
mysql> create table test(id int primary key  auto_increment,name varchar(20) );
Query OK, 0 rows affected (0.01 sec)
```

在 master2 上面查看是否同步过来

```mysql
mysql> show databases;
+--------------------+
 Database           
+--------------------+
 information_schema 
 mysql              
 performance_schema 
 qiyuesuo_test      
 sys                
+--------------------+
5 rows in set (0.00 sec)

mysql> use qiyuesuo_test;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-------------------------+
 Tables_in_qiyuesuo_test 
+-------------------------+
 test                    
+-------------------------+
1 row in set (0.00 sec)
```

说明 mater2 同步了 msater1 的数据 现在测试 master1 同步 master2，在 master2 中向 test 表插入一条数据

```mysql
mysql>insert into test(name) values('zhangsan');
```

在 master1 上查看

```mysql
mysql> select * from test;
+----+----------+
 id  name     
+----+----------+
  2  zhangsan 
+----+----------+
1 row in set (0.00 sec)
```

以上就完成了 mysql 双主的安装配置，下面进行配置keepalived

## 4\. keepalived 安装配置

#### 4.1 介绍

> keepalived是集群管理中保证集群高可用的一个软件解决方案，其功能类似于heartbeat，用来防止单点故障 keepalived是以VRRP协议为实现基础的，VRRP全称VirtualRouter Redundancy Protocol，即虚拟路由冗余协议。 虚拟路由冗余协议，可以认为是实现路由器高可用的协议，即将N台提供相同功能的路由器组成一个路由器组，这个组里面有一个master和多个backup，master上面有一个对外提供服务的vip，master会发组播（组播地址为224.0.0.18），当backup收不到vrrp包时就认为master宕掉了，这时就需要根据VRRP的优先级来选举一个backup当master，这样的话就可以保证路由器的高可用了。 keepalived主要有三个模块，分别是core 、check和vrrp。core模块为keepalived的核心，负责主进程的启动、维护以及全局配置文件的加载和解析。check负责健康检查，包括常见的各种检查方式。vrrp模块是来实现VRRP协议的。

#### 4.2 keepalived 安装

```bash
yum install -y keepalived
```

#### 4.3 keepalived 配置

在 master1 中配置

```bash
cat /etc/keepalived/keepalived.conf

! Configuration File for keepalived

global_defs {
  router_id mysql-1
}

vrrp_instance VI_1 {
   state BACKUP
   interface eth0
   virtual_router_id 51
   priority 100
   advert_int 1
   nopreempt
   authentication {
       auth_type PASS
       auth_pass 1111
   }

   virtual_ipaddress {

       192.168.53.100

   }

}

virtual_server 192.168.53.100 3306 {

   delay_loop 2
   lb_algo rr
   lb_kind DR
   persistence_timeout 60
   protocol TCP

   real_server 192.168.53.199 3306 {
       weight 3
       notify_down    /etc/keepalived/bin/mysql.sh
       TCP_CHECK {
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
            connect_port 3306
       }
   }
}
```

在 master2 中配置

```
cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
  router_id mysql-2
}

vrrp_instance VI_1 {
   state BACKUP
   interface eth0
   virtual_router_id 51
   priority 50
   advert_int 1
   authentication {
       auth_type PASS
       auth_pass 1111
   }

   virtual_ipaddress {

       192.168.53.100

   }

}

virtual_server 192.168.53.100 3306 {

   delay_loop 2
   lb_algo rr
   lb_kind DR
   persistence_timeout 60
   protocol TCP

   real_server 192.168.53.191 3306 {
       weight 3
       notify_down    /etc/keepalived/bin/mysql.sh
       TCP_CHECK {
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
            connect_port 3306
       }
   }
}
```

解释说明

```bash
! Configuration File for keepalived  //注释

global_defs {
  router_id mysql-1   //表示运行 keepalived 服务器的一个标识
}

vrrp_instance VI_1 {
   state BACKUP       // 指定角色，两台配置均为 BACKUP，设为 BACKUP 将根据优先级决定主从
   interface eth0     // 指定 HA 检测网络的接口，可使用 ip addr 命令查找
   virtual_router_id 51  // 虚拟路由标识，同一个 vrrp 实例要使用唯一的标识，确保和 master2 相同
   priority 100   //用来选举 master，取值 1-255，此处 master2 设置为 50
   advert_int 1   // 发 vrrp 间隔，即多久进行一次 master 选举
   nopreempt      // 不抢占，即允许一个 priority 比较低的节点作为 master
   authentication {
       auth_type PASS   //认证区域
       auth_pass 1111
   }

   virtual_ipaddress {

       192.168.53.100 //vip ,指定 vip 地址

   }

}

virtual_server 192.168.53.100 3306 { // 设置虚拟服务器，需要指定虚拟 IP地址和端口，ip 与端口之间用空格隔开

   delay_loop 2 // 设置允许情况检查时间，单位秒
   lb_algo rr //设置后端调度算法，这里设置 rr，即轮询算法
   lb_kind DR  // 设置 LVS 实现负载均衡的机制，
   persistence_timeout 60 //会话保持时间
   protocol TCP  // 指定转发协议，有 tcp 和 udp


   real_server 192.168.53.199 3306 {  //配置服务节点
       weight 3     //权值
       notify_down    /etc/keepalived/bin/mysql.sh  // 检测到 realserver 的 mysql 服务 down 后执行的脚本
       TCP_CHECK {
            connect_timeout 3  // 连接超时时间
            nb_get_retry 3    // 重连次数
            delay_before_retry 3 // 重连间隔时间
            connect_port 3306  //健康检查端口
       }
   }
}
```

启动 keepalived

```bash
service keepalived start
# 设置开机自启
systemctl enable keepalived
```

在 master1 上查看

```bash
[root@localhost ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:1c:42:22:37:06 brd ff:ff:ff:ff:ff:ff
    inet 192.168.53.191/22 brd 192.168.55.255 scope global noprefixroute dynamic eth0
       valid_lft 69601sec preferred_lft 69601sec
    inet 192.168.53.100/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::21c:42ff:fe22:3706/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

在 master2 上查看

```bash
[root@localhost keepalived]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:1c:42:7d:37:4f brd ff:ff:ff:ff:ff:ff
    inet 192.168.53.199/22 brd 192.168.55.255 scope global noprefixroute dynamic eth0
       valid_lft 69567sec preferred_lft 69567sec
    inet6 fe80::21c:42ff:fe7d:374f/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

#### 4.4 添加检测脚本

```bash
mkdir /etc/keepalived/bin
vim /etc/keepalived/bin/mysql.sh
# 内容如下
#!/bin/bash
pkill keepalived
/sbin/ifdown eth0 && /sbin/ifup eth0

# 赋予可执行权限
chmod +x /etc/keepalived/bin/mysql.sh

```

连接即可使用192.168.53.100创建数据库使用

```shell
# 在 host1 上修改
! Configuration File for keepalived

global_defs {  #主要是配置故障发生时的通知对象以及机器标识。
   router_id HA_MySQL  #标识，双主相同
}

vrrp_instance VI_1 {  #用来定义对外提供服务的VIP区域以及机器标识
    state BACKUP  #注意，主从两端都配置成了backup，因为使用了nopreempt,即非抢占模式
    interface eth0
    virtual_router_id 51  #分组，主备相同
    priority 100  #优先级，这个高一点则先把他当作为master
    advert_int 1
    nopreempt  #不主动抢占资源，设置非抢占模式
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.10
    }
}

virtual_server 192.168.1.10 3306 {  #设置虚拟服务器，需要指定虚拟IP地址和服务端口，IP与端口直接用空格隔开
    delay_loop 2  #设置运行情况检查时间，单位是秒
    lb_algo wrr   #设置后端调度算法，rr为轮询算法，这里设置的wrr是带有权重的轮询算法
    lb_kind DR    #设置LVS实现负载均衡的机制。有NAR、TUN、DR三个模式可选
    persistence_timeout 60  #回话保持的时间，单位是秒。这个选项对动态网页是非常有用的，为了集群系统中的session
                            #共享提供了一个很好的解决方案。有了这个会话保持功能，用户的请求会被一直分发到某个服务节点，
                            #直到超过这个会话的保持时间。
   protocol TCP   #指定转发协议类型，有TCP和UDP两种

    real_server 192.168.1.51 3306 {  #配置服务节点1，需要制定real server的真实IP地址和端口号，IP与端口之间用空格隔开
        weight 3   #配置服务节点的权值，权值大小用数字表示，数字越大，权值越高，设置全职大小为分区不同性能的服务器
        notify_down /usr/local/keepalived/mysql.sh #检测到real server的mysql服务down后执行的脚本
        TCP_CHECK {
            connect_timeout 3  #连接超时时间
            nb_get_retry 3     #重连次数
            delay_before_retry 3 #重连间隔时间
            connect_port 3306  #健康检查端口，设置车工自己mysql的服务端口

     } 
  }
}


# 在 hosts2 上修改
! Configuration File for keepalived

global_defs {  
   router_id HA_MySQL  
}

vrrp_instance VI_1 {  
    state BACKUP  
    interface eth0
    virtual_router_id 51  
    priority 90   #优先级，这个要低一点
    advert_int 1
    #nopreempt  #这里的nopreempt(即非抢占模式)去掉，因为该配置项一般只在优先级高的mysql上配置 
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.10
    }
}
virtual_server 192.168.1.52 3306 {  
    delay_loop 2 
    lb_algo wrr   
    lb_kind DR   
    persistence_timeout 60                            
    protocol TCP   

    real_server 192.168.1.9 3306 {  
        weight 3   
        notify_down /usr/local/keepalived/mysql.sh 
        TCP_CHECK {
            connect_timeout 3  
            nb_get_retry 3    
            delay_before_retry 3 
            connect_port 3306  

    } 
  }
}

```

配置杀死 keepalived 的脚本 vi /usr/local/keepalived/mysql.sh

```bash
#!/bin/bash
#kill掉keepalived进程，以防止脑裂问题。
pkill keepalived
```

启动keepalived服务（两台都要启动）

```bash
chmod +x /usr/local/keepalived/mysql.sh
/etc/init.d/keepalived start
```

参考文档：https://blog.csdn.net/shiyu1157758655/article/details/78672110