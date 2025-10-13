# Credit Note System Specification

**Date:** October 12, 2025  
**Version:** 1.0  
**Status:** Not Implemented  

---

## 1. Overview

The Credit Note System handles negative adjustments to invoices and provides formal credit documentation for refunds, overpayments, returns, and billing corrections. It maintains proper accounting relationships and ensures compliance with financial reporting standards.

---

## 2. Business Requirements

### 2.1 Core Functionality
- **Invoice Credits**: Issue credits against specific invoices for corrections
- **Standalone Credits**: Issue credits not tied to specific invoices
- **Automatic Generation**: Auto-generate credits from negative change orders
- **Credit Application**: Apply credits to outstanding invoices or customer accounts
- **Refund Processing**: Process refunds for credits exceeding customer balances
- **Tax Handling**: Proper tax treatment for credited amounts

### 2.2 Accounting Requirements
- **Double-Entry Accounting**: Maintain proper debit/credit relationships
- **Account Reconciliation**: Credit notes affect customer account balances
- **Tax Compliance**: Handle tax implications of credits and refunds
- **Financial Reporting**: Credits included in financial statements
- **Audit Trail**: Complete history of all credit transactions
- **Reference Integrity**: Maintain relationships between credits and source documents

### 2.3 Business Rules
- Credit notes must have unique sequential numbering (CN-YYYY-XXX)
- Credits cannot exceed the original invoice amount (unless approved)
- Credit notes require approval workflow for amounts over threshold
- Applied credits automatically update customer account balances
- Credit notes are immutable once issued
- All credits must specify reason and supporting documentation

---

## 3. Data Model

### 3.1 Database Schema

```prisma
model CreditNote {
  id                  Int                   @id @default(autoincrement())
  number              String                @unique // CN-YYYY-XXX format
  status              CreditNoteStatus
  creditType          CreditNoteType
  
  // Source relationships
  invoiceId           Int?                  // Credit against specific invoice
  changeOrderId       Int?                  // Generated from change order
  originalAmount      Float?                // Original invoice amount (if applicable)
  
  // Financial details
  subtotal            Float
  taxAmount           Float                 @default(0.00)
  total               Float
  appliedAmount       Float                 @default(0.00) // Amount applied to invoices
  remainingBalance    Float                 // Available credit balance
  
  // Credit details
  reason              CreditReason
  reasonDescription   String?               // Additional details
  supportingDocs      Json?                 // Supporting document references
  
  // Client and project
  clientId            Int
  projectId           Int?
  accountId           Int                   // For multi-tenant support
  
  // Approval workflow
  requiresApproval    Boolean               @default(false)
  approvedAt          DateTime?
  approvedBy          Int?                  // User who approved
  approvalNotes       String?
  
  // Refund handling
  refundRequested     Boolean               @default(false)
  refundAmount        Float?
  refundMethod        RefundMethod?
  refundReference     String?               // External refund reference
  refundProcessedAt   DateTime?
  refundProcessedBy   Int?
  
  // Metadata
  issueDate           DateTime              @default(now())
  notes               String?
  createdAt           DateTime              @default(now())
  updatedAt           DateTime              @updatedAt
  createdBy           Int
  
  // Relations
  invoice             Invoice?              @relation(fields: [invoiceId], references: [id])
  changeOrder         ChangeOrder?          @relation(fields: [changeOrderId], references: [id])
  client              Client                @relation(fields: [clientId], references: [id])
  project             Project?              @relation(fields: [projectId], references: [id])
  account             Account               @relation(fields: [accountId], references: [id])
  createdByUser       User                  @relation("CreditNoteCreatedBy", fields: [createdBy], references: [id])
  approvedByUser      User?                 @relation("CreditNoteApprovedBy", fields: [approvedBy], references: [id])
  refundProcessedByUser User?               @relation("CreditNoteRefundProcessedBy", fields: [refundProcessedBy], references: [id])
  
  items               CreditNoteItem[]
  applications        CreditApplication[]
  history             CreditNoteHistory[]
  
  @@index([clientId])
  @@index([invoiceId])
  @@index([status])
  @@index([accountId])
  @@index([issueDate])
}

model CreditNoteItem {
  id              Int           @id @default(autoincrement())
  creditNoteId    Int
  
  // Item details
  description     String
  quantity        Float
  unitPrice       Float         // Negative for credits
  total           Float         // Negative for credits
  
  // Source tracking
  sourceType      String?       // INVOICE_ITEM, CHANGE_ORDER_ITEM, MANUAL
  sourceId        Int?          // Reference to source item
  
  // Tax information
  taxable         Boolean       @default(true)
  taxRate         Float?
  taxAmount       Float         @default(0.00)
  
  // Grouping
  groupName       String?
  groupType       String?
  groupOrder      Int?
  sortOrder       Int
  
  createdAt       DateTime      @default(now())
  updatedAt       DateTime      @updatedAt
  
  // Relations
  creditNote      CreditNote    @relation(fields: [creditNoteId], references: [id], onDelete: Cascade)
  
  @@index([creditNoteId])
  @@index([sourceType, sourceId])
}

model CreditApplication {
  id              Int           @id @default(autoincrement())
  creditNoteId    Int
  invoiceId       Int           // Invoice credit is applied to
  
  // Application details
  appliedAmount   Float         // Amount of credit applied
  appliedAt       DateTime      @default(now())
  appliedBy       Int           // User who applied the credit
  
  // Metadata
  reference       String?       // Application reference number
  notes           String?
  
  // Relations
  creditNote      CreditNote    @relation(fields: [creditNoteId], references: [id])
  invoice         Invoice       @relation(fields: [invoiceId], references: [id])
  appliedByUser   User          @relation(fields: [appliedBy], references: [id])
  
  @@unique([creditNoteId, invoiceId]) // One application per credit-invoice pair
  @@index([creditNoteId])
  @@index([invoiceId])
  @@index([appliedAt])
}

model CreditNoteHistory {
  id              Int           @id @default(autoincrement())
  creditNoteId    Int
  
  // Change details
  action          CreditAction  // CREATED, APPROVED, APPLIED, REFUNDED, etc.
  previousStatus  String?
  newStatus       String?
  changes         Json          // Detailed change information
  
  // Context
  performedBy     Int?
  performedAt     DateTime      @default(now())
  ipAddress       String?
  userAgent       String?
  notes           String?
  
  // Relations
  creditNote      CreditNote    @relation(fields: [creditNoteId], references: [id], onDelete: Cascade)
  performedByUser User?         @relation(fields: [performedBy], references: [id])
  
  @@index([creditNoteId])
  @@index([performedAt])
  @@index([action])
}

model CreditNoteSettings {
  id                    Int               @id @default(autoincrement())
  accountId             Int               @unique
  
  // Numbering
  numberPrefix          String            @default("CN")
  nextNumber            Int               @default(1)
  numberFormat          String            @default("CN-YYYY-XXX")
  
  // Approval workflow
  approvalRequired      Boolean           @default(true)
  approvalThreshold     Float             @default(1000.00) // Amount requiring approval
  autoApproveRefunds    Boolean           @default(false)
  
  // Default settings
  defaultReason         CreditReason      @default(BILLING_ERROR)
  requireReasonDescription Boolean        @default(true)
  requireSupportingDocs Boolean           @default(false)
  
  // Refund settings
  allowRefunds          Boolean           @default(true)
  defaultRefundMethod   RefundMethod      @default(ORIGINAL_PAYMENT)
  refundProcessingDays  Int               @default(5)
  
  // Notification settings
  notifyOnCreation      Boolean           @default(true)
  notifyOnApproval      Boolean           @default(true)
  notifyOnApplication   Boolean           @default(false)
  
  createdAt             DateTime          @default(now())
  updatedAt             DateTime          @updatedAt
  
  // Relations
  account               Account           @relation(fields: [accountId], references: [id], onDelete: Cascade)
  
  @@index([accountId])
}

enum CreditNoteStatus {
  DRAFT
  PENDING_APPROVAL
  APPROVED
  ISSUED
  PARTIALLY_APPLIED
  FULLY_APPLIED
  REFUNDED
  CANCELLED
}

enum CreditNoteType {
  INVOICE_CREDIT      // Credit against specific invoice
  STANDALONE_CREDIT   // General credit to customer account
  CHANGE_ORDER_CREDIT // Generated from negative change order
  REFUND_CREDIT      // Credit for refund processing
}

enum CreditReason {
  BILLING_ERROR
  OVERPAYMENT
  PRODUCT_RETURN
  SERVICE_CANCELLATION
  PRICING_ADJUSTMENT
  GOODWILL_CREDIT
  DUPLICATE_CHARGE
  CHANGE_ORDER
  OTHER
}

enum RefundMethod {
  ORIGINAL_PAYMENT     // Refund to original payment method
  BANK_TRANSFER       // ACH/wire transfer
  CHECK               // Physical check
  STORE_CREDIT        // Keep as account credit
  OTHER
}

enum CreditAction {
  CREATED
  SUBMITTED_FOR_APPROVAL
  APPROVED
  REJECTED
  ISSUED
  APPLIED_TO_INVOICE
  REFUND_REQUESTED
  REFUND_PROCESSED
  CANCELLED
  MODIFIED
}
```

