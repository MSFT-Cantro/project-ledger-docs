# Invoices - Partially Implemented Features Completion Spec

> **Document Type**: Implementation Guide  
> **Priority**: High  
> **Created**: 2025-10-16  
> **Status**: Active

## Overview

This document outlines features that are partially implemented in the Invoice system and provides detailed specifications for completing them. These features have foundational work done but need additional implementation to meet the full specification requirements.

---

## 1. Payment Gateway Integration (Stripe)

### Current State
- ✅ Stripe schema fields exist in Invoice model
- ✅ Payment model exists with gateway fields
- ⚠️ Minimal Stripe integration implemented
- ❌ No webhook handling for payment events
- ❌ No payment processing UI

### What's Missing
- Full Stripe payment processing
- Webhook event handling
- Payment intent creation
- Payment method management
- Refund processing

### Implementation Tasks

#### 1.1 Install Stripe SDK
```bash
cd apps/backend
npm install stripe
npm install --save-dev @types/stripe
```

#### 1.2 Stripe Service Implementation
**File**: `apps/backend/src/services/stripeService.ts`

```typescript
import Stripe from 'stripe';
import { PrismaClient } from '@prisma/client';

export class StripeService {
  private stripe: Stripe;
  private db: PrismaClient;

  constructor(db: PrismaClient) {
    this.stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
      apiVersion: '2023-10-16'
    });
    this.db = db;
  }

  async createPaymentIntent(invoiceId: number): Promise<Stripe.PaymentIntent> {
    const invoice = await this.db.invoice.findUnique({
      where: { id: invoiceId },
      include: { client: true }
    });

    if (!invoice) {
      throw new Error('Invoice not found');
    }

    // Get or create Stripe customer
    let stripeCustomerId = invoice.client.stripeCustomerId;
    
    if (!stripeCustomerId) {
      const customer = await this.stripe.customers.create({
        email: invoice.client.email,
        name: invoice.client.name,
        metadata: {
          clientId: invoice.clientId.toString()
        }
      });

      stripeCustomerId = customer.id;

      // Save to client
      await this.db.client.update({
        where: { id: invoice.clientId },
        data: { stripeCustomerId }
      });
    }

    // Create payment intent
    const paymentIntent = await this.stripe.paymentIntents.create({
      amount: Math.round(invoice.balance * 100), // Convert to cents
      currency: invoice.currency.toLowerCase(),
      customer: stripeCustomerId,
      metadata: {
        invoiceId: invoice.id.toString(),
        invoiceNumber: invoice.number
      },
      automatic_payment_methods: {
        enabled: true
      },
      description: `Payment for Invoice ${invoice.number}`
    });

    // Store payment intent ID
    await this.db.invoice.update({
      where: { id: invoiceId },
      data: {
        stripePaymentIntentId: paymentIntent.id,
        stripePaymentIntentStatus: paymentIntent.status
      }
    });

    return paymentIntent;
  }

  async confirmPayment(paymentIntentId: string): Promise<Stripe.PaymentIntent> {
    return this.stripe.paymentIntents.confirm(paymentIntentId);
  }

  async getPaymentIntent(paymentIntentId: string): Promise<Stripe.PaymentIntent> {
    return this.stripe.paymentIntents.retrieve(paymentIntentId);
  }

  async createRefund(paymentId: number, amount?: number): Promise<Stripe.Refund> {
    const payment = await this.db.payment.findUnique({
      where: { id: paymentId },
      include: { invoice: true }
    });

    if (!payment || !payment.stripePaymentIntentId) {
      throw new Error('Payment not found or not paid via Stripe');
    }

    const refund = await this.stripe.refunds.create({
      payment_intent: payment.stripePaymentIntentId,
      amount: amount ? Math.round(amount * 100) : undefined, // Partial or full refund
      metadata: {
        paymentId: payment.id.toString(),
        invoiceId: payment.invoiceId.toString()
      }
    });

    // Record refund
    await this.db.payment.create({
      data: {
        invoiceId: payment.invoiceId,
        amount: -(refund.amount / 100),
        method: 'stripe_refund',
        status: 'completed',
        stripePaymentIntentId: payment.stripePaymentIntentId,
        stripeRefundId: refund.id,
        paymentDate: new Date(),
        reference: `Refund for ${payment.reference}`
      }
    });

    return refund;
  }

  async handleWebhook(event: Stripe.Event): Promise<void> {
    switch (event.type) {
      case 'payment_intent.succeeded':
        await this.handlePaymentSuccess(event.data.object as Stripe.PaymentIntent);
        break;

      case 'payment_intent.payment_failed':
        await this.handlePaymentFailure(event.data.object as Stripe.PaymentIntent);
        break;

      case 'charge.refunded':
        await this.handleRefund(event.data.object as Stripe.Charge);
        break;

      default:
        console.log(`Unhandled event type: ${event.type}`);
    }
  }

  private async handlePaymentSuccess(paymentIntent: Stripe.PaymentIntent): Promise<void> {
    const invoiceId = parseInt(paymentIntent.metadata.invoiceId);

    // Update invoice
    await this.db.invoice.update({
      where: { id: invoiceId },
      data: {
        stripePaymentIntentStatus: paymentIntent.status
      }
    });

    // Record payment
    await this.db.payment.create({
      data: {
        invoiceId,
        amount: paymentIntent.amount / 100,
        method: 'stripe',
        status: 'completed',
        stripePaymentIntentId: paymentIntent.id,
        paymentDate: new Date(),
        reference: `Stripe Payment ${paymentIntent.id}`
      }
    });

    // Update invoice status via InvoiceService
    const invoiceService = new InvoiceService(this.db);
    await invoiceService.recordPayment(invoiceId, {
      amount: paymentIntent.amount / 100,
      method: 'stripe',
      reference: paymentIntent.id
    });
  }

  private async handlePaymentFailure(paymentIntent: Stripe.PaymentIntent): Promise<void> {
    const invoiceId = parseInt(paymentIntent.metadata.invoiceId);

    await this.db.invoice.update({
      where: { id: invoiceId },
      data: {
        stripePaymentIntentStatus: paymentIntent.status
      }
    });

    // Notify about failure
    // await notificationService.notifyPaymentFailed(invoiceId);
  }

  private async handleRefund(charge: Stripe.Charge): Promise<void> {
    // Handle refund webhook
    console.log('Refund processed:', charge.id);
  }
}
```

