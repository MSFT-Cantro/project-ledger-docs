# Pricing Plan Integration Plan

## Overview
This document outlines the implementation plan for a two-tier pricing system in Project Ledger with Free and Paid plans, including limitations and access controls.

## Plan Structure

### Free Plan (Starter)
- **Clients**: Maximum 25 clients
- **Projects**: Maximum 25 projects  
- **Quotes**: Maximum 25 quotes
- **Invoices**: Maximum 25 invoices
- **User Accounts**: Maximum 5 user accounts
- **Inventory**: Not accessible (feature disabled)
- **Storage**: Basic cloud storage
- **Support**: Community support only

### Paid Plan (Professional)
- **Clients**: Unlimited
- **Projects**: Unlimited
- **Quotes**: Unlimited
- **Invoices**: Unlimited
- **User Accounts**: Unlimited
- **Inventory**: Full access
- **Storage**: Enhanced cloud storage
- **Support**: Priority email support
- **Advanced Features**: All features unlocked

## Technical Implementation

### 1. Database Schema Changes

#### New Tables
```sql
-- Subscription plans table
CREATE TABLE subscription_plans (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(50) UNIQUE NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    billing_cycle ENUM('monthly', 'yearly') NOT NULL,
    features JSON,
    limits JSON,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- User subscriptions table
CREATE TABLE user_subscriptions (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    plan_id INT NOT NULL,
    status ENUM('active', 'inactive', 'cancelled', 'expired') DEFAULT 'active',
    starts_at TIMESTAMP NOT NULL,
    ends_at TIMESTAMP NULL,
    trial_ends_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (plan_id) REFERENCES subscription_plans(id)
);

-- Usage tracking table
CREATE TABLE usage_tracking (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    resource_type ENUM('clients', 'projects', 'quotes', 'invoices', 'users') NOT NULL,
    current_count INT DEFAULT 0,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    UNIQUE KEY unique_user_resource (user_id, resource_type)
);
```

#### Seed Data
```sql
INSERT INTO subscription_plans (name, slug, price, billing_cycle, features, limits) VALUES
('Free', 'free', 0.00, 'monthly', 
 '["basic_features", "community_support"]',
 '{"clients": 25, "projects": 25, "quotes": 25, "invoices": 25, "users": 5, "inventory_access": false}'),
('Professional', 'professional', 29.99, 'monthly',
 '["all_features", "priority_support", "advanced_analytics", "inventory_management"]',
 '{"clients": -1, "projects": -1, "quotes": -1, "invoices": -1, "users": -1, "inventory_access": true}');
```

### 2. Backend Implementation

#### New Services

**SubscriptionService** (`apps/backend/src/services/SubscriptionService.ts`)
```typescript
export class SubscriptionService {
  async getCurrentPlan(userId: number): Promise<SubscriptionPlan>
  async getUserLimits(userId: number): Promise<PlanLimits>
  async checkResourceLimit(userId: number, resourceType: string): Promise<boolean>
  async incrementResourceCount(userId: number, resourceType: string): Promise<void>
  async decrementResourceCount(userId: number, resourceType: string): Promise<void>
  async upgradePlan(userId: number, planId: number): Promise<void>
  async cancelSubscription(userId: number): Promise<void>
}
```

**UsageTrackingService** (`apps/backend/src/services/UsageTrackingService.ts`)
```typescript
export class UsageTrackingService {
  async getCurrentUsage(userId: number): Promise<ResourceUsage>
  async updateResourceCount(userId: number, resourceType: string, count: number): Promise<void>
  async canCreateResource(userId: number, resourceType: string): Promise<boolean>
  async syncResourceCounts(userId: number): Promise<void>
}
```

#### Middleware for Plan Validation

**PlanLimitMiddleware** (`apps/backend/src/middleware/planLimitMiddleware.ts`)
```typescript
export const checkPlanLimit = (resourceType: string) => {
  return async (req: Request, res: Response, next: NextFunction) => {
    const userId = req.user.id;
    const canCreate = await usageTrackingService.canCreateResource(userId, resourceType);
    
    if (!canCreate) {
      return res.status(403).json({
        error: 'Plan limit exceeded',
        message: `You have reached the maximum number of ${resourceType} for your current plan.`,
        upgradeRequired: true
      });
    }
    
    next();
  };
};
```

#### Route Protection Updates

Update existing routes to include plan validation:
- `POST /api/clients` - Check client limit
- `POST /api/projects` - Check project limit  
- `POST /api/quotes` - Check quote limit
- `POST /api/invoices` - Check invoice limit
- `POST /api/users` - Check user limit
- `GET /api/inventory/*` - Check inventory access

#### New API Endpoints

**Subscription Routes** (`apps/backend/src/routes/subscriptions.ts`)
```typescript
// GET /api/subscriptions/current - Get current user's subscription
// GET /api/subscriptions/plans - Get available plans
// GET /api/subscriptions/usage - Get current usage stats
// POST /api/subscriptions/upgrade - Upgrade to paid plan
// POST /api/subscriptions/cancel - Cancel subscription
```

### 3. Frontend Implementation

#### New Components

**PlanLimitModal** (`apps/frontend/src/components/PlanLimitModal.tsx`)
- Shows when user hits plan limits
- Displays current usage vs limits
- Call-to-action to upgrade plan

**UsageIndicator** (`apps/frontend/src/components/UsageIndicator.tsx`)
- Shows usage progress bars for each resource
- Color coding (green < 70%, yellow 70-90%, red > 90%)

**PlanBadge** (`apps/frontend/src/components/PlanBadge.tsx`)
- Displays current plan in header/sidebar

#### Updated Components