### 3.2 TypeScript Interfaces

```typescript
export interface CreditNote {
  id: string;
  number: string;
  status: CreditNoteStatus;
  creditType: CreditNoteType;
  
  // Source relationships
  invoiceId?: string;
  changeOrderId?: string;
  originalAmount?: number;
  
  // Financial
  subtotal: number;
  taxAmount: number;
  total: number;
  appliedAmount: number;
  remainingBalance: number;
  
  // Details
  reason: CreditReason;
  reasonDescription?: string;
  supportingDocs?: any;
  
  // Client and project
  clientId: string;
  projectId?: string;
  accountId: string;
  
  // Approval
  requiresApproval: boolean;
  approvedAt?: string;
  approvedBy?: string;
  approvalNotes?: string;
  
  // Refund
  refundRequested: boolean;
  refundAmount?: number;
  refundMethod?: RefundMethod;
  refundReference?: string;
  refundProcessedAt?: string;
  refundProcessedBy?: string;
  
  // Metadata
  issueDate: string;
  notes?: string;
  createdAt: string;
  updatedAt: string;
  createdBy: string;
  
  // Relations
  items: CreditNoteItem[];
  applications: CreditApplication[];
  history: CreditNoteHistory[];
  invoice?: Invoice;
  changeOrder?: ChangeOrder;
  client: Client;
  project?: Project;
}

export interface CreditNoteItem {
  id?: string;
  description: string;
  quantity: number;
  unitPrice: number; // Negative for credits
  total: number;     // Negative for credits
  
  sourceType?: string;
  sourceId?: string;
  
  taxable: boolean;
  taxRate?: number;
  taxAmount: number;
  
  groupName?: string;
  groupType?: string;
  groupOrder?: number;
  sortOrder: number;
}

export interface CreditApplication {
  id: string;
  creditNoteId: string;
  invoiceId: string;
  appliedAmount: number;
  appliedAt: string;
  appliedBy: string;
  reference?: string;
  notes?: string;
  
  invoice: Invoice;
}

export interface CreateCreditNoteInput {
  creditType: CreditNoteType;
  invoiceId?: string;
  changeOrderId?: string;
  clientId: string;
  projectId?: string;
  
  reason: CreditReason;
  reasonDescription?: string;
  supportingDocs?: any;
  
  items: Omit<CreditNoteItem, 'id' | 'total' | 'taxAmount'>[];
  
  notes?: string;
  requiresApproval?: boolean;
}

export interface ApplyCreditInput {
  creditNoteId: string;
  invoiceId: string;
  appliedAmount: number;
  notes?: string;
}

export interface ProcessRefundInput {
  creditNoteId: string;
  refundAmount: number;
  refundMethod: RefundMethod;
  refundReference?: string;
  notes?: string;
}
```

