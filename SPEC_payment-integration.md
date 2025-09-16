# PayPal Integration Implementation - Active Development

## ✅ IMPLEMENTATION STATUS: PHASE 1 STARTING

**Last Updated**: September 16, 2025  
**Current Status**: Phase 1 (PayPal Backend Integration) - Ready to implement  
**Prerequisites**: ✅ Pricing plan integration complete, ✅ Account-based subscription system operational  
**Next Phase**: PayPal payment processing integration to enable credit card functionality in billing page

## Overview
This document outlines the implementation plan for integrating PayPal payment processing to handle subscription payments for Project Ledger's pricing plans. This integration will enable users to upgrade from free to paid plans and manage their billing through PayPal's robust payment system.

**Building Upon**: Complete account-based subscription system with Free ($0) and Professional ($99.99) plans already operational

## PayPal Integration Requirements

### Core Features
- **Subscription Management**: Create, update, and cancel PayPal subscriptions
- **Payment Processing**: Secure credit card, PayPal wallet, and bank payment processing
- **Billing Management**: Invoice generation, payment history, and receipts
- **Webhook Handling**: Real-time subscription status updates from PayPal
- **Tax Calculation**: Automatic tax calculation based on location
- **Multiple Payment Methods**: Credit cards, PayPal balance, bank transfers

### Security & Compliance
- PCI DSS compliance through PayPal
- Secure payment tokenization
- Strong Customer Authentication (SCA) compliance
- Data encryption in transit and at rest
- PayPal's fraud protection

## Technical Architecture

### 1. PayPal Account Setup

#### PayPal Business Account Configuration
- **Business Type**: SaaS Platform
- **Products**: 
  - Professional Plan Monthly ($99.99/month)
  - Professional Plan Yearly ($999.99/year) - 17% discount
- **Tax Settings**: Enable automatic tax collection
- **Webhooks**: Configure for subscription events
- **API Access**: Enable REST API access for subscriptions

#### Environment Configuration
```typescript
// Backend environment variables
PAYPAL_CLIENT_ID=AXxxx... // Production
PAYPAL_CLIENT_ID=AYxxx... // Sandbox/Development
PAYPAL_CLIENT_SECRET=ELxxx... // Production secret
PAYPAL_CLIENT_SECRET=EMxxx... // Sandbox secret
PAYPAL_WEBHOOK_ID=WH-xxx... // Webhook ID for verification
PAYPAL_MODE=sandbox // or 'live' for production

// Frontend environment variables
REACT_APP_PAYPAL_CLIENT_ID=AXxxx...
REACT_APP_PAYPAL_MODE=sandbox
```

### 2. Database Schema Updates

#### New Tables
```sql
-- PayPal customer mapping
CREATE TABLE paypal_customers (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL UNIQUE,
    paypal_customer_id VARCHAR(255) NOT NULL UNIQUE,
    paypal_email VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- PayPal payment methods (stored references, not sensitive data)
CREATE TABLE payment_methods (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    paypal_payment_method_id VARCHAR(255) NOT NULL,
    type ENUM('paypal', 'card', 'bank_account') NOT NULL,
    last_four VARCHAR(4),
    brand VARCHAR(50),
    exp_month INT,
    exp_year INT,
    paypal_email VARCHAR(255),
    is_default BOOLEAN DEFAULT false,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_user_payment_methods (user_id)
);

-- Subscription transactions
CREATE TABLE subscription_transactions (
    id INT PRIMARY KEY AUTO_INCREMENT,
    account_id INT NOT NULL,
    subscription_id INT NOT NULL,
    paypal_transaction_id VARCHAR(255),
    paypal_subscription_id VARCHAR(255),
    paypal_order_id VARCHAR(255),
    amount DECIMAL(10,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    status ENUM('pending', 'completed', 'failed', 'cancelled', 'refunded') NOT NULL,
    transaction_type ENUM('subscription', 'upgrade', 'downgrade', 'refund') NOT NULL,
    billing_period_start DATE,
    billing_period_end DATE,
    paypal_fee DECIMAL(10,2) DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (account_id) REFERENCES accounts(id) ON DELETE CASCADE,
    FOREIGN KEY (subscription_id) REFERENCES account_subscriptions(id) ON DELETE CASCADE,
    INDEX idx_account_transactions (account_id),
    INDEX idx_subscription_transactions (subscription_id),
    INDEX idx_paypal_transaction (paypal_transaction_id)
);

-- PayPal webhook events log
CREATE TABLE paypal_webhook_events (
    id INT PRIMARY KEY AUTO_INCREMENT,
    paypal_event_id VARCHAR(255) NOT NULL UNIQUE,
    event_type VARCHAR(100) NOT NULL,
    processed BOOLEAN DEFAULT false,
    processing_error TEXT NULL,
    raw_data JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    processed_at TIMESTAMP NULL,
    INDEX idx_event_type (event_type),
    INDEX idx_processed (processed)
);
```

