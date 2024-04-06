---
date: 2019-10-10
title: SSL-Handshake-Session-Resumption
tags: [SSL, Session-Resumption, SessionId, SessionTicket]
---

SSL(HTTPS) 已是当下互联网服务提供者必备选项，SSL给网络带来安全同时也引入性能损耗，性能损耗表现在首次连接握手耗时、通信过程消息加解密耗时耗力。本文介绍SSL握手耗时以及Session复用技术。

## Session 复用 (Session Resumption) 的两种机制：SessionId 与 SessionTicket

SSL协议每次新建连接时都需要经过一次SSL握手流程，SSL握手需要至少2个RTT，加上TCP建连接的一个RTT，总共需要至少3个RTT，对于频繁建链以及网络条件较差（特别是移动网络）场景，这个时间会难以接受（移动网络场景下SSL握手耗时通常需要100~200ms，网络条件较差的场景下可能翻倍）。针对这个问题的一个解决方案是SSL Session复用。
SSL Session复用有SessionId复用与SessionTicket复用两种实现方式。

### SessionId 复用

SessionId 复用机制是服务端缓存 SSL Session ，客户端在 SSL 握手消息中带上 SessionId，服务端根据客户端发送的 SessionId 到缓存中查找，如果找到了则复用此 Session ，否则重新握手。

### SessionTicket 复用

SessionTicket 复用机制是客户端缓存 SessionTicket 这么一个结构，在握手消息(ClientHello) 中发送给服务端，服务端通过私钥验证 SessionTicket 合法性，如果验证通过则复用此Sesion，否则重新握手。

### 两种机制比较

SessionTicket 机制相比较于 SessionId 要更优，主要表现为：

- SessionId 要求服务端缓存SessionId，当客户端数量很多的时候，服务端需要分配大量内存来存储Session
- SessionId 缓存在服务端，通常需要搭建分布式缓存中间件(如Redis或Memcache)来存储，数据同步时延不可避免
- SessionTicket存储在客户端，服务端只负责校验不存储，简化了服务端的逻辑与资源消耗

因此下面内容重点介绍 SessionTicket 机制。

## RFC5077 定义 SSL 握手协议中如何使用 SessionTicket 机制复用 Session

### 数据结构定义

#### NewSessionTicket 结构

```c
struct {
    HandshakeType msg_type;
    uint24 length;
    select (HandshakeType) {
        case hello_request:       HelloRequest;
        case client_hello:        ClientHello;
        case server_hello:        ServerHello;
        case certificate:         Certificate;
        case server_key_exchange: ServerKeyExchange;
        case certificate_request: CertificateRequest;
        case server_hello_done:   ServerHelloDone;
        case certificate_verify:  CertificateVerify;
        case client_key_exchange: ClientKeyExchange;
        case finished:            Finished;
        case session_ticket:      NewSessionTicket; /* NEW */
    } body;
} Handshake;


struct {
    uint32 ticket_lifetime_hint;
    opaque ticket<0..2^16-1>;
} NewSessionTicket;

```

```NewSessionTicket``` 结构体包含Ticket超时时间与ticket内容两个字段

- ticket_lifetime_hint: ticket 超时时间，单位为秒；客户端应当在废弃超时的SessionTicket，不再传递给服务端。
- ticket: SessionTicket 结构

#### SessionTicket 结构

```c
struct {
    opaque key_name[16];
    opaque iv[16];
    opaque encrypted_state<0..2^16-1>;
    opaque mac[32];
} ticket;
```

- key_name: 用来加密```StatePlaintext```的AES Key名称 (HMAC-SHA-256的Key？是哪個？共用？)
- encrypted_state: AES-128-CBC 加密的 SSLSession 状态结构体 (用什么Key加密的？SSL握手协商后的AES KEY？)
- iv: 加密 ```encrypted_state``` 的初始向量
- mac: HMAC-SHA-256(key_name(16字节), iv(16字节), len(encrypted_state)(2字节), encrypted_state)

#### StatePlaintext(encrypted_state明文)结构

```c
struct {
    ProtocolVersion protocol_version;
    CipherSuite cipher_suite;
    CompressionMethod compression_method;
    opaque master_secret[48];
    ClientIdentity client_identity;
    uint32 timestamp;
} StatePlaintext;

enum {
    anonymous(0),
    certificate_based(1),
    psk(2)
} ClientAuthenticationType;

struct {
    ClientAuthenticationType client_authentication_type;
    select (ClientAuthenticationType) {
        case anonymous: struct {};
        case certificate_based:
            ASN.1Cert certificate_list<0..2^24-1>;
        case psk:
            opaque psk_identity<0..2^16-1>;   /* from [RFC4279] */

    }
} ClientIdentity;
```

