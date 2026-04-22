## Context

**当前状态**：
- 系统已实现 Stripe Checkout 首次订阅（AI/VC/Care Paywall）
- 用户可通过 Stripe 购买年付/月付套餐，年付套餐支持 7 天免费试用
- 目前缺少订阅变更能力，用户无法升级或降级套餐

**约束条件**：
- 必须符合 Stripe 订阅管理 API 规范
- 套餐有严格的优先级排序（见 PRD 3.1 需求3）
- 同一 Subscription Group 内同一时间仅允许 1 个有效订阅
- 升降级必须在前端做拦截，后台判断，Stripe API 执行

**利益相关者**：前端 Paywall 团队、后端订阅服务团队、财务团队（退款逻辑）

## Goals / Non-Goals

**Goals**:
- 实现套餐升级：立即生效，旧套餐按比例退款
- 实现套餐降级：延迟到下一计费周期生效，无退款
- 前端拦截重复购买，展示升降级确认弹窗
- 后台根据套餐优先级自动判断升级/降级
- Stripe Webhook 同步订阅状态变更
- 支持 Apple Pay/Google Pay 作为支付方式

**Non-Goals**:
- Web Portal 订阅取消功能（需求5，但不在本次升降级范围内）
- 兑换码兑换逻辑（已存在，不涉及）
- 套餐价格调整（价格由 Stripe Product 配置决定）

## Decisions

### 1. 升降级判断策略

**决策**：在用户点击新套餐 CTA 时，后端判断当前订阅与新套餐的关系

**方案**：
- 前端调用 `GET /api/subscription/check-upgrade` 传入 `targetPlanId`
- 后端查询用户当前有效订阅，根据优先级表判断：
  - 相同套餐 → 返回 `same_plan`，前端拦截
  - 目标套餐优先级更高 → 返回 `upgrade`，展示升级确认弹窗
  - 目标套餐优先级更低 → 返回 `downgrade`，展示降级确认弹窗

**替代方案**：
- ~~前端本地判断~~：不安全，用户可绕过前端拦截
- ~~Stripe 判断~~：Stripe 不了解业务套餐优先级，需后端维护

### 2. 升级执行方案

**决策**：调用 Stripe `subscriptions.update` API，设置 `proration_behavior: 'always_invoice'`

**方案**：
1. 用户确认升级 → 调用 `POST /api/subscription/upgrade`
2. 后端调用 Stripe API：
   ```javascript
   stripe.subscriptions.update(subscriptionId, {
     items: [{ id: itemId, price: newPriceId }],
     proration_behavior: 'always_invoice', // 立即按比例退款并收费
     payment_behavior: 'pending_if_incomplete'
   })
   ```
3. Stripe 自动计算退款金额并生成发票
4. Webhook 监听 `customer.subscription.updated`，同步本地订阅状态

**替代方案**：
- ~~先取消再创建新订阅~~：会丢失订阅历史，不符合 Stripe 最佳实践
- ~~手动计算退款~~：容易出错，Stripe 的 proration 已经过生产验证

### 3. 降级执行方案

**决策**：调用 Stripe `subscriptions.update` API，设置 `proration_behavior: 'none'` + `cancel_at_period_end`

**方案**：
1. 用户确认降级 → 调用 `POST /api/subscription/schedule-downgrade`
2. 后端记录降级预约信息到数据库：
   ```sql
   INSERT INTO subscription_downgrade_schedule 
   (user_id, current_plan_id, target_plan_id, scheduled_at)
   VALUES (...)
   ```
3. 当前订阅周期结束时，Webhook 监听 `customer.subscription.deleted`
4. 后端检测到有降级预约 → 调用 Stripe 创建新订阅（目标套餐）

**替代方案**：
- ~~使用 Stripe Subscription Schedules~~：功能更强大但复杂度高，当前需求用不上
- ~~降级立即生效~~：不符合 PRD 要求，降级需延迟到周期结束

### 4. 前端二次确认弹窗

**决策**：在支付前展示模态弹窗，用户必须点击 "Confirm" 才能继续

**方案**：
- 升级弹窗文案：
  - 标题：Confirm Plan Change
  - 内容：Your new plan will take effect immediately. The unused portion of your current plan will be automatically credited.
  - 按钮：Confirm / Cancel
