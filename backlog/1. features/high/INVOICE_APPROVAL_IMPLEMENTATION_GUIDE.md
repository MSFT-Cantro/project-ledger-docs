# Invoice Approval Workflow - Implementation Guide

**Date:** October 20, 2025  
**Status:** Ready to Implement  
**Estimated Time:** 5 days  
**Branch:** `feature/invoice-approval-workflow`

---

## 📋 Overview

This guide provides step-by-step instructions to implement the invoice approval workflow in the client portal, mirroring the successful quote approval implementation.

**Scope:**
- Backend API endpoints for accept/dispute
- Frontend UI components and dialogs
- Email notification system
- Database schema updates
- Testing and deployment

---

## 🎯 Implementation Checklist

### Phase 1: Database Schema (Day 1 - Morning)

- [ ] Create migration file: `add_invoice_acceptance_table.sql`
- [ ] Add ACCEPTED and DISPUTED to InvoiceStatus enum
- [ ] Create InvoiceAcceptance table
- [ ] Add foreign key relationships
- [ ] Create indexes
- [ ] Update Prisma schema
- [ ] Generate Prisma client
- [ ] Test migration in development

### Phase 2: Backend API (Day 1 - Afternoon & Day 2)

- [ ] Create validation schemas (Zod)
- [ ] Implement POST /api/portal/invoices/:id/accept
- [ ] Implement POST /api/portal/invoices/:id/dispute
- [ ] Add email notification methods
- [ ] Add portal activity logging
- [ ] Write unit tests
- [ ] Test API endpoints with curl/Postman

### Phase 3: Frontend UI (Day 3 & Day 4)

- [ ] Update PortalInvoiceDetailPage.tsx
- [ ] Add state management (dialogs, notes, loading)
- [ ] Create Accept Invoice dialog
- [ ] Create Dispute Invoice dialog
- [ ] Add action buttons (SENT invoices only)
- [ ] Implement API integration
- [ ] Add toast notifications
- [ ] Test UI flows

### Phase 4: Testing & QA (Day 5)

- [ ] Backend unit tests
- [ ] Frontend component tests
- [ ] E2E test scenarios
- [ ] Email delivery testing
- [ ] Security testing
- [ ] Manual QA across browsers

### Phase 5: Documentation & Deployment

- [ ] Update API documentation
- [ ] Update user guides
- [ ] Create deployment checklist
- [ ] Deploy to staging
- [ ] Production deployment

---

## 📁 Files to Create/Modify

### New Files
```
apps/backend/prisma/migrations/[timestamp]_add_invoice_acceptance_table/migration.sql
docs/deployment/INVOICE_APPROVAL_DEPLOYMENT.md
```

### Files to Modify
```
apps/backend/prisma/schema.prisma
apps/backend/src/routes/portal/data.ts
apps/backend/src/lib/emailService.ts
apps/frontend/src/pages/portal/PortalInvoiceDetailPage.tsx
apps/frontend/src/api/portal.ts (optional - for API client)
```

---

## 🚀 Step-by-Step Implementation

### Step 1: Create Database Migration

**File:** `apps/backend/prisma/migrations/[timestamp]_add_invoice_acceptance_table/migration.sql`

```sql
-- Migration: Add Invoice Acceptance System
-- Date: 2025-10-20

-- Add new status values to InvoiceStatus enum
ALTER TYPE "InvoiceStatus" ADD VALUE IF NOT EXISTS 'ACCEPTED';
ALTER TYPE "InvoiceStatus" ADD VALUE IF NOT EXISTS 'DISPUTED';

-- Create InvoiceAcceptance table
CREATE TABLE "InvoiceAcceptance" (
  "id" SERIAL PRIMARY KEY,
  "invoiceId" INTEGER NOT NULL,
  "clientId" INTEGER NOT NULL,
  "action" TEXT NOT NULL CHECK ("action" IN ('ACCEPTED', 'DISPUTED')),
  "reason" TEXT,
  "notes" TEXT,
  "ipAddress" TEXT,
  "userAgent" TEXT,
  "createdAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT "InvoiceAcceptance_invoiceId_fkey" 
    FOREIGN KEY ("invoiceId") REFERENCES "Invoice"("id") ON DELETE CASCADE,
  CONSTRAINT "InvoiceAcceptance_clientId_fkey" 
    FOREIGN KEY ("clientId") REFERENCES "Client"("id") ON DELETE CASCADE
);

-- Create indexes for performance
CREATE INDEX "InvoiceAcceptance_invoiceId_idx" ON "InvoiceAcceptance"("invoiceId");
CREATE INDEX "InvoiceAcceptance_clientId_idx" ON "InvoiceAcceptance"("clientId");
CREATE INDEX "InvoiceAcceptance_createdAt_idx" ON "InvoiceAcceptance"("createdAt");
CREATE INDEX "InvoiceAcceptance_action_idx" ON "InvoiceAcceptance"("action");

-- Add comment to table
COMMENT ON TABLE "InvoiceAcceptance" IS 'Tracks client acceptance or disputes of invoices with audit trail';
```

