---
date: 2020-4-5
title: Microsoft-XCloud分析
tags: [游戏, 云游戏, CloudGaming, XCloud]
---

本文从产品功能，技术实现，基础设施等方面分析Microsoft Project XCloud。

## 1 Microsoft Project xCloud基本情况

- **2019年9月**: Project xCloud (Preview) 开始Beta测试。
- **2020年9月15日**: Xbox Game Pass Ultimate开始Beta测试，Project xCloud (Preview) 将于2020.9.11关闭。

![](/images/xcloud/xcloud-1.png)

## 2 Project xCloud 技术栈

![](/images/xcloud/xcloud-stack.png)

1. 云游戏技术栈与Xbox Console游戏技术栈一致（**Console Native**），并提供TAK，存量游戏无需修改即可云化。
2. 真实情况是，Console游戏转为云游戏，场景存在差异（输入设备、屏幕大小等），游戏需要针对云化场景适配，因此提供了Cloud
Aware API。

### Streaming Protocol

xCloud的串流技术栈官方未透露，明确的是使用UDP协议，其他信息不明。

### Touch Adaptation Kit

#### 背景问题

Xbox手柄有18个物理按键，存量游戏已经适配好了手柄输入。

![](/images/xcloud/tak-1.png)

当游戏云化后在移动设备运行时，引入了触控输入需求。面临两个问题：

1. Xbox手柄有18个物理按键，不同游戏场景下使用按键组合不同，不能简单粗暴显示到移动设备上，否则屏幕上被按键占满了。

![](/images/xcloud/tak-2.png)

2. 移动设备本机触摸事件处理。

#### 方案

1. TAK：开发者自定义不同游戏场景下的触控输入布局，通过Cloud Aware API控制布局显示/隐藏、随场景切换布局。

![](/images/xcloud/tak-3.png)

2. 支持本机触摸，最终转换为鼠标输入给云端游戏。

### Cloud Aware APIs

游戏通过Cloud Aware API，感知云游戏场景，并做相应的适配处理。

[xgamestreaming.h](https://docs.microsoft.com/zh-cn/gaming/game-streaming/reference/xgamestreaming/xgamestreaming_members)

## Reference

[project-xcloud](https://www.xbox.com/en-US/xbox-game-streaming/project-xcloud)

[xbox-game-pass-cloud-gaming](https://www.xbox.com/en-US/xbox-game-pass/cloud-gaming)

[project-xcloud-everything-we-know-about-microsofts-cloud-streaming-service](https://www.techradar.com/news/project-xcloud-everything-we-know-about-microsofts-cloud-streaming-service)

[heres-a-closer-look-at-the-hardware-behind-project-xcloud](https://www.onmsft.com/news/heres-a-closer-look-at-the-hardware-behind-project-xcloud)

[microsoft_reveals_project_xcloud_the_xbox_cloud_streaming_service](https://www.overclock3d.net/news/software/microsoft_reveals_project_xcloud_the_xbox_cloud_streaming_service/1)