---

## 4. API Endpoints

### 4.1 Credit Note CRUD Operations

```typescript
// Create credit note
POST /api/credit-notes
Body: CreateCreditNoteInput
Response: CreditNote

// Get credit note
GET /api/credit-notes/:id
Response: CreditNote

// Update credit note (draft only)
PATCH /api/credit-notes/:id
Body: Partial<CreateCreditNoteInput>
Response: CreditNote

// Delete credit note (draft only)
DELETE /api/credit-notes/:id
Response: { success: boolean }

// List credit notes
GET /api/credit-notes
Query: {
  clientId?: string,
  invoiceId?: string,
  status?: CreditNoteStatus,
  dateFrom?: string,
  dateTo?: string,
  reason?: CreditReason
}
Response: CreditNote[]
```

### 4.2 Credit Note Workflow

```typescript
// Submit for approval
POST /api/credit-notes/:id/submit-approval
Body: { notes?: string }
Response: CreditNote

// Approve credit note
POST /api/credit-notes/:id/approve
Body: { 
  approvedBy: string,
  approvalNotes?: string 
}
Response: CreditNote

// Reject credit note
POST /api/credit-notes/:id/reject
Body: { 
  rejectedBy: string,
  rejectionReason: string 
}
Response: CreditNote

// Issue credit note (make immutable)
POST /api/credit-notes/:id/issue
Body: { issuedBy: string }
Response: CreditNote
```

### 4.3 Credit Application

```typescript
// Apply credit to invoice
POST /api/credit-notes/:id/apply
Body: ApplyCreditInput
Response: {
  creditNote: CreditNote,
  application: CreditApplication,
  updatedInvoice: Invoice
}

// Remove credit application
DELETE /api/credit-notes/:id/applications/:applicationId
Response: {
  creditNote: CreditNote,
  updatedInvoice: Invoice
}

// Get available invoices for credit application
GET /api/credit-notes/:id/applicable-invoices
Response: Invoice[]
```

### 4.4 Refund Processing

```typescript
// Request refund
POST /api/credit-notes/:id/request-refund
Body: {
  refundAmount: number,
  refundMethod: RefundMethod,
  notes?: string
}
Response: CreditNote

// Process refund
POST /api/credit-notes/:id/process-refund
Body: ProcessRefundInput
Response: {
  creditNote: CreditNote,
  refundTransaction: RefundTransaction
}

// Get refund status
GET /api/credit-notes/:id/refund-status
Response: {
  status: string,
  amount: number,
  method: RefundMethod,
  reference?: string,
  processedAt?: string
}
```

### 4.5 Reporting & Analytics

```typescript
// Credit note summary
GET /api/credit-notes/summary
Query: {
  dateFrom?: string,
  dateTo?: string,
  clientId?: string
}
Response: {
  totalCredits: number,
  totalAmount: number,
  appliedAmount: number,
  refundedAmount: number,
  outstandingCredits: number,
  byReason: Record<CreditReason, number>,
  byStatus: Record<CreditNoteStatus, number>
}

// Generate PDF
GET /api/credit-notes/:id/pdf
Response: Binary PDF

// Export credit notes
GET /api/credit-notes/export
Query: {
  format: 'CSV' | 'XLSX' | 'PDF',
  dateFrom?: string,
  dateTo?: string,
  filters?: any
}
Response: Binary file
```

---

## 5. Business Logic Services

### 5.1 Credit Note Service

