## ADDED Requirements

### Requirement: Plan downgrade detection
The system SHALL detect when a user is downgrading to a lower tier plan.

#### Scenario: Downgrade detected by lower priority
- **WHEN** current plan priority is higher than target plan priority within same subscription group
- **THEN** classify as downgrade operation

### Requirement: Delayed downgrade effect
The system SHALL apply downgraded plan benefits at the next billing cycle.

#### Scenario: Downgrade takes effect next billing cycle
- **WHEN** downgrade is confirmed
- **THEN** new plan becomes effective at the end of current billing period

#### Scenario: Current benefits continue during billing period
- **WHEN** downgrade is confirmed but not yet effective
- **THEN** user continues to enjoy current (higher tier) benefits until billing period ends

### Requirement: No refund for downgrades
The system SHALL not issue refunds for downgrades.

#### Scenario: No proration for downgrades
- **WHEN** processing a downgrade
- **THEN** do not request proration - user pays current rate until next billing cycle

### Requirement: AI Family downgrade device rebinding
The system SHALL handle device rebinding when downgrading from AI Premium Family.

#### Scenario: AI Family downgrade with 3+ devices
- **WHEN** downgrading from AI Premium Family and user has 3 or more devices
- **THEN** prompt user to select devices for the new plan binding

#### Scenario: AI Family downgrade with 2 devices
- **WHEN** downgrading from AI Premium Family and user has 2 devices
- **THEN** prompt user to select devices for the new plan binding

#### Scenario: AI Family downgrade with 1 device
- **WHEN** downgrading from AI Premium Family and user has 1 device
- **THEN** automatically bind to current device at downgrade effective date

### Requirement: Care Plus downgrade device handling
The system SHALL not require device rebinding for Care Plus to Care Standard downgrade.

#### Scenario: Care downgrade - no device binding needed
- **WHEN** downgrading from Care Plus to Care Standard
- **THEN** proceed without device rebinding requirements
