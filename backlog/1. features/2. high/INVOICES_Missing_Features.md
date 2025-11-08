# Invoices - Missing Features Specification

> **Document Type**: Feature Specification  
> **Priority**: High to Low (by section)  
> **Created**: 2025-10-16  
> **Status**: Planning

## Overview

This document outlines features that are completely missing from the Invoice system but are specified in the original requirements. Features are organized by priority to help guide implementation planning.

---

## HIGH PRIORITY FEATURES

### 1. Client Portal (Secure Invoice View & Payment)

**Priority**: 🔴 High  
**Business Value**: Essential for client self-service  
**Effort**: High  

#### Description
Secure, tokenized links that allow clients to view invoices, make payments, and download PDFs without logging in.

#### Current State
- ❌ No public invoice view
- ❌ No token generation for client access
- ❌ No client payment interface

#### Requirements

##### 1.1 Add Token Fields to Schema
**File**: `apps/backend/prisma/schema.prisma`

```prisma
model Invoice {
  // ... existing fields
  
  // Client portal access
  clientToken      String?         @unique  // Secure token for client access
  tokenExpiresAt   DateTime?                // Token expiration (90 days)
  viewedAt         DateTime?                // When client first viewed
  lastViewedAt     DateTime?                // Last view timestamp
  viewCount        Int             @default(0)
}
```

##### 1.2 Token Generation Service
**File**: `apps/backend/src/services/invoiceService.ts`

```typescript
import crypto from 'crypto';

async generateClientToken(invoiceId: number): Promise<string> {
  const invoice = await this.getInvoice(invoiceId);
  
  if (!invoice) {
    throw new Error('Invoice not found');
  }

  // Generate secure random token
  const token = crypto.randomBytes(32).toString('hex');
  
  // Token expires in 90 days
  const tokenExpiresAt = new Date(Date.now() + 90 * 24 * 60 * 60 * 1000);

  await this.db.invoice.update({
    where: { id: invoiceId },
    data: {
      clientToken: token,
      tokenExpiresAt
    }
  });

  return token;
}

async getInvoiceByToken(token: string): Promise<Invoice | null> {
  const invoice = await this.db.invoice.findUnique({
    where: { clientToken: token },
    include: {
      items: true,
      client: true,
      project: true,
      payments: {
        where: { status: 'completed' },
        orderBy: { paymentDate: 'desc' }
      },
      createdBy: {
        select: {
          firstName: true,
          lastName: true,
          email: true,
          phone: true
        }
      }
    }
  });

  if (!invoice) {
    return null;
  }

  // Check if token expired
  if (invoice.tokenExpiresAt && invoice.tokenExpiresAt < new Date()) {
    throw new Error('This invoice link has expired. Please contact us for a new link.');
  }

  // Update view tracking
  const viewCount = invoice.viewCount + 1;
  const updateData: any = {
    viewCount,
    lastViewedAt: new Date()
  };

  if (!invoice.viewedAt) {
    updateData.viewedAt = new Date();
  }

  await this.db.invoice.update({
    where: { id: invoice.id },
    data: updateData
  });

  return invoice;
}
```

##### 1.3 Public API Routes
**File**: `apps/backend/src/routes/public/invoices.ts`

```typescript
import express from 'express';
import { InvoiceService } from '../../services/invoiceService';
import { StripeService } from '../../services/stripeService';

const router = express.Router();

// GET /api/public/invoices/:token - View invoice
router.get('/:token', async (req, res) => {
  try {
    const invoiceService = new InvoiceService(db);
    const invoice = await invoiceService.getInvoiceByToken(req.params.token);
    
    if (!invoice) {
      return res.status(404).json({ error: 'Invoice not found' });
    }

    res.json(invoice);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// GET /api/public/invoices/:token/pdf - Download PDF
router.get('/:token/pdf', async (req, res) => {
  try {
    const invoiceService = new InvoiceService(db);
    const invoice = await invoiceService.getInvoiceByToken(req.params.token);
    
    if (!invoice) {
      return res.status(404).json({ error: 'Invoice not found' });
    }

    const pdfService = new PDFService();
    const pdfBuffer = await pdfService.generateInvoicePDF(invoice);

    res.setHeader('Content-Type', 'application/pdf');
    res.setHeader('Content-Disposition', `attachment; filename=Invoice-${invoice.number}.pdf`);
    res.send(pdfBuffer);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// POST /api/public/invoices/:token/payment-intent - Create payment intent
router.post('/:token/payment-intent', async (req, res) => {
  try {
    const invoiceService = new InvoiceService(db);
    const invoice = await invoiceService.getInvoiceByToken(req.params.token);
    
    if (!invoice) {
      return res.status(404).json({ error: 'Invoice not found' });
    }

    if (invoice.balance <= 0) {
      return res.status(400).json({ error: 'Invoice is already paid' });
    }

    const stripeService = new StripeService(db);
    const paymentIntent = await stripeService.createPaymentIntent(invoice.id);

    res.json({
      clientSecret: paymentIntent.client_secret,
      amount: invoice.balance
    });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

export default router;
```

