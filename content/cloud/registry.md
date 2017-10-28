---
title: "部署一个带有基础认证的 docker 镜像仓库"
date: 2017-10-28T20:46:54+08:00
tags: ["docker", "registry"]
categories: 云计算
slug: deploy-docker-registry-with-basic-authentication
type: post
---



### 一、创建 auth 目录
创建 auth 目录来存放 用户名和密码

```shell
$ mkdir auth
```

### 二、生成认证文件

```shell
$ docker run \
  --entrypoint htpasswd \
  registry:2 -Bbn testuser testpassword > auth/htpasswd
```

生成了一个用户名为 `testuser`，密码为 `testpassword` 的认证文件

### 三、启动镜像仓库

```shell
docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v `pwd`/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  registry:2
```

将认证文件挂载到容器中即可。


如果有证书

```shell
$ docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v `pwd`/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  -v `pwd`/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  registry:2
```
