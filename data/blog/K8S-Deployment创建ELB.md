---
date: 2018-10-17
title: K8S-Deployment创建ELB
tags: [Cloud-Service-Mapping, Computing, K8S, 负载均衡]
---

### 场景
Service发布到负载均衡是最常用的功能，当前云服务商提供的负载均衡并未与K8S Service关联，但是当负载均衡对象不存在的时候，需要手动创建Service。

### 实现
#### 阿里云
在K8S Service 通过注解定义负载均衡，负载均衡的生命周期与K8S Service 绑定。

```
annotations: 
service.beta.kubernetes.io/alicloud-loadbalancer-spec: slb.s2.small     service.beta.kubernetes.io/alicloud-loadbalancer-address-type: internet     service.beta.kubernetes.io/alicloud-loadbalancer-slb-network-type: vpc     service.beta.kubernetes.io/alicloud-loadbalancer-protocol-port: tcp:9033
```

#### 华为云
1. 支持通过应用编排服务AOS（类似AWS CloudFormation）将应用与其他云资源定义到模板中编排执行。但是这种方式不能方便的关联Service与负载均衡的生命周期。

#### 腾讯云
暂时未知

#### AWS
暂时未知

#### Azure
暂时未知

#### Google Cloud
暂时未知