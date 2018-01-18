---
title: "Deploy K8s With Kubeadm"
date: 2018-01-07T20:32:40+08:00
tags: ["kubernetes","deploy", "kubeadm"]
categories: 云计算
slug: deploy-k8s-with-kubeadm
type: post
toc: true
---

## 一、环境准备

两台 centos 7.2 mini 系统，virtualbox 安装的系统，桥接模式，能正常访问网络  
node1： 192.168.88.50  
node2： 192.168.88.51  

## 二、环境预处理在两台机器上都执行下面的命令
### 1. 关闭防火墙

```bash
systemctl stop firewalld
```

### 2. sysctl 配置

```bash
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

### 3. 时间同步

```bash
yum install -y ntpdate
ntpdate cn.ntp.org.cn
```

### 4. 启用 epel

```bash
yum install -y epel-release
```
### 5. 禁用虚拟内存

```bash
swapoff -a
```
### 6. 设置 hostname

node1 上执行
```bash
hostnamectl set-hostname node1
```

node2 上执行
```bash
hostnamectl set-hostname node2
```

## 三、安装 docker

```bash
yum install -y docker
systemctl enable docker && systemctl start docker
```

## 四、安装 kubeadm, kubelet and kubectl

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
setenforce 0
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet
```

## 五、运行 kubeadm

```bash
kubeadm init --pod-network-cidr=172.20.0.0/16
```

## 六、部署网络插件 calico

```bash
kubectl apply -f https://docs.projectcalico.org/v2.6/getting-started/kubernetes/installation/hosted/kubeadm/1.6/calico.yaml
```


## 七、对应视频

<iframe width="560" height="315" src="https://www.youtube.com/embed/_PCJPJdEIUE" frameborder="0" gesture="media" allow="encrypted-media" allowfullscreen></iframe>