**Commands to run:**
```bash
# Create migration directory
mkdir -p apps/backend/prisma/migrations/$(date +%Y%m%d%H%M%S)_add_invoice_acceptance_table

# Create migration.sql file
# Copy the SQL above into the file

# Apply migration
cd apps/backend
npx prisma migrate dev --name add_invoice_acceptance_table

# Verify migration
npx prisma migrate status
```

---

### Step 2: Update Prisma Schema

**File:** `apps/backend/prisma/schema.prisma`

Add to the existing Invoice model:
```prisma
model Invoice {
  // ... existing fields
  acceptances     InvoiceAcceptance[]
}

// Add new model
model InvoiceAcceptance {
  id          Int      @id @default(autoincrement())
  invoiceId   Int
  clientId    Int
  action      String   // "ACCEPTED" or "DISPUTED"
  reason      String?  // Dispute reason category
  notes       String?  // Client feedback
  ipAddress   String?
  userAgent   String?
  createdAt   DateTime @default(now())
  
  invoice     Invoice  @relation(fields: [invoiceId], references: [id], onDelete: Cascade)
  client      Client   @relation(fields: [clientId], references: [id], onDelete: Cascade)
  
  @@index([invoiceId])
  @@index([clientId])
  @@index([createdAt])
  @@index([action])
}

// Update enum (if not already updated by migration)
enum InvoiceStatus {
  DRAFT
  SENT
  ACCEPTED    // Add this
  DISPUTED    // Add this
  PAID
  OVERDUE
  VOID
}
```

**Commands to run:**
```bash
# Generate Prisma client
cd apps/backend
npx prisma generate

# Verify schema
npx prisma validate
```

---

### Step 3: Add Backend API Endpoints

**File:** `apps/backend/src/routes/portal/data.ts`

Add these validation schemas near the top (after imports):
```typescript
// Add to existing validation schemas
const invoiceAcceptSchema = z.object({
  notes: z.string().optional(),
});

const invoiceDisputeSchema = z.object({
  notes: z.string().min(10, 'Please provide detailed feedback (at least 10 characters)'),
  reason: z.enum([
    'INCORRECT_AMOUNT',
    'INCORRECT_ITEMS',
    'WORK_NOT_COMPLETED',
    'QUALITY_ISSUES',
    'BILLING_ERROR',
    'DUPLICATE_CHARGE',
    'OTHER'
  ]),
});
```

