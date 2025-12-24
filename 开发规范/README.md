# DevOps 开发规范

本目录包含团队开发规范和最佳实践。

## 文章列表

| 文章 | 适用场景 |
|------|----------|
| [Git 分支与 Commit 规范](/post/开发规范/Git%20分支与%20Commit%20规范.md) | 中大型研发团队（100-500人） |

## Git 规范要点

### 分支命名

```
<type>/<jira-key>-<简短描述>
```

- `feature/proj-123-add-user-login` - 新功能
- `bugfix/proj-456-fix-order-npe` - Bug 修复
- `hotfix/proj-789-fix-payment-gateway` - 热修复

### Commit 规范

遵循 Conventional Commits：

```
<type>(<scope>): <主体描述>

<正文说明（可选）>

Jira: PROJ-123
```

**type 类型**：
- `feat`：新功能
- `fix`：修复 Bug
- `docs`：文档变更
- `refactor`：重构
- `perf`：性能优化
- `test`：测试
- `chore`：构建/工具

## 工作流要点

1. 从 `master` 创建 feature 分支
2. 本地自测 → merge 到 `develop` → QA 测试
3. 测试通过 → 将 `master` merge 回 feature 分支
4. 提交 MR，Code Review 通过后合并到 `master`
5. 上线并观察生产指标

## 推荐工具

- **分支对比**：Sourcetree、GitKraken
- **Commit 检查**：husky + commitlint
- **代码审查**：GitLab MR、GitHub PR

