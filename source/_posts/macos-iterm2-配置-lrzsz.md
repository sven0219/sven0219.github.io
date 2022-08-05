---
title: MacOS iTerm2 配置 lrzsz
tags:
  - Mac
  - 记录
id: '12274'
categories:
  - - skills
date: 2020-12-17 19:21:12
---

# 写在前面

公司换过好几次电脑，每次配置 iTerm2 都要查教程配置，自己记录下以备不时之需。

# 环境

Mac OS 11.1.0 ITerm2 3.4.3

# 安装 ITerm2

[官网](https://iterm2.com/ "官网")下载安装就可以了，不赘述了.

# 安装 rz sz

```bash
# 需要先安装 homebrew
brew install lrzsz
```

# 配置 rz sz

```bash
vim iterm2-recv-zmodem.sh
```

输入以下内容

```shell
#!/bin/bash
# Author: Matt Mastracci (matthew@mastracci.com)
# AppleScript from http://stackoverflow.com/questions/4309087/cancel-button-on-osascript-in-a-bash-script
# licensed under cc-wiki with attribution required
# Remainder of script public domain

osascript -e 'tell application "iTerm2" to version' > /dev/null 2>&1 && NAME=iTerm2  NAME=iTerm
if [[ $NAME = "iTerm" ]]; then
    FILE=`osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose folder with prompt "Choose a folder to place received files in"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
else
    FILE=`osascript -e 'tell application "iTerm2" to activate' -e 'tell application "iTerm2" to set thefile to choose folder with prompt "Choose a folder to place received files in"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
fi

if [[ $FILE = "" ]]; then
    echo Cancelled.
    # Send ZModem cancel
    echo -e \\x18\\x18\\x18\\x18\\x18
    sleep 1
    echo
    echo \# Cancelled transfer
else
    cd "$FILE"
    /usr/local/bin/rz -E -e -b
    sleep 1
    echo
    echo
    echo \# Sent \-\> $FILE
fi
```

保存退出，配置 send

```bash
vim iterm2-send-zmodem.sh
```

输入以下内容

```shell
#!/bin/bash
# Author: Matt Mastracci (matthew@mastracci.com)
# AppleScript from http://stackoverflow.com/questions/4309087/cancel-button-on-osascript-in-a-bash-script
# licensed under cc-wiki with attribution required
# Remainder of script public domain

osascript -e 'tell application "iTerm2" to version' > /dev/null 2>&1 && NAME=iTerm2  NAME=iTerm
if [[ $NAME = "iTerm" ]]; then
    FILE=`osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose file with prompt "Choose a file to send"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
else
    FILE=`osascript -e 'tell application "iTerm2" to activate' -e 'tell application "iTerm2" to set thefile to choose file with prompt "Choose a file to send"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`
fi
if [[ $FILE = "" ]]; then
    echo Cancelled.
    # Send ZModem cancel
    echo -e \\x18\\x18\\x18\\x18\\x18
    sleep 1
    echo
    echo \# Cancelled transfer
else
    /usr/local/bin/sz "$FILE" -e -b
    sleep 1
    echo
    echo \# Received $FILE
fi
```

保存退出，并修改权限

```bash
chmof +x iterm2-*
```

# iTerm2 配置添加rz sz 功能

点击 iTerm2 的设置界面 Perference-> Profiles -> Default -> Advanced -> Triggers 的 Edit 按钮 [![](https://i.loli.net/2020/12/17/XjG1nqtLIpeK9Yy.jpg)](https://i.loli.net/2020/12/17/XjG1nqtLIpeK9Yy.jpg) 添加两个 Triggers 内容如下：

```shell
Regular expression: rz waiting to receive.\*\*B0100
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-send-zmodem.sh


Regular expression: \*\*B00000000000000
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-recv-zmodem.sh

```

添加完成后如下 [![](https://i.loli.net/2020/12/17/r7kiqIyaVbum3x9.jpg)](https://i.loli.net/2020/12/17/r7kiqIyaVbum3x9.jpg) 至此就完成了配置，可以在服务器上使用 rz sz 命令上传下载文件了。 备注： 服务器上也要安装 lrzsz

```bash
# centos 
yum install lrzsz
#ubuntu
apt-get install lrzsz
```