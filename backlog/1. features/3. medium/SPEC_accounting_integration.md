# Accounting System Integration Specification

**Date:** October 12, 2025  
**Version:** 1.0  
**Status:** Not Implemented  

---

## 1. Overview

The Accounting System Integration provides seamless synchronization between the ProjectLedger billing system and external accounting platforms (QuickBooks Online, QuickBooks Desktop, Xero). It handles automatic export of invoices, payments, clients, and financial data while maintaining data integrity and compliance with accounting standards.

---

## 2. Business Requirements

### 2.1 Core Integration Features
- **Automated Invoice Export**: Sync issued invoices to accounting system
- **Payment Reconciliation**: Match payments between systems
- **Client/Customer Sync**: Maintain consistent customer records
- **Chart of Accounts Mapping**: Map billing categories to accounting accounts
- **Tax Code Synchronization**: Align tax handling between systems
- **Credit Note Integration**: Export credit notes and adjustments

### 2.2 Supported Accounting Systems
- **QuickBooks Online**: Cloud-based integration via REST API
- **QuickBooks Desktop**: Integration via QB SDK (Windows only)
- **Xero**: Cloud-based integration via REST API
- **Generic Export**: CSV/Excel export for manual import

### 2.3 Data Synchronization
- **Real-time Sync**: Immediate export of issued documents
- **Batch Processing**: Scheduled bulk synchronization
- **Selective Sync**: Choose which data types to synchronize
- **Conflict Resolution**: Handle data discrepancies between systems
- **Audit Trail**: Complete log of all synchronization activities

---

## 3. Data Model

### 3.1 Database Schema

```prisma
model AccountingIntegration {
  id                    Int                     @id @default(autoincrement())
  accountId             Int
  
  // Integration settings
  provider              AccountingProvider      // QUICKBOOKS_ONLINE, QUICKBOOKS_DESKTOP, XERO
  isActive              Boolean                 @default(false)
  
  // Authentication credentials (encrypted)
  accessToken           String?                 // OAuth access token
  refreshToken          String?                 // OAuth refresh token
  tokenExpiresAt        DateTime?               // Token expiration
  clientId              String?                 // OAuth client ID
  clientSecret          String?                 // OAuth client secret (encrypted)
  
  // QuickBooks specific
  companyId             String?                 // QB Company ID
  realmId               String?                 // QB Realm ID
  
  // Xero specific
  tenantId              String?                 // Xero Tenant ID
  organizationId        String?                 // Xero Organization ID
  
  // Sync configuration
  syncSettings          Json                    // Sync preferences and mappings
  lastSyncAt            DateTime?               // Last successful sync
  syncFrequency         SyncFrequency           @default(REAL_TIME)
  
  // Error handling
  lastError             String?                 // Last error message
  errorCount            Int                     @default(0)
  isHealthy             Boolean                 @default(true)
  
  // Metadata
  createdAt             DateTime                @default(now())
  updatedAt             DateTime                @updatedAt
  createdBy             Int
  
  // Relations
  account               Account                 @relation(fields: [accountId], references: [id], onDelete: Cascade)
  createdByUser         User                    @relation(fields: [createdBy], references: [id])
  syncLogs              AccountingSyncLog[]
  mappings              AccountingMapping[]
  
  @@unique([accountId, provider])
  @@index([accountId])
  @@index([provider])
}

model AccountingMapping {
  id                    Int                     @id @default(autoincrement())
  integrationId         Int
  
  // Mapping definition
  sourceType            MappingSourceType       // INVOICE, PAYMENT, CLIENT, TAX_CODE, ACCOUNT
  sourceField           String                  // Field name in ProjectLedger
  targetField           String                  // Field name in accounting system
  
  // Value mapping
  mappingType           MappingType             // DIRECT, LOOKUP, CALCULATED, CUSTOM
  defaultValue          String?                 // Default value if no mapping found
  valueMapping          Json?                   // Value mapping rules
  
  // Conditions
  conditions            Json?                   // Conditional mapping rules
  isRequired            Boolean                 @default(false)
  isActive              Boolean                 @default(true)
  
  // Metadata
  createdAt             DateTime                @default(now())
  updatedAt             DateTime                @updatedAt
  
  // Relations
  integration           AccountingIntegration   @relation(fields: [integrationId], references: [id], onDelete: Cascade)
  
  @@index([integrationId])
  @@index([sourceType])
}

model AccountingSyncLog {
  id                    Int                     @id @default(autoincrement())
  integrationId         Int
  
  // Sync details
  syncType              SyncType                // EXPORT, IMPORT, RECONCILE
  entityType            EntityType              // INVOICE, PAYMENT, CLIENT, etc.
  entityId              Int                     // ID of synced entity
  externalId            String?                 // ID in accounting system
  
  // Sync result
  status                SyncStatus              // SUCCESS, FAILED, PARTIAL, SKIPPED
  operation             SyncOperation           // CREATE, UPDATE, DELETE
  
  // Data payload
  requestData           Json?                   // Data sent to accounting system
  responseData          Json?                   // Response from accounting system
  
  // Error handling
  errorMessage          String?                 // Error details if failed
  errorCode             String?                 // Error code from accounting system
  retryCount            Int                     @default(0)
  nextRetryAt           DateTime?               // When to retry if failed
  
  // Timing
  startedAt             DateTime                @default(now())
  completedAt           DateTime?
  duration              Int?                    // Duration in milliseconds
  
  // Relations
  integration           AccountingIntegration   @relation(fields: [integrationId], references: [id], onDelete: Cascade)
  
  @@index([integrationId])
  @@index([status])
  @@index([syncType, entityType])
  @@index([startedAt])
}

model AccountingAccount {
  id                    Int                     @id @default(autoincrement())
  integrationId         Int
  
  // Account details from accounting system
  externalId            String                  // Account ID in accounting system
  name                  String                  // Account name
  accountType           String                  // Account type (Asset, Liability, etc.)
  accountNumber         String?                 // Account number
  description           String?                 // Account description
  
  // Hierarchy
  parentAccountId       String?                 // Parent account in accounting system
  level                 Int                     @default(0) // Account hierarchy level
  
  // Status
  isActive              Boolean                 @default(true)
  canPost               Boolean                 @default(true) // Can post transactions
  
  // Sync metadata
  lastSyncAt            DateTime                @default(now())
  
  // Relations
  integration           AccountingIntegration   @relation(fields: [integrationId], references: [id], onDelete: Cascade)
  
  @@unique([integrationId, externalId])
  @@index([integrationId])
  @@index([accountType])
}

model AccountingTaxCode {
  id                    Int                     @id @default(autoincrement())
  integrationId         Int
  
  // Tax code details from accounting system
  externalId            String                  // Tax code ID in accounting system
  name                  String                  // Tax code name
  description           String?                 // Tax code description
  rate                  Float                   // Tax rate (as decimal)
  
  // Tax code properties
  isActive              Boolean                 @default(true)
  isSalesTax            Boolean                 @default(true)
  isPurchaseTax         Boolean                 @default(false)
  
  // Mapping to ProjectLedger tax codes
  localTaxCodeId        Int?                    // Mapped local tax code
  
  // Sync metadata
  lastSyncAt            DateTime                @default(now())
  
  // Relations
  integration           AccountingIntegration   @relation(fields: [integrationId], references: [id], onDelete: Cascade)
  localTaxCode          TaxRate?                @relation(fields: [localTaxCodeId], references: [id])
  
  @@unique([integrationId, externalId])
  @@index([integrationId])
  @@index([localTaxCodeId])
}

enum AccountingProvider {
  QUICKBOOKS_ONLINE
  QUICKBOOKS_DESKTOP  
  XERO
  GENERIC_EXPORT
}

enum SyncFrequency {
  REAL_TIME       // Sync immediately when document is issued
  HOURLY          // Sync every hour
  DAILY           // Sync once per day
  WEEKLY          // Sync weekly
  MANUAL          // Manual sync only
}

enum MappingSourceType {
  INVOICE
  PAYMENT
  CLIENT
  TAX_CODE
  ACCOUNT
  LINE_ITEM
  CREDIT_NOTE
}

enum MappingType {
  DIRECT          // Direct field mapping
  LOOKUP          // Lookup from mapping table
  CALCULATED      // Calculated value
  CUSTOM          // Custom transformation
}

enum SyncType {
  EXPORT          // Export from ProjectLedger to accounting system
  IMPORT          // Import from accounting system to ProjectLedger
  RECONCILE       // Reconcile differences between systems
}

enum EntityType {
  INVOICE
  PAYMENT
  CLIENT
  TAX_CODE
  ACCOUNT
  CREDIT_NOTE
  ITEM
}

enum SyncStatus {
  SUCCESS
  FAILED
  PARTIAL
  SKIPPED
  PENDING
  IN_PROGRESS
}

enum SyncOperation {
  CREATE
  UPDATE
  DELETE
  READ
}
```

