---
date: 2019-1-7
title: SDK与ServiceMesh微服务模式融合趋势
tags: [微服务, SDK, ServiceMesh, Dubbo, SpringCloud, ServiceComb]
---

### 现状与挑战
自从Netflix在AWS上实践PaaS平台的经验被推广后，同时借助开源出来的SpringCloud微服务框架，微服务领域一直是Java语言+Java SDK模式占统治地位。

经过这么些年的发展，微服务架构思想已经深入影响了各行各业架构师的架构选型，在技术架构选型的备选中，微服务通常是第一个考虑的。SDK模式的微服务架构已经被广泛应用于生产系统。


随着这两年ServiceMesh技术的火热，部分新技术敏感的用户提出如何从当前SDK模式演进到ServiceMesh模式，换句话说，如何做到在同一个微服务架构系统中，让SDK模式与ServiceMesh模式两种应用融合互通？因为演进的过程可能是一个长期过程，也有可能是最终态。

ServiceMesh模式的微服务架构是非侵入、语言无关的，而多语言的场景比较常见，对于稍大型的公司，业务类别比较多，不同业务类别可能需要不同语言。

### Dubbo 与 ServiceMesh 融合的思考与方案
[Dubbo Mesh ｜ Service Mesh的实践与探索](http://dubbo.apache.org/zh-cn/blog/dubbo-mesh-service-mesh-exploring.html)
[Dubbo在Service Mesh下的思考和方案](http://dubbo.apache.org/zh-cn/blog/dubbo-mesh-in-thinking.html)

上面两篇文章提出了Dubbo微服务框架与Istio融合的构想：
1. Dubbo应用在构建阶段自动生成其deployment和service的声明文件。这个主要是解决Dubbo与kubernetes的服务映射。
2. Dubbo地址注册针对kubernetes的扩展实现，通过Kubernetes的APIServer来拉取并监听某个服务的podIP。这样，在kubernetes集群内，Dubbo服务就能在其podID的虚拟网络内实现服务发现。

### ServiceComb 的 ServiceMesh 解决方案
ServiceComb是华为贡献给Apache基金会的开源微服务框架，ServiceComb目前已经具备了ServiceMesh解决方案，可以实现SDK模式与Mesher模式混合使用。
不过这种方式，微服务的注册中心，与K8S的服务发布还是隔离的，从方案完备性上看还有一些不足。期待ServiceComb的进一步完善。