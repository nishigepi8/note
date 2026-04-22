## 1. Subscription Group Configuration

- [ ] 1.1 Create SubscriptionGroup enum with AI, VC, Care groups
- [ ] 1.2 Create SubscriptionPlanPriority configuration with plan-to-priority mapping
- [ ] 1.3 Implement SubscriptionGroupService to identify plan groups and priorities
- [ ] 1.4 Add Stripe price ID to plan mapping configuration

## 2. Upgrade Implementation

- [ ] 2.1 Add upgrade detection logic in StripePayService
- [ ] 2.2 Implement upgrade API call to Stripe (subscription update with proration)
- [ ] 2.3 Add proration preview endpoint support
- [ ] 2.4 Implement immediate plan benefit activation after upgrade
- [ ] 2.5 Add trial period check to block upgrades during trial

## 3. Downgrade Implementation

- [ ] 3.1 Add downgrade detection logic in StripePayService
- [ ] 3.2 Implement downgrade API call to Stripe (subscription update without proration)
- [ ] 3.3 Implement scheduled downgrade at next billing cycle
- [ ] 3.4 Add AI Family downgrade device rebinding logic
- [ ] 3.5 Handle Care downgrade without device requirements

## 4. Confirmation Dialog Handling

- [ ] 4.1 Add plan change confirmation API endpoint
- [ ] 4.2 Implement upgrade confirmation response with proper messaging
- [ ] 4.3 Implement downgrade confirmation response with next billing date
- [ ] 4.4 Add button debounce (1.5s) on frontend (coordinated with mobile team)

## 5. Webhook Processing

- [ ] 5.1 Add customer.subscription.updated webhook handler
- [ ] 5.2 Implement subscription status sync logic
- [ ] 5.3 Handle device rebinding when downgrade takes effect
- [ ] 5.4 Add webhook idempotency checks

## 6. Existing Subscription Check

- [ ] 6.1 Implement check for existing subscription in same group
- [ ] 6.2 Add error message for duplicate plan purchase attempt
- [ ] 6.3 Add validation for same-plan selection

## 7. Testing

- [ ] 7.1 Write unit tests for subscription group priority logic
- [ ] 7.2 Write unit tests for upgrade/downgrade detection
- [ ] 7.3 Write integration tests for Stripe API calls
- [ ] 7.4 Test webhook processing scenarios