### 3.2 TypeScript Interfaces

```typescript
export interface AccountingIntegration {
  id: string;
  provider: AccountingProvider;
  isActive: boolean;
  
  // Authentication
  accessToken?: string;
  refreshToken?: string;
  tokenExpiresAt?: string;
  companyId?: string;
  realmId?: string;
  tenantId?: string;
  
  // Configuration
  syncSettings: AccountingSyncSettings;
  lastSyncAt?: string;
  syncFrequency: SyncFrequency;
  
  // Health
  lastError?: string;
  errorCount: number;
  isHealthy: boolean;
  
  mappings: AccountingMapping[];
}

export interface AccountingSyncSettings {
  // What to sync
  syncInvoices: boolean;
  syncPayments: boolean;
  syncClients: boolean;
  syncTaxCodes: boolean;
  
  // Invoice settings
  invoiceAccountId?: string;        // Revenue account
  invoiceTemplate?: string;         // Invoice template in QB/Xero
  invoiceTerms?: string;           // Payment terms
  
  // Payment settings
  bankAccountId?: string;          // Bank account for payments
  unidentifiedPaymentAccount?: string; // Account for unmatched payments
  
  // Client settings
  clientNameFormat: string;        // How to format client names
  clientAccountType: string;       // Customer/Client account type
  
  // Tax settings
  defaultTaxCode?: string;         // Default tax code mapping
  taxInclusive: boolean;           // Tax inclusive/exclusive
  
  // General settings
  dateFormat: string;              // Date format preference
  currencyCode: string;            // Currency code
  includeLineItemDetails: boolean; // Detailed line items vs summary
}

export interface AccountingMapping {
  id: string;
  sourceType: MappingSourceType;
  sourceField: string;
  targetField: string;
  mappingType: MappingType;
  defaultValue?: string;
  valueMapping?: Record<string, string>;
  conditions?: any;
  isRequired: boolean;
  isActive: boolean;
}

export interface SyncResult {
  status: SyncStatus;
  operation: SyncOperation;
  externalId?: string;
  errorMessage?: string;
  duration: number;
  data?: any;
}

export interface AccountingSyncLog {
  id: string;
  syncType: SyncType;
  entityType: EntityType;
  entityId: string;
  externalId?: string;
  status: SyncStatus;
  operation: SyncOperation;
  errorMessage?: string;
  startedAt: string;
  completedAt?: string;
  duration?: number;
}
```

---

## 4. API Endpoints

### 4.1 Integration Management

```typescript
// Create accounting integration
POST /api/accounting-integrations
Body: CreateIntegrationInput
Response: AccountingIntegration

// Get integration
GET /api/accounting-integrations/:id
Response: AccountingIntegration

// Update integration
PATCH /api/accounting-integrations/:id
Body: Partial<CreateIntegrationInput>
Response: AccountingIntegration

// Delete integration
DELETE /api/accounting-integrations/:id
Response: { success: boolean }

// List integrations
GET /api/accounting-integrations
Response: AccountingIntegration[]

// Test integration connection
POST /api/accounting-integrations/:id/test-connection
Response: {
  isConnected: boolean,
  companyInfo?: any,
  error?: string
}
```

