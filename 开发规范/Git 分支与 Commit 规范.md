---
title: 中大型团队的 Git 实践
description: 这是一份经过实际项目打磨、适合中大型研发团队（100-500 人）的 Git 使用规范。目标是：保持代码历史清晰、可追溯、易协作，同时降低合并冲突和上线风险。
author: ga666666
date: 2026-01-10
updated: 2026-01-10
keywords: 中大型团队的, Git, 实践, 前言, 分支命名规范, Commit, Message, 规范
tags: []
---


# 中大型团队的 Git 实践

## 前言

这是一份经过实际项目打磨、适合中大型研发团队（100-500 人）的 Git 使用规范。目标是：**保持代码历史清晰、可追溯、易协作，同时降低合并冲突和上线风险**。

适用于后端、前端、移动端等多端共用仓库的项目。如果你的团队也在寻找一套简单、可落地、不折腾的 Git 规范，这份可以直接拿来参考或稍作调整后使用。

**参考依据**：GitFlow 变体 + Conventional Commits + Jira 集成实践

---

## 一、分支命名规范

统一使用小写 + 连字符，强制包含 Jira 任务号（或工单号），便于自动化追踪。

**格式**：`<type>/<jira-key>-<简短描述>`

### 分支类型

| 类型 | 前缀 | 用途 | 示例 |
|------|------|------|------|
| 新功能 | feature | 迭代内的新需求 | `feature/proj-123-add-user-login` |
| Bug 修复 | bugfix | 常规缺陷修复 | `bugfix/proj-456-fix-order-npe` |
| 热修复 | hotfix | 线上紧急问题，从 master 修复 | `hotfix/proj-789-fix-payment-gateway` |
| 重构/优化 | refactor | 不影响功能的代码改进 | `refactor/proj-999-extract-common-utils` |
| 文档/配置 | docs / chore | 文档、构建脚本、依赖更新 | `docs/proj-101-update-readme` |

**好处**：
- Jira 任务页面自动显示相关分支、Commit 和 PR
- 所有人一看分支名就知道这是哪个需求、什么类型的工作

---

## 二、Commit Message 规范

严格遵循 **Conventional Commits**，但主体描述允许使用中文（中国团队沟通更精准）。

### 标准格式

```
<type>(<scope>): <主体描述>

<正文说明（可选）>

<底部信息（可选）>
```

### type 类型（必填，英文）

| 类型 | 说明 |
|------|------|
| feat | 新功能 |
| fix | 修复 Bug |
| docs | 仅文档变更 |
| style | 代码格式调整（不影响逻辑） |
| refactor | 重构 |
| perf | 性能优化 |
| test | 添加/修改测试 |
| chore | 工具、配置、构建脚本变更 |
| revert | 回滚某次提交 |

### 书写规范

- **scope（可选）**：影响的模块，如 `user`、`order`、`api`、`admin` 等
- **主体描述**：中文，50 字以内，使用祈使语气（"添加"而不是"添加了"），结尾不加句号

### 优秀示例

```
feat(user): 添加手机号注册功能

- 支持短信验证码
- 增加注册来源字段

Jira: PROJ-123
```

```
fix(order): 修复支付回调空指针异常

支付成功后 paymentService 未初始化导致 NPE
增加空值判断并记录日志

Jira: PROJ-456
```

**好处**：
- 自动生成 Changelog
- 配合 semantic-release 可实现自动化版本管理
- 中文描述让团队看 commit 历史时更直观

---

## 三、分支与工作流

### 3.1 分支角色

| 分支 | 角色 | 代码状态 | 谁可以提交 |
|------|------|----------|-----------|
| `master` | 生产分支 | 已上线的稳定代码 | 仅通过 MR 合并 |
| `develop` | 开发集成分支 | 下一版本待发布的代码 | 所有开发者（通过合并） |

### 3.2 分支关系图

