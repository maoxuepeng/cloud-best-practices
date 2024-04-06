---
date: 2021-06-03
title: CNCF-supply-chain-security
tags: [CNCF, Security, Supply-Chain]
---

软件的供应链安全是软件安全威胁的源头，在[CNCF安全白皮书](https://github.com/cncf/tag-security/blob/main/security-whitepaper/CNCF_cloud-native-security-whitepaper-Nov2020.pdf)中，软件供应链安全也占据了重要的一部分。[美国联邦政府国家数字安全能力提升行政令](https://www.whitehouse.gov/briefing-room/presidential-actions/2021/05/12/executive-order-on-improving-the-nations-cybersecurity/)中也提别提到软件供应量的安全策略。基于上述背景，CNCF提出了[软件供应链安全最佳实践白皮书](https://github.com/cncf/tag-security/blob/main/supply-chain-security/supply-chain-security-paper/CNCF_SSCP_v1.pdf)。

![](/images/software-supply-chain/2caad364f0eff4d34a4f1d14b53d6746.png)

## 概述

白皮书提出软件供应链安全的4个原则。

白皮书的内容包括一系列最佳实践，可选工具，设计考量，用于消减软件供应链的安全威胁。

白皮书不包含工具的具体使用与配置方法，不包含完整的软件供应链安全解决方案。

## 假设与约束

### 应用 Assurance 分级

谈论软件安全需结合业务场景考量，而不只是简单的堆砌安全措施，需综合考虑安全与成本。不同业务场景的应用对安全要求是不同的。

- Low Assurance: 无需安全保障的软件应用程序。如原型应用、alpha版本的应用等。这类应用不在此白皮书中讨论。
- Moderate Assurance: 中度安全保障的软件应用程序，通常是使用“enterprise ready”的开发流程开发出的应用。此白皮书主要讨论这类应用的供应链安全。
- High Assurance: 必须保证防篡改以及一致性。这类软件通常是人体健康、医疗、关键基础设施（如电力）、金融服务等领域。

### 安全风险环境 Risk Environment 分级

- Low risk enviornment: 在低风险环境中运行的软件，本白皮书的建议不适用。
- Moderate risk enviornment: 中度风险环境包含除个人身份信息、个人健康信息、生命支撑系统数据、关键基础设施信息之外的所有环境。本白皮书的建议适用与中度风险环境。
- High risk enviornment: 高风险环境包含上述的个人身份信息、个人健康信息、生命支撑系统数据、关键基础设施信息处理的应用所在环境。

## 安全加固软件供应链

供应链是指产品到消费的流程，在软件领域，软件供应链是指软件开发流程，包含从概念到产品发布（二进制包）与分发。

上图是一个软件应用链与传统制造业供应量对比，相比较与传统制造业供应链，软件供应链有下面几个显著区别。

1. Intangible 抽象性: 软件是虚拟无形的。
2. Mercurial 多变性: 无形的特征导致容易变化。
3. Iterative reuse 无尽复用: 一个产品的供应量，包含n个产品和nn各供应链。

基于上述特征，业界又将软件供应链称为```软件工厂```。软件工厂安全加固的最佳实践指导原则如下。

1. Verification 验证: 软件供应链每个阶段都需要验证，才能生产出可信的软件。
2. Automation 自动化: 通过自动化减少人引入的不确定性。
3. Authorization in Controlled Environments: 授权访问环境，避免因不必要的人或系统访问引入问题。
4. Secure Authentication: 确保流程中的所有实体，都有唯一身份标识。

本文章将软件供应链分成5个阶段来阐述安全措施建议。

1. Securing the Source Code: 源代码安全。
2. Securing the Materials: 二方、三方库安全。
3. Securing the Build Pipelines: 构建任务、流水线安全。
4. Securing the Artefacts: 软件包安全。
5. Securing Deployments: 部署安全。

同时，针对每个建议，都会加上 ```Assurance``` 和 ```Risk Environment``` 分级。

## 1. Securing the Source Code: 源代码安全

源代码安全是所有软件供应链安全的基石。软件供应链安全的第一步，就是建立源代码安全机制，保证源代码的```integrity 完整性```。

### 1.1 Verification

#### 1.1.1 commit需签名 [assurance: moderate to high; risk: moderate to hight]

[启用代码仓(git)签名](https://git-scm.com/book/en/v2/Git-Tools-Signing-Your-Work)。
参考[Signing commits with GPG](https://docs.gitlab.com/ee/user/project/repository/gpg_signed_commits/)，或 [telling-git-about-your-signing-key](https://docs.github.com/en/github/authenticating-to-github/telling-git-about-your-signing-key)。

#### 1.1.2 启用完整验证连机制以保护分支 [assurance: high; risk: hight]

```out-of-band 带外``` 修改代码后分支merge，是一个潜在的安全风险源。启用```full attestation 完整验证链```机制，消除分支merge引入的风险。

```full attestation 完整验证链``` 定义是 ```一旦MR中所有的commit都签名验证通过，则签名最后一个commit```。```git merget -S --verify-signatures```。

### 1.2 Automation

#### 1.2.1 防止秘钥与凭证被提交到代码仓库 [assurance: moderate to high;risk: moderate to high]

秘钥与凭证，如SSH秘钥、token等不能提交到源代码仓库。有一些检测工具可以扫描源代码仓库中的秘钥，如 [trufflehog](https://github.com/trufflesecurity/truffleHog)，可以通过在代码仓库提交的回调(客户端的pre-hook, server端的pre-receiver)中实施检测。另外一种方法是通过秘钥[文件清单方式，阻止在清单中的文件被提交到代码仓](https://docs.gitlab.com/ee/push_rules/push_rules.html#prevent-pushing-secrets-to-the-repository)

### 1.2.2 明确代码仓库的Committer并负责代码规范落地 [assurance: high; risk: high]

代码仓库的管理员需要明确谁有权限合入代码到仓库，并定义代码测试、代码检查的规则。这些规则可以在客户端的```pre-commit```与服务器端的```pre-receiver```回调来实现。
还可以使用 ```.gitignore```, [.gitattribute](https://rehansaeed.com/gitattributes-best-practices/), ```denylist``` 文件来设定仓库提交策略，以满足多样的安全需求。 

### 1.2.3 自动化安全扫描与测试 [assurance: moderate to high; risk: moderate to high]

与单元测试、功能测试、集成测试来保证功能一样，软件供应链过程中安全测试也是必不可少的。安全测试分为 ```Static Application Security Tests (SAST)```, ```Dynamic Application Security Tests(DAST)``` 两类。静态扫描尽早的介入开发流程，包括IDE的集成。[OWASP](https://owasp.org/)组织提供了[静态扫描最佳实践](https://owasp.org/www-project-application-security-verification-standard/)与[工具](https://owasp.org/www-community/Source_Code_Analysis_Tools)。

代码安全扫描结果的元数据需要以Hash形式连接到代码发布包，用于追溯与证明。同时覆盖率等扫描测试报告也需要在代码仓里发布，以便下游使用者可以清晰明了获得发布包的安全状态。

### 1.3 Controlled Environments

#### 1.3.1 定制并执行协同开发的代码代码提交策略 [assurance: moderate to high; risk: moderate to high]

#### 1.3.2 定义协同开发的角色职责 [assurance: moderate to high; risk: moderate to high]

协同开发流程中角色有开发、维护、Onwer、代码检视等角色，定义这些角色权限并与人员对应。

#### 1.3.3 执行四眼原则 [assurance: moderate to high; risk: moderate to high]

[四眼原则](https://dzone.com/articles/devops-guide-implementing-four-eyes-principle-with) 是指 ```一个特定的活动，需要至少2个人批准```。

#### 1.3.4 制定分支保护策略 [assurance: moderate to high; risk: moderate to high]

代码仓库中一些分支需要执行更严格的合入策略，如禁止在主分支上强制提交```git push --force```以避免历史记录被覆盖。

### 1.4 Secure Authentication

#### 1.4.1 启用代码仓库的多因子认证 [assurance: moderate to high; risk: moderate to high]

#### 1.4.2 使用SSH-Key认证方式访问代码仓库 [assurance: moderate to high; risk: moderate to high]

SSH-Key认证方式比用户密码方式安全性更高。

#### 1.4.3 考虑提供Key轮换机制 [assurance: moderate to high; risk: moderate to high]

使用SSH-Key访问，存在开发者私钥泄漏的情况，通过Key轮换机制，可以应对开发人员Key泄漏的问题。

#### 1.4.4 系统对接时候使用有时间限制、临时的凭证 [assurance: moderate to high; risk: moderate to high]

## 2. Securing the Materials: 二方、三方库安全

### 2.1 Verification

#### 2.1.1 验证三方库、开源库的安全性

最基本的要求是，使用[Software Composition Analysis (SCA)](https://www.whitesourcesoftware.com/resources/blog/software-composition-analysis/)工具验证所有三方、开源库是否存在潜在的安全漏洞。

#### 2.1.2 三方软件供应商需提供SBOM  [assurance: high; risk: high]

#### 2.1.3 管理三方库的依赖关系 [assurance: moderate to high; risk: moderate to high]

三方库自身有复杂的依赖关系，需要有三方库依赖关系跟踪与管理的工具，推荐使用OWASP 的 [Dependency-Track](https://owasp.org/www-project-dependency-track/)工具。

#### 2.1.4 基于源码编译三方库而不是直接使用二进制  [assurance: high; risk: high]

使用三方库时候，直接使用二进制会引入潜在风险，你并不清楚二进制包与源代码的准确对应关系，以及编译选项是否符合安全要求。

#### 2.1.5 构建可信的软件包仓库  [assurance: high; risk: high]

通过包签名、仓库数字证书身份认证等技术手段。

#### 2.1.6 为软件包生成不可更改的SBOM码

构建软件包时候，生成不可更改的[SBOM](https://www.ntia.gov/SBOM)编码，是一个推荐的实践。在软件包中包含所有内容的SBOM编码清单，使用此软件包的下游团队，根据软件包SBOM编码可以分析对应的漏洞。

对于三方库的SBOM编码使用[Software Composition Analysis (SCA)](https://www.whitesourcesoftware.com/resources/blog/software-composition-analysis/)工具管理。

当前2个主流的SBOM规范：[SPDX](https://github.com/spdx/spdx-spec), [CycloneDX](https://github.com/CycloneDX/specification) 。

### 2.2 Automation

#### 2.2.1 扫描软件漏洞

#### 2.2.2 扫描软件包License

构建过程中需要检查License履行义务，Linux软件基金会维护的[Open Compliance Program](https://compliance.linuxfoundation.org/references/tools/)提供了一些License合规检查的工具。License检查的元数据需要包含在软件包的SBOM中。

### 2.2.3 对依赖的二方、三方软件包执行软件成分分析(SCA)

使用SCA工具分析，SCA工具还具备SBOM校验功能。

### 2.3 Controlled Environments

无。

### 2.4 Secure Authentication

无。

## 3. Securing the Build Pipelines: 构建任务、流水线安全

在第一章中讨论了第一方代码、二方库、三方库这3部分软件成分的安全建议，那么流水线就是将这三部分材料集成组装。安全的构建流水线，需要包含4部分内容。

1. 构建步骤
2. 构建执行机
3. 构建工具
4. 流水线编排

同时，构建的元数据需要签名存档，用于带外（外部系统集成时）的验证。

流水线由一些列的安全加固的步骤组成，承载指令的容器镜像存储在安全加固的仓库中，并被安全加固的编排服务调度执行（如K8S）。

举个例子，美国空军的[Platform One](https://www.nist.gov/system/files/documents/2021/02/22/05%20-%20Nicolas%20-%20DoD%20Enterprise%20DevSecOps%20Initiative%20-%20Keynote%20Presentation%20v2.0-w%20ZT.pdf)就是实现上述安全理念的流水线平台服务。```Platform One```参考 [DoD Container Hardning Guide](/archives/security/Final_DevSecOps_Enterprise_Container_Hardening_Guide_1.1.pdf) 加固流水线指令镜像，加固容器镜像仓库（称为```Iron Bank 钢铁银行```）。

通过安全加固后的组件构建的流水线平台，能够降低软件供应链过程中软件包被篡改的可能性。流水线编排确定每个阶段精确的执行步骤，减少可能的攻击面。整个流水线设计需要满足攻击受损范围可控，一个步骤被篡改了不会影响到整个流水线；同时，通过带外验证手段消除流水线的根凭证丢失带来的风险。

### 3.1 构建安全流水线的指导性原则

1. 每个组件component都有唯一的责任人/团队。
2. 每个步骤都需要定义明确的输入与输出，便于通过数据控制与观察流水线。
3. 输出内容需要签名```防止抵赖 non-repudiation```。
4. 流水线的基础设施与配置都是不可变的。
5. 流水线的每个步骤自身也是可被自动化测试的，以验证流水线自身的安全性。
6. 每条流水线中需要有一个步骤输出SBOM。

### 3.2 一个样例流水线例子

一个应用依赖关系如下。

![](/images/software-supply-chain/da450b0415c5a9c101f7719d1f92c1e7.png)

**步骤一：编译依赖库**。

**步骤二：链接依赖库**。
![](/images/software-supply-chain/e62a77a3faacd30271cb829dd364bc97.png)

**步骤三：编译应用**。
![](/images/software-supply-chain/8ba5341036226df20cf47b1e98b6107d.png)

**步骤四：测试**。
![](/images/software-supply-chain/45770e135fb6acbcfe74e8f5ea1c1245.png)

**步骤五：编译组件上架发布仓库**。
![](/images/software-supply-chain/098f84bbee1c5ba2ff3b596b5e9b543c.png)

**步骤六：部署**。
![](/images/software-supply-chain/f090aee9d09ac19b1cff83072d099fa9.png)

### 3.3 Verification

#### 3.3.1 使用密码学理论建立防护策略 [assurance: moderate to high; risk: moderate to high]

参考CNCF项目 [in-toto](https://in-toto.io/)。

#### 3.3.2 使用环境与依赖之前先验证有效性 [assurance: moderate to high; risk: moderate to high]

#### 3.3.3 验证执行机的安全性  [assurance: moderate to high; risk: moderate to high]

使用系统加固工具，如```seccomp```, ```AppArmor```, ```SELinux``` 加固执行机。执行机的访问必须授权，发现安全事件上报到安全事件中心([Security Information and Event Management](https://www.gartner.com/en/documents/3981040/magic-quadrant-for-security-information-and-event-manage) )。

#### 3.3.4 通过可重建的机制验证构建制品  [assurance: high; risk: high]

对于确定的构建流程，同样的输入，多次构建制品是相同的。[Reproducible-builds.org](https://reproducible-builds.org/) 给出了通过可重建的构建验证构建制品可信的最佳实践。[Reproducible-builds.org](https://reproducible-builds.org/) 项目的动机是提供一种机制验证软件工厂生产的制品中没有漏洞与后门。

### 3.4 可重建构建的建议

#### 3.4.1 在构建流程中锁定并验证外部需求 [assurance: moderate to high; risk: moderate to high]

#### 3.4.2 找出不消除源代码中的不确定因素 [assurance: moderate to high; risk: moderate to high]

时间戳、区域元素、版本号等因素都是构建流程中的不确定因素。[Reproducible-builds.org](https://reproducible-builds.org/)项目中有不少这方便的建议，[Diffscope](https://diffoscope.org/)工具可以用于探索并找出影响源代码中不确定性的因素。

#### 3.4.3 记录构建环境信息  [assurance: high; risk: high]

为了实现可重建的构建，构建过程中的工具版本、配置信息、依赖库版本等等，都需要被记录。Debian 包构建时候会产生一个 [.buildinfo](https://wiki.debian.org/ReproducibleBuilds/BuildinfoFiles) 文件，```可重建构建```就是其目标之一（另外一个目标是分析、调试场景）。

#### 3.4.4 自动创建构建环境  [assurance: high; risk: high]

HashiCorp的[Vagrant](https://www.vagrantup.com/)工具可用于快速创建虚拟机、docker以及安装相关工具。

#### 3.4.5 实施（基础设施）分布式构建  [assurance: high; risk: high]

一个构建任务在2个基础设施上并行构建，并对比构建制品的Hash以验证构建的一致性。[rebuilderd](https://github.com/kpcyrd/rebuilderd)工具可以辅助创建并行构建任务。

### 3.5 Automation

#### 3.5.1 使用Pipeline As Code 实现CI/CD的过程全自动化 [assurance: moderate to high; risk: moderate to high]

[Pipeline As Code](https://www.jenkins.io/doc/book/pipeline-as-code/) 。

#### 3.5.2 制定流水线标准 [assurance: moderate to high; risk: moderate to high]

#### 3.5.3 流水线编排服务自身安全保障 [assurance: moderate to high; risk: moderate to high]

#### 3.5.4 执行机单一用途 [assurance: high; risk: moderate]

### 3.6 Controlled Environments

#### 3.6.1 维持连接到软件工厂的连接数量最小化 [assurance: high; risk: high]

#### 3.6.2 执行机的职责单一 [assurance: high; risk: high]

#### 3.6.3 执行机从流水线调度获取参数，不从本地环境获取参数参数 [assurance: high; risk: high]

#### 3.6.4 将构建输出存储到与输入隔离的独立仓库 [assurance: high; risk: high]

### 3.7 Secure Authentication

#### 3.7.1 只允许通过"Pipeline As Code"方式修改流水线 [assurance: moderate to high; risk: moderate to high]

#### 3.7.2 定义用户角色 [assurance: moderate to high; risk: moderate to high]

#### 3.7.3 使用离线的信任根，构建认证体系 [assurance: high; risk: high]

参考 [Solving the Bottom Turtle](https://spiffe.io/book/) 。

#### 3.7.4 使用临时凭证 [assurance: high; risk: high]

## 4. Securing the Artefacts: 软件包安全

### 4.1 Verification

#### 4.1.1 签名构建过程每个步骤输出件 [assurance: moderate to high; risk: moderate to high]

#### 4.1.2 验证构建过程中每个步骤的签名 [assurance: moderate to high; risk: moderate to high]

### 4.2 Automation

#### 4.2.1 使用 TUF/Notary管理构建制品的签名 [assurance: moderate to high; risk: moderate to high]

#### 4.2.2 管理来自in-toto的元数据

[Grafeas](https://grafeas.io/) 是构建制品元数据管理工具，通过[Kritis](https://github.com/grafeas/kritis)控制器与K8S集成。

### 4.3 Controlled Environments

#### 4.3.1 构建制品的组成部件供应方都是认证的 [assurance: high; risk: high]

#### 4.3.2 构建秘钥轮换系统 [assurance: high; risk: high]

#### 4.3.3 使用支持OCI镜像规范的容器镜像仓库 [assurance: high; risk: high]

### 4.4 加密

#### 4.4.1 构建制品在分发之前加密，并确保只有认证方才能解密 [assurance: high; risk: high]

## 5. Securing Deployments: 部署安全

### 5.1 Verification

#### 5.1.1 确保部署方能够验证制品以及元数据 [assurance: moderate to high; risk: moderate to high]

#### 5.1.2 确保部署方能够验证文件的“新鲜度” [assurance: moderate to high; risk: moderate to high]

### 5.2 Automation

#### 5.2.1 使用The Update Framework [assurance: moderate to high; risk: moderate to high]

[TUF规范](https://theupdateframework.github.io/specification/latest/index.html) 。

## Prior Art / References:

CISQ Tool to Tool Software Bill of Materials Exchange - https://www.it-cisq.org/software-bill-of-materials/

SCVS OWASP Software Component Verification Standard - https://owasp.org/www-project-software-component-verification-standard/

Vulnerabilities in the Core - Preliminary Report and Census II https://www.coreinfrastructure.org/wp-content/uploads/sites/6/2020/02/census_ii_vulnerabilities_in_the_core.pdf

OWASP Packman https://github.com/OWASP/packman

CII Best Practices Badge Program https://bestpractices.coreinfrastructure.org/en

OSSF Scorecard https://github.com/ossf/scorecard

OSSF Open Source Project Criticality Score https://github.com/ossf/criticality_score

CycloneDX Specification https://cyclonedx.org/docs/1.2/

CycloneDX Tools https://cyclonedx.org/tool-center/

SPDX Specification https://spdx.github.io/spdx-spec/

OWASP Dependency-Track https://dependencytrack.org/

What is sigstore? https://sigstore.dev/what_is_sigstore/

Google Binary Authorization/SLSA 
https://github.com/slsa-framework/slsa
https://cloud.google.com/binary-authorization
https://cloud.google.com/security/binary-authorization-for-borg
https://aws.amazon.com/blogs/aws/new-code-signing-a-trust-and-integrity-control-for-aws-lambda/
https://patentimages.storage.googleapis.com/ae/e4/9b/92cf7c83be565a/US20200201620A1.pdf

Red Hat SORD and Trusted Supply Chain reference implementation - software factory operator
https://github.com/ploigos/ploigos-software-factory-operator
https://blog.autsoft.hu/a-confusing-dependency/
https://dwheeler.com/trusting-trust/dissertation/wheeler-trusting-trust-ddc.pdf

DevOps Automated Governance Reference Architecture https://itrevolution.com/book/devops-automated-governance-reference-architecture/

Reproducible builds organisation https://reproducible-builds.org/

Security Consideration for Code Signing (Reference for key management aspect) https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.01262018.pdf 

2020 data-breach investigation report https://enterprise.verizon.com/resources/reports/2020-data-breach-investigations-report.pdf

Policy-driven continuous integration with Open Policy Agent https://blog.openpolicyagent.org/policy-driven-continuous-integration-with-open-policy-agent-b98a8748e536

Snyk Best Practices https://res.cloudinary.com/snyk/image/upload/v1535626770/blog/10_GitHub_Security_Best_Practices_cheat_
sheet.pdf

Infosec Institute security best practices  https://resources.infosecinstitute.com/topic/security-best-practices-for-git-users/

DoD Enterprise DevSecOps Reference Design: https://dodcio.defense.gov/Portals/0/Documents/DoD%20Enterprise%20DevSecOps%20Reference%20Design%20v1.0_Public%20Release.pdf?ver=2019-09-26-115824-583

## Reference

[CNCF Paper Defines Best Practices for Supply Chain Security](https://www.cncf.io/announcements/2021/05/14/cncf-paper-defines-best-practices-for-supply-chain-security/)

[Executive Order on Improving the Nation’s Cybersecurity](https://www.whitehouse.gov/briefing-room/presidential-actions/2021/05/12/executive-order-on-improving-the-nations-cybersecurity/)

[CNCF_supply_chain_security_v1.pdf](https://github.com/cncf/tag-security/blob/main/supply-chain-security/supply-chain-security-paper/CNCF_SSCP_v1.pdf)

[Breaking trust: Shades of crisis across an insecure software supply chain](https://www.atlanticcouncil.org/in-depth-research-reports/report/breaking-trust-shades-of-crisis-across-an-insecure-software-supply-chain/)

[State of Cybersecurity Industry Exposure at Dark Web](https://www.immuniweb.com/blog/state-cybersecurity-dark-web-exposure.html)

[.gitattributes Best Practices](https://rehansaeed.com/gitattributes-best-practices/)

[If you’re not using SSH certificates you’re doing SSH wrong](https://smallstep.com/blog/use-ssh-certificates/)

[DoD Container Hardning Guide](https://dl.dod.cyber.mil/wp-content/uploads/devsecops/pdf/Final_DevSecOps_Enterprise_Container_Hardening_Guide_1.1.pdf)

[Is your software supply chain secure?](https://blog.convisoappsec.com/en/is-your-software-supply-chain-secure/)