### 4.2 OAuth Authentication

```typescript
// Get OAuth authorization URL
GET /api/accounting-integrations/oauth/authorize
Query: {
  provider: AccountingProvider,
  redirectUri: string
}
Response: {
  authorizationUrl: string,
  state: string
}

// Handle OAuth callback
POST /api/accounting-integrations/oauth/callback
Body: {
  provider: AccountingProvider,
  code: string,
  state: string,
  realmId?: string,
  companyId?: string
}
Response: AccountingIntegration

// Refresh access token
POST /api/accounting-integrations/:id/refresh-token
Response: {
  accessToken: string,
  expiresAt: string,
  success: boolean
}
```

### 4.3 Data Synchronization

```typescript
// Manual sync trigger
POST /api/accounting-integrations/:id/sync
Body: {
  entityTypes?: EntityType[],
  entityIds?: string[],
  force?: boolean
}
Response: {
  syncJobId: string,
  status: string
}

// Get sync status
GET /api/accounting-integrations/:id/sync-status/:jobId
Response: {
  status: string,
  progress: number,
  results: SyncResult[]
}

// Sync specific invoice
POST /api/accounting-integrations/:id/sync-invoice/:invoiceId
Response: SyncResult

// Sync specific payment
POST /api/accounting-integrations/:id/sync-payment/:paymentId
Response: SyncResult

// Bulk sync
POST /api/accounting-integrations/:id/bulk-sync
Body: {
  entityType: EntityType,
  dateFrom?: string,
  dateTo?: string,
  status?: string[]
}
Response: {
  syncJobId: string,
  totalItems: number
}
```

### 4.4 Account and Tax Code Management

```typescript
// Refresh chart of accounts
POST /api/accounting-integrations/:id/refresh-accounts
Response: {
  accountsCount: number,
  success: boolean
}

// Get accounts from accounting system
GET /api/accounting-integrations/:id/accounts
Query: {
  accountType?: string,
  search?: string
}
Response: AccountingAccount[]

// Refresh tax codes
POST /api/accounting-integrations/:id/refresh-tax-codes
Response: {
  taxCodesCount: number,
  success: boolean
}

// Get tax codes from accounting system
GET /api/accounting-integrations/:id/tax-codes
Response: AccountingTaxCode[]

// Map local tax code to accounting tax code
POST /api/accounting-integrations/:id/map-tax-code
Body: {
  localTaxCodeId: string,
  externalTaxCodeId: string
}
Response: { success: boolean }
```

### 4.5 Sync Logs and Reporting

```typescript
// Get sync logs
GET /api/accounting-integrations/:id/sync-logs
Query: {
  entityType?: EntityType,
  status?: SyncStatus,
  dateFrom?: string,
  dateTo?: string,
  limit?: number
}
Response: AccountingSyncLog[]

// Get sync statistics
GET /api/accounting-integrations/:id/sync-stats
Query: {
  dateFrom?: string,
  dateTo?: string
}
Response: {
  totalSyncs: number,
  successfulSyncs: number,
  failedSyncs: number,
  averageDuration: number,
  syncsByEntityType: Record<EntityType, number>,
  syncsByStatus: Record<SyncStatus, number>
}

// Export sync report
GET /api/accounting-integrations/:id/sync-report
Query: {
  format: 'CSV' | 'XLSX' | 'PDF',
  dateFrom?: string,
  dateTo?: string
}
Response: Binary file
```

---

## 5. Integration Services

### 5.1 Base Accounting Service

```typescript
export abstract class BaseAccountingService {
  protected integration: AccountingIntegration;
  
  constructor(integration: AccountingIntegration) {
    this.integration = integration;
  }
  
  // Abstract methods to be implemented by provider-specific services
  abstract authenticate(): Promise<boolean>;
  abstract refreshToken(): Promise<void>;
  abstract getCompanyInfo(): Promise<any>;
  abstract getAccounts(): Promise<AccountingAccount[]>;
  abstract getTaxCodes(): Promise<AccountingTaxCode[]>;
  abstract createInvoice(invoice: Invoice): Promise<SyncResult>;
  abstract updateInvoice(invoice: Invoice, externalId: string): Promise<SyncResult>;
  abstract createPayment(payment: Payment): Promise<SyncResult>;
  abstract createClient(client: Client): Promise<SyncResult>;
  abstract createCreditNote(creditNote: CreditNote): Promise<SyncResult>;
  
  // Common utility methods
  protected async logSync(
    entityType: EntityType,
    entityId: string,
    operation: SyncOperation,
    result: SyncResult
  ): Promise<void> {
    await prisma.accountingSyncLog.create({
      data: {
        integrationId: parseInt(this.integration.id),
        syncType: 'EXPORT',
        entityType,
        entityId: parseInt(entityId),
        externalId: result.externalId,
        status: result.status,
        operation,
        requestData: result.data?.request,
        responseData: result.data?.response,
        errorMessage: result.errorMessage,
        startedAt: new Date(Date.now() - result.duration),
        completedAt: new Date(),
        duration: result.duration
      }
    });
  }
  
  protected mapEntityData(entity: any, mappings: AccountingMapping[]): any {
    const mapped: any = {};
    
    for (const mapping of mappings) {
      const value = this.getFieldValue(entity, mapping.sourceField);
      const mappedValue = this.applyMapping(value, mapping);
      
      if (mappedValue !== null && mappedValue !== undefined) {
        this.setFieldValue(mapped, mapping.targetField, mappedValue);
      }
    }
    
    return mapped;
  }
  
  private applyMapping(value: any, mapping: AccountingMapping): any {
    switch (mapping.mappingType) {
      case 'DIRECT':
        return value;
        
      case 'LOOKUP':
        return mapping.valueMapping?.[value] || mapping.defaultValue;
        
      case 'CALCULATED':
        return this.calculateValue(value, mapping.valueMapping);
        
      case 'CUSTOM':
        return this.applyCustomMapping(value, mapping);
        
      default:
        return value;
    }
  }
}
```

### 5.2 QuickBooks Online Service

