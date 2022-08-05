---
title: Nginx 高可用方案
tags:
  - nginx
  - 运维
id: '11934'
categories:
  - - skills
date: 2020-03-19 12:12:26
---

# 介绍

### 什么是高可用

高可用`HA（High Availability）`是分布式系统架构设计中必须考虑的因素之一，它通常是指，通过设计减少系统不能提供服务的时间。如果一个系统能够一直提供服务，那么这个可用性则是百分之百，但是天有不测风云。所以我们只能尽可能的去减少服务的故障。

### 解决什么问题

在生产环境上很多时候是以`Nginx`做反向代理对外提供服务，但是一天`Nginx`难免遇见故障，如：服务器宕机。当`Nginx`宕机那么所有对外提供的接口都将导致无法访问。 虽然我们无法保证服务器百分之百可用，但是也得想办法避免这种悲剧，今天我们使用`keepalived`来实现`Nginx`的高可用。

### 双机热备方案

这种方案是国内企业中最为普遍的一种高可用方案，双机热备其实就是指一台服务器在提供服务，另一台为某服务的备用状态，当一台服务器不可用另外一台就会顶替上去。

#### keepalived是什么？

`Keepalived`软件起初是专为`LVS`负载均衡软件设计的，用来管理并监控LVS集群系统中各个服务节点的状态，后来又加入了可以实现高可用的`VRRP`(`Virtual Router Redundancy Protocol` ,虚拟路由器冗余协议）功能。因此，`Keepalived`除了能够管理`LVS`软件外，还可以作为其他服务（例如：`Nginx`、`Haproxy`、`MySQL`等）的高可用解决方案软件

#### 故障转移机制

`Keepalived`高可用服务之间的故障切换转移，是通过`VRRP` 来实现的。 在 `Keepalived`服务正常工作时，主 `Master`节点会不断地向备节点发送（多播的方式）心跳消息，用以告诉备`Backup`节点自己还活着，当主 `Master`节点发生故障时，就无法发送心跳消息，备节点也就因此无法继续检测到来自主 `Master`节点的心跳了，于是调用自身的接管程序，接管主`Master`节点的 `IP`资源及服务。而当主 `Master`节点恢复时，备`Backup`节点又会释放主节点故障时自身接管的`IP`资源及服务，恢复到原来的备用角色。

# 准备

机器都为 `Centos7` 操作系统 机器 1：`192.168.1.140`（主） 机器 2：`192.168.1.141`（从） 虚拟 ip：`192.168.1.150`

# 安装 Nginx

```
# 安装
yum install nginx -y
# 启动停止
systemctl start nginx 
systemctl stop nginx 
```

# 安装 keepalived

```
yum -y install keepalived
```

# 配置高可用

#### 修改机器 1（192.168.1.140）配置文件

```
vim /etc/keepalived/keepalived.conf
```

`keepalived.conf`

```
#检测脚本
vrrp_script chk_http_port {
    script "/usr/local/src/check_nginx_pid.sh" #心跳执行的脚本，检测nginx是否启动
    interval 2                          #（检测脚本执行的间隔，单位是秒）
    weight 2                            #权重
}
#vrrp 实例定义部分
vrrp_instance VI_1 {
    state MASTER            # 指定keepalived的角色，MASTER为主，BACKUP为备
    interface ens33         # 当前进行vrrp通讯的网络接口卡(当前centos的网卡) 用ifconfig查看你具体的网卡
    virtual_router_id 66    # 虚拟路由编号，主从要一直
    priority 100            # 优先级，数值越大，获取处理请求的优先级越高
    advert_int 1            # 检查间隔，默认为1s(vrrp组播周期秒数)
    #授权访问
    authentication {
        auth_type PASS #设置验证类型和密码，MASTER和BACKUP必须使用相同的密码才能正常通信
        auth_pass 1111
    }
    track_script {
        chk_http_port            #（调用检测脚本）
    }
    virtual_ipaddress {
        192.168.1.150            # 定义虚拟ip(VIP)，可多设，每行一个
    }
}
```

#### 修改机器 2（192.168.1.141）配置文件

```
vim /etc/keepalived/keepalived.conf
```

`keepalived.conf`

```
#检测脚本
vrrp_script chk_http_port {
    script "/usr/local/src/check_nginx_pid.sh" #心跳执行的脚本，检测nginx是否启动
    interval 2                          #（检测脚本执行的间隔）
    weight 2                            #权重
}
#vrrp 实例定义部分
vrrp_instance VI_1 {
    state BACKUP                        # 指定keepalived的角色，MASTER为主，BACKUP为备
    interface ens33                      # 当前进行vrrp通讯的网络接口卡(当前centos的网卡) 用ifconfig查看你具体的网卡
    virtual_router_id 66                # 虚拟路由编号，主从要一直
    priority 99                         # 优先级，数值越大，获取处理请求的优先级越高
    advert_int 1                        # 检查间隔，默认为1s(vrrp组播周期秒数)
    #授权访问
    authentication {
        auth_type PASS #设置验证类型和密码，MASTER和BACKUP必须使用相同的密码才能正常通信
        auth_pass 1111
    }
    track_script {
        chk_http_port                   #（调用检测脚本）
    }
    virtual_ipaddress {
        192.168.16.130                   # 定义虚拟ip(VIP)，可多设，每行一个
    }
}
```

#### 添加检测脚本

```
vim /usr/local/src/check_nginx_pid.sh
```

`check_nginx_pid.sh`

```
#!/bin/bash
#检测nginx是否启动了
A=`ps -C nginx --no-header wc -l`        
if [ $A -eq 0 ];then    #如果nginx没有启动就启动nginx                        
      systemctl start nginx                #重启nginx
      if [ `ps -C nginx --no-header wc -l` -eq 0 ];then    #nginx重启失败，则停掉keepalived服务，进行VIP转移
              killall keepalived                    
      fi
fi
```

#### 脚本授权

```
# 赋予可执行权限
chmod +x /usr/local/src/check_nginx_pid.sh
```

#### 模拟nginx故障：

修改两个服务器默认访问的Nginx的html页面作为区别。 首先访问192.168.1.150,通过vip进行访问，页面显示192.168.1.140；说明当前是主服务器提供的服务。 这个时候192.168.1.140主服务器执行命令：

```
systemctl stop nginx; #停止nginx
```

再次访问vip(192.168.1.150)发现这个时候页面显示的还是：192.168.1.140，这是脚本里面自动重启。 现在直接将192.168.1.140服务器关闭，在此访问vip(192.168.1.150)现在发现页面显示192.168.1.141这个时候keepalived就自动故障转移了，一套企业级生产环境的高可用方案就搭建好了。