# Payment Gateway Enhancement Specification

**Date:** October 12, 2025  
**Version:** 1.0  
**Status:** Partially Implemented  

---

## 1. Overview

The Payment Gateway Enhancement extends the existing basic Stripe integration to support multiple payment providers, methods, and advanced features. It provides a unified payment processing interface while supporting ACH transfers, additional credit card processors, digital wallets, and international payment methods.

---

## 2. Business Requirements

### 2.1 Core Enhancement Features
- **Multiple Payment Providers**: Support Stripe, PayPal, Square, Authorize.Net
- **ACH Bank Transfers**: Direct bank account payments with lower fees
- **Digital Wallets**: Apple Pay, Google Pay, Samsung Pay integration
- **International Payments**: Multi-currency support and international methods
- **Payment Installments**: Split payments and payment plans
- **Subscription Billing**: Enhanced recurring payment management

### 2.2 Current State Analysis
**Existing Implementation:**
- ✅ Basic Stripe payment intent creation
- ✅ Stripe webhook handling for payment status updates
- ✅ PayPal subscription integration for account plans
- ⚠️ Limited to basic credit card processing
- ⚠️ No ACH or bank transfer support
- ❌ Single payment provider dependency

### 2.3 Enhancement Goals
- **Payment Method Diversity**: Support 10+ payment methods
- **Lower Transaction Costs**: ACH transfers at 0.8% vs 2.9% credit cards
- **International Expansion**: Support payments from 50+ countries
- **Improved Success Rates**: Multiple providers for failover
- **Enhanced User Experience**: One-click payments and saved methods

---

## 3. Data Model Extensions

### 3.1 Database Schema Enhancements

