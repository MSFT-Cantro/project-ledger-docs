# Quotes - Missing Features Specification

> **Document Type**: Feature Specification  
> **Priority**: High to Low (by section)  
> **Created**: 2025-10-16  
> **Status**: Planning

## Overview

This document outlines features that are completely missing from the Quote system but are specified in the original requirements. Features are organized by priority to help guide implementation planning.

---

## HIGH PRIORITY FEATURES

### 1. Quote-to-Invoice Conversion

**Priority**: 🔴 High  
**Business Value**: Critical for workflow completion  
**Effort**: Medium  

#### Description
Allow users to convert an accepted quote directly into an invoice with one click.

#### Current State
- ❌ No conversion endpoint exists
- ❌ Quote-Invoice linking not implemented
- ❌ Line items not transferable

#### Requirements

##### 1.1 API Endpoint
**File**: `apps/backend/src/routes/quotes.ts`

```typescript
// POST /api/quotes/:id/convert-to-invoice
router.post('/:id/convert-to-invoice', async (req, res) => {
  try {
    const quoteId = parseInt(req.params.id);
    const userId = req.user.id;

    const invoice = await quoteService.convertToInvoice(quoteId, userId, {
      dueDate: req.body.dueDate,
      paymentTerms: req.body.paymentTerms || 'Net 30',
      notes: req.body.notes,
      sendToClient: req.body.sendToClient || false
    });

    res.json(invoice);
  } catch (error) {
    logger.error('Error converting quote to invoice:', error);
    res.status(500).json({ error: error.message });
  }
});
```

##### 1.2 Service Implementation
**File**: `apps/backend/src/services/quoteService.ts`

```typescript
async convertToInvoice(
  quoteId: number,
  userId: number,
  options: {
    dueDate?: Date;
    paymentTerms?: string;
    notes?: string;
    sendToClient?: boolean;
  }
): Promise<Invoice> {
  const quote = await this.getQuote(quoteId);
  
  if (!quote) {
    throw new Error('Quote not found');
  }

  if (quote.status !== QuoteStatus.ACCEPTED) {
    throw new Error('Only accepted quotes can be converted to invoices');
  }

  // Check if already converted
  const existingInvoice = await this.db.invoice.findFirst({
    where: { quoteId }
  });

  if (existingInvoice) {
    throw new Error('Quote has already been converted to an invoice');
  }

  return this.db.$transaction(async (tx) => {
    // Create invoice
    const invoice = await tx.invoice.create({
      data: {
        number: await this.invoiceService.generateInvoiceNumber(),
        quoteId,
        clientId: quote.clientId,
        projectId: quote.projectId,
        status: InvoiceStatus.DRAFT,
        paymentStatus: PaymentStatus.UNPAID,
        issueDate: new Date(),
        dueDate: options.dueDate || new Date(Date.now() + 30 * 24 * 60 * 60 * 1000),
        paymentTerms: options.paymentTerms,
        subtotal: quote.subtotal,
        discounts: quote.discounts || 0,
        fees: quote.fees || 0,
        tax: quote.tax,
        total: quote.total,
        amountPaid: 0,
        balance: quote.total,
        currency: quote.currency,
        notes: options.notes || quote.notes,
        createdById: userId
      }
    });

    // Copy line items
    const items = await tx.quoteItem.findMany({
      where: { quoteId }
    });

    for (const item of items) {
      await tx.invoiceItem.create({
        data: {
          invoiceId: invoice.id,
          description: item.description,
          quantity: item.quantity,
          unitPrice: item.unitPrice,
          total: item.total,
          taxable: item.taxable,
          groupName: item.groupName,
          groupType: item.groupType,
          groupOrder: item.groupOrder
        }
      });
    }

    // Update quote status
    await tx.quote.update({
      where: { id: quoteId },
      data: { status: QuoteStatus.CONVERTED }
    });

    // Log history
    await tx.quoteHistory.create({
      data: {
        quoteId,
        version: quote.version,
        changedById: userId,
        changes: {
          status: QuoteStatus.CONVERTED,
          convertedToInvoice: invoice.id,
          convertedAt: new Date()
        }
      }
    });

    // Send to client if requested
    if (options.sendToClient) {
      await this.invoiceService.sendInvoice(invoice.id, userId);
    }

    return invoice;
  });
}
```

##### 1.3 Frontend UI
**File**: `apps/frontend/src/components/quotes/QuoteDetailPage.tsx`