##### 1.4 Frontend Public Invoice View
**File**: `apps/frontend/src/pages/public/InvoiceView.tsx`

```typescript
import React, { useState, useEffect } from 'react';
import { useParams } from 'react-router-dom';
import axios from 'axios';
import {
  Container,
  Paper,
  Typography,
  Box,
  Grid,
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  Button,
  Chip,
  Alert,
  CircularProgress,
  Divider
} from '@mui/material';
import { StripePaymentForm } from '../../components/invoices/StripePaymentForm';

export const PublicInvoiceView: React.FC = () => {
  const { token } = useParams();
  const [invoice, setInvoice] = useState(null);
  const [loading, setLoading] = useState(true);
  const [showPayment, setShowPayment] = useState(false);

  useEffect(() => {
    loadInvoice();
  }, [token]);

  const loadInvoice = async () => {
    try {
      const response = await axios.get(`/api/public/invoices/${token}`);
      setInvoice(response.data);
    } catch (error) {
      // Show error
    } finally {
      setLoading(false);
    }
  };

  const handleDownloadPDF = async () => {
    try {
      const response = await axios.get(`/api/public/invoices/${token}/pdf`, {
        responseType: 'blob'
      });
      
      const url = window.URL.createObjectURL(new Blob([response.data]));
      const link = document.createElement('a');
      link.href = url;
      link.setAttribute('download', `Invoice-${invoice.number}.pdf`);
      document.body.appendChild(link);
      link.click();
      link.remove();
    } catch (error) {
      // Show error
    }
  };

  if (loading) return <CircularProgress />;

  if (!invoice) {
    return (
      <Container maxWidth="md" sx={{ py: 4 }}>
        <Alert severity="error">Invoice not found or link has expired.</Alert>
      </Container>
    );
  }

  return (
    <Container maxWidth="lg" sx={{ py: 4 }}>
      {/* Header */}
      <Paper sx={{ p: 3, mb: 3 }}>
        <Box sx={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center' }}>
          <Box>
            <Typography variant="h4">Invoice {invoice.number}</Typography>
            <Typography color="textSecondary" sx={{ mt: 1 }}>
              Issued: {new Date(invoice.issueDate).toLocaleDateString()}
            </Typography>
            <Typography color="textSecondary">
              Due: {new Date(invoice.dueDate).toLocaleDateString()}
            </Typography>
          </Box>
          <Box>
            <Chip 
              label={invoice.paymentStatus.replace('_', ' ')} 
              color={getPaymentStatusColor(invoice.paymentStatus)}
              sx={{ mb: 1 }}
            />
            <Typography variant="h4" color="primary">
              ${invoice.balance.toFixed(2)}
            </Typography>
            <Typography variant="caption" color="textSecondary">
              Amount Due
            </Typography>
          </Box>
        </Box>
      </Paper>

      {/* From/To */}
      <Grid container spacing={3}>
        <Grid item xs={12} md={6}>
          <Paper sx={{ p: 2 }}>
            <Typography variant="h6" gutterBottom>From</Typography>
            <Typography>{invoice.createdBy.firstName} {invoice.createdBy.lastName}</Typography>
            <Typography color="textSecondary">{invoice.createdBy.email}</Typography>
            {invoice.createdBy.phone && (
              <Typography color="textSecondary">{invoice.createdBy.phone}</Typography>
            )}
          </Paper>
        </Grid>
        <Grid item xs={12} md={6}>
          <Paper sx={{ p: 2 }}>
            <Typography variant="h6" gutterBottom>Bill To</Typography>
            <Typography>{invoice.client.name}</Typography>
            <Typography color="textSecondary">{invoice.client.email}</Typography>
            {invoice.client.address && (
              <Typography color="textSecondary">{invoice.client.address}</Typography>
            )}
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
              {invoice.items.map((item, idx) => (
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
              <Typography>${invoice.subtotal.toFixed(2)}</Typography>
            </Box>
            <Box sx={{ display: 'flex', justifyContent: 'space-between', mb: 1 }}>
              <Typography>Tax:</Typography>
              <Typography>${invoice.tax.toFixed(2)}</Typography>
            </Box>
            <Divider sx={{ my: 1 }} />
            <Box sx={{ display: 'flex', justifyContent: 'space-between', mb: 1 }}>
              <Typography variant="h6">Total:</Typography>
              <Typography variant="h6">${invoice.total.toFixed(2)}</Typography>
            </Box>
            {invoice.amountPaid > 0 && (
              <>
                <Box sx={{ display: 'flex', justifyContent: 'space-between', mb: 1 }}>
                  <Typography color="textSecondary">Paid:</Typography>
                  <Typography color="textSecondary">-${invoice.amountPaid.toFixed(2)}</Typography>
                </Box>
                <Divider sx={{ my: 1 }} />
                <Box sx={{ display: 'flex', justifyContent: 'space-between' }}>
                  <Typography variant="h6" color="primary">Balance Due:</Typography>
                  <Typography variant="h6" color="primary">${invoice.balance.toFixed(2)}</Typography>
                </Box>
              </>
            )}
          </Box>
        </Box>
      </Paper>

      {/* Payment History */}
      {invoice.payments && invoice.payments.length > 0 && (
        <Paper sx={{ p: 3, mt: 3 }}>
          <Typography variant="h6" gutterBottom>Payment History</Typography>
          <TableContainer>
            <Table size="small">
              <TableHead>
                <TableRow>
                  <TableCell>Date</TableCell>
                  <TableCell>Method</TableCell>
                  <TableCell>Reference</TableCell>
                  <TableCell align="right">Amount</TableCell>
                </TableRow>
              </TableHead>
              <TableBody>
                {invoice.payments.map((payment, idx) => (
                  <TableRow key={idx}>
                    <TableCell>{new Date(payment.paymentDate).toLocaleDateString()}</TableCell>
                    <TableCell>{payment.method}</TableCell>
                    <TableCell>{payment.reference || '-'}</TableCell>
                    <TableCell align="right">${payment.amount.toFixed(2)}</TableCell>
                  </TableRow>
                ))}
              </TableBody>
            </Table>
          </TableContainer>
        </Paper>
      )}

      {/* Notes */}
      {invoice.notes && (
        <Paper sx={{ p: 3, mt: 3 }}>
          <Typography variant="h6" gutterBottom>Notes</Typography>
          <Typography>{invoice.notes}</Typography>
        </Paper>
      )}

      {/* Actions */}
      <Paper sx={{ p: 3, mt: 3 }}>
        <Grid container spacing={2}>
          <Grid item xs={12} md={6}>
            <Button
              variant="outlined"
              fullWidth
              onClick={handleDownloadPDF}
              size="large"
            >
              Download PDF
            </Button>
          </Grid>
          {invoice.balance > 0 && (
            <Grid item xs={12} md={6}>
              <Button
                variant="contained"
                color="primary"
                fullWidth
                size="large"
                onClick={() => setShowPayment(true)}
              >
                Pay ${invoice.balance.toFixed(2)}
              </Button>
            </Grid>
          )}
        </Grid>

        {/* Payment Form */}
        {showPayment && invoice.balance > 0 && (
          <Box sx={{ mt: 3, p: 2, bgcolor: 'grey.50', borderRadius: 1 }}>
            <Typography variant="h6" gutterBottom>
              Pay with Credit Card
            </Typography>
            <StripePaymentForm
              invoiceId={invoice.id}
              amount={invoice.balance}
              onSuccess={() => {
                setShowPayment(false);
                loadInvoice();
              }}
            />
          </Box>
        )}
      </Paper>

      {/* Status Messages */}
      {invoice.paymentStatus === 'PAID' && (
        <Alert severity="success" sx={{ mt: 3 }}>
          This invoice has been paid in full. Thank you!
        </Alert>
      )}

      {invoice.paymentStatus === 'OVERDUE' && (
        <Alert severity="warning" sx={{ mt: 3 }}>
          This invoice is overdue. Please make payment as soon as possible.
        </Alert>
      )}
    </Container>
  );
};

function getPaymentStatusColor(status: string) {
  switch (status) {
    case 'PAID': return 'success';
    case 'PARTIALLY_PAID': return 'info';
    case 'OVERDUE': return 'error';
    default: return 'default';
  }
}
```