```prisma
// Extend existing Payment model
model Payment {
  // ... existing fields ...
  
  // Enhanced payment method support
  paymentProviderId     Int?                    // Which provider processed this payment
  paymentMethodType     PaymentMethodType       @default(CREDIT_CARD)
  paymentMethodDetails  Json?                   // Provider-specific method details
  
  // ACH/Bank transfer specific
  bankAccountLast4      String?                 // Last 4 digits of bank account
  bankName              String?                 // Bank name for ACH transfers
  achReturnCode         String?                 // ACH return code if failed
  
  // International payments
  currencyCode          String                  @default("USD")
  exchangeRate          Float?                  // Exchange rate used
  originalAmount        Float?                  // Amount in original currency
  
  // Installment payments
  isInstallment         Boolean                 @default(false)
  installmentPlan       String?                 // Installment plan identifier
  installmentNumber     Int?                    // Which installment this is
  totalInstallments     Int?                    // Total number of installments
  
  // Enhanced tracking
  processingTimeMs      Int?                    // Time to process payment
  failureReason         String?                 // Detailed failure reason
  riskScore             Float?                  // Fraud risk score
  
  // Relations
  paymentProvider       PaymentProvider?        @relation(fields: [paymentProviderId], references: [id])
  installmentPlanRef    PaymentPlan?            @relation(fields: [installmentPlan], references: [id])
}

model PaymentProvider {
  id                    Int                     @id @default(autoincrement())
  accountId             Int                     // Multi-tenant support
  
  // Provider details
  name                  String                  // "Stripe", "PayPal", "Square", etc.
  providerType          PaymentProviderType
  isActive              Boolean                 @default(true)
  priority              Int                     @default(0) // Provider priority for routing
  
  // Configuration (encrypted)
  apiKey                String?                 // Primary API key
  secretKey             String?                 // Secret key
  webhookSecret         String?                 // Webhook secret
  merchantId            String?                 // Merchant ID
  configuration         Json                    // Provider-specific config
  
  // Supported features
  supportedMethods      Json                    // Supported payment methods
  supportedCurrencies   Json                    // Supported currencies
  minAmount             Float                   @default(0.50) // Minimum payment amount
  maxAmount             Float?                  // Maximum payment amount
  
  // Fee structure
  flatFee               Float                   @default(0.00) // Flat fee per transaction
  percentageFee         Float                   @default(2.90) // Percentage fee
  achFee                Float                   @default(0.80) // ACH fee percentage
  
  // Status and health
  isHealthy             Boolean                 @default(true)
  lastHealthCheck       DateTime?               // Last health check
  healthCheckUrl        String?                 // Health check endpoint
  
  // Usage statistics
  totalTransactions     Int                     @default(0)
  totalVolume           Float                   @default(0.00)
  successRate           Float                   @default(100.00)
  avgProcessingTime     Int                     @default(2000) // ms
  
  // Metadata
  createdAt             DateTime                @default(now())
  updatedAt             DateTime                @updatedAt
  createdBy             Int
  
  // Relations
  account               Account                 @relation(fields: [accountId], references: [id])
  createdByUser         User                    @relation(fields: [createdBy], references: [id])
  payments              Payment[]
  savedMethods          SavedPaymentMethod[]
  
  @@index([accountId])
  @@index([providerType])
  @@index([isActive, priority])
}

model SavedPaymentMethod {
  id                    Int                     @id @default(autoincrement())
  clientId              Int                     // Customer who owns this method
  paymentProviderId     Int                     // Provider storing this method
  
  // Method details
  providerMethodId      String                  // ID from payment provider
  methodType            PaymentMethodType
  
  // Display information (non-sensitive)
  displayName           String                  // "Visa ****1234"
  last4                 String?                 // Last 4 digits
  brand                 String?                 // Card brand or bank name
  expiryMonth           Int?                    // Card expiry month
  expiryYear            Int?                    // Card expiry year
  
  // ACH/Bank account info
  accountType           String?                 // "checking", "savings"
  bankName              String?                 // Bank name
  
  // Status
  isActive              Boolean                 @default(true)
  isDefault             Boolean                 @default(false)
  
  // Usage tracking
  lastUsedAt            DateTime?
  usageCount            Int                     @default(0)
  
  // Metadata
  createdAt             DateTime                @default(now())
  updatedAt             DateTime                @updatedAt
  
  // Relations
  client                Client                  @relation(fields: [clientId], references: [id])
  paymentProvider       PaymentProvider         @relation(fields: [paymentProviderId], references: [id])
  
  @@unique([clientId, providerMethodId, paymentProviderId])
  @@index([clientId])
  @@index([paymentProviderId])
}

model PaymentPlan {
  id                    String                  @id @default(cuid()) // String ID for easier reference
  invoiceId             Int                     // Invoice this plan is for
  clientId              Int                     // Client who owns this plan
  
  // Plan details
  planName              String                  // "3-Month Payment Plan"
  totalAmount           Float                   // Total amount to be paid
  numberOfInstallments  Int                     // Number of payments
  installmentAmount     Float                   // Amount per installment
  frequency             PaymentFrequency        // WEEKLY, MONTHLY, etc.
  
  // Schedule
  startDate             DateTime                // When first payment is due
  endDate               DateTime                // When final payment is due
  nextPaymentDate       DateTime?               // Next scheduled payment
  
  // Status
  status                PaymentPlanStatus       @default(ACTIVE)
  paidInstallments      Int                     @default(0)
  remainingAmount       Float                   // Amount still owed
  
  // Configuration
  autoCharge            Boolean                 @default(true) // Auto-charge on due date
  paymentMethodId       Int?                    // Default payment method
  reminderDaysBefore    Int                     @default(3) // Days before due date to remind
  
  // Late payment handling
  lateFeeAmount         Float                   @default(0.00)
  gracePeriodDays       Int                     @default(3)
  
  // Metadata
  createdAt             DateTime                @default(now())
  updatedAt             DateTime                @updatedAt
  createdBy             Int
  
  // Relations
  invoice               Invoice                 @relation(fields: [invoiceId], references: [id])
  client                Client                  @relation(fields: [clientId], references: [id])
  createdByUser         User                    @relation(fields: [createdBy], references: [id])
  payments              Payment[]
  
  @@index([clientId])
  @@index([status])
  @@index([nextPaymentDate])
}

model PaymentAttempt {
  id                    Int                     @id @default(autoincrement())
  paymentId             Int?                    // Associated payment if successful
  invoiceId             Int                     // Invoice being paid
  paymentProviderId     Int                     // Provider used for attempt
  
  // Attempt details
  amount                Float                   // Amount attempted
  currencyCode          String                  @default("USD")
  paymentMethodType     PaymentMethodType
  
  // Provider response
  providerTransactionId String?                 // Transaction ID from provider
  providerStatus        String                  // Provider-specific status
  providerResponse      Json?                   // Full provider response
  
  // Result
  status                PaymentAttemptStatus
  failureReason         String?                 // Why payment failed
  failureCode           String?                 // Provider failure code
  
  // Timing
  attemptedAt           DateTime                @default(now())
  processedAt           DateTime?               // When provider responded
  processingTimeMs      Int?                    // Processing duration
  
  // Retry logic
  isRetry               Boolean                 @default(false)
  originalAttemptId     Int?                    // Original attempt if this is retry
  retryCount            Int                     @default(0)
  nextRetryAt           DateTime?               // When to retry if failed
  
  // Relations
  payment               Payment?                @relation(fields: [paymentId], references: [id])
  invoice               Invoice                 @relation(fields: [invoiceId], references: [id])
  paymentProvider       PaymentProvider         @relation(fields: [paymentProviderId], references: [id])
  originalAttempt       PaymentAttempt?         @relation("PaymentAttemptRetry", fields: [originalAttemptId], references: [id])
  retryAttempts         PaymentAttempt[]        @relation("PaymentAttemptRetry")
  
  @@index([invoiceId])
  @@index([status])
  @@index([attemptedAt])
  @@index([nextRetryAt])
}

enum PaymentProviderType {
  CREDIT_CARD_PROCESSOR  // Stripe, Square, Authorize.Net
  DIGITAL_WALLET        // PayPal, Apple Pay, Google Pay
  BANK_TRANSFER         // ACH, Wire Transfer
  CRYPTOCURRENCY        // Bitcoin, Ethereum
  BUY_NOW_PAY_LATER     // Klarna, Afterpay
  INTERNATIONAL         // Wise, Remitly
}

enum PaymentMethodType {
  CREDIT_CARD
  DEBIT_CARD
  ACH_BANK_TRANSFER
  WIRE_TRANSFER
  PAYPAL
  APPLE_PAY
  GOOGLE_PAY
  SAMSUNG_PAY
  VENMO
  CASH_APP
  ZELLE
  CHECK
  CASH  
  CRYPTOCURRENCY
  BUY_NOW_PAY_LATER
  INTERNATIONAL_TRANSFER
  OTHER
}

enum PaymentFrequency {
  WEEKLY
  BIWEEKLY
  MONTHLY
  QUARTERLY
  ANNUALLY
  CUSTOM
}

enum PaymentPlanStatus {
  ACTIVE
  PAUSED
  COMPLETED
  CANCELLED
  DEFAULTED
}

enum PaymentAttemptStatus {
  PENDING
  PROCESSING
  SUCCESS
  FAILED
  CANCELLED
  EXPIRED
}
```

### 3.2 TypeScript Interface Extensions

