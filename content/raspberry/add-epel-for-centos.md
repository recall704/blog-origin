---
title: "Add epel for centos"
date: 2017-10-21T17:01:53+08:00
tags: ['树莓派', 'epel', "centos","raspberry"]
categories: 树莓派
series: 树莓派
type: post
---




```
cat > /etc/yum.repos.d/epel.repo << EOF
[epel]
name=Epel rebuild for armhfp
baseurl=https://armv7.dev.centos.org/repodir/epel-pass-1/
enabled=1
gpgcheck=0

EOF
```