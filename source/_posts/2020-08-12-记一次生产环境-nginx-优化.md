---
title: 记一次生产环境 nginx 优化
tags:
  - nginx
  - 记录
  - 运维
id: '12143'
categories:
  - - skills
date: 2020-08-12 09:05:40
---

# nginx 优化

由于业务场景需要，近期将生产环境由阿里云的 SLB 更换为自建 nginx，之前在测试环境使用了一段时间一直没有问题，但是上周上到生产后出现了一系列问题，因此对 nginx 做了一些优化，记录下以备以后需要。
<!--more-->
## 高并发优化

生产环境相对于测试环境访问量多了很多，并发量大的时候会出现部分请求 502 的情况，因此对高并发进行了一点优化，主要分两方面，一方面是系统层面，一方面是 nginx 层面。

#### 系统内核层面

修改`/etc/sysctl.conf`内核参数。

```properties
fs.file-max = 999999
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_keepalive_time = 15
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_max_tw_buckets = 5000
net.core.netdev_max_backlog = 262144
net.ipv4.tcp_max_orphans = 262144
net.ipv4.tcp_tw_reuse = 1

```

使配置生效

```bash
sysctl -p
```

参数说明：

*   fs.file-max：999999
    
    > 这个参数表示进程（比如一个worker进程）可以同时打开的最大句柄数，这 个参数直接限制最大并发连接数，需根据实际情况配置。
    
*   net.ipv4.tcp\_tw\_reuse：
    
    > 这个参数设置为1，表示允许将TIME-WAIT状态的socket重新用于新的 TCP连接，这对于服务器来说很有意义，因为服务器上总会有大量TIME-WAIT状态的连接。
    
*   net.ipv4.tcp\_keepalive\_time：
    
    > 这个参数表示当keepalive启用时，TCP发送keepalive消息的频度。 默认是2小时，若将其设置得小一些，可以更快地清理无效的连接。
    
*   net.ipv4.tcp\_fin\_timeout：
    
    > 这个参数表示当服务器主动关闭连接时，socket保持在FIN-WAIT-2状态的最大时间。
    
*   net.ipv4.tcp\_max\_tw\_buckets：
    
    > 这个参数表示操作系统允许TIME\_WAIT套接字数量的最大值， 如果超过这个数字，TIME\_WAIT套接字将立刻被清除并打印警告信息。该参数默认为 180000，过多的TIME\_WAIT套接字会使Web服务器变慢。
    
*   net.core.netdev\_max\_backlog = 262144
    
    > 每个网络接口接收数据包的速率比内核处理这些包的速率快时，允许送到队列的数据包的最大数目。
    
*   net.ipv4.tcp\_max\_orphans = 262144
    
    > 系统中最多有多少个TCP套接字不被关联到任何一个用户文件句柄上。如果超过这个数字，孤儿连接将即刻被复位并打印出警告信息。这个限制仅仅是为了防止简单的DoS攻击，不能过分依靠它或者人为地减小这个值，更应该增加这个值(如果增加了内存之后)。
    
*   net.ipv4.tcp\_tw\_reuse = 1
    
    > 表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭
    

#### Nginx 配置方面

*   worker\_processes 4
    
    > nginx进程数，建议按照cpu数目来指定，一般跟cpu核数相同或为它的倍数。
    
*   worker\_cpu\_affinity 00000001 00000010 00000100 00001000 ;
    
    > 为每个进程分配cpu，上例中将4个进程分配到4个cpu，当然可以写多个，或者将一个进程分配到多个cpu。
    
*   worker\_rlimit\_nofile
    
    > 这个指令是指当一个nginx进程打开的最多文件描述符数目，理论值应该是系统的最多打开文件数（ulimit -n）与nginx进程数相除，但是nginx分配请求并不是那么均匀，所以最好与ulimit -n的值保持一致。
    
*   use epoll;
    
    > 使用epoll的I/O模型，用这个模型来高效处理异步事件
    
*   worker\_connections 65535;
    
    > 每个进程允许的最多连接数，理论上每台nginx服务器的最大连接数为worker\_processes_worker\_connections。这个值太小的后果就是你的系统会报：too many open files 等错误，导致你的系统死掉。一般给到服务器后最好通过 ulimit 命令结合修改 /etc/security/limits.conf（增加_ soft nofile = \* hard nofile=）两个配置值，将其设置的更大些。
    
