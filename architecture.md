# Project Management Application Architecture

## Data Model

### Core Entities

```mermaid
erDiagram
    Account {
        string id
        string name
        string subscription_tier
        datetime created_at
        datetime updated_at
    }
    User {
        string id
        string account_id
        string email
        string name
        enum role
        datetime created_at
        datetime updated_at
    }
    Project {
        string id
        string account_id
        string name
        string description
        string client_id
        enum status
        decimal budget_amount
        decimal budget_threshold
        datetime start_date
        datetime end_date
        User[] assigned_users
        datetime created_at
        datetime updated_at
    }
    Milestone {
        string id
        string project_id
        string title
        string description
        datetime target_date
        enum status
        datetime created_at
    }
    Client {
        string id
        string account_id
        string name
        string email
        string contact_info
        datetime created_at
        datetime updated_at
    }
    Quote {
        string id
        string account_id
        string project_id
        int version
        enum status
        string currency
        decimal total_amount
        decimal tax_amount
        decimal discount_amount
        datetime valid_until
        PaymentTerms payment_terms
        datetime created_at
    }
    QuoteLineItem {
        string id
        string quote_id
        string group_id
        string description
        decimal unit_price
        int quantity
        decimal tax_rate
        string tax_category
        decimal total
    }
    Invoice {
        string id
        string account_id
        string project_id
        string quote_id
        enum status
        string currency
        decimal total_amount
        decimal tax_amount
        decimal amount_paid
        PaymentTerms payment_terms
        RecurringConfig recurring_config
        datetime due_date
        datetime created_at
    }
    InvoiceLineItem {
        string id
        string invoice_id
        string group_id
        string description
        decimal unit_price
        int quantity
        decimal tax_rate
        string tax_category
        decimal total
    }
    PaymentTerms {
        string id
        enum term_type
        int days_due
        bool allows_partial
        string[] schedule
        datetime created_at
    }
    TaxRule {
        string id
        string jurisdiction
        string category
        decimal rate
        bool is_exempt
        datetime effective_from
        datetime effective_to
    }
    Payment {
        string id
        string invoice_id
        string provider_id
        decimal amount
        string currency
        enum status
        datetime processed_at
    }
    RecurringConfig {
        string id
        enum interval
        datetime start_date
        datetime end_date
        int day_of_month
        bool is_active
    }

    Account ||--o{ User : has
    Account ||--o{ Project : owns
    Account ||--o{ Client : manages
    Account ||--o{ Quote : contains
    Account ||--o{ Invoice : contains
    User }o--o{ Project : assigned_to
    Project ||--o{ Milestone : has
    Project ||--o{ Quote : has
    Project ||--o{ Invoice : has
    Client ||--o{ Project : has
    Quote ||--o{ QuoteLineItem : contains
    Quote ||--o{ TaxRule : applies
    Invoice ||--o{ InvoiceLineItem : contains
    Invoice ||--o{ TaxRule : applies
    Invoice ||--o{ Payment : receives
```

### Entity Relationships
- Account is the top-level entity for multi-tenant separation
- All major entities belong to an Account
- Projects now include budget tracking and milestones
- Quotes and Invoices support multiple currencies and tax rules
- Payment terms can be customized and inherited
- Tax rules support location-based calculations and exemptions
- Simple recurring invoice configuration
- Payments track Stripe integration with extensibility

## Component Architecture

```mermaid
graph TD
    A[App] --> B[AuthProvider]
    B --> C[Layout]
    C --> D[Navigation]
    C --> E[MainContent]
    
    E --> F[Dashboard]
    E --> AA[AccountSettings]
    E --> G[ProjectModule]
    E --> H[QuoteModule]
    E --> I[InvoiceModule]
    E --> J[ClientModule]
    E --> K[TaxModule]
    E --> L[PaymentModule]
    
    G --> G1[ProjectList]
    G --> G2[ProjectDetail]
    G --> G3[ProjectForm]
    G --> G4[MilestoneTracker]
    G --> G5[BudgetMonitor]
    
    H --> H1[QuoteList]
    H --> H2[QuoteDetail]
    H --> H3[QuoteForm]
    H --> H4[LineItemEditor]
    H --> H5[TaxCalculator]
    
    I --> I1[InvoiceList]
    I --> I2[InvoiceDetail]
    I --> I3[InvoiceForm]
    I --> I4[LineItemEditor]
    I --> I5[PaymentProcessor]
    I --> I6[RecurringManager]
    
    K --> K1[TaxRuleManager]
    K --> K2[TaxCalculator]
    K --> K3[ReportGenerator]
    
    L --> L1[PaymentsList]
    L --> L2[StripeIntegration]
    L --> L3[PaymentTermsManager]
```

