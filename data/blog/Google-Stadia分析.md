---
date: 2020-4-3
title: Google-Stadia分析
tags: [游戏, 云游戏, CloudGaming, Stadia]
---

本文从产品功能，技术实现，基础设施等方面分析Google Stadia云游戏。

## 1 Google Stadia 基本情况

### 状态

- 2019-3月 GDC大会发布

- 2019-11月上线，首批上线14个国家：美國、加拿大、英國、法國、德國、意大利、西班牙、荷蘭、比利時、愛爾蘭、丹麥、瑞典、挪威、芬蘭

### 价格

- Stadia Pro \$9.9/月，游戏需单独购买（提供部分免费游戏），分辨率可达4K\@60FPS

- Stadia Base 免费，游戏需单独购买（无免费游戏），分辨率1080P\@60FPS

### 外设与接入方式

- 手柄（Stadia Controller）操控，可插入耳机，带麦克风，振动反馈

- 多端接入：手机: Stadia App(Android, iOS)，PC: Chrome浏览器，TV: Chromecast

### 网络准入条件

- 分辨率720P\@60FPS起，带宽要求10Mbps；最大4K\@60FPS，带宽要求35Mbps；画质可随网速提升而平滑提升

- 提供测速服务(<https://projectstream.google.com/speedtest>)，由合作伙伴MLab提供服务

完善中的功能

- Google Assistant

- Stream Connect(2020.7已灰度）

- State Share

- Crowd play(2020.7已灰度）

### Staida给游戏产业带来新模式

- 玩游戏更有趣：即玩、Staida分享到YouTube、Staida唤起YouTube游戏指南或提示、从YouTube进入游戏

- 游戏分发渠道更宽广：超链接分发，无需绑定Store

- 游戏创作更简单：提供游戏创作辅助工具

![](/images/stadia/stadia-controller.png)

Stadia控制器在顶部具有两个独特按钮：一个用于Google智能助理，另一个用于屏幕捕捉。它们以清晰的白色图标呈现在光滑的黑色表面上。捕捉按钮用于尽可能轻松和无摩擦地与YouTube共享，而助手包含在那里帮助游戏玩家在YouTube上找到指南或提示，而无需离开他们的游戏会话。

## 2 Stadia 商业目标

Stadia的商业目标有两部分：

- 通过云游戏作为切入，尝试做游戏，成为一家游戏公司
- 为YouTube引流，将云游戏流量引入YouTube

第一个目标是长期目标，不确定性较大。第二个目标是短期目标，从当前Stadia产品设计、以及人事调动 [Justin Uberti宣布离开Google Duo项目 现担任Google Stadia首席工程师](https://tech.sina.com.cn/roll/2019-12-06/doc-iihnzhfz4067512.shtml) 都可以明确推导出来。

## 3 Google Stadia 技术栈

![](/images/stadia/stadia-stack.png)

谷歌Stadia技术栈，支撑存量游戏云化、原生云游戏开发两个场景。

- 存量游戏云化：Stadia SDK与游戏云化工具。

- 原生云游戏开发：Project Chimera、GameBus等。

## 4 Stadia SDK：游戏集成SDK实现Stadia游戏特有功能

游戏集成Stadia SDK实现Stadia游戏特有功能：Click to Play、Crowd Play、Crowd Choice、State Share、Assistant。```由于未申请通过Stadia开发者，没能获取到SDK包。```

### Stadia SDK: Click to Play

Stadia Streamer基于WebRTC实现游戏串流，在有浏览器的设备上都可以玩云游戏。

Click to Play: 在观看YouTube视频时候，可以通过一个按钮进入游戏体验游戏

![](/images/stadia/click-to-play-1.png)

Click to Play: 通过链接分享游戏到社交网站(Twitter, Reddit等），其他人点开链接即玩

![](/images/stadia/click-to-play-2.png)

### Stadia SDK: Crowd Play

Crowd Play（类似互动直播） 玩法:观众观看游戏时候可加入游戏，作为主播的队友或对手角色。

![](/images/stadia/crowd-play-1.png)

Crowd Play原理：主播开启Crowd Play，观众点按钮加入游戏，游戏通过Stadia SDK获取新加入的玩家。

![](/images/stadia/crowd-play-2.png)

Crowd Play 游戏集成方法:在游戏包里通过标记位控制是否开启Crowd Play（怎么配置？？）

![](/images/stadia/crowd-play-3.png)

游戏支持Crowd Play，需要先支持Multi Player模式，并使用Stadia SDK从Stadia获取玩家加入、离开的事件

![](/images/stadia/crowd-play-4.png)

### Stadia SDK: Crowd Choice

使用方法:观众通过投票影响游戏内容，包括选择主播使用的武器、选择游戏中的NPC（非玩家角色）等。

![](/images/stadia/crowd-choice-1.png)


4种投票类型：

- Multi Choice 多选

- Tug-of-War 两个选项选一个

- Crowd Boost 一个选项

- Chat 可定制对话内容，20+选项


![](/images/stadia/crowd-choice-2.png)

Crowd Choice 游戏集成方法

![](/images/stadia/crowd-choice-3.png)

### Stadia SDK: State Share

使用方法: 玩家在YouTube上找到玩家分享的游戏，可以进入到分享时的状态继续玩。

![](/images/stadia/state-share-1.png)

原理:玩家上传带状态元数据的视频到YouTube、或使用Stadia直播游戏；观众找到视频可进入游戏；游戏读取视频元数据，进入指定状态（如关卡）。

![](/images/stadia/state-share-2.png)

游戏集成方法

![](/images/stadia/state-share-3.png)

![](/images/stadia/state-share-4.png)

### Stadia SDK: Assistant

使用方法：手柄按下Assistant按钮，说出问题（如怎样击败这个Boss？），Assistant会搜寻出一个教学视频

![](/images/stadia/assistant-1.png)

Assistant 工作原理：游戏通过Stadia SDK上报游戏场景对应的标签，给视频流打标签

![](/images/stadia/assistant-2.png)

Assistant 游戏集成方法

![](/images/stadia/assistant-3.png)

### Stadia Streamer：构建在WebRTC之上

WebRTC架构现状

![](/images/stadia/stadia-streamer-1.png)

Stadia 场景下WebRTC架构

![](/images/stadia/stadia-streamer-2.png)

#### Stadia场景下WebRTC修改点（对外接口未发生变化）

1. 音视频采集由摄像头、麦克风变为GPU、声卡
2. Codec采用VP9编码
3. 增加Rate Adaptation，协同编码与传输
4. 支持C/S模式
5. 传输协议使用QUIC替换

#### 为什么谷歌选择WebRTC

- 游戏免安装，通过Chrome即可玩，支撑云游戏跨终端推广目标
- WebRTC定位是超低时延(half-second)实时音视频流传输协议
- 增强WebRTC生态

#### 为什么需要WebRTC Over QUIC

- 简化WebRTC协议栈
- QUIC优势：零RTT建链、改进的拥塞控制、多路复用、连接迁移、前向冗余纠错

### Stadia GameBus：原生云游戏引擎

GameBus两个功能：分布式游戏引擎；Stadia多实例级联，实现大型多人在线游戏。

GameBus是一个分布式架构的游戏引擎，分布式架构的优势是可以容易横向扩展，如物理系统算力增强、使用云端强大的AI算力实现更有趣的NPC角色。

![](/images/stadia/gamebus-1.png)

通过GameBus级联多个Stadia实例，实现大型多人在线游戏。

![](/images/stadia/gamebus-2.png)

### Stadia Playability Toolkit：云游戏测试、调优工具集

Stadia 为游戏云化提供一系列工具

- Chrome Test Client(Network Simulator, Frame Capture)
- Stream Profile API
- Media Stream API
- Stream Capabilities API
- Frame Token API
- Video Diff
- Smoothness View

## 5 Google Stadia 网络带宽要求与串流时延

### 玩Stadia云游戏网络带宽要求

720P@60FPS带宽10Mbps，1080P@60FPS 20Mbps带宽，4K分辨率则要求35Mbps带宽。如果带宽达不到要求，则可能会出现卡顿或花屏（Stadia在编码与传输自适应上做了不少工作以保障流畅度）。

![Google官方给出的带宽需求表](/images/stadia/stadia-latency-1.png)

### 第三方评测的时延数据

通过Stadia玩游戏，与本地游戏比较，串流时延平均在40\~90ms之间。

![](/images/stadia/stadia-latency-2.png)

[数据来源](https://www.pcgamer.com/heres-how-stadias-input-lag-compares-to-native-pc-gaming/)

- Stadia PC对应键盘鼠标输入
- Stadia TV对应手柄输入
- 第二个游戏在TV上延迟特别高，作者也不清楚原因，延迟高，但是不卡顿。

![](/images/stadia/stadia-latency-3.png)

[数据来源](https://www.eurogamer.net/articles/digitalfoundry-2019-stadia-tech-review)

### Stadia Streaming(串流)技术栈

Google Stadia通过全球骨干网就近接入、定制云游戏主机、优化编解码器、优化传输协议，实现游戏画质4K\@60FPS，操作响应时延150ms的目标。

编解码与传输协议

- VP9编码格式: 更高的压缩比
- 定制AMD GPU，4K\@60FPS编码（后续支持8K\@120FPS）
- WebRTC Over QUIC：低时延传输协议，BBR拥塞控制
- 编码与传输协同：根据网络质量实时调整编码参数(码率、分辨率、帧率等），保障游戏流畅体验

游戏主机

- CPU: X86处理器，2.7GHz; Mem: 16G; GPU: AMD V340 16GB;
- 图形API：Vulkan
- Linux操作系统

全球部署

- Edge POP点就近接入
- 不确定Stadia服务器最终是否部署到Edge POP

## Reference

### 1

[Stadia使用体验：这东西没做完](https://www.gcores.com/articles/117365)

[Google Stadia：与 YouTube 集成，低硬件门槛收割「大众」流量](https://www.ifanr.com/1301307)

[Google Stadia是YouTube的未来，而不是游戏的](http://www.cniteyes.com/archives/34518)

[Google 新设 Stadia 游戏工作室，由前圣莫尼卡工作室负责人领导](https://cn.technode.com/post/2020-03-05/google-stadia-playa-vista-studio-sony-santa-monica-studio-shannon-studstill/)

[Google Stadia 首发评测汇总：云游戏不是梦，但谷歌会让你望而却步](https://www.gcores.com/articles/117284)

### 2

[How YouTube Paved the Way for Google's Stadia Cloud Gaming Service](https://spectrum.ieee.org/tech-talk/telecom/internet/how-the-youtube-era-made-cloud-gaming-possible)

### 3

[Project-Chimera-Googles-Next-Big-Thing](https://stadiasource.com/article/562/Project-Chimera-Googles-Next-Big-Thing)

### 4

[Low Latency Video Streaming](https://datatracker.ietf.org/meeting/105/materials/slides-105-mops-low-latency-video-streaming-00.pdf)

[how-the-youtube-era-made-cloud-gaming-possible](https://spectrum.ieee.org/tech-talk/telecom/internet/how-the-youtube-era-made-cloud-gaming-possible)

[google-stadia-engineering](https://9to5google.com/2019/12/05/google-stadia-engineering/)

[Who needs QUIC in WebRTC anyway?](https://bloggeek.me/who-needs-quic-in-webrtc/)

[在基于WebRTC的实时流系统中使用QUIC](https://www.w3.org/2019/03/23-chinese-web-jianjun-intel.pdf)

[High Performance Browser Networking WebRTC](https://hpbn.co/webrtc/)

[webrtc-peer-to-peer-imx6](https://www.iwavesystems.com/webrtc-peer-to-peer-imx6)
