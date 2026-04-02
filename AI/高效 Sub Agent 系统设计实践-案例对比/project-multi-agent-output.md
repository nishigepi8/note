---
title: 项目版多 Agent 真实输出
description: 使用 ~/Desktop/ollama-multi-agent 项目运行得到的真实输出。
author: ga666666
date: 2026-04-03
updated: 2026-04-03
tags: [AI, Agent, 多Agent, 真实输出]
---

# 项目版多 Agent 真实输出

## 任务

> 设计一个本地 Ollama 多智能体系统，要能分析需求、给实现建议、检查冲突并输出最终结论。

## 运行方式

使用项目目录：`~/Desktop/ollama-multi-agent`

入口：

```bash
node cli.js "设计一个本地 Ollama 多智能体系统，要能分析需求、给实现建议、检查冲突并输出最终结论"
```

## 真实输出摘要

```text
[main] task=设计一个本地 Ollama 多智能体系统，要能分析需求、给实现建议、检查冲突并输出最终结论
[main] raw plan
[
  {
    "agent": "需求分析",
    "role": "负责需求收集和分析",
    "task": "收集用户需求文档，理解业务流程，识别关键问题。",
    "dependsOn": []
  },
  {
    "agent": "实现建议",
    "role": "提供技术方案和实现建议",
    "task": "根据需求分析结果，提出可行的技术解决方案和代码实现建议。",
    "dependsOn": ["需求分析"]
  },
  {
    "agent": "冲突检查",
    "role": "检测并解决潜在的冲突",
    "task": "审查实现建议，识别可能的冲突点，并提供相应的解决策略。",
    "dependsOn": ["实现建议"]
  },
  {
    "agent": "最终结论",
    "role": "总结和输出最终决策",
    "task": "综合需求分析、实现建议和冲突检查结果，生成最终的系统设计和实施计划。",
    "dependsOn": ["需求分析", "实现建议", "冲突检查"]
  }
]
```

### 真实输出特征

- 主代理会动态生成角色，而不是写死固定职责
- `dependsOn` 会影响执行顺序
- relay 会按接收者角色定向投递消息
- 最终输出会收敛成 `plan / messages / summary / conclusion`

### 结论摘要

真实项目版输出的特点是：

- 角色划分清楚
- 依赖关系明确
- 协作轨迹完整
- 更接近一套可交付的协作系统
