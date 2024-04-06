---
date: 2019-8-166
title: HTTP2协议
tags: [HTTP/2]
---

HTTP/2 协议相比较 HTTP1 有诸多优势，越来越多应用程序使用HTTP2协议。

## HTTP2 与 HTTP1 区别

| | HTTP1 | HTTP2 | 对比说明 |
| --- | --- | --- |
| 消息编码格式 | 文本 | 二进制 | HTTP2 的二进制编码格式效率更高，但是问题定位更麻烦，需借助解码工具查看头或body |
| 消息压缩 | ```gzip encoding``` 压缩 ```body``` | ```gzip encoding``` 压缩 ```body```; 默认使用 ```HPACK``` 压缩头 | |
| HTTP请求/连接关系 | 单域多连接，每个连接同一时刻只允许一个请求 | 单域单连接，每个连接允许多请求并发(抽象为Stream) | HTTP2连接数量更少，特别是在TLS建连接耗时情况下效率提升明显；HTTP2 的多 Stream 并发并未解决 TCP 层面 HOL 问题，TCP顺序传包机制，如果前面包丢失重传，后面的包会被阻塞 |
| Server Push | 浏览器加载页面渲染后，再根据页面内定义的CSS/JS路径去请求资源 | 服务器端主动推送其猜测(??)的客户端需要缓存的内容 | |
| 消息优先级 | 协议本身没有 | 适合混杂页面，JS/CSS会被优先请求 | |

## 体验HTTP/2接口

### 使用 curl 访问 HTTP/2 接口

**curl** 工具在 **7.43.0** 版本支持 ```http2``` 协议，在请求时候指定 ```--http2``` 选项，```curl``` 客户端以 ```http2``` 协议发起请求，如果服务器支持 ```http2``` ，则在消息头里返回的协议版本为 ```HTTP2```。

```bash
curl -v --http2 -I https://1906714720.rsc.cdn77.org/http2/http2.html

*   Trying 185.180.13.17...
* TCP_NODELAY set
* Connected to 1906714720.rsc.cdn77.org (185.180.13.17) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
  CApath: /etc/ssl/certs
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Client hello (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server accepted to use h2
* Server certificate:
*  subject: C=GB; L=London; O=DataCamp Limited; CN=rsc.cdn77.org
*  start date: Sep 13 00:00:00 2019 GMT
title: HTTP2协议
*  expire date: Jun  9 12:00:00 2020 GMT
title: HTTP2协议
*  subjectAltName: host "1906714720.rsc.cdn77.org" matched cert's "*.rsc.cdn77.org"
*  issuer: C=US; O=DigiCert Inc; CN=DigiCert SHA2 Secure Server CA
*  SSL certificate verify ok.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x7fffdb276b70)
> HEAD /http2/http2.html HTTP/2
> Host: 1906714720.rsc.cdn77.org
> User-Agent: curl/7.58.0
> Accept: */*
>
* Connection state changed (MAX_CONCURRENT_STREAMS updated)!
< HTTP/2 200
HTTP/2 200
< date: Fri, 11 Oct 2019 02:42:22 GMT
title: HTTP2协议
date: Fri, 11 Oct 2019 02:42:22 GMT
title: HTTP2协议
< content-type: text/html
content-type: text/html
< content-length: 17857
content-length: 17857
< etag: "570b88dc-45c1"
etag: "570b88dc-45c1"
< cache-control: no-cache
cache-control: no-cache
< access-control-allow-origin: *
access-control-allow-origin: *
< server: CDN77-Turbo
server: CDN77-Turbo
< x-cache: HIT
x-cache: HIT
< x-age: 252401
x-age: 252401
< accept-ranges: bytes
accept-ranges: bytes

<
* Connection #0 to host 1906714720.rsc.cdn77.org left intact
```

在 ```curl``` 命令返回消息中，有一行 ```* ALPN, server accepted to use h2``` ，这个输出的意思是使用 [ALPN: Application-Layer Protocal Negotiation](https://en.wikipedia.org/wiki/Application-Layer_Protocol_Negotiation) 协商协议，协商到了 HTTP/2 协议。这个协商协议是为了客户端与服务端兼容而设计的，下面介绍HTTP协议协商。

## 协议协商与兼容性

### HTTP/1.1 协议协商

为了协议更新时候方便向前兼容，HTTP/1.1引入了协议协商机制，这个机制在 RFC7230 的 [Upgrade](https://httpwg.org/specs/rfc7230.html#header.upgrade)章节描述，流程为：

- 客户端发起协议选择请求，在请求 Header 中包含协商的协议列表

```html
Connection: upgrade
Upgrade: protocol-name[/protocol-version]
```

如

```html
GET /hello.txt HTTP/1.1
Host: www.example.com
Connection: upgrade
Upgrade: HTTP/2.0, SHTTP/1.3, IRC/6.9, RTA/x11
```

- 服务端如果接受列表中的协议，则返回一个选择的协议，返回码为 101

```html
HTTP/1.1 101 Switching Protocols
Connection: upgrade
Upgrade: protocol-name[/protocol-version]

[... data defined by new protocol ...]
```

如

```html
HTTP/1.1 101 Switching Protocols
Connection: upgrade
Upgrade: HTTP/2.0

[... data stream switches to HTTP/2.0 with an appropriate response
(as defined by new protocol) to the "GET /hello.txt" request ...]
```

- 服务端还可以选择忽略协议协商，按照服务端协议返回数据

```html
HTTP/1.0 200

[... data stream switches to HTTP/2.0 with an appropriate response
(as defined by new protocol) to the "GET /hello.txt" request ...]
```

### HTTP/2 协议协商

HTTP/2 最初由谷歌的 SPDY 发展而来，SPDY 协议中包含了一个名为 NPN(Next Protocal Negotiation) 的SSL握手扩展。随着 SPDY 升级为 HTTP/2 ，NPN 也升级为 ALPN 。

ALPN 的协商流程为：

- 客户端在SSL握手的 ClientHello 请求中，通过ALPN扩展，发送要协商的协议列表
- 服务端在SSL握手的 ServerHello 请求中，通过ALPN扩展，返回选择的协议列表

## Reference

[[译]使用HTTP/2提升性能的7个建议](https://www.w3ctech.com/topic/1563)

[HTTP/2 Frequently Asked Questions](https://http2.github.io/faq/#is-it-http20-or-http2)

[HTTP/2 TECHNOLOGY DEMO](http://www.http2demo.io/)

[为什么我们应该尽快支持 ALPN？](https://imququ.com/post/enable-alpn-asap.html)

[谈谈 HTTP/2 的协议协商机制](https://imququ.com/post/protocol-negotiation-in-http2.html)