#### Acceptance Criteria
- [ ] Generate secure token for each invoice
- [ ] Token expires after 90 days
- [ ] Client can view invoice without login
- [ ] Client can download PDF
- [ ] Client can make payment via Stripe
- [ ] View tracking (first view, last view, count)
- [ ] Payment history displayed
- [ ] Mobile-responsive design
- [ ] Secure token validation
- [ ] Expired token handling

---

### 2. Automated Payment Reminders

**Priority**: 🔴 High  
**Business Value**: Improves cash flow  
**Effort**: Medium  

#### Description
Automatically send email reminders to clients at specific intervals before and after due date.

#### Current State
- ❌ No reminder system
- ❌ No reminder scheduling
- ❌ No reminder templates

#### Requirements

##### 2.1 Reminder Configuration Schema
**File**: `apps/backend/prisma/schema.prisma`

```prisma
model InvoiceReminder {
  id               Int             @id @default(autoincrement())
  invoiceId        Int
  invoice          Invoice         @relation(fields: [invoiceId], references: [id])
  reminderType     ReminderType
  scheduledFor     DateTime
  sentAt           DateTime?
  status           ReminderStatus  @default(PENDING)
  emailContent     String?
  error            String?
  createdAt        DateTime        @default(now())

  @@index([invoiceId])
  @@index([scheduledFor, status])
}

enum ReminderType {
  BEFORE_DUE_3_DAYS
  DUE_TODAY
  OVERDUE_3_DAYS
  OVERDUE_7_DAYS
  OVERDUE_14_DAYS
}

enum ReminderStatus {
  PENDING
  SENT
  FAILED
  CANCELLED
}
```

