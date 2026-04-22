## ADDED Requirements

### Requirement: System shall prevent duplicate subscription purchase
The system SHALL prevent users from purchasing the same subscription plan they already have active.

#### Scenario: User attempts to purchase same plan
- **WHEN** user clicks CTA for a plan they already have active
- **THEN** system SHALL block the purchase and display "You already have an active subscription to this plan."

#### Scenario: User with no active subscription can purchase any plan
- **WHEN** user with no active subscription clicks any plan CTA
- **THEN** system SHALL allow the purchase to proceed

### Requirement: System shall determine upgrade vs downgrade
The system SHALL automatically determine whether a plan change is an upgrade or downgrade based on plan priority hierarchy.

#### Scenario: Upgrade detection - higher priority plan
- **WHEN** user with AI Standard (Yearly) attempts to purchase AI Premium (Yearly)
- **THEN** system SHALL identify this as an UPGRADE (Premium has higher priority than Standard)

#### Scenario: Downgrade detection - lower priority plan
- **WHEN** user with AI Premium Family (Yearly) attempts to purchase AI Standard (Yearly)
- **THEN** system SHALL identify this as a DOWNGRADE (Standard has lower priority than Premium Family)

#### Scenario: Cross-group plan change
- **WHEN** user with AI Premium attempts to purchase Video Cloud Standard
- **THEN** system SHALL allow the purchase (different subscription groups can coexist)

### Requirement: Upgrade shall take effect immediately
The system SHALL activate upgraded plans immediately and provide prorated refund for unused time on the old plan.

#### Scenario: Successful upgrade execution
- **WHEN** user confirms upgrade from AI Standard to AI Premium
- **THEN** system SHALL call Stripe API to update subscription immediately
- **AND** system SHALL set `proration_behavior` to `always_invoice`
- **AND** new plan benefits SHALL be available immediately
- **AND** unused portion of old plan SHALL be credited automatically by Stripe

#### Scenario: Upgrade confirmation dialog
- **WHEN** system detects an upgrade attempt
- **THEN** system SHALL display confirmation dialog with:
  - Title: "Confirm Plan Change"
  - Message: "Your new plan will take effect immediately. The unused portion of your current plan will be automatically credited."
  - Buttons: "Confirm" and "Cancel"

### Requirement: Downgrade shall take effect at next billing cycle
The system SHALL schedule downgrades to take effect at the end of the current billing period with no refund.

#### Scenario: Successful downgrade scheduling
- **WHEN** user confirms downgrade from AI Premium Family to AI Premium
- **THEN** system SHALL schedule the downgrade for the next billing cycle
- **AND** system SHALL NOT refund current billing period
- **AND** current plan benefits SHALL remain active until billing cycle ends
- **AND** new plan SHALL automatically activate at next billing date

#### Scenario: Downgrade confirmation dialog
- **WHEN** system detects a downgrade attempt
- **THEN** system SHALL display confirmation dialog with:
  - Title: "Confirm Plan Change"
  - Message: "Your new plan will begin on [Next Billing Date]. No refund applies to the current billing period."
  - Buttons: "Continue" and "Cancel"

#### Scenario: Downgrade cancellation
- **WHEN** user has a scheduled downgrade and requests to cancel it
- **THEN** system SHALL cancel the scheduled downgrade
- **AND** current subscription SHALL continue unchanged

### Requirement: System shall handle device binding on downgrade
The system SHALL release device bindings when AI Family plan downgrades to a plan with fewer device slots.

#### Scenario: AI Family downgrade with 3+ devices
- **WHEN** user with AI Family (3 device bindings) downgrades to AI Standard (1 device limit)
- **THEN** system SHALL notify user that device bindings will be released on next billing date
- **AND** at billing cycle end, all device bindings SHALL be released
- **AND** user SHALL be prompted to select which device to bind under new plan

#### Scenario: AI Family downgrade with 1 device
- **WHEN** user with AI Family (1 device binding) downgrades to AI Standard
- **THEN** system SHALL automatically bind the current device under new plan
- **AND** no user selection SHALL be required