#### 1.3 Webhook Endpoint
**File**: `apps/backend/src/routes/webhooks/stripe.ts`

```typescript
import express from 'express';
import Stripe from 'stripe';
import { StripeService } from '../../services/stripeService';

const router = express.Router();
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2023-10-16'
});

// POST /api/webhooks/stripe
router.post('/', express.raw({ type: 'application/json' }), async (req, res) => {
  const sig = req.headers['stripe-signature'];

  if (!sig) {
    return res.status(400).send('Missing stripe-signature header');
  }

  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(
      req.body,
      sig,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch (err) {
    console.error('Webhook signature verification failed:', err.message);
    return res.status(400).send(`Webhook Error: ${err.message}`);
  }

  // Handle the event
  const stripeService = new StripeService(db);
  
  try {
    await stripeService.handleWebhook(event);
    res.json({ received: true });
  } catch (error) {
    console.error('Error handling webhook:', error);
    res.status(500).json({ error: 'Webhook handler failed' });
  }
});

export default router;
```

#### 1.4 Payment API Endpoint
**File**: `apps/backend/src/routes/invoices.ts`

```typescript
// POST /api/invoices/:id/payment-intent
router.post('/:id/payment-intent', async (req, res) => {
  try {
    const invoiceId = parseInt(req.params.id);
    const userId = req.user?.id;

    if (!userId) {
      return res.status(401).json({ error: 'Unauthorized' });
    }

    const stripeService = new StripeService(db);
    const paymentIntent = await stripeService.createPaymentIntent(invoiceId);

    res.json({
      clientSecret: paymentIntent.client_secret,
      paymentIntentId: paymentIntent.id
    });
  } catch (error) {
    logger.error('Error creating payment intent:', error);
    res.status(500).json({ error: error.message });
  }
});
```