##### 2.2 Reminder Service
**File**: `apps/backend/src/services/reminderService.ts`

```typescript
import { PrismaClient, ReminderType, ReminderStatus } from '@prisma/client';
import { EmailService } from './emailService';

export class ReminderService {
  private db: PrismaClient;
  private emailService: EmailService;

  constructor(db: PrismaClient) {
    this.db = db;
    this.emailService = new EmailService();
  }

  async scheduleRemindersForInvoice(invoiceId: number): Promise<void> {
    const invoice = await this.db.invoice.findUnique({
      where: { id: invoiceId }
    });

    if (!invoice || invoice.status === 'VOID') {
      return;
    }

    const dueDate = new Date(invoice.dueDate);
    const reminders = [
      {
        type: ReminderType.BEFORE_DUE_3_DAYS,
        scheduledFor: new Date(dueDate.getTime() - 3 * 24 * 60 * 60 * 1000)
      },
      {
        type: ReminderType.DUE_TODAY,
        scheduledFor: dueDate
      },
      {
        type: ReminderType.OVERDUE_3_DAYS,
        scheduledFor: new Date(dueDate.getTime() + 3 * 24 * 60 * 60 * 1000)
      },
      {
        type: ReminderType.OVERDUE_7_DAYS,
        scheduledFor: new Date(dueDate.getTime() + 7 * 24 * 60 * 60 * 1000)
      },
      {
        type: ReminderType.OVERDUE_14_DAYS,
        scheduledFor: new Date(dueDate.getTime() + 14 * 24 * 60 * 60 * 1000)
      }
    ];

    // Only schedule future reminders
    const now = new Date();
    for (const reminder of reminders) {
      if (reminder.scheduledFor > now) {
        await this.db.invoiceReminder.create({
          data: {
            invoiceId,
            reminderType: reminder.type,
            scheduledFor: reminder.scheduledFor,
            status: ReminderStatus.PENDING
          }
        });
      }
    }
  }

  async processPendingReminders(): Promise<void> {
    const now = new Date();

    const pendingReminders = await this.db.invoiceReminder.findMany({
      where: {
        status: ReminderStatus.PENDING,
        scheduledFor: { lte: now }
      },
      include: {
        invoice: {
          include: {
            client: true,
            createdBy: true
          }
        }
      }
    });

    for (const reminder of pendingReminders) {
      await this.sendReminder(reminder);
    }
  }

  private async sendReminder(reminder: any): Promise<void> {
    const { invoice } = reminder;

    // Don't send if invoice is paid or voided
    if (invoice.paymentStatus === 'PAID' || invoice.status === 'VOID') {
      await this.db.invoiceReminder.update({
        where: { id: reminder.id },
        data: { status: ReminderStatus.CANCELLED }
      });
      return;
    }

    try {
      const emailContent = this.generateReminderEmail(invoice, reminder.reminderType);
      
      await this.emailService.sendMail({
        to: invoice.client.email,
        subject: this.getReminderSubject(invoice, reminder.reminderType),
        html: emailContent
      });

      await this.db.invoiceReminder.update({
        where: { id: reminder.id },
        data: {
          status: ReminderStatus.SENT,
          sentAt: new Date(),
          emailContent
        }
      });

      console.log(`Sent ${reminder.reminderType} reminder for invoice ${invoice.number}`);
    } catch (error) {
      await this.db.invoiceReminder.update({
        where: { id: reminder.id },
        data: {
          status: ReminderStatus.FAILED,
          error: error.message
        }
      });

      console.error(`Failed to send reminder for invoice ${invoice.number}:`, error);
    }
  }

  private getReminderSubject(invoice: any, type: ReminderType): string {
    switch (type) {
      case ReminderType.BEFORE_DUE_3_DAYS:
        return `Reminder: Invoice ${invoice.number} due in 3 days`;
      case ReminderType.DUE_TODAY:
        return `Invoice ${invoice.number} is due today`;
      case ReminderType.OVERDUE_3_DAYS:
      case ReminderType.OVERDUE_7_DAYS:
      case ReminderType.OVERDUE_14_DAYS:
        return `Overdue: Invoice ${invoice.number}`;
      default:
        return `Invoice ${invoice.number} reminder`;
    }
  }

  private generateReminderEmail(invoice: any, type: ReminderType): string {
    const viewUrl = `${process.env.APP_URL}/public/invoices/${invoice.clientToken}`;
    const payUrl = `${viewUrl}#payment`;
    
    let message = '';
    let urgency = '';

    switch (type) {
      case ReminderType.BEFORE_DUE_3_DAYS:
        message = 'This is a friendly reminder that the following invoice will be due in 3 days.';
        urgency = 'info';
        break;
      case ReminderType.DUE_TODAY:
        message = 'This invoice is due today. Please make payment at your earliest convenience.';
        urgency = 'warning';
        break;
      case ReminderType.OVERDUE_3_DAYS:
      case ReminderType.OVERDUE_7_DAYS:
      case ReminderType.OVERDUE_14_DAYS:
        message = 'This invoice is now overdue. Please make payment immediately to avoid late fees.';
        urgency = 'error';
        break;
    }

    return `
      <!DOCTYPE html>
      <html>
      <head>
        <style>
          body { font-family: Arial, sans-serif; line-height: 1.6; color: #333; }
          .container { max-width: 600px; margin: 0 auto; padding: 20px; }
          .header { background: ${urgency === 'error' ? '#dc3545' : urgency === 'warning' ? '#ffc107' : '#17a2b8'}; 
                   color: white; padding: 20px; text-align: center; }
          .content { padding: 20px; background: #f9f9f9; }
          .invoice-details { background: white; padding: 15px; margin: 20px 0; border-left: 4px solid #28a745; }
          .button { 
            display: inline-block; 
            padding: 12px 24px; 
            background: #28a745; 
            color: white; 
            text-decoration: none; 
            border-radius: 4px;
            margin: 10px 5px;
          }
          .amount { font-size: 24px; font-weight: bold; color: #dc3545; }
        </style>
      </head>
      <body>
        <div class="container">
          <div class="header">
            <h1>Invoice Reminder</h1>
          </div>
          
          <div class="content">
            <p>Hello ${invoice.client.name},</p>
            
            <p>${message}</p>
            
            <div class="invoice-details">
              <table style="width: 100%;">
                <tr>
                  <td><strong>Invoice Number:</strong></td>
                  <td>${invoice.number}</td>
                </tr>
                <tr>
                  <td><strong>Issue Date:</strong></td>
                  <td>${invoice.issueDate.toLocaleDateString()}</td>
                </tr>
                <tr>
                  <td><strong>Due Date:</strong></td>
                  <td>${invoice.dueDate.toLocaleDateString()}</td>
                </tr>
              </table>
              
              <div style="margin-top: 20px; padding-top: 20px; border-top: 2px solid #eee;">
                <p style="margin: 0;">Amount Due</p>
                <p class="amount" style="margin: 5px 0;">$${invoice.balance.toFixed(2)}</p>
              </div>
            </div>

            <div style="text-align: center; margin: 30px 0;">
              <a href="${payUrl}" class="button">Pay Now</a>
              <a href="${viewUrl}" class="button" style="background: #007bff;">View Invoice</a>
            </div>

            <p style="font-size: 12px; color: #666;">
              If you have any questions about this invoice, please contact ${invoice.createdBy.firstName} ${invoice.createdBy.lastName} at ${invoice.createdBy.email}.
            </p>
          </div>
        </div>
      </body>
      </html>
    `;
  }
}
```

##### 2.3 Cron Job
**File**: `apps/backend/src/jobs/reminderJob.ts`

```typescript
import cron from 'node-cron';
import { PrismaClient } from '@prisma/client';
import { ReminderService } from '../services/reminderService';