#### Update Existing Tables
```sql
-- Add PayPal subscription ID to account_subscriptions
ALTER TABLE account_subscriptions 
ADD COLUMN paypal_subscription_id VARCHAR(255) NULL,
ADD COLUMN paypal_plan_id VARCHAR(255) NULL,
ADD INDEX idx_paypal_subscription (paypal_subscription_id);

-- Add billing fields to subscription_plans
ALTER TABLE subscription_plans
ADD COLUMN paypal_plan_id_monthly VARCHAR(255) NULL,
ADD COLUMN paypal_plan_id_yearly VARCHAR(255) NULL,
ADD COLUMN paypal_product_id VARCHAR(255) NULL;
```

### 3. Backend Implementation

#### PayPal Service Layer

**PayPalService** (`apps/backend/src/services/PayPalService.ts`)
```typescript
import axios from 'axios';

interface PayPalAccessToken {
  access_token: string;
  token_type: string;
  expires_in: number;
}

export class PayPalService {
  private baseURL: string;
  private clientId: string;
  private clientSecret: string;
  private accessToken: string | null = null;
  private tokenExpiry: number = 0;
  
  constructor() {
    this.baseURL = process.env.PAYPAL_MODE === 'live' 
      ? 'https://api.paypal.com' 
      : 'https://api.sandbox.paypal.com';
    this.clientId = process.env.PAYPAL_CLIENT_ID!;
    this.clientSecret = process.env.PAYPAL_CLIENT_SECRET!;
  }

  // Authentication
  async getAccessToken(): Promise<string>
  private async refreshAccessToken(): Promise<PayPalAccessToken>

  // Product & Plan Management
  async createProduct(name: string, description: string): Promise<any>
  async createSubscriptionPlan(productId: string, planName: string, amount: string, interval: 'MONTH' | 'YEAR'): Promise<any>
  async updateSubscriptionPlan(planId: string, updateData: any): Promise<any>
  async activateSubscriptionPlan(planId: string): Promise<any>

  // Subscription Management  
  async createSubscription(planId: string, subscriberInfo: any): Promise<any>
  async getSubscription(subscriptionId: string): Promise<any>
  async cancelSubscription(subscriptionId: string, reason: string): Promise<any>
  async activateSubscription(subscriptionId: string, reason: string): Promise<any>
  async suspendSubscription(subscriptionId: string, reason: string): Promise<any>

  // Payment & Transaction Management
  async capturePayment(orderId: string): Promise<any>
  async refundPayment(captureId: string, amount?: { currency_code: string; value: string }): Promise<any>
  async getTransactionDetails(transactionId: string): Promise<any>

  // Webhook Management
  async verifyWebhookSignature(headers: any, body: string): Promise<boolean>
  async createWebhook(url: string, eventTypes: string[]): Promise<any>
}
```

