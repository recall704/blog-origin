---
title: "Clair 源码简要分析（未完待续）"
date: 2017-12-20T20:24:42+08:00
tags: ["scan", "clair", "容器", "镜像"]
categories: 云计算
slug: clair
type: post
toc: true
---


## 概述
[clair](https://github.com/coreos/clair) 是 coreos 发布的一款开源容器漏洞扫描工具，该工具可以交叉检查Docker镜像的操作系统以及上面安装的任何包是否与任何已知不安全的包版本相匹配。

在开始分析 clair 之前，我们明白几点：  
1. clair 是以静态分析的方式对镜像进行分析的，有点类似于杀毒软件用特征码来扫描病毒。
2. clair 镜像分析是按镜像层级来进行的，如果某一层的软件有漏洞，在上层被删除了，该漏洞还是存在的。
3. clair 的漏洞扫描是通过软件版本比对来完成的，如果某个应用，比如 nginx ，它在镜像中的版本为 1.0.0，而该版本在数据库中存在 1.0.0 对应的漏洞数据，则表示该镜像存在对应的漏洞。

## clair 中的几个概念

### 1. Namespace
这里的 namespace 既不是`租户`也不是`命名空间`，而是系统或软件的名称。  
比如 debian，NodeJS。

### 2. Feature
在镜像层中检测到的软件包，但是没有检测到 namespace。  
比如： Name: OpenSSL, Version: 1.0, VersionFormat: dpkg

### 3. NamespacedFeature
和 Feature 类似，同时检测到了 namespace。  
比如： OpenSSL 1.0 dpkg Debian:7

### 4. Vulnerability
Vulnerability 是应用对应的漏洞描述。

## 代码分析
我们直接从镜像扫描开始分析代码

代码在 `github.com/coreos/clair/worker.go` 中

```go
func processLayers(datastore database.Datastore, imageFormat string, requests []LayerRequest) ([]database.LayerWithContent, error) {
	toDetect := []processRequest{}
	layers := map[string]database.LayerWithContent{}
	for _, req := range requests {
		if _, ok := layers[req.Hash]; ok {
			continue
		}
		layer, preq, err := getLayer(datastore, req)
		if err != nil {
			return nil, err
		}
		layers[req.Hash] = layer
		if preq != nil {
			toDetect = append(toDetect, *preq)
		}
	}

	namespaces, features, partialRes, err := processRequests(imageFormat, toDetect)
	if err != nil {
		return nil, err
	}

	// Store partial results.
	if err := persistNamespaces(datastore, namespaces); err != nil {
		return nil, err
	}

	if err := persistFeatures(datastore, features); err != nil {
		return nil, err
	}

	for _, res := range partialRes {
		if err := persistPartialLayer(datastore, res); err != nil {
			return nil, err
		}
	}

	// NOTE(Sida): The full layers are computed using partially
	// processed layers in current database session. If any other instances of
	// Clair are changing some layers in this set of layers, it might generate
	// different results especially when the other Clair is with different
	// processors.
	completeLayers := []database.LayerWithContent{}
	for _, req := range requests {
		if partialLayer, ok := partialRes[req.Hash]; ok {
			completeLayers = append(completeLayers, combineLayers(layers[req.Hash], partialLayer))
		} else {
			completeLayers = append(completeLayers, layers[req.Hash])
		}
	}

	return completeLayers, nil
}
```