export class ReminderJob {
  private db: PrismaClient;
  private reminderService: ReminderService;

  constructor(db: PrismaClient) {
    this.db = db;
    this.reminderService = new ReminderService(db);
  }

  start() {
    // Run every hour
    cron.schedule('0 * * * *', async () => {
      console.log('Processing invoice reminders...');
      await this.reminderService.processPendingReminders();
    });
  }
}
```

#### Acceptance Criteria
- [ ] Reminders scheduled when invoice created
- [ ] Reminders sent at correct intervals (T-3, Due, T+3, T+7, T+14)
- [ ] Reminders not sent for paid/voided invoices
- [ ] Email content appropriate for urgency
- [ ] Failed reminders logged
- [ ] Reminder history viewable
- [ ] Cron job runs reliably
- [ ] Reminders can be manually cancelled

---

### 3. Automated Overdue Marking

**Priority**: 🔴 High  
**Business Value**: Automatic status management  
**Effort**: Low  

#### Description
Automatically mark invoices as OVERDUE when due date passes and balance remains.

#### Requirements

##### 3.1 Cron Job Implementation
**File**: `apps/backend/src/jobs/overdueInvoiceJob.ts`

```typescript
import cron from 'node-cron';
import { PrismaClient, InvoiceStatus } from '@prisma/client';
import { InvoiceService } from '../services/invoiceService';