**BillingService** (`apps/backend/src/services/BillingService.ts`)
```typescript
export class BillingService {
  private paypalService: PayPalService;
  private subscriptionService: SubscriptionService;

  constructor() {
    this.paypalService = new PayPalService();
    this.subscriptionService = new SubscriptionService();
  }

  async initializeCustomer(userId: number): Promise<PayPalCustomer>
  async upgradeSubscription(userId: number, planId: number): Promise<{ approvalUrl: string; subscriptionId: string }>
  async confirmSubscription(userId: number, subscriptionId: string, paypalSubscriptionId: string): Promise<AccountSubscription>
  async cancelSubscription(userId: number, reason?: string): Promise<AccountSubscription>
  async getBillingHistory(userId: number): Promise<SubscriptionTransaction[]>
  async processWebhookEvent(eventType: string, eventData: any): Promise<void>
  async syncSubscriptionStatus(paypalSubscriptionId: string): Promise<void>
  async handleFailedPayment(subscriptionId: string, reason: string): Promise<void>
}
```

#### Webhook Handler

**PayPalWebhookHandler** (`apps/backend/src/handlers/paypalWebhookHandler.ts`)
```typescript
export class PayPalWebhookHandler {
  private paypalService: PayPalService;
  private billingService: BillingService;

  async handleWebhook(req: Request, res: Response): Promise<void> {
    const webhookId = process.env.PAYPAL_WEBHOOK_ID!;
    const headers = req.headers;
    const body = JSON.stringify(req.body);

    try {
      // Verify webhook signature
      const isValid = await this.paypalService.verifyWebhookSignature(headers, body);
      if (!isValid) {
        return res.status(400).send('Invalid webhook signature');
      }

      const event = req.body;
      
      // Log the event
      await this.logWebhookEvent(event);
      
      switch (event.event_type) {
        case 'BILLING.SUBSCRIPTION.CREATED':
          await this.handleSubscriptionCreated(event.resource);
          break;
        case 'BILLING.SUBSCRIPTION.ACTIVATED':
          await this.handleSubscriptionActivated(event.resource);
          break;
        case 'BILLING.SUBSCRIPTION.UPDATED':
          await this.handleSubscriptionUpdated(event.resource);
          break;
        case 'BILLING.SUBSCRIPTION.CANCELLED':
          await this.handleSubscriptionCancelled(event.resource);
          break;
        case 'BILLING.SUBSCRIPTION.SUSPENDED':
          await this.handleSubscriptionSuspended(event.resource);
          break;
        case 'PAYMENT.SALE.COMPLETED':
          await this.handlePaymentCompleted(event.resource);
          break;
        case 'PAYMENT.SALE.DENIED':
          await this.handlePaymentFailed(event.resource);
          break;
        default:
          console.log(`Unhandled event type: ${event.event_type}`);
      }

      res.status(200).json({ received: true });
    } catch (error) {
      console.error('Webhook processing failed:', error);
      res.status(500).send('Webhook processing error');
    }
  }

  private async handleSubscriptionCreated(subscription: any): Promise<void>
  private async handleSubscriptionActivated(subscription: any): Promise<void>
  private async handleSubscriptionUpdated(subscription: any): Promise<void>
  private async handleSubscriptionCancelled(subscription: any): Promise<void>
  private async handleSubscriptionSuspended(subscription: any): Promise<void>
  private async handlePaymentCompleted(payment: any): Promise<void>
  private async handlePaymentFailed(payment: any): Promise<void>
  private async logWebhookEvent(event: any): Promise<void>
}
```

#### API Routes

**Billing Routes** (`apps/backend/src/routes/billing.ts`)
```typescript
// PayPal Integration Routes
POST   /api/billing/paypal/create-subscription    // Create PayPal subscription
POST   /api/billing/paypal/confirm-subscription   // Confirm subscription after user approval
POST   /api/billing/paypal/cancel-subscription    // Cancel PayPal subscription
GET    /api/billing/paypal/subscription/:id       // Get subscription details

// Payment Management
GET    /api/billing/payment-methods               // List user's payment methods (limited info)
DELETE /api/billing/payment-methods/:id          // Remove saved payment reference

// Billing History & Transactions
GET    /api/billing/history                       // Get billing history
GET    /api/billing/transactions                  // List transactions
GET    /api/billing/transactions/:id              // Get specific transaction

// Webhooks
POST   /api/billing/webhooks/paypal               // PayPal webhook endpoint

// Plan Management
GET    /api/billing/plans                         // Get available subscription plans
POST   /api/billing/sync-plans                    // Sync plans with PayPal (admin only)
```

