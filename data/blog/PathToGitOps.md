---
date: 2023-9-3
title: PathToGitOps
tags: [GitOps]
published: ture
---

Container 与 K8s 改变了软件包交付的模式，传统DevOps的CI/CD流程中的CD正逐步被GitOps工作流替代。**"The Path to GitOps"** 这本书详细介绍了GitOps起源、概念以及最佳实践。

## 关于书与作者

"The Path to Gitops" 由Redhat Developer出版社出版，作者是 Christian Hernandez 。

他曾经在红帽的混合平台组织的客户和现场参与团队工作了八年，并主持了双周 [GitOps银河系指南](https://www.youtube.com/playlist?list=PLaR6Rq6Z4IqfGCkI28cUMbNhPhsnj4nq3)。

Christian是Argo CD和OpenGitOps的撰稿人，是一位在基础设施工程、系统管理、企业架构、技术支持、宣传和管理方面拥有经验的技术专家。最近，他一直专注于K8s、运营模式、云原生架构和GitOps实践。

## 起源

## 概念

## 最佳实践

### 1. 模板化（as-code）

#### 1.1 使用Git仓库存储所有内容（K8s Manifest文件等）

[GitOps原则](https://github.com/open-gitops/documents/blob/release-v1.0.0/PRINCIPLES.md) 中第一条和第二条，要求系统的状态是申明式描述、不可变的。

#### 1.2 不要重复模板内容，使用 Kustomize 

Kustomize 支持按照约定的目录结构、将差异化内容与共性内容分开存储，从而避免重复YAML文件。

下面是一个列子：

K8s 与 kubectl 都内置支持 Kustomize，无需额外安装与配置。

另外一个选择是Helm。


### 2. Git 工作流

#### 2.1 将源代码与部署配置分仓库存储

分仓库存储的原因是：源代码与部署配置（K8s Manifest YAML文件等）的生命周期不同：

1. 更改部署配置（如副本数），并不期望重新编译源代码。
2. 更改部署配置并不需要从CI起点开始发布。
3. 很多企业中，维护部署配置与业务代码的是不同团队。

#### 2.2 不同Stage（或环境）使用不同目录存放，而不是分支

使用不同的分支对应不同的Stage会使得在不同Stage之间部署应用的管理流程变得复杂。

1. 不同环境之间存在固有的差异化数据（如秘钥凭证等），因此分支得长期存在。
2. 修改了共性的配置，需要分支之间同步，容易导致将差异化的内容错误同步。
3. 不如使用Kustomize这种以目录形式管理差异化内容工具来的方便。

#### 2.3 使用 "One-Trunk" 模式开发

使用 [One-Trunk](https://trunkbaseddevelopment.com/)模式，与Kustomize配合使用，可以很清晰看到每个环境的差异化内容。

使用[One-Trunk](https://trunkbaseddevelopment.com/)模式，分支管理更简单。

#### 2.4 开启分支保护

由于Git仓库里存储的内容会被定时同步到各个环境中，因此有必要开启[分支保护](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches#about-branch-protection-rules)，避免合入错误内容导致环境受损。

### 3. 代码仓库与目录结构

#### 3.1 遵循康威定律

典型的如运维团队负责维护部署配置YAML文件，开发团队负责维护代码，则将代码与部署配置文件分仓库存储。

#### 3.2 Monorepo 模式

#### 3.3 Polyrepo 模式

#### 3.4 目录结构

### 4. CI/CD 集成 GitOps

#### 4.1 CI/CD 应当解耦

CI 负责完成镜像构建，CD负责镜像发布。CI通常是同步的，CD是异步的，他们并不能很好融合到一个流程中。

CI/CD 集成 GitOps 有3中模式：

- CI Managed: 由CI工具管理GitOps
- CI Owned and CD via GitOps: GitOps作为CI的一部分
- CI Triggerd and GitOps Owned: CI触发GitOps

#### 4.2 CI Managed: 由CI工具管理GitOps


#### 4.3 CI Owned and CD via GitOps: GitOps作为CI的一部分

#### 4.4 CI Triggerd and GitOps Owned: CI触发GitOps

### 5. 处理敏感数据（秘钥）

#### 5.1 敏感数据以密文形式存储在Git仓库

#### 5.2 敏感数据存储在外部系统，Git仓库存储外部系统引用

## Reference

[Path-to-GitOps-Red-Hat-Developer-e-book](/archives/gitops/Path-to-GitOps-Red-Hat-Developer-e-book.pdf)