```typescript
export interface PaymentProvider {
  id: string;
  name: string;
  providerType: PaymentProviderType;
  isActive: boolean;
  priority: number;
  
  // Configuration
  apiKey?: string;
  secretKey?: string;
  webhookSecret?: string;
  merchantId?: string;
  configuration: Record<string, any>;
  
  // Capabilities
  supportedMethods: PaymentMethodType[];
  supportedCurrencies: string[];
  minAmount: number;
  maxAmount?: number;
  
  // Fees
  flatFee: number;
  percentageFee: number;
  achFee: number;
  
  // Health
  isHealthy: boolean;
  lastHealthCheck?: string;
  successRate: number;
  avgProcessingTime: number;
}

export interface SavedPaymentMethod {
  id: string;
  providerMethodId: string;
  methodType: PaymentMethodType;
  displayName: string;
  last4?: string;
  brand?: string;
  expiryMonth?: number;
  expiryYear?: number;
  isDefault: boolean;
  lastUsedAt?: string;
}

export interface PaymentPlan {
  id: string;
  invoiceId: string;
  planName: string;
  totalAmount: number;
  numberOfInstallments: number;
  installmentAmount: number;
  frequency: PaymentFrequency;
  startDate: string;
  nextPaymentDate?: string;
  status: PaymentPlanStatus;
  paidInstallments: number;
  remainingAmount: number;
  autoCharge: boolean;
}

export interface ProcessPaymentRequest {
  invoiceId: string;
  amount: number;
  currencyCode?: string;
  paymentMethodType: PaymentMethodType;
  
  // Payment method details
  paymentMethodId?: string; // For saved methods
  cardToken?: string;       // For new card payments
  bankAccountToken?: string; // For ACH payments
  
  // Customer details
  billingAddress?: Address;
  savePaymentMethod?: boolean;
  
  // Options
  preferredProviderId?: string;
  metadata?: Record<string, any>;
}

export interface ProcessPaymentResponse {
  success: boolean;
  paymentId?: string;
  status: PaymentAttemptStatus;
  
  // Provider details
  providerId: string;
  providerTransactionId?: string;
  
  // Client actions (for 3D Secure, etc.)
  requiresAction?: boolean;
  clientSecret?: string;
  redirectUrl?: string;
  
  // Error details
  errorMessage?: string;
  errorCode?: string;
  
  // Payment details
  amount: number;
  fees: {
    processingFee: number;
    platformFee: number;
  };
}
```

---

## 4. API Endpoint Extensions

### 4.1 Payment Provider Management

```typescript
// Get available payment providers
GET /api/payment-providers
Query: {
  methodType?: PaymentMethodType,
  amount?: number,
  currency?: string
}
Response: PaymentProvider[]

// Configure payment provider
POST /api/payment-providers
Body: CreatePaymentProviderInput
Response: PaymentProvider

// Update provider configuration
PATCH /api/payment-providers/:id
Body: Partial<PaymentProviderConfig>
Response: PaymentProvider

// Test provider connection
POST /api/payment-providers/:id/health-check
Response: {
  isHealthy: boolean,
  responseTime: number,
  error?: string
}

// Get provider statistics
GET /api/payment-providers/:id/stats
Query: {
  dateFrom?: string,
  dateTo?: string
}
Response: {
  totalTransactions: number,
  totalVolume: number,
  successRate: number,
  avgProcessingTime: number,
  transactionsByMethod: Record<PaymentMethodType, number>
}
```

### 4.2 Enhanced Payment Processing

```typescript
// Process payment with provider selection
POST /api/payments/process-enhanced
Body: ProcessPaymentRequest
Response: ProcessPaymentResponse

// Get payment methods for customer
GET /api/clients/:clientId/payment-methods
Response: SavedPaymentMethod[]

// Save new payment method
POST /api/clients/:clientId/payment-methods
Body: {
  methodType: PaymentMethodType,
  token: string, // Provider token
  isDefault?: boolean,
  billingAddress?: Address
}
Response: SavedPaymentMethod

// Delete saved payment method
DELETE /api/payment-methods/:id
Response: { success: boolean }

// Create payment plan
POST /api/invoices/:invoiceId/payment-plan
Body: {
  numberOfInstallments: number,
  frequency: PaymentFrequency,
  startDate: string,
  paymentMethodId?: string
}
Response: PaymentPlan

// Update payment plan
PATCH /api/payment-plans/:id
Body: {
  autoCharge?: boolean,
  paymentMethodId?: string,
  reminderDaysBefore?: number
}
Response: PaymentPlan

// Process installment payment
POST /api/payment-plans/:id/charge-installment
Body: {
  paymentMethodId?: string,
  amount?: number // Override amount if needed
}
Response: ProcessPaymentResponse
```

### 4.3 Multi-Currency Support

```typescript
// Get supported currencies
GET /api/payment-providers/currencies
Response: {
  [providerId: string]: {
    currencies: string[],
    defaultCurrency: string,
    exchangeRates: Record<string, number>
  }
}

// Convert currency
POST /api/payments/convert-currency
Body: {
  amount: number,
  fromCurrency: string,
  toCurrency: string
}
Response: {
  originalAmount: number,
  convertedAmount: number,
  exchangeRate: number,
  fees: number
}

// Process international payment
POST /api/payments/process-international
Body: {
  invoiceId: string,
  amount: number,
  currency: string,
  paymentMethodType: PaymentMethodType,
  recipientDetails: InternationalRecipient
}
Response: ProcessPaymentResponse
```

---

## 5. Enhanced Payment Services

### 5.1 Payment Router Service

