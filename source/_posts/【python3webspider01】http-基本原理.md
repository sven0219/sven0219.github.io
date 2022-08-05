---
title: 【Python3WebSpider01】HTTP 基本原理
tags:
  - http
  - Python
  - 爬虫
id: '12121'
categories:
  - - skills
date: 2020-08-06 17:18:27
---

# HTTP 基本原理

## 1\. URI 和 URL

URI的全称为Uniform Resource Identifier，即统一资源标志符，URL的全称为Universal Resource Locator，即统一资源定位符。 简单来说，https://www.baidu.com/favicon.ico 是 Baidu 的网站图标链接，它是一个 URI 也是一个 URL。即有这样的一个图标资源，我们用URL/URI来唯一指定了它的访问方式，这其中包括了访问协议https、访问路径（/即根目录）和资源名称favicon.ico。通过这样一个链接，我们便可以从互联网上找到这个资源，这就是URL/URI。 <!--more-->URL是URI的子集，也就是说每个URL都是URI，但不是每个URI都是URL。URI 还包括一个子类叫做 URN，全称Universal Resource Name，即统一资源名称。URN只命名资源而不指定如何定位资源，比如一本书的ISBN，可以唯一标识这本书，但是没有指定到哪里定位这本书，这就是URN。

## 2\. 超文本

我们在浏览器里看到的网页就是超文本解析而成的，其网页源代码是一系列HTML代码，里面包含了一系列标签，比如img显示图片，p指定显示段落等。浏览器解析这些标签后，便形成了我们平常看到的网页，而网页的源代码HTML就可以称作超文本。

## 3\. HTTP 和 HTTPS

在爬虫中，抓取的页面通常是 http 或者 https 协议，这和我们常用的 ftp、sftp、smb 一样都是访问资源的一种协议类型。 HTTP的全称是Hyper Text Transfer Protocol，中文名叫作超文本传输协议。HTTP协议是用于从网络传输超文本数据到本地浏览器的传送协议，它能保证高效而准确地传送超文本文档。HTTP由万维网协会（World Wide Web Consortium）和Internet工作小组IETF（Internet Engineering Task Force）共同合作制定的规范，目前广泛使用的是HTTP 1.1版本。 HTTPS的全称是Hyper Text Transfer Protocol over Secure Socket Layer，是以安全为目标的HTTP通道，简单讲是HTTP的安全版，即HTTP下加入SSL层，简称为HTTPS。 HTTPS的安全基础是SSL，因此通过它传输的内容都是经过SSL加密的。

## 4.HTTP 请求过程

[http请求过程](https://www.52ynn.top/index.php/2020/06/17/%e6%b5%8f%e8%a7%88%e5%99%a8%e8%ae%bf%e9%97%ae%e4%b8%80%e4%b8%aa%e7%bd%91%e9%a1%b5%e7%9a%84%e8%af%a6%e7%bb%86%e8%bf%87%e7%a8%8b/ "http请求过程")

## 5.请求

请求，由客户端向服务端发出，可以分为4部分内容：请求方法（Request Method）、请求的网址（Request URL）、请求头（Request Headers）、请求体（Request Body）。

*   请求方法 常见的请求方法有两种：GET和POST。
    
    方法
    
    描述
    
    GET
    
    请求页面，并返回页面内容
    
    POST
    
    大多用于提交表单或上传文件，数据包含在请求体中
    
    PUT
    
    从客户端向服务器传送的数据取代指定文档中的内容
    
    DELETE
    
    请求服务器删除指定的页面
    
    CONNECT
    
    把服务器当作跳板，让服务器代替客户端访问其他网页
    
    OPTIONS
    
    允许客户端查看服务器的性能
    
    TRACE
    
    回显服务器收到的请求，主要用于测试或诊断
    
*   请求的网址 请求的网址，即统一资源定位符URL，它可以唯一确定我们想请求的资源。
    
*   请求头

```
请求头，用来说明服务器要使用的附加信息，比较重要的信息有Cookie、Referer、User-Agent等。下面简要说明一些常用的头信息。

Accept：请求报头域，用于指定客户端可接受哪些类型的信息。
Accept-Language：指定客户端可接受的语言类型。
Accept-Encoding：指定客户端可接受的内容编码。
Host：用于指定请求资源的主机IP和端口号，其内容为请求URL的原始服务器或网关的位置。从HTTP 1.1版本开始，请求必须包含此内容。
Cookie：也常用复数形式 Cookies，这是网站为了辨别用户进行会话跟踪而存储在用户本地的数据。它的主要功能是维持当前访问会话。例如，我们输入用户名和密码成功登录某个网站后，服务器会用会话保存登录状态信息，后面我们每次刷新或请求该站点的其他页面时，会发现都是登录状态，这就是Cookies的功劳。Cookies里有信息标识了我们所对应的服务器的会话，每次浏览器在请求该站点的页面时，都会在请求头中加上Cookies并将其发送给服务器，服务器通过Cookies识别出是我们自己，并且查出当前状态是登录状态，所以返回结果就是登录之后才能看到的网页内容。
Referer：此内容用来标识这个请求是从哪个页面发过来的，服务器可以拿到这一信息并做相应的处理，如作来源统计、防盗链处理等。
User-Agent：简称UA，它是一个特殊的字符串头，可以使服务器识别客户使用的操作系统及版本、浏览器及版本等信息。在做爬虫时加上此信息，可以伪装为浏览器；如果不加，很可能会被识别出为爬虫。
Content-Type：也叫互联网媒体类型（Internet Media Type）或者MIME类型，在HTTP协议消息头中，它用来表示具体请求中的媒体类型信息。例如，text/html代表HTML格式，image/gif代表GIF图片，application/json代表JSON类型，更多对应关系可以查看此对照表：http://tool.oschina.net/commons。
因此，请求头是请求的重要组成部分，在写爬虫时，大部分情况下都需要设定请求头。
```

*   请求体 请求体一般承载的内容是POST请求中的表单数据，而对于GET请求，请求体则为空。

## 6 响应

响应，由服务端返回给客户端，可以分为三部分：响应状态码（Response Status Code）、响应头（Response Headers）和响应体（Response Body）

*   响应状态码 响应状态码表示服务器的响应状态，如200代表服务器正常响应，404代表页面未找到，500代表服务器内部发生错误。在爬虫中，我们可以根据状态码来判断服务器响应状态，如状态码为200，则证明成功返回数据，再进行进一步的处理，否则直接忽略。表2-3列出了常见的错误代码及错误原因。
    
*   响应头 响应头包含了服务器对请求的应答信息，如Content-Type、Server、Set-Cookie等。下面简要说明一些常用的头信息。
    

```
Date：标识响应产生的时间。
Last-Modified：指定资源的最后修改时间。
Content-Encoding：指定响应内容的编码。
Server：包含服务器的信息，比如名称、版本号等。
Content-Type：文档类型，指定返回的数据类型是什么，如text/html代表返回HTML文档，application/x-javascript则代表返回JavaScript文件，image/jpeg则代表返回图片。
Set-Cookie：设置Cookies。响应头中的Set-Cookie告诉浏览器需要将此内容放在Cookies中，下次请求携带Cookies请求。
Expires：指定响应的过期时间，可以使代理服务器或浏览器将加载的内容更新到缓存中。如果再次访问时，就可以直接从缓存中加载，降低服务器负载，缩短加载时间。
```

*   响应体 最重要的当属响应体的内容了。响应的正文数据都在响应体中，比如请求网页时，它的响应体就是网页的HTML代码；请求一张图片时，它的响应体就是图片的二进制数据。我们做爬虫请求网页后，要解析的内容就是响应体