```typescript
export class CreditNoteService {
  async createCreditNote(input: CreateCreditNoteInput, createdBy: string): Promise<CreditNote> {
    // Generate credit note number
    const number = await this.generateCreditNoteNumber();
    
    // Calculate totals (negative amounts for credits)
    const { subtotal, taxAmount, total } = await this.calculateCreditTotals(input.items);
    
    // Determine if approval is required
    const settings = await this.getCreditNoteSettings();
    const requiresApproval = input.requiresApproval !== false && 
      (settings.approvalRequired && Math.abs(total) >= settings.approvalThreshold);
    
    const creditNote = await prisma.creditNote.create({
      data: {
        number,
        status: requiresApproval ? 'PENDING_APPROVAL' : 'APPROVED',
        creditType: input.creditType,
        invoiceId: input.invoiceId ? parseInt(input.invoiceId) : null,
        changeOrderId: input.changeOrderId ? parseInt(input.changeOrderId) : null,
        clientId: parseInt(input.clientId),
        projectId: input.projectId ? parseInt(input.projectId) : null,
        accountId: parseInt(createdBy), // Get from context
        subtotal,
        taxAmount,
        total,
        remainingBalance: Math.abs(total), // Available credit
        reason: input.reason,
        reasonDescription: input.reasonDescription,
        supportingDocs: input.supportingDocs,
        requiresApproval,
        notes: input.notes,
        createdBy: parseInt(createdBy),
        items: {
          create: input.items.map((item, index) => ({
            ...item,
            total: item.quantity * item.unitPrice, // Should be negative
            taxAmount: item.taxable ? (item.quantity * item.unitPrice * (item.taxRate || 0)) : 0,
            sortOrder: index
          }))
        }
      },
      include: {
        items: true,
        client: true,
        invoice: true,
        changeOrder: true
      }
    });
    
    // Log creation
    await this.logCreditNoteAction(creditNote.id, 'CREATED', createdBy);
    
    // Send notifications
    if (requiresApproval) {
      await this.notifyApprovalRequired(creditNote);
    }
    
    return creditNote;
  }
  
  async applyCreditToInvoice(
    creditNoteId: string,
    invoiceId: string,
    appliedAmount: number,
    appliedBy: string,
    notes?: string
  ): Promise<{ creditNote: CreditNote; application: CreditApplication; updatedInvoice: Invoice }> {
    const creditNote = await this.getCreditNote(creditNoteId);
    const invoice = await this.getInvoice(invoiceId);
    
    // Validation
    if (creditNote.status !== 'ISSUED') {
      throw new Error('Credit note must be issued before applying');
    }
    
    if (appliedAmount > creditNote.remainingBalance) {
      throw new Error('Applied amount exceeds available credit balance');
    }
    
    if (appliedAmount > invoice.remainingBalance) {
      throw new Error('Applied amount exceeds invoice balance');
    }
    
    return await prisma.$transaction(async (tx) => {
      // Create credit application
      const application = await tx.creditApplication.create({
        data: {
          creditNoteId: parseInt(creditNoteId),
          invoiceId: parseInt(invoiceId),
          appliedAmount,
          appliedBy: parseInt(appliedBy),
          reference: `CA-${Date.now()}`,
          notes
        },
        include: { invoice: true }
      });
      
      // Update credit note
      const updatedCredit = await tx.creditNote.update({
        where: { id: parseInt(creditNoteId) },
        data: {
          appliedAmount: { increment: appliedAmount },
          remainingBalance: { decrement: appliedAmount },
          status: creditNote.remainingBalance - appliedAmount <= 0 ? 'FULLY_APPLIED' : 'PARTIALLY_APPLIED'
        },
        include: {
          items: true,
          applications: { include: { invoice: true } },
          client: true
        }
      });
      
      // Update invoice
      const newInvoiceBalance = invoice.remainingBalance - appliedAmount;
      const updatedInvoice = await tx.invoice.update({
        where: { id: parseInt(invoiceId) },
        data: {
          amountPaid: { increment: appliedAmount },
          remainingBalance: newInvoiceBalance,
          status: newInvoiceBalance <= 0 ? 'PAID' : 'SENT'
        }
      });
      
      // Log actions
      await this.logCreditNoteAction(creditNote.id, 'APPLIED_TO_INVOICE', appliedBy, {
        invoiceId,
        appliedAmount,
        invoiceNumber: invoice.number
      });
      
      return {
        creditNote: updatedCredit,
        application,
        updatedInvoice
      };
    });
  }
  
  async processRefund(
    creditNoteId: string,
    refundAmount: number,
    refundMethod: RefundMethod,
    processedBy: string,
    refundReference?: string,
    notes?: string
  ): Promise<{ creditNote: CreditNote; refundTransaction: any }> {
    const creditNote = await this.getCreditNote(creditNoteId);
    
    if (refundAmount > creditNote.remainingBalance) {
      throw new Error('Refund amount exceeds available credit balance');
    }
    
    return await prisma.$transaction(async (tx) => {
      // Update credit note
      const updatedCredit = await tx.creditNote.update({
        where: { id: parseInt(creditNoteId) },
        data: {
          refundRequested: true,
          refundAmount,
          refundMethod,
          refundReference,
          refundProcessedAt: new Date(),
          refundProcessedBy: parseInt(processedBy),
          remainingBalance: { decrement: refundAmount },
          status: creditNote.remainingBalance - refundAmount <= 0 ? 'REFUNDED' : 'PARTIALLY_APPLIED'
        },
        include: {
          items: true,
          client: true,
          applications: { include: { invoice: true } }
        }
      });
      
      // Create refund transaction (integrate with payment processor)
      const refundTransaction = await this.createRefundTransaction({
        creditNoteId,
        amount: refundAmount,
        method: refundMethod,
        reference: refundReference,
        processedBy
      });
      
      // Log refund
      await this.logCreditNoteAction(creditNote.id, 'REFUND_PROCESSED', processedBy, {
        refundAmount,
        refundMethod,
        refundReference
      });
      
      // Send notification
      await this.notifyRefundProcessed(updatedCredit, refundTransaction);
      
      return {
        creditNote: updatedCredit,
        refundTransaction
      };
    });
  }
  
  private async calculateCreditTotals(items: CreditNoteItem[]) {
    let subtotal = 0;
    let taxAmount = 0;
    
    for (const item of items) {
      const itemTotal = item.quantity * item.unitPrice; // Should be negative
      subtotal += itemTotal;
      
      if (item.taxable && item.taxRate) {
        taxAmount += itemTotal * item.taxRate;
      }
    }
    
    const total = subtotal + taxAmount;
    
    return {
      subtotal: Number(subtotal.toFixed(2)),
      taxAmount: Number(taxAmount.toFixed(2)),
      total: Number(total.toFixed(2))
    };
  }
  
  private async generateCreditNoteNumber(): Promise<string> {
    const settings = await this.getCreditNoteSettings();
    const year = new Date().getFullYear();
    const nextNumber = settings.nextNumber;
    
    // Update counter
    await prisma.creditNoteSettings.update({
      where: { id: settings.id },
      data: { nextNumber: { increment: 1 } }
    });
    
    return settings.numberFormat
      .replace('YYYY', year.toString())
      .replace('XXX', nextNumber.toString().padStart(3, '0'));
  }
}
```

### 5.2 Automated Credit Generation