```typescript
export class PaymentRouterService {
  async processPayment(request: ProcessPaymentRequest): Promise<ProcessPaymentResponse> {
    // Select optimal payment provider
    const provider = await this.selectPaymentProvider(request);
    
    // Create payment attempt record
    const attempt = await this.createPaymentAttempt(request, provider);
    
    try {
      // Process payment with selected provider
      const result = await this.processWithProvider(provider, request);
      
      // Update attempt with result
      await this.updatePaymentAttempt(attempt.id, result);
      
      // Create payment record if successful
      if (result.success) {
        const payment = await this.createPaymentRecord(request, result, provider);
        return {
          ...result,
          paymentId: payment.id
        };
      }
      
      // Handle failure - try fallback provider if available
      if (!result.success && this.shouldRetryWithFallback(result)) {
        return await this.retryWithFallbackProvider(request, attempt);
      }
      
      return result;
      
    } catch (error) {
      // Update attempt with error
      await this.updatePaymentAttempt(attempt.id, {
        success: false,
        status: 'FAILED',
        errorMessage: error.message
      });
      
      throw error;
    }
  }
  
  private async selectPaymentProvider(
    request: ProcessPaymentRequest
  ): Promise<PaymentProvider> {
    const providers = await this.getAvailableProviders(
      request.paymentMethodType,
      request.amount,
      request.currencyCode
    );
    
    // Filter by preferred provider if specified
    if (request.preferredProviderId) {
      const preferred = providers.find(p => p.id === request.preferredProviderId);
      if (preferred && preferred.isHealthy) {
        return preferred;
      }
    }
    
    // Sort by priority and health
    const sortedProviders = providers
      .filter(p => p.isHealthy && p.isActive)
      .sort((a, b) => {
        // Primary sort: priority (higher is better)
        if (a.priority !== b.priority) {
          return b.priority - a.priority;
        }
        // Secondary sort: success rate
        return b.successRate - a.successRate;
      });
    
    if (sortedProviders.length === 0) {
      throw new Error('No healthy payment providers available');
    }
    
    return sortedProviders[0];
  }
  
  private async processWithProvider(
    provider: PaymentProvider,
    request: ProcessPaymentRequest
  ): Promise<ProcessPaymentResponse> {
    const service = this.getProviderService(provider);
    
    const startTime = Date.now();
    
    try {
      const result = await service.processPayment(request);
      
      // Update provider statistics
      await this.updateProviderStats(provider.id, {
        success: result.success,
        processingTime: Date.now() - startTime,
        amount: request.amount
      });
      
      return result;
      
    } catch (error) {
      // Update provider statistics for failure
      await this.updateProviderStats(provider.id, {
        success: false,
        processingTime: Date.now() - startTime,
        amount: request.amount
      });
      
      throw error;
    }
  }
  
  private getProviderService(provider: PaymentProvider): BasePaymentService {
    switch (provider.providerType) {
      case 'CREDIT_CARD_PROCESSOR':
        if (provider.name === 'Stripe') {
          return new StripeService(provider);
        } else if (provider.name === 'Square') {
          return new SquareService(provider);
        } else if (provider.name === 'Authorize.Net') {
          return new AuthorizeNetService(provider);
        }
        break;
        
      case 'DIGITAL_WALLET':
        if (provider.name === 'PayPal') {
          return new PayPalService(provider);
        }
        break;
        
      case 'BANK_TRANSFER':
        return new ACHService(provider);
        
      default:
        throw new Error(`Unsupported payment provider: ${provider.name}`);
    }
    
    throw new Error(`No service implementation for provider: ${provider.name}`);
  }
}
```

### 5.2 Enhanced Stripe Service

```typescript
export class EnhancedStripeService extends BasePaymentService {
  private stripe: Stripe;
  
  constructor(provider: PaymentProvider) {
    super(provider);
    this.stripe = new Stripe(provider.secretKey!, {
      apiVersion: '2023-10-16'
    });
  }
  
  async processPayment(request: ProcessPaymentRequest): Promise<ProcessPaymentResponse> {
    const startTime = Date.now();
    
    try {
      let paymentIntent: Stripe.PaymentIntent;
      
      if (request.paymentMethodId) {
        // Use saved payment method
        paymentIntent = await this.createPaymentIntentWithSavedMethod(request);
      } else {
        // Create new payment intent
        paymentIntent = await this.createPaymentIntent(request);
      }
      
      // Confirm payment intent
      const confirmedIntent = await this.stripe.paymentIntents.confirm(
        paymentIntent.id,
        this.getConfirmationParams(request)
      );
      
      const processingTime = Date.now() - startTime;
      
      return this.formatStripeResponse(confirmedIntent, processingTime, request);
      
    } catch (error) {
      return {
        success: false,
        status: 'FAILED',
        providerId: this.provider.id,
        errorMessage: error.message,
        errorCode: error.code,
        amount: request.amount,
        fees: { processingFee: 0, platformFee: 0 }
      };
    }
  }
  
  async processACHPayment(request: ProcessPaymentRequest): Promise<ProcessPaymentResponse> {
    try {
      const paymentIntent = await this.stripe.paymentIntents.create({
        amount: Math.round(request.amount * 100), // Convert to cents
        currency: request.currencyCode || 'usd',
        payment_method_types: ['us_bank_account'],
        payment_method: request.bankAccountToken,
        confirm: true,
        metadata: {
          invoiceId: request.invoiceId,
          ...request.metadata
        }
      });
      
      return this.formatStripeResponse(paymentIntent, 0, request);
      
    } catch (error) {
      return {
        success: false,
        status: 'FAILED',
        providerId: this.provider.id,
        errorMessage: error.message,
        errorCode: error.code,
        amount: request.amount,
        fees: { processingFee: 0, platformFee: 0 }
      };
    }
  }
  
  async setupFuturePayment(
    clientId: string,
    paymentMethodType: PaymentMethodType
  ): Promise<{ clientSecret: string; setupIntentId: string }> {
    const setupIntent = await this.stripe.setupIntents.create({
      customer: await this.getOrCreateStripeCustomer(clientId),
      payment_method_types: this.getStripePaymentMethodTypes(paymentMethodType),
      usage: 'off_session'
    });
    
    return {
      clientSecret: setupIntent.client_secret!,
      setupIntentId: setupIntent.id
    };
  }
  
  async savePaymentMethod(
    clientId: string,
    setupIntentId: string
  ): Promise<SavedPaymentMethod> {
    const setupIntent = await this.stripe.setupIntents.retrieve(setupIntentId);
    
    if (setupIntent.status !== 'succeeded' || !setupIntent.payment_method) {
      throw new Error('Setup intent not completed successfully');
    }
    
    const paymentMethod = await this.stripe.paymentMethods.retrieve(
      setupIntent.payment_method as string
    );
    
    // Save to database
    const savedMethod = await prisma.savedPaymentMethod.create({
      data: {
        clientId: parseInt(clientId),
        paymentProviderId: parseInt(this.provider.id),
        providerMethodId: paymentMethod.id,
        methodType: this.mapStripeMethodType(paymentMethod.type),
        displayName: this.formatPaymentMethodDisplay(paymentMethod),
        last4: paymentMethod.card?.last4 || paymentMethod.us_bank_account?.last4,
        brand: paymentMethod.card?.brand || paymentMethod.us_bank_account?.bank_name,
        expiryMonth: paymentMethod.card?.exp_month,
        expiryYear: paymentMethod.card?.exp_year,
        accountType: paymentMethod.us_bank_account?.account_type
      }
    });
    
    return savedMethod;
  }
  
  private async createPaymentIntent(
    request: ProcessPaymentRequest
  ): Promise<Stripe.PaymentIntent> {
    const customer = await this.getOrCreateStripeCustomer(
      request.invoiceId // We'll extract client ID from invoice
    );
    
    return await this.stripe.paymentIntents.create({
      amount: Math.round(request.amount * 100), // Convert to cents
      currency: request.currencyCode || 'usd',
      customer: customer.id,
      payment_method_types: this.getStripePaymentMethodTypes(request.paymentMethodType),
      metadata: {
        invoiceId: request.invoiceId,
        ...request.metadata
      },
      // Enable automatic payment methods for wallet payments
      automatic_payment_methods: {
        enabled: true,
        allow_redirects: 'never'
      }
    });
  }
  
  private getStripePaymentMethodTypes(methodType: PaymentMethodType): string[] {
    switch (methodType) {
      case 'CREDIT_CARD':
      case 'DEBIT_CARD':
        return ['card'];
      case 'ACH_BANK_TRANSFER':
        return ['us_bank_account'];
      case 'APPLE_PAY':
      case 'GOOGLE_PAY':
        return ['card']; // Wallets use card payment method type
      default:
        return ['card'];
    }
  }
  
  private calculateFees(amount: number, paymentMethodType: PaymentMethodType): {
    processingFee: number;
    platformFee: number;
  } {
    let feePercentage = this.provider.percentageFee;
    
    // Use ACH fee for bank transfers
    if (paymentMethodType === 'ACH_BANK_TRANSFER') {
      feePercentage = this.provider.achFee;
    }
    
    const processingFee = (amount * feePercentage / 100) + this.provider.flatFee;
    const platformFee = 0; // Could add platform fees here
    
    return {
      processingFee: Math.round(processingFee * 100) / 100,
      platformFee
    };
  }
}
```