Add these endpoints after the invoice detail endpoint:
```typescript
// POST /api/portal/invoices/:id/accept - Accept an invoice
router.post('/invoices/:id/accept', async (req: any, res: Response) => {
  try {
    const validationResult = invoiceAcceptSchema.safeParse(req.body);
    if (!validationResult.success) {
      return res.status(400).json({ 
        error: 'Invalid request data', 
        details: validationResult.error 
      });
    }

    const clientId = req.client.clientId;
    const invoiceId = parseInt(req.params.id);
    const { notes } = validationResult.data;
    const ipAddress = req.ip || req.socket.remoteAddress;
    const userAgent = req.get('user-agent');

    if (isNaN(invoiceId)) {
      return res.status(400).json({ error: 'Invalid invoice ID' });
    }

    // Verify invoice exists and belongs to client
    const invoice = await prisma.invoice.findFirst({
      where: { id: invoiceId, clientId },
      select: { 
        id: true, 
        status: true, 
        number: true,
        total: true,
      },
    });

    if (!invoice) {
      return res.status(404).json({ error: 'Invoice not found' });
    }

    // Validate invoice can be accepted (only SENT invoices)
    if (invoice.status !== 'SENT') {
      return res.status(400).json({ 
        error: `Invoice cannot be accepted. Current status: ${invoice.status}. Only SENT invoices can be accepted.` 
      });
    }

    // Check for duplicate acceptance
    const existingAcceptance = await prisma.invoiceAcceptance.findFirst({
      where: { invoiceId, action: 'ACCEPTED' },
    });

    if (existingAcceptance) {
      return res.status(409).json({ 
        error: 'Invoice has already been accepted' 
      });
    }

    // Create acceptance record and update status in transaction
    const result = await prisma.$transaction(async (tx) => {
      const acceptance = await tx.invoiceAcceptance.create({
        data: {
          invoiceId,
          clientId,
          action: 'ACCEPTED',
          notes,
          ipAddress,
          userAgent,
        },
      });

      const updatedInvoice = await tx.invoice.update({
        where: { id: invoiceId },
        data: { status: 'ACCEPTED' },
        include: {
          client: { select: { name: true, email: true } },
          project: { select: { name: true } },
        },
      });

      return { acceptance, invoice: updatedInvoice };
    });

    // Log portal activity
    await logPortalActivity(
      clientId,
      'ACCEPT_INVOICE',
      'INVOICE',
      invoiceId,
      ipAddress,
      userAgent
    );

    // Send email notification to admins
    try {
      await emailService.sendInvoiceAcceptanceNotification({
        invoiceNumber: result.invoice.number,
        clientName: result.invoice.client.name,
        clientEmail: result.invoice.client.email,
        projectName: result.invoice.project?.name,
        invoiceTotal: result.invoice.total,
        notes: notes || undefined,
        acceptedAt: new Date(),
      });
    } catch (emailError) {
      console.error('Failed to send acceptance notification:', emailError);
      // Don't fail the request if email fails
    }

    console.log(`Invoice ${invoice.number} accepted by client ${clientId}`);

    res.json({ 
      success: true, 
      message: 'Invoice accepted successfully',
      invoice: result.invoice,
      acceptance: result.acceptance 
    });
  } catch (error) {
    console.error('Invoice acceptance error:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
});

// POST /api/portal/invoices/:id/dispute - Dispute an invoice
router.post('/invoices/:id/dispute', async (req: any, res: Response) => {
  try {
    const validationResult = invoiceDisputeSchema.safeParse(req.body);
    if (!validationResult.success) {
      return res.status(400).json({ 
        error: 'Invalid request data', 
        details: validationResult.error 
      });
    }

    const clientId = req.client.clientId;
    const invoiceId = parseInt(req.params.id);
    const { notes, reason } = validationResult.data;
    const ipAddress = req.ip || req.socket.remoteAddress;
    const userAgent = req.get('user-agent');

    if (isNaN(invoiceId)) {
      return res.status(400).json({ error: 'Invalid invoice ID' });
    }

    // Verify invoice exists and belongs to client
    const invoice = await prisma.invoice.findFirst({
      where: { id: invoiceId, clientId },
      select: { 
        id: true, 
        status: true, 
        number: true,
        total: true,
      },
    });

    if (!invoice) {
      return res.status(404).json({ error: 'Invoice not found' });
    }

    // Validate invoice can be disputed (only SENT invoices)
    if (invoice.status !== 'SENT') {
      return res.status(400).json({ 
        error: `Invoice cannot be disputed. Current status: ${invoice.status}. Only SENT invoices can be disputed.` 
      });
    }

    // Create dispute record and update status in transaction
    const result = await prisma.$transaction(async (tx) => {
      const dispute = await tx.invoiceAcceptance.create({
        data: {
          invoiceId,
          clientId,
          action: 'DISPUTED',
          reason,
          notes,
          ipAddress,
          userAgent,
        },
      });

      const updatedInvoice = await tx.invoice.update({
        where: { id: invoiceId },
        data: { status: 'DISPUTED' },
        include: {
          client: { select: { name: true, email: true } },
          project: { select: { name: true } },
        },
      });

      return { dispute, invoice: updatedInvoice };
    });

    // Log portal activity
    await logPortalActivity(
      clientId,
      'DISPUTE_INVOICE',
      'INVOICE',
      invoiceId,
      ipAddress,
      userAgent
    );

    // Send email notification to admins
    try {
      await emailService.sendInvoiceDisputeNotification({
        invoiceNumber: result.invoice.number,
        clientName: result.invoice.client.name,
        clientEmail: result.invoice.client.email,
        projectName: result.invoice.project?.name,
        invoiceTotal: result.invoice.total,
        reason,
        notes,
        disputedAt: new Date(),
      });
    } catch (emailError) {
      console.error('Failed to send dispute notification:', emailError);
      // Don't fail the request if email fails
    }

    console.log(`Invoice ${invoice.number} disputed by client ${clientId} - Reason: ${reason}`);

    res.json({ 
      success: true, 
      message: 'Invoice dispute submitted successfully',
      invoice: result.invoice,
      dispute: result.dispute 
    });
  } catch (error) {
    console.error('Invoice dispute error:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
});

// GET /api/portal/invoices/:id/history - Get invoice acceptance history
router.get('/invoices/:id/history', async (req: any, res: Response) => {
  try {
    const clientId = req.client.clientId;
    const invoiceId = parseInt(req.params.id);
    const ipAddress = req.ip || req.socket.remoteAddress;
    const userAgent = req.get('user-agent');

    if (isNaN(invoiceId)) {
      return res.status(400).json({ error: 'Invalid invoice ID' });
    }

    // Verify invoice belongs to client
    const invoice = await prisma.invoice.findFirst({
      where: { id: invoiceId, clientId },
      select: { id: true, number: true, status: true },
    });

    if (!invoice) {
      return res.status(404).json({ error: 'Invoice not found' });
    }

    // Get acceptance history
    const acceptanceHistory = await prisma.invoiceAcceptance.findMany({
      where: { invoiceId },
      orderBy: { createdAt: 'desc' },
      select: {
        id: true,
        action: true,
        reason: true,
        notes: true,
        createdAt: true,
        ipAddress: true,
      },
    });

    // Log activity
    await logPortalActivity(
      clientId,
      'VIEW_INVOICE_HISTORY',
      'INVOICE',
      invoiceId,
      ipAddress,
      userAgent
    );

    res.json({
      invoice,
      acceptanceHistory,
    });
  } catch (error) {
    console.error('Invoice history error:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
});
```

