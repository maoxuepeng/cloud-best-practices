---
date: 2019-1-16
title: 在容器中获取Pod或Container的属性
tags: [Kubernetes, Environment, Pod, Container]
---

### 在容器内获取Pod或Container属性
容器内的应用程序有获取容器定义元数据的需求，如容器名称，容器IP地址，容器资源配额等，这些信息在容器调用系统K8S中都存在。在早期的K8S中是不暴露的应用使用的，随着客户诉求的增多，K8S开放了这些信息给应用程序。
[通过环境变量将Pod信息呈现给容器](https://kubernetes.io/zh/docs/tasks/inject-data-application/environment-variable-expose-pod-information/)