### 5.3 ACH Payment Service

```typescript
export class ACHService extends BasePaymentService {
  async processPayment(request: ProcessPaymentRequest): Promise<ProcessPaymentResponse> {
    // ACH payments typically take 3-5 business days
    const startTime = Date.now();
    
    try {
      // Create ACH payment through provider (Stripe, Plaid, etc.)
      const achPayment = await this.createACHPayment(request);
      
      const processingTime = Date.now() - startTime;
      
      return {
        success: true,
        status: 'PROCESSING', // ACH payments are typically async
        providerId: this.provider.id,
        providerTransactionId: achPayment.id,
        amount: request.amount,
        fees: this.calculateACHFees(request.amount)
      };
      
    } catch (error) {
      return {
        success: false,
        status: 'FAILED',
        providerId: this.provider.id,
        errorMessage: error.message,
        amount: request.amount,
        fees: { processingFee: 0, platformFee: 0 }
      };
    }
  }
  
  private async createACHPayment(request: ProcessPaymentRequest): Promise<any> {
    // Implementation depends on ACH provider
    // This could use Stripe ACH, Plaid, or direct ACH processor
    
    const bankAccount = await this.getBankAccountDetails(request.bankAccountToken!);
    
    return await this.provider.apiClient.createACHPayment({
      amount: request.amount,
      routingNumber: bankAccount.routingNumber,
      accountNumber: bankAccount.accountNumber,
      accountType: bankAccount.accountType,
      accountHolderName: bankAccount.accountHolderName,
      description: `Payment for Invoice ${request.invoiceId}`,
      metadata: request.metadata
    });
  }
  
  private calculateACHFees(amount: number): { processingFee: number; platformFee: number } {
    // ACH fees are typically much lower than credit card fees
    const processingFee = Math.max(
      amount * (this.provider.achFee / 100),
      0.25 // Minimum ACH fee
    );
    
    return {
      processingFee: Math.round(processingFee * 100) / 100,
      platformFee: 0
    };
  }
}
```

### 5.4 Payment Plan Service