export class OverdueInvoiceJob {
  private db: PrismaClient;
  private invoiceService: InvoiceService;

  constructor(db: PrismaClient) {
    this.db = db;
    this.invoiceService = new InvoiceService(db);
  }

  start() {
    // Run daily at 1 AM
    cron.schedule('0 1 * * *', async () => {
      await this.markOverdueInvoices();
    });
  }

  async markOverdueInvoices(): Promise<void> {
    const now = new Date();

    const overdueInvoices = await this.db.invoice.findMany({
      where: {
        dueDate: { lt: now },
        balance: { gt: 0 },
        status: { in: [InvoiceStatus.SENT, InvoiceStatus.DRAFT] },
        paymentStatus: { not: 'PAID' }
      }
    });

    for (const invoice of overdueInvoices) {
      try {
        await this.invoiceService.markAsOverdue(invoice.id);
        console.log(`Marked invoice ${invoice.number} as overdue`);
      } catch (error) {
        console.error(`Failed to mark invoice ${invoice.number} as overdue:`, error);
      }
    }

    console.log(`Marked ${overdueInvoices.length} invoices as overdue`);
  }
}
```

#### Acceptance Criteria
- [ ] Daily job runs automatically
- [ ] Invoices past due date marked OVERDUE
- [ ] Only unpaid/partially paid invoices marked
- [ ] VOID invoices excluded
- [ ] History logged
- [ ] Metrics tracked

---

## MEDIUM PRIORITY FEATURES

### 4. Credit Notes

**Priority**: 🟡 Medium  
**Business Value**: Proper refund/adjustment handling  
**Effort**: High  

#### Description
Issue credit notes for returns, refunds, or adjustments.

#### Requirements

##### 4.1 Schema
```prisma
model CreditNote {
  id               Int             @id @default(autoincrement())
  number           String          @unique
  invoiceId        Int
  invoice          Invoice         @relation(fields: [invoiceId], references: [id])
  clientId         Int
  client           Client          @relation(fields: [clientId], references: [id])
  status           CreditNoteStatus @default(DRAFT)
  issueDate        DateTime        @default(now())
  amount           Float
  reason           String
  notes            String?
  items            CreditNoteItem[]
  createdById      Int
  createdBy        User            @relation(fields: [createdById], references: [id])
  createdAt        DateTime        @default(now())
  updatedAt        DateTime        @updatedAt

  @@index([invoiceId])
  @@index([clientId])
}

model CreditNoteItem {
  id               Int             @id @default(autoincrement())
  creditNoteId     Int
  creditNote       CreditNote      @relation(fields: [creditNoteId], references: [id])
  description      String
  quantity         Float
  unitPrice        Float
  total            Float
  
  @@index([creditNoteId])
}

enum CreditNoteStatus {
  DRAFT
  ISSUED
  APPLIED
  VOID
}
```

##### 4.2 Service Methods
```typescript
async createCreditNote(data: {
  invoiceId: number;
  amount: number;
  reason: string;
  items: Array<{
    description: string;
    quantity: number;
    unitPrice: number;
  }>;
}): Promise<CreditNote> {
  // Implementation
}

async applyCreditNote(creditNoteId: number): Promise<void> {
  // Reduce invoice balance
  // Update credit note status
  // Log transaction
}
```

#### Acceptance Criteria
- [ ] Credit notes created against invoices
- [ ] Unique numbering (CN-YYYY-NNNN)
- [ ] Can be partially or fully credited
- [ ] Applied credits reduce invoice balance
- [ ] PDF generation for credit notes
- [ ] Client notification
- [ ] History tracked

---

### 5. Recurring Invoice Templates

**Priority**: 🟡 Medium  
**Business Value**: Automate repeat billing  
**Effort**: High  

#### Description
Create templates for recurring invoices (subscriptions, retainers, etc.).

#### Requirements

##### 5.1 Enhanced Schema
```prisma
model RecurringInvoice {
  id               Int             @id @default(autoincrement())
  templateName     String
  clientId         Int
  client           Client          @relation(fields: [clientId], references: [id])
  frequency        Frequency
  startDate        DateTime
  endDate          DateTime?
  nextInvoiceDate  DateTime
  lastInvoiceDate  DateTime?
  status           RecurringStatus @default(ACTIVE)
  
  // Invoice template data
  items            Json            // Array of line items
  paymentTerms     String?
  notes            String?
  
  // Generated invoices
  generatedInvoices Invoice[]      @relation("RecurringInvoices")
  
  createdById      Int
  createdBy        User            @relation(fields: [createdById], references: [id])
  createdAt        DateTime        @default(now())
  updatedAt        DateTime        @updatedAt

  @@index([clientId])
  @@index([nextInvoiceDate, status])
}

