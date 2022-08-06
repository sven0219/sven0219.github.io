---
title: win10 wsl linux 子系统ssh服务自启
tags:
  - windows
  - 运维
id: '11796'
categories:
  - - skills
date: 2019-12-13 16:27:27
---

# win10 wsl linux 子系统ssh服务自启

创建并编辑`/etc/init.wsl`，写入如下内容

```
#!/bin/bash
/ect/init.d/ssh $1
```
<!--more-->
添加可执行权限

```
sudo chmod +x /etc/init.wsl
```

编辑 sudoers ,避免输入密码

```
sudo visudo
```

添加一行

```
%sudo ALL=NOPASSWD: /etc/init.wsl
```

创建一个 start.vbs 脚本，内容为

```
Set ws = WScript.CreateObject("WScript.Shell")
ws.run "ubuntu run sudo /etc/init.wsl start", vbhide
```

win10 的开始-运行里面输入`shell:startup`打开启动文件夹，将该文件放入即可