```typescript
export class PaymentPlanService {
  async createPaymentPlan(
    invoiceId: string,
    planConfig: {
      numberOfInstallments: number;
      frequency: PaymentFrequency;
      startDate: string;
      paymentMethodId?: string;
    }
  ): Promise<PaymentPlan> {
    const invoice = await this.getInvoice(invoiceId);
    
    const installmentAmount = Math.round(
      (invoice.remainingBalance / planConfig.numberOfInstallments) * 100
    ) / 100;
    
    // Calculate payment dates
    const paymentDates = this.calculatePaymentDates(
      new Date(planConfig.startDate),
      planConfig.numberOfInstallments,
      planConfig.frequency
    );
    
    const paymentPlan = await prisma.paymentPlan.create({
      data: {
        invoiceId: parseInt(invoiceId),
        clientId: invoice.clientId,
        planName: `${planConfig.numberOfInstallments}-Payment Plan`,
        totalAmount: invoice.remainingBalance,
        numberOfInstallments: planConfig.numberOfInstallments,
        installmentAmount,
        frequency: planConfig.frequency,
        startDate: new Date(planConfig.startDate),
        endDate: paymentDates[paymentDates.length - 1],
        nextPaymentDate: paymentDates[0],
        remainingAmount: invoice.remainingBalance,
        paymentMethodId: planConfig.paymentMethodId ? parseInt(planConfig.paymentMethodId) : null,
        createdBy: this.getCurrentUserId()
      }
    });
    
    // Schedule payment reminders
    await this.schedulePaymentReminders(paymentPlan, paymentDates);
    
    return paymentPlan;
  }
  
  async processScheduledPayment(paymentPlanId: string): Promise<ProcessPaymentResponse> {
    const plan = await this.getPaymentPlan(paymentPlanId);
    
    if (plan.status !== 'ACTIVE' || !plan.nextPaymentDate) {
      throw new Error('Payment plan is not active or has no scheduled payments');
    }
    
    const paymentRequest: ProcessPaymentRequest = {
      invoiceId: plan.invoiceId,
      amount: plan.installmentAmount,
      paymentMethodType: 'CREDIT_CARD', // Default, will be overridden by stored method
      paymentMethodId: plan.paymentMethodId,
      metadata: {
        paymentPlanId: plan.id,
        installmentNumber: plan.paidInstallments + 1,
        totalInstallments: plan.numberOfInstallments
      }
    };
    
    try {
      const result = await this.paymentRouter.processPayment(paymentRequest);
      
      if (result.success) {
        // Update payment plan progress
        await this.updatePaymentPlanProgress(plan, result);
      } else {
        // Handle failed payment
        await this.handleFailedInstallment(plan, result);
      }
      
      return result;
      
    } catch (error) {
      await this.handleFailedInstallment(plan, {
        success: false,
        status: 'FAILED',
        errorMessage: error.message,
        providerId: '',
        amount: plan.installmentAmount,
        fees: { processingFee: 0, platformFee: 0 }
      });
      
      throw error;
    }
  }
  
  private async updatePaymentPlanProgress(
    plan: PaymentPlan,
    paymentResult: ProcessPaymentResponse
  ): Promise<void> {
    const newPaidInstallments = plan.paidInstallments + 1;
    const newRemainingAmount = plan.remainingAmount - plan.installmentAmount;
    
    const updates: any = {
      paidInstallments: newPaidInstallments,
      remainingAmount: Math.max(0, newRemainingAmount)
    };
    
    // Check if plan is complete
    if (newPaidInstallments >= plan.numberOfInstallments || newRemainingAmount <= 0) {
      updates.status = 'COMPLETED';
      updates.nextPaymentDate = null;
    } else {
      // Calculate next payment date
      updates.nextPaymentDate = this.calculateNextPaymentDate(
        plan.nextPaymentDate!,
        plan.frequency
      );
    }
    
    await prisma.paymentPlan.update({
      where: { id: plan.id },
      data: updates
    });
    
    // Send completion notification if plan is finished
    if (updates.status === 'COMPLETED') {
      await this.sendPaymentPlanCompletionNotification(plan);
    }
  }
  
  private calculatePaymentDates(
    startDate: Date,
    numberOfInstallments: number,
    frequency: PaymentFrequency
  ): Date[] {
    const dates: Date[] = [];
    let currentDate = new Date(startDate);
    
    for (let i = 0; i < numberOfInstallments; i++) {
      dates.push(new Date(currentDate));
      
      // Calculate next date based on frequency
      switch (frequency) {
        case 'WEEKLY':
          currentDate.setDate(currentDate.getDate() + 7);
          break;
        case 'BIWEEKLY':
          currentDate.setDate(currentDate.getDate() + 14);
          break;
        case 'MONTHLY':
          currentDate.setMonth(currentDate.getMonth() + 1);
          break;
        case 'QUARTERLY':
          currentDate.setMonth(currentDate.getMonth() + 3);
          break;
        case 'ANNUALLY':
          currentDate.setFullYear(currentDate.getFullYear() + 1);
          break;
      }
    }
    
    return dates;
  }
}
```

---

## 6. User Interface Enhancements

### 6.1 Enhanced Payment Form