```typescript
const handleConvertToInvoice = async () => {
  try {
    const response = await axios.post(`/api/quotes/${quoteId}/convert-to-invoice`, {
      dueDate: addDays(new Date(), 30),
      paymentTerms: 'Net 30',
      sendToClient: sendImmediately
    });

    showNotification('Quote converted to invoice successfully', 'success');
    navigate(`/invoices/${response.data.id}`);
  } catch (error) {
    showNotification(error.message, 'error');
  }
};

// Add button to quote actions
{quote.status === 'ACCEPTED' && !quote.convertedToInvoice && (
  <Button
    variant="contained"
    color="primary"
    onClick={handleConvertToInvoice}
    startIcon={<ReceiptIcon />}
  >
    Convert to Invoice
  </Button>
)}
```

#### Acceptance Criteria
- [ ] Only ACCEPTED quotes can be converted
- [ ] Quote cannot be converted twice
- [ ] All line items copied to invoice
- [ ] Quote status updated to CONVERTED
- [ ] Invoice links back to quote
- [ ] History logged on both quote and invoice
- [ ] Optional immediate send to client
- [ ] User redirected to new invoice

---

### 2. Client-Facing Quote View

**Priority**: 🔴 High  
**Business Value**: Essential for client interaction  
**Effort**: High  

#### Description
Secure, tokenized links that allow clients to view, accept, or decline quotes without logging in.

#### Current State
- ❌ No public quote view
- ❌ No token generation
- ❌ No client actions (accept/decline)

#### Requirements

##### 2.1 Add Token Fields to Schema
**File**: `apps/backend/prisma/schema.prisma`

```prisma
model Quote {
  // ... existing fields
  
  // Client access
  clientToken      String?         @unique  // Secure token for client access
  tokenExpiresAt   DateTime?                // Token expiration
  viewedAt         DateTime?                // When client first viewed
  clientAcceptedAt DateTime?                // When client accepted
  clientDeclinedAt DateTime?                // When client declined
  clientNotes      String?                  // Client's notes on decline
}
```

##### 2.2 Token Generation Service
**File**: `apps/backend/src/services/quoteService.ts`

```typescript
import crypto from 'crypto';

async generateClientToken(quoteId: number, userId: number): Promise<string> {
  const quote = await this.getQuote(quoteId);
  
  if (!quote) {
    throw new Error('Quote not found');
  }

  // Generate secure random token
  const token = crypto.randomBytes(32).toString('hex');
  
  // Token expires in 90 days or at quote validUntil, whichever is sooner
  const tokenExpiresAt = new Date(
    Math.min(
      Date.now() + 90 * 24 * 60 * 60 * 1000,
      quote.validUntil.getTime()
    )
  );

  await this.db.quote.update({
    where: { id: quoteId },
    data: {
      clientToken: token,
      tokenExpiresAt
    }
  });

  return token;
}

async getQuoteByToken(token: string): Promise<Quote | null> {
  const quote = await this.db.quote.findUnique({
    where: { clientToken: token },
    include: {
      items: true,
      client: true,
      project: true,
      createdBy: {
        select: {
          firstName: true,
          lastName: true,
          email: true
        }
      }
    }
  });

  if (!quote) {
    return null;
  }

  // Check if token expired
  if (quote.tokenExpiresAt && quote.tokenExpiresAt < new Date()) {
    throw new Error('This quote link has expired');
  }

  // Check if quote expired
  if (quote.validUntil < new Date()) {
    throw new Error('This quote has expired');
  }

  // Mark as viewed if first time
  if (!quote.viewedAt) {
    await this.db.quote.update({
      where: { id: quote.id },
      data: { viewedAt: new Date() }
    });
  }

  return quote;
}

async clientAcceptQuote(token: string, signature?: string): Promise<Quote> {
  const quote = await this.getQuoteByToken(token);
  
  if (!quote) {
    throw new Error('Quote not found');
  }

  if (quote.status !== QuoteStatus.SENT) {
    throw new Error('Quote cannot be accepted in current status');
  }

  return this.db.quote.update({
    where: { id: quote.id },
    data: {
      status: QuoteStatus.ACCEPTED,
      clientAcceptedAt: new Date(),
      signature
    }
  });
}

async clientDeclineQuote(token: string, reason: string): Promise<Quote> {
  const quote = await this.getQuoteByToken(token);
  
  if (!quote) {
    throw new Error('Quote not found');
  }

  if (quote.status !== QuoteStatus.SENT) {
    throw new Error('Quote cannot be declined in current status');
  }

  return this.db.quote.update({
    where: { id: quote.id },
    data: {
      status: QuoteStatus.REJECTED,
      clientDeclinedAt: new Date(),
      clientNotes: reason
    }
  });
}
```

