---
title: 监控工具 zabbix 实践
tags:
  - zabbix
  - 监控
  - 运维
id: '12162'
categories:
  - - skills
date: 2020-08-18 13:59:03
---

# 介绍

引用百度百科

> zabbix 是一个基于WEB界面的提供分布式系统监视以及网络监视功能的企业级的开源解决方案。 zabbix能监视各种网络参数，保证服务器系统的安全运营；并提供灵活的通知机制以让系统管理员快速定位/解决存在的各种问题。 zabbix由2部分构成，zabbix server与可选组件zabbix agent。 zabbix server可以通过SNMP，zabbix agent，ping，端口监视等方法提供对远程服务器/网络状态的监视，数据收集等功能，它可以运行在Linux，Solaris，HP-UX，AIX，Free BSD，Open BSD，OS X等平台上。

# 安装

这里我用的机器是阿里云采购的 ECS，系统版本如下 [![](https://i.loli.net/2020/08/18/NGFpVQa3q8Uz7T6.jpg)](https://i.loli.net/2020/08/18/NGFpVQa3q8Uz7T6.jpg) 使用的 zabbix 为目前最新版本，zabbix5.0，使用二进制包安装，数据库使用的位 mysql5.7。 [![](https://i.loli.net/2020/08/18/c6bOi14uBNZkeaF.jpg)](https://i.loli.net/2020/08/18/c6bOi14uBNZkeaF.jpg)

### 安装repository

```bash
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
yum clean all
```

### 安装 zabbix server 和 agent

```bash
yum install zabbix-server-mysql zabbix-agent
```

由于网络问题， yum 下载慢上述命令可能会报错 [![](https://i.loli.net/2020/08/18/iFLs5SjabwREN29.jpg)](https://i.loli.net/2020/08/18/iFLs5SjabwREN29.jpg) 在服务器外部将 rpm 包下载好后上传至服务器进行安装 [![](https://i.loli.net/2020/08/18/E2QJBzwjdR3pTaH.jpg)](https://i.loli.net/2020/08/18/E2QJBzwjdR3pTaH.jpg)

```bash
# 通过本地二进制包安装
yum install ./zabbix-* -y
```

### 安装 zabbix frontend

```bash
# 开启Red Hat Software Collections
yum install centos-release-scl
# 编辑 /etc/yum.repos.d/zabbix.repo
[zabbix-frontend]
...
enabled=1
...
# 安装 frontend package
yum install zabbix-web-mysql-scl zabbix-nginx-conf-scl
```

这个步骤会有`zabbix-web-5.0.2-1.el7.noarch.rpm`下载失败的情况出现，这里也是服务器外部下载好上传至服务器安装。

> 无法下载的 rpm 包链接: https://pan.baidu.com/s/1iZczDNB-4Sz52bUbPMowNw 密码: ovr8

# 创建初始化数据库

这里安装 mysql 服务就省略了，按照官网文档或者搜索引擎上有很多教程。

```shell
#登陆数据库
mysql -uroot -p
# 创建数据库
create database zabbix character set utf8 collate utf8_bin;
# 创建用户
 create user zabbix@localhost identified by 'password';
# 赋权
grant all privileges on zabbix.* to zabbix@localhost;
# 退出 mysql 命令行
quit;
```

数据库创建完成后初始化数据库

```bash
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz  mysql -uzabbix -p zabbix
```

等待 1 分钟左右即可，查询数据库中部分表如下： [![](https://i.loli.net/2020/08/18/bOlmG9rKe6xv24a.jpg)](https://i.loli.net/2020/08/18/bOlmG9rKe6xv24a.jpg)

# 配置

### 为 zabbix server 配置数据库

编辑/etc/zabbix/zabbix\_server.conf

```conf
# password 修改为自己设置的密码
DBPassword=password
```

[![](https://i.loli.net/2020/08/18/lI5iqQu28KVeDnR.jpg)](https://i.loli.net/2020/08/18/lI5iqQu28KVeDnR.jpg) 如用户名、数据库名、数据库地址不一样也需要修改配置

### 为 zabbix frontend 配置 PHP

编辑`/etc/opt/rh/rh-nginx116/nginx/conf.d/zabbix.conf` 将图示这两行注释去掉，server\_name 改成自己的域名或者服务器 ip [![](https://i.loli.net/2020/08/18/liTdkcI4JnsOePZ.jpg)](https://i.loli.net/2020/08/18/liTdkcI4JnsOePZ.jpg) 编辑`/etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf`,在`listen.acl_users`行添加 nginx，并修改正确时区 [![](https://i.loli.net/2020/08/18/BDdjoqPp28y6swZ.jpg)](https://i.loli.net/2020/08/18/BDdjoqPp28y6swZ.jpg)

# 启动 server 和 agent

```bash
# 启动
systemctl restart zabbix-server zabbix-agent rh-nginx116-nginx rh-php72-php-fpm
# 设置开机自启
systemctl enable zabbix-server zabbix-agent rh-nginx116-nginx rh-php72-php-fpm
```

# 访问服务

通过 ip 或者域名（前面配置的 server\_name）访问 **初始化页面** [![](https://i.loli.net/2020/08/18/jgW6yXkO4r5xnGH.jpg)](https://i.loli.net/2020/08/18/jgW6yXkO4r5xnGH.jpg) **环境监测** [![](https://i.loli.net/2020/08/18/DItweYT2dk6aFKR.jpg)](https://i.loli.net/2020/08/18/DItweYT2dk6aFKR.jpg) **数据库配置** [![](https://i.loli.net/2020/08/18/nGbVye1HxdTw43j.jpg)](https://i.loli.net/2020/08/18/nGbVye1HxdTw43j.jpg) **设置服务名称** [![](https://i.loli.net/2020/08/18/KDoYn4ieyhTpBZ9.jpg)](https://i.loli.net/2020/08/18/KDoYn4ieyhTpBZ9.jpg) **检查配置** [![](https://i.loli.net/2020/08/18/4cZsRHYfUzeNgOT.jpg)](https://i.loli.net/2020/08/18/4cZsRHYfUzeNgOT.jpg) **安装完成** [![](https://i.loli.net/2020/08/18/nPtfirwkSOLZ8vU.jpg)](https://i.loli.net/2020/08/18/nPtfirwkSOLZ8vU.jpg) **首次登陆** 初始账号 `Admin`，初始密码`zabbix`（登陆之后第一时间修改密码） [![](https://i.loli.net/2020/08/18/2qj7vZNUGRrhJud.jpg)](https://i.loli.net/2020/08/18/2qj7vZNUGRrhJud.jpg) **修改语言（非必须）** [![](https://i.loli.net/2020/08/18/BLJubf4ly6wDdpx.jpg)](https://i.loli.net/2020/08/18/BLJubf4ly6wDdpx.jpg) 至此，安装已完成，下面配置一个简单的监控。

# 配置监控

刚进入会有一个主机了，这个是 zabbix 服务器所在的主机，现在准备新建一个主机。 [![](https://i.loli.net/2020/08/18/QXokuI48FRAMbZt.jpg)](https://i.loli.net/2020/08/18/QXokuI48FRAMbZt.jpg)

### 在新主机上安装 agent

下载`zabbix-agent-5.0.2-1.el7.x86_64.rpm`,上面的百度网盘的链接，上传至服务器 安装 agent

```bash
yum install zabbix-agent-5.0.2-1.el7.x86_64.rpm
```

### 配置 agent

vi /etc/zabbix/zabbix\_agentd.conf

```bash
…
Server=172.16.177.24 #zabbix-server端IP
…
ServerActive=Server=172.16.177.24  #zabbix-server端IP
…
Hostname= swq_aliyun #主动模式IP要一致
```

### web端配置

添加主机 [![](https://i.loli.net/2020/08/18/nM3hbDLywFpoG2j.jpg)](https://i.loli.net/2020/08/18/nM3hbDLywFpoG2j.jpg) 添加配置模板 [![](https://i.loli.net/2020/08/18/qHbpKRF3y6IQovW.jpg)](https://i.loli.net/2020/08/18/qHbpKRF3y6IQovW.jpg) [![](https://i.loli.net/2020/08/18/HQsyhobVgJUTrOW.jpg)](https://i.loli.net/2020/08/18/HQsyhobVgJUTrOW.jpg) 添加成功后即可看到数据 [![](https://i.loli.net/2020/08/18/FKzCinHRm75I1qs.jpg)](https://i.loli.net/2020/08/18/FKzCinHRm75I1qs.jpg) [![](https://i.loli.net/2020/08/18/sBThqApaG3x12DI.jpg)](https://i.loli.net/2020/08/18/sBThqApaG3x12DI.jpg) 表格字体乱码可在服务器上安装黑体即可解决。 至此，zabbix 安装及简单配置已完成，zabbix 功能很强大，更多的配置需要自己去摸索。