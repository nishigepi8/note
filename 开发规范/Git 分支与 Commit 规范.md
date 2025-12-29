# 中大型团队的 Git 实践

## 前言

这是一份经过实际项目打磨、适合中大型研发团队（100-500 人）的 Git 使用规范。目标是：**保持代码历史清晰、可追溯、易协作，同时降低合并冲突和上线风险**。

适用于后端、前端、移动端等多端共用仓库的项目。如果你的团队也在寻找一套简单、可落地、不折腾的 Git 规范，这份可以直接拿来参考或稍作调整后使用。

**更新时间**：2025 年 12 月  
**参考依据**：GitFlow 变体 + Conventional Commits + Jira 集成实践

## 一、分支命名规范

统一使用小写 + 连字符，强制包含 Jira 任务号（或工单号），便于自动化追踪。

**格式**：`<type>/<jira-key>-<简短描述>`

**常见类型**：

| 类型       | 前缀       | 用途                              | 示例                                      |
|------------|------------|-----------------------------------|-------------------------------------------|
| 新功能     | feature    | 迭代内的新需求                    | `feature/proj-123-add-user-login`         |
| Bug 修复   | bugfix     | 常规缺陷修复                      | `bugfix/proj-456-fix-order-npe`           |
| 热修复     | hotfix     | 线上紧急问题，直接从 master 修复  | `hotfix/proj-789-fix-payment-gateway`     |
| 重构/优化  | refactor   | 不影响功能的代码改进              | `refactor/proj-999-extract-common-utils`  |
| 文档/配置  | docs / chore | 文档、构建脚本、依赖更新等        | `docs/proj-101-update-readme`             |

**好处**：
- Jira 任务页面自动显示相关分支、Commit 和 PR。
- 所有人一看分支名就知道这是哪个需求、什么类型的工作。

## 二、Commit Message 规范

严格遵循 **Conventional Commits**，但主体描述允许使用中文（中国团队沟通更精准）。

**标准格式**：

```
<type>(<scope>): <主体描述>

<正文说明（可选）>

<底部信息（可选）>
```

**type 类型（必填，英文）**：

- `feat`：新功能
- `fix`：修复 Bug
- `docs`：仅文档变更
- `style`：代码格式调整（不影响逻辑）
- `refactor`：重构
- `perf`：性能优化
- `test`：添加/修改测试
- `chore`：工具、配置、构建脚本变更
- `revert`：回滚某次提交

**scope（可选）**：影响的模块，如 `user`、`order`、`api`、`admin` 等。

**主体描述**：中文，50 字以内，使用祈使语气（“添加”而不是“添加了”），结尾不加句号。

**优秀示例**：

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

## 三、核心分支与工作流

**主分支**：
- `master`：生产分支，只存放已上线的稳定代码
- `develop`：集成分支，用于日常测试环境发布（部分团队称 `zelda` 或 `integration`）

**日常开发流程**：

1. **开始新任务**：先 pull 最新 `master`，然后从 `master` 创建 feature 或 bugfix 分支。
2. **本地开发自测通过** → merge 到 `develop` 分支 → 发布测试环境供 QA 测试。
3. **测试发现 Bug**：必须回原 feature/bugfix 分支修复，再重新 merge 到 `develop`。  
   **严禁直接在 develop 上修改代码或提交 commit**。
4. **功能测试通过准备上线**：
   - 将最新 `master` merge 到自己的分支，解决冲突
   - 重点检查配置差异（如数据源、读写分离，线上必须走 slave 库）
   - 本地编译、运行单元测试、启动项目，确保一切正常
5. 提交 Pull Request / Merge Request 到 `master`
6. Code Review 通过后合并
7. 上线

**分支清理**：已合并的分支保留 1 个月后删除，避免仓库臃肿。

**develop 失控时**：如果 develop 与 master 差异过大，可直接删除 develop，从 master 重新创建，并通知全团队。

## 四、上线前必做检查清单

1. 将 master merge 到自己的分支，对比 diff
2. 重点检查：
   - 数据库配置（数据源切换、读写分离）
   - 环境变量、特征开关
   - 日志级别、监控埋点
3. 执行完整 build（compile + test）
4. 提交 MR 并 @ 相关负责人 review
5. 合并后观察生产环境指标

## 五、推荐工具与最佳实践

- **分支对比**：Sourcetree、GitKraken、Kaleidoscope（Mac）
- **强制 Commit 规范**：引入 husky + commitlint（前端）或 git hook（后端）
- **小步提交**：一次 commit 只做一件事，便于 review 和回滚
- **定期 rebase**：开发中途定期把 master 或 develop rebase 到自己的分支，减少最终合并冲突
- **Code Review 必做**：所有代码进 master 前必须至少 1 人 review

## 写在最后

这套规范在我之前的公司使用了 3 年多，配合 Jira + GitLab，整体协作效率很高，线上事故率显著下降。最重要的是：**简单、明确、可执行**，不需要大家记住太多复杂规则。

Git 规范没有绝对完美的，只有最适合自己团队的。建议每半年到一年回顾一次，根据团队规模、项目阶段、工具变化进行微调。

如果你正在制定或优化团队的 Git 规范，欢迎交流你的实践经验～