```typescript
interface EnhancedPaymentFormProps {
  invoice: Invoice;
  onPaymentSuccess: (payment: Payment) => void;
  onPaymentError: (error: string) => void;
}

export function EnhancedPaymentForm({ invoice, onPaymentSuccess, onPaymentError }: EnhancedPaymentFormProps) {
  const [paymentMethod, setPaymentMethod] = useState<PaymentMethodType>('CREDIT_CARD');
  const [useStoredMethod, setUseStoredMethod] = useState(false);
  const [selectedStoredMethod, setSelectedStoredMethod] = useState<string>('');
  const [showPaymentPlan, setShowPaymentPlan] = useState(false);
  const [isProcessing, setIsProcessing] = useState(false);

  const { data: paymentProviders } = useQuery({
    queryKey: ['payment-providers', paymentMethod, invoice.remainingBalance],
    queryFn: () => getPaymentProviders({
      methodType: paymentMethod,
      amount: invoice.remainingBalance,
      currency: 'USD'
    })
  });

  const { data: storedMethods } = useQuery({
    queryKey: ['stored-payment-methods', invoice.clientId],
    queryFn: () => getStoredPaymentMethods(invoice.clientId)
  });

  const processPayment = async (paymentData: any) => {
    setIsProcessing(true);
    
    try {
      const request: ProcessPaymentRequest = {
        invoiceId: invoice.id,
        amount: paymentData.amount || invoice.remainingBalance,
        paymentMethodType: paymentMethod,
        paymentMethodId: useStoredMethod ? selectedStoredMethod : undefined,
        cardToken: paymentData.cardToken,
        bankAccountToken: paymentData.bankAccountToken,
        billingAddress: paymentData.billingAddress,
        savePaymentMethod: paymentData.savePaymentMethod,
        metadata: {
          customerNote: paymentData.notes
        }
      };
      
      const result = await processEnhancedPayment(request);
      
      if (result.success) {
        onPaymentSuccess(result.payment);
        toast.success('Payment processed successfully!');
      } else {
        onPaymentError(result.errorMessage || 'Payment failed');
        
        // Show specific error messages
        if (result.errorCode === 'insufficient_funds') {
          toast.error('Insufficient funds. Please try a different payment method.');
        } else if (result.errorCode === 'card_declined') {
          toast.error('Card declined. Please check your card details or try a different card.');
        }
      }
      
    } catch (error) {
      onPaymentError(error.message);
      toast.error('Payment processing failed: ' + error.message);
    } finally {
      setIsProcessing(false);
    }
  };

  const renderPaymentMethodSelector = () => (
    <FormControl fullWidth sx={{ mb: 3 }}>
      <InputLabel>Payment Method</InputLabel>
      <Select
        value={paymentMethod}
        onChange={(e) => setPaymentMethod(e.target.value as PaymentMethodType)}
      >
        <MenuItem value="CREDIT_CARD">
          <Box sx={{ display: 'flex', alignItems: 'center', gap: 1 }}>
            <CreditCard />
            Credit/Debit Card
            <Chip label="2.9%" size="small" />
          </Box>
        </MenuItem>
        <MenuItem value="ACH_BANK_TRANSFER">
          <Box sx={{ display: 'flex', alignItems: 'center', gap: 1 }}>
            <AccountBalance />
            Bank Transfer (ACH)
            <Chip label="0.8%" size="small" color="success" />
          </Box>
        </MenuItem>
        <MenuItem value="PAYPAL">
          <Box sx={{ display: 'flex', alignItems: 'center', gap: 1 }}>
            <img src="/images/paypal-logo.png" alt="PayPal" height="20" />
            PayPal
          </Box>
        </MenuItem>
        <MenuItem value="APPLE_PAY">
          <Box sx={{ display: 'flex', alignItems: 'center', gap: 1 }}>
            <img src="/images/apple-pay-logo.png" alt="Apple Pay" height="20" />
            Apple Pay
          </Box>
        </MenuItem>
        <MenuItem value="GOOGLE_PAY">
          <Box sx={{ display: 'flex', alignItems: 'center', gap: 1 }}>
            <img src="/images/google-pay-logo.png" alt="Google Pay" height="20" />
            Google Pay
          </Box>
        </MenuItem>
      </Select>
    </FormControl>
  );

  const renderStoredMethods = () => (
    <Box sx={{ mb: 3 }}>
      <FormControlLabel
        control={
          <Checkbox
            checked={useStoredMethod}
            onChange={(e) => setUseStoredMethod(e.target.checked)}
          />
        }
        label="Use saved payment method"
      />
      
      {useStoredMethod && storedMethods && (
        <FormControl fullWidth sx={{ mt: 2 }}>
          <InputLabel>Saved Payment Methods</InputLabel>
          <Select
            value={selectedStoredMethod}
            onChange={(e) => setSelectedStoredMethod(e.target.value)}
          >
            {storedMethods.map(method => (
              <MenuItem key={method.id} value={method.id}>
                <Box sx={{ display: 'flex', alignItems: 'center', gap: 2 }}>
                  <PaymentMethodIcon type={method.methodType} />
                  <Box>
                    <Typography variant="body2">
                      {method.displayName}
                    </Typography>
                    <Typography variant="caption" color="text.secondary">
                      Last used: {method.lastUsedAt ? format(new Date(method.lastUsedAt), 'MMM d, yyyy') : 'Never'}
                    </Typography>
                  </Box>
                  {method.isDefault && (
                    <Chip label="Default" size="small" color="primary" />
                  )}
                </Box>
              </MenuItem>
            ))}
          </Select>
        </FormControl>
      )}
    </Box>
  );

  const renderPaymentOptions = () => (
    <Grid container spacing={2} sx={{ mb: 3 }}>
      <Grid item xs={12} sm={6}>
        <Button
          fullWidth
          variant="contained"
          size="large"
          onClick={() => processPayment({ amount: invoice.remainingBalance })}
          disabled={isProcessing}
          startIcon={isProcessing ? <CircularProgress size={20} /> : <Payment />}
        >
          {isProcessing ? 'Processing...' : `Pay Full Amount ($${invoice.remainingBalance.toLocaleString()})`}
        </Button>
      </Grid>
      
      <Grid item xs={12} sm={6}>
        <Button
          fullWidth
          variant="outlined"
          size="large"
          onClick={() => setShowPaymentPlan(true)}
          startIcon={<Schedule />}
        >
          Set Up Payment Plan
        </Button>
      </Grid>
    </Grid>
  );

  return (
    <Card sx={{ p: 3 }}>
      <Typography variant="h6" gutterBottom>
        Payment Options
      </Typography>

      <Alert severity="info" sx={{ mb: 3 }}>
        <Typography variant="body2">
          Amount due: <strong>${invoice.remainingBalance.toLocaleString()}</strong>
        </Typography>
        <Typography variant="caption">
          Due date: {format(new Date(invoice.dueDate), 'MMMM d, yyyy')}
        </Typography>
      </Alert>

      {renderPaymentMethodSelector()}
      {renderStoredMethods()}

      {/* Payment Provider Info */}
      {paymentProviders && paymentProviders.length > 0 && (
        <Alert severity="success" sx={{ mb: 3 }}>
          <Typography variant="body2">
            Processed securely by {paymentProviders[0].name}
          </Typography>
          <Typography variant="caption">
            Processing fee: {paymentProviders[0].percentageFee}% + ${paymentProviders[0].flatFee}
          </Typography>
        </Alert>
      )}

      {/* Payment Method Forms */}
      {!useStoredMethod && (
        <Box sx={{ mb: 3 }}>
          {paymentMethod === 'CREDIT_CARD' && (
            <CreditCardForm onSubmit={processPayment} isProcessing={isProcessing} />
          )}
          {paymentMethod === 'ACH_BANK_TRANSFER' && (
            <BankAccountForm onSubmit={processPayment} isProcessing={isProcessing} />
          )}
          {['PAYPAL', 'APPLE_PAY', 'GOOGLE_PAY'].includes(paymentMethod) && (
            <DigitalWalletForm 
              methodType={paymentMethod as PaymentMethodType}
              onSubmit={processPayment} 
              isProcessing={isProcessing} 
            />
          )}
        </Box>
      )}

      {renderPaymentOptions()}

      {/* Payment Plan Dialog */}
      {showPaymentPlan && (
        <PaymentPlanDialog
          invoice={invoice}
          open={showPaymentPlan}
          onClose={() => setShowPaymentPlan(false)}
          onPlanCreated={() => {
            setShowPaymentPlan(false);
            toast.success('Payment plan created successfully');
          }}
        />
      )}
    </Card>
  );
}
```

### 6.2 Payment Provider Dashboard

