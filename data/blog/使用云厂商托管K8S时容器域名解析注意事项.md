---
date: 2019-1-28
title: 使用云厂商托管K8S时容器域名解析注意事项
tags: [Kubernetes, Pod, Container, DNS]
---

### 云厂商托管 Kubernetes 服务的 Pod 域名解析注意事项
使用云厂家提供托管式Kubernetes，Pod的域名解析参数，通过界面创建Pod的话，可能厂商界面没有开放dnsConfig配置，采用了一些默认值，在使用时候，需要了解清楚厂商提供的默认配置，否则会存在问题。
典型的一个配置是 ndots ，如果你在Pod内访问的域名字符串，**点** 数量在 ndots 阈值范围内，则被认为是Kubernetes集群内部域名，会被追加 **.<namespace>.svc.cluster.local** 后缀，这样会导致每次解析域名时候有2次（IP4/IP6）无效解析，在大规模并发场景下存在性能瓶颈。

云厂商一般将Kubernetes的DNS服务（CoreDNS或SkyDNS）与厂商提供的外部DNS级联了，因此这种问题再功能测试阶段不会被暴露，只在性能测试阶段能暴露出来。


### DNS查找原理与规则
DNS域名解析配置文件 ```/etc/resolv.conf```

```
nameserver 10.247.x.x
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:3
```

参数说明
- nameserver 域名解析服务器
- search 域名的查找后缀规则，查找配置越多，说明域名解析查找匹配次数越多，这里匹配有3个后缀，则查找规则至少6次，因为IPV4,IPV6都要匹配一次
- options 域名解析选项，多个KV值；其中典型的有 ndots ，访问的域名字符串内的点字符数量超过 ndots 值，则认为是完整域名，直接解析

### Kubernetes的dnsConfig配置说明
[Kubernetes官网的dns配置说明](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)

- nameservers：将用作Pod的DNS服务器的IP地址列表。最多可以指定3个IP地址。当Pod dnsPolicy 设置为“ None”时，列表必须至少包含一个IP地址，否则此属性是可选的。列出的服务器将合并到从指定的DNS策略生成的基本名称服务器，并删除重复的地址。
- searches：Pod中主机名查找的DNS搜索域列表。此属性是可选的。指定后，提供的列表将合并到从所选DNS策略生成的基本搜索域名中。删除重复的域名。Kubernetes最多允许6个搜索域。
- options：可选的对象列表，其中每个对象可以具有name 属性（必需）和value属性（可选）。此属性中的内容将合并到从指定的DNS策略生成的选项中。删除重复的条目

#### dnsPolicy域名解析的几种场景应用
对应的容器中的deployment的yaml中的dnsPolicy有三种配置参数ClusterFirst, Default, None
- Default：Pod从运行pod的节点继承名称解析配置。有关 详细信息，请参阅相关讨论
- ClusterFirst：任何与配置的群集域后缀不匹配的DNS查询（例如"www.kubernetes.io"）将转发到从该节点继承的上游名称服务器。群集管理员可能配置了额外的存根域和上游DNS服务器。有关 在这些情况下如何处理DNS查询的详细信息，请参阅相关讨论。
- ClusterFirstWithHostNet：对于使用hostNetwork运行的Pod，您应该明确设置其DNS策略“ ClusterFirstWithHostNet”。
- None：Kubernetes v1.9（Beta in v1.10）中引入的新选项值。它允许Pod忽略Kubernetes环境中的DNS设置。应使用dnsConfigPod规范中的字段提供所有DNS设置。请参阅下面的DNS配置子部分。

##### 1. 场景1-采用自定义DNS

采用自己建的DNS来解析Pods中的应用域名配置，可以参考以下代码配置，此配置在Pod中的DNS可以完全自定义，适用于已经有自己建的DNS,迁移后的应用也不需要去修改相关的配置；

```
apiVersion: v1
kind: Pod
metadata:
  namespace: default
  name: dns-example
spec:
  containers:
    - name: test
      image: nginx
  dnsPolicy: "None"
  dnsConfig:
    nameservers:
      - 1.2.3.4
    searches:
      - ns1.svc.cluster.local
      - my.dns.search.suffix
    options:
      - name: ndots
        value: "2"
      - name: edns0
```

##### 2. 场景2-采用kubernets的CoreDNS

优先使用Kubernetes的DNS服务解析，失败后再使用外部级联的DNS服务解析。

```
apiVersion: v1
kind: Pod
metadata:
  namespace: default
  name: dns-example
spec:
  containers:
    - name: test
      image: nginx
  dnsPolicy: ClusterFirst
```

##### 3. 场景3-采用云公网域名解析

适用于Pods中的域名配置都在公网访问，这样的话Pods中的应用都从外部的DNS中解析对应的域名

```
apiVersion: v1
kind: Pod
metadata:
  namespace: default
  name: dns-example
spec:
  containers:
    - name: test
      image: nginx
  dnsPolicy: Default
```

##### 4. 场景4-采用HostNet的DNS解析

如果在POD中使用hostNetwork:true配置网络，pod中运行的应用程序可以直接看到宿主主机的网络接口，宿主主机所在的局域网上所有网络接口都可以访问到该应用程序

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx
spec:
  template:
    metadata:
      labels:
        app: nginx
    spec:
      hostNetwork: true
      dnsPolicy: ClusterFirstWithHostNet
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

如果不加上dnsPolicy: ClusterFirstWithHostNet, pod默认使用所在宿主主机使用的DNS，这样也会导致容器内不能通过service name 访问k8s集群中其他POD。