enum Frequency {
  WEEKLY
  BIWEEKLY
  MONTHLY
  QUARTERLY
  ANNUALLY
}

enum RecurringStatus {
  ACTIVE
  PAUSED
  CANCELLED
  EXPIRED
}
```

##### 5.2 Generation Job
```typescript
async generateRecurringInvoices(): Promise<void> {
  const now = new Date();

  const dueRecurring = await this.db.recurringInvoice.findMany({
    where: {
      status: RecurringStatus.ACTIVE,
      nextInvoiceDate: { lte: now }
    }
  });

  for (const recurring of dueRecurring) {
    // Generate invoice from template
    // Update nextInvoiceDate
    // Send to client if configured
  }
}
```

#### Acceptance Criteria
- [ ] Create recurring invoice templates
- [ ] Support multiple frequencies
- [ ] Automatic invoice generation
- [ ] Can pause/resume
- [ ] Can cancel
- [ ] End date support
- [ ] Notification on generation

---

### 6. Late Fee Automation

**Priority**: 🟡 Medium  
**Business Value**: Incentivize timely payment  
**Effort**: Medium  

#### Requirements

##### 6.1 Configuration
```prisma
model LateFeeConfig {
  id               Int             @id @default(autoincrement())
  userId           Int             @unique
  user             User            @relation(fields: [userId], references: [id])
  enabled          Boolean         @default(false)
  feeType          FeeType         @default(PERCENTAGE)
  feeAmount        Float           // Percentage or fixed amount
  gracePeriodDays  Int             @default(0)
  maxFee           Float?          // Maximum fee cap
  compoundDaily    Boolean         @default(false)
}