---

### Step 4: Add Email Notification Methods

**File:** `apps/backend/src/lib/emailService.ts`

Add these methods to the EmailService class:

```typescript
/**
 * Send invoice acceptance notification to admins
 */
async sendInvoiceAcceptanceNotification(data: {
  invoiceNumber: string;
  clientName: string;
  clientEmail: string;
  projectName?: string;
  invoiceTotal: number;
  notes?: string;
  acceptedAt: Date;
}): Promise<void> {
  const subject = `Invoice ${data.invoiceNumber} Accepted by ${data.clientName}`;
  
  const html = `
    <div style="font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto;">
      <div style="background-color: #4CAF50; color: white; padding: 20px; border-radius: 8px 8px 0 0;">
        <h1 style="margin: 0; font-size: 24px;">✅ Invoice Accepted</h1>
      </div>
      
      <div style="background-color: #f5f5f5; padding: 20px; border-radius: 0 0 8px 8px;">
        <p style="font-size: 16px; color: #333; margin-top: 0;">
          Good news! An invoice has been accepted by your client.
        </p>
        
        <h2 style="color: #333; margin-top: 20px;">Invoice Details</h2>
        <table style="width: 100%; border-collapse: collapse;">
          <tr>
            <td style="padding: 8px; border-bottom: 1px solid #ddd; font-weight: bold;">Invoice Number:</td>
            <td style="padding: 8px; border-bottom: 1px solid #ddd;">${data.invoiceNumber}</td>
          </tr>
          <tr>
            <td style="padding: 8px; border-bottom: 1px solid #ddd; font-weight: bold;">Client:</td>
            <td style="padding: 8px; border-bottom: 1px solid #ddd;">${data.clientName} (${data.clientEmail})</td>
          </tr>
          ${data.projectName ? `
          <tr>
            <td style="padding: 8px; border-bottom: 1px solid #ddd; font-weight: bold;">Project:</td>
            <td style="padding: 8px; border-bottom: 1px solid #ddd;">${data.projectName}</td>
          </tr>
          ` : ''}
          <tr>
            <td style="padding: 8px; border-bottom: 1px solid #ddd; font-weight: bold;">Total Amount:</td>
            <td style="padding: 8px; border-bottom: 1px solid #ddd; font-weight: bold; color: #4CAF50;">$${data.invoiceTotal.toFixed(2)}</td>
          </tr>
          <tr>
            <td style="padding: 8px; border-bottom: 1px solid #ddd; font-weight: bold;">Accepted:</td>
            <td style="padding: 8px; border-bottom: 1px solid #ddd;">${data.acceptedAt.toLocaleString()}</td>
          </tr>
        </table>
        
        ${data.notes ? `
        <h3 style="color: #333; margin-top: 20px;">Client Notes</h3>
        <div style="background-color: white; padding: 15px; border-radius: 4px; border-left: 4px solid #4CAF50;">
          ${data.notes}
        </div>
        ` : ''}
        
        <div style="margin-top: 20px; padding: 15px; background-color: #e8f5e8; border-radius: 4px;">
          <p style="margin: 0; color: #2e7d32;">
            <strong>Next Steps:</strong> The client has confirmed the invoice. Proceed with payment processing or project delivery.
          </p>
        </div>
      </div>
    </div>
  `;

  const text = `Invoice ${data.invoiceNumber} has been accepted by ${data.clientName}.
  
