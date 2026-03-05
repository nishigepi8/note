## 1. Database Setup

- [ ] 1.1 Create `subscription_downgrade_schedule` table migration
- [ ] 1.2 Add `stripe_subscription_item_id` column to subscriptions table
- [ ] 1.3 Create `plan_priorities` configuration table
- [ ] 1.4 Seed plan priority data for AI/VC/Care plans

## 2. Backend - Core Infrastructure

- [ ] 2.1 Create plan priority service for upgrade/downgrade detection
- [ ] 2.2 Implement plan comparison logic (same/upgrade/downgrade/new)
- [ ] 2.3 Add Stripe subscription update utility functions
- [ ] 2.4 Create subscription state sync service

## 3. Backend - API Endpoints

- [ ] 3.1 Implement `GET /api/subscription/check-upgrade` endpoint
- [ ] 3.2 Implement `POST /api/subscription/upgrade` endpoint with Stripe integration
- [ ] 3.3 Implement `POST /api/subscription/schedule-downgrade` endpoint
- [ ] 3.4 Implement `DELETE /api/subscription/schedule-downgrade` endpoint
- [ ] 3.5 Add API request validation and error handling
- [ ] 3.6 Add comprehensive logging for upgrade/downgrade operations

## 4. Backend - Webhook Handlers

- [ ] 4.1 Implement `customer.subscription.updated` webhook handler
- [ ] 4.2 Implement `customer.subscription.deleted` webhook with downgrade execution
- [ ] 4.3 Implement `invoice.payment_failed` webhook handler
- [ ] 4.4 Add webhook idempotency protection
- [ ] 4.5 Create webhook event logging and monitoring

## 5. Backend - Device Binding

- [ ] 5.1 Implement device binding release logic for downgrades
- [ ] 5.2 Create device rebind notification service
- [ ] 5.3 Add device count validation on downgrade

## 6. Frontend - UI Components

- [ ] 6.1 Create upgrade confirmation modal component
- [ ] 6.2 Create downgrade confirmation modal component
- [ ] 6.3 Implement duplicate purchase blocking UI
- [ ] 6.4 Add button debounce (1.5s) to all Paywall CTA buttons
- [ ] 6.5 Create downgrade scheduled status display

## 7. Frontend - Integration

- [ ] 7.1 Integrate upgrade check API on Paywall CTA click
- [ ] 7.2 Connect upgrade confirmation to upgrade API
- [ ] 7.3 Connect downgrade confirmation to schedule-downgrade API
- [ ] 7.4 Implement downgrade cancellation UI flow
- [ ] 7.5 Add Apple Pay / Google Pay support to Paywall

## 8. Testing - Backend

- [ ] 8.1 Write unit tests for plan priority service
- [ ] 8.2 Write unit tests for upgrade/downgrade detection logic
- [ ] 8.3 Write integration tests for upgrade API endpoint
- [ ] 8.4 Write integration tests for downgrade scheduling API
- [ ] 8.5 Write webhook handler tests with mock Stripe events
- [ ] 8.6 Test Stripe subscription update with proration
- [ ] 8.7 Test device binding release on downgrade

## 9. Testing - Frontend

- [ ] 9.1 Write component tests for upgrade confirmation modal
- [ ] 9.2 Write component tests for downgrade confirmation modal
- [ ] 9.3 Test button debounce functionality
- [ ] 9.4 Test duplicate purchase blocking flow
- [ ] 9.5 Test Apple Pay / Google Pay integration

## 10. Deployment Preparation

- [ ] 10.1 Create feature flag for upgrade/downgrade functionality
- [ ] 10.2 Set up monitoring dashboards for subscription changes
- [ ] 10.3 Configure Stripe webhook endpoint in production
- [ ] 10.4 Create rollback runbook
- [ ] 10.5 Document API endpoints for frontend team

## 11. Deployment & Monitoring

- [ ] 11.1 Deploy database migrations to staging
- [ ] 11.2 Deploy backend API to staging
- [ ] 11.3 Deploy frontend changes to staging
- [ ] 11.4 Execute end-to-end testing on staging
- [ ] 11.5 Enable feature flag for 10% users (AI Paywall)
- [ ] 11.6 Monitor error rates and conversion metrics
- [ ] 11.7 Gradually roll out to 100% users across all Paywalls
- [ ] 11.8 Set up alerts for webhook failures