```typescript
export class QuickBooksOnlineService extends BaseAccountingService {
  private apiClient: QuickBooksAPIClient;
  
  constructor(integration: AccountingIntegration) {
    super(integration);
    this.apiClient = new QuickBooksAPIClient({
      accessToken: integration.accessToken!,
      realmId: integration.realmId!,
      sandbox: process.env.NODE_ENV !== 'production'
    });
  }
  
  async authenticate(): Promise<boolean> {
    try {
      const companyInfo = await this.getCompanyInfo();
      return !!companyInfo;
    } catch (error) {
      console.error('QuickBooks authentication failed:', error);
      return false;
    }
  }
  
  async refreshToken(): Promise<void> {
    const response = await fetch('https://oauth.platform.intuit.com/oauth2/v1/tokens/bearer', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
        'Authorization': `Basic ${Buffer.from(`${this.integration.clientId}:${this.integration.clientSecret}`).toString('base64')}`
      },
      body: new URLSearchParams({
        grant_type: 'refresh_token',
        refresh_token: this.integration.refreshToken!
      })
    });
    
    if (!response.ok) {
      throw new Error('Failed to refresh QuickBooks token');
    }
    
    const tokens = await response.json();
    
    // Update integration with new tokens
    await prisma.accountingIntegration.update({
      where: { id: parseInt(this.integration.id) },
      data: {
        accessToken: tokens.access_token,
        refreshToken: tokens.refresh_token,
        tokenExpiresAt: new Date(Date.now() + tokens.expires_in * 1000)
      }
    });
  }
  
  async getCompanyInfo(): Promise<any> {
    const response = await this.apiClient.get('/v3/companyinfo/' + this.integration.realmId);
    return response.QueryResponse?.CompanyInfo?.[0];
  }
  
  async getAccounts(): Promise<AccountingAccount[]> {
    const response = await this.apiClient.get('/v3/accounts');
    const accounts = response.QueryResponse?.Account || [];
    
    return accounts.map((account: any) => ({
      externalId: account.Id,
      name: account.Name,
      accountType: account.AccountType,
      accountNumber: account.AcctNum,
      description: account.Description,
      parentAccountId: account.ParentRef?.value,
      isActive: account.Active,
      canPost: !account.SubAccount
    }));
  }
  
  async getTaxCodes(): Promise<AccountingTaxCode[]> {
    const response = await this.apiClient.get('/v3/taxcodes');
    const taxCodes = response.QueryResponse?.TaxCode || [];
    
    return taxCodes.map((taxCode: any) => ({
      externalId: taxCode.Id,
      name: taxCode.Name,
      description: taxCode.Description,
      rate: taxCode.TaxRate || 0,
      isActive: taxCode.Active,
      isSalesTax: taxCode.Taxable
    }));
  }
  
  async createInvoice(invoice: Invoice): Promise<SyncResult> {
    const startTime = Date.now();
    
    try {
      // Map invoice data to QuickBooks format
      const mappings = this.integration.mappings.filter(m => m.sourceType === 'INVOICE');
      const qbInvoice = this.mapInvoiceToQuickBooks(invoice, mappings);
      
      const response = await this.apiClient.post('/v3/invoice', qbInvoice);
      const createdInvoice = response.Invoice;
      
      const result: SyncResult = {
        status: 'SUCCESS',
        operation: 'CREATE',
        externalId: createdInvoice.Id,
        duration: Date.now() - startTime,
        data: {
          request: qbInvoice,
          response: createdInvoice
        }
      };
      
      await this.logSync('INVOICE', invoice.id, 'CREATE', result);
      return result;
      
    } catch (error) {
      const result: SyncResult = {
        status: 'FAILED',
        operation: 'CREATE',
        errorMessage: error.message,
        duration: Date.now() - startTime
      };
      
      await this.logSync('INVOICE', invoice.id, 'CREATE', result);
      return result;
    }
  }
  
  private mapInvoiceToQuickBooks(invoice: Invoice, mappings: AccountingMapping[]): any {
    const qbInvoice: any = {
      CustomerRef: {
        value: invoice.client.externalAccountingId || this.createQuickBooksCustomer(invoice.client)
      },
      TxnDate: invoice.issueDate,
      DueDate: invoice.dueDate,
      DocNumber: invoice.number,
      PrivateNote: invoice.notes,
      Line: []
    };
    
    // Map line items
    invoice.items.forEach((item, index) => {
      qbInvoice.Line.push({
        Id: (index + 1).toString(),
        LineNum: index + 1,
        Amount: item.total,
        DetailType: 'SalesItemLineDetail',
        SalesItemLineDetail: {
          ItemRef: {
            value: this.getOrCreateQuickBooksItem(item),
            name: item.description
          },
          Qty: item.quantity,
          UnitPrice: item.unitPrice,
          TaxCodeRef: {
            value: this.mapTaxCode(item.taxRate)
          }
        }
      });
    });
    
    // Add tax line if applicable
    if (invoice.tax > 0) {
      qbInvoice.Line.push({
        Amount: invoice.tax,
        DetailType: 'TaxLineDetail',
        TaxLineDetail: {
          TaxRateRef: {
            value: this.mapTaxCode(invoice.taxRate)
          }
        }
      });
    }
    
    // Apply custom mappings
    const customMapped = this.mapEntityData(invoice, mappings);
    return { ...qbInvoice, ...customMapped };
  }
  
  async createPayment(payment: Payment): Promise<SyncResult> {
    const startTime = Date.now();
    
    try {
      const qbPayment = {
        CustomerRef: {
          value: payment.invoice.client.externalAccountingId
        },
        TotalAmt: payment.amount,
        TxnDate: payment.paidAt,
        PaymentRefNum: payment.reference,
        PrivateNote: payment.notes,
        DepositToAccountRef: {
          value: this.integration.syncSettings.bankAccountId
        },
        Line: [{
          Amount: payment.amount,
          LinkedTxn: [{
            TxnId: payment.invoice.externalAccountingId,
            TxnType: 'Invoice'
          }]
        }]
      };
      
      const response = await this.apiClient.post('/v3/payment', qbPayment);
      const createdPayment = response.Payment;
      
      const result: SyncResult = {
        status: 'SUCCESS',
        operation: 'CREATE',
        externalId: createdPayment.Id,
        duration: Date.now() - startTime
      };
      
      await this.logSync('PAYMENT', payment.id, 'CREATE', result);
      return result;
      
    } catch (error) {
      const result: SyncResult = {
        status: 'FAILED',
        operation: 'CREATE',
        errorMessage: error.message,
        duration: Date.now() - startTime
      };
      
      await this.logSync('PAYMENT', payment.id, 'CREATE', result);
      return result;
    }
  }
  
  async createClient(client: Client): Promise<SyncResult> {
    const startTime = Date.now();
    
    try {
      const qbCustomer = {
        Name: client.name,
        CompanyName: client.company,
        BillAddr: {
          Line1: client.address,
          City: client.city,
          CountrySubDivisionCode: client.state,
          PostalCode: client.zipCode,
          Country: client.country
        },
        PrimaryEmailAddr: {
          Address: client.email
        },
        PrimaryPhone: {
          FreeFormNumber: client.phone
        },
        Notes: client.notes
      };
      
      const response = await this.apiClient.post('/v3/customer', qbCustomer);
      const createdCustomer = response.Customer;
      
      // Update client with external ID
      await this.updateClientExternalId(client.id, createdCustomer.Id);
      
      const result: SyncResult = {
        status: 'SUCCESS',
        operation: 'CREATE',
        externalId: createdCustomer.Id,
        duration: Date.now() - startTime
      };
      
      await this.logSync('CLIENT', client.id, 'CREATE', result);
      return result;
      
    } catch (error) {
      const result: SyncResult = {
        status: 'FAILED',
        operation: 'CREATE',
        errorMessage: error.message,
        duration: Date.now() - startTime
      };
      
      await this.logSync('CLIENT', client.id, 'CREATE', result);
      return result;
    }
  }
  
  async createCreditNote(creditNote: CreditNote): Promise<SyncResult> {
    const startTime = Date.now();
    
    try {
      const qbCreditMemo = {
        CustomerRef: {
          value: creditNote.client.externalAccountingId
        },
        TxnDate: creditNote.issueDate,
        DocNumber: creditNote.number,
        PrivateNote: creditNote.reasonDescription,
        Line: creditNote.items.map((item, index) => ({
          Id: (index + 1).toString(),
          LineNum: index + 1,
          Amount: Math.abs(item.total), // Credit memos use positive amounts
          DetailType: 'SalesItemLineDetail',
          SalesItemLineDetail: {
            ItemRef: {
              name: item.description
            },
            Qty: item.quantity,
            UnitPrice: Math.abs(item.unitPrice)
          }
        }))
      };
      
      const response = await this.apiClient.post('/v3/creditmemo', qbCreditMemo);
      const createdCreditMemo = response.CreditMemo;
      
      const result: SyncResult = {
        status: 'SUCCESS',
        operation: 'CREATE',
        externalId: createdCreditMemo.Id,
        duration: Date.now() - startTime
      };
      
      await this.logSync('CREDIT_NOTE', creditNote.id, 'CREATE', result);
      return result;
      
    } catch (error) {
      const result: SyncResult = {
        status: 'FAILED',
        operation: 'CREATE',
        errorMessage: error.message,
        duration: Date.now() - startTime
      };
      
      await this.logSync('CREDIT_NOTE', creditNote.id, 'CREATE', result);
      return result;
    }
  }
}
```

