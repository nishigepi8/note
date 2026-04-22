## Why

Currently the Stripe payment integration only supports new subscription purchases. When users want to switch between subscription plans (upgrade to a higher tier or downgrade to a lower tier), there is no implementation. This is a critical feature for the AI/VC/Care Paywall 2.0 release that enables users to manage their subscription plans through Stripe.

## What Changes

- **New Capability**: Implement Stripe subscription upgrade/downgrade logic
  - Define subscription group hierarchy (AI, VC, Care packages with monthly/yearly variants)
  - Implement upgrade logic: immediate effect, prorated refund for unused time
  - Implement downgrade logic: delayed effect at next billing cycle
  - Add confirmation dialog handling for plan change requests
  - Handle device rebinding for AI Family downgrades

## Capabilities

### New Capabilities

- `stripe-subscription-group`: Define subscription groups and plan priorities for upgrade/downgrade logic
  - AI Subscription Group: AI Premium Family > AI Premium > AI Standard (both monthly and yearly)
  - VC Subscription Group: Video Cloud Plus > Video Cloud Standard
  - Care Subscription Group: Care Plus > Care Standard
- `stripe-plan-upgrade`: Handle plan upgrade scenarios with immediate effect and proration
- `stripe-plan-downgrade`: Handle plan downgrade scenarios with delayed effect at next billing cycle
- `stripe-plan-change-confirmation`: Handle plan change confirmation dialogs and validations

### Modified Capabilities

- None - this is a net new capability

## Impact

- **New Files**:
  - Subscription group configuration and priority logic
  - Upgrade/downgrade service methods
  - Webhook handler for subscription updates
- **Modified Files**:
  - StripePayService.java - add upgrade/downgrade methods
  - Stripe webhook handler - process subscription updates
- **External Dependencies**:
  - Stripe API: subscription update endpoint
  - Stripe Webhooks: customer.subscription.updated event
