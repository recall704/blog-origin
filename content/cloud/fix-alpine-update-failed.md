---
title: "Fix Alpine Update Failed"
date: 2018-01-18T17:54:53+08:00
tags: ["docker","alpine", "apk", "update"]
categories: 云计算
slug: fix-alpine-update-failed
type: post
include_toc: false
---


之前为了统一一个 golang 的编译环境，做了一个对应的镜像， Dockerfile 如下

```
FROM golang:1.8.5-alpine3.6
MAINTAINER @recall704 https://github.com/recall704

RUN apk update && \
    apk add gcc linux-headers musl-dev && \
    rm -rf /var/cache/* && \
    echo "https://mirrors.ustc.edu.cn/alpine/v3.6/main/" > /etc/apk/repositories && \
    echo "https://mirrors.ustc.edu.cn/alpine/v3.6/community/" >> /etc/apk/repositories
```

正常跑着没问题，但是

```
docker run --rm -it win7/golang:v1.8.5-gcc sh
```

进入容器后，我想更新一个 go 的 package，发现没有 git，很自然的想到 

```
apk update
apk add git
```

但是

```
/go # apk update
fetch https://mirrors.ustc.edu.cn/alpine/v3.6/main/x86_64/APKINDEX.tar.gz
ERROR: https://mirrors.ustc.edu.cn/alpine/v3.6/main/: Bad file descriptor
WARNING: Ignoring APKINDEX.c71de075.tar.gz: Bad file descriptor
1 errors; 26 distinct packages available
```

这神了个奇了，随手 google 了下。

修改 Dockerfile 为

```
FROM golang:1.8.5-alpine3.6
MAINTAINER @recall704 https://github.com/recall704

RUN apk update && \
    apk add gcc linux-headers musl-dev && \
    rm -rf /var/cache/* && \
    mkdir -p /var/cache/apk/ && \
    echo "https://mirrors.ustc.edu.cn/alpine/v3.6/main/" > /etc/apk/repositories && \
    echo "https://mirrors.ustc.edu.cn/alpine/v3.6/community/" >> /etc/apk/repositories
```

就加了一行

```
mkdir -p /var/cache/apk/
```

然后就可以正常更新了， 不信的可以自己试试。