---
title: "centos 安装指定版本的 docker"
date: 2017-10-21T16:56:06+08:00
tags: ['centos', 'docker', "install"]
categories: 运维
series: ops
type: post
---


增加 docker 源

```
cat >/etc/yum.repos.d/docker.repo <<-EOF

[docker-main-repo]
name=Docker main Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF
```

```
yum install epel-release
yum makecache
```

列出所有版本：
```
# yum list docker-engine --showduplicates | sort -r
Loaded plugins: langpacks
Installed Packages
docker-engine.x86_64          1.9.1-1.el7.centos               docker-main-repo 
docker-engine.x86_64          1.9.0-1.el7.centos               docker-main-repo 
docker-engine.x86_64          1.8.3-1.el7.centos               docker-main-repo 
docker-engine.x86_64          1.8.2-1.el7.centos               docker-main-repo 
docker-engine.x86_64          1.8.1-1.el7.centos               docker-main-repo 
docker-engine.x86_64          1.8.0-1.el7.centos               docker-main-repo 
docker-engine.x86_64          1.7.1-1.el7.centos               docker-main-repo 
docker-engine.x86_64          17.03.0.ce-1.el7.centos          docker-main-repo 
docker-engine.x86_64          1.7.0-1.el7.centos               docker-main-repo 
docker-engine.x86_64          1.13.1-1.el7.centos              docker-main-repo 
docker-engine.x86_64          1.13.1-1.el7.centos              @docker-main-repo
docker-engine.x86_64          1.13.0-1.el7.centos              docker-main-repo 
docker-engine.x86_64          1.12.6-1.el7.centos              docker-main-repo 
docker-engine.x86_64          1.12.5-1.el7.centos              docker-main-repo 
docker-engine.x86_64          1.12.4-1.el7.centos              docker-main-repo 
docker-engine.x86_64          1.12.3-1.el7.centos              docker-main-repo 
docker-engine.x86_64          1.12.2-1.el7.centos              docker-main-repo 
docker-engine.x86_64          1.12.1-1.el7.centos              docker-main-repo 
docker-engine.x86_64          1.12.0-1.el7.centos              docker-main-repo 
docker-engine.x86_64          1.11.2-1.el7.centos              docker-main-repo 
docker-engine.x86_64          1.11.1-1.el7.centos              docker-main-repo 
docker-engine.x86_64          1.11.0-1.el7.centos              docker-main-repo 
docker-engine.x86_64          1.10.3-1.el7.centos              docker-main-repo 
docker-engine.x86_64          1.10.2-1.el7.centos              docker-main-repo 
docker-engine.x86_64          1.10.1-1.el7.centos              docker-main-repo 
docker-engine.x86_64          1.10.0-1.el7.centos              docker-main-repo 
```

安装指定版本
```
yum -y install docker-engine-<VERSION_STRING>
yum -y install docker-engine-1.12.6-1.el7.centos docker-engine-selinux-1.12.6-1.el7.centos
```

允许开机启动与启动服务
```
systemctl enable docker.service
systemctl start docker.service
```

docker 配置

```
cat >/etc/docker/daemon.json <<-EOF
{
    "insecure-registries": ["192.168.1.111:5000"],
	"registry-mirrors": ["https://0yjmvwvv.mirror.aliyuncs.com"],
	"storage-driver": "devicemapper"
}
EOF
```

删除 docker
```
yum list installed|grep docker
yum remove docker-engine
```