*   keepalive\_timeout 30;
    
    > http连接超时时间，默认是60s，功能是使客户端到服务器端的连接在设定的时间内持续有效，当出现对服务器的后继请求时，该功能避免了建立或者重新建立连接。切记这个参数也不能设置过大！否则会导致许多无效的http连接占据着nginx的连接数，终nginx崩溃！
    
*   client\_header\_buffer\_size 4k;
    
    > 客户端请求头部的缓冲区大小，这个可以根据你的系统分页大小来设置，一般一个请求的头部大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为分页大小。分页大小可以用命令getconf PAGESIZE取得。
    
*   open\_file\_cache max=102400 inactive=20s;
    
    > 这个参数将为打开文件指定缓存，默认是没有启用的，max指定缓存数量，建议和打开文件数一致，inactive是指经过多长时间文件没被请求后删除缓存。
    
*   open\_file\_cache\_valid 30s;
    
    > 指多长时间检查一次缓存的有效信息。
    
*   open\_file\_cache\_min\_uses 1
    
    > open\_file\_cache指令中的inactive参数时间内文件的最少使用次数，如果超过这个数字，文件描述符一直是在缓存中打开的，如上例，如果有一个文件在inactive时间内一次没被使用，它将被移除。
    
*   sendfile on;
    
    > sendfile()可以在磁盘和TCP socket之间互相拷贝数据(或任意两个文件描述符)。Pre-sendfile是传送数据之前在用户空间申请数据缓冲区。之后用read()将数据从文件拷贝到这个缓冲区，write()将缓冲区数据写入网络。sendfile()是立即将数据从磁盘读到OS缓存。因为这种拷贝是在内核完成的，sendfile()要比组合read()和write()以及打开关闭丢弃缓冲更加有效(更多有关于sendfile)。
    
*   tcp\_nopush on;
    
    > 告诉nginx在一个数据包里发送所有头文件，而不一个接一个的发送。就是说数据包不会马上传送出去，等到数据包最大时，一次性的传输出去，这样有助于解决网络堵塞。
    
*   tcp\_nodelay on;
    
    > 告诉nginx不要缓存数据，而是一段一段的发送--当需要及时发送数据时，就应该给应用设置这个属性，这样发送一小块数据信息时就不能立即得到返回值。
    

## upstream 容错配置说明

1.  判断节点失效状态 Nginx默认判断失败节点状态以connect refuse和time out状态为准，不以HTTP错误状态进行判断失败，因为HTTP只要能返回状态说明该节点还可以正常连接，所以nginx判断其还是存活状态；除非添加了proxy\_next\_upstream指令设置对404、502、503、504、500和time out等错误进行转到备机处理，在next\_upstream过程中，会对fails进行累加，如果备用机处理还是错误则直接返回错误信息（但404不进行记录到错误数，如果不配置错误状态也不对其进行错误状态记录），综述，nginx记录错误数量只记录timeout 、connect refuse、502、500、503、504这6种状态，timeout和connect refuse是永远被记录错误状态，而502、500、503、504只有在配置proxy\_next\_upstream后nginx才会记录这4种HTTP错误到fails中，当fails大于等于max\_fails时，则该节点失效；
    
2.  nginx 处理节点失效和恢复的触发条件 nginx可以通过设置max\_fails（最大尝试失败次数）和fail\_timeout（失效时间，在到达最大尝试失败次数后，在fail\_timeout的时间范围内节点被置为失效，除非所有节点都失效，否则该时间内，节点不进行恢复）对节点失败的尝试次数和失效时间进行设置，当超过最大尝试次数或失效时间未超过配置失效时间，则nginx会对节点状会置为失效状态，nginx不对该后端进行连接，**直到超过失效时间或者所有节点都失效后，该节点重新置为有效，重新探测**；
    
3.  所有节点失效后nginx将重新恢复所有节点进行探测 如果探测所有节点均失效，备机也为失效时，那么nginx会对所有节点恢复为有效，重新尝试探测有效节点，如果探测到有效节点则返回正确节点内容，如果还是全部错误，那么继续探测下去，当没有正确信息时，节点失效时默认返回状态为502，但是下次访问节点时会继续探测正确节点，直到找到正确的为止。
    
