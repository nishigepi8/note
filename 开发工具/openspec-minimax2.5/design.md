## Context

The current Stripe integration only supports new subscription purchases. The PRD requires implementing subscription upgrade/downgrade functionality for AI Paywall, VC Paywall, and Care Paywall. This involves:

1. **Subscription Group Hierarchy**: Define which plans belong to the same group and their priority for upgrade/downgrade decisions
2. **Upgrade Logic**: Immediate effect with prorated refund
3. **Downgrade Logic**: Delayed effect at next billing cycle
4. **Device Rebinding**: For AI Family downgrades, handle device rebinding

## Goals / Non-Goals

**Goals:**
- Implement Stripe subscription upgrade logic (immediate effect, prorated refund)
- Implement Stripe subscription downgrade logic (delayed at next billing cycle)
- Define subscription group priorities for AI, VC, and Care packages
- Handle confirmation dialog flow for plan changes
- Process Stripe webhook events for subscription updates

**Non-Goals:**
- Web Portal subscription management (out of scope)
- Apple Pay / Google Pay specific upgrade flows (handled generically)
- Device binding for new purchases (existing logic)
- Refund processing UI (Stripe handles automatically)

## Decisions

### 1. Subscription Group Configuration

**Decision**: Use configuration-based approach to define subscription groups and priorities.

**Rationale**: Easy to modify priorities without code changes. Allows adding new plans easily.

**Alternatives Considered**:
- Hardcoded enum: Too rigid, requires code changes for new plans
- Database-driven: Over-engineered for this use case

**Subscription Groups from PRD:**
- AI: AI Premium Family (Yearly) > AI Premium (Yearly) > AI Standard (Yearly) > AI Premium Family (Monthly) > AI Premium (Monthly) > AI Standard (Monthly)
- VC: Video Cloud Plus (Yearly) > Video Cloud Standard (Yearly) > Video Cloud Plus (Monthly) > Video Cloud Standard (Monthly)
- Care: Care Plus (Yearly) > Care Standard (Yearly) > Care Plus (Monthly) > Care Standard (Monthly)

### 2. Upgrade Implementation

**Decision**: Use Stripe's subscription update API with proration behavior.

**Rationale**: Stripe natively supports proration for upgrades. We just need to call the update API with the new price ID.

**Flow:**
1. User clicks upgrade CTA → Show confirmation dialog
2. Call Stripe API to update subscription with new price
3. Stripe handles proration and charges difference
4. Webhook receives subscription.updated event
5. Update local subscription status

### 3. Downgrade Implementation

**Decision**: Use Stripe's subscription update API with proration_behavior=none and update the subscription schedule.

**Rationale**: Stripe supports delayed proration - the new price takes effect at next billing cycle.

**Flow:**
1. User clicks downgrade CTA → Show confirmation dialog with next billing date
2. Call Stripe API to update subscription with new price (proration_behavior=none)
3. Current subscription continues until end of period
4. Webhook receives subscription.updated event at billing cycle change
5. Update local subscription to new tier

### 4. Device Rebinding for AI Family Downgrade

**Decision**: Handle device rebinding at the time of downgrade confirmation, not at billing cycle change.

**Rationale**: User needs to select devices before downgrade takes effect. Prompt user during confirmation.

**Flow:**
1. Detect downgrade from AI Family to lower tier
2. Check user's available devices
3. If devices >= 3 or 2: prompt user to select devices for new plan
4. If device = 1: auto-bind to current device

## Risks / Trade-offs

**[Risk]**: Race condition when user has multiple active subscriptions
**Mitigation**: Check for existing subscriptions in the same group before allowing purchase/upgrade

**[Risk]**: Webhook processing failure
**Mitigation**: Implement idempotency checks and retry logic for webhook handlers

**[Risk]**: Proration calculation discrepancy
**Mitigation**: Use Stripe's preview endpoint to show proration amount before confirming upgrade

**[Risk]**: Trial period handling during upgrade
**Mitigation**: Disable upgrade for subscriptions in trial period until trial ends
