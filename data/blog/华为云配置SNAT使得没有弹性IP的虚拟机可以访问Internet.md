---
tags: [公有云, 云厂商, 华为云, snat, internet]
date: 2017-10-27
title: 华为云配置SNAT使得没有弹性IP的虚拟机可以访问Internet
---

## 问题场景
一个VPC内有多台虚拟机，其中只有某一台或几台需要绑定弹性IP，允许Internet访问进来；但是所有的虚拟机都要访问Internet。

## 解决方案
给每个虚拟机配置一个弹性IP地址是一个方法，但是资源消耗太大，且虚拟机并不需要被Internet访问进来，不合适。
有另外一种解决方案：SNAT，原理是选择一台有弹性IP的虚拟机，通过IPTables的SNAT能力，把此虚拟机配置为一台路由器，其他没有弹性IP的虚拟机，通过配置路由的方式，将消息包转发到SNAT的虚拟机，再由此虚拟机转发出Internet。
```
iptables -t nat -A POSTROUTING -o eth0 -s 192.168.0.0/24 -j SNAT --to 192.168.0.111
```

[华为云提供了详细的指导书](http://support.huaweicloud.com/usermanual-vpc/zh-cn_topic_0038764344.html)。

## Update 2018
现在华为云有了[NAT网关服务](http://www.huaweicloud.com/product/nat.html)，可以直接在NAT网关服务打开SNAT了，应用程序可更便捷访问Internet。