### Requirement: System shall support Apple Pay and Google Pay
The system SHALL allow users to pay using Apple Pay or Google Pay for both initial and subsequent payments.

#### Scenario: First-time payment with Apple Pay
- **WHEN** user selects Apple Pay as payment method for first subscription
- **THEN** system SHALL initiate Apple Pay checkout flow
- **AND** subscription SHALL be created upon successful Apple Pay authorization

#### Scenario: Upgrade payment with Google Pay
- **WHEN** user with existing subscription upgrades to higher plan using Google Pay
- **THEN** system SHALL process upgrade payment via Google Pay
- **AND** subscription update SHALL reflect immediately

### Requirement: System shall sync subscription state via Webhook
The system SHALL synchronize local subscription state with Stripe via Webhook events.

#### Scenario: Subscription updated webhook
- **WHEN** Stripe sends `customer.subscription.updated` webhook event
- **THEN** system SHALL update local subscription record with new plan details
- **AND** system SHALL record upgrade/downgrade timestamp
- **AND** webhook handler SHALL be idempotent (safe to receive multiple times)

#### Scenario: Subscription deleted webhook with pending downgrade
- **WHEN** Stripe sends `customer.subscription.deleted` webhook (billing cycle ended)
- **AND** user has a scheduled downgrade
- **THEN** system SHALL create new subscription with target plan
- **AND** system SHALL clear the downgrade schedule record

#### Scenario: Payment failed webhook
- **WHEN** Stripe sends `invoice.payment_failed` webhook
- **AND** subscription status is `incomplete`
- **THEN** system SHALL cancel the incomplete subscription
- **AND** system SHALL notify user of payment failure

### Requirement: System shall enforce button debounce
The system SHALL prevent accidental double-clicks on all Paywall CTA buttons.

#### Scenario: Rapid button clicks
- **WHEN** user clicks CTA button multiple times within 1.5 seconds
- **THEN** system SHALL only process the first click
- **AND** subsequent clicks SHALL be ignored

### Requirement: System shall expose upgrade check API
The system SHALL provide an API endpoint to check upgrade/downgrade status before purchase.

#### Scenario: Check upgrade status API call
- **WHEN** frontend calls `GET /api/subscription/check-upgrade?targetPlanId=<id>`
- **THEN** system SHALL return JSON with:
  - `status`: "same_plan" | "upgrade" | "downgrade" | "new_subscription"
  - `currentPlan`: current plan details (if any)
  - `targetPlan`: target plan details
  - `nextBillingDate`: for downgrades, the date when change takes effect

### Requirement: System shall expose upgrade execution API
The system SHALL provide an API endpoint to execute plan upgrades.

#### Scenario: Upgrade API call
- **WHEN** frontend calls `POST /api/subscription/upgrade` with `targetPlanId`
- **THEN** system SHALL validate user has active subscription
- **AND** system SHALL call Stripe API to update subscription
- **AND** system SHALL return new subscription details

### Requirement: System shall expose downgrade scheduling API
The system SHALL provide an API endpoint to schedule plan downgrades.

#### Scenario: Schedule downgrade API call
- **WHEN** frontend calls `POST /api/subscription/schedule-downgrade` with `targetPlanId`
- **THEN** system SHALL validate user has active subscription
- **AND** system SHALL create downgrade schedule record in database
- **AND** system SHALL return scheduled downgrade details

#### Scenario: Cancel downgrade API call
- **WHEN** frontend calls `DELETE /api/subscription/schedule-downgrade`
- **THEN** system SHALL remove pending downgrade schedule
- **AND** system SHALL return success confirmation

### Requirement: System shall validate plan priority configuration
The system SHALL ensure plan priority configuration is valid and consistent.

#### Scenario: Duplicate priority detection
- **WHEN** admin configures plan priorities with duplicate values in same group
- **THEN** system SHALL reject the configuration
- **AND** system SHALL return validation error

#### Scenario: Priority-based comparison
- **WHEN** system compares two plans for upgrade/downgrade detection
- **THEN** system SHALL use configured priority values from database
- **AND** higher priority value SHALL indicate higher-tier plan