#### 1.5 Frontend Payment Component
**File**: `apps/frontend/src/components/invoices/StripePaymentForm.tsx`

```typescript
import React, { useState, useEffect } from 'react';
import { loadStripe } from '@stripe/stripe-js';
import {
  PaymentElement,
  Elements,
  useStripe,
  useElements
} from '@stripe/react-stripe-js';
import { Box, Button, Alert, CircularProgress } from '@mui/material';

const stripePromise = loadStripe(process.env.REACT_APP_STRIPE_PUBLIC_KEY!);

interface PaymentFormProps {
  invoiceId: number;
  amount: number;
  onSuccess: () => void;
}

const CheckoutForm: React.FC<PaymentFormProps> = ({ invoiceId, amount, onSuccess }) => {
  const stripe = useStripe();
  const elements = useElements();
  const [message, setMessage] = useState('');
  const [isProcessing, setIsProcessing] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    if (!stripe || !elements) {
      return;
    }

    setIsProcessing(true);

    const { error } = await stripe.confirmPayment({
      elements,
      confirmParams: {
        return_url: `${window.location.origin}/invoices/${invoiceId}/payment-success`
      }
    });

    if (error) {
      setMessage(error.message || 'Payment failed');
      setIsProcessing(false);
    } else {
      onSuccess();
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <PaymentElement />
      
      {message && (
        <Alert severity="error" sx={{ mt: 2 }}>
          {message}
        </Alert>
      )}

      <Button
        type="submit"
        variant="contained"
        fullWidth
        disabled={!stripe || isProcessing}
        sx={{ mt: 3 }}
      >
        {isProcessing ? (
          <CircularProgress size={24} />
        ) : (
          `Pay $${amount.toFixed(2)}`
        )}
      </Button>
    </form>
  );
};

export const StripePaymentForm: React.FC<PaymentFormProps> = (props) => {
  const [clientSecret, setClientSecret] = useState('');
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Create payment intent
    fetch(`/api/invoices/${props.invoiceId}/payment-intent`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' }
    })
      .then(res => res.json())
      .then(data => {
        setClientSecret(data.clientSecret);
        setLoading(false);
      })
      .catch(err => {
        console.error('Error creating payment intent:', err);
        setLoading(false);
      });
  }, [props.invoiceId]);

  if (loading) {
    return <CircularProgress />;
  }

  if (!clientSecret) {
    return <Alert severity="error">Failed to initialize payment</Alert>;
  }

  const options = {
    clientSecret,
    appearance: { theme: 'stripe' as const }
  };

  return (
    <Elements stripe={stripePromise} options={options}>
      <CheckoutForm {...props} />
    </Elements>
  );
};
```

### Acceptance Criteria
- [ ] Payment intent created for unpaid invoices
- [ ] Stripe customer created/linked to client
- [ ] Payment form integrated in invoice view
- [ ] Webhooks handle payment success/failure
- [ ] Payment recorded in database
- [ ] Invoice status updated automatically
- [ ] Refunds can be processed
- [ ] Payment history shows Stripe transactions
- [ ] Error handling for failed payments
- [ ] PCI compliance maintained (Stripe handles sensitive data)

---

## 2. Email Send Functionality

### Current State
- ✅ Send endpoint exists (`POST /api/invoices/:id/send`)
- ⚠️ Only updates status, doesn't send email
- ❌ No email template
- ❌ No email tracking