```typescript
export class AutoCreditService {
  async generateCreditFromChangeOrder(changeOrder: ChangeOrder): Promise<CreditNote | null> {
    // Only generate credits for deductive change orders
    if (changeOrder.changeType !== 'DEDUCTIVE') {
      return null;
    }
    
    const creditItems: CreditNoteItem[] = changeOrder.items
      .filter(item => item.changeType === 'REMOVE' || 
                     (item.changeType === 'MODIFY' && item.priceChange < 0))
      .map(item => ({
        description: item.description,
        quantity: Math.abs(item.quantity),
        unitPrice: -Math.abs(item.unitPrice), // Negative for credit
        total: -Math.abs(item.total),
        sourceType: 'CHANGE_ORDER_ITEM',
        sourceId: item.id,
        taxable: true, // Inherit from original item
        sortOrder: item.sortOrder
      }));
    
    if (creditItems.length === 0) {
      return null;
    }
    
    const creditNote = await this.creditNoteService.createCreditNote({
      creditType: 'CHANGE_ORDER_CREDIT',
      changeOrderId: changeOrder.id.toString(),
      clientId: changeOrder.quote.clientId.toString(),
      projectId: changeOrder.quote.projectId?.toString(),
      reason: 'CHANGE_ORDER',
      reasonDescription: `Credit generated from Change Order ${changeOrder.number}`,
      items: creditItems,
      requiresApproval: false // Auto-approve change order credits
    }, 'SYSTEM');
    
    // Auto-issue the credit
    await this.creditNoteService.issueCredit(creditNote.id, 'SYSTEM');
    
    return creditNote;
  }
  
  async generateCreditFromOverpayment(
    invoice: Invoice,
    overpaymentAmount: number,
    paymentReference: string
  ): Promise<CreditNote> {
    return await this.creditNoteService.createCreditNote({
      creditType: 'STANDALONE_CREDIT',
      invoiceId: invoice.id,
      clientId: invoice.clientId,
      projectId: invoice.projectId,
      reason: 'OVERPAYMENT',
      reasonDescription: `Credit for overpayment on Invoice ${invoice.number}`,
      supportingDocs: { paymentReference },
      items: [{
        description: `Overpayment credit for Invoice ${invoice.number}`,
        quantity: 1,
        unitPrice: -overpaymentAmount, // Negative for credit
        taxable: false,
        sortOrder: 0
      }],
      requiresApproval: overpaymentAmount >= 1000 // Require approval for large amounts
    }, 'SYSTEM');
  }
}
```

---

## 6. User Interface Components

### 6.1 Credit Note Creation Form

