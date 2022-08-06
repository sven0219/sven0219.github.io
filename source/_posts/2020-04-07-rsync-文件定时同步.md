---
title: 文件定时同步(Linux+windows)
tags:
  - Linux
  - rsync
  - windows
  - 运维
id: '12059'
categories:
  - - skills
date: 2020-04-07 20:02:28
---

# Linux rsync 实现文件定时同步

本文档适用于私有云文件存储方式为本地存储、磁盘挂载 思路：服务器A和B上都安装rsync，其中B服务器上是以服务器模式运行rsync，而A上则以客户端方式运行rsync。这样在web服务器B上运行rsync守护进程，在A上定时运行客户程序来同步服务器B上需要同步的内容。 准备：
<!--more-->
机器

ip

目录

A（源机器）

192.168.1.146

文件目录：/data

B（备份机器）

192.168.1.147

备份目录：/backup

### 1\. 安装 rsync（两台机器都要执行）

```bash
yum install rsync
```

### 2\. 配置

rsync的主要有以下三个配置文件rsyncd.conf(主配置文件)、rsyncd.secrets(密码文件)、rsyncd.motd(rysnc服务器信息)

#### rsyncd 服务端配置（A 机器配置）

/etc/rsyncd/rsyncd.conf

```text
#/etc/rsyncd/rsyncd.conf 所属用户ID，一般为root
uid =root
#/etc/rsyncd/rsyncd.conf 所属权限组
gid =root


use chroot = no   #在传输文件的之前，是否转到用户根目录。
max connections = 4   #最大连接数

#服务进程pid保存文件
pid file = /var/run/rsyncd.pid 
#锁文件路径
lock file = /var/run/rsyncd.lock 

log file = /var/log/rsyncd.log  #日至文件路径

log format = %t %a %m %f %b

#要备份的模块名，该名称客户端进行同步时需要调用   
[qysdata] 
path = /data   #要备份的目录
ignore errors    #可以忽略一些无关的IO错误

read only = true  # // 只读
list = false  #//不允许列文件
hosts allow = 192.168.1.147 # 允许访问的 ip
hosts deny = 0.0.0.0/32 # 禁止访问的 ip
#//认证的用户名，如果没有这行则表明是匿名，此用户与系统无关
auth users = qys   
#//密码和用户名对比表，密码文件自己生成
secrets file = /etc/rsyncd/rsyncd.secrets  

comment = qiyuesuo data backup   #这个模块的注释信息
```

/etc/rsyncd/rsyncd.secrets 配置rsync密码（在上边的配置文件中已经写好路径,名字随便写，只要和上边配置文件里的一致即可），格式(一行一个用户)

```
qys:admin#1234
```

赋予权限

```
chmod 600 /etc/rsyncd/rsyncd.secrets
chmod 600 /etc/rsyncd/rsyncd.conf
```

#### 客户端配置（B 机器配置）

配置密码文件（这个密码是rsync请求服务端需要的认证密码） /etc/rsyncd/client.pass

```
admin#1234
```

赋予权限

```
chmod 600 /etc/rsyncd/client.pass
```

#### 启动服务（A 机器启动）

```bash
rsync --daemon --config=/etc/rsyncd/rsyncd.conf
```

设置开机自启

```bash
echo "rsync --daemon --config=/etc/rsyncd/rsyncd.conf" >>  /etc/rc.local 
```

#### 客户端拉取文件测试（B 服务器执行）

```
rsync -vzrtopg --progress --delete qys@192.168.1.146::qysdata /backup --password-file=/etc/rsyncd/client.pass
```

如B服务器上 backup目录有 A 上的文件则同步成功

### 3\. 配置定时同步

#### 定时脚本

/usr/local/sbin/backup.sh 将上述命令写入脚本

```bash
#!/bin/bash
rsync -vzrtopg --progress --delete qys@192.168.1.146::qysdata /backup --password-file=/etc/rsyncd/client.pass
```

添可执行权限

```
chmod +x /usr/local/sbin/backup.sh
```

#### 设置定时任务

```bash
crontab -e
```

输入

```
*/1 * * * * /usr/local/sbin/backup.sh
```

以上完成定时每分钟同步一次文件

# Windows SyncToy 定时同步文件

思路：使用微软官方的SyncToy将源机器文件同步至备份机器的共享文件夹 准备

机器

ip

目录

A（源机器）

192.168.1.131

文件目录：D:\\data

B（备份机器）

192.168.1.130

备份目录：D:\\backup

### 1\. 设置共享文件夹（B 机器上执行）

创建共享文件夹 ![image.png](https://i.loli.net/2020/04/08/Cr3JoVi17mn6Gbp.png) 在 A 机器上测试共享文件夹是否创建成功 ![image.png](https://i.loli.net/2020/04/08/mKeilNRIaJnSuM2.png) 出现以下内容说明 创建成功 ![image.png](https://i.loli.net/2020/04/08/UhcvIVFsAxE2BwC.png)

### 2\. 安装 RsyncToy（在 A 机器上安装）

下载地址：https://download.microsoft.com/download/6/c/4/6c406239-a648-4e01-833e-2c452deed3b6/SyncToySetupPackage\_v21\_x64.exe 双击安装即可，安装好后打开界面如下 ![image.png](https://i.loli.net/2020/04/08/9RLraqby2QGZsWK.png)

### 3\. 配置

![image.png](https://i.loli.net/2020/04/08/1A6vDs4NU5BX82q.png) 点击下一步后会有三种模式

*   Synchronize:新文件和更改过的文件在左右目录中将互相复制，同时，若两个目录中有同样的文件，在其中一个目录有重命名或者删除的，在另一个目录中也将执行同样操作。
    
*   Echo:以左侧目录为准，左目录中的新文件和更改过的文件将复制到右目录中；同时，若两个目录中有同样的文件，在左目录中有重命名或者删除的，在右目录中也将执行同样操作。
    
*   Contribute：和Echo的操作类似，是一种更加安全的方式。不执行删除操作，相当于右侧文件夹是左侧的增量备份
    

这里我们选择第三种模式 ![image.png](https://i.loli.net/2020/04/08/iGyYEMPr3HNJCdl.png) 命名 ![image.png](https://i.loli.net/2020/04/08/snZ1JMjPOIdGwYo.png) 测试是否成功，点击 run 运行 ![image.png](https://i.loli.net/2020/04/08/G7hqEIgrKmzYVvn.png) 在 B 机器上查看，文件同步过来即成功 ![image.png](https://i.loli.net/2020/04/08/IKubUpYxlfmnj3k.png)

### 4\. 配置定时计划

在控制面板中找到定时计划程序 ![image.png](https://i.loli.net/2020/04/08/FUjWXhQRglSmKB3.png) 创建任务 ![image.png](https://i.loli.net/2020/04/08/8uYTgHcwMjylq1X.png) 配置基本信息 ![image.png](https://i.loli.net/2020/04/08/YOhM3wnRxFo8UB7.png) 配置触发器 ![image.png](https://i.loli.net/2020/04/08/PuIL9gwqcUmZYrd.png) 配置操作 ![image.png](https://i.loli.net/2020/04/08/hE2nrtdO71w9VMA.png) 输入密码保存即可。 验证是否成功，在 A 机器data 目录新建一个文件等待五分钟左右，看文件是否同步成功。