- protocol_version: SSL握手协商好的协议版本，如 ```TLS1.2``
- cipher_suite: SSL握手协商好的对称加密算法，如 ```ECDHE-ECDSA-AES128-GCM-SHA256```
- compression_method: 应用消息发送前压缩算法
- master_secret: SSL握手结果中的 master_secret，客户端与服务端保持同一个 master_secret ，其他的Key(AES加密密钥/HMAC密钥都基于此Key计算得到）；参考 [rfc5246](https://tools.ietf.org/html/rfc5246), [pre-master secret&master secret](https://crypto.stackexchange.com/questions/27131/differences-between-the-terms-pre-master-secret-master-secret-private-key)

```html 
master_secret = PRF(pre_master_secret, "master secret",
                    ClientHello.random + ServerHello.random)
                    [0..47];
```

- client_identity:
  - anonymous(客户端不认证服务端证书), 值为空结构体
  - certificate_list(客户端通过证书认证服务端)，值为服务端证书(链)
  - psk_identity(客户端通过 [PSK(Pre-Shared Key)](https://tools.ietf.org/html/rfc4279)认证服务端)，值为 PSK 结构
- timestamp: 生成 SessionTicket 的时间戳；服务端根据时间戳与超时时间判定 SessionTicekt 是否超时。

### 首次握手时候，服务端在握手完成的消息中带上 NewSessionTicket ，客户端缓存 SessionTicket

```html
         Client                                               Server

         ClientHello
        (empty SessionTicket extension)-------->
                                                         ServerHello
                                     (empty SessionTicket extension)
                                                        Certificate*
                                                  ServerKeyExchange*
                                                 CertificateRequest*
                                      <--------      ServerHelloDone
         Certificate*
         ClientKeyExchange
         CertificateVerify*
         [ChangeCipherSpec]
         Finished                     -------->
                                                    NewSessionTicket
                                                  [ChangeCipherSpec]
                                      <--------             Finished
         Application Data             <------->     Application Data
```

### 客户端要复用Session时候，在 ClientHello 消息中带上 SessionTicket ，服务端可以选择复用此Session，可以选择不复用（重新生成）

```html
         Client                                                Server
         ClientHello
         (SessionTicket extension)      -------->
                                                          ServerHello
                                      (empty SessionTicket extension)
                                                     NewSessionTicket
                                                   [ChangeCipherSpec]
                                       <--------             Finished
         [ChangeCipherSpec]
         Finished                      -------->
         Application Data              <------->     Application Data
```

#### 服务端选择复用Session

此时服务端在 ServerHello 消息中返回 NewSessionTicket 。客户端应更新 SessionTicket 。

```html
         Client                                                Server
         ClientHello
         (SessionTicket extension)      -------->
                                                          ServerHello
                                      (empty SessionTicket extension)
                                                     NewSessionTicket
                                                   [ChangeCipherSpec]
                                       <--------             Finished
         [ChangeCipherSpec]
         Finished                      -------->
         Application Data              <------->     Application Data
```

#### 服务端选择不复用，走一次完整握手流程

```html
         Client                                               Server

         ClientHello
         (SessionTicket extension) -------->
                                                         ServerHello
                                     (empty SessionTicket extension)
                                                        Certificate*
                                                  ServerKeyExchange*
                                                 CertificateRequest*
                                  <--------          ServerHelloDone
         Certificate*
         ClientKeyExchange
         CertificateVerify*
         [ChangeCipherSpec]
         Finished                 -------->
                                                    NewSessionTicket
                                                  [ChangeCipherSpec]
                                  <--------                 Finished
         Application Data         <------->         Application Data