### 4. Frontend Implementation

#### PayPal Integration Setup

**PayPal Provider** (`apps/frontend/src/providers/PayPalProvider.tsx`)
```typescript
import { PayPalScriptProvider } from '@paypal/react-paypal-js';

const initialOptions = {
  'client-id': process.env.REACT_APP_PAYPAL_CLIENT_ID!,
  currency: 'USD',
  intent: 'subscription',
  vault: true,
  components: 'buttons,hosted-fields',
};

export const PayPalProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  return (
    <PayPalScriptProvider options={initialOptions}>
      {children}
    </PayPalScriptProvider>
  );
};
```

#### Payment Components

**PayPalSubscriptionButton** (`apps/frontend/src/components/billing/PayPalSubscriptionButton.tsx`)
```typescript
import { PayPalButtons, usePayPalScriptReducer } from '@paypal/react-paypal-js';

interface PayPalSubscriptionButtonProps {
  planId: string;
  onSuccess: (subscriptionId: string) => void;
  onError: (error: any) => void;
}

export const PayPalSubscriptionButton: React.FC<PayPalSubscriptionButtonProps> = ({
  planId,
  onSuccess,
  onError
}) => {
  const [{ isPending }] = usePayPalScriptReducer();

  if (isPending) {
    return <div>Loading PayPal...</div>;
  }

  return (
    <PayPalButtons
      style={{
        shape: 'rect',
        color: 'gold',
        layout: 'vertical',
        label: 'subscribe',
      }}
      createSubscription={(data, actions) => {
        return actions.subscription.create({
          plan_id: planId,
          application_context: {
            brand_name: 'Project Ledger',
            locale: 'en-US',
            shipping_preference: 'NO_SHIPPING',
            user_action: 'SUBSCRIBE_NOW',
            payment_method: {
              payer_selected: 'PAYPAL',
              payee_preferred: 'IMMEDIATE_PAYMENT_REQUIRED',
            },
            return_url: `${window.location.origin}/billing/success`,
            cancel_url: `${window.location.origin}/billing/cancel`,
          },
        });
      }}
      onApprove={async (data, actions) => {
        try {
          // Call backend to confirm subscription
          const response = await fetch('/api/billing/paypal/confirm-subscription', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ 
              subscriptionId: data.subscriptionID,
              planId: planId 
            }),
          });
          
          if (response.ok) {
            onSuccess(data.subscriptionID);
          } else {
            throw new Error('Failed to confirm subscription');
          }
        } catch (error) {
          onError(error);
        }
      }}
      onError={onError}
      onCancel={() => {
        console.log('Subscription cancelled by user');
      }}
    />
  );
};
```