```
+------------------+     +------------------+     +------------------+
|     master       |     |     develop      |     |  feature/xxx     |
|  (生产环境)       |     |  (测试环境)       |     |  (开发分支)       |
+------------------+     +------------------+     +------------------+
        ^                        ^                        |
        |                        |                        |
        |    <-- merge           |                        |
        +------------------------+                        |
                  |                                      |
                  |         从 master 创建               |
                  +--------------------------------------+
                  |
                  |         本地开发完成                 |
                  +-------------------------------------->
                            merge 到 develop
```

### 3.3 核心操作规则

#### 从哪个分支拉取？

| 场景 | 拉取来源 | 创建分支 |
|------|---------|----------|
| 新功能开发 | `master` | `feature/xxx` |
| 常规 Bug 修复 | `master` | `bugfix/xxx` |
| 线上紧急修复 | `master` | `hotfix/xxx` |

> **原则**：所有开发分支均从 `master` 创建，保证代码来源清晰

#### 合并到哪个分支？

| 源分支 | 合并目标 | 说明 |
|--------|---------|------|
| `feature/xxx` | `develop` | 提测前合并，触发测试环境发布 |
| `bugfix/xxx` | `develop` | 修复测试环境 Bug |
| `feature/xxx` / `bugfix/xxx` | `master` | 功能验收通过后发布生产环境 |
| `hotfix/xxx` | `master` + `develop` | 紧急修复后同步到两个分支 |

### 3.4 标准开发流程

```
1. 【开始任务】
   git checkout master && git pull
   git checkout -b feature/proj-xxx

2. 【本地开发】
   本地自测通过

3. 【提测】
   git checkout develop && git pull
   git merge feature/proj-xxx
   → 触发测试环境自动部署

4. 【测试发现 Bug】
   git checkout feature/proj-xxx
   修复后重新 merge 到 develop

5. 【准备上线】
   git checkout feature/proj-xxx
   git merge master          # 同步最新生产代码
   解决冲突，检查配置差异
   本地编译 + 单元测试

6. 【合并生产】
   提交 MR 到 master
   Code Review 通过后合并
   → 自动/手动部署生产环境

7. 【同步分支】
   git checkout develop
   git merge master          # develop 同步 master
```

### 3.5 重要规则

| 规则 | 说明 |
|------|------|
| **禁止** | 直接在 `develop` 或 `master` 上提交代码 |
| **禁止** | 绕过 Code Review 直接合并 |
| **必须** | 上线前将 `master` merge 到自己的分支 |
| **必须** | Bug 修复必须在原分支进行，禁止直接在 develop 上修改 |
| **建议** | 开发中途定期 rebase `master` 到自己分支，减少最终冲突 |

### 3.6 分支清理

- 已合并到 `master` 的分支，保留 **1 个月** 后删除
- `develop` 失控时：可直接删除后从 `master` 重建，并通知全团队

---

## 四、上线前检查清单

- [ ] 将 `master` merge 到自己的分支，对比 diff
- [ ] 重点检查：
  - 数据库配置（数据源切换、读写分离）
  - 环境变量、特征开关
  - 日志级别、监控埋点
- [ ] 执行完整 build（compile + test）
- [ ] 提交 MR 并 @ 相关负责人 review
- [ ] 合并后观察生产环境指标

---

## 五、推荐工具与最佳实践

| 场景 | 工具推荐 |
|------|----------|
| 分支对比 | Sourcetree、GitKraken、Kaleidoscope（Mac） |
| 强制 Commit 规范 | husky + commitlint（前端）/ git hook（后端） |

### 最佳实践

- **小步提交**：一次 commit 只做一件事，便于 review 和回滚
- **定期 rebase**：开发中途定期把 master rebase 到自己的分支
- **Code Review**：所有代码进 master 前必须至少 1 人 review

---

## 写在最后

这套规范在我之前的公司使用了 3 年多，配合 Jira + GitLab，整体协作效率很高，线上事故率显著下降。最重要的是：**简单、明确、可执行**，不需要大家记住太多复杂规则。

Git 规范没有绝对完美的，只有最适合自己团队的。建议每半年到一年回顾一次，根据团队规模、项目阶段、工具变化进行微调。
