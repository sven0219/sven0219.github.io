---
title: 华为云 arm 架构安装 mysql 5.7
tags:
  - MySQL
  - 软件安装
  - 运维
id: '11837'
categories:
  - - skills
date: 2020-01-06 15:43:28
---

# 华为云 arm centos7 安装部署 mysql

# 1\. 配置环境

## 1.1 防火墙并取消开机自启动

步骤 1 停止防火墙

```bash
systemctl stop firewalld.service
```

步骤 2 关闭开机自启

```bash
systemctl disable firewalld.service
```

# 1.2 修改selinux 为 disable

```bash
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```

# 1.3 创建组和用户

服务器环境下，为了系统安全，通常会为进程分配单独的用户，以实现权限隔离。本章节创建的组和用户都是操作系统层面的，不是数据库层面的。 创建mysql用户(组)

```bash
groupadd mysql
useradd -g mysql mysql
```

## 1.4 创建数据目录

```bash
mkdir /data
mkdir -p /data/mysql-5.7.27-1/mysql
cd /data/mysql-5.7.27-1/mysql
mkdir data tmp run log
chown -R mysql:mysql /data/mysql-5.7.27-1/mysql
```

# 2.安装、运行、卸载

## 2.1 安装

### 2.1.1 镜像站 RPM 安装

镜像站中的mysql-5.7.27-1.aarch64.rpm安装包根据《MySQL 5.7.27 编译指南（CentOS 7.6）》打包而成，然后将其上传到镜像站。此安装方式需要连接外网。如果没有外网，则下载RPM包，上传到服务器任意路径，并在该路径下执行命令`rpm -ivh mysql-5.7.27-1.aarch64.rpm`安装即可。本小节相关连接如下： RPM下载链接为： **[https://mirrors.huaweicloud.com/kunpeng/yum/el/7/aarch64/Packages/mysql-5.7.27-1.aarch64.rpm](x-wc://file=zh-cn_topic_0200581783.html)** 编译指南链接为： https://bbs.huaweicloud.com/forum/thread-26825-1-1.html

### 2.1.2 编译安装

参考《MySQL 5.7.27 编译指南（CentOS 7.6）.doc》，文档链接为：[**https://bbs.huaweicloud.com/forum/thread-26825-1-1.html**](https://bbs.huaweicloud.com/forum/thread-26825-1-1.html)

## 2.2 运行

如果采用镜像站rpm安装方式安装，则需要额外做以下操作步骤避免初始化数据库失败：

1.  从连接[https://bbs.huaweicloud.com/forum/thread-29895-1-1.html](x-wc://file=zh-cn_topic_0200581780.html)中下载压缩包rpm-bug.zip并上传到服务器/home目录下
    
2.  解压压缩包
    

```bash
cd /home/
unzip rpm-bug.zip
```

3.  进入解压后的文件目录并将其中的文件放入到指定目录

```bash
cd rpm-bug
cp libatomic.so.1 /usr/lib64/
cp libstdc++.so.6.0.24 /lib64/  
rm /lib64/libstdc++.so.6
ln -s /lib64/libstdc++.so.6.0.24  /lib64/libstdc++.so.6
cp libaio.so.1.0.1 /usr/lib64/libaio.so.1
```

### 2.2.1 修改配置文件

步骤 1 编辑my.cnf配置文件，其中文件路径（包括软件安装路径basedir、数据路径datadir等）根据实际情况修改。

```bash
rm -f /etc/my.cnf
echo -e "[mysqld_safe]\nlog-error=/data/mysql-5.7.27-1/mysql/log/mysql.log\npid-file=/data/mysql-5.7.27-1/mysql/run/mysqld.pid\n[mysqldump]\nquick\n[mysql]\nno-auto-rehash\n[client]\ndefault-character-set=utf8\n[mysqld]\nbasedir=/home/mysql-5.7.27-1/mysql\nsocket=/data/mysql-5.7.27-1/mysql/run/mysql.sock\ntmpdir=/data/mysql-5.7.27-1/mysql/tmp\ndatadir=/data/mysql-5.7.27-1/mysql/data\nsocket=/data/mysql-5.7.27-1/mysql/run/mysql.sock\nport=3306\nuser=root\n" > /etc/my.cnf

```

步骤 2 确保 my.cnf 配置文件修改正确

```bash
cat /etc/my.cnf
```

```bash
[mysqld_safe]
log-error=/data/mysql-5.7.27-1/mysql/log/mysql.log
pid-file=/data/mysql-5.7.27-1/mysql/run/mysqld.pid
[mysqldump]
quick
[mysql]
no-auto-rehash
[client]
default-character-set=utf8
[mysqld]
basedir=/home/mysql-5.7.27-1/mysql
socket=/data/mysql-5.7.27-1/mysql/run/mysql.sock
tmpdir=/data/mysql-5.7.27-1/mysql/tmp
datadir=/data/mysql-5.7.27-1/mysql/data
socket=/data/mysql-5.7.27-1/mysql/run/mysql.sock
port=3306
user=root
```

### 2.2.2 配置环境变量

**步骤 1** 安装完成后，将MySQL二进制文件路径到PATH。 其中PATH中的“/home/mysql-5.7/mysql/bin”路径，为MySQL软件安装目录下的bin文件的绝对路径。

```bash
echo  export  PATH=$PATH:/home/mysql-5.7.27-1/mysql/bin  >> /etc/profile
```

步骤 2 使环境变量配置生效

```bash
source /etc/profile     
```

### 2.2.3 初始化数据库

步骤 1 初始化数据库

```bash
/home/mysql-5.7.27-1/mysql/bin/mysqld --defaults-file=/etc/my.cnf --initialize  --basedir=/home/mysql-5.7.27-1/mysql --datadir=/data/mysql-5.7.27-1/mysql/data --user=mysql
```

回显信息倒数第一行会显示初始密码，需要保存，后面登陆会使用到

### 2.2.4 启动数据库

步骤 1 启动 MySQL，执行下面命令回车

```bash
/home/mysql-5.7.27-1/mysql/bin/mysqld_safe --basedir=/home/mysql-5.7.27-1/mysql/ --datadir=/data/mysql-5.7.27-1/mysql/data --user=root &
```

### 2.2.5 登陆数据库

```bash
/home/mysql-5.7.27-1/mysql/bin/mysql -uroot -p -S /data/mysql-5.7.27-1/mysql/run/mysql.sock
```

此处使用初始密码登陆

### 2.2.6 配置数据库账号和密码

步骤 1 修改本地root用户登录密码。

```mysql
alter user 'root'@'localhost' identified by "123456";
```

步骤 2 创建全域root用户（允许root从其他服务器访问）。

```mysql
create user 'root'@'%' identified by '123456';
```

步骤 3 进行授权。

```mysql
grant all privileges on *.* to 'root'@'%';
flush privileges;
```

## 2.3 卸载

编译安装只是生成对应的文件，不涉及卸载。 步骤 1 关闭数据库进程 mysql\_safe 和 mysqld

```bash
ps -ef  grep mysql
kill -9 进程ID
```

步骤 2 卸载数据库 RPM 包

```bash
rpm -e mysql-5.7.27-1.aarch64
```

步骤 3 删除数据库的安装目录和数据目录

```bash
rm -rf /home/mysql-5.7.27-1
rm -rf /data/mysql-5.7.27-1
```