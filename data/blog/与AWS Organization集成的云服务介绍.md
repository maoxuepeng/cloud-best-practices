---
date: 2021-08-15
title: 与AWS Organization集成的云服务介绍
tags: [Cloud, AWS, Organization]
---

AWS管理与监管（12个）、安全合规（11个）总共23个服务与AWS Organization集成，便于企业在云上实施规范落地与合规审计。

## Backup

[Backup](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_backup.html) 服务与 AWS Organization集成，组织内管理员账号设置组织内统一备份策略。

## Tag policies

标签服务 与 AWS Organization集成，管理账号设置组织内资源都需遵从标签规则。

## Artifact

AWS [Artifact](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-artifact.html) 服务两个用途：查看和下载AWS自身的合规报告；租户查看/签署与AWS之间的协议。
当用于组织场景时，成员账号可以查看管理账号申请了的合规报告；管理账号可以代表成员账号签署与AWS之间的协议。

## Audit Manager

AWS Audit Manager 用于帮助客户出行业合规的审计报告。Audit Manager提供将行业合规报告模板与AWS资源映射的方式，将资源使用方式转换为合规报告，以便审计员阅读。Audit Manager提供预设合规报告框架，也支持客户自定义合规报告框架与资源映射。

AWS Audit Manager 中的预构建框架包括 AWS Control Tower、AWS License Manager、CIS AWS Foundations 基准测试 1.2.0 和 1.3.0；CIS 控制 v7.1 实施组 1、由 Allgress 制定的 FedRAMP 中等基准、一般数据保护法规 (GDPR)、GxP 21 CFR 第 11 部分、《健康保险流通与责任法案》(HIPAA)、HITRUST v9.4 第 1 级、支付卡行业数据安全标准 (PCI DSS) v3.2.1、服务组织控制 2 (SOC 2) 和 NIST 800-53 (Rev 5)。

支持的框架列表： https://docs.aws.amazon.com/audit-manager/latest/userguide/framework-overviews.html ，总共22个。

![](/images/aws/audit-manager-framework-libs.png)

其中就包含“AWS架构完善框架”的规则，这些规则都是通过AWS Config服务落地的。

![](/images/aws/audit-manager-framework-waf.png)

## CloudFormation StackSet

使用 [CloudFormation StackSet](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-cloudformation.html) 可以（组织内）多个账号下的多个Region内创建资源，因此可以使用此特性快速配置组织内账号下的服务控制策略、角色权限等。


## CloudTrail 

[CloudTrail](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-cloudtrail.html) 用来记录账号下的活动日志，组织管理员可以设置活动日志记录规则，组织成员可以查看规则但是不能修改规则；日志记录存储到S3桶中，组织成员不能读取到桶中的日志文件。

## Compute Optimizer

[Compute Optimizer](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-compute-optimizer.html) 分析账号下资源配置与利用率并给出报告，报告中包含一些建议，可以帮助降低费用并提升资源利用率。组织管理员使用此服务分析组织内的资源利用率并提升工作负载的性能。

## Config

[AWS Config](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-config.html) 用来记录租户账号下AWS资源配置与变更记录，并可以设定资源配置变化的规则与通知。Config通过Aggregator资源，与AWS Organization配合，组织管理账号可以将组织内所有成员账号、所有Region下的资源配置汇聚到管理账号下的Config服务中，统一监控。

