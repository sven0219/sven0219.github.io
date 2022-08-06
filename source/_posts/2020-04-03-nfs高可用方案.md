---
title: NFS高可用方案
tags:
  - NFS
  - 教程
  - 运维
id: '12056'
categories:
  - - skills
date: 2020-04-03 19:58:36
---

# NFS 高可用方案（NFS+keepalived+Sersync）

## 1\. 简述

### 1.1 介绍

本方案 NFS 的高可用方案，应用服务器为 Client ，两台文件服务器分别Master和 Slave，使用 keepalived 生成一个虚拟 IP，使用 Sersync 进行 Master 与 Slave 之间文件相互同步，确保高可用。

### 1.2 网络拓扑图

当 Master NFS 服务正常时 [![](https://i.loli.net/2020/06/17/CyFrMB2T7WuVk6S.jpg)](https://i.loli.net/2020/06/17/CyFrMB2T7WuVk6S.jpg) 当 Master NFS 服务宕机时 [![](https://i.loli.net/2020/06/17/hZY89BAksxK4WN2.jpg)](https://i.loli.net/2020/06/17/hZY89BAksxK4WN2.jpg)
<!--more-->
## 2.安装前准备

服务器信息：

角色

系统版本

ip

虚拟 ip（Vip）

无

192.168.50.143

Client

centos 7.5

192.168.51.246

Master

centos 7.5

192.168.50.8

Slave

centos 7.5

192.168.50.71

服务器环境准备： 在 Master 和 Slave 上创建共享目录

```bash
mkdir /data
```

在 Client 上创建挂载目录

```bash
mkdir /qiyuesuodata
```

关闭 Client 、Master 和 Slave 服务器上的防火墙

```bash
# 关闭防火墙
systemctl stop firewalld
# 关闭开机自启
systemctl disable firewalld
```

## 3\. 安装 NFS 并配置

在 Client 、Master 和 Slave 服务器上安装 NFS 服务

```bash
yum -y install nfs-utils rpcbind
```

**配置 NFS 共享目录** 在 Master 上执行

```bash
# 其中/data 为共享的目录，192.168.51.246 为 Client ip，如有多个私有云服务集群可用空格分隔
# 如 echo '/data 192.168.51.246(rw,sync,all_squash) 192.168.51.247(rw,sync,all_squash)' >> /etc/exports
 echo '/data 192.168.51.246(rw,sync,all_squash)' >> /etc/exports
# 开启服务
 systemctl start rpcbind && systemctl start nfs
# 设置开机自启
 systemctl enable rpcbind && systemctl enable nfs
# 出现：Created symlink from /etc/systemd/system/multi-user.target.wants/nfs-server.service to /usr/lib/systemd/system/nfs-server.service.即成功
```

在 Slave 上执行

```bash
# 其中/data 为共享的目录，192.168.51.246 为 Client ip，如有多个私有云服务集群可用空格分隔
# 如 echo '/data 192.168.51.246(rw,sync,all_squash) 192.168.51.247(rw,sync,all_squash)' >> /etc/exports
 echo '/data 192.168.51.246(rw,sync,all_squash)' >> /etc/exports
# 开启服务
 systemctl start rpcbind && systemctl start nfs
# 设置开机自启
 systemctl enable rpcbind && systemctl enable nfs
# # 出现：Created symlink from /etc/systemd/system/multi-user.target.wants/nfs-server.service to /usr/lib/systemd/system/nfs-server.service.即成功
```

**测试挂载是否成功** 在 Client 上执行挂载测试

```bash
# 测试 Master 
# 其中 ip 为 Master 的 ip，/data为 Master 共享的目录，/qiyuesuodata 为需要挂载至 Client 的目录
 mount -t nfs 192.168.50.8:/data /qiyuesuodata
# 检查 
df -Th 
# 出现  192.168.50.8:/data      nfs4       29G  7.6G   22G  27% /qiyuesuodata 即为成功
# 去除挂载
umount /qiyuesuodata

# 测试 Slave
# 其中 ip 为 Master 的 ip，/data为 Master 共享的目录，/qiyuesuodata 为需要挂载至 Client 的目录
 mount -t nfs 192.168.50.71:/data /qiyuesuodata
# 检查 
df -Th 
# 出现  192.168.50.71:/data      nfs4       29G  7.6G   22G  27% /qiyuesuodata 即为成功
# 去除挂载
umount /qiyuesuodata

```

## 4\. 配置文件同步

#### 在 Slave 进行同步 Master 数据

```bash
# 安装 rsync
yum -y install rsync.x86_64
# 修改 /etc/rsyncd.conf 如下,其中 hosts allow 填写 master ip
uid = nfsnobody
gid = nfsnobody
port = 873
pid file = /var/rsyncd.pid
log file = /var/log/rsyncd.log
use chroot = no
max connections = 200
read only = false
list = false
fake super = yes
ignore errors
[data]
path = /data
auth users = qiyuesuo
secrets file = /etc/rsync_salve.pass
hosts allow = 192.168.50.8

# 生成认证文件
echo 'qiyuesuo:qiyuesuo123' > /etc/rsync_salve.pass
chmod 600 /etc/rsync_salve.pass
# 修改 文件夹权限
chown -R nfsnobody:nfsnobody /data/
# 启动服务
 rsync --daemon --config=/etc/rsyncd.conf 
```

在 Master 上测试

```bash
yum -y install rsync.x86_64
chown -R nfsnobody:nfsnobody /data/
echo "qiyuesuo123" > /etc/rsync.pass
chmod 600 /etc/rsync.pass
#创建测试文件,测试推送
cd /data/
echo "This is test file" > file.txt
rsync -arv /data/ qiyuesuo@192.168.50.71::data --password-file=/etc/rsync.pass
#在 slave 上测试
ls /data 
# 出现 file.txt 即可
```

在 Master 上配置自动同步

```bash
 cd /usr/local/
 wget https://dl.qiyuesuo.com/private/nfs/sersync2.5.4_64bit_binary_stable_final.tar.gz
 tar xvf sersync2.5.4_64bit_binary_stable_final.tar.gz
 mv GNU-Linux-x86/ sersync
 cd sersync/
 # 修改配置文件
sed -ri 's#<delete start="true"/>#<delete start="false"/>#g' confxml.xml
sed -ri '24s#<localpath watch="/opt/tongbu">#<localpath watch="/data">#g' confxml.xml
sed -ri '25s#<remote ip="127.0.0.1" name="tongbu1"/>#<remote ip="192.168.50.71" name="data"/>#g' confxml.xml
sed -ri '30s#<commonParams params="-artuz"/>#<commonParams params="-az"/>#g' confxml.xml
sed -ri '31s#<auth start="false" users="root" passwordfile="/etc/rsync.pas"/>#<auth start="true" users="qiyuesuo" passwordfile="/etc/rsync.pass"/>#g' confxml.xml
sed -ri '33s#<timeout start="false" time="100"/><!-- timeout=100 -->#<timeout start="true" time="100"/><!-- timeout=100 -->#g' confxml.xml
#启动Sersync
/usr/local/sersync/sersync2 -dro /usr/local/sersync/confxml.xml
```

测试

```bash
# 在 master 中的/data 目录创建文件
touch test
# 查看 salve 中的 /data 是否有该文件
```

以上就做完了 salve 同步 master 的文件，但是当 master 宕机后恢复，master 无法同步 salve 文件，所以要配置 master 同步 salve 文件

#### 在 Master 进行同步 slave 数据

```bash
# 修改 /etc/rsyncd.conf 如下,其中 hosts allow 填写 slave ip
uid = nfsnobody
gid = nfsnobody
port = 873
pid file = /var/rsyncd.pid
log file = /var/log/rsyncd.log
use chroot = no
max connections = 200
read only = false
list = false
fake super = yes
ignore errors
[data]
path = /data
auth users = qiyuesuo
secrets file = /etc/rsync_master.pass
hosts allow = 192.168.50.71


# 生成认证文件
echo 'qiyuesuo:qiyuesuo123' > /etc/rsync_master.pass
chmod 600 /etc/rsync_master.pass
# 修改 文件夹权限
chown -R nfsnobody:nfsnobody /data/
# 启动服务
 rsync --daemon --config=/etc/rsyncd.conf 
```

在 Slave 上测试

```bash
chown -R nfsnobody:nfsnobody /data/
echo "qiyuesuo123" > /etc/rsync.pass
chmod 600 /etc/rsync.pass
#创建测试文件,测试推送
cd /data/
echo "This is test file" > file.2.txt
rsync -arv /data/ qiyuesuo@192.168.50.8::data --password-file=/etc/rsync.pass
#在 slave 上测试
ls /data 
# 出现 file.2.txt 即可
```

在 Slave 上配置自动同步

```bash
 cd /usr/local/
 wget https://dl.qiyuesuo.com/private/nfs/sersync2.5.4_64bit_binary_stable_final.tar.gz
 tar xvf sersync2.5.4_64bit_binary_stable_final.tar.gz
 mv GNU-Linux-x86/ sersync
 cd sersync/
 # 修改配置文件
sed -ri 's#<delete start="true"/>#<delete start="false"/>#g' confxml.xml
sed -ri '24s#<localpath watch="/opt/tongbu">#<localpath watch="/data">#g' confxml.xml
sed -ri '25s#<remote ip="127.0.0.1" name="tongbu1"/>#<remote ip="192.168.50.8" name="data"/>#g' confxml.xml
sed -ri '30s#<commonParams params="-artuz"/>#<commonParams params="-az"/>#g' confxml.xml
sed -ri '31s#<auth start="false" users="root" passwordfile="/etc/rsync.pas"/>#<auth start="true" users="qiyuesuo" passwordfile="/etc/rsync.pass"/>#g' confxml.xml
sed -ri '33s#<timeout start="false" time="100"/><!-- timeout=100 -->#<timeout start="true" time="100"/><!-- timeout=100 -->#g' confxml.xml
#启动Sersync
/usr/local/sersync/sersync2 -dro /usr/local/sersync/confxml.xml
```

至此我们已经做好了主从相互同步的操作 ##5. 安装 Keepalived 在 Master 上执行

```bash
yum -y install keepalived.x86_64
# 修改 /etc/keepalived/keepalived.conf
# 其中 enp0s3 为绑定网卡名称，可以使用 ip addr 命令查看
# 其中 192.168.50.143  为虚拟 ip ，注意不要和其它 ip 冲突
! Configuration File for keepalived

global_defs {
   router_id NFS-Master
}

vrrp_instance VI_1 {
    state MASTER
    interface enp0s3
    virtual_router_id 51
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass qiyuesuo123
    }
    virtual_ipaddress {
        192.168.50.143  
    }
}
# 启动服务
systemctl start  keepalived.service && systemctl enable keepalived.service
```

在 Slave 上执行

```bash
yum -y install keepalived.x86_64
# 修改 /etc/keepalived/keepalived.conf
# 其中 enp0s3 为绑定网卡名称，可以使用 ip addr 命令查看
# 其中 192.168.50.143  为虚拟 ip ，注意不要和其它 ip 冲突
! Configuration File for keepalived

global_defs {
   router_id NFS-Slave
}

vrrp_instance VI_1 {
    state MASTER
    interface enp0s3
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass qiyuesuo123
    }
    virtual_ipaddress {
        192.168.50.143  
    }
}
# 启动服务
systemctl start  keepalived.service && systemctl enable keepalived.service
```

**查看虚拟IP是否存在** 在 Master 上执行：

```bash
ip a  grep  192.168.50.143
# 出现
# inet 192.168.50.143/32 scope global enp0s3
# 即成功
```

**VIP挂载测试** 在 Client 上通过 vip 挂载测试

```bash
mount -t nfs 192.168.50.143:/data /qiyuesuodata
# 如/qiyuesuodata目录中有共享目录中文件则说明挂载成功
umount /qiyuesuodata/
```

模拟机器Down机,测试虚拟IP地址是否会漂移

```bash
# 在 Master 上关闭 keepalived
systemctl stop keepalived.service
# 执行ip a  grep  192.168.50.143会无输出则关闭成功
# 在 Slave 上查看
ip a  grep  192.168.50.143
# 出现
# inet 192.168.50.143/32 scope global enp0s3
# 即成功
```

则说明 ip 漂移成功

## 设置 keepalived 脚本

因为 ip 的漂移是根据 keepalived 的存活来判断的，所以在 nfs 宕机之后需要手动停止 keepalived 服务来进行ip 的切换，这里在 Master 上编写一个定时任务来检测 nfs 服务是否宕机

```bash
cd /usr/local/sbin
# 生成文件check_nfs.sh
#!/bin/sh
# 每秒执行一次
step=1 #间隔的秒数，不能大于60 
for (( i = 0; i < 60; i=(i+step) )); do 
  ###检查nfs可用性：进程和是否能够挂载
  /sbin/service nfs status &>/dev/null
  if [ $? -ne 0 ];then
    ###如果服务状态不正常，先尝试重启服务
    /sbin/service nfs restart
    /sbin/service nfs status &>/dev/null
    if [ $? -ne 0 ];then
       # 如服务仍不正常，停止 keepalived
       systemctl stop keepalived.service
    fi
  fi
  sleep $step 
done 

```

加入定时任务

```bash
chmod 777 /usr/local/sbin/check_nfs.sh
crontab -e
# 输入定时任务
* * * * *  /usr/local/sbin/check_nfs.sh &> /dev/null
```

在 Client 添加定时任务，当 Master 宕机时进行重新挂载

```bash
cd /usr/local/sbin
# 生成文件check_mount.sh

#!/bin/sh
# 每秒执行一次
step=1 #间隔的秒数，不能大于60 
for (( i = 0; i < 60; i=(i+step) )); do 
  mount=`df -Thgrep qiyuesuodata`
  if [ $mount = "" ];then
     umount /qiyuesuodata
     mount mount -t nfs 192.168.50.143:/data /qiyuesuodata
  fi
  sleep $step 
done 
```

加入定时任务

```bash
chmod 777 /usr/local/sbin/check_mount.sh
crontab -e
# 输入定时任务
* * * * *  /usr/local/sbin/check_nfs.sh &> /dev/null
```