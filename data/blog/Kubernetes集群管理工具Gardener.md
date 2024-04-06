---
date: 2019-5-23
title: Kubernetes集群管理工具Gardener
tags: [Kubernetes]
---

随着应用容器化推进，容器多云混合云的诉求越来越旺盛，多Kubernetes管理的工具也越来越多，不同的工具采取的实现思路也不同。[Gardener](https://gardener.cloud)是一个多Kubernetes集群管理工具，通过在Kubernetes控制面增加Controller方式实现多Kubernetes Master管理。[Gardener](https://gardener.cloud)已经在[Github](https://github.com/gardener/gardener)开源。

Gardener的管理模式与[集群联邦](https://github.com/kubernetes-sigs/kubefed)不同，Gardener把被管理的Kubernetes的Master组件（etcd/api-server/controller...）部署到一个称为 **seed cluster**的Kubernetes集群中，而实际的worker节点所在的集群是一个虚拟集群，并没有Kubernetes Master，所有的worker节点都连接到seed cluster里的Kubernetes Master上。

Gardener的架构图如下
![](https://github.com/gardener/gardener-docs/blob/master/images/tam-block-diagram-overview.png?raw=true)

几个关键概念如下：

- Garden Cluster：包含Gardener Controller的集群，Gardener Controller本身也是一些容器进程
- Seed Cluster：运行Shoot Cluster的Kubernetes Master组件的集群
- Shoot Cluster：worker集群，是一个虚拟集群，不包含Kubernetes Master