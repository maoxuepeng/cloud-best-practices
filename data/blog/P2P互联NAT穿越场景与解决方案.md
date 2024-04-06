---
date: 2020-9-7
title: P2P互联NAT穿越场景与解决方案
tags: [P2P, NAT]
---

## 4种 NAT 类型

完全圆锥型
IP限制圆锥型
IP+Port限制圆锥型
对称型

只有完全圆锥型不需要先向对方发送信息，对方就能发来信息。如果两边都不是完全圆锥型，都无法先到达对方，则P2P无法建立。

## NAT 穿越协议

STUN   (Simple Traversal of UDP over Nats) 是一种网络协议。
            建立P2P链接。

Turn   又称SPAN(Simple Protocol for Augmenting)方式。
            建立中继链接。

ICE     不是一种协议，是一个框架，综合STUN和TURN的产物

## Reference

https://github.com/coturn/coturn

http://www.creytiv.com/restund.html
