---
date: 2018-08-21
title: DotNet应用开发技术栈
tags: [.net]
---

.net 应用的市场份额还不小，结合当前的docker技术，很多.net应用也希望能够运行在docker中。
.net 应用运行在docker内有两种选择：
- Docker on Windows
- Docker on Linux

### Docker On Windows
Docker On Windows目前支持的并不是很好，Windows Server 1709已经支持了Docker，但是只是Preview版本，很多特性不完善，不能商用。

### Docker On Linux
Docker On Linux最大的阻碍是.net framework在Linux上的支持。
微软虽然提供了一个[.net core的Linux版本](https://www.microsoft.com/net/download)，但是并没有大规模商用案例。

目前倒是有一些开源项目能够将.net 应用在Linux上run起来。

- [Mono](https://www.mono-project.com/): 微软赞助的.net framework Linux版本
- [Nancy](https://github.com/NancyFx/Nancy): 基于Mono的Web容器