Invoice Details:
- Number: ${data.invoiceNumber}
- Client: ${data.clientName} (${data.clientEmail})
${data.projectName ? `- Project: ${data.projectName}\n` : ''}- Total: $${data.invoiceTotal.toFixed(2)}
- Accepted: ${data.acceptedAt.toLocaleString()}

${data.notes ? `Client Notes: ${data.notes}\n\n` : ''}Next Steps: The client has confirmed the invoice. Proceed with payment processing.`;

  const adminEmail = process.env.ADMIN_EMAIL || 'admin@projectledger.ca';
  
  await this.sendEmail({
    to: adminEmail,
    subject,
    html,
    text,
  });
}

/**
 * Send invoice dispute notification to admins
 */
async sendInvoiceDisputeNotification(data: {
  invoiceNumber: string;
  clientName: string;
  clientEmail: string;
  projectName?: string;
  invoiceTotal: number;
  reason: string;
  notes: string;
  disputedAt: Date;
}): Promise<void> {
  const subject = `⚠️ Invoice ${data.invoiceNumber} Disputed by ${data.clientName}`;
  
  const reasonLabels: Record<string, string> = {
    'INCORRECT_AMOUNT': 'Incorrect Amount',
    'INCORRECT_ITEMS': 'Incorrect Items/Services',
    'WORK_NOT_COMPLETED': 'Work Not Completed',
    'QUALITY_ISSUES': 'Quality Issues',
    'BILLING_ERROR': 'Billing Error',
    'DUPLICATE_CHARGE': 'Duplicate Charge',
    'OTHER': 'Other Issue',
  };
  
  const html = `
    <div style="font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto;">
      <div style="background-color: #FF9800; color: white; padding: 20px; border-radius: 8px 8px 0 0;">
        <h1 style="margin: 0; font-size: 24px;">⚠️ Invoice Disputed</h1>
      </div>
      
      <div style="background-color: #f5f5f5; padding: 20px; border-radius: 0 0 8px 8px;">
        <p style="font-size: 16px; color: #333; margin-top: 0;">
          <strong>Action Required:</strong> A client has raised a dispute about an invoice.
        </p>
        
        <h2 style="color: #333; margin-top: 20px;">Invoice Details</h2>
        <table style="width: 100%; border-collapse: collapse;">
          <tr>
            <td style="padding: 8px; border-bottom: 1px solid #ddd; font-weight: bold;">Invoice Number:</td>
            <td style="padding: 8px; border-bottom: 1px solid #ddd;">${data.invoiceNumber}</td>
          </tr>
          <tr>
            <td style="padding: 8px; border-bottom: 1px solid #ddd; font-weight: bold;">Client:</td>
            <td style="padding: 8px; border-bottom: 1px solid #ddd;">${data.clientName} (${data.clientEmail})</td>
          </tr>
          ${data.projectName ? `
          <tr>
            <td style="padding: 8px; border-bottom: 1px solid #ddd; font-weight: bold;">Project:</td>
            <td style="padding: 8px; border-bottom: 1px solid #ddd;">${data.projectName}</td>
          </tr>
          ` : ''}
          <tr>
            <td style="padding: 8px; border-bottom: 1px solid #ddd; font-weight: bold;">Total Amount:</td>
            <td style="padding: 8px; border-bottom: 1px solid #ddd; font-weight: bold; color: #FF9800;">$${data.invoiceTotal.toFixed(2)}</td>
          </tr>
          <tr>
            <td style="padding: 8px; border-bottom: 1px solid #ddd; font-weight: bold;">Disputed:</td>
            <td style="padding: 8px; border-bottom: 1px solid #ddd;">${data.disputedAt.toLocaleString()}</td>
          </tr>
          <tr style="background-color: #fff3e0;">
            <td style="padding: 8px; border-bottom: 1px solid #ddd; font-weight: bold;">Dispute Reason:</td>
            <td style="padding: 8px; border-bottom: 1px solid #ddd; color: #F57C00; font-weight: bold;">${reasonLabels[data.reason] || data.reason}</td>
          </tr>
        </table>
        
        <h3 style="color: #333; margin-top: 20px;">Client Feedback</h3>
        <div style="background-color: white; padding: 15px; border-radius: 4px; border-left: 4px solid #FF9800;">
          ${data.notes}
        </div>
        
        <div style="margin-top: 20px; padding: 15px; background-color: #fff3e0; border-radius: 4px; border-left: 3px solid #F57C00;">
          <p style="margin: 0; color: #e65100;">
            <strong>⚠️ Action Required:</strong>
          </p>
          <ul style="margin: 10px 0 0 0; padding-left: 20px; color: #e65100;">
            <li>Contact ${data.clientName} to discuss the dispute</li>
            <li>Review the invoice for accuracy</li>
            <li>Issue a corrected invoice if necessary</li>
            <li>Document resolution in the system</li>
          </ul>
        </div>
        
        <div style="margin-top: 15px; padding: 10px; background-color: #e3f2fd; border-radius: 4px;">
          <p style="margin: 0; color: #1565c0; font-size: 14px;">
            <strong>Tip:</strong> Quick resolution of disputes improves client satisfaction and payment timelines.
          </p>
        </div>
      </div>
    </div>
  `;

  const text = `⚠️ INVOICE DISPUTED - Action Required