##### 2.3 Public API Routes
**File**: `apps/backend/src/routes/public/quotes.ts`

```typescript
import express from 'express';
const router = express.Router();

// GET /api/public/quotes/:token - View quote
router.get('/:token', async (req, res) => {
  try {
    const quote = await quoteService.getQuoteByToken(req.params.token);
    
    if (!quote) {
      return res.status(404).json({ error: 'Quote not found' });
    }

    res.json(quote);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// POST /api/public/quotes/:token/accept - Accept quote
router.post('/:token/accept', async (req, res) => {
  try {
    const { signature } = req.body;
    const quote = await quoteService.clientAcceptQuote(req.params.token, signature);
    
    // Send notification to quote creator
    await notificationService.sendQuoteAccepted(quote.id);
    
    res.json({ success: true, quote });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// POST /api/public/quotes/:token/decline - Decline quote
router.post('/:token/decline', async (req, res) => {
  try {
    const { reason } = req.body;
    const quote = await quoteService.clientDeclineQuote(req.params.token, reason);
    
    // Send notification to quote creator
    await notificationService.sendQuoteDeclined(quote.id, reason);
    
    res.json({ success: true, quote });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

export default router;
```

##### 2.4 Frontend Public View
**File**: `apps/frontend/src/pages/public/QuoteView.tsx`

```typescript
import React, { useState, useEffect } from 'react';
import { useParams } from 'react-router-dom';
import axios from 'axios';

export const PublicQuoteView: React.FC = () => {
  const { token } = useParams();
  const [quote, setQuote] = useState(null);
  const [loading, setLoading] = useState(true);
  const [accepting, setAccepting] = useState(false);
  const [declining, setDeclining] = useState(false);

  useEffect(() => {
    loadQuote();
  }, [token]);

  const loadQuote = async () => {
    try {
      const response = await axios.get(`/api/public/quotes/${token}`);
      setQuote(response.data);
    } catch (error) {
      // Show error
    } finally {
      setLoading(false);
    }
  };

  const handleAccept = async () => {
    setAccepting(true);
    try {
      await axios.post(`/api/public/quotes/${token}/accept`);
      // Show success message
      loadQuote(); // Refresh
    } catch (error) {
      // Show error
    } finally {
      setAccepting(false);
    }
  };

  const handleDecline = async (reason: string) => {
    setDeclining(true);
    try {
      await axios.post(`/api/public/quotes/${token}/decline`, { reason });
      // Show success message
      loadQuote(); // Refresh
    } catch (error) {
      // Show error
    } finally {
      setDeclining(false);
    }
  };

  if (loading) return <CircularProgress />;

  return (
    <Container maxWidth="lg" sx={{ py: 4 }}>
      {/* Quote header */}
      <Paper sx={{ p: 3, mb: 3 }}>
        <Typography variant="h4">Quote {quote.number}</Typography>
        <Typography color="textSecondary">
          Valid until: {formatDate(quote.validUntil)}
        </Typography>
        <Chip 
          label={quote.status} 
          color={getStatusColor(quote.status)}
          sx={{ mt: 1 }}
        />
      </Paper>

      {/* Client & Company Info */}
      <Grid container spacing={3}>
        <Grid item xs={12} md={6}>
          <Paper sx={{ p: 2 }}>
            <Typography variant="h6">From</Typography>
            <Typography>{quote.createdBy.firstName} {quote.createdBy.lastName}</Typography>
            <Typography color="textSecondary">{quote.createdBy.email}</Typography>
          </Paper>
        </Grid>
        <Grid item xs={12} md={6}>
          <Paper sx={{ p: 2 }}>
            <Typography variant="h6">To</Typography>
            <Typography>{quote.client.name}</Typography>
            <Typography color="textSecondary">{quote.client.email}</Typography>
          </Paper>
        </Grid>
      </Grid>

      {/* Line Items */}
      <Paper sx={{ p: 3, mt: 3 }}>
        <Typography variant="h6" gutterBottom>Items</Typography>
        <TableContainer>
          <Table>
            <TableHead>
              <TableRow>
                <TableCell>Description</TableCell>
                <TableCell align="right">Quantity</TableCell>
                <TableCell align="right">Unit Price</TableCell>
                <TableCell align="right">Total</TableCell>
              </TableRow>
            </TableHead>
            <TableBody>
              {quote.items.map((item, idx) => (
                <TableRow key={idx}>
                  <TableCell>{item.description}</TableCell>
                  <TableCell align="right">{item.quantity}</TableCell>
                  <TableCell align="right">${item.unitPrice.toFixed(2)}</TableCell>
                  <TableCell align="right">${item.total.toFixed(2)}</TableCell>
                </TableRow>
              ))}
            </TableBody>
          </Table>
        </TableContainer>

        {/* Totals */}
        <Box sx={{ mt: 3, display: 'flex', justifyContent: 'flex-end' }}>
          <Box sx={{ minWidth: 300 }}>
            <Box sx={{ display: 'flex', justifyContent: 'space-between', mb: 1 }}>
              <Typography>Subtotal:</Typography>
              <Typography>${quote.subtotal.toFixed(2)}</Typography>
            </Box>
            <Box sx={{ display: 'flex', justifyContent: 'space-between', mb: 1 }}>
              <Typography>Tax:</Typography>
              <Typography>${quote.tax.toFixed(2)}</Typography>
            </Box>
            <Divider sx={{ my: 1 }} />
            <Box sx={{ display: 'flex', justifyContent: 'space-between' }}>
              <Typography variant="h6">Total:</Typography>
              <Typography variant="h6">${quote.total.toFixed(2)}</Typography>
            </Box>
          </Box>
        </Box>
      </Paper>

      {/* Action Buttons */}
      {quote.status === 'SENT' && (
        <Paper sx={{ p: 3, mt: 3 }}>
          <Typography variant="h6" gutterBottom>Your Response</Typography>
          <Box sx={{ display: 'flex', gap: 2 }}>
            <Button
              variant="contained"
              color="success"
              size="large"
              onClick={handleAccept}
              disabled={accepting}
            >
              Accept Quote
            </Button>
            <Button
              variant="outlined"
              color="error"
              size="large"
              onClick={() => setDeclining(true)}
            >
              Decline Quote
            </Button>
          </Box>
        </Paper>
      )}

      {/* Status Messages */}
      {quote.status === 'ACCEPTED' && (
        <Alert severity="success" sx={{ mt: 3 }}>
          You accepted this quote on {formatDate(quote.clientAcceptedAt)}
        </Alert>
      )}

      {quote.status === 'REJECTED' && (
        <Alert severity="info" sx={{ mt: 3 }}>
          You declined this quote on {formatDate(quote.clientDeclinedAt)}
        </Alert>
      )}
    </Container>
  );
};
```