- 降级弹窗文案：
  - 标题：Confirm Plan Change
  - 内容：Your new plan will begin on [Next Billing Date]. No refund applies to the current billing period.
  - 按钮：Continue / Cancel

**替代方案**：
- ~~Toast 提示~~：容易被忽略，升降级是重要决策需要明确确认
- ~~Stripe Checkout 页面展示~~：Stripe Checkout 不支持自定义确认文案

### 5. 设备绑定降级处理（AI Family）

**决策**：降级预约时，提醒用户下个周期需重新绑定设备

**方案**：
- AI Family 降级到其他套餐时，在确认弹窗中额外提示：
  - "Your current device bindings will be released on [date]. You'll need to rebind your devices under the new plan."
- 当前周期结束 + 降级生效时，自动解绑所有设备
- 用户下次登录时，提示重新选择设备绑定

**替代方案**：
- ~~立即解绑~~：不符合降级延迟生效原则
- ~~保留绑定~~：新套餐可能设备数量限制不同，会导致数据不一致

## Risks / Trade-offs

### Risk 1: Stripe Webhook 延迟或失败

**风险**：订阅更新后 Webhook 延迟到达或丢失，导致本地状态不一致

**缓解措施**：
- 实现幂等性处理，Webhook 重复调用不会产生副作用
- 添加定时任务，每 5 分钟轮询 Stripe 订阅状态，对比本地状态并同步
- 关键操作（升级、降级预约）在调用 Stripe API 后立即记录到数据库，Webhook 仅用于状态同步

### Risk 2: 降级预约在周期结束前用户取消订阅

**风险**：用户预约降级后，在周期结束前手动取消订阅，导致降级预约失效

**缓解措施**：
- 监听 `customer.subscription.deleted` 事件，检查是否有降级预约
- 如有预约，记录日志并通知运营团队（降级预约失效属于边缘 case）
- 前端展示降级预约状态时，明确提示"如果您在周期结束前取消订阅，降级将不会生效"

### Risk 3: Apple Pay/Google Pay 支付失败

**风险**：用户选择 Apple Pay/Google Pay 后，支付失败但订阅已创建

**缓解措施**：
- 设置 `payment_behavior: 'pending_if_incomplete'`，订阅创建但状态为 `incomplete`
- Webhook 监听 `invoice.payment_failed`，如果支付失败则自动取消订阅
- 前端展示支付失败提示，引导用户重试或选择其他支付方式

### Risk 4: 套餐优先级配置错误

**风险**：套餐优先级配置错误，导致升降级判断逻辑异常

**缓解措施**：
- 优先级配置存储在数据库，支持热更新（无需发版）
- 添加配置校验逻辑，确保同一 Subscription Group 内优先级唯一
- 升降级判断时记录详细日志，方便排查问题

## Migration Plan

**阶段 1：数据库准备**（无停机）
- 新增 `subscription_downgrade_schedule` 表
- 订阅表新增字段：`stripe_subscription_item_id`（记录 Stripe subscription item ID，用于升降级）

**阶段 2：后端 API 上线**（无停机）
- 上线升降级判断 API：`GET /api/subscription/check-upgrade`
- 上线升级 API：`POST /api/subscription/upgrade`
- 上线降级预约 API：`POST /api/subscription/schedule-downgrade`
- 上线 Webhook 处理逻辑

**阶段 3：前端上线**（灰度发布）
- AI Paywall 灰度 10% 用户
- VC Paywall 灰度 10% 用户
- Care Paywall 灰度 10% 用户
- 监控转化率、错误率，逐步扩大灰度

**回滚策略**：
- 前端回滚：关闭功能开关，用户无法看到升降级功能
- 后端回滚：API 返回 503，前端 fallback 到原有购买流程
- 数据库回滚：无需回滚，新表/字段向后兼容

## Open Questions

1. **降级预约的取消**：用户预约降级后，能否取消预约？（建议：可以，提供 `DELETE /api/subscription/schedule-downgrade` API）
2. **升降级历史记录**：是否需要在前端展示升降级历史？（建议：暂时不做，后续迭代）
3. **降级生效通知**：降级生效时，是否需要邮件/推送通知用户？（建议：需要，避免用户困惑）
