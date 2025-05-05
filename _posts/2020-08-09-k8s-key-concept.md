---
title: k8s的核心概念
date: 2020-08-09 20:30:00 +0800
categories: [blog, 技术]
tags: [k8s]     # TAG names should always be lowercase
---

# k8s的核心概念

[K8S集群架构图](https://d33wubrfki0l68.cloudfront.net/7016517375d10c702489167e704dcb99e570df85/7bb53/images/docs/components-of-kubernetes.png)

## Control Plane Componet

### kubectl

与集群api服务沟通的命令行工具

### kube-apiserver

集群的api服务器

### etcd

存储k8s集群数据的分布式数据库。

### kube-scheduler

调度器。

### kube-control-manager

控制进程管理器。

* Node Control： 控制Node
* Replication Control： 控制副本
* EndPoint Control： 端点控制
* Service Control & Token Control： 服务控制和token 控制
* 


## Node Componet

### kubelet

运行在集群中的每一个节点上，管理所有容器都运行在Pod中；不管理非K8S创建的容器。

### kube-proxy

运行在集群的每一个节点，是网络的代理。控制来自外部、内部的网络和Pod通信。

### container-runtime

容器运行时，容器运行的环境 or 运行容器的软件。K8S支持的容器运行时包括Docker、rktlet、以及实现了K8S的CRI的，CRI的意思是容器运行时接口。

