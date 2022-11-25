---
title: Docker upgrade issue
date: 2022-11-25 17:32:22
tags:
 - skills
 - docker
categories:
 - skills
---

# Why

We have a server running docker version `17.06`，one day there was a issue：[issue](https://github.com/docker/for-linux/issues/1)

<!--more-->
So yesterday we started upgrading Docker from `17.06` to `20.10`

Then came the disaster that lasted for more than a day😂😂

# Issue

After the upgrade, there have error when start docker container:

```
docker: Error response from daemon: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container 
```

We tried restarting docker and restarting the server, neither worked.

But we start docker with debug is work, then we found all container network has issue . A container under the same network cannot link to another container through the container name, and the ip can be pinged, but the port is blocked.



We created a httpserver on that server(Docker 20.10.21 )

```
docker run -p 8000:8000 -it python:3.7-slim python3 -m http.server --bind 0.0.0.0
```

Then we run `curl 127.0.0.1:8000` on monitor server , got

```
curl: (56) Recv failure: Connection reset by peer
```

But I tested it on other server (Docker 19.03.11) server and got

```
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>Directory listing for /</title>
</head>
<body>
<h1>Directory listing for /</h1>
<hr>
<ul>
<li><a href=".ansible/">.ansible/</a></li>
<li><a href=".ansible_galaxy">.ansible_galaxy</a></li>
...
...
...
```

And we also tested in my local(Docker 20.10.21), Same as on kibana.

# Resloved

We found a same [issue](https://stackoverflow.com/questions/43153503/docker-on-centos-7-2-kernelunregister-netdevice-waiting-for-lo-to-become-free)

And upgraded the kernel form `3.10.0-514.26.2.el7.x86_64` to `3.10.0-1160.80.1.el7.x86_64` and rebooted server .
Not sure what was causing the problem, 

now the newly created container is also normal.

