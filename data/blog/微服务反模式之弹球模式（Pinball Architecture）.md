---
date: 2023-5-28
title: 微服务反模式之弹球模式（Pinball Architecture）
tags: [微服务]
---

微服务架构在国内已经被广泛采用，由于微服务本身并没有严格定义，只要符合一系列特征，就可以认为是微服务架构。这种定义概念的方式，典型的是面向对象中的[鸭子类型](https://zh.wikipedia.org/wiki/%E9%B8%AD%E5%AD%90%E7%B1%BB%E5%9E%8B)：```当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子。```。

鉴于此，云计算领头羊AWS，主推Lambda服务作为微服务架构应用托管服务。由于Lambda本质是函数托管服务，如果使用Lambda实现微服务架构必然是要求将微服务拆分到函数级别，粒度细到了极致，因此就容易出现所谓的```Lambda弹球```反模式。

### 什么是弹球模式

弹球 Pinball 是一种游戏：```玩家在投币之后(如果弹珠台以营利为目的)，透过侧边的弹簧弹出一颗金属小球，让小球进入矩形框架里任意弹跳，并依据经过的地点得分。当小球往下掉的时候，玩家可以操控最底端的板子将小球打回去、继续弹跳。如果玩家让小球落到最底端，则游戏结束。```

![](/images/pinball-game.jpeg)

没玩过实体游戏机的，都应当玩过windows系统自带的弹球游戏电脑版。

在微服务架构中的弹球模式，是指微服务拆分太细，导致开发人员需要花费大量精力聚焦请求/数据如何在基础设施与业务代码之间流转，就像弹球。

### 弹球模式的例子

前段时间的热点话题[AWS-Prime-Video-从微服务转为单体架构成本降低 90%](https://mp.weixin.qq.com/s?__biz=MzU4OTI0MTM5Mw==&mid=2247483935&idx=1&sn=e702ca63df82f34091f99ae1aec30070&chksm=fdd1cf41caa64657c7837d621d8524c9b78e506dab8405ade15824a690a8a80c689a3a268a53&token=1855851797&lang=zh_CN#rd)，里面的应用程序，就是典型的“弹球模式”。

### 如何避免弹球模式

在微服务架构实施过程中，需避免教条主义，需瞄准微服务架构给组织带来的价值为锚点，来实施微服务架构。微服务最大的价值，业务视角是更短的TTM（Time To Market），技术视角是架构的可扩展性。

可扩展性通过扩展魔方定义，在三个维度的可扩展。

![](/images/AKF_Scale_Cube_Explained.jpg)

- x：容量扩展
- y：分区扩展
- z：组织扩展


## Reference

[Thoughtworks技术雷达28期](https://www.thoughtworks.com/content/dam/thoughtworks/documents/radar/2023/04/tr_technology_radar_vol_28_cn.pdf)

[Is Your Microservices Architecture More Pinball or More McDonald’s?](https://betterprogramming.pub/is-your-microservices-architecture-more-pinball-or-mcdonalds-9d2a79224da7)

[鸭子类型](https://zh.wikipedia.org/wiki/%E9%B8%AD%E5%AD%90%E7%B1%BB%E5%9E%8B)

[AKF SCALE CUBE EXPLAINED – CLOUD SCALABILITY RULES](https://akfpartners.com/growth-blog/scaling-your-systems-in-the-cloud-akf-scale-cube-explained)

