---
title: Nginx 配置正向代理
tags:
  - nginx
  - 正向代理
  - 运维
id: '12012'
categories:
  - - skills
date: 2020-03-24 12:07:30
---

# Nginx 配置正向代理

之前一直使用 squid 进行正向代理配置，今天尝试采用 nginx 进行正向代理配置。 nginx本身是不支持https协议请求转发，为了让nginx能达到这一效果需要借助第三方模块[ngx\_http\_proxy\_connect\_module](https://github.com/chobits/ngx_http_proxy_connect_module.git "ngx_http_proxy_connect_module")。

# 安装基础依赖

```bash
yum -y install pcre-devel zlib-devel gcc gcc+c++ make openssl-devel pcre-devel  zlib-devel patch   git 
```

# 下载 nginx 安装包

这里下载最新的稳定版本 [1.16.1](http://nginx.org/download/nginx-1.16.1.tar.gz "1.16.1")
<!--more-->
```bash
wget http://nginx.org/download/nginx-1.16.1.tar.gz
```

# 下载 ngx\_http\_proxy\_connect\_module

```bash
git clone https://github.com/chobits/ngx_http_proxy_connect_module.git
```

# 配置

我这里下载好的包都在/root目录下

#### 编译安装

```
cd /root
tar -zxvf nginx-1.16.1.tar.gz 
cd nginx-1.16.1
patch -p1 <  ../ngx_http_proxy_connect_module/patch/proxy_connect_rewrite_101504.patch 
./configure --add-module=../ngx_http_proxy_connect_module
make && make install
```

默认是安装在`/usr/local/nginx`目录下

#### 配置代理

```
 vi /usr/local/nginx/conf/nginx.conf
```

http 段中添加如下内容

```nginx
server {
       resolver 8.8.8.8;   #dns解析地址
       listen 8080;          #代理监听端口
       proxy_connect;
       proxy_connect_allow          80 443 563;
       location / {
             proxy_pass $scheme://$http_host$request_uri;     #设定https代理服务器的协议和地址
             proxy_set_header HOST $host;
             proxy_buffers 256 4k;
             proxy_max_temp_file_size 0k;
             proxy_connect_timeout 30;
             proxy_send_timeout 60;
             proxy_read_timeout 60;
             proxy_next_upstream error timeout invalid_header http_502;

       }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }  

```

#### 启动 nginx

```
/usr/local/nginx/sbin/nginx
```

# 检测

添加本地代理（如要永久添加需要将代理配置添加到环境变量）

```
# 添加 http 代理
export http_proxy=http://127.0.0.1:8080
# 添加 https 代理
export http_proxy=https://127.0.0.1:8080
```

检测 http 代理是否成功

```
[root@centos-linux nginx]# wget http://baidu.com

```

出现

```
--2020-03-24 10:00:55--  http://baidu.com/
Connecting to 127.0.0.1:8080... connected.
Proxy request sent, awaiting response... 200 OK
Length: 81 [text/html]
Saving to: 'index.html'

100%[==================================================================================================================================================================>] 81          --.-K/s   in 0s      

2020-03-24 10:00:55 (5.14 MB/s) - 'index.html' saved [81/81]
```

检测 https 代理是否成功

```
wget https://baidu.com
```

出现

```
--2020-03-24 10:02:49--  https://baidu.com/
Connecting to 127.0.0.1:8080... connected.
Proxy request sent, awaiting response... 302 Moved Temporarily
Location: http://www.baidu.com/ [following]
--2020-03-24 10:02:49--  http://www.baidu.com/
Connecting to 127.0.0.1:8080... connected.
Proxy request sent, awaiting response... 200 OK
Length: 2381 (2.3K) [text/html]
Saving to: 'index.html.1'

100%[==================================================================================================================================================================>] 2,381       --.-K/s   in 0s      

2020-03-24 10:02:51 (154 MB/s) - 'index.html.1' saved [2381/2381]
```