```typescript
interface CreditNoteFormProps {
  invoiceId?: string; // Pre-fill if creating credit for specific invoice
  onSubmit: (data: CreateCreditNoteInput) => void;
  onCancel: () => void;
}

export function CreditNoteForm({ invoiceId, onSubmit, onCancel }: CreditNoteFormProps) {
  const [creditType, setCreditType] = useState<CreditNoteType>('INVOICE_CREDIT');
  const [selectedInvoice, setSelectedInvoice] = useState<Invoice | null>(null);
  const [items, setItems] = useState<CreditNoteItem[]>([]);
  const [reason, setReason] = useState<CreditReason>('BILLING_ERROR');
  const [reasonDescription, setReasonDescription] = useState('');

  // Load invoice if specified
  useEffect(() => {
    if (invoiceId) {
      loadInvoice(invoiceId).then(setSelectedInvoice);
    }
  }, [invoiceId]);

  const handleInvoiceSelect = (invoice: Invoice) => {
    setSelectedInvoice(invoice);
    // Pre-populate items from invoice (user can modify)
    const creditItems = invoice.items.map(item => ({
      description: item.description,
      quantity: item.quantity,
      unitPrice: -item.unitPrice, // Negative for credit
      taxable: item.taxable,
      sourceType: 'INVOICE_ITEM',
      sourceId: item.id,
      sortOrder: 0
    }));
    setItems(creditItems);
  };

  const calculateTotals = () => {
    const subtotal = items.reduce((sum, item) => sum + (item.quantity * item.unitPrice), 0);
    const taxAmount = items.reduce((sum, item) => {
      return sum + (item.taxable ? (item.quantity * item.unitPrice * 0.1) : 0); // Simplified tax
    }, 0);
    return { subtotal, taxAmount, total: subtotal + taxAmount };
  };

  const { subtotal, taxAmount, total } = calculateTotals();

  return (
    <Paper sx={{ p: 3 }}>
      <Typography variant="h5" gutterBottom>
        Create Credit Note
      </Typography>

      <Box component="form" onSubmit={(e) => {
        e.preventDefault();
        onSubmit({
          creditType,
          invoiceId: selectedInvoice?.id,
          clientId: selectedInvoice?.clientId || '',
          projectId: selectedInvoice?.projectId,
          reason,
          reasonDescription,
          items: items.map(({ total, taxAmount, ...item }) => item), // Remove calculated fields
        });
      }}>
        
        {/* Credit Type Selection */}
        <FormControl fullWidth sx={{ mb: 3 }}>
          <InputLabel>Credit Type</InputLabel>
          <Select value={creditType} onChange={(e) => setCreditType(e.target.value as CreditNoteType)}>
            <MenuItem value="INVOICE_CREDIT">Credit Against Invoice</MenuItem>
            <MenuItem value="STANDALONE_CREDIT">Standalone Credit</MenuItem>
            <MenuItem value="CHANGE_ORDER_CREDIT">Change Order Credit</MenuItem>
          </Select>
        </FormControl>

        {/* Invoice Selection */}
        {creditType === 'INVOICE_CREDIT' && (
          <Box sx={{ mb: 3 }}>
            <InvoiceSelector
              value={selectedInvoice}
              onChange={handleInvoiceSelect}
              helperText="Select the invoice to credit against"
            />
            
            {selectedInvoice && (
              <Alert severity="info" sx={{ mt: 2 }}>
                <Typography variant="subtitle2">Selected Invoice: {selectedInvoice.number}</Typography>
                <Typography variant="body2">
                  Original Amount: ${selectedInvoice.total.toLocaleString()} | 
                  Remaining: ${selectedInvoice.remainingBalance.toLocaleString()}
                </Typography>
              </Alert>
            )}
          </Box>
        )}

        {/* Credit Reason */}
        <Grid container spacing={2} sx={{ mb: 3 }}>
          <Grid item xs={12} sm={6}>
            <FormControl fullWidth>
              <InputLabel>Reason</InputLabel>
              <Select value={reason} onChange={(e) => setReason(e.target.value as CreditReason)}>
                <MenuItem value="BILLING_ERROR">Billing Error</MenuItem>
                <MenuItem value="OVERPAYMENT">Overpayment</MenuItem>
                <MenuItem value="PRODUCT_RETURN">Product Return</MenuItem>
                <MenuItem value="SERVICE_CANCELLATION">Service Cancellation</MenuItem>
                <MenuItem value="PRICING_ADJUSTMENT">Pricing Adjustment</MenuItem>
                <MenuItem value="GOODWILL_CREDIT">Goodwill Credit</MenuItem>
                <MenuItem value="DUPLICATE_CHARGE">Duplicate Charge</MenuItem>
                <MenuItem value="OTHER">Other</MenuItem>
              </Select>
            </FormControl>
          </Grid>
          <Grid item xs={12} sm={6}>
            <TextField
              fullWidth
              label="Reason Description"
              value={reasonDescription}
              onChange={(e) => setReasonDescription(e.target.value)}
              multiline
              rows={2}
              required
            />
          </Grid>
        </Grid>

        {/* Credit Items */}
        <Typography variant="h6" gutterBottom>
          Credit Items
        </Typography>

        {items.map((item, index) => (
          <Card key={index} sx={{ mb: 2, p: 2 }}>
            <Grid container spacing={2} alignItems="center">
              <Grid item xs={12} sm={4}>
                <TextField
                  fullWidth
                  label="Description"
                  value={item.description}
                  onChange={(e) => updateItem(index, 'description', e.target.value)}
                  required
                />
              </Grid>
              <Grid item xs={6} sm={2}>
                <TextField
                  fullWidth
                  type="number"
                  label="Quantity"
                  value={item.quantity}
                  onChange={(e) => updateItem(index, 'quantity', parseFloat(e.target.value))}
                  inputProps={{ min: 0, step: 0.01 }}
                  required
                />
              </Grid>
              <Grid item xs={6} sm={2}>
                <TextField
                  fullWidth
                  type="number"
                  label="Unit Price"
                  value={Math.abs(item.unitPrice)} // Show as positive, store as negative
                  onChange={(e) => updateItem(index, 'unitPrice', -Math.abs(parseFloat(e.target.value)))}
                  inputProps={{ min: 0, step: 0.01 }}
                  required
                />
              </Grid>
              <Grid item xs={6} sm={2}>
                <Typography variant="body1" sx={{ fontWeight: 'medium' }}>
                  ${Math.abs(item.quantity * item.unitPrice).toFixed(2)}
                </Typography>
              </Grid>
              <Grid item xs={6} sm={2}>
                <FormControlLabel
                  control={
                    <Checkbox
                      checked={item.taxable}
                      onChange={(e) => updateItem(index, 'taxable', e.target.checked)}
                    />
                  }
                  label="Taxable"
                />
              </Grid>
              <Grid item xs={12} sm={1}>
                <IconButton onClick={() => removeItem(index)} color="error">
                  <Delete />
                </IconButton>
              </Grid>
            </Grid>
          </Card>
        ))}

        <Button
          startIcon={<Add />}
          onClick={() => addItem()}
          variant="outlined"
          sx={{ mb: 3 }}
        >
          Add Credit Item
        </Button>

        {/* Totals Summary */}
        <Card sx={{ bgcolor: 'grey.50', p: 2, mb: 3 }}>
          <Typography variant="h6" gutterBottom>
            Credit Summary
          </Typography>
          <Box sx={{ display: 'flex', justifyContent: 'space-between', mb: 1 }}>
            <Typography>Subtotal:</Typography>
            <Typography>${Math.abs(subtotal).toFixed(2)}</Typography>
          </Box>
          <Box sx={{ display: 'flex', justifyContent: 'space-between', mb: 1 }}>
            <Typography>Tax:</Typography>
            <Typography>${Math.abs(taxAmount).toFixed(2)}</Typography>
          </Box>
          <Divider sx={{ my: 1 }} />
          <Box sx={{ display: 'flex', justifyContent: 'space-between' }}>
            <Typography variant="h6">Total Credit:</Typography>
            <Typography variant="h6" color="success.main">
              ${Math.abs(total).toFixed(2)}
            </Typography>
          </Box>
        </Card>

        {/* Form Actions */}
        <Box sx={{ display: 'flex', gap: 2, justifyContent: 'flex-end' }}>
          <Button variant="outlined" onClick={onCancel}>
            Cancel
          </Button>
          <Button 
            type="submit" 
            variant="contained"
            disabled={items.length === 0 || total >= 0}
          >
            Create Credit Note
          </Button>
        </Box>
      </Box>
    </Paper>
  );
}
```

### 6.2 Credit Application Interface

