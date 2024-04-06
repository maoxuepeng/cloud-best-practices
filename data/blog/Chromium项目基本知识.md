---
date: 2020-9-9
title: Chromium项目基本知识
tags: [Web, Chromium, Chrome]
---

本文介绍Google Chromium 项目的基本知识，澄清 Chromium, Chrome, WebView 等产品关系。

## Chromium 项目中的产品

Google [Chromium 项目](www.chromium.org)包含几个产品，他们之间关系如下。

- Chrome浏览器: 谷歌Chrome浏览器。
- Chromium 开源项目: Chrome浏览器部分开源代码，Chrome浏览器还有一些闭源代码。
- Blink：基于WebKit fork出来的Web渲染引擎（Web渲染包含DOM树解析、图层合成、图形绘制）。
- CEF(Chromium Embed Framework):  基于Chromium基础上封装的浏览器框架，使用此框架可以开发一个浏览器应用
- [Android WebView](https://www.chromium.org/developers/androidwebview): Android 内置的浏览器View，是Google的闭源产品，基于Blink内核。

## Chromium 架构

Chromium 采用多进程架构，每个Tab页对应一个进程，GPU渲染共用一个独立进程，进程之间通过IPC通信。

![](/images/rendering/chromium-arch-1.png)

逻辑架构图如下。

![](/images/rendering/chromium-arch-2.png)

- **WebKit:** Rendering engine shared between Safari, Chromium, and all other
    WebKit-based browsers. The **Port** is a part of WebKit that integrates with
    platform dependent system services such as resource loading and graphics.

- **Glue:** Converts WebKit types to Chromium types. This is our "WebKit
    embedding layer." It is the basis of two browsers, Chromium, and test_shell
    (which allows us to test WebKit).

- **Renderer / Render host:** This is Chromium's "multi-process embedding
    layer." It proxies notifications and commands across the process boundary.

- **WebContents:** A reusable component that is the main class of the Content
    module. It's easily embeddable to allow multiprocess rendering of HTML into
    a view. See the [content module pages](https://www.chromium.org/developers/content-module) for more
    information.

- **Browser:** Represents the browser window, it contains multiple
    WebContentses.

- **Tab Helpers**: Individual objects that can be attached to a WebContents
    (via the WebContentsUserData mixin). The Browser attaches an assortment of
    them to the WebContentses that it holds (one for favicons, one for infobars,
    etc).

## Reference

https://juejin.im/post/5d2c446ee51d45778f076dd3

[androidwebview](https://www.chromium.org/developers/androidwebview)

[](https://www.chromium.org/developers/how-tos/getting-a)

[multi-process-architectureround-the-chrome-source-code](https://www.chromium.org/developers/design-documents/multi-process-architectureround-the-chrome-source-code)

[content module pages](https://www.chromium.org/developers/content-module)
