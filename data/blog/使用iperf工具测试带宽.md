---
date: 2020-3-3
title: 使用iperf工具测试带宽
tags: [iperf]
---

[iperf](https://iperf.fr/iperf-download.php) 是一款带宽测试工具，可以用于测试通信双方的带宽。

## 安装

在官方网站提供了各种版本的可执行文件，包括[Windows](https://iperf.fr/iperf-download.php#windows)、各种Linux发行版、[Android](https://iperf.fr/iperf-download.php#android)、[iOS](https://iperf.fr/iperf-download.php#ios)、[Mac](https://iperf.fr/iperf-download.php#macosppc)、[Docker Image](https://registry.hub.docker.com/u/networkstatic/iperf3/)，直接下载即可使用。

对于Android版本，[Magic iPerf](https://play.google.com/store/apps/details?id=com.nextdoordeveloper.miperf.miperf) 小巧又好用，iperf的参数全部开放，但是是在Google Play上以App方式提供的，大陆访问不了，通过这个链接可以下载：[http://blog.best-practice.cloud/tools/Magic_iPerf_including_iPerf3_v1.0_apkpure.com.apk](http://blog.best-practice.cloud/tools/Magic_iPerf_including_iPerf3_v1.0_apkpure.com.apk)。

## 使用

iperf 使用很简单：

1. ```iperf3 -s -p 7777 -I 1``` 以server方式启动，侦听7777端口，每秒刷新一次输出。
2. ```iperf3 -c 192.168.1.2 -p 7777 -i 1 -b 20M -u -R -t 600``` 以client方式启动，连接服务端，带宽20Mbps，以UDP协议传输，逆向服务端发数据客户端收数据，持续600秒。

详细参考命令行帮助。

## Android 命令行版本

针对使用Android作为服务端场景，希望使用iperf命令行方式启动 iperf Server ，可从 [KnightWhoSayNi/android-iperf](https://github.com/KnightWhoSayNi/android-iperf) 获取如何编译Android二进制的指导，或者直接下载二进制文件。

## Reference

[iperf](https://iperf.fr/iperf-download.php)
[Magic iPerf](https://play.google.com/store/apps/details?id=com.nextdoordeveloper.miperf.miperf)