```

## SessionTicket 生命周期

SessionTicket 的生命周期，包含时间维度，空间维度。

- 时间维度：超时控制
- 空间维度：缓存的存储空间控制

### SessionTicket 超时控制

SessionTicket 超时由服务端控制，在 NewSessionTicket 结构中的 ```ticket_lifetime_hint``` 属性是超时时间，单位为秒。SessionTicket超时时间可以超过24小时。

```ticket_lifetime_hint``` 字段值可以是0，表示服务端未指定超时时间，这种情况下客户端可以自己决定什么时候废弃缓存的Session。

客户端如果使用OpenSSL库，OpenSSL默认的Session缓存时间是300秒。[OpenSSL::SSL_CTX_set_timeout](https://www.openssl.org/docs/manmaster/man3/SSL_CTX_set_timeout.html), [OpenSSL::SSL_get_default_timeout](https://www.openssl.org/docs/manmaster/man3/SSL_get_default_timeout.html)

```html
The default value for session timeout is decided on a per protocol basis, see SSL_get_default_timeout(3). All currently supported protocols have the same default timeout value of <strong>300</strong> seconds.
```

客户端应当在超时之后删除Session缓存。

### SessionTicket 存储空间控制

SessionTicket 存储在内存还是文件？允许最大空间是多大？超过最大存储空间后的清理策略？

上面几个问题，依赖客户端的SSL实现，在Android环境下的OkHttp的行为如下:

- SessionTicket 复用默认启用
- SessionTicket 默认存储在内存
- SessionTicket 支持使用外部持久化存储，要实现此功能，可以扩展实现一个```SSLSocketFactory```类，参考 []() 实现；这个实现中，可支持App定义一个持久化目录，SessionTicket会以文件形式存储到目录下，最大12个，超过之后按照 LRU 算法删除一个

## 安全性

引入SessionTicket之后，可能引入安全隐患的点有:

- SSL 握手流程简化可能引入安全隐患
- SessionTicket客户端存储，泄漏后可能引入安全隐患
- SessionTicket服务端存储，加上验证SessionTicket的Key，泄漏后可能引入安全隐患

复用的流程与SessionTicket敏感泄漏两方面中的安全性。

### SSL 握手流程简化引入的安全隐患: 中间人攻击？

客户端在ClientHello消息中带上SessionTicket，服务端接收到客户端发送过来的SessionTicket之后，使用服务端的SessionTicket加密Key验证SessionTicket：

- 如果验证失败，可能不是自己签发的，或者服务端已经更新了SessionTicket验证Key（从安全角度建议服务端周期更新Key），那么走一次完整的SSL握手流程。这种情况没有引入额外安全隐患。
- 如果验证成功，服务端回复ServerHello消息中回复ChangeCipherSpec消息，并回复Finished，握手结束。

验证成功的场景下，客户端不会执行认证服务端证书步骤，如果此时服务端被伪造了，会发生中间人攻击(??待验证??)，客户端连接到了一个伪冒的服务端。

### SessionTicket客户端存储泄漏后可能的安全隐患: 较轻

如果SessionTicket泄漏后，攻击者持有SessionTicket以简化流程与服务端快速建立SSL连接，此处并未额外增加安全隐患，完整握手流程下客户端也可以与服务端连接SSL连接。

### SessionTicket服务端存储泄漏后可能的安全隐患: 连接被侦听

如果服务端用来解密SessionTicket的Key(AES-128算法)泄漏了，会导致SessionTicket内容别解密，攻击者通过网络抓包截流之后，可以解密通道的内容。

## 测试服务端是否支持SessionTicket复用

使用 OpenSSL 命令行工具可以测试服务端是否支持 SessionTicket 复用。当前绝大多数服务接口都支持SessionTicket复用了，因为 Nginx 版本已经支持SessionTicket复用，只需要打开即可，而客户端库/浏览器也都支持SessionTicket复用。

我们测试 ```page.aliyun.com``` 这个接口 ```openssl s_client -connect page.aliyun.com:443 -reconnect``` 。

```html
...

