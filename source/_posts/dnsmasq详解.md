---
title: 记一次Dnsmasq使用
tags:
  - 教程
  - 运维
id: '12114'
categories:
  - - skills
date: 2020-07-15 13:41:14
---

# 背景

公司做灰度发布，于是想本地搭建一个 DNS 解析分别印射到不同的环境，方便测试同事进行测试。

# 什么是 DNSmasq

Dnsmasq提供DNS缓存和DHCP服务、Tftp服务功能。作为域名解析服务器(DNS)，Dnsmasq可以通过缓存DNS请求来提高对访问过的网址的连接速度。作为DHCP服务器，Dnsmasq可以为局域网电脑提供内网ip地址和路由。DNS和DHCP两个功能可以同时或分别单独实现。Dnsmasq轻量且易配置，适用于个人用户或少于50台主机的网络。此外它还自带了一个PXE服务器。

# DNSmasq 主要作用

1.  将Dnsmasq作为本地DNS服务器使用，直接修改电脑的本地DNS的IP地址即可。
2.  应对ISP的DNS劫持（反DNS劫持），输入一个不存在的域名，正常的情况下浏览器是显示无法连接，DNS劫持会跳转到一个广告页面。先随便nslookup 一个不存在的域名，看看ISP商劫持的IP地址。
3.  智能DNS加快解析速度，打开/etc/dnsmasq.conf文件，server=后面可以添加指定的DNS，例如国内外不同的网站使用不同的DNS。

**国内指定DNS**

```
server=/cn/114.114.114.114
server=/taobao.com/114.114.114.114
server=/taobaocdn.com/114.114.114.114
```

**国外指定DNS**

```
server=/google.com/8.8.8.8
```

4.  屏蔽网页广告，将指广告的URL指定127这个IP，就可以将网页上讨厌的广告给去掉了。

```
address=/ad.youku.com/127.0.0.1
address=/ad.iqiyi.com/127.0.0.1
```

5.  指定域名解析到特定的IP上。这个功能可以让你控制一些网站的访问，非法的DNS就经常把一些正规的网站解析到不正确IP上。

```
address=/freehao123.com/123.123.123.123
```

6.  管理控制内网DNS，首先将局域网中的所有的设备的本地DNS设置为已经安装Dnsmasq的服务器IP地址。然后修改已经安装Dnsmasq的服务器Hosts文件：/etc/hosts，指定域名到特定的IP中。

# DNSmasq 原理

dnsmasq先去解析hosts文件， 再去解析/etc/dnsmasq.d/下的\*.conf文件，并且这些文件的优先级要高于dnsmasq.conf，我们自定义的resolv.dnsmasq.conf中的DNS也被称为上游DNS，这是最后去查询解析的； 如果不想用hosts文件做解析，我们可以在/etc/dnsmasq.conf中加入no-hosts这条语句，这样的话就直接查询上游DNS了，如果我们不想做上游查询，就是不想做正常的解析，我们可以加入no-reslov这条语句。

# 安装

Linux 软件仓库已经提供了 DNSmasq，相关命令如下

```
yum -y install dnsmasq
```

dnsmasq 的配置文件在`/etc/dnsmasq.conf` ,这个配置文件包含大量的选项注释

# 配置

`resolv-file` ： 定义dnsmasq从哪里获取上游DNS服务器的地址， 默认从/etc/resolv.conf获取。 `strict-order` ：表示严格按照resolv-file文件中的顺序从上到下进行DNS解析，直到第一个解析成功为止。 `listen-address` ：定义dnsmasq监听的地址，默认是监控本机的所有网卡上。 `address`：启用泛域名解析，即自定义解析a记录，例如：address=/long.com/192.168.115.10 访问long.com时的所有域名都会被解析成192.168.115.10 `bogus-nxdomain` ： 对于任何被解析到此 IP 的域名，将响应 NXDOMAIN 使其解析失效，可以多次指定通常用于对于访问不存在的域名，禁止其跳转到运营商的广告站点 `server` ： 指定使用哪个DNS服务器进行解析，对于不同的网站可以使用不同的域名对应解析。 例如：server=/google.com/8.8.8.8 #表示对于google的服务，使用谷歌的DNS解析 因为我这边需求只需要用到内网 dns 控制，故只修改如下配置

```
resolv-file=/etc/resolv.dnsmasq.conf    //dnsmasq 会从这个文件中寻找上游dns服务器
strict-order             //去掉前面的#
addn-hosts=/etc/dnsmasq.hosts         //在这个文件里面添加记录
listen-address=127.0.0.1,192.168.1.15     //监听地址
```

查看配置文件语法是否正确

```
dnsmasq -test
```