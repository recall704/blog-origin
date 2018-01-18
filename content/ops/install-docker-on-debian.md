---
title: "debian 安装指定版本的 docker"
date: 2017-10-21T16:56:20+08:00
tags: ['debian', 'docker', "install"]
categories: 运维
series: ops
type: post
toc: true
---


### 1. 加入软件源

```
cat >> /etc/apt/sources.list <<-EOF
deb [arch=amd64] https://apt.dockerproject.org/repo debian-jessie main
EOF
```

不能连接 https 时

```
apt install apt-transport-https
```

### 2. 更新 metadata

```
apt update
```

### 3. 列出可选版本

```
apt-cache policy docker-engine
docker-engine:
    Installed: 1.13.1-0~debian-jessie
    Candidate: 1.13.1-0~debian-jessie
    Version table:
    *** 1.13.1-0~debian-jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
        100 /var/lib/dpkg/status
    1.13.0-0~debian-jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.12.6-0~debian-jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.12.5-0~debian-jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.12.4-0~debian-jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.12.3-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.12.2-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.12.1-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.12.0-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.11.2-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.11.1-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.11.0-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.10.3-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.10.2-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.10.1-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.10.0-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.9.1-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.9.0-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.8.3-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.8.2-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.8.1-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.8.0-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.7.1-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.7.0-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.6.2-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.6.1-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.6.0-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
    1.5.0-0~jessie 0
        500 https://apt.dockerproject.org/repo/ debian-jessie/main amd64 Packages
```

### 4. 安装

```
apt install docker-engine=1.12.6-0~debian-jessie
```


### 5. 卸载

```
dpkg -l|grep docker
apt-get purge docker-engine
```

### 6. 其它
1. GPG 错误解决  
如果出现
W: GPG error: https://apt.dockerproject.org debian-jessie InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY

```
gpg --keyserver pgpkeys.mit.edu --recv-key  F76221572C52609D      
gpg -a --export F76221572C52609D | sudo apt-key add -
```

2. debian 版本号  

```
debian 6    squeeze  
debian 7    wheezy  
debian 8    jessie  
debian 9    stretch  
```

软件源需要根据系统版本进行修改。