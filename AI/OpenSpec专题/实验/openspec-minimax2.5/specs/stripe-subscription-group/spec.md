## ADDED Requirements

### Requirement: Subscription group hierarchy
The system SHALL define subscription groups with plan priorities for upgrade/downgrade decisions.

#### Scenario: AI subscription group priority ordering
- **WHEN** determining if a plan change is an upgrade or downgrade within AI plans
- **THEN** use the following priority order (highest to lowest):
  1. AI Premium Family (Yearly)
  2. AI Premium (Yearly)
  3. AI Standard (Yearly)
  4. AI Premium Family (Monthly)
  5. AI Premium (Monthly)
  6. AI Standard (Monthly)

#### Scenario: VC subscription group priority ordering
- **WHEN** determining if a plan change is an upgrade or downgrade within VC plans
- **THEN** use the following priority order (highest to lowest):
  1. Video Cloud Plus (Yearly)
  2. Video Cloud Standard (Yearly)
  3. Video Cloud Plus (Monthly)
  4. Video Cloud Standard (Monthly)

#### Scenario: Care subscription group priority ordering
- **WHEN** determining if a plan change is an upgrade or downgrade within Care plans
- **THEN** use the following priority order (highest to lowest):
  1. Care Plus (Yearly)
  2. Care Standard (Yearly)
  3. Care Plus (Monthly)
  4. Care Standard (Monthly)

### Requirement: Same group single subscription
The system SHALL ensure only one active subscription per subscription group at any time.

#### Scenario: Prevent duplicate subscription purchase
- **WHEN** user clicks to purchase a plan they already have active in the same group
- **THEN** display error message: "You already have an active subscription to this plan."

#### Scenario: Check existing subscription before upgrade
- **WHEN** user attempts to upgrade or downgrade
- **THEN** first check for any existing active subscription in the same group

### Requirement: Subscription group identification
The system SHALL map Stripe price IDs to their corresponding subscription groups.

#### Scenario: Identify AI plan from Stripe price ID
- **WHEN** receiving a Stripe price ID
- **THEN** identify if it belongs to AI subscription group and extract plan type and billing cycle

#### Scenario: Identify VC plan from Stripe price ID
- **WHEN** receiving a Stripe price ID
- **THEN** identify if it belongs to VC subscription group and extract plan type and billing cycle

#### Scenario: Identify Care plan from Stripe price ID
- **WHEN** receiving a Stripe price ID
- **THEN** identify if it belongs to Care subscription group and extract plan type and billing cycle
