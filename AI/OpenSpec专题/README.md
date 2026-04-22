---
title: OpenSpec 专题
description: 聚合 OpenSpec 的方法论、对比分析、定制化实践、跨仓库协作方案，以及基于不同模型生成的实验产出。
author: ga666666
date: 2026-04-22
updated: 2026-04-22
keywords: OpenSpec, Spec-Driven Development, AI Agent, 上下文工程, 变更管理
tags: [AI, OpenSpec, 专题]
---

# OpenSpec 专题

这个专题围绕 `OpenSpec` 展开，但重点不只是“它是什么”，而是它在 AI 协作开发里到底扮演什么角色，以及它怎样和上下文工程、跨仓库协作、模型实验、内部定制这些问题连起来。

对我来说，`OpenSpec` 最重要的价值，不是多了一套文档格式，而是它把“需求、变更、实现”之间原本隐性的关系，变成了 AI 可以消费、团队可以审查、后续可以归档的 artifact 结构。

```mermaid
flowchart LR
    A[需求意图] --> B[proposal]
    B --> C[spec delta]
    C --> D[design / tasks]
    D --> E[AI 实施]
    E --> F[archive 回写 specs]
```

## 阅读顺序

- [OpenSpec 是什么以及怎么用](./OpenSpec 是什么以及怎么用.md)
  - 适合先快速理解 `OpenSpec` 的定位、核心结构和基本使用方式

- [OpenSpec 对比 Superpowers：规范驱动与技能驱动的 AI 开发工作流](./OpenSpec 对比 Superpowers：规范驱动与技能驱动的 AI 开发工作流.md)
  - 适合比较 `OpenSpec` 与另一类 Agent 工作流系统的差异与组合关系

- [OpenSpec Fork 定制化实践报告](./OpenSpec-Fork-定制化实践报告.md)
  - 适合关注内部定制、流程增强、standards 注入与模板扩展的人

- [跨仓库AI协作方案-Spec与上下文工程](./跨仓库AI协作方案-Spec与上下文工程.md)
  - 适合关注多仓库、多角色、多 Agent 共读同一份规范的人

- [实验/README](./实验/README.md)
  - 进入基于不同模型生成的 OpenSpec 实验结果

## 这个专题包含什么

这个目录大致分成两部分：

### 1. 方法与实践

- `OpenSpec` 的基本理念
- 与其他 AI 协作工作流的差异
- 如何在团队里落地
- 如何做 fork 和定制化
- 如何支持跨仓库共享规范

### 2. 实验与产出

- `openspec-glm5`
- `openspec-minimax2.5`

这些实验目录保留了 proposal、design、tasks 和 specs 等完整产出，便于直接对比不同模型在 `OpenSpec` 结构下的实际表现。

## 为什么单独做成专题

原来这些内容是分散的：

- 一部分是方法文章
- 一部分是内部定制经验
- 一部分是模型实验目录

分散放着时，每篇文章都能看，但很难形成一个完整认知。  
整理成专题以后，路径会更清楚：

1. 先理解 `OpenSpec` 是什么  
2. 再理解它和其他工作流系统的区别  
3. 再看定制与跨仓库实践  
4. 最后看模型实验产出

这样更适合后续继续扩展案例和实验。
