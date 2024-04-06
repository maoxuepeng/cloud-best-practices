---
date: 2021-05-29
title: CNCF-Security-Whitepaper
tags: [CNCF, Security]
---

2020年11月，[CNCF安全SIG发布](https://www.cncf.io/blog/2020/11/18/announcing-the-cloud-native-security-white-paper/)了《[云原生安全白皮书](https://github.com/cncf/tag-security/blob/main/security-whitepaper/CNCF_cloud-native-security-whitepaper-Nov2020.pdf)》，在白皮书中给出了云原生应用生命周期各个阶段的安全建议。本文做一个简要解读。

## 结构

白皮书提出了一个云原生应用生命周期的层次视图（Cloud Native Stack），然后针对这个层次视图中不同区块的安全性展开建议性描述。

![cncf-layer](/images/cncf-security-wp-layer.png)

值得注意的是层次图的3个Layer：Fundational, Environment, Lifecycle，这3个Layer比较抽象，特别是Fundational。

- Fundational：这个不等于基础设施，其范围应用是包含基础设施，以及Lifecycle过程中需用到的工具集合。目的是支撑Lifecycle过程。
- Environment：资源集合。
- Lifecycle：软件生命周期。

后续的内容都是围绕这3块展开。

## 解读

1. 白皮书中的内容比较泛，不包含具体实施建议，而是给出了安全的全景图。企业可将此白皮书作为技术沙盘参考，但是缺少实施建议，需要企业根据实际情况，找到一些合适的最佳实践，填充到沙盘中对应的位置。
2. 在整个Lifecycle中，Develop阶段重要程度最高，因为所有的安全风险源头来源于此，开发阶段因为是人的活动，人本身的技能、以及管理流程等都是安全风险引入的高发区。
3. 白皮书中缺少对变更实施过程引入安全风险的描述，人是安全防护中最薄弱的点，很多安全事故是都因为变更管理不当、或者变更人员操作不当导致。

## Reference

[CNCF_cloud-native-security-whitepaper-Nov2020](https://github.com/cncf/tag-security/blob/main/security-whitepaper/CNCF_cloud-native-security-whitepaper-Nov2020.pdf)