4.  通过proxy\_next\_upstream实现容灾和重复处理问题 ngx\_http\_proxy\_module 模块中包括proxy\_next\_upstream指令 语法: proxy\_next\_upstream error timeout invalid\_header http\_500 http\_502 http\_503 http\_504 http\_404 off ...; 默认值: proxy\_next\_upstream error timeout; 上下文: http, server, location
    

```
其中：
error   表示和后端服务器建立连接时，或者向后端服务器发送请求时，或者从后端服务器接收响应头时，出现错误。
timeout   表示和后端服务器建立连接时，或者向后端服务器发送请求时，或者从后端服务器接收响应头时，出现超时。
invalid_header   表示后端服务器返回空响应或者非法响应头
http_500   表示后端服务器返回的响应状态码为500
http_502   表示后端服务器返回的响应状态码为502
http_503   表示后端服务器返回的响应状态码为503
http_504   表示后端服务器返回的响应状态码为504
http_404   表示后端服务器返回的响应状态码为404
off   表示停止将请求发送给下一台后端服务器
```

* * *

**常用 nginx 标准配置（仅供参考）**

```properties
user   www www;
worker_processes 8;
worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000;
error_log   /www/log/nginx_error.log   crit;
pid         /usr/local/nginx/nginx.pid;
worker_rlimit_nofile 65535;

events
{
   use epoll;
   worker_connections 65535;
}

http
{
   include       mime.types;
   default_type   application/octet-stream;

   charset   utf-8;

   server_names_hash_bucket_size 128;
   client_header_buffer_size 2k;
   large_client_header_buffers 4 4k;
   client_max_body_size 8m;

   sendfile on;
   tcp_nopush     on;

   keepalive_timeout 60;

   fastcgi_cache_path /usr/local/nginx/fastcgi_cache levels=1:2
                 keys_zone=TEST:10m
                 inactive=5m;
   fastcgi_connect_timeout 300;
   fastcgi_send_timeout 300;
   fastcgi_read_timeout 300;
   fastcgi_buffer_size 16k;
   fastcgi_buffers 16 16k;
   fastcgi_busy_buffers_size 16k;
   fastcgi_temp_file_write_size 16k;
   fastcgi_cache TEST;
   fastcgi_cache_valid 200 302 1h;
   fastcgi_cache_valid 301 1d;
   fastcgi_cache_valid any 1m;
   fastcgi_cache_min_uses 1;
   fastcgi_cache_use_stale error timeout invalid_header http_500; 
   open_file_cache max=204800 inactive=20s;
   open_file_cache_min_uses 1;
   open_file_cache_valid 30s; 

   tcp_nodelay on;

   gzip on;
   gzip_min_length   1k;
   gzip_buffers     4 16k;
   gzip_http_version 1.0;
   gzip_comp_level 2;
   gzip_types       text/plain application/x-javascript text/css application/xml;
   gzip_vary on;

 upstream  test {
        server 192.168.5.34:9083 max_fails=3  fail_timeout=5s;
        server 192.168.5.35:9083 max_fails=3  fail_timeout=5s;
        server 192.168.5.36:9083 max_fails=3  fail_timeout=5s;
    }
   server
   {
     listen       80;
     server_name   www.test.com;
     index index.php index.htm;
     root   /www/html/;

   location /
     {
        proxy_pass http://test;
            proxy_set_header   Host    $host;
            proxy_set_header   X-Real-IP   $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
     }
     location /status
     {
         stub_status on;
     }

     location ~ .*\.(phpphp5)?$
     {
         fastcgi_pass 127.0.0.1:9000;
         fastcgi_index index.php;
         include fcgi.conf;
     }

     location ~ .*\.(gifjpgjpegpngbmpswfjscss)$
     {
       expires       30d;
     }

     log_format   access   '$remote_addr - $remote_user [$time_local] "$request" '
               '$status $body_bytes_sent "$http_referer" '
               '"$http_user_agent" $http_x_forwarded_for';
     access_log   /www/log/access.log   access;
       }
}
```