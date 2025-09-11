# Migration to Account-Based Subscriptions

## Overview
This document outlines the migration from per-user subscriptions to account-wide subscriptions in Project Ledger.

## Key Changes

### Database Schema Changes
1. **Added `AccountSubscription` model** - Replaces `UserSubscription` for account-wide subscriptions
2. **Updated `SubscriptionTransaction`** - Now references `accountId` instead of `userId`
3. **Added relations** - Account now has AccountSubscription and SubscriptionTransaction relations
4. **Deprecated `UserSubscription`** - Commented out but kept temporarily for data migration

### Backend API Changes
1. **Updated `/api/subscriptions/current`** - Now returns account subscription instead of user subscription
2. **Updated `/api/subscriptions/usage`** - Now aggregates usage across all users in the account
3. **Updated `/api/subscriptions/cancel`** - Now cancels account subscription
4. **Updated middleware** - Plan limit checks now work at account level

### Frontend Type Changes
1. **Added `AccountSubscription` interface** - New type for account-based subscriptions
2. **Updated `SubscriptionData`** - Now uses `AccountSubscription` instead of `UserSubscription`
3. **Maintained compatibility** - Frontend components work the same but now operate on account data

### Shared Types Updates
1. **Added `AccountSubscription`** - Available in shared types package
2. **Updated imports** - Backend now imports `AccountSubscription` instead of `UserSubscription`

## Business Impact

### Before (Per-User Model)
- Each user needs individual $99.99/month subscription
- Company with 5 users = $499.95/month total
- No shared benefits across team members
- Individual billing management per user

### After (Account-Based Model)
- One subscription covers entire account/company
- Company with 5 users = $99.99/month total
- All users in account get Professional features
- Centralized billing management
- Much more competitive for B2B customers

## Technical Implementation

### Usage Tracking
- **Before**: Tracked per individual user
- **After**: Aggregated across all users in the account
- Usage limits apply to entire account, not individual users

### Plan Limits
- **Before**: Each user had individual limits
- **After**: Account-wide limits shared across all users
- Example: 1000 client limit shared by all users in the account

### Subscription Management
- **Before**: Each user managed their own subscription
- **After**: Account-level subscription management
- Any user in the account can see subscription status
- Subscription affects all users in the account

## Migration Strategy

### Database Migration
1. **Schema migration** - Added AccountSubscription table and relations
2. **Data migration** - Convert existing UserSubscriptions to AccountSubscriptions
3. **Cleanup** - Remove UserSubscription references after migration complete

### API Compatibility
- All existing API endpoints maintained
- Response format unchanged for frontend compatibility
- Internal logic changed to use account-based data

### Frontend Compatibility
- No changes required to existing components
- Subscription context works the same way
- User experience remains consistent

## Testing
1. **Created test data** - Professional plan and account subscription
2. **Verified API endpoints** - All subscription endpoints work with account model
3. **Confirmed usage tracking** - Aggregates correctly across account users

## Benefits
1. **Better B2B fit** - Matches how businesses expect SaaS subscriptions to work
2. **Improved economics** - More reasonable pricing for team usage
3. **Simplified billing** - One subscription per company instead of per user
4. **Team collaboration** - All users get access with one subscription
5. **Easier onboarding** - Add team members without additional subscriptions

## Next Steps
1. **Test thoroughly** - Verify all subscription flows work correctly
2. **Update documentation** - Reflect account-based subscription model
3. **Marketing update** - Update pricing pages to reflect new model
4. **Customer communication** - Notify existing customers of the change (benefit to them)