**SubscriptionUpgrade** (`apps/frontend/src/components/billing/SubscriptionUpgrade.tsx`)
```typescript
export const SubscriptionUpgrade: React.FC = () => {
  const { currentPlan, limits, usage, refreshSubscription } = useSubscription();
  const [selectedPlan, setSelectedPlan] = useState<'monthly' | 'yearly'>('monthly');
  const [isProcessing, setIsProcessing] = useState(false);
  const [paypalPlanId, setPaypalPlanId] = useState<string>('');

  useEffect(() => {
    // Fetch PayPal plan ID based on selection
    fetchPayPalPlanId();
  }, [selectedPlan]);

  const fetchPayPalPlanId = async () => {
    try {
      const response = await fetch(`/api/billing/plans?type=${selectedPlan}`);
      const data = await response.json();
      setPaypalPlanId(data.paypalPlanId);
    } catch (error) {
      console.error('Failed to fetch PayPal plan ID:', error);
    }
  };

  const handleUpgradeSuccess = async (subscriptionId: string) => {
    setIsProcessing(true);
    try {
      // Refresh subscription context
      await refreshSubscription();
      
      // Show success message
      showSuccessToast('Successfully upgraded to Professional plan!');
      
      // Redirect to billing page
      window.location.href = '/billing?success=true';
    } catch (error) {
      showErrorToast('Upgrade completed but failed to refresh data. Please refresh the page.');
    } finally {
      setIsProcessing(false);
    }
  };

  const handleUpgradeError = (error: any) => {
    console.error('PayPal subscription error:', error);
    showErrorToast('Failed to process subscription. Please try again.');
  };

  return (
    <div className="subscription-upgrade">
      <div className="plan-comparison">
        <div className="current-plan">
          <h3>Current Plan: {currentPlan?.name}</h3>
          <div className="usage-stats">
            <UsageIndicator resource="clients" current={usage?.clients} limit={limits?.clients} />
            <UsageIndicator resource="projects" current={usage?.projects} limit={limits?.projects} />
            <UsageIndicator resource="quotes" current={usage?.quotes} limit={limits?.quotes} />
            <UsageIndicator resource="invoices" current={usage?.invoices} limit={limits?.invoices} />
            <UsageIndicator resource="users" current={usage?.users} limit={limits?.users} />
          </div>
        </div>
        
        <div className="upgrade-options">
          <h3>Upgrade to Professional</h3>
          <div className="billing-toggle">
            <label>
              <input 
                type="radio" 
                value="monthly" 
                checked={selectedPlan === 'monthly'}
                onChange={(e) => setSelectedPlan(e.target.value as 'monthly')}
              />
              Monthly - $99.99/month
            </label>
            <label>
              <input 
                type="radio" 
                value="yearly" 
                checked={selectedPlan === 'yearly'}
                onChange={(e) => setSelectedPlan(e.target.value as 'yearly')}
              />
              Yearly - $999.99/year (Save 17%)
            </label>
          </div>
          
          <div className="paypal-subscription">
            <h4>Subscribe with PayPal</h4>
            <p>Secure payment processing with PayPal. Cancel anytime.</p>
            
            {paypalPlanId && (
              <PayPalSubscriptionButton
                planId={paypalPlanId}
                onSuccess={handleUpgradeSuccess}
                onError={handleUpgradeError}
              />
            )}
            
            {isProcessing && (
              <div className="processing-overlay">
                <p>Processing your subscription...</p>
              </div>
            )}
          </div>
        </div>
      </div>
    </div>
  );
};
```

**BillingHistory** (`apps/frontend/src/components/billing/BillingHistory.tsx`)
```typescript
export const BillingHistory: React.FC = () => {
  const [transactions, setTransactions] = useState<SubscriptionTransaction[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadBillingHistory();
  }, []);

  const loadBillingHistory = async () => {
    try {
      const response = await fetch('/api/billing/history');
      const data = await response.json();
      setTransactions(data.transactions || []);
    } catch (error) {
      console.error('Failed to load billing history:', error);
    } finally {
      setLoading(false);
    }
  };

  const getStatusColor = (status: string) => {
    switch (status) {
      case 'completed': return 'text-green-600';
      case 'pending': return 'text-yellow-600';
      case 'failed': return 'text-red-600';
      case 'cancelled': return 'text-gray-600';
      case 'refunded': return 'text-blue-600';
      default: return 'text-gray-600';
    }
  };

  if (loading) {
    return <div>Loading billing history...</div>;
  }

  return (
    <div className="billing-history">
      <h3>Transaction History</h3>
      
      {transactions.length === 0 ? (
        <p className="text-gray-500">No transactions found.</p>
      ) : (
        <div className="transaction-list">
          {transactions.map(transaction => (
            <div key={transaction.id} className="transaction-item border rounded-lg p-4 mb-4">
              <div className="flex justify-between items-start">
                <div>
                  <h4 className="font-medium">
                    {transaction.transactionType.charAt(0).toUpperCase() + 
                     transaction.transactionType.slice(1)}
                  </h4>
                  <p className="text-sm text-gray-600">
                    {formatDate(transaction.createdAt)}
                  </p>
                  {transaction.paypalTransactionId && (
                    <p className="text-xs text-gray-500">
                      PayPal ID: {transaction.paypalTransactionId}
                    </p>
                  )}
                </div>
                <div className="text-right">
                  <p className="font-medium">${transaction.amount}</p>
                  <p className={`text-sm ${getStatusColor(transaction.status)}`}>
                    {transaction.status.charAt(0).toUpperCase() + transaction.status.slice(1)}
                  </p>
                </div>
              </div>
              
              {transaction.billingPeriodStart && transaction.billingPeriodEnd && (
                <div className="mt-2 text-sm text-gray-600">
                  Billing Period: {formatDate(transaction.billingPeriodStart)} - {formatDate(transaction.billingPeriodEnd)}
                </div>
              )}
            </div>
          ))}
        </div>
      )}
    </div>
  );
};
```