```typescript
export function PaymentProviderDashboard() {
  const [selectedProvider, setSelectedProvider] = useState<string>('');

  const { data: providers } = useQuery({
    queryKey: ['payment-providers'],
    queryFn: () => getPaymentProviders()
  });

  const { data: providerStats } = useQuery({
    queryKey: ['provider-stats', selectedProvider],
    queryFn: () => getProviderStats(selectedProvider),
    enabled: !!selectedProvider
  });

  return (
    <Container maxWidth="xl">
      <Typography variant="h4" gutterBottom>
        Payment Provider Dashboard
      </Typography>

      {/* Provider Overview Cards */}
      <Grid container spacing={3} sx={{ mb: 4 }}>
        {providers?.map(provider => (
          <Grid item xs={12} sm={6} md={4} key={provider.id}>
            <Card 
              sx={{ 
                p: 2, 
                cursor: 'pointer',
                border: selectedProvider === provider.id ? 2 : 0,
                borderColor: 'primary.main'
              }}
              onClick={() => setSelectedProvider(provider.id)}
            >
              <Box sx={{ display: 'flex', alignItems: 'center', mb: 2 }}>
                <Avatar sx={{ bgcolor: provider.isHealthy ? 'success.main' : 'error.main' }}>
                  {provider.name.charAt(0)}
                </Avatar>
                <Box sx={{ ml: 2 }}>
                  <Typography variant="h6">{provider.name}</Typography>
                  <Typography variant="caption" color="text.secondary">
                    {provider.providerType.replace('_', ' ')}
                  </Typography>
                </Box>
                <Box sx={{ ml: 'auto' }}>
                  <Chip 
                    label={provider.isHealthy ? 'Healthy' : 'Unhealthy'}
                    color={provider.isHealthy ? 'success' : 'error'}
                    size="small"
                  />
                </Box>
              </Box>
              
              <Grid container spacing={2}>
                <Grid item xs={6}>
                  <Typography variant="body2" color="text.secondary">Success Rate</Typography>
                  <Typography variant="h6">{provider.successRate.toFixed(1)}%</Typography>
                </Grid>
                <Grid item xs={6}>
                  <Typography variant="body2" color="text.secondary">Avg Response</Typography>
                  <Typography variant="h6">{provider.avgProcessingTime}ms</Typography>
                </Grid>
              </Grid>
            </Card>
          </Grid>
        ))}
      </Grid>

      {/* Provider Details */}
      {selectedProvider && providerStats && (
        <Card sx={{ p: 3 }}>
          <Typography variant="h6" gutterBottom>
            Provider Statistics
          </Typography>
          
          <Grid container spacing={3}>
            <Grid item xs={12} sm={6} md={3}>
              <MetricCard
                title="Total Transactions"
                value={providerStats.totalTransactions.toLocaleString()}
                icon={<Receipt />}
              />
            </Grid>
            <Grid item xs={12} sm={6} md={3}>
              <MetricCard
                title="Total Volume"
                value={`$${providerStats.totalVolume.toLocaleString()}`}
                icon={<AttachMoney />}
              />
            </Grid>
            <Grid item xs={12} sm={6} md={3}>
              <MetricCard
                title="Success Rate"
                value={`${providerStats.successRate.toFixed(1)}%`}
                icon={<CheckCircle />}
                color={providerStats.successRate >= 95 ? 'success' : 'warning'}
              />
            </Grid>
            <Grid item xs={12} sm={6} md={3}>
              <MetricCard
                title="Avg Processing Time"
                value={`${providerStats.avgProcessingTime}ms`}
                icon={<Timer />}
              />
            </Grid>
          </Grid>

          {/* Transaction Method Breakdown */}
          <Box sx={{ mt: 4 }}>
            <Typography variant="h6" gutterBottom>
              Transactions by Payment Method
            </Typography>
            <Grid container spacing={2}>
              {Object.entries(providerStats.transactionsByMethod).map(([method, count]) => (
                <Grid item xs={12} sm={6} md={4} key={method}>
                  <Box sx={{ p: 2, bgcolor: 'grey.50', borderRadius: 1 }}>
                    <Typography variant="subtitle2">{method.replace('_', ' ')}</Typography>
                    <Typography variant="h5">{count}</Typography>
                  </Box>
                </Grid>
              ))}
            </Grid>
          </Box>
        </Card>
      )}
    </Container>
  );
}
```

---

## 7. Implementation Plan

### Phase 1: Core Infrastructure (Week 1-2)
- [ ] Database schema extensions for payment providers and methods
- [ ] Payment router service architecture
- [ ] Enhanced payment provider interfaces
- [ ] Basic ACH payment processing capability

### Phase 2: Provider Integrations (Week 2-4)
- [ ] Enhanced Stripe service with ACH and digital wallets
- [ ] PayPal enhanced integration
- [ ] Square payment processor integration
- [ ] Authorize.Net integration
- [ ] Provider health monitoring and failover

### Phase 3: Advanced Payment Features (Week 4-5)
- [ ] Saved payment methods management
- [ ] Payment plan creation and management
- [ ] Multi-currency support
- [ ] International payment methods
- [ ] Fraud detection and risk scoring

### Phase 4: User Interface (Week 5-6)
- [ ] Enhanced payment form with multiple methods
- [ ] Payment provider dashboard
- [ ] Payment plan management interface
- [ ] Saved payment methods management
- [ ] Payment analytics and reporting

### Phase 5: Testing & Optimization (Week 6)
- [ ] End-to-end payment flow testing
- [ ] Provider failover testing
- [ ] Performance optimization
- [ ] Security audit
- [ ] Production deployment

---

## 8. Success Criteria

### Functional Requirements
- [ ] Support 5+ payment providers with automatic failover
- [ ] Process ACH payments with 0.8% fees vs 2.9% credit cards
- [ ] Enable saved payment methods for repeat customers
- [ ] Support payment plans with automated installment processing
- [ ] Handle multi-currency payments for international clients
- [ ] Provide comprehensive payment analytics and reporting

### Technical Requirements
- [ ] Payment processing success rate >98%
- [ ] Average payment processing time <3 seconds
- [ ] Support for payment volumes up to $1M/month
- [ ] PCI DSS compliance for credit card processing
- [ ] Secure handling of sensitive payment data
- [ ] Real-time payment status updates and notifications

### Business Requirements
- [ ] Reduce payment processing costs by 30% through ACH adoption
- [ ] Increase payment success rates by 15% through provider diversity
- [ ] Improve customer payment experience with saved methods
- [ ] Enable payment plan options to increase large invoice collections
- [ ] Support international expansion with multi-currency payments
- [ ] Provide detailed payment analytics for business insights

---

**File Version:** 1.0 – Payment Gateway Enhancement Specification  
**Last Updated:** October 12, 2025