### 5.3 Xero Service

```typescript
export class XeroService extends BaseAccountingService {
  private xeroClient: XeroAPIClient;
  
  constructor(integration: AccountingIntegration) {
    super(integration);
    this.xeroClient = new XeroAPIClient({
      accessToken: integration.accessToken!,
      tenantId: integration.tenantId!
    });
  }
  
  async authenticate(): Promise<boolean> {
    try {
      const organisations = await this.xeroClient.get('/organisations');
      return organisations.Organisations.length > 0;
    } catch (error) {
      console.error('Xero authentication failed:', error);
      return false;
    }
  }
  
  async refreshToken(): Promise<void> {
    const response = await fetch('https://identity.xero.com/connect/token', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
        'Authorization': `Basic ${Buffer.from(`${this.integration.clientId}:${this.integration.clientSecret}`).toString('base64')}`
      },
      body: new URLSearchParams({
        grant_type: 'refresh_token',
        refresh_token: this.integration.refreshToken!
      })
    });
    
    if (!response.ok) {
      throw new Error('Failed to refresh Xero token');
    }
    
    const tokens = await response.json();
    
    await prisma.accountingIntegration.update({
      where: { id: parseInt(this.integration.id) },
      data: {
        accessToken: tokens.access_token,
        refreshToken: tokens.refresh_token,
        tokenExpiresAt: new Date(Date.now() + tokens.expires_in * 1000)
      }
    });
  }
  
  async getCompanyInfo(): Promise<any> {
    const response = await this.xeroClient.get('/organisations');
    return response.Organisations?.[0];
  }
  
  async createInvoice(invoice: Invoice): Promise<SyncResult> {
    const startTime = Date.now();
    
    try {
      const xeroInvoice = {
        Type: 'ACCREC',
        Contact: {
          ContactID: invoice.client.externalAccountingId || await this.createXeroContact(invoice.client)
        },
        Date: invoice.issueDate,
        DueDate: invoice.dueDate,
        InvoiceNumber: invoice.number,
        Reference: invoice.reference,
        Status: 'AUTHORISED',
        LineItems: invoice.items.map(item => ({
          Description: item.description,
          Quantity: item.quantity,
          UnitAmount: item.unitPrice,
          AccountCode: this.getRevenueAccountCode(),
          TaxType: this.mapTaxType(item.taxRate)
        }))
      };
      
      const response = await this.xeroClient.post('/invoices', { Invoices: [xeroInvoice] });
      const createdInvoice = response.Invoices[0];
      
      const result: SyncResult = {
        status: 'SUCCESS',
        operation: 'CREATE',
        externalId: createdInvoice.InvoiceID,
        duration: Date.now() - startTime
      };
      
      await this.logSync('INVOICE', invoice.id, 'CREATE', result);
      return result;
      
    } catch (error) {
      const result: SyncResult = {
        status: 'FAILED',
        operation: 'CREATE',
        errorMessage: error.message,
        duration: Date.now() - startTime
      };
      
      await this.logSync('INVOICE', invoice.id, 'CREATE', result);
      return result;
    }
  }
  
  // Additional Xero-specific methods...
}
```