#### Updated Settings Page

**Billing Tab** in Settings (`apps/frontend/src/pages/Settings/BillingTab.tsx`)
```typescript
export const BillingTab: React.FC = () => {
  const { currentPlan, usage, limits } = useSubscription();

  return (
    <div className="billing-tab">
      <div className="current-subscription">
        <h3>Current Subscription</h3>
        <div className="plan-info">
          <PlanBadge plan={currentPlan} />
          <div className="plan-details">
            <p>Next billing date: {formatDate(currentPlan?.next_billing_date)}</p>
            <p>Amount: ${currentPlan?.amount}/month</p>
          </div>
        </div>
        
        {currentPlan?.name === 'Free' && (
          <div className="upgrade-prompt">
            <h4>Upgrade to unlock unlimited access</h4>
            <SubscriptionUpgrade />
          </div>
        )}
      </div>

      <div className="usage-overview">
        <h3>Usage Overview</h3>
        <div className="usage-grid">
          <UsageIndicator resource="clients" current={usage?.clients} limit={limits?.clients} />
          <UsageIndicator resource="projects" current={usage?.projects} limit={limits?.projects} />
          <UsageIndicator resource="quotes" current={usage?.quotes} limit={limits?.quotes} />
          <UsageIndicator resource="invoices" current={usage?.invoices} limit={limits?.invoices} />
          <UsageIndicator resource="users" current={usage?.users} limit={limits?.users} />
        </div>
      </div>

      <div className="payment-methods">
        <h3>Payment Methods</h3>
        <PaymentMethodManager />
      </div>

      <div className="billing-history">
        <BillingHistory />
      </div>

      <div className="billing-portal">
        <h3>Manage Billing</h3>
        <p>Access PayPal's billing management to update payment information, view invoices, and manage your subscription.</p>
        <button onClick={() => openPayPalBillingPortal()}>
          Open Billing Portal
        </button>
      </div>
    </div>
  );
};
```

### 5. Shared Types Updates