### Reusable Components
- LineItemEditor: Used in both quotes and invoices
- TaxCalculator: Shared tax computation logic
- PaymentProcessor: Abstract payment handling
- StatusBadge: Display status across entities
- AlertManager: Simple in-app notifications
- FilterPanel: Reusable filtering interface
- TableComponent: Sortable, filterable data table

### State Management
- React Context for:
  - Authentication state
  - Current account context
  - User preferences
  - Alert notifications
  - Global UI state
- React Query for:
  - Data fetching and caching
  - Server state management
  - Real-time updates
- Local component state for UI-specific state

## System Flows

### Quote to Project to Invoice Flow

```mermaid
stateDiagram-v2
    [*] --> QuoteDraft
    QuoteDraft --> QuoteSent: Calculate Tax
    QuoteSent --> QuoteAccepted
    QuoteSent --> QuoteRejected
    QuoteAccepted --> ProjectCreated
    ProjectCreated --> ProjectInProgress
    ProjectInProgress --> ProjectCompleted
    ProjectInProgress --> InvoiceCreated
    ProjectCompleted --> InvoiceCreated
    InvoiceCreated --> InvoiceSent: Calculate Tax
    InvoiceSent --> ProcessingPayment
    ProcessingPayment --> PaymentReceived
    PaymentReceived --> [*]
```

### Payment Processing Flow

```mermaid
stateDiagram-v2
    [*] --> InvoiceSent
    InvoiceSent --> PaymentInitiated: Stripe
    PaymentInitiated --> ProcessingPayment
    ProcessingPayment --> PaymentFailed
    ProcessingPayment --> PaymentSucceeded
    PaymentFailed --> RetryPayment
    RetryPayment --> ProcessingPayment
    PaymentSucceeded --> [*]
```

## API Endpoints

### Project Management
- GET /api/projects
- GET /api/projects/:id
- POST /api/projects
- PUT /api/projects/:id
- GET /api/projects/:id/milestones
- POST /api/projects/:id/milestones
- PUT /api/projects/:id/milestones/:milestoneId

### Financial Management
- GET /api/quotes
- POST /api/quotes
- PUT /api/quotes/:id
- POST /api/quotes/:id/calculate-tax
- GET /api/invoices
- POST /api/invoices
- PUT /api/invoices/:id
- POST /api/invoices/:id/calculate-tax
- POST /api/invoices/:id/payments
- GET /api/payments
- GET /api/payment-terms
- POST /api/payment-terms

### Tax Management
- GET /api/tax-rules
- POST /api/tax-rules
- PUT /api/tax-rules/:id
- GET /api/tax-reports/monthly
- GET /api/tax-reports/quarterly
- POST /api/tax-reports/export

### Budget and Alerts
- GET /api/projects/:id/budget
- POST /api/projects/:id/budget/threshold
- GET /api/alerts

## Data Export/Import

### Export Format
```json
{
  "metadata": {
    "version": "1.0",
    "export_date": "ISO-8601 timestamp",
    "account_id": "string"
  },
  "data": {
    "clients": [...],
    "projects": [...],
    "quotes": [...],
    "invoices": [...],
    "payments": [...],
    "tax_rules": [...]
  }
}
```

### Export Types
1. Full Account Export
   - All account data
   - Tax rules and history
   - Payment records
2. Financial Reports
   - Monthly/Quarterly tax reports
   - Payment summaries
   - Budget reports

All API endpoints automatically scope data to the current account context based on the authenticated user's account_id.