```typescript
interface CreditApplicationProps {
  creditNote: CreditNote;
  onApplicationComplete: (application: CreditApplication) => void;
}

export function CreditApplicationDialog({ creditNote, onApplicationComplete }: CreditApplicationProps) {
  const [selectedInvoice, setSelectedInvoice] = useState<Invoice | null>(null);
  const [appliedAmount, setAppliedAmount] = useState(0);
  const [notes, setNotes] = useState('');

  const { data: applicableInvoices } = useQuery({
    queryKey: ['applicable-invoices', creditNote.id],
    queryFn: () => getApplicableInvoices(creditNote.id)
  });

  const maxApplicationAmount = Math.min(
    creditNote.remainingBalance,
    selectedInvoice?.remainingBalance || 0
  );

  const handleApply = async () => {
    if (!selectedInvoice || appliedAmount <= 0) return;

    try {
      const result = await applyCreditToInvoice({
        creditNoteId: creditNote.id,
        invoiceId: selectedInvoice.id,
        appliedAmount,
        notes
      });
      
      onApplicationComplete(result.application);
      toast.success('Credit applied successfully');
    } catch (error) {
      toast.error('Failed to apply credit: ' + error.message);
    }
  };

  return (
    <Dialog open maxWidth="md" fullWidth>
      <DialogTitle>
        Apply Credit Note {creditNote.number}
      </DialogTitle>
      
      <DialogContent>
        <Typography variant="body1" paragraph>
          Available Credit Balance: <strong>${creditNote.remainingBalance.toLocaleString()}</strong>
        </Typography>

        {/* Invoice Selection */}
        <FormControl fullWidth sx={{ mb: 3 }}>
          <InputLabel>Select Invoice</InputLabel>
          <Select
            value={selectedInvoice?.id || ''}
            onChange={(e) => {
              const invoice = applicableInvoices?.find(inv => inv.id === e.target.value);
              setSelectedInvoice(invoice || null);
              setAppliedAmount(invoice ? Math.min(creditNote.remainingBalance, invoice.remainingBalance) : 0);
            }}
          >
            {applicableInvoices?.map(invoice => (
              <MenuItem key={invoice.id} value={invoice.id}>
                <Box>
                  <Typography variant="body1">
                    {invoice.number} - ${invoice.remainingBalance.toLocaleString()} remaining
                  </Typography>
                  <Typography variant="caption" color="text.secondary">
                    Total: ${invoice.total.toLocaleString()} | Due: {format(new Date(invoice.dueDate), 'MMM d, yyyy')}
                  </Typography>
                </Box>
              </MenuItem>
            ))}
          </Select>
        </FormControl>

        {selectedInvoice && (
          <Box sx={{ mb: 3 }}>
            <Alert severity="info">
              <Typography variant="subtitle2">Selected Invoice: {selectedInvoice.number}</Typography>
              <Typography variant="body2">
                Remaining Balance: ${selectedInvoice.remainingBalance.toLocaleString()}
              </Typography>
            </Alert>

            {/* Amount Input */}
            <TextField
              fullWidth
              type="number"
              label="Amount to Apply"
              value={appliedAmount}
              onChange={(e) => setAppliedAmount(Math.min(maxApplicationAmount, parseFloat(e.target.value) || 0))}
              inputProps={{ 
                min: 0, 
                max: maxApplicationAmount,
                step: 0.01 
              }}
              helperText={`Maximum: $${maxApplicationAmount.toFixed(2)}`}
              sx={{ mt: 2, mb: 2 }}
            />

            {/* Notes */}
            <TextField
              fullWidth
              multiline
              rows={3}
              label="Application Notes (Optional)"
              value={notes}
              onChange={(e) => setNotes(e.target.value)}
            />
          </Box>
        )}
      </DialogContent>
      
      <DialogActions>
        <Button onClick={() => onApplicationComplete(null)}>
          Cancel
        </Button>
        <Button
          variant="contained"
          onClick={handleApply}
          disabled={!selectedInvoice || appliedAmount <= 0 || appliedAmount > maxApplicationAmount}
        >
          Apply Credit (${appliedAmount.toFixed(2)})
        </Button>
      </DialogActions>
    </Dialog>
  );
}
```

### 6.3 Credit Note Dashboard