### What's Missing
- Actual email sending
- Email templates
- Delivery tracking
- Attachment support

### Implementation Tasks

#### 2.1 Email Template
**File**: `apps/backend/src/templates/invoiceEmail.ts`

```typescript
import { Invoice, Client, User } from '@prisma/client';

export interface InvoiceEmailData {
  invoice: Invoice & {
    client: Client;
    createdBy: User;
  };
  viewUrl: string;
  payUrl: string;
  pdfUrl: string;
}

export function generateInvoiceEmail(data: InvoiceEmailData): string {
  const { invoice, viewUrl, payUrl, pdfUrl } = data;

  return `
    <!DOCTYPE html>
    <html>
    <head>
      <style>
        body { font-family: Arial, sans-serif; line-height: 1.6; color: #333; }
        .container { max-width: 600px; margin: 0 auto; padding: 20px; }
        .header { background: #28a745; color: white; padding: 20px; text-align: center; }
        .content { padding: 20px; background: #f9f9f9; }
        .invoice-details {
          background: white;
          padding: 15px;
          margin: 20px 0;
          border-left: 4px solid #28a745;
        }
        .button {
          display: inline-block;
          padding: 12px 24px;
          background: #28a745;
          color: white;
          text-decoration: none;
          border-radius: 4px;
          margin: 10px 5px;
        }
        .button-secondary {
          background: #007bff;
        }
        .total {
          font-size: 24px;
          font-weight: bold;
          color: #28a745;
        }
        .footer { padding: 20px; text-align: center; color: #666; font-size: 12px; }
      </style>
    </head>
    <body>
      <div class="container">
        <div class="header">
          <h1>Invoice ${invoice.number}</h1>
        </div>
        
        <div class="content">
          <p>Hello ${invoice.client.name},</p>
          
          <p>Thank you for your business! Please find your invoice details below.</p>
          
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
              <tr>
                <td><strong>Payment Terms:</strong></td>
                <td>${invoice.paymentTerms || 'Due on receipt'}</td>
              </tr>
            </table>
            
            <div style="margin-top: 20px; padding-top: 20px; border-top: 2px solid #eee;">
              <p style="margin: 0;">Amount Due</p>
              <p class="total" style="margin: 5px 0;">$${invoice.balance.toFixed(2)}</p>
            </div>
          </div>

          ${invoice.notes ? `<p><strong>Notes:</strong><br>${invoice.notes}</p>` : ''}

          <div style="text-align: center; margin: 30px 0;">
            <a href="${payUrl}" class="button">Pay Invoice</a>
            <a href="${viewUrl}" class="button button-secondary">View Invoice</a>
            <a href="${pdfUrl}" class="button button-secondary">Download PDF</a>
          </div>

          <p style="font-size: 12px; color: #666;">
            If you have any questions about this invoice, please contact us.
          </p>
        </div>
        
        <div class="footer">
          <p>This is an automated message. Please do not reply to this email.</p>
          <p>${invoice.createdBy.firstName} ${invoice.createdBy.lastName} | ${invoice.createdBy.email}</p>
        </div>
      </div>
    </body>
    </html>
  `;
}
```

#### 2.2 Update InvoiceService
**File**: `apps/backend/src/services/invoiceService.ts`

```typescript
async sendInvoice(invoiceId: number, userId: number): Promise<void> {
  const invoice = await this.db.invoice.findUnique({
    where: { id: invoiceId },
    include: {
      client: true,
      createdBy: true
    }
  });

  if (!invoice) {
    throw new Error('Invoice not found');
  }

  // Generate client token if not exists
  let token = invoice.clientToken;
  if (!token) {
    token = await this.generateClientToken(invoiceId);
  }

  const baseUrl = process.env.APP_URL;
  const viewUrl = `${baseUrl}/public/invoices/${token}`;
  const payUrl = `${baseUrl}/public/invoices/${token}/pay`;
  const pdfUrl = `${baseUrl}/api/invoices/${invoiceId}/pdf?token=${token}`;

  // Generate email HTML
  const emailHtml = generateInvoiceEmail({
    invoice,
    viewUrl,
    payUrl,
    pdfUrl
  });

  // Send email
  const emailService = new EmailService();
  await emailService.sendMail({
    to: invoice.client.email,
    subject: `Invoice ${invoice.number} from ${invoice.createdBy.firstName} ${invoice.createdBy.lastName}`,
    html: emailHtml
  });

  // Update invoice
  await this.db.invoice.update({
    where: { id: invoiceId },
    data: {
      status: InvoiceStatus.SENT,
      sentAt: new Date()
    }
  });

  // Log history
  await this.db.invoiceHistory.create({
    data: {
      invoiceId,
      changedById: userId,
      changes: {
        status: InvoiceStatus.SENT,
        sentAt: new Date(),
        sentTo: invoice.client.email
      }
    }
  });
}
```

### Acceptance Criteria
- [ ] Send endpoint actually sends email
- [ ] Email contains invoice details
- [ ] Secure view/pay links included
- [ ] PDF attachment or download link
- [ ] Email template is professional and branded
- [ ] Delivery status tracked
- [ ] Bounce handling implemented
- [ ] History logged with send timestamp

---

## 3. Multi-Rate Tax Calculation

### Current State
- ✅ TaxService integration exists
- ✅ Single tax field calculated
- ⚠️ Multi-jurisdiction capability not fully leveraged
- ❌ No tax breakdown stored

### What's Missing
- Multi-rate tax support
- Tax jurisdiction selection
- Tax breakdown display
- Compound tax handling

### Implementation Tasks

#### 3.1 Update Invoice Schema
**File**: `apps/backend/prisma/schema.prisma`

```prisma
model Invoice {
  // ... existing fields
  
  // Enhanced tax fields
  taxDetails       Json?           // Store array of applied taxes
  jurisdiction     String?         // Tax jurisdiction
  compoundTax      Boolean         @default(false)
}
```

#### 3.2 Enhance Tax Calculation in InvoiceService
**File**: `apps/backend/src/services/invoiceService.ts`

```typescript
interface TaxLineItem {
  code: string;
  label: string;
  rate: number;
  taxableAmount: number;
  taxAmount: number;
  isCompound: boolean;
}

private async calculateInvoiceTotals(
  items: InvoiceItem[],
  userId: number,
  clientId: number
): Promise<{
  subtotal: number;
  tax: number;
  total: number;
  taxDetails: TaxLineItem[];
}> {
  const subtotal = items.reduce((sum, item) => sum + item.total, 0);

  // Get multi-rate tax calculation
  const taxCalc = await this.taxService.calculateTax({
    userId,
    clientId,
    lineItems: items.map((item, idx) => ({
      id: idx + 1,
      description: item.description,
      quantity: item.quantity,
      unitPrice: item.unitPrice,
      amount: item.total,
      taxable: item.taxable || true,
      category: 'service'
    })),
    overrides: {}
  });

  const taxDetails: TaxLineItem[] = taxCalc.appliedTaxes.map(tax => ({
    code: tax.taxType,
    label: tax.taxLabel || tax.taxType,
    rate: tax.rate,
    taxableAmount: tax.taxableAmount,
    taxAmount: tax.amount,
    isCompound: tax.isCompound || false
  }));

  return {
    subtotal,
    tax: taxCalc.totalTaxAmount,
    total: subtotal + taxCalc.totalTaxAmount,
    taxDetails
  };
}
```

#### 3.3 Frontend Tax Breakdown Display
**File**: `apps/frontend/src/components/invoices/InvoiceTaxBreakdown.tsx`

```typescript
interface TaxBreakdownProps {
  taxDetails: Array<{
    code: string;
    label: string;
    rate: number;
    taxableAmount: number;
    taxAmount: number;
    isCompound: boolean;
  }>;
}

export const InvoiceTaxBreakdown: React.FC<TaxBreakdownProps> = ({ taxDetails }) => {
  if (!taxDetails || taxDetails.length === 0) {
    return null;
  }

  return (
    <Box sx={{ mt: 2 }}>
      <Typography variant="subtitle2" gutterBottom>
        Tax Breakdown
      </Typography>
      {taxDetails.map((tax, idx) => (
        <Box
          key={idx}
          sx={{
            display: 'flex',
            justifyContent: 'space-between',
            py: 0.5,
            fontSize: '0.875rem'
          }}
        >
          <Typography variant="body2">
            {tax.label} ({(tax.rate * 100).toFixed(2)}%)
            {tax.isCompound && ' (Compound)'}
          </Typography>
          <Typography variant="body2">
            ${tax.taxAmount.toFixed(2)}
          </Typography>
        </Box>
      ))}
    </Box>
  );
};
```

### Acceptance Criteria
- [ ] Multiple tax rates calculated correctly
- [ ] Compound taxes supported
- [ ] Tax jurisdiction stored
- [ ] Tax breakdown displayed in UI
- [ ] Tax breakdown included in PDF
- [ ] Historical tax details preserved

---

## 4. Quote-to-Invoice Conversion

### Current State
- ✅ quoteId field exists in Invoice schema
- ❌ No conversion endpoint
- ❌ No automatic data transfer

### What's Missing
- Conversion API endpoint
- Data mapping logic
- Status updates
- Link preservation

### Implementation Tasks

#### 4.1 Add Conversion Method to QuoteService
**File**: `apps/backend/src/services/quoteService.ts`

```typescript
async convertToInvoice(
  quoteId: number,
  userId: number,
  options: {
    dueDate?: Date;
    paymentTerms?: string;
    notes?: string;
  }
): Promise<Invoice> {
  const quote = await this.getQuote(quoteId);
  
  if (!quote || quote.status !== QuoteStatus.ACCEPTED) {
    throw new Error('Only accepted quotes can be converted');
  }

  // Check if already converted
  const existing = await this.db.invoice.findFirst({
    where: { quoteId }
  });

  if (existing) {
    throw new Error('Quote already converted to invoice');
  }

  return this.db.$transaction(async (tx) => {
    const invoiceService = new InvoiceService(this.db);
    const invoiceNumber = await invoiceService.generateInvoiceNumber();

    // Create invoice
    const invoice = await tx.invoice.create({
      data: {
        number: invoiceNumber,
        quoteId,
        clientId: quote.clientId,
        projectId: quote.projectId,
        status: InvoiceStatus.DRAFT,
        paymentStatus: PaymentStatus.UNPAID,
        issueDate: new Date(),
        dueDate: options.dueDate || new Date(Date.now() + 30 * 24 * 60 * 60 * 1000),
        paymentTerms: options.paymentTerms || 'Net 30',
        subtotal: quote.subtotal,
        tax: quote.tax,
        total: quote.total,
        amountPaid: 0,
        balance: quote.total,
        currency: quote.currency,
        notes: options.notes || quote.notes,
        taxDetails: quote.taxDetails,
        jurisdiction: quote.jurisdiction,
        createdById: userId
      }
    });

    // Copy line items
    const quoteItems = await tx.quoteItem.findMany({
      where: { quoteId }
    });

    for (const item of quoteItems) {
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

    return invoice;
  });
}
```

#### 4.2 Add API Endpoint
**File**: `apps/backend/src/routes/quotes.ts`

```typescript
// POST /api/quotes/:id/convert-to-invoice
router.post('/:id/convert-to-invoice', async (req, res) => {
  try {
    const quoteId = parseInt(req.params.id);
    const userId = req.user.id;

    const invoice = await quoteService.convertToInvoice(quoteId, userId, {
      dueDate: req.body.dueDate ? new Date(req.body.dueDate) : undefined,
      paymentTerms: req.body.paymentTerms,
      notes: req.body.notes
    });

    res.json(invoice);
  } catch (error) {
    logger.error('Error converting quote to invoice:', error);
    res.status(500).json({ error: error.message });
  }
});
```

### Acceptance Criteria
- [ ] Accepted quotes can be converted
- [ ] All quote data transferred to invoice
- [ ] Line items copied correctly
- [ ] Quote status updated to CONVERTED
- [ ] Cannot convert same quote twice
- [ ] Invoice links back to quote
- [ ] History logged on both records

---

## 5. Validation Strengthening

### Current State
- ✅ Basic Zod validation exists
- ⚠️ Not comprehensive
- ❌ Business rules not enforced
- ❌ Date validation incomplete

### What's Missing
- Comprehensive field validation
- Business rule enforcement
- Date range validation
- Amount validations

### Implementation Tasks

#### 5.1 Comprehensive Validation Schema
**File**: `apps/backend/src/routes/invoices.ts`

```typescript
const createInvoiceSchema = z.object({
  clientId: z.union([z.string(), z.number()])
    .transform(val => typeof val === 'string' ? parseInt(val) : val)
    .refine(val => val > 0, 'Valid client required'),
  projectId: z.union([z.string(), z.number()])
    .transform(val => typeof val === 'string' ? parseInt(val) : val)
    .optional(),
  quoteId: z.union([z.string(), z.number()])
    .transform(val => typeof val === 'string' ? parseInt(val) : val)
    .optional(),
  issueDate: z.string()
    .datetime()
    .refine(date => new Date(date) <= new Date(), 'Issue date cannot be in the future'),
  dueDate: z.string()
    .datetime()
    .refine(date => new Date(date) > new Date(), 'Due date must be in the future'),
  paymentTerms: z.string().max(100).optional(),
  items: z.array(z.object({
    description: z.string().min(1, 'Item description required').max(500),
    quantity: z.number().positive('Quantity must be positive'),
    unitPrice: z.number().min(0, 'Unit price cannot be negative'),
    taxable: z.boolean().optional().default(true)
  }))
    .min(1, 'At least one line item required')
    .refine(items => items.some(item => item.quantity * item.unitPrice > 0), 
      'At least one billable item required'),
  currency: z.string().length(3, 'Currency must be 3-character code').default('USD'),
  notes: z.string().max(5000, 'Notes too long').optional()
}).refine(
  data => new Date(data.dueDate) > new Date(data.issueDate),
  { message: 'Due date must be after issue date', path: ['dueDate'] }
);

const recordPaymentSchema = z.object({
  amount: z.number()
    .positive('Payment amount must be positive')
    .refine(val => val <= 10000000, 'Payment amount exceeds maximum'),
  method: z.enum(['cash', 'check', 'bank_transfer', 'credit_card', 'stripe', 'paypal']),
  reference: z.string().max(200).optional(),
  paymentDate: z.string().datetime().optional(),
  notes: z.string().max(1000).optional()
});
```

#### 5.2 Business Rule Validation
**File**: `apps/backend/src/services/invoiceService.ts`

```typescript
private async validateInvoiceBusinessRules(input: CreateInvoiceInput): Promise<void> {
  // Client exists and active
  const client = await this.db.client.findUnique({
    where: { id: input.clientId }
  });
  
  if (!client) {
    throw new Error('Client not found');
  }

  // Project validation
  if (input.projectId) {
    const project = await this.db.project.findUnique({
      where: { id: input.projectId }
    });
    
    if (!project) {
      throw new Error('Project not found');
    }
    
    if (project.clientId !== input.clientId) {
      throw new Error('Project does not belong to selected client');
    }
  }

  // Quote validation
  if (input.quoteId) {
    const quote = await this.db.quote.findUnique({
      where: { id: input.quoteId }
    });
    
    if (!quote) {
      throw new Error('Quote not found');
    }
    
    if (quote.clientId !== input.clientId) {
      throw new Error('Quote does not belong to selected client');
    }
  }

  // Date validation
  const issueDate = new Date(input.issueDate);
  const dueDate = new Date(input.dueDate);
  const now = new Date();

  if (issueDate > now) {
    throw new Error('Issue date cannot be in the future');
  }

  if (dueDate <= issueDate) {
    throw new Error('Due date must be after issue date');
  }

  // Amount validation
  const total = input.items.reduce((sum, item) => 
    sum + (item.quantity * item.unitPrice), 0
  );

  if (total <= 0) {
    throw new Error('Invoice total must be greater than zero');
  }

  if (total > 10000000) {
    throw new Error('Invoice total exceeds maximum allowed amount');
  }
}
```

### Acceptance Criteria
- [ ] All required fields validated
- [ ] Date ranges validated correctly
- [ ] Business rules enforced
- [ ] Amount limits enforced
- [ ] Consistent error messages
- [ ] Validation errors include field names
- [ ] Frontend displays errors clearly

---

## Testing Requirements

### Unit Tests
```typescript
describe('StripeService', () => {
  it('should create payment intent', async () => {});
  it('should handle payment success webhook', async () => {});
  it('should handle payment failure webhook', async () => {});
  it('should process refund', async () => {});
});

describe('InvoiceService - Email', () => {
  it('should send invoice email', async () => {});
  it('should generate client token', async () => {});
  it('should update status on send', async () => {});
});

describe('InvoiceService - Tax', () => {
  it('should calculate multi-rate taxes', async () => {});
  it('should handle compound taxes', async () => {});
  it('should store tax breakdown', async () => {});
});
```

---

## Migration Plan

### Database Migration
```sql
-- Add new fields
ALTER TABLE "Invoice" ADD COLUMN "taxDetails" JSONB;
ALTER TABLE "Invoice" ADD COLUMN "jurisdiction" TEXT;
ALTER TABLE "Invoice" ADD COLUMN "compoundTax" BOOLEAN DEFAULT false;
ALTER TABLE "Invoice" ADD COLUMN "clientToken" TEXT UNIQUE;
ALTER TABLE "Invoice" ADD COLUMN "tokenExpiresAt" TIMESTAMP;
ALTER TABLE "Invoice" ADD COLUMN "sentAt" TIMESTAMP;

-- Add indexes
CREATE INDEX "idx_invoice_client_token" ON "Invoice"("clientToken");
CREATE INDEX "idx_invoice_stripe_payment_intent" ON "Invoice"("stripePaymentIntentId");
```

---

## Rollout Plan

1. **Phase 1: Payment Gateway** (Week 1-2)
   - Stripe integration
   - Webhook handling
   - Frontend payment form
   - Testing

2. **Phase 2: Email System** (Week 1)
   - Email templates
   - Send functionality
   - Delivery tracking

3. **Phase 3: Tax Enhancement** (Week 1)
   - Multi-rate calculation
   - Tax breakdown storage
   - Frontend display

4. **Phase 4: Quote Conversion** (Week 1)
   - Conversion endpoint
   - Data mapping
   - Testing

5. **Phase 5: Validation** (Week 1)
   - Comprehensive schemas
   - Business rules
   - Error handling

---

## Success Metrics

- [ ] All 5 partially implemented features completed
- [ ] Payment success rate > 95%
- [ ] Email delivery rate > 95%
- [ ] Zero payment processing errors
- [ ] Unit test coverage > 80%
- [ ] Integration test coverage > 70%
- [ ] All acceptance criteria met

---

**Document Status**: Ready for Implementation  
**Estimated Effort**: 6 weeks  
**Team Size**: 2-3 developers