#### Acceptance Criteria
- [ ] Generate secure token for each quote
- [ ] Token expires at quote validUntil or 90 days
- [ ] Client can view quote without login
- [ ] Client can accept quote with optional signature
- [ ] Client can decline quote with reason
- [ ] First view timestamp recorded
- [ ] Quote creator notified of client actions
- [ ] Expired quotes/tokens show appropriate message
- [ ] Mobile-responsive design
- [ ] PDF download available

---

### 3. Email Sending & Notifications

**Priority**: 🔴 High  
**Business Value**: Critical for client communication  
**Effort**: Medium  

#### Description
Send quotes to clients via email with secure links and track delivery.

#### Current State
- ❌ No email integration
- ❌ No send endpoint
- ❌ No email templates

#### Requirements

##### 3.1 Email Service Integration
**File**: `apps/backend/src/services/emailService.ts`

```typescript
import nodemailer from 'nodemailer';
import { Quote } from '@prisma/client';

export class EmailService {
  private transporter: nodemailer.Transporter;

  constructor() {
    this.transporter = nodemailer.createTransport({
      host: process.env.SMTP_HOST,
      port: parseInt(process.env.SMTP_PORT || '587'),
      secure: process.env.SMTP_SECURE === 'true',
      auth: {
        user: process.env.SMTP_USER,
        pass: process.env.SMTP_PASS
      }
    });
  }

  async sendQuoteToClient(quote: Quote, token: string): Promise<void> {
    const viewUrl = `${process.env.APP_URL}/public/quotes/${token}`;
    const pdfUrl = `${process.env.APP_URL}/api/quotes/${quote.id}/pdf?token=${token}`;

    const html = `
      <!DOCTYPE html>
      <html>
      <head>
        <style>
          body { font-family: Arial, sans-serif; line-height: 1.6; color: #333; }
          .container { max-width: 600px; margin: 0 auto; padding: 20px; }
          .header { background: #007bff; color: white; padding: 20px; text-align: center; }
          .content { padding: 20px; background: #f9f9f9; }
          .button { 
            display: inline-block; 
            padding: 12px 24px; 
            background: #007bff; 
            color: white; 
            text-decoration: none; 
            border-radius: 4px;
            margin: 10px 0;
          }
          .footer { padding: 20px; text-align: center; color: #666; font-size: 12px; }
        </style>
      </head>
      <body>
        <div class="container">
          <div class="header">
            <h1>New Quote: ${quote.number}</h1>
          </div>
          <div class="content">
            <p>Hello,</p>
            <p>You have received a new quote.</p>
            
            <table style="width: 100%; margin: 20px 0;">
              <tr>
                <td><strong>Quote Number:</strong></td>
                <td>${quote.number}</td>
              </tr>
              <tr>
                <td><strong>Valid Until:</strong></td>
                <td>${quote.validUntil.toLocaleDateString()}</td>
              </tr>
              <tr>
                <td><strong>Total:</strong></td>
                <td>$${quote.total.toFixed(2)}</td>
              </tr>
            </table>

            <p>
              <a href="${viewUrl}" class="button">View Quote Online</a>
            </p>
            
            <p>
              <a href="${pdfUrl}">Download PDF</a>
            </p>

            ${quote.notes ? `<p><strong>Notes:</strong><br>${quote.notes}</p>` : ''}

            <p>This link will expire on ${quote.validUntil.toLocaleDateString()}.</p>
          </div>
          <div class="footer">
            <p>This is an automated message. Please do not reply to this email.</p>
          </div>
        </div>
      </body>
      </html>
    `;

    await this.transporter.sendMail({
      from: `"${process.env.APP_NAME}" <${process.env.SMTP_FROM}>`,
      to: quote.client.email,
      subject: `New Quote ${quote.number}`,
      html
    });
  }

  async sendQuoteAcceptedNotification(quote: Quote): Promise<void> {
    // Email to quote creator
    const html = `
      <p>Great news! Quote ${quote.number} has been accepted by ${quote.client.name}.</p>
      <p><a href="${process.env.APP_URL}/quotes/${quote.id}">View Quote</a></p>
    `;

    await this.transporter.sendMail({
      from: `"${process.env.APP_NAME}" <${process.env.SMTP_FROM}>`,
      to: quote.createdBy.email,
      subject: `Quote ${quote.number} Accepted!`,
      html
    });
  }

  async sendQuoteDeclinedNotification(quote: Quote, reason: string): Promise<void> {
    const html = `
      <p>Quote ${quote.number} has been declined by ${quote.client.name}.</p>
      ${reason ? `<p><strong>Reason:</strong> ${reason}</p>` : ''}
      <p><a href="${process.env.APP_URL}/quotes/${quote.id}">View Quote</a></p>
    `;

    await this.transporter.sendMail({
      from: `"${process.env.APP_NAME}" <${process.env.SMTP_FROM}>`,
      to: quote.createdBy.email,
      subject: `Quote ${quote.number} Declined`,
      html
    });
  }
}
```

##### 3.2 Send Quote Endpoint
**File**: `apps/backend/src/routes/quotes.ts`

```typescript
// POST /api/quotes/:id/send
router.post('/:id/send', async (req, res) => {
  try {
    const quoteId = parseInt(req.params.id);
    const userId = req.user.id;

    // Generate token if not exists
    let token = await quoteService.getClientToken(quoteId);
    if (!token) {
      token = await quoteService.generateClientToken(quoteId, userId);
    }

    // Send email
    const quote = await quoteService.getQuote(quoteId);
    await emailService.sendQuoteToClient(quote, token);

    // Update quote status
    await quoteService.updateQuote(quoteId, userId, {
      status: QuoteStatus.SENT,
      sentAt: new Date()
    });

    res.json({ success: true, message: 'Quote sent successfully' });
  } catch (error) {
    logger.error('Error sending quote:', error);
    res.status(500).json({ error: error.message });
  }
});
```

#### Acceptance Criteria
- [ ] Email sent with secure link
- [ ] Email includes quote summary
- [ ] PDF attachment or download link
- [ ] Email template is professional and branded
- [ ] Send action updates quote status to SENT
- [ ] Quote creator notified when client accepts/declines
- [ ] Email delivery tracked
- [ ] Bounce handling implemented

---

## MEDIUM PRIORITY FEATURES

### 4. Internal Approval Workflow

**Priority**: 🟡 Medium  
**Business Value**: Process control  
**Effort**: Medium  

#### Description
Multi-stage approval process before quotes can be sent to clients.

#### Requirements

##### 4.1 Schema Updates
```prisma
model Quote {
  // ... existing fields
  
  // Approval workflow
  requiresApproval Boolean         @default(false)
  approvalStatus   ApprovalStatus? @default(PENDING)
  approvedBy       Int?
  approver         User?           @relation("QuoteApprover", fields: [approvedBy], references: [id])
  approvedAt       DateTime?
  approvalNotes    String?
}

enum ApprovalStatus {
  PENDING
  APPROVED
  REJECTED
  NOT_REQUIRED
}
```

##### 4.2 Approval Service Methods
```typescript
async submitForApproval(quoteId: number, userId: number): Promise<Quote> {
  const quote = await this.getQuote(quoteId);
  
  if (quote.status !== QuoteStatus.INTERNAL_REVIEW) {
    throw new Error('Quote must be in internal review to submit for approval');
  }

  // Determine if approval required (e.g., total > $10,000)
  const requiresApproval = quote.total > 10000;

  if (!requiresApproval) {
    return this.updateQuote(quoteId, userId, {
      approvalStatus: ApprovalStatus.NOT_REQUIRED,
      status: QuoteStatus.READY_TO_SEND
    });
  }

  // Get approver (could be manager, role-based, etc.)
  const approver = await this.getApproverForQuote(quote);

  await this.db.quote.update({
    where: { id: quoteId },
    data: {
      requiresApproval: true,
      approvalStatus: ApprovalStatus.PENDING,
      approvedBy: approver.id
    }
  });

  // Notify approver
  await this.notificationService.notifyApprovalRequired(quote, approver);

  return this.getQuote(quoteId);
}

async approveQuote(quoteId: number, userId: number, notes?: string): Promise<Quote> {
  const quote = await this.getQuote(quoteId);
  
  if (quote.approvalStatus !== ApprovalStatus.PENDING) {
    throw new Error('Quote is not pending approval');
  }

  if (quote.approvedBy !== userId) {
    throw new Error('You are not authorized to approve this quote');
  }

  return this.db.quote.update({
    where: { id: quoteId },
    data: {
      approvalStatus: ApprovalStatus.APPROVED,
      approvedAt: new Date(),
      approvalNotes: notes,
      status: QuoteStatus.READY_TO_SEND
    }
  });
}

async rejectQuote(quoteId: number, userId: number, reason: string): Promise<Quote> {
  const quote = await this.getQuote(quoteId);
  
  if (quote.approvalStatus !== ApprovalStatus.PENDING) {
    throw new Error('Quote is not pending approval');
  }

  if (quote.approvedBy !== userId) {
    throw new Error('You are not authorized to reject this quote');
  }

  return this.db.quote.update({
    where: { id: quoteId },
    data: {
      approvalStatus: ApprovalStatus.REJECTED,
      approvalNotes: reason,
      status: QuoteStatus.INTERNAL_REVIEW
    }
  });
}
```

#### Acceptance Criteria
- [ ] Quotes over threshold require approval
- [ ] Approval routing configurable
- [ ] Approver receives notification
- [ ] Approver can approve or reject with notes
- [ ] Quote creator notified of decision
- [ ] Approval history tracked
- [ ] Approval status visible in UI

---

### 5. Quote Expiry Automation

**Priority**: 🟡 Medium  
**Business Value**: Automatic status management  
**Effort**: Low  

#### Description
Automatically mark quotes as expired when validUntil date passes.

#### Requirements

##### 5.1 Cron Job Implementation
**File**: `apps/backend/src/jobs/quoteExpiryJob.ts`

```typescript
import cron from 'node-cron';
import { PrismaClient, QuoteStatus } from '@prisma/client';

export class QuoteExpiryJob {
  private db: PrismaClient;

  constructor(db: PrismaClient) {
    this.db = db;
  }

  start() {
    // Run daily at 2 AM
    cron.schedule('0 2 * * *', async () => {
      await this.expireQuotes();
    });
  }

  async expireQuotes(): Promise<void> {
    const now = new Date();

    const expiredQuotes = await this.db.quote.findMany({
      where: {
        validUntil: { lt: now },
        status: { in: [QuoteStatus.SENT, QuoteStatus.INTERNAL_REVIEW, QuoteStatus.DRAFT] }
      }
    });

    for (const quote of expiredQuotes) {
      await this.db.quote.update({
        where: { id: quote.id },
        data: { status: QuoteStatus.EXPIRED }
      });

      // Log history
      await this.db.quoteHistory.create({
        data: {
          quoteId: quote.id,
          version: quote.version,
          changes: {
            status: QuoteStatus.EXPIRED,
            expiredAt: now,
            autoExpired: true
          }
        }
      });

      // Notify creator
      await this.notificationService.notifyQuoteExpired(quote);
    }

    console.log(`Expired ${expiredQuotes.length} quotes`);
  }
}
```

#### Acceptance Criteria
- [ ] Daily job runs automatically
- [ ] Quotes past validUntil marked EXPIRED
- [ ] History logged with auto-expiry flag
- [ ] Quote creator notified
- [ ] Job logs execution metrics

---

### 6. Optional Line Items

**Priority**: 🟡 Medium  
**Business Value**: Flexible pricing  
**Effort**: Medium  

#### Description
Mark certain line items as optional, allowing clients to toggle them on/off.

#### Requirements

##### 6.1 Schema Update
```prisma
model QuoteItem {
  // ... existing fields
  itemType         ItemType        @default(STANDARD)
  isOptional       Boolean         @default(false)
  clientSelected   Boolean?        // null until client makes choice
}

enum ItemType {
  STANDARD
  OPTIONAL
  DISCOUNT
  FEE
  TAX
}
```

##### 6.2 Service Logic
```typescript
interface QuoteItemInput {
  description: string;
  quantity: number;
  unitPrice: number;
  itemType?: 'STANDARD' | 'OPTIONAL' | 'DISCOUNT' | 'FEE';
  isOptional?: boolean;
}

async calculateQuoteTotals(items: QuoteItemInput[], selectedOptionals: number[]): Promise<QuoteTotals> {
  const includedItems = items.filter((item, idx) => {
    if (item.itemType === 'OPTIONAL') {
      return selectedOptionals.includes(idx);
    }
    return true;
  });

  // Calculate totals only for included items
  // ... rest of calculation logic
}
```

##### 6.3 Frontend UI
```typescript
// In public quote view
{quote.items.map((item, idx) => (
  <TableRow key={idx}>
    <TableCell>
      {item.isOptional && (
        <Checkbox
          checked={selectedOptionals.includes(idx)}
          onChange={(e) => handleToggleOptional(idx, e.target.checked)}
        />
      )}
      {item.description}
      {item.isOptional && <Chip label="Optional" size="small" />}
    </TableCell>
    <TableCell align="right">{item.quantity}</TableCell>
    <TableCell align="right">${item.unitPrice.toFixed(2)}</TableCell>
    <TableCell align="right">
      {selectedOptionals.includes(idx) || !item.isOptional 
        ? `$${item.total.toFixed(2)}` 
        : '-'}
    </TableCell>
  </TableRow>
))}
```

#### Acceptance Criteria
- [ ] Items can be marked as optional
- [ ] Optional items shown with checkbox in client view
- [ ] Total recalculates when optional items toggled
- [ ] Client selections saved when accepting quote
- [ ] Invoice includes only selected items
- [ ] Optional items clearly distinguished in PDF

---

## LOW PRIORITY FEATURES

### 7. Estimate Type Distinction

**Priority**: 🟢 Low  
**Business Value**: Professional presentation  
**Effort**: Low  

#### Description
Distinguish between Estimates (ballpark, non-binding) and Quotes (binding, fixed price).

#### Requirements

```prisma
model Quote {
  // ... existing fields
  documentType     DocumentType    @default(QUOTE)
  isBinding        Boolean         @default(true)
}

enum DocumentType {
  ESTIMATE
  QUOTE
}
```

Update numbering:
- Estimates: `EST-2025-0001-v1`
- Quotes: `Q-2025-0001-v1`

---

### 8. Client Signature Capture

**Priority**: 🟢 Low  
**Business Value**: Legal compliance  
**Effort**: Medium  

#### Requirements

```prisma
model Quote {
  // ... existing fields
  signature        String?         // Base64 encoded signature image
  signedAt         DateTime?
  signerName       String?
  signerTitle      String?
  ipAddress        String?
}
```

Frontend: Use signature pad library like `react-signature-canvas`

---

### 9. Quote Attachments

**Priority**: 🟢 Low  
**Business Value**: Additional documentation  
**Effort**: Medium  

#### Requirements

```prisma
model QuoteAttachment {
  id               Int             @id @default(autoincrement())
  quoteId          Int
  quote            Quote           @relation(fields: [quoteId], references: [id])
  filename         String
  originalName     String
  mimeType         String
  size             Int
  url              String
  uploadedAt       DateTime        @default(now())
  uploadedBy       Int
  uploader         User            @relation(fields: [uploadedBy], references: [id])
}
```

File storage: AWS S3 or Azure Blob Storage

---

### 10. Reminder System

**Priority**: 🟢 Low  
**Business Value**: Follow-up automation  
**Effort**: Medium  

#### Description
Send reminders to clients about pending quotes.

#### Requirements

Schedule reminders:
- 7 days before expiry
- 3 days before expiry
- On expiry day

```typescript
async sendQuoteReminder(quoteId: number): Promise<void> {
  const quote = await this.getQuote(quoteId);
  
  if (quote.status !== QuoteStatus.SENT) {
    return;
  }

  const daysUntilExpiry = Math.ceil(
    (quote.validUntil.getTime() - Date.now()) / (1000 * 60 * 60 * 24)
  );

  await emailService.sendQuoteReminder(quote, daysUntilExpiry);
}
```

---

### 11. Contingency Percentage

**Priority**: 🟢 Low  
**Business Value**: Risk management  
**Effort**: Low  

#### Requirements

```prisma
model Quote {
  // ... existing fields
  contingency      Float?          @default(0)    // Percentage (0-100)
  contingencyAmount Float?         @default(0)    // Calculated amount
}
```

Apply after subtotal, before tax.

---

### 12. External Integrations

**Priority**: 🟢 Low  
**Business Value**: Workflow automation  
**Effort**: High  

#### Integrations Needed

- **CRM**: Sync quotes to HubSpot, Salesforce
- **Accounting**: Export to QuickBooks, Xero
- **Slack**: Notifications for quote events
- **Zapier**: General automation

Each integration requires:
- OAuth setup
- API client
- Webhook handling
- Data mapping
- Error handling

---

## Testing Requirements

### Unit Tests
```typescript
describe('Quote Client Portal', () => {
  it('should generate unique tokens', async () => {});
  it('should expire tokens correctly', async () => {});
  it('should allow client to accept quote', async () => {});
  it('should allow client to decline quote', async () => {});
  it('should prevent actions on expired quotes', async () => {});
});

describe('Quote to Invoice Conversion', () => {
  it('should convert accepted quote to invoice', async () => {});
  it('should prevent duplicate conversions', async () => {});
  it('should copy all line items', async () => {});
  it('should link quote and invoice', async () => {});
});

describe('Email Service', () => {
  it('should send quote email', async () => {});
  it('should send acceptance notification', async () => {});
  it('should send decline notification', async () => {});
  it('should handle email failures gracefully', async () => {});
});
```

### Integration Tests
```typescript
describe('Quote Workflow E2E', () => {
  it('should complete full quote lifecycle', async () => {
    // Create -> Send -> Client View -> Accept -> Convert to Invoice
  });
});
```

---

## Implementation Roadmap

### Phase 1: Core Client Features (High Priority)
**Timeline**: 4 weeks

1. Client-facing quote view (Week 1-2)
2. Quote-to-invoice conversion (Week 2)
3. Email sending & notifications (Week 3-4)

### Phase 2: Workflow Enhancements (Medium Priority)
**Timeline**: 3 weeks

4. Approval workflow (Week 1-2)
5. Quote expiry automation (Week 2)
6. Optional line items (Week 3)

### Phase 3: Advanced Features (Low Priority)
**Timeline**: 4 weeks

7. Estimate vs Quote distinction (Week 1)
8. Client signature capture (Week 1-2)
9. Quote attachments (Week 2-3)
10. Reminder system (Week 3-4)
11. Contingency percentage (Week 4)

### Phase 4: Integrations (Low Priority)
**Timeline**: 6 weeks

12. External integrations (CRM, Accounting, etc.)

---

## Success Metrics

- [ ] 100% of high priority features implemented
- [ ] Client portal adoption > 80%
- [ ] Quote-to-invoice conversion used for > 90% of accepted quotes
- [ ] Email delivery rate > 95%
- [ ] Zero security vulnerabilities in client portal
- [ ] Mobile usability score > 85%
- [ ] Page load time < 2 seconds
- [ ] Unit test coverage > 85%
- [ ] Integration test coverage > 70%

---

**Total Estimated Effort**: 17 weeks  
**Team Size**: 2-3 developers  
**Priority**: High → Medium → Low implementation order
