## Why

目前系统支持 Stripe Checkout 首次订阅，但缺少套餐升降级能力。用户想要变更套餐时（如从月付升级到年付、从 Standard 升级到 Premium）无法平滑过渡。需要实现完整的订阅升降级策略，符合 Stripe 订阅管理最佳实践，提升用户订阅体验。

## What Changes

- **新增套餐升级功能**：用户升级套餐时立即生效，旧套餐未使用费用按比例折算退款
- **新增套餐降级功能**：用户降级时延迟到下一计费周期生效，当前周期继续享受原权益
- **前端拦截策略**：相同套餐不可重复购买，提示用户 "You already have an active subscription to this plan"
- **升降级二次确认弹窗**：支付前展示确认弹窗，明确告知用户升级/降级的影响
- **后台升降级判断逻辑**：根据套餐优先级表判断是升级还是降级
- **Stripe Webhook 集成**：监听订阅更新事件，同步本地订阅状态
- **支持 Apple Pay/Google Pay**：首次和二次支付都支持 Apple Pay 和 Google Pay

## Capabilities

### New Capabilities

- `stripe-subscription-upgrade-downgrade`: Stripe 订阅升降级核心功能，包括升级/降级策略判断、费用折算、权益切换、二次确认弹窗

### Modified Capabilities

<!-- 无现有 capabilities 需要修改 -->

## Impact

**后端**:
- 订阅管理服务：新增升降级逻辑、费用折算、权益管理
- Stripe API 集成：调用 Stripe 订阅更新 API (subscriptions.update)
- Webhook 处理：监听 `customer.subscription.updated` 等事件

**前端**:
- Paywall 页面：展示升降级确认弹窗、拦截重复购买
- 套餐对比：根据当前订阅状态动态展示 CTA

**数据库**:
- 订阅表：可能需要新增字段记录升降级历史、降级预约信息

**业务逻辑**:
- 套餐优先级排序（AI/VC/Care 三类套餐各有优先级）
- 升级：立即生效 + 按比例退款
- 降级：延迟生效 + 无退款
- 设备绑定处理（AI Family 降级时需重新绑定设备）
