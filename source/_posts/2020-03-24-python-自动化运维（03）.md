---
title: Python 自动化运维（03）
tags:
  - Python
  - 自动化
  - 运维
id: '12021'
categories:
  - - skills
date: 2020-03-24 21:57:41
---

> 操作系统 macOS Catalina Python版本：3.7 所有代码可在 [GitHub](https://github.com/Swq1227/devops "GitHub") 获取
<!--more-->
# DNS 处理模块 dnspython

`dnspython`是 Python 实现的一个 DNS 工具包，它几乎支持所有的记录类型，可以用于查询。传输并动态更新 ZONE 信息，同时支持 TSIG（事务签名）验证消息和 EDNS0（扩展 DNS）。

### 常见解析类型实例说明

常见的 DNS 解析类型包括 A、MX、NS、CNAME 等，利用 dnspython 的 `dns.resolver.query`方法可以简单实现这些 DNS 类型的查询。

#### A 记录查询

```python
import dns.resolver

domain = input('Please input an domain: ')  # 输入域名地址
A = dns.resolver.query(domain, 'A')  # 指定查询类型为 A 记录
for i in A.response.answer:
    for j in i.items:
        print(j.address)

```

运行结果 ![image.png](https://i.loli.net/2020/03/24/qFmW9xjbLRCo5Si.png)

#### MX 记录查询

```python
import dns.resolver

domain = input('Please input an domain: ')  # 输入域名地址
MX = dns.resolver.query(domain, 'MX')  # 指定查询类型为 MX# 记录
for i in MX:
    print('MX preference =',i.preference,'mail exchanger=',i.exchange)

```

![image.png](https://i.loli.net/2020/03/24/rWEgtG2On4H7UKM.png)

#### NS 记录查询

```python
import dns.resolver

domain = input('Please input an domain: ')  # 输入域名地址
NS = dns.resolver.query(domain, 'NS')  # 指定查询类型为 NS 记录
for i in NS.response.answer:
    for j in i.items:
        print(j.to_text)
```

只限输入一级域名，运行结果 ![image.png](https://i.loli.net/2020/03/24/rWEgtG2On4H7UKM.png)

#### CNAME 记录查询

```python
import dns.resolver

domain = input('Please input an domain: ')  # 输入域名地址
CNAME = dns.resolver.query(domain, 'CNAME')  # 指定查询类型为 CNAME 记录
for i in CNAME.response.answer:
    for j in i.items:
        print(j.to_text)
```

运行结果 ![image.png](https://i.loli.net/2020/03/24/SjBQ1izRNrZkUlc.png)

### DNS 域名轮循业务监控

查询业务域名 A 记录信息，查询出所有 IP 地址列表，再使用 httplib 模块的 request 方法以 GET 方式请求监控页面，监控业务所有的 IP 是否服务正常

```python
import dns.resolver
import os
import httplib2

iplist = []  # 定义域名 IP 列表变量
appdomain = 'www.baidu.com'  # 定义业务域名


def get_iplist(domain=""):  # 域名解析行数，解析成功的 ip 将被追加到 iplist
    try:
        A = dns.resolver.query(domain, 'A')
    except Exception as e:
        print("dns resolver error:", str(e))
        return
    for i in A.response.answer:
        for j in i.items:
            iplist.append(j.address)  # 将 ip 追加到 iplist
    return True


def checkip(ip):
    checkurl = ip + ":80"
    getcontent = ""
    httplib2.socket.setdefaulttimeout(5)  # 定义 http 连接超时时间，单位秒
    conn = httplib2.HTTPConnectionWithTimeout(checkurl)  # 创建连接对象

    try:
        conn.request("GET", "/", headers={"Host": appdomain})  # 发起 URL 请求，添加 host 主机头
        r = conn.getresponse()
        getcontent = r.read(15)  # 获取 URL 页面前 15 个字符
    finally:
        if "html" in str(getcontent):  # 判断是否包含 html 字符，包含说明请求成功
            print(ip + "[OK]")
        else:
            print(ip + "[ERROR]")


if __name__ == '__main__':
    if get_iplist(appdomain) and len(iplist) > 0:  # 条件：域名解析正确且至少返回一个 IP
        for ip in iplist:
            checkip(ip)
    else:
        print("dns reslover error")

```

运行结果 ![image.png](https://i.loli.net/2020/03/24/HtLNymF8Xo5alEQ.png) 将此脚本放到 crontab 中定时运行，再结合告警程序，这样一个基于域名轮循的业务监控就已经完成。