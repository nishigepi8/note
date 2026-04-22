## ADDED Requirements

### Requirement: Plan upgrade detection
The system SHALL detect when a user is upgrading to a higher tier plan.

#### Scenario: Upgrade detected by higher priority
- **WHEN** current plan priority is lower than target plan priority within same subscription group
- **THEN** classify as upgrade operation

#### Scenario: Same plan - not an upgrade
- **WHEN** user selects the same plan they currently have
- **THEN** classify as same-plan (not upgrade or downgrade)

### Requirement: Immediate upgrade effect
The system SHALL apply upgraded plan benefits immediately upon successful payment.

#### Scenario: Upgrade takes effect immediately
- **WHEN** upgrade payment is successful
- **THEN** immediately activate new plan benefits for the user

### Requirement: Proration for upgrades
The system SHALL calculate and apply prorated refund for unused portion of current subscription.

#### Scenario: Stripe handles proration
- **WHEN** processing an upgrade
- **THEN** rely on Stripe to calculate proration and apply credit/refund

### Requirement: Upgrade - no plan stacking
The system SHALL ensure upgraded plan does not stack with existing subscription.

#### Scenario: Upgrade replaces existing subscription
- **WHEN** processing upgrade
- **THEN** terminate old subscription and activate new subscription (no parallel/overlapping)

### Requirement: Upgrade trial period handling
The system SHALL not allow upgrades for subscriptions in trial period.

#### Scenario: Block upgrade during trial
- **WHEN** user attempts to upgrade while in trial period
- **THEN** display message explaining upgrade will be available after trial ends
