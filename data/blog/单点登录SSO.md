---
date: 2020-12-11
title: 单点登录SSO
tags: [SSO, CAS]
---

SSO(Single Sign On) 是一个很古老的技术了，之前项目中也实践过，近期项目中又需要这个知识点，时间长有点模糊了，总结在此强化此知识点。

## 问题域

单点登录解决多个业务都有独立认证功能，用户时候这些业务系统时候需要分别登录，用户体验差的问题。

## 方案与流程

单点登录的方案是提供一个集中认证服务器，所有业务系统都通过此集中认证服务器登录，且一次登录后获得的凭证，可以在多个业务系统之间共享，实现多个业务系统只需一次登录效果。

单点登录的流程不算复杂，在理解方案之前，先要理解关键角色与概念。

- CAS Server: 单点登录服务器，提供用户认证、TGT签发、ST验证功能。
- CAS Client: 单点登录客户端，也就是业务服务(Service)。业务服务与CAS Server交互，完成用户登录验证、凭证签发。
- TGT(Ticket Granted Ticket): CAS Server在认证用户成功后，在响应中通过Set-Cookie回写的认证凭证；TGT是CAS Server的Session凭证。
- ST(Service Ticket): ST是TGT签发给具体一个业务服务会话的凭证，业务服务拿到此凭证后调用CAS Server接口校验凭证，如果校验通过则表示认证成功，业务系统生成Session。

![](/images/sso-flow.png)

### 单点登录与鉴权

单点登录方案只涵盖了用户认证，并未对用户鉴权做任何规范约束。通常业务系统对用户鉴权，需要获得用户属性（如RBAC鉴权模式下则需要获取用户角色信息）。单点登录模式下，业务系统没有用户身份信息，那么就需要通过用户身份管理系统获取用户身份信息。有两种实现方案：

1. CAS Server同时作为身份管理服务，这种情形下，在CAS Server的ST验证接口的返回数据中，可以带上用户属性信息，业务系统保存此用户属性信息，用于鉴权使用。
2. 独立的身份管理服务，这种情形下，业务系统需要调用身份管理服务接口获取用户属性，这里又涉及到接口调用的访问凭证，会稍微复杂一些。

## Reference

[CAS单点登陆/oAuth2授权登陆](https://www.cnblogs.com/zhuzhenwei918/p/9298943.html)

[apereo-cas-server](https://github.com/apereo/cas)

[apereo-cas-java-client](https://github.com/apereo/java-cas-client)

[cas-sample-java-webapp](https://github.com/cas-projects/cas-sample-java-webapp)
