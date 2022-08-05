---
title: Linux ssh加固方案
tags:
  - Linux
  - 运维
id: '11894'
categories:
  - - skills
date: 2020-02-04 17:47:53
---

## 1\. 设置密码有效期和长度

默认配置

```bash
[root@swq_test ~]# cat /etc/login.defs grep PASS_ grep -v '#'
PASS_MAX_DAYS    99999
PASS_MIN_DAYS    0
PASS_MIN_LEN    5
PASS_WARN_AGE    7
```

加固方案

```
# 1.备份配置文件：

# cp -a /etc/login.defs /etc/login.defs.default

# 2.编辑配置文件并将相关参数改成如下

# vi /etc/login.defs
PASS_MAX_DAYS 90
PASS_MIN_DAYS 6
PASS_MIN_LEN 8
PASS_WARN_AGE 30
```

参数说明： PASS\_MAX\_DAYS 密码有效期 PASS\_MIN\_DAYS 修改密码的最短期限 PASS\_MIN\_LEN 密码最短长度 PASS\_WARN\_AGE 密码过期提醒

## 2\. 密码复杂度

默认配置

```
[root@swq_test ~]# cat /etc/pam.d/system-auth  grep  "pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type="
password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
```

加固方案

```
1.备份配置文件：
# cp -a /etc/pam.d/system-auth /etc/pam.d/system-auth.default
2.编辑配置文件
# vi /etc/pam.d/system-auth
将password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
 注释并在其下面新增1行 password requisite pam_cracklib.so try_first_pass minlen=8 difok=5 dcredit=-1 lcredit=-1 ocredit=-1 retry=1 type= 
3.保存配置文件
```

备注: try\_first\_pass而当pam\_unix验证模块与password验证类型一起使用时，该选项主要用来防止用户新设定的密码与以前的旧密码相同。 minlen=8：最小长度8位 difok=5:新、旧密码最少5个字符不同 dcredit=-1：最少1个数字 lcredit=-1：最少1个小写字符，（ucredit=-1:最少1个大写字符) ocredit=-1：最少1个特殊字符 retry=1:1次错误后返回错误信息 type=xxx：此选项用来修改缺省的密码提示文本

## 3\. 设置会话超时时间

默认配置 无 加固方案

```
1.备份配置文件：
# cp -a /etc/profile /etc/profile.default
2.编辑配置文件：
vi /etc/profile
在文件的末尾添加参数 
export TMOUT=300
3.保存配置文件
```

备注： 五分钟不操作会中断会话

## 4\. 设置登陆失败锁定

默认配置 无 加固方案

```
1.备份配置文件
2.编辑配置文件：
# vi /etc/pam.d/system-auth
在# User changes will be destroyed the next time authconfig is run.行的下面，添加
auth       required     pam_tally2.so deny=5 unlock_time=1800 even_deny_root root_unlock_time=1800
3.保存配置文件
```

备注: 通过终端登录，5次登录失败后锁定账号30分钟，锁定期间此账号无法再次登录。

## 5.禁止 Root 账号远程登陆

默认配置:

```
# cat /etc/ssh/sshd_config grep PermitRootLogin
#PermitRootLogin yes
```

加固方案

```
1.备份配置文件
# cp -a /etc/ssh/sshd_config /etc/ssh/sshd_config.default
2.编辑配置文件
vi /etc/ssh/sshd_config
将配置参数#PermitRootLogin yes改成PermitRootLogin no
3.保存配置文件
4.重启ssh服务
# /etc/init.d/sshd restart
```

备注： 若需要使用 root 用户，可通过`su`切换到 root 用户

## 6\. 修改 ssh 默认端口

默认配置

```
# cat /etc/ssh/sshd_configgrep Port
Port 22
```

加固方案

```
1.备份配置文件
# cp -a /etc/ssh/sshd_config /etc/ssh/sshd_config.default
2.编辑配置文件
vi /etc/ssh/sshd_config
将配置参数Port 22改成Port 60022
3.保存配置文件
4.重启ssh服务
# /etc/init.d/sshd restart
```

备注： 修改后 ssh 登陆需要指定端口 60022