**Billing Types** (`packages/shared-types/billing/index.ts`)
```typescript
export interface PayPalCustomer {
  id: number;
  userId: number;
  paypalCustomerId: string;
  paypalEmail?: string;
  createdAt: Date;
  updatedAt: Date;
}

export interface PaymentMethod {
  id: number;
  userId: number;
  paypalPaymentMethodId: string;
  type: 'paypal' | 'card' | 'bank_account';
  lastFour?: string;
  brand?: string;
  expMonth?: number;
  expYear?: number;
  paypalEmail?: string;
  isDefault: boolean;
  isActive: boolean;
}

export interface SubscriptionTransaction {
  id: number;
  accountId: number;
  subscriptionId: number;
  paypalTransactionId?: string;
  paypalSubscriptionId?: string;
  paypalOrderId?: string;
  amount: number;
  currency: string;
  status: 'pending' | 'completed' | 'failed' | 'cancelled' | 'refunded';
  transactionType: 'subscription' | 'upgrade' | 'downgrade' | 'refund';
  billingPeriodStart?: Date;
  billingPeriodEnd?: Date;
  paypalFee?: number;
  createdAt: Date;
}

export interface PayPalSubscription {
  id: string;
  status: 'APPROVAL_PENDING' | 'APPROVED' | 'ACTIVE' | 'SUSPENDED' | 'CANCELLED' | 'EXPIRED';
  plan_id: string;
  subscriber: {
    email_address: string;
    name?: {
      given_name: string;
      surname: string;
    };
  };
  billing_info: {
    outstanding_balance: {
      currency_code: string;
      value: string;
    };
    cycle_executions: Array<{
      tenure_type: string;
      sequence: number;
      cycles_completed: number;
      cycles_remaining: number;
      current_pricing_scheme_version: number;
    }>;
    last_payment: {
      amount: {
        currency_code: string;
        value: string;
      };
      time: string;
    };
    next_billing_time: string;
    final_payment_time: string;
    failed_payments_count: number;
  };
  create_time: string;
  update_time: string;
  links: Array<{
    href: string;
    rel: string;
    method: string;
  }>;
}

export interface PayPalPlan {
  id: string;
  product_id: string;
  name: string;
  description: string;
  status: 'CREATED' | 'INACTIVE' | 'ACTIVE';
  billing_cycles: Array<{
    frequency: {
      interval_unit: 'MONTH' | 'YEAR';
      interval_count: number;
    };
    tenure_type: 'REGULAR' | 'TRIAL';
    sequence: number;
    total_cycles: number;
    pricing_scheme: {
      fixed_price: {
        currency_code: string;
        value: string;
      };
    };
  }>;
  payment_preferences: {
    auto_bill_outstanding: boolean;
    setup_fee: {
      currency_code: string;
      value: string;
    };
    setup_fee_failure_action: 'CONTINUE' | 'CANCEL';
    payment_failure_threshold: number;
  };
  taxes: {
    percentage: string;
    inclusive: boolean;
  };
  create_time: string;
  update_time: string;
  links: Array<{
    href: string;
    rel: string;
    method: string;
  }>;
}
```

## Implementation Phases

### Phase 1: PayPal Account & Backend Foundation (1-2 weeks)
1. **PayPal Business Account Setup**
   - Create PayPal Business account
   - Configure products and subscription plans
   - Set up webhook endpoints
   - Configure tax settings and compliance
   - Generate API credentials (Client ID, Secret)

2. **Database Setup**
   - Create migration scripts for PayPal-specific tables
   - Update existing tables with PayPal references
   - Set up indexes for performance optimization
   - Add PayPal plan IDs to subscription_plans table

3. **Core Services Implementation**
   - Implement PayPalService class with authentication
   - Create BillingService for business logic integration
   - Build webhook handler for real-time PayPal updates
   - Implement subscription lifecycle management

### Phase 2: API Integration (1-2 weeks)
1. **Billing API Endpoints**
   - Customer management endpoints
   - Subscription creation and management
   - Transaction history and reporting
   - Plan synchronization with PayPal

2. **Webhook Processing**
   - Event logging and deduplication
   - Subscription status synchronization
   - Payment completion/failure handling
   - Subscription lifecycle event processing

3. **Testing Infrastructure**
   - Unit tests for all PayPal services
   - Integration tests with PayPal Sandbox
   - Webhook testing with PayPal simulator
   - End-to-end subscription flow testing

### Phase 3: Frontend Integration (1-2 weeks)
1. **PayPal Components**
   - PayPal SDK integration
   - Subscription button components
   - Payment method display (limited info)
   - Subscription management interface

2. **Enhanced Billing Dashboard**
   - PayPal-specific transaction history
   - Subscription status display
   - Plan upgrade/downgrade flows
   - Payment success/failure handling

