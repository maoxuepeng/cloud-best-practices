---
date: 2023-6-24
title: SOFABoot开发体验
tags: [SOFABoot, 开发体验]
---

SOFA Boot是蚂蚁开源的一套开发框架，在Spring Boot之上叠加了一些功能，用于快速接入SOFA Stack云服务体系。与SOFA Boot紧密关联的项目还有SOFA RPC，SOFA Registry。本次开发体验是使用SOFA Boot创建项目管理依赖，使用SOFA RPC作为通信协议，使用SOFA Registry作为注册发现服务。

## 用户旅程

### 本地运行

1. 使用 start.spring.io 创建一个spring boot项目
2. 按照 [SOFA Boot快速开始](https://www.sofastack.tech/projects/sofa-boot/quick-start/) 指导引入SOFA Boot
3. 按照 [SOFA RPC快速开始](https://www.sofastack.tech/projects/sofa-rpc/getting-started-with-sofa-boot/)指导引入SOFA RPC

这时候一切顺利，客户端与服务端之间通过本地注册发现方式找到对方。

### 上云

如果要在云上运行，那么本地运行这么一套行不通，需要修改：

1. 引入商业版本依赖 [商业版SOFA Boot（SOFA Stack）开发环境搭建指南](https://help.aliyun.com/document_detail/310983.html)
2. 使用SOFA的商业Maven仓库 [settings.xml](https://gw.alipayobjects.com/os/bmw-prod/ecd6bc11-b67f-4acc-89df-63a6e83e7f8e.xml?spm=a2c4g.133192.0.0.2491778f9Zj64T&file=ecd6bc11-b67f-4acc-89df-63a6e83e7f8e.xml)
3. ```application.properties```文件需要修改为以sofa stack工作空间结尾：```application.properties.{工作空间名称}```，并将sofa stack中间件访问信息填入到此文件。

** 需要注意的是，官方文档的settings.xml并不工作，没有镜像完整的依赖项，需要在settings.xml增加额外仓库才行，添加如下行 **

```xml
        <repository>
            <id>aliyun</id> <!-- Don't change this! -->
             <url>https://maven.aliyun.com/nexus/content/groups/public</url>
            <releases>
                <enabled>true</enabled>
                <checksumPolicy>fail</checksumPolicy>
            </releases>
            <snapshots>
                <enabled>true</enabled>
                <checksumPolicy>warn</checksumPolicy>
            </snapshots>
        </repository>
```

## 商业版与开源版两张皮：一定要区分是商业版还是开源版

SOFA Boot开源项目并不能**平迁**到SOFA Stack商用平台上，需要重新引入SOFA商业依赖库，这个改动还是比较大。

## Demo 代码

[sofa boot rpc server](https://github.com/maoxuepeng/sofabootdemo-server)

[sofa boot rpc client](https://github.com/maoxuepeng/sofabootdemo-client)

## Reference

[开源SOFA Boot项目](https://www.sofastack.tech/projects/sofa-boot/overview/)

[开源SOFA RPC项目](https://www.sofastack.tech/projects/sofa-rpc/overview/)

[商业版SOFA Boot（SOFA Stack）开发环境搭建指南](https://help.aliyun.com/document_detail/310983.html)

[本地开发环境连接SOFA Stack注册中心联调指导](https://help.aliyun.com/document_detail/149866.html)