**Navigation**
- Hide inventory section for free plan users
- Add plan badge to user menu

**Create Forms**
- Add pre-validation before showing forms
- Show usage warnings near limits

#### New Pages

**Billing Page** (`apps/frontend/src/pages/Billing.tsx`)
- Current plan overview
- Usage statistics
- Plan comparison table
- Upgrade/downgrade options
- Billing history

**Plan Upgrade Page** (`apps/frontend/src/pages/PlanUpgrade.tsx`)
- Plan comparison
- Feature matrix
- Pricing calculator
- Payment integration (future)

#### Context Updates

**SubscriptionContext** (`apps/frontend/src/context/SubscriptionContext.tsx`)
```typescript
interface SubscriptionContextType {
  currentPlan: SubscriptionPlan | null;
  usage: ResourceUsage | null;
  limits: PlanLimits | null;
  canCreateResource: (resourceType: string) => boolean;
  hasFeatureAccess: (feature: string) => boolean;
  isLoading: boolean;
  refreshSubscription: () => Promise<void>;
}
```

### 4. Shared Types

**Subscription Types** (`packages/shared-types/subscriptions/index.ts`)
```typescript
export interface SubscriptionPlan {
  id: number;
  name: string;
  slug: string;
  price: number;
  billingCycle: 'monthly' | 'yearly';
  features: string[];
  limits: PlanLimits;
  isActive: boolean;
}

export interface PlanLimits {
  clients: number; // -1 for unlimited
  projects: number;
  quotes: number;
  invoices: number;
  users: number;
  inventoryAccess: boolean;
}

export interface ResourceUsage {
  clients: number;
  projects: number;
  quotes: number;
  invoices: number;
  users: number;
}

export interface UserSubscription {
  id: number;
  userId: number;
  planId: number;
  status: 'active' | 'inactive' | 'cancelled' | 'expired';
  startsAt: Date;
  endsAt?: Date;
  trialEndsAt?: Date;
}
```

## Implementation Phases

### Phase 1: Database and Backend Foundation (1-2 weeks)
1. Create database migrations for new tables
2. Implement SubscriptionService and UsageTrackingService
3. Create middleware for plan validation
4. Add subscription routes to API
5. Update existing routes with limit checks

### Phase 2: Frontend Integration (1-2 weeks)
1. Create SubscriptionContext
2. Build core components (PlanLimitModal, UsageIndicator, PlanBadge)
3. Update navigation and create forms
4. Implement billing page
5. Add plan upgrade flow

### Phase 3: Testing and Polish (1 week)
1. Unit tests for all services
2. Integration tests for limit enforcement
3. E2E tests for upgrade flows
4. Performance testing
5. UI/UX refinements

### Phase 4: Deployment and Monitoring (1 week)
1. Production deployment
2. Usage analytics setup
3. Error monitoring
4. Performance monitoring
5. User feedback collection

## Technical Considerations

### Performance
- Cache plan limits in memory/Redis for frequently accessed data
- Use database indexes on user_id and resource_type columns
- Implement batch usage updates where possible

### Security
- Validate plan limits on both frontend and backend
- Prevent tampering with usage counts
- Secure subscription upgrade endpoints

### Data Integrity
- Implement usage count sync functionality
- Handle edge cases (concurrent creates, failed transactions)
- Regular usage auditing jobs

### User Experience
- Graceful degradation when limits are reached
- Clear messaging about plan benefits
- Smooth upgrade flow without interruption

## Migration Strategy

### Existing Users
1. All existing users start on Free plan
2. Grace period for users exceeding free limits
3. Migration script to populate initial usage counts
4. Communication plan for plan changes

### Data Migration
```sql
-- Migrate existing users to free plan
INSERT INTO user_subscriptions (user_id, plan_id, status, starts_at)
SELECT id, 1, 'active', NOW() FROM users;

-- Calculate current usage for existing users
INSERT INTO usage_tracking (user_id, resource_type, current_count)
SELECT user_id, 'clients', COUNT(*) FROM clients GROUP BY user_id;
-- Repeat for projects, quotes, invoices, users
```

## Success Metrics

### Business Metrics
- Conversion rate from free to paid plans
- Monthly recurring revenue (MRR)
- Customer lifetime value (CLV)
- Plan upgrade/downgrade rates

### Technical Metrics
- API response times for limit checks
- Database query performance
- System reliability during usage spikes
- Error rates for plan validation

### User Metrics
- User engagement with usage indicators
- Time to upgrade after hitting limits
- User satisfaction with plan features
- Support ticket volume related to plans

## Future Enhancements

### Additional Plan Tiers
- Team plan (middle tier)
- Enterprise plan (custom limits)
- Usage-based pricing options

### Advanced Features
- Plan analytics dashboard
- Automatic plan recommendations
- Team collaboration features
- API rate limiting based on plans

### Integration Opportunities
- Third-party billing systems
- Usage-based alerts
- Plan optimization suggestions
- A/B testing for pricing strategies

## Risk Mitigation

### Technical Risks
- **Database performance**: Implement proper indexing and caching
- **Race conditions**: Use database transactions for usage updates
- **Data inconsistency**: Regular audit and sync jobs

### Business Risks
- **User churn**: Generous free tier limits and clear upgrade value
- **Support burden**: Comprehensive documentation and self-service options
- **Competition**: Regular market analysis and feature updates

### Operational Risks
- **Billing issues**: Robust error handling and manual override capabilities
- **Plan confusion**: Clear communication and simple pricing structure
- **System outages**: Graceful degradation and plan validation bypass options

This comprehensive plan provides a roadmap for implementing the pricing plan integration feature while maintaining system performance, user experience, and business objectives.
