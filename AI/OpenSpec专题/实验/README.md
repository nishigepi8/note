---
title: OpenSpec 实验
description: 收录基于不同模型生成的 OpenSpec 实验产出，便于对比 proposal、design、tasks 与 specs 的结构和质量。
author: ga666666
date: 2026-04-22
updated: 2026-04-22
keywords: OpenSpec, 实验, GLM5, MiniMax2.5, proposal, design, tasks
tags: [AI, OpenSpec, 实验]
---

# OpenSpec 实验

这个目录收录的是基于不同模型生成的 `OpenSpec` 产出。  
保留这些目录的目的，不是为了展示“模型谁更强”，而是为了保留一组可直接复查的 artifact：

- `proposal.md`
- `design.md`
- `tasks.md`
- `specs/`

## 实验目录

- [openspec-glm5](./openspec-glm5/proposal.md)
  - 包含 GLM5 生成的一组完整 OpenSpec 产出

- [openspec-minimax2.5](./openspec-minimax2.5/proposal.md)
  - 包含 MiniMax 2.5 生成的一组完整 OpenSpec 产出

## 怎么看这些实验

建议按这个顺序看：

1. `proposal.md`
   看模型是否能把需求背景和目标讲清楚

2. `design.md`
   看模型是否能给出结构化实现方案

3. `tasks.md`
   看任务拆解是否具备执行性

4. `specs/`
   看行为定义是否清晰、边界是否稳定

## 相关背景文章

- [GLM5 与 MiniMax2.5 生成质量对比报告](../../GLM5 与 MiniMax2.5 生成质量对比报告.md)
- [OpenSpec 是什么以及怎么用](../OpenSpec 是什么以及怎么用.md)