enum FeeType {
  PERCENTAGE
  FIXED
}
```

##### 6.2 Calculation Logic
```typescript
async calculateLateFee(invoiceId: number): Promise<number> {
  const invoice = await this.getInvoice(invoiceId);
  const config = await this.getLateFeeConfig(invoice.createdById);

  if (!config.enabled) return 0;

  const daysOverdue = Math.max(0, 
    Math.floor((Date.now() - invoice.dueDate.getTime()) / (1000 * 60 * 60 * 24)) - config.gracePeriodDays
  );

  if (daysOverdue <= 0) return 0;

  let fee = 0;
  if (config.feeType === 'PERCENTAGE') {
    fee = invoice.balance * (config.feeAmount / 100);
    if (config.compoundDaily) {
      fee *= daysOverdue;
    }
  } else {
    fee = config.feeAmount * (config.compoundDaily ? daysOverdue : 1);
  }

  if (config.maxFee) {
    fee = Math.min(fee, config.maxFee);
  }

  return fee;
}
```

---

## LOW PRIORITY FEATURES

### 7. Digital Signature & Hash

**Priority**: 🟢 Low  
**Business Value**: Audit trail, tamper-proof  
**Effort**: Medium  

#### Requirements

```prisma
model Invoice {
  // ... existing fields
  pdfHash          String?         // SHA-256 hash of PDF
  digitalSignature String?         // Digital signature
  signedAt         DateTime?
}
```

Generate hash when PDF created, verify on retrieval.

---

### 8. Multi-Currency Support

**Priority**: 🟢 Low  
**Business Value**: International clients  
**Effort**: Medium  

#### Requirements

- Currency selector in invoice form
- Exchange rate storage
- Currency display formatting
- Multi-currency reporting

Already partially implemented (currency field exists).

---

### 9. Payment Terms Field

**Priority**: 🟢 Low  
**Business Value**: Clear expectations  
**Effort**: Low  

#### Requirements

```prisma
model Invoice {
  // ... existing fields
  paymentTerms     String?         // "Net 30", "Net 15", "Due on receipt"
}
```

Predefined options + custom text.

---

### 10. Delivery/Service Date

**Priority**: 🟢 Low  
**Business Value**: Service tracking  
**Effort**: Low  

#### Requirements

```prisma
model Invoice {
  // ... existing fields
  serviceDate      DateTime?       // Date service performed
  deliveryDate     DateTime?       // Date goods delivered
}
```

---

### 11. Attachments

**Priority**: 🟢 Low  
**Business Value**: Supporting documentation  
**Effort**: Medium  

#### Requirements

```prisma
model InvoiceAttachment {
  id               Int             @id @default(autoincrement())
  invoiceId        Int
  invoice          Invoice         @relation(fields: [invoiceId], references: [id])
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

File storage: AWS S3 or Azure Blob.

---

### 12. External Integrations

**Priority**: 🟢 Low  
**Business Value**: Workflow automation  
**Effort**: Very High  

#### Integrations Needed

**Accounting Software:**
- QuickBooks Online
- Xero
- FreshBooks

**Payment Gateways:**
- PayPal (in addition to Stripe)
- Square
- ACH/Bank transfers

**CRM:**
- HubSpot
- Salesforce

**Notifications:**
- Slack webhooks
- Microsoft Teams

Each requires:
- OAuth setup
- API client
- Webhook handling
- Data mapping
- Sync logic
- Error handling
- Retry mechanism

---

### 13. Compliance Features

**Priority**: 🟢 Low  
**Business Value**: Legal compliance  
**Effort**: Medium  

#### Requirements

**GDPR Compliance:**
- Data export
- Data deletion
- Consent tracking

**Tax Compliance:**
- Tax registration number display
- VAT/GST compliance
- Tax reporting

**Data Retention:**
- 7-year retention policy
- Automatic archival
- Audit logs

---

### 14. Advanced Status States

**Priority**: 🟢 Low  
**Business Value**: Better tracking  
**Effort**: Low  

#### Requirements

Add to InvoiceStatus enum:
- VIEWED (client opened invoice)
- CLOSED (archived, not voided)
- DISPUTED (client contests charges)

Track transitions and reasons.

---

## Testing Requirements

### Unit Tests
```typescript
describe('Client Portal', () => {
  it('should generate unique tokens', async () => {});
  it('should track view counts', async () => {});
  it('should handle expired tokens', async () => {});
});

describe('Reminder Service', () => {
  it('should schedule reminders correctly', async () => {});
  it('should send reminders at right time', async () => {});
  it('should cancel reminders for paid invoices', async () => {});
});

describe('Credit Notes', () => {
  it('should create credit note', async () => {});
  it('should apply credit to invoice', async () => {});
  it('should reduce balance correctly', async () => {});
});
```

### Integration Tests
```typescript
describe('Invoice Lifecycle E2E', () => {
  it('should complete full payment flow', async () => {
    // Create -> Send -> Client View -> Pay -> Confirm
  });

  it('should handle overdue flow', async () => {
    // Create -> Send -> Overdue -> Reminder -> Pay
  });
});
```

---

## Implementation Roadmap

### Phase 1: Client-Facing Features (High Priority)
**Timeline**: 4 weeks

1. Client portal (Week 1-2)
2. Payment reminders (Week 2-3)
3. Overdue automation (Week 3)
4. Testing (Week 4)

### Phase 2: Financial Features (Medium Priority)
**Timeline**: 4 weeks

5. Credit notes (Week 1-2)
6. Recurring invoices (Week 2-3)
7. Late fees (Week 3-4)

### Phase 3: Enhancement Features (Low Priority)
**Timeline**: 3 weeks

8. Digital signatures (Week 1)
9. Multi-currency (Week 1)
10. Attachments (Week 2)
11. Advanced statuses (Week 2)
12. Compliance features (Week 3)

### Phase 4: Integrations (Low Priority)
**Timeline**: 8 weeks

13. External integrations (accounting, CRM, payment gateways)

---

## Success Metrics

- [ ] 100% of high priority features implemented
- [ ] Client portal adoption > 90%
- [ ] Online payment rate > 75%
- [ ] Reminder effectiveness > 60% (payment after reminder)
- [ ] Overdue rate reduced by 40%
- [ ] Zero security vulnerabilities
- [ ] Page load time < 2 seconds
- [ ] Unit test coverage > 85%
- [ ] Integration test coverage > 75%
- [ ] Payment processing uptime > 99.9%

---

## Dependencies

- Email service (SMTP/SendGrid/AWS SES)
- Stripe payment gateway
- Cron job scheduler
- File storage (AWS S3/Azure Blob)
- PDF generation library

---

## Risk Mitigation

**Risk**: Payment gateway downtime
- **Mitigation**: Fallback payment methods, status monitoring, client notifications

**Risk**: Email delivery failures
- **Mitigation**: Retry logic, bounce handling, alternative notification methods

**Risk**: Security vulnerabilities in client portal
- **Mitigation**: Security audit, penetration testing, token expiration, rate limiting

**Risk**: Cron job failures
- **Mitigation**: Monitoring, alerting, manual trigger endpoints, job logs

---

**Total Estimated Effort**: 19 weeks  
**Team Size**: 2-3 developers  
**Priority**: High → Medium → Low implementation order