Config服务提供[rdk(rules development kit)](https://github.com/awslabs/aws-config-rdk )工具，开发者使用此工具可以开发自定义的规则，上传到Config服务；也支持使用AWS Lambda服务运行规则代码。同时Config也提供内置的规则 191条（ https://docs.aws.amazon.com/config/latest/developerguide/managed-rules-by-aws-config.html ）。

## Directory Service / Managed Microsoft AD
[AWS Managed Microsoft AD](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-directory-service.html) 服务与 AWS Organization集成，组织的所有账号可以在共用一个AD服务实例；账号下对AD实例对接的资源在同一个Region、可以是不同的VPC。


## Firewall Manager

[AWS Firewall Manager](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-fms.html) 与 AWS Organization集成，可以对组织中所有账号实施集中的安全配置和防火墙规则管理，可管理的安全规则包括：AWS WAF规则、AWS Shield（DDoS防护）规则、AWS Network Firewall规则。

## GuardDuty

[AWS GuardDuty](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-guardduty.html) 是一项安全威胁检测服务，通过检测算法对AWS资源的日志扫描分析给出威胁报告。GuardDuty与AWS Organization集成，组织管理账号对应为GuardDuty的管理员用户，管理员用户可以配置检测规则并对组织内所有成员账号下的资源生效。

## Health

[AWS Health](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-health.html ) 提供租户资源健康看板，包括租户的资源健康状态、以及AWS服务健康状态。AWS Health 与 AWS Organization集成，管理员账号可以在PHD（Personal Health Dashboard）上查看到组织内所有账号下的资源监控状态、以及事件通知。

![](/images/aws/aws-health-event-log-1.png)
![](/images/aws/aws-health-event-log-2.png)


## License Manager
[AWS License Manager](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-license-manager.html) 提供管理软件License消耗的功能，支持AWS云与客户数据中心的License消耗统一管理，让License的使用情况容易管理，避免License超限等违规使用情况发生。
AWS License Manager 与 AWS Organization集成，管理员账号可以管理组织内所有成员账号的License使用情况。

## Macie
[AWS Macie](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-macie.html) 是一项数据安全与数据隐私合规服务，Macie扫描S3中存储的数据，发现并持续监测敏感数据并给出报告。Macie与AWS Organization集成，管理员账号作为Macie服务的管理员，对组织内所有成员账号下的S3桶扫描并监测敏感数据，及时发现数据违规。

## Marketplace
[AWS Marketplace](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-marketplace.html) 与AWS Organization集成，管理员账号在Marketplace购买的软件，可以授权给组织成员账号使用。如果Marketplace上购买的软件需要License，Marketplace的软件提供商通过集成License Manager，分配License给对应的组织内的账号。

## Resource Access Manager
[AWS Resource Access Manager](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-ram.html) 可以实现在AWS账号之间共享资源。此服务与AWS Organization集成，组织管理员账号可以要求成员账号使用公共资源，典型的场景为管理员账号统一规划网络，将VPC共享给成员账号，要求成员账号资源都加入到共享的VPC（同时通过组织服务策略限制成员账号的VPC访问）。
已经与Resource Access Manager对接支持共享的资源如下。

- AWS App Mesh
- Amazon Aurora
- AWS Certificate Manager Private Certificate Authority
- AWS CodeBuild
- Amazon EC2
- EC2 Image Builder
- AWS Glue
- AWS License Manager
- AWS Network Firewall
- AWS Outposts
- Amazon S3 on Outposts
- AWS Resource Groups
- Amazon Route 53
- AWS Systems Manager Incident Manager
- Amazon VPC

## Security Hub
[AWS Security Hub](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-securityhub.html) 提供安全看板，聚合多个AWS安全服务（ Amazon GuardDuty、Amazon Inspector 和 Amazon Macie）的事件与警报统一管理。此服务与AWS Organization集成，管理员账号设置成员账号自动开启Security Hub服务，包括新账号加入组织。

## AWS S3 Storage Lens
[AWS S3 Storage Lens](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-s3lens.html) 管理组织中数量众多的S3桶，提供29+统计指标，掌控组织内S3桶的使用情况。

## Service Catalog
[AWS Service Catalog](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-servicecatalog.html) 与AWS Organization集成，可以在组织内共享portfolio。

## Service Quotas
[AWS Service Quota](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-servicequotas.html) 与 AWS Organization集成，组织管理员账号可以配置“配额请求模板”（最多10个），当新的组织成员账号加入时候，会自动触发执行“配额请求模板”，为账号设置配额。
注意，新创建账号时候并不能设置默认的账号配额。

![](/images/aws/aws-org-quota.png)

## Single Sign-On
[AWS Single Sing-On](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-sso.html) 是一个单点登录服务，此服务以AWS作为SP，链接各种IdP(AWS, AD, SMAL2.0)，实现统一账号访问AWS并使用AWS服务。此服务可以与企业已有的身份源（如AD）对接，使用企业已有账号登录AWS，并在AWS SSO中统一管理权限。

## System Manager
[AWS System Manager](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-ssm.html) 是一个资源信息汇总的仪表盘，并提供变更流程管理（变更创建、审核、影响报告）。此服务与AWS Organization集成，提供组织内的变更管理，以及定制组织内所有账号资源汇聚的可视化仪表盘。

## Trusted Advisor
[AWS Trusted Advisor](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-ta.html) 分析AWS资源并给出报告（包含改进建议），类似专家服务。此服务与AWS Organization集成，可对组织内的资源分析并给出改进报告。

## Reference