3. **User Experience Improvements**
   - Error handling and user feedback
   - Loading states during PayPal redirects
   - Mobile-responsive PayPal integration
   - Success/cancellation page flows

### Phase 4: Testing & Deployment (1 week)
1. **Comprehensive Testing**
   - End-to-end subscription flows
   - Payment method testing (sandbox)
   - Webhook reliability validation
   - Cross-browser PayPal integration testing

2. **Security & Compliance Review**
   - PayPal webhook signature verification
   - API endpoint security validation
   - Data handling compliance check
   - Payment flow security audit

3. **Production Deployment**
   - Environment configuration for live PayPal
   - PayPal live account activation
   - Monitoring and alerting setup
   - User communication and gradual rollout

## PayPal-Specific Advantages

### Easier Setup
- **Faster Integration**: PayPal's subscription API is more straightforward than Stripe
- **Lower Technical Complexity**: Fewer moving parts and simpler webhook handling
- **Built-in UI**: PayPal handles most of the payment UI reducing frontend complexity
- **Instant Approval**: Users can approve subscriptions immediately with existing PayPal accounts

### Business Benefits
- **Higher Conversion**: Many users already have PayPal accounts
- **Trust Factor**: PayPal brand recognition increases user confidence
- **Global Reach**: Built-in international payment support
- **Lower Development Cost**: Faster to implement than custom payment forms

### Security & Compliance
- **Automatic PCI Compliance**: PayPal handles all sensitive payment data
- **Fraud Protection**: Built-in fraud detection and buyer protection
- **Dispute Resolution**: PayPal's dispute resolution system
- **Regulatory Compliance**: Automatic compliance with international payment regulations

## Testing Strategy

### PayPal Sandbox Testing
```typescript
// PayPal test credentials for sandbox
const testCredentials = {
  clientId: 'AYxxx...', // Sandbox Client ID
  clientSecret: 'EMxxx...', // Sandbox Secret
  mode: 'sandbox'
};

// Test subscription plans
const testPlans = {
  monthly: 'P-5ML4271244454362WXNWU5NQ', // Sandbox plan ID
  yearly: 'P-62J51527V5405803FXNWU6DY'   // Sandbox plan ID
};
```

### Test Scenarios
1. **Successful Subscription**: User completes PayPal flow successfully
2. **Cancelled Subscription**: User cancels during PayPal flow
3. **Failed Payment**: Insufficient funds or declined payment method
4. **Webhook Processing**: Verify all webhook events are handled correctly
5. **Subscription Management**: Cancel, suspend, and reactivate subscriptions

## Security Considerations

### PayPal-Specific Security
- **Webhook Verification**: Verify all webhook signatures using PayPal's verification
- **API Authentication**: Secure storage and rotation of PayPal API credentials
- **HTTPS Requirement**: All PayPal communications require HTTPS
- **Subscription Validation**: Verify subscription details match our plans

### Data Protection
- **Minimal Data Storage**: Store only necessary PayPal references, not payment data
- **Audit Logging**: Log all PayPal API interactions for troubleshooting
- **User Privacy**: Respect PayPal's data handling requirements
- **GDPR Compliance**: Properly handle EU customer data with PayPal integration

## Monitoring & Analytics

### PayPal-Specific Metrics
- **Subscription Conversion Rate**: PayPal approval to active subscription rate
- **Payment Success Rate**: Successful PayPal payment processing rate
- **Churn Analysis**: Subscription cancellation patterns via PayPal
- **Revenue Tracking**: Monthly recurring revenue through PayPal
- **Webhook Processing Health**: PayPal event processing reliability

### Alerting Setup
- PayPal API failures and rate limiting
- Webhook processing failures or delays
- Subscription payment failures
- High subscription cancellation rates
- PayPal account status changes

This PayPal integration approach provides a simpler, faster path to subscription billing while maintaining security and providing excellent user experience through PayPal's trusted payment platform.