### 5.4 Sync Manager Service

```typescript
export class AccountingSyncManager {
  async syncInvoice(
    integrationId: string,
    invoiceId: string,
    force: boolean = false
  ): Promise<SyncResult> {
    const integration = await this.getIntegration(integrationId);
    const invoice = await this.getInvoice(invoiceId);
    
    // Check if already synced
    if (!force && invoice.externalAccountingId) {
      return {
        status: 'SKIPPED',
        operation: 'CREATE',
        externalId: invoice.externalAccountingId,
        duration: 0
      };
    }
    
    // Get appropriate service
    const service = await this.getAccountingService(integration);
    
    // Ensure authentication
    const isAuthenticated = await service.authenticate();
    if (!isAuthenticated) {
      await service.refreshToken();
    }
    
    // Sync invoice
    const result = await service.createInvoice(invoice);
    
    // Update invoice with external ID if successful
    if (result.status === 'SUCCESS' && result.externalId) {
      await prisma.invoice.update({
        where: { id: parseInt(invoiceId) },
        data: { externalAccountingId: result.externalId }
      });
    }
    
    return result;
  }
  
  async bulkSync(
    integrationId: string,
    entityType: EntityType,
    filters: any = {}
  ): Promise<string> {
    const jobId = generateUniqueId();
    
    // Start background job
    this.startBackgroundSync(jobId, integrationId, entityType, filters);
    
    return jobId;
  }
  
  private async startBackgroundSync(
    jobId: string,
    integrationId: string,
    entityType: EntityType,
    filters: any
  ): Promise<void> {
    try {
      const entities = await this.getEntitiesToSync(entityType, filters);
      const integration = await this.getIntegration(integrationId);
      const service = await this.getAccountingService(integration);
      
      for (const entity of entities) {
        try {
          let result: SyncResult;
          
          switch (entityType) {
            case 'INVOICE':
              result = await service.createInvoice(entity);
              break;
            case 'PAYMENT':
              result = await service.createPayment(entity);
              break;
            case 'CLIENT':
              result = await service.createClient(entity);
              break;
            case 'CREDIT_NOTE':
              result = await service.createCreditNote(entity);
              break;
            default:
              continue;
          }
          
          // Update sync job progress
          await this.updateSyncJobProgress(jobId, entity.id, result);
          
        } catch (error) {
          console.error(`Failed to sync ${entityType} ${entity.id}:`, error);
          
          await this.updateSyncJobProgress(jobId, entity.id, {
            status: 'FAILED',
            operation: 'CREATE',
            errorMessage: error.message,
            duration: 0
          });
        }
      }
      
      await this.completeSyncJob(jobId);
      
    } catch (error) {
      console.error(`Bulk sync job ${jobId} failed:`, error);
      await this.failSyncJob(jobId, error.message);
    }
  }
  
  private getAccountingService(
    integration: AccountingIntegration
  ): BaseAccountingService {
    switch (integration.provider) {
      case 'QUICKBOOKS_ONLINE':
        return new QuickBooksOnlineService(integration);
      case 'XERO':
        return new XeroService(integration);
      case 'QUICKBOOKS_DESKTOP':
        return new QuickBooksDesktopService(integration);
      default:
        throw new Error(`Unsupported accounting provider: ${integration.provider}`);
    }
  }
}
```

---

## 6. User Interface Components

### 6.1 Integration Setup Wizard

