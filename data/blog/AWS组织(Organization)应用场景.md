---
date: 2021-08-08
title: AWS组织(Organization)应用场景
tags: [Cloud, AWS, Organization]
---

AWS Organization 用于企业IT团队管理多个AWS账号，以实施统一的IT管控策略。AWS Organization 于 [2016-11-29发布公测版本](https://aws.amazon.com/cn/about-aws/whats-new/2016/11/announcing-aws-organizations-now-in-preview/)，于 [2017-2-27发布正式版本](https://aws.amazon.com/cn/about-aws/whats-new/2016/11/announcing-aws-organizations-now-in-preview/)。

## 概念与术语

![](/archives/aws/aws_organization.png)

### 组织 Organization

一个统一管理多个账号Account的实体。

### 根 Root

准确说是组织的根，所有组织的父容器。当前AWS只支持一个账号一个Root，在开通AWS Organization服务时候设置。

### 组织单元 Organization Unit

Root下的账号管理单元，OU可以嵌套。一个OU只能有一个父OU，一个账号只能属于一个OU。

### 账号 Account

AWS租户（注意不是AWS User，AWS User可以是一个AWS IAM User或AWS IAM Role）。
AWS 组织中有两类账号：管理账号（1个），成员账号（0个或多个）。

#### 管理账号职能

- 在组织中创建账号
- 邀请已有账号加入组织
- 移除组织中已有账号
- 实施管控策略
- 在OU内启用支持AWS OU的AWS服务，在成员账号之间共享使用、协同工作
- 作为“支付账号”支付组织内所有成员账号的费用

### 成员账号

一个AWS账号只能属于一个组织。成员账号是组织下的账号。

### 邀请

账号加入组织的流程。只有管理员账号可以发起邀请。

邀请的内容可以是两种：邀请账号加入组织，邀请账号同意feature set变更（如从consolidated billing变更为all features）。

邀请的流程通过handshake（握手）完成，使用AWS Console的时候不感知握手过程，使用AWS CLI或AWS API的时候，需要处理握手过程。

### Handshake 握手

两方交换信息的一个多步骤过程。邀请的过程通过握手完成。

### 可用的特性集 Available feature sets

- All features：组织内默认的特性集，包含所有consolidated bill特性，以及一些高级特性，这些高级特性用于管控组织内的账号。管控维度包括
    - 通过Service Control Policy控制账号能否访问的服务，
    - 禁止账号离开组织，
    - 在组织内账号之间共享使用AWS服务（参考 [与组织集成的服务](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_integrate_services_list.html) ）
- Consolidated billing：合并账单。只是有管理账号支付账单，管理账号不管控成员账号。


### Service Control Policy 服务管控策略

[SCP](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)是限制账号可以使用AWS服务、操作的策略。SCP与IAM Policy的差别是SCP不能对账号授权，其他无差别。SCP可以生效在组织、组织单元、账号上。

典型的严格管控场景：设定一个策略，Deny All；设定一个策略，允许使用某个服务的、某些资源的、某些操作权限。

### Allow list vs. Deny list

白名单和黑名单与SCP配合使用，实现对组织内账号的管控。

### Backup Policy

组织内账号下的资源备份策略。备份策略可以配置资源的备份计划。

### Tag Policy

组织内账号下资源标签策略。可以为每种资源指定标签规则。

### Service Quota

组织内的管理账号可以设置资源配额，新账号加入时候会自动触发请求配额。
[Using service quota request template](https://docs.aws.amazon.com/servicequotas/latest/userguide/organization-templates.html)

## 组织开通、用户加入

- 任何AWS 账号都可以开通组织，开通之后默认有一个Root，一个组织。
- 邀请用户加入组织，可以邀请已有用户账号，或直接创新账号。
- 邀请了账号加入组织之后，可以创建组织单元，将用户放置到组织单元下。
- 组织结构设置好了之后，可以对组织、组织单元设置策略。


## 组织策略

四种类别的策略：AI服务选择退出策略、备份策略、服务控制策略、标签策略。

## AWS管理服务与Organization集成

AWS Organization作为AWS的企业管理的一个基础功能，AWS部分服务支持与AWS Organization集成，提供企业部门之间协同工作。

当前有23个服务支持与AWS Organization集成，这23个服务绝大多数都是管理服务，用于组织管理员方便管成员账号的行为。可以分为3个类别：合规检查与报告、集中策略管控、资源共享。

### 合规检查与报告

- AWS Audit Manager
- AWS Security Hub
- AWS Config
- AWS Artifact

### 集中策略管控

- AWS CloudTrail
- AWS Firewall Manager
- Amazon GuardDuty
- Amazon Macie
- AWS Backup
- AWS CloudFormation StackSets
- Amazon S3 Storage Lens
- AWS Compute Optimizer
- AWS Health
- AWS Service Catalog
- Service Quotas
- AWS Systems Manager
- Tag policies
- AWS Trusted Advisor

### 资源共享

- AWS Resource Access Manager
- AWS Single Sign-On
- AWS Directory Service
- AWS License Manager
- AWS Marketplace

### 账单服务与Organization集成

组织内管理账号统一支付账单，管理账号可以查看每个成员账号的详细账单。

### 资源服务与Organization集成

管理服务更多适用于企业的非业务部门（合规审计部、安全部门等）对组织的管理需求。在企业内部的业务部门，他们在同一个组织，属于组织成员，他们之间需要共享资源。

典型的是多个部门业务集成时候，希望共享网络规划、共享VPC（ https://aws.amazon.com/cn/blogs/networking-and-content-delivery/vpc-sharing-a-new-approach-to-multiple-accounts-and-vpc-management/）。

AWS Resource Access Manager 服务提供组织不同账号之间内资源共享的能力。

## Reference

[AWS organizations](https://aws.amazon.com/cn/organizations/)

[A History of Amazon Web Services (AWS)](https://www.awsgeek.com/AWS-History/)

[Azure subscription and service limits, quotas, and constraints](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/azure-subscription-service-limits)