SSL handshake has read 6464 bytes and written 302 bytes
Verification: OK
---
New, TLSv1.2, Cipher is ECDHE-ECDSA-AES128-GCM-SHA256
Server public key is 256 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-ECDSA-AES128-GCM-SHA256
    Session-ID: 6E274C5517BFB2892BC795BC5A588C16335917FA14DBF8670FF2F8F702ABC195
    Session-ID-ctx:
    Master-Key: 7B841AAB20586BFD1DC439C54F7297619FB654D203A82E4CA7FDC0E9887FC0173A872EDE8A176B446069174E35B92A99
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 28800 (seconds)
    TLS session ticket:
    0000 - b6 93 10 8b ff a7 0b 8a-45 24 ff f1 bb 48 00 c7   ........E$...H..
    0010 - 58 8c c6 12 05 b5 19 d5-25 73 9b d8 74 3d a9 a3   X.......%s..t=..
    0020 - ab 51 d3 0d 98 3a fd 23-65 b5 36 58 35 0f c8 46   .Q...:.#e.6X5..F
    0030 - 50 b8 99 01 26 39 80 25-6f a1 f6 6f 16 9f 4d a3   P...&9.%o..o..M.
    0040 - d8 cd 99 b2 1e fe 7c 69-87 c3 70 dc 27 8f cb 15   ......|i..p.'...
    0050 - 91 30 b3 61 91 0f bc f3-69 8b 43 e8 1f e4 4a 24   .0.a....i.C...J$
    0060 - 2a d9 10 e8 9f 2a 7a 63-86 4e 58 07 4e 6c 31 e2   *....*zc.NX.Nl1.
    0070 - 0e d9 dc 1a f4 f8 77 be-fb 66 33 7c cc fb 72 73   ......w..f3|..rs
    0080 - de c7 95 bb b5 1a 7a 7c-92 56 12 04 cd 87 e5 bf   ......z|.V......
    0090 - 7a 2c 95 60 98 a6 9d 5a-50 b9 37 1c 94 38 20 a3   z,.`...ZP.7..8 .
    00a0 - 55 c7 6a 2d a1 d1 7d e1-f5 b6 09 e2 38 87 03 ff   U.j-..}.....8...

    Start Time: 1570930128
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
    Extended master secret: no
---
drop connection and then reconnect
CONNECTED(00000003)
Verification: OK
---
Reused, TLSv1.2, Cipher is ECDHE-ECDSA-AES128-GCM-SHA256
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-ECDSA-AES128-GCM-SHA256
    Session-ID: 6E274C5517BFB2892BC795BC5A588C16335917FA14DBF8670FF2F8F702ABC195
    Session-ID-ctx:
    Master-Key: 7B841AAB20586BFD1DC439C54F7297619FB654D203A82E4CA7FDC0E9887FC0173A872EDE8A176B446069174E35B92A99
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 28800 (seconds)
    TLS session ticket:
    0000 - b6 93 10 8b ff a7 0b 8a-45 24 ff f1 bb 48 00 c7   ........E$...H..
    0010 - 58 8c c6 12 05 b5 19 d5-25 73 9b d8 74 3d a9 a3   X.......%s..t=..
    0020 - ab 51 d3 0d 98 3a fd 23-65 b5 36 58 35 0f c8 46   .Q...:.#e.6X5..F
    0030 - 50 b8 99 01 26 39 80 25-6f a1 f6 6f 16 9f 4d a3   P...&9.%o..o..M.
    0040 - d8 cd 99 b2 1e fe 7c 69-87 c3 70 dc 27 8f cb 15   ......|i..p.'...
    0050 - 91 30 b3 61 91 0f bc f3-69 8b 43 e8 1f e4 4a 24   .0.a....i.C...J$
    0060 - 2a d9 10 e8 9f 2a 7a 63-86 4e 58 07 4e 6c 31 e2   *....*zc.NX.Nl1.
    0070 - 0e d9 dc 1a f4 f8 77 be-fb 66 33 7c cc fb 72 73   ......w..f3|..rs
    0080 - de c7 95 bb b5 1a 7a 7c-92 56 12 04 cd 87 e5 bf   ......z|.V......
    0090 - 7a 2c 95 60 98 a6 9d 5a-50 b9 37 1c 94 38 20 a3   z,.`...ZP.7..8 .
    00a0 - 55 c7 6a 2d a1 d1 7d e1-f5 b6 09 e2 38 87 03 ff   U.j-..}.....8...

    Start Time: 1570930128
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
    Extended master secret: no
---
```

从上面的输出，可以清晰看出，首次连接时候执行了一次完整的SSL握手，并协商了新的Session：

```html
...

SSL handshake has read 6464 bytes and written 302 bytes
Verification: OK
---
New, TLSv1.2, Cipher is ECDHE-ECDSA-AES128-GCM-SHA256

...
```

重连的时候，复用了上次的Session：

```html
...

drop connection and then reconnect
CONNECTED(00000003)
Verification: OK
---
Reused, TLSv1.2, Cipher is ECDHE-ECDSA-AES128-GCM-SHA256

...
```

## Reference

[TLS Session Resumption: Full-speed and Secure](https://blog.cloudflare.com/tls-session-resumption-full-speed-and-secure/)

[Speeding up HTTPS with session resumption](https://calendar.perfplanet.com/2014/speeding-up-https-with-session-resumption/)

[Java Secure Socket Extension (JSSE) Reference Guide](https://docs.oracle.com/javase/9/security/java-secure-socket-extension-jsse-reference-guide.htm#JSSEC-GUID-DC583ED6-06AD-435C-BC9C-763F4642B2B3)

[Add TLS support for RFC 5077 Session Ticket](https://bugs.openjdk.java.net/browse/JDK-8134497)

[What's new in NSS 3.12.\* - Transport Layer Security (TLS) Session Resumption without Server-Side State](https://blogs.oracle.com/meena/whats-new-in-nss-312-transport-layer-security-tls-session-resumption-without-server-side-state)

[How should I check if SSL session resumption is working or not?](https://serverfault.com/questions/345891/how-should-i-check-if-ssl-session-resumption-is-working-or-not)

[Web性能权威指南](https://www.ituring.com.cn/book/1194)

[boringssl-ssl.h](https://commondatastorage.googleapis.com/chromium-boringssl-docs/ssl.h.html)

[rfc4507bis](https://tools.ietf.org/html/draft-salowey-tls-rfc4507bis-01)

[rfc7627](https://tools.ietf.org/html/rfc7627)

[About TLS Perfect Forward Secrecy and Session Resumption](https://blog.compass-security.com/2017/06/about-tls-perfect-forward-secrecy-and-session-resumption/)