```typescript
interface IntegrationSetupWizardProps {
  onComplete: (integration: AccountingIntegration) => void;
  onCancel: () => void;
}

export function IntegrationSetupWizard({ onComplete, onCancel }: IntegrationSetupWizardProps) {
  const [step, setStep] = useState(1);
  const [provider, setProvider] = useState<AccountingProvider>('QUICKBOOKS_ONLINE');
  const [authUrl, setAuthUrl] = useState<string>('');
  const [isConnecting, setIsConnecting] = useState(false);

  const handleProviderSelect = (selectedProvider: AccountingProvider) => {
    setProvider(selectedProvider);
    setStep(2);
  };

  const handleConnect = async () => {
    setIsConnecting(true);
    
    try {
      const response = await getOAuthAuthorizationUrl(provider);
      setAuthUrl(response.authorizationUrl);
      
      // Open popup window for OAuth
      const popup = window.open(
        response.authorizationUrl,
        'oauth',
        'width=600,height=600,scrollbars=yes,resizable=yes'
      );
      
      // Listen for popup close or message
      const checkClosed = setInterval(() => {
        if (popup?.closed) {
          clearInterval(checkClosed);
          setIsConnecting(false);
          // Check if integration was created
          checkIntegrationStatus();
        }
      }, 1000);
      
    } catch (error) {
      toast.error('Failed to connect: ' + error.message);
      setIsConnecting(false);
    }
  };

  const renderStep = () => {
    switch (step) {
      case 1:
        return (
          <Box>
            <Typography variant="h6" gutterBottom>
              Choose Your Accounting System
            </Typography>
            
            <Grid container spacing={2}>
              <Grid item xs={12} sm={4}>
                <Card 
                  sx={{ p: 2, cursor: 'pointer', '&:hover': { bgcolor: 'action.hover' } }}
                  onClick={() => handleProviderSelect('QUICKBOOKS_ONLINE')}
                >
                  <Box sx={{ textAlign: 'center' }}>
                    <img src="/images/quickbooks-logo.png" alt="QuickBooks" height="60" />
                    <Typography variant="h6" sx={{ mt: 2 }}>
                      QuickBooks Online
                    </Typography>
                    <Typography variant="body2" color="text.secondary">
                      Cloud-based accounting platform
                    </Typography>
                  </Box>
                </Card>
              </Grid>
              
              <Grid item xs={12} sm={4}>
                <Card 
                  sx={{ p: 2, cursor: 'pointer', '&:hover': { bgcolor: 'action.hover' } }}
                  onClick={() => handleProviderSelect('XERO')}
                >
                  <Box sx={{ textAlign: 'center' }}>
                    <img src="/images/xero-logo.png" alt="Xero" height="60" />
                    <Typography variant="h6" sx={{ mt: 2 }}>
                      Xero
                    </Typography>
                    <Typography variant="body2" color="text.secondary">
                      Online accounting software
                    </Typography>
                  </Box>
                </Card>
              </Grid>
              
              <Grid item xs={12} sm={4}>
                <Card 
                  sx={{ p: 2, cursor: 'pointer', '&:hover': { bgcolor: 'action.hover' } }}
                  onClick={() => handleProviderSelect('QUICKBOOKS_DESKTOP')}
                >
                  <Box sx={{ textAlign: 'center' }}>
                    <img src="/images/quickbooks-desktop-logo.png" alt="QB Desktop" height="60" />
                    <Typography variant="h6" sx={{ mt: 2 }}>
                      QuickBooks Desktop
                    </Typography>
                    <Typography variant="body2" color="text.secondary">
                      Desktop accounting software
                    </Typography>
                  </Box>
                </Card>
              </Grid>
            </Grid>
          </Box>
        );
        
      case 2:
        return (
          <Box>
            <Typography variant="h6" gutterBottom>
              Connect to {provider.replace('_', ' ')}
            </Typography>
            
            <Alert severity="info" sx={{ mb: 3 }}>
              You'll be redirected to {provider.replace('_', ' ')} to authorize the connection.
              Please sign in with your accounting system credentials.
            </Alert>
            
            <Box sx={{ textAlign: 'center' }}>
              <Button
                variant="contained"
                size="large"
                onClick={handleConnect}
                disabled={isConnecting}
                startIcon={isConnecting ? <CircularProgress size={20} /> : <Link />}
              >
                {isConnecting ? 'Connecting...' : `Connect to ${provider.replace('_', ' ')}`}
              </Button>
            </Box>
          </Box>
        );
        
      case 3:
        return <IntegrationConfiguration provider={provider} onComplete={onComplete} />;
        
      default:
        return null;
    }
  };

  return (
    <Dialog open maxWidth="md" fullWidth>
      <DialogTitle>
        Setup Accounting Integration
      </DialogTitle>
      
      <DialogContent>
        {/* Step Indicator */}
        <Stepper activeStep={step - 1} sx={{ mb: 4 }}>
          <Step>
            <StepLabel>Choose Provider</StepLabel>
          </Step>
          <Step>
            <StepLabel>Connect Account</StepLabel>
          </Step>
          <Step>
            <StepLabel>Configure Settings</StepLabel>
          </Step>
        </Stepper>
        
        {renderStep()}
      </DialogContent>
      
      <DialogActions>
        <Button onClick={onCancel}>
          Cancel
        </Button>
        {step > 1 && (
          <Button onClick={() => setStep(step - 1)}>
            Back
          </Button>
        )}
      </DialogActions>
    </Dialog>
  );
}
```

### 6.2 Sync Dashboard

