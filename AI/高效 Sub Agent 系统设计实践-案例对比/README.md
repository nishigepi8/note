---
title: 高效 Sub Agent 系统设计实践 - 真实输出对比
description: 同一个任务在项目版多 Agent 系统与单 Agent 模式下的真实输出对比。
author: ga666666
date: 2026-04-03
updated: 2026-04-03
tags: [AI, Agent, 案例, 对比]
---

# 真实输出对比

本文档收录的是同一个任务在两种模式下的真实输出：

- **项目版多 Agent**：使用 `~/Desktop/ollama-multi-agent` 跑出来的结果
- **单 Agent**：直接调用 Ollama 模型生成的结果

任务统一为：

> 设计一个本地 Ollama 多智能体系统，要能分析需求、给实现建议、检查冲突并输出最终结论。

## 文件列表

- [项目版多 Agent 输出](./project-multi-agent-output.md)
- [单 Agent 输出](./single-agent-output.md)