Invoice ${data.invoiceNumber} has been disputed by ${data.clientName}.
  
Invoice Details:
- Number: ${data.invoiceNumber}
- Client: ${data.clientName} (${data.clientEmail})
${data.projectName ? `- Project: ${data.projectName}\n` : ''}- Total: $${data.invoiceTotal.toFixed(2)}
- Disputed: ${data.disputedAt.toLocaleString()}
- Reason: ${reasonLabels[data.reason] || data.reason}

Client Feedback:
${data.notes}

Action Required:
1. Contact ${data.clientName} to discuss the dispute
2. Review the invoice for accuracy
3. Issue a corrected invoice if necessary
4. Document resolution in the system

Tip: Quick resolution improves client satisfaction and payment timelines.`;

  const adminEmail = process.env.ADMIN_EMAIL || 'admin@projectledger.ca';
  
  await this.sendEmail({
    to: adminEmail,
    subject,
    html,
    text,
  });
}
```

---

### Step 5: Update Frontend Invoice Detail Page

**File:** `apps/frontend/src/pages/portal/PortalInvoiceDetailPage.tsx`

This is the largest change. I'll create a complete implementation guide in the next section.

---

## 📝 Next Steps

1. **Review this implementation guide**
2. **Create a new branch:** `feature/invoice-approval-workflow`
3. **Follow each step in order**
4. **Test thoroughly at each phase**
5. **Deploy to staging before production**

---

## 🧪 Testing Checklist

### Backend Tests
- [ ] Accept invoice with valid data succeeds
- [ ] Dispute invoice with valid data succeeds
- [ ] Cannot accept/dispute non-SENT invoices
- [ ] Cannot accept already accepted invoices
- [ ] Cannot dispute without notes/reason
- [ ] Email notifications sent correctly
- [ ] Portal activity logged correctly
- [ ] Cross-client access prevented

### Frontend Tests
- [ ] Accept button only shows for SENT invoices
- [ ] Dispute button only shows for SENT invoices
- [ ] Dialogs open/close correctly
- [ ] Form validation works
- [ ] API errors handled gracefully
- [ ] Success messages displayed
- [ ] Invoice status updates immediately
- [ ] History dialog shows all actions

---

**Ready to Begin Implementation!**

Start with Phase 1 (Database Schema) and work through each phase systematically.