```typescript
export function CreditNotesDashboard() {
  const [dateRange, setDateRange] = useState({ from: '', to: '' });
  const [statusFilter, setStatusFilter] = useState<CreditNoteStatus | ''>('');

  const { data: summary } = useQuery({
    queryKey: ['credit-notes-summary', dateRange],
    queryFn: () => getCreditNotesSummary(dateRange)
  });

  const { data: creditNotes } = useQuery({
    queryKey: ['credit-notes', dateRange, statusFilter],
    queryFn: () => getCreditNotes({ 
      dateFrom: dateRange.from,
      dateTo: dateRange.to,
      status: statusFilter || undefined
    })
  });

  return (
    <Container maxWidth="xl">
      <Box sx={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center', mb: 4 }}>
        <Typography variant="h4">
          Credit Notes
        </Typography>
        <Button
          variant="contained"
          startIcon={<Add />}
          onClick={() => navigate('/credit-notes/new')}
        >
          Create Credit Note
        </Button>
      </Box>

      {/* Summary Cards */}
      <Grid container spacing={3} sx={{ mb: 4 }}>
        <Grid item xs={12} sm={6} md={3}>
          <MetricCard
            title="Total Credits"
            value={summary?.totalCredits || 0}
            icon={<Receipt />}
            color="primary"
          />
        </Grid>
        <Grid item xs={12} sm={6} md={3}>
          <MetricCard
            title="Total Amount"
            value={`$${(summary?.totalAmount || 0).toLocaleString()}`}
            icon={<AttachMoney />}
            color="success"
          />
        </Grid>
        <Grid item xs={12} sm={6} md={3}>
          <MetricCard
            title="Applied Credits"
            value={`$${(summary?.appliedAmount || 0).toLocaleString()}`}
            icon={<CheckCircle />}
            color="info"
          />
        </Grid>
        <Grid item xs={12} sm={6} md={3}>
          <MetricCard
            title="Outstanding"
            value={`$${(summary?.outstandingCredits || 0).toLocaleString()}`}
            icon={<Schedule />}
            color="warning"
          />
        </Grid>
      </Grid>

      {/* Filters */}
      <Card sx={{ mb: 3 }}>
        <CardContent>
          <Grid container spacing={2} alignItems="center">
            <Grid item xs={12} sm={3}>
              <TextField
                fullWidth
                type="date"
                label="From Date"
                value={dateRange.from}
                onChange={(e) => setDateRange(prev => ({ ...prev, from: e.target.value }))}
                InputLabelProps={{ shrink: true }}
              />
            </Grid>
            <Grid item xs={12} sm={3}>
              <TextField
                fullWidth
                type="date"
                label="To Date"
                value={dateRange.to}
                onChange={(e) => setDateRange(prev => ({ ...prev, to: e.target.value }))}
                InputLabelProps={{ shrink: true }}
              />
            </Grid>
            <Grid item xs={12} sm={3}>
              <FormControl fullWidth>
                <InputLabel>Status</InputLabel>
                <Select
                  value={statusFilter}
                  onChange={(e) => setStatusFilter(e.target.value as CreditNoteStatus)}
                >
                  <MenuItem value="">All Statuses</MenuItem>
                  <MenuItem value="DRAFT">Draft</MenuItem>
                  <MenuItem value="PENDING_APPROVAL">Pending Approval</MenuItem>
                  <MenuItem value="APPROVED">Approved</MenuItem>
                  <MenuItem value="ISSUED">Issued</MenuItem>
                  <MenuItem value="PARTIALLY_APPLIED">Partially Applied</MenuItem>
                  <MenuItem value="FULLY_APPLIED">Fully Applied</MenuItem>
                  <MenuItem value="REFUNDED">Refunded</MenuItem>
                </Select>
              </FormControl>
            </Grid>
            <Grid item xs={12} sm={3}>
              <Button
                variant="outlined"
                startIcon={<Download />}
                onClick={() => exportCreditNotes({ dateRange, status: statusFilter })}
                fullWidth
              >
                Export
              </Button>
            </Grid>
          </Grid>
        </CardContent>
      </Card>

      {/* Credit Notes Table */}
      <CreditNotesTable
        creditNotes={creditNotes || []}
        onView={(creditNote) => navigate(`/credit-notes/${creditNote.id}`)}
        onApply={(creditNote) => setApplyingCredit(creditNote)}
        onRefund={(creditNote) => setRefundingCredit(creditNote)}
      />

      {/* Dialogs */}
      {applyingCredit && (
        <CreditApplicationDialog
          creditNote={applyingCredit}
          onApplicationComplete={() => {
            setApplyingCredit(null);
            // Refresh data
          }}
        />
      )}
    </Container>
  );
}
```

---

## 7. Implementation Plan

### Phase 1: Core Infrastructure (Week 1-2)
- [ ] Database schema migration for credit note tables
- [ ] Basic CRUD operations and API endpoints
- [ ] Credit note number generation system
- [ ] Basic credit note creation and management
- [ ] Credit note settings configuration

### Phase 2: Business Logic (Week 2-3)
- [ ] Credit calculation engine with tax handling
- [ ] Approval workflow implementation
- [ ] Credit application system (apply credits to invoices)
- [ ] Integration with existing invoice system
- [ ] Audit trail and history tracking

### Phase 3: User Interface (Week 3-4)
- [ ] Credit note creation form with item management
- [ ] Credit note dashboard and listing
- [ ] Credit application interface
- [ ] Approval workflow UI for managers
- [ ] Credit note detail view and PDF generation

### Phase 4: Advanced Features (Week 4-5)
- [ ] Refund processing system
- [ ] Automated credit generation from change orders
- [ ] Overpayment credit handling
- [ ] Bulk operations and reporting
- [ ] Email notifications and alerts

### Phase 5: Integration & Testing (Week 5-6)
- [ ] Integration with existing document immutability system
- [ ] Payment processor integration for refunds
- [ ] Accounting system integration (QuickBooks/Xero)
- [ ] End-to-end testing of credit workflows
- [ ] Performance optimization

### Phase 6: Polish & Launch (Week 6)
- [ ] User acceptance testing
- [ ] Documentation and training materials
- [ ] Security audit and compliance verification
- [ ] Production deployment and monitoring
- [ ] User training and rollout

---

## 8. Success Criteria

### Functional Requirements
- [ ] Create credit notes for various scenarios (billing errors, returns, overpayments)
- [ ] Apply credits to outstanding invoices automatically updating balances
- [ ] Process refunds for excess credits through integrated payment systems
- [ ] Generate professional PDF credit notes matching company branding
- [ ] Maintain complete audit trail for all credit transactions
- [ ] Support approval workflows for credits above threshold amounts

### Technical Requirements
- [ ] Credit note creation and processing < 2 seconds
- [ ] 99.9% accuracy in credit calculations and applications
- [ ] Support for high volume credit processing (1000+ credits/month)
- [ ] Integration with existing invoice and payment systems
- [ ] Secure handling of financial data with encryption
- [ ] Comprehensive API for external integrations

### Business Requirements
- [ ] Reduce credit processing time by 75%
- [ ] Eliminate manual credit calculation errors
- [ ] Provide real-time visibility into credit balances
- [ ] Support compliance with accounting standards
- [ ] Enable self-service credit applications
- [ ] Improve customer satisfaction with faster credit resolution

---

**File Version:** 1.0 – Credit Note System Specification  
**Last Updated:** October 12, 2025