```typescript
export function AccountingSyncDashboard() {
  const [selectedIntegration, setSelectedIntegration] = useState<string>('');
  const [syncLogs, setSyncLogs] = useState<AccountingSyncLog[]>([]);
  const [syncStats, setSyncStats] = useState<any>(null);

  const { data: integrations } = useQuery({
    queryKey: ['accounting-integrations'],
    queryFn: () => getAccountingIntegrations()
  });

  const { data: logs } = useQuery({
    queryKey: ['sync-logs', selectedIntegration],
    queryFn: () => getSyncLogs(selectedIntegration),
    enabled: !!selectedIntegration
  });

  const { data: stats } = useQuery({
    queryKey: ['sync-stats', selectedIntegration],
    queryFn: () => getSyncStats(selectedIntegration),
    enabled: !!selectedIntegration
  });

  const handleManualSync = async (entityType: EntityType) => {
    try {
      const result = await triggerManualSync(selectedIntegration, { entityTypes: [entityType] });
      toast.success(`${entityType} sync started`);
      
      // Refresh logs
      setTimeout(() => {
        queryClient.invalidateQueries(['sync-logs', selectedIntegration]);
      }, 2000);
      
    } catch (error) {
      toast.error('Sync failed: ' + error.message);
    }
  };

  return (
    <Container maxWidth="xl">
      <Typography variant="h4" gutterBottom>
        Accounting Integration Dashboard
      </Typography>

      {/* Integration Selector */}
      <Card sx={{ p: 3, mb: 3 }}>
        <FormControl fullWidth>
          <InputLabel>Select Integration</InputLabel>
          <Select
            value={selectedIntegration}
            onChange={(e) => setSelectedIntegration(e.target.value)}
          >
            {integrations?.map(integration => (
              <MenuItem key={integration.id} value={integration.id}>
                <Box sx={{ display: 'flex', alignItems: 'center', gap: 2 }}>
                  <Chip 
                    label={integration.provider.replace('_', ' ')} 
                    size="small" 
                    color={integration.isHealthy ? 'success' : 'error'}
                  />
                  {integration.lastSyncAt && (
                    <Typography variant="caption" color="text.secondary">
                      Last sync: {format(new Date(integration.lastSyncAt), 'MMM d, h:mm a')}
                    </Typography>
                  )}
                </Box>
              </MenuItem>
            ))}
          </Select>
        </FormControl>
      </Card>

      {selectedIntegration && (
        <>
          {/* Sync Statistics */}
          {stats && (
            <Grid container spacing={3} sx={{ mb: 3 }}>
              <Grid item xs={12} sm={6} md={3}>
                <MetricCard
                  title="Total Syncs"
                  value={stats.totalSyncs}
                  icon={<Sync />}
                  color="primary"
                />
              </Grid>
              <Grid item xs={12} sm={6} md={3}>
                <MetricCard
                  title="Success Rate"
                  value={`${((stats.successfulSyncs / stats.totalSyncs) * 100).toFixed(1)}%`}
                  icon={<CheckCircle />}
                  color="success"
                />
              </Grid>
              <Grid item xs={12} sm={6} md={3}>
                <MetricCard
                  title="Failed Syncs"
                  value={stats.failedSyncs}
                  icon={<Error />}
                  color="error"
                />
              </Grid>
              <Grid item xs={12} sm={6} md={3}>
                <MetricCard
                  title="Avg Duration"
                  value={`${(stats.averageDuration / 1000).toFixed(1)}s`}
                  icon={<Timer />}
                  color="info"
                />
              </Grid>
            </Grid>
          )}

          {/* Manual Sync Actions */}
          <Card sx={{ p: 3, mb: 3 }}>
            <Typography variant="h6" gutterBottom>
              Manual Sync
            </Typography>
            
            <Grid container spacing={2}>
              <Grid item>
                <Button
                  variant="outlined"
                  startIcon={<Receipt />}
                  onClick={() => handleManualSync('INVOICE')}
                >
                  Sync Invoices
                </Button>
              </Grid>
              <Grid item>
                <Button
                  variant="outlined"
                  startIcon={<Payment />}
                  onClick={() => handleManualSync('PAYMENT')}
                >
                  Sync Payments
                </Button>
              </Grid>
              <Grid item>
                <Button
                  variant="outlined"
                  startIcon={<People />}
                  onClick={() => handleManualSync('CLIENT')}
                >
                  Sync Clients
                </Button>
              </Grid>
              <Grid item>
                <Button
                  variant="outlined"
                  startIcon={<CreditCard />}
                  onClick={() => handleManualSync('CREDIT_NOTE')}
                >
                  Sync Credit Notes
                </Button>
              </Grid>
            </Grid>
          </Card>

          {/* Sync Logs */}
          <Card>
            <CardHeader title="Recent Sync Activity" />
            <CardContent>
              <DataGrid
                rows={logs || []}
                columns={[
                  { field: 'startedAt', headerName: 'Time', width: 150, valueFormatter: ({ value }) => format(new Date(value), 'MMM d, h:mm a') },
                  { field: 'entityType', headerName: 'Type', width: 100 },
                  { field: 'operation', headerName: 'Operation', width: 100 },
                  { 
                    field: 'status', 
                    headerName: 'Status', 
                    width: 120,
                    renderCell: ({ value }) => (
                      <Chip 
                        label={value} 
                        size="small"
                        color={value === 'SUCCESS' ? 'success' : value === 'FAILED' ? 'error' : 'default'}
                      />
                    )
                  },
                  { field: 'duration', headerName: 'Duration', width: 100, valueFormatter: ({ value }) => `${(value / 1000).toFixed(1)}s` },
                  { field: 'externalId', headerName: 'External ID', width: 150 },
                  { field: 'errorMessage', headerName: 'Error', width: 200, renderCell: ({ value }) => value ? <Tooltip title={value}><Error color="error" /></Tooltip> : null }
                ]}
                pageSize={25}
                autoHeight
                disableSelectionOnClick
              />
            </CardContent>
          </Card>
        </>
      )}
    </Container>
  );
}
```

---

## 7. Implementation Plan

### Phase 1: Core Infrastructure (Week 1-2)
- [ ] Database schema migration for accounting integration tables
- [ ] OAuth authentication flow for QuickBooks Online and Xero
- [ ] Base accounting service architecture and interfaces
- [ ] Basic sync logging and error handling

### Phase 2: QuickBooks Online Integration (Week 2-3)
- [ ] QuickBooks Online service implementation
- [ ] Invoice, payment, and client sync functionality
- [ ] Chart of accounts and tax code synchronization
- [ ] Mapping configuration and management

### Phase 3: Xero Integration (Week 3-4)
- [ ] Xero service implementation
- [ ] Xero-specific data mapping and formatting
- [ ] Credit note and adjustment synchronization
- [ ] Xero webhook handling for real-time updates

### Phase 4: User Interface (Week 4-5)
- [ ] Integration setup wizard with OAuth flow
- [ ] Sync dashboard and monitoring interface
- [ ] Mapping configuration UI
- [ ] Sync logs and error reporting

### Phase 5: Advanced Features (Week 5-6)
- [ ] Bulk synchronization and batch processing
- [ ] Conflict resolution and data validation
- [ ] Scheduled sync jobs and automation
- [ ] Performance optimization and caching

### Phase 6: Testing & Deployment (Week 6)
- [ ] End-to-end integration testing
- [ ] Error handling and edge case testing
- [ ] Security audit of OAuth and token handling
- [ ] Production deployment and monitoring

---

## 8. Success Criteria

### Functional Requirements
- [ ] Successfully authenticate with QuickBooks Online and Xero
- [ ] Automatically export issued invoices to accounting system
- [ ] Sync payments and update invoice status in accounting system
- [ ] Maintain consistent client/customer records between systems
- [ ] Handle credit notes and adjustments correctly
- [ ] Provide comprehensive sync logging and error reporting

### Technical Requirements
- [ ] OAuth token refresh and re-authentication handling
- [ ] Sync processing time < 5 seconds per document
- [ ] 99% success rate for document synchronization
- [ ] Secure handling of authentication credentials
- [ ] Comprehensive error handling and retry mechanisms
- [ ] Support for bulk operations and large data sets

### Business Requirements
- [ ] Eliminate manual data entry between systems
- [ ] Reduce accounting reconciliation time by 80%
- [ ] Ensure accurate financial reporting and compliance
- [ ] Support multiple accounting system providers
- [ ] Provide real-time sync status and error notifications
- [ ] Enable easy setup and configuration for non-technical users

---

**File Version:** 1.0 – Accounting System Integration Specification  
**Last Updated:** October 12, 2025