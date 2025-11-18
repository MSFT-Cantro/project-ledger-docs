# Email Notification System Specification

**Date:** October 15, 2025  
**Version:** 1.0  
**Status:** Phase 4 Foundation (Part of Change Orders)  
**Priority:** HIGH  
**Dependencies:** Change Orders Phase 4, Quotes, Invoices, Auth  

---

## 1. Overview

A general-purpose email notification system that provides a unified infrastructure for sending transactional emails across all ProjectLedger modules (Quotes, Invoices, Change Orders, Authentication, Reports, etc.).

### Key Goals
- **Reusability**: Single system for all email types
- **Reliability**: Queue-based with retry logic
- **Flexibility**: Support multiple email providers
- **Maintainability**: Template management UI
- **Analytics**: Track delivery, opens, clicks
- **Professional**: Consistent branding across all emails
- **Cost-Effective**: Start with SendGrid free tier (100 emails/day), upgrade only when needed

---

## 2. System Architecture

### 2.1 High-Level Components

```
┌─────────────────────────────────────────────────────────────┐
│                     Application Layer                        │
│  (Quotes, Invoices, Change Orders, Auth, Reports)           │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                   Email Service API                          │
│  • sendEmail(template, data, recipient)                     │
│  • sendBulkEmail(template, recipients[])                    │
│  • trackEmail(emailId)                                       │
│  • retryFailedEmail(emailId)                                 │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                   Email Queue System                         │
│  • Bull/BullMQ Redis-based queue                            │
│  • Retry logic with exponential backoff                     │
│  • Priority levels (high, normal, low)                      │
│  • Rate limiting per provider                               │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                 Template Engine                              │
│  • Handlebars/Mjml for HTML emails                          │
│  • Variable interpolation {{user.name}}                     │
│  • Conditional rendering {{#if premium}}                    │
│  • Partials for headers/footers                             │
│  • Preview mode for testing                                 │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│              Email Provider Abstraction                      │
│  • SendGrid (primary)                                        │
│  • AWS SES (backup)                                          │
│  • SMTP (development/self-hosted)                           │
│  • Mailgun (optional)                                        │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                Email Tracking & Analytics                    │
│  • Delivery status (sent, failed, bounced)                  │
│  • Open tracking (pixel-based)                              │
│  • Click tracking (link wrapping)                           │
│  • Unsubscribe management                                   │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Database Schema

### 3.1 Email Tables

```prisma
model EmailTemplate {
  id              Int                 @id @default(autoincrement())
  name            String              @unique // e.g., "quote_sent", "invoice_reminder"
  subject         String
  htmlBody        String              @db.Text // HTML template with {{variables}}
  textBody        String?             @db.Text // Plain text version
  description     String?
  category        EmailCategory
  
  // Metadata
  isActive        Boolean             @default(true)
  version         Int                 @default(1)
  createdAt       DateTime            @default(now())
  updatedAt       DateTime            @updatedAt
  createdById     Int
  
  // Relations
  createdBy       User                @relation(fields: [createdById], references: [id])
  emails          Email[]
  
  @@index([category])
  @@index([isActive])
}

model Email {
  id              Int                 @id @default(autoincrement())
  
  // Template & Content
  templateId      Int?
  subject         String
  htmlBody        String              @db.Text
  textBody        String?             @db.Text
  
  // Recipients
  to              String              // Email address
  toName          String?
  cc              String?             // Comma-separated
  bcc             String?             // Comma-separated
  replyTo         String?
  
  // Delivery
  status          EmailStatus
  provider        EmailProvider?
  providerMessageId String?           @unique
  
  // Tracking
  sentAt          DateTime?
  deliveredAt     DateTime?
  openedAt        DateTime?
  clickedAt       DateTime?
  bouncedAt       DateTime?
  failedAt        DateTime?
  
  // Retry Logic
  attemptCount    Int                 @default(0)
  lastAttemptAt   DateTime?
  nextRetryAt     DateTime?
  errorMessage    String?             @db.Text
  
  // Metadata
  priority        EmailPriority       @default(NORMAL)
  metadata        Json?               // Custom data for tracking
  createdAt       DateTime            @default(now())
  updatedAt       DateTime            @updatedAt
  
  // Relations
  template        EmailTemplate?      @relation(fields: [templateId], references: [id])
  attachments     EmailAttachment[]
  events          EmailEvent[]
  
  @@index([status])
  @@index([to])
  @@index([sentAt])
  @@index([nextRetryAt])
  @@index([provider])
}

model EmailAttachment {
  id              Int                 @id @default(autoincrement())
  emailId         Int
  filename        String
  contentType     String
  size            Int                 // bytes
  url             String?             // Cloud storage URL
  content         Bytes?              // Or store inline (for small files)
  
  createdAt       DateTime            @default(now())
  
  email           Email               @relation(fields: [emailId], references: [id], onDelete: Cascade)
  
  @@index([emailId])
}

model EmailEvent {
  id              Int                 @id @default(autoincrement())
  emailId         Int
  eventType       EmailEventType      // opened, clicked, bounced, delivered, etc.
  eventData       Json?               // Additional event details
  userAgent       String?
  ipAddress       String?
  timestamp       DateTime            @default(now())
  
  email           Email               @relation(fields: [emailId], references: [id], onDelete: Cascade)
  
  @@index([emailId])
  @@index([eventType])
  @@index([timestamp])
}

enum EmailCategory {
  QUOTE              // Quote-related emails
  INVOICE            // Invoice-related emails
  CHANGE_ORDER       // Change order emails
  AUTHENTICATION     // Login, password reset, etc.
  NOTIFICATION       // General notifications
  REPORT             // Scheduled reports
  MARKETING          // Marketing emails (future)
}

enum EmailStatus {
  QUEUED             // Waiting in queue
  SENDING            // Currently being sent
  SENT               // Successfully sent
  DELIVERED          // Confirmed delivered
  OPENED             // Recipient opened email
  CLICKED            // Recipient clicked link
  BOUNCED            // Email bounced
  FAILED             // Send failed
  CANCELLED          // Manually cancelled
}

enum EmailProvider {
  SENDGRID
  AWS_SES
  SMTP
  MAILGUN
}

enum EmailPriority {
  HIGH               // Send immediately (password reset, urgent)
  NORMAL             // Standard queue
  LOW                // Can be batched (reports, marketing)
}

enum EmailEventType {
  QUEUED
  SENT
  DELIVERED
  OPENED
  CLICKED
  BOUNCED
  COMPLAINED         // Marked as spam
  UNSUBSCRIBED
  FAILED
}
```

---

## 4. API Design

### 4.1 Email Service Interface

```typescript
// Core email service
interface EmailService {
  // Send single email
  sendEmail(options: SendEmailOptions): Promise<Email>;
  
  // Send bulk emails
  sendBulkEmail(options: BulkEmailOptions): Promise<Email[]>;
  
  // Send using template
  sendTemplateEmail(options: TemplateEmailOptions): Promise<Email>;
  
  // Retry failed email
  retryEmail(emailId: string): Promise<Email>;
  
  // Get email status
  getEmailStatus(emailId: string): Promise<EmailStatus>;
  
  // Cancel queued email
  cancelEmail(emailId: string): Promise<void>;
}

interface SendEmailOptions {
  to: string | string[];
  subject: string;
  html?: string;
  text?: string;
  cc?: string[];
  bcc?: string[];
  replyTo?: string;
  attachments?: EmailAttachment[];
  priority?: EmailPriority;
  metadata?: Record<string, any>;
}

interface TemplateEmailOptions {
  templateName: string;
  to: string | string[];
  data: Record<string, any>; // Variables for template
  cc?: string[];
  bcc?: string[];
  replyTo?: string;
  attachments?: EmailAttachment[];
  priority?: EmailPriority;
}

interface BulkEmailOptions {
  templateName: string;
  recipients: Array<{
    email: string;
    name?: string;
    data: Record<string, any>;
  }>;
  priority?: EmailPriority;
}
```

### 4.2 REST API Endpoints

```typescript
// Email Management
POST   /api/emails                    // Send email
POST   /api/emails/bulk               // Send bulk emails
POST   /api/emails/template           // Send template email
GET    /api/emails                    // List emails (with filters)
GET    /api/emails/:id                // Get email details
POST   /api/emails/:id/retry          // Retry failed email
DELETE /api/emails/:id                // Cancel queued email

// Template Management
GET    /api/email-templates           // List templates
GET    /api/email-templates/:id       // Get template
POST   /api/email-templates           // Create template
PUT    /api/email-templates/:id       // Update template
DELETE /api/email-templates/:id       // Delete template
POST   /api/email-templates/:id/preview  // Preview with sample data

// Analytics
GET    /api/emails/stats              // Email statistics
GET    /api/emails/:id/events         // Email event history

// Webhooks (for provider callbacks)
POST   /api/webhooks/sendgrid         // SendGrid webhook
POST   /api/webhooks/ses              // AWS SES webhook
```

---

## 5. Email Templates

### 5.1 Template Structure

```typescript
interface EmailTemplateData {
  // Common variables available in all templates
  user: {
    name: string;
    email: string;
  };
  company: {
    name: string;
    logo: string;
    address: string;
    phone: string;
    email: string;
  };
  
  // Template-specific data
  [key: string]: any;
}
```

### 5.2 Standard Templates

#### Quote Templates
1. **quote_sent** - Send quote to client
2. **quote_reminder** - Remind client about pending quote
3. **quote_accepted** - Confirmation of quote acceptance
4. **quote_declined** - Notification of quote decline
5. **quote_expired** - Quote expiration notice

#### Invoice Templates
6. **invoice_sent** - Send invoice to client
7. **invoice_reminder** - Payment reminder
8. **invoice_overdue** - Overdue payment notice
9. **invoice_paid** - Payment confirmation
10. **receipt** - Payment receipt

#### Change Order Templates
11. **change_order_sent** - Send change order for approval
12. **change_order_approved** - Change order approval confirmation
13. **change_order_declined** - Change order decline notification
14. **change_order_reminder** - Approval reminder

#### Authentication Templates
15. **welcome** - Welcome new user
16. **password_reset** - Password reset link
17. **email_verification** - Verify email address
18. **login_alert** - Suspicious login notification

#### Notification Templates
19. **project_update** - Project milestone notification
20. **report_ready** - Scheduled report available
21. **system_alert** - Important system notification

### 5.3 Example Template (Handlebars + MJML)

```handlebars
<!-- change_order_sent.mjml -->
<mjml>
  <mj-head>
    <mj-title>{{company.name}} - Change Order Approval Required</mj-title>
    <mj-preview>Change Order #{{changeOrder.number}} requires your approval</mj-preview>
  </mj-head>
  <mj-body>
    <!-- Header -->
    <mj-section background-color="#2563eb">
      <mj-column>
        <mj-image src="{{company.logo}}" alt="{{company.name}}" width="150px" />
      </mj-column>
    </mj-section>
    
    <!-- Content -->
    <mj-section>
      <mj-column>
        <mj-text font-size="24px" font-weight="bold">
          Change Order Approval Required
        </mj-text>
        
        <mj-text font-size="16px" line-height="24px">
          Hello {{client.name}},
        </mj-text>
        
        <mj-text font-size="16px" line-height="24px">
          We have prepared Change Order #{{changeOrder.number}} for your review and approval.
        </mj-text>
        
        <mj-text font-size="16px" line-height="24px">
          <strong>Project:</strong> {{changeOrder.quote.project.name}}<br/>
          <strong>Original Quote:</strong> ${{formatCurrency changeOrder.originalTotal}}<br/>
          <strong>Change Amount:</strong> 
          <span style="color: {{#if changeOrder.deltaPositive}}#ef4444{{else}}#10b981{{/if}}">
            {{#if changeOrder.deltaPositive}}+{{/if}}${{formatCurrency changeOrder.deltaAmount}}
          </span><br/>
          <strong>New Total:</strong> ${{formatCurrency changeOrder.newTotal}}
        </mj-text>
        
        <!-- CTA Button -->
        <mj-button background-color="#2563eb" href="{{approvalUrl}}">
          Review & Approve Change Order
        </mj-button>
        
        <mj-text font-size="14px" color="#666666">
          This link will expire on {{formatDate changeOrder.validUntil}}
        </mj-text>
      </mj-column>
    </mj-section>
    
    <!-- Footer -->
    <mj-section background-color="#f3f4f6">
      <mj-column>
        <mj-text font-size="12px" color="#666666" align="center">
          {{company.name}}<br/>
          {{company.address}}<br/>
          {{company.phone}} | {{company.email}}
        </mj-text>
        
        <mj-text font-size="12px" color="#666666" align="center">
          <a href="{{unsubscribeUrl}}" style="color: #666666;">Unsubscribe</a> | 
          <a href="{{preferencesUrl}}" style="color: #666666;">Email Preferences</a>
        </mj-text>
      </mj-column>
    </mj-section>
  </mj-body>
</mjml>
```

---

## 6. SendGrid Integration

### 6.1 Why SendGrid?

SendGrid is chosen as the primary email provider for the following reasons:

1. **Reliability**: 99.9% uptime SLA, trusted by major companies
2. **Deliverability**: Industry-leading delivery rates with reputation monitoring
3. **Features**: Built-in templates, analytics, event webhooks
4. **Scalability**: Handles from 100 to millions of emails per month
5. **Developer Experience**: Excellent Node.js SDK and comprehensive documentation
6. **Cost-Effective**: **Free tier (100 emails/day forever)** for development, then pay-as-you-grow
7. **Compliance**: Built-in unsubscribe handling, GDPR/CAN-SPAM compliant
8. **No Credit Card Required**: Start developing immediately with free tier

### 6.2 SendGrid Setup

#### Account Configuration
1. **Create SendGrid Account**: https://signup.sendgrid.com/
2. **Verify Domain**: 
   - Add DNS records (CNAME) for domain authentication
   - Enables "on behalf of" removal from emails
3. **Create API Key**:
   - Mail Send: Full Access
   - Mail Settings: Read Access (for tracking)
   - Tracking: Full Access (for webhooks)
4. **Configure Sender Identity**:
   - Single Sender Verification: noreply@projectledger.ca
   - Domain Authentication: projectledger.ca

#### DNS Records Required
```dns
# SPF Record
TXT @ "v=spf1 include:sendgrid.net ~all"

# DKIM Records (from SendGrid console)
CNAME s1._domainkey CNAME s1.domainkey.u12345678.wl123.sendgrid.net
CNAME s2._domainkey CNAME s2.domainkey.u12345678.wl123.sendgrid.net

# DMARC Record
TXT _dmarc "v=DMARC1; p=quarantine; rua=mailto:dmarc@projectledger.ca"
```

### 6.3 SendGrid API Integration

#### Install SDK
```bash
npm install @sendgrid/mail @sendgrid/client
```

#### SendGrid Service Implementation

```typescript
// apps/backend/src/services/email/providers/sendgrid.provider.ts
import sgMail from '@sendgrid/mail';
import sgClient from '@sendgrid/client';
import { EmailProvider, SendEmailResult } from '../email.types';

export class SendGridProvider implements EmailProvider {
  private initialized = false;

  constructor(private apiKey: string) {
    if (!apiKey) {
      throw new Error('SendGrid API key is required');
    }
  }

  async initialize(): Promise<void> {
    sgMail.setApiKey(this.apiKey);
    sgClient.setApiKey(this.apiKey);
    this.initialized = true;
    
    // Test connection
    await this.verifyConnection();
  }

  async verifyConnection(): Promise<boolean> {
    try {
      const request = {
        url: '/v3/user/account',
        method: 'GET' as const,
      };
      await sgClient.request(request);
      return true;
    } catch (error) {
      console.error('SendGrid connection failed:', error);
      throw new Error('Failed to connect to SendGrid');
    }
  }

  async sendEmail(options: {
    to: string | string[];
    subject: string;
    html: string;
    text?: string;
    from: { email: string; name: string };
    cc?: string[];
    bcc?: string[];
    replyTo?: string;
    attachments?: Array<{
      filename: string;
      content: string; // Base64 encoded
      contentType: string;
    }>;
    customArgs?: Record<string, string>; // For tracking
    categories?: string[]; // For analytics grouping
  }): Promise<SendEmailResult> {
    if (!this.initialized) {
      throw new Error('SendGrid provider not initialized');
    }

    try {
      const msg = {
        to: options.to,
        from: options.from,
        subject: options.subject,
        html: options.html,
        text: options.text || this.htmlToText(options.html),
        cc: options.cc,
        bcc: options.bcc,
        replyTo: options.replyTo,
        attachments: options.attachments?.map(att => ({
          filename: att.filename,
          content: att.content,
          type: att.contentType,
          disposition: 'attachment',
        })),
        customArgs: options.customArgs,
        categories: options.categories,
        trackingSettings: {
          clickTracking: { enable: true },
          openTracking: { enable: true },
          subscriptionTracking: { enable: true },
        },
      };

      const response = await sgMail.send(msg);
      
      return {
        success: true,
        messageId: response[0].headers['x-message-id'],
        provider: 'SENDGRID',
      };
    } catch (error: any) {
      console.error('SendGrid send error:', error);
      
      return {
        success: false,
        error: error.message || 'Failed to send email',
        errorCode: error.code,
      };
    }
  }

  async sendBulk(emails: Array<{
    to: string;
    subject: string;
    html: string;
    text?: string;
    customArgs?: Record<string, string>;
  }>): Promise<SendEmailResult[]> {
    // SendGrid supports batch sending with personalizations
    const msg = {
      from: {
        email: process.env.EMAIL_FROM_ADDRESS!,
        name: process.env.EMAIL_FROM_NAME!,
      },
      personalizations: emails.map(email => ({
        to: [{ email: email.to }],
        subject: email.subject,
        customArgs: email.customArgs,
      })),
      content: [
        {
          type: 'text/html',
          value: emails[0].html, // Use first email's content as base
        },
      ],
    };

    try {
      const response = await sgMail.send(msg as any);
      
      return emails.map((_, index) => ({
        success: true,
        messageId: response[index]?.headers['x-message-id'] || `bulk-${index}`,
        provider: 'SENDGRID',
      }));
    } catch (error: any) {
      console.error('SendGrid bulk send error:', error);
      
      return emails.map(() => ({
        success: false,
        error: error.message,
      }));
    }
  }

  private htmlToText(html: string): string {
    // Simple HTML to text conversion
    return html
      .replace(/<style[^>]*>.*<\/style>/gm, '')
      .replace(/<script[^>]*>.*<\/script>/gm, '')
      .replace(/<[^>]+>/gm, '')
      .replace(/\s+/g, ' ')
      .trim();
  }
}
```

### 6.4 SendGrid Event Webhooks

SendGrid can send real-time notifications for email events (delivered, opened, clicked, bounced, etc.)

#### Webhook Handler

```typescript
// apps/backend/src/routes/webhooks/sendgrid.ts
import { Router } from 'express';
import crypto from 'crypto';
import { prisma } from '@/lib/prisma';

const router = Router();

// Verify SendGrid webhook signature
function verifyWebhookSignature(
  payload: string,
  signature: string,
  timestamp: string
): boolean {
  const publicKey = process.env.SENDGRID_WEBHOOK_PUBLIC_KEY!;
  
  const data = timestamp + payload;
  const hash = crypto
    .createHmac('sha256', publicKey)
    .update(data)
    .digest('base64');
  
  return hash === signature;
}

router.post('/webhooks/sendgrid', async (req, res) => {
  try {
    const signature = req.headers['x-twilio-email-event-webhook-signature'] as string;
    const timestamp = req.headers['x-twilio-email-event-webhook-timestamp'] as string;
    
    // Verify webhook authenticity
    const payload = JSON.stringify(req.body);
    if (!verifyWebhookSignature(payload, signature, timestamp)) {
      return res.status(401).json({ error: 'Invalid signature' });
    }

    // Process events
    const events = req.body as SendGridEvent[];
    
    for (const event of events) {
      await processWebhookEvent(event);
    }

    res.status(200).json({ received: events.length });
  } catch (error) {
    console.error('SendGrid webhook error:', error);
    res.status(500).json({ error: 'Webhook processing failed' });
  }
});

async function processWebhookEvent(event: SendGridEvent): Promise<void> {
  const messageId = event.sg_message_id;
  
  // Find email by SendGrid message ID
  const email = await prisma.email.findUnique({
    where: { providerMessageId: messageId },
  });

  if (!email) {
    console.warn(`Email not found for message ID: ${messageId}`);
    return;
  }

  // Create event record
  await prisma.emailEvent.create({
    data: {
      emailId: email.id,
      eventType: mapEventType(event.event),
      eventData: event,
      userAgent: event.useragent,
      ipAddress: event.ip,
      timestamp: new Date(event.timestamp * 1000),
    },
  });

  // Update email status
  await updateEmailStatus(email.id, event.event, event.timestamp);
}

function mapEventType(sendgridEvent: string): string {
  const mapping: Record<string, string> = {
    'processed': 'QUEUED',
    'delivered': 'DELIVERED',
    'open': 'OPENED',
    'click': 'CLICKED',
    'bounce': 'BOUNCED',
    'dropped': 'FAILED',
    'spamreport': 'COMPLAINED',
    'unsubscribe': 'UNSUBSCRIBED',
  };
  return mapping[sendgridEvent] || sendgridEvent.toUpperCase();
}

async function updateEmailStatus(
  emailId: number,
  event: string,
  timestamp: number
): Promise<void> {
  const updates: any = {};
  const date = new Date(timestamp * 1000);

  switch (event) {
    case 'delivered':
      updates.status = 'DELIVERED';
      updates.deliveredAt = date;
      break;
    case 'open':
      updates.status = 'OPENED';
      updates.openedAt = date;
      break;
    case 'click':
      updates.status = 'CLICKED';
      updates.clickedAt = date;
      break;
    case 'bounce':
    case 'dropped':
      updates.status = 'BOUNCED';
      updates.bouncedAt = date;
      break;
  }

  if (Object.keys(updates).length > 0) {
    await prisma.email.update({
      where: { id: emailId },
      data: updates,
    });
  }
}

interface SendGridEvent {
  email: string;
  timestamp: number;
  event: string;
  sg_message_id: string;
  sg_event_id: string;
  useragent?: string;
  ip?: string;
  url?: string;
  category?: string[];
  [key: string]: any;
}

export default router;
```

#### Configure Webhook in SendGrid
1. Go to Settings → Mail Settings → Event Webhook
2. Enable webhook
3. HTTP POST URL: `https://api.projectledger.ca/api/webhooks/sendgrid`
4. Select events: Processed, Delivered, Open, Click, Bounce, Dropped, Spam Report, Unsubscribe
5. Enable Signature Verification
6. Save public key to `.env` as `SENDGRID_WEBHOOK_PUBLIC_KEY`

### 6.5 SendGrid Templates (Dynamic Templates)

SendGrid supports dynamic templates with Handlebars syntax directly in their UI.

#### Option 1: Use SendGrid's Template UI
- Create templates in SendGrid dashboard
- Reference by template ID
- No need to store HTML in database

```typescript
await sgMail.send({
  to: 'client@example.com',
  from: 'noreply@projectledger.ca',
  templateId: 'd-1234567890abcdef', // SendGrid template ID
  dynamicTemplateData: {
    client: {
      name: 'John Doe',
    },
    changeOrder: {
      number: 'CO-2025-001',
      total: 5000,
    },
  },
});
```

#### Option 2: Store Templates in Database (Recommended for ProjectLedger)
- More flexibility and version control
- Templates can be edited by admins without SendGrid access
- Use our template engine (Handlebars + MJML)

### 6.6 Rate Limiting

SendGrid rate limits vary by plan:
- **Free**: 100 emails/day
- **Essentials**: 40,000 emails/month ($19.95/mo)
- **Pro**: 100,000 emails/month ($89.95/mo)

Implement rate limiting in queue:

```typescript
// apps/backend/src/services/email/queue.ts
import Queue from 'bull';

const emailQueue = new Queue('email', process.env.REDIS_URL!, {
  limiter: {
    max: 100, // Max 100 jobs
    duration: 1000, // per second
  },
});

// Adjust based on SendGrid plan
const rateLimits = {
  free: { max: 100, duration: 86400000 }, // 100/day
  essentials: { max: 40000, duration: 2592000000 }, // 40k/month
  pro: { max: 100000, duration: 2592000000 }, // 100k/month
};
```

## 7. Implementation Plan

### Phase 4A: Foundation (Days 1-2)
- [x] Choose SendGrid as email provider
- [ ] Set up SendGrid account and verify domain
- [ ] Configure DNS records (SPF, DKIM, DMARC)
- [ ] Install SendGrid SDK (`@sendgrid/mail`)
- [ ] Database schema migration for email tables
- [ ] Create SendGrid provider class with connection verification
- [ ] Email service abstraction layer
- [ ] Basic send email functionality with error handling
- [ ] Email queue setup with Bull/BullMQ
- [ ] Configure rate limiting based on SendGrid plan

### Phase 4B: Template System (Days 3-4)
- [ ] Template engine integration (Handlebars + MJML)
- [ ] MJML to HTML compiler setup
- [ ] Template management API (CRUD operations)
- [ ] Template preview functionality with sample data
- [ ] Create 5 core templates:
  - [ ] `change_order_sent` - Change order approval request
  - [ ] `quote_sent` - Quote sent to client
  - [ ] `invoice_sent` - Invoice sent to client
  - [ ] `password_reset` - Password reset email
  - [ ] `welcome` - Welcome new user
- [ ] Variable interpolation and Handlebars helpers
- [ ] Test templates in multiple email clients (Gmail, Outlook, Apple Mail)

### Phase 4C: Advanced Features (Days 5-6)
- [ ] Configure SendGrid Event Webhook endpoint
- [ ] Implement webhook signature verification
- [ ] Webhook handler for email events (delivered, opened, clicked, bounced)
- [ ] Retry logic with exponential backoff for failed emails
- [ ] Email tracking integration (opens, clicks via SendGrid)
- [ ] Attachment support with Base64 encoding
- [ ] SendGrid category/custom args for analytics
- [ ] Email analytics dashboard (delivery rates, open rates, click rates)
- [ ] Bounce handling and suppression list management

### Phase 4D: UI & Testing (Day 7)
- [ ] Template management UI (admin panel)
  - [ ] List all templates with preview thumbnails
  - [ ] Create/edit templates with live MJML preview
  - [ ] Test send functionality with sample data
- [ ] Email history/logs page
  - [ ] Filterable table (status, date, recipient, template)
  - [ ] Event timeline for each email
  - [ ] Retry failed emails from UI
- [ ] Email preview in UI (before sending)
- [ ] Unit tests for email service
  - [ ] SendGrid provider tests (mocked)
  - [ ] Template rendering tests
  - [ ] Queue processing tests
- [ ] Integration tests with SendGrid test mode
- [ ] End-to-end test for change order approval email flow
- [ ] Load testing (1000 emails) with SendGrid sandbox

---

## 8. SendGrid Configuration Guide

### 8.1 Environment Variables

```bash
# SendGrid Configuration (Primary Provider)
EMAIL_PROVIDER=sendgrid
SENDGRID_API_KEY=SG.xxxxxxxxxxxxxxxxxxxxx
SENDGRID_WEBHOOK_PUBLIC_KEY=MFkwEwYH...  # For webhook signature verification
SENDGRID_PLAN=free                       # free, essentials, pro (affects rate limits)

# Email Settings
EMAIL_FROM_ADDRESS=noreply@projectledger.ca
EMAIL_FROM_NAME=Project Ledger
EMAIL_REPLY_TO=support@projectledger.ca

# SMTP (Development/Testing - optional fallback)
SMTP_HOST=smtp.mailtrap.io
SMTP_PORT=2525
SMTP_USER=xxxxx
SMTP_PASS=xxxxx
SMTP_SECURE=false

# Redis (for Bull queue)
REDIS_URL=redis://localhost:6379

# Email Queue Configuration
EMAIL_QUEUE_CONCURRENCY=10               # Parallel processing
EMAIL_RETRY_ATTEMPTS=3                   # Retry failed emails 3 times
EMAIL_RETRY_DELAY=60000                  # Wait 1 minute between retries
EMAIL_RETRY_BACKOFF=exponential          # exponential or fixed

# Tracking & Analytics
EMAIL_TRACKING_ENABLED=true
EMAIL_TRACKING_DOMAIN=track.projectledger.ca
EMAIL_OPEN_TRACKING=true
EMAIL_CLICK_TRACKING=true

# Application URLs
APP_URL=https://app.projectledger.ca
API_URL=https://api.projectledger.ca
```

### 8.2 SendGrid Account Setup Checklist

#### 1. Create Account
- Sign up at https://signup.sendgrid.com/
- **Use Free tier initially** (no credit card required):
  - **Free**: 100 emails/day forever (3,000/month)
    - Perfect for development, testing, and initial production
    - No credit card required
    - Includes all core features (API, tracking, webhooks)
    - Monitor usage and upgrade when you have actual users
  - **Upgrade to Essentials** ($19.95/mo, 40,000/month) when:
    - You exceed 80 emails/day consistently
    - You have paying customers
    - You need dedicated IP or email validation
  - **Pro**: 100,000/month ($89.95) - For mature product with high volume

#### 2. Domain Authentication
- Navigate to Settings → Sender Authentication → Authenticate Your Domain
- Enter `projectledger.ca`
- Add DNS records provided by SendGrid to domain registrar:
  ```
  Type: CNAME
  Host: s1._domainkey
  Value: s1.domainkey.u12345.wl234.sendgrid.net
  
  Type: CNAME
  Host: s2._domainkey
  Value: s2.domainkey.u12345.wl234.sendgrid.net
  ```
- Wait for DNS propagation (can take up to 48 hours, usually 1-2 hours)
- Verify in SendGrid console

#### 3. Single Sender Verification
- Settings → Sender Authentication → Verify a Single Sender
- Add: noreply@projectledger.ca
- Verify email inbox for confirmation link
- This allows sending before full domain authentication

#### 4. Create API Key
- Settings → API Keys → Create API Key
- Name: `ProjectLedger Production`
- Permissions:
  - Mail Send: **Full Access**
  - Mail Settings: **Read Access**
  - Tracking: **Full Access**
  - Suppressions: **Full Access**
- Copy key and store in `.env` as `SENDGRID_API_KEY`
- **Important**: Key is only shown once, store securely

#### 5. Configure Event Webhook
- Settings → Mail Settings → Event Webhook
- Status: **Enabled**
- HTTP POST URL: `https://api.projectledger.ca/api/webhooks/sendgrid`
- Select all events:
  - ☑ Processed
  - ☑ Delivered
  - ☑ Open
  - ☑ Click
  - ☑ Bounce
  - ☑ Dropped
  - ☑ Spam Report
  - ☑ Unsubscribe
- Enable **Signature Verification**
- Copy Verification Key and store as `SENDGRID_WEBHOOK_PUBLIC_KEY`

#### 6. Enable Click & Open Tracking
- Settings → Tracking
- Click Tracking: **Enabled**
- Open Tracking: **Enabled**
- Subscription Tracking: **Enabled** (adds unsubscribe link)

#### 7. Configure Suppression Management
- Suppressions → Create Suppression Group
- Name: "General Notifications"
- Description: "Transactional emails from Project Ledger"
- This allows users to unsubscribe from specific email types

### 8.3 DNS Configuration

Add these records to your DNS provider (e.g., Cloudflare, Route53):

```dns
# SPF Record (allows SendGrid to send on your behalf)
Type: TXT
Name: @
Value: v=spf1 include:sendgrid.net ~all

# DKIM Records (provided by SendGrid after domain authentication)
Type: CNAME
Name: s1._domainkey
Value: s1.domainkey.u12345678.wl123.sendgrid.net

Type: CNAME
Name: s2._domainkey
Value: s2.domainkey.u12345678.wl123.sendgrid.net

# DMARC Record (email authentication policy)
Type: TXT
Name: _dmarc
Value: v=DMARC1; p=quarantine; rua=mailto:dmarc@projectledger.ca; pct=100

# Custom Tracking Domain (optional, for branded tracking links)
Type: CNAME
Name: track
Value: sendgrid.net
```

### 8.4 Rate Limiting Configuration

Configure based on SendGrid plan:

```typescript
// apps/backend/src/config/email.config.ts
export const emailConfig = {
  sendgrid: {
    rateLimits: {
      free: {
        maxPerDay: 100,
        maxPerSecond: 1,
      },
      essentials: {
        maxPerMonth: 40000,
        maxPerSecond: 10,
      },
      pro: {
        maxPerMonth: 100000,
        maxPerSecond: 20,
      },
    },
  },
  queue: {
    concurrency: process.env.SENDGRID_PLAN === 'free' ? 1 : 10,
    retryAttempts: 3,
    retryDelay: 60000, // 1 minute
    backoff: 'exponential' as const,
  },
};
```

---

## 9. Usage Examples

### 9.1 Send Change Order Email

```typescript
// In change-orders service
import { emailService } from '@/services/email';

async function sendChangeOrderForApproval(changeOrderId: string) {
  const changeOrder = await getChangeOrder(changeOrderId);
  const approvalToken = generateApprovalToken(changeOrderId);
  const approvalUrl = `${process.env.APP_URL}/change-orders/approve/${approvalToken}`;
  
  await emailService.sendTemplateEmail({
    templateName: 'change_order_sent',
    to: changeOrder.quote.client.email,
    data: {
      client: {
        name: changeOrder.quote.client.name,
        email: changeOrder.quote.client.email,
      },
      changeOrder: {
        number: changeOrder.number,
        originalTotal: changeOrder.originalTotal,
        deltaAmount: changeOrder.deltaAmount,
        deltaPositive: changeOrder.deltaAmount > 0,
        newTotal: changeOrder.newTotal,
        validUntil: changeOrder.validUntil,
        quote: {
          project: {
            name: changeOrder.quote.project.name,
          },
        },
      },
      approvalUrl,
    },
    priority: 'HIGH',
    metadata: {
      type: 'change_order_approval',
      changeOrderId,
    },
  });
  
  // Update change order status
  await updateChangeOrderStatus(changeOrderId, 'SENT');
}
```

### 8.2 Send Invoice Reminder

```typescript
// In invoices service
async function sendInvoiceReminder(invoiceId: string) {
  const invoice = await getInvoice(invoiceId);
  
  await emailService.sendTemplateEmail({
    templateName: 'invoice_reminder',
    to: invoice.client.email,
    data: {
      client: {
        name: invoice.client.name,
      },
      invoice: {
        number: invoice.number,
        total: invoice.total,
        dueDate: invoice.dueDate,
        daysOverdue: calculateDaysOverdue(invoice.dueDate),
      },
      paymentUrl: `${process.env.APP_URL}/invoices/${invoice.id}/pay`,
    },
    attachments: [
      {
        filename: `Invoice-${invoice.number}.pdf`,
        contentType: 'application/pdf',
        url: await generateInvoicePdf(invoiceId),
      },
    ],
  });
}
```

---

## 10. Testing Strategy

### 10.1 Unit Tests
- Email service methods
- Template rendering
- Provider abstraction
- Retry logic
- Tracking functionality

### 10.2 Integration Tests
- Queue processing
- Database operations
- Provider API calls (mocked)
- Webhook handling

### 10.3 End-to-End Tests
- Send email and verify delivery
- Track opens/clicks
- Retry failed emails
- Template preview

### 10.4 Manual Testing
- Test with Mailtrap (dev)
- Send real emails (staging)
- Verify formatting on different clients (Gmail, Outlook, mobile)
- Test unsubscribe flow

---

## 11. Monitoring & Observability

### 11.1 Metrics to Track
- **Volume**: Emails sent per hour/day
- **Success Rate**: Percentage of successfully delivered emails
- **Bounce Rate**: Percentage of bounced emails
- **Open Rate**: Percentage of opened emails (by category)
- **Click Rate**: Percentage of clicked links
- **Queue Length**: Number of emails waiting
- **Processing Time**: Time from queue to sent

### 11.2 Alerts
- Email send failures exceed threshold
- Queue length grows too large
- Provider API errors
- High bounce rate for specific domain

---

## 12. Security Considerations

### 12.1 Email Security
- **SPF Record**: Authorize sending servers
- **DKIM**: Sign emails cryptographically
- **DMARC**: Policy for handling failed authentication
- **TLS**: Encrypt email transmission

### 12.2 Data Protection
- Encrypt email content at rest
- Redact sensitive data from logs
- Implement unsubscribe mechanism (GDPR/CAN-SPAM)
- Rate limiting to prevent abuse
- Token expiration for approval links

---

## 13. Future Enhancements

### Phase 2 (Post-Launch)
- [ ] Email scheduling (send at specific time)
- [ ] A/B testing for templates
- [ ] Dynamic content personalization
- [ ] Email preview for different clients
- [ ] Bulk import/export of templates
- [ ] Multi-language support
- [ ] SMS notifications integration
- [ ] Push notification integration

### Phase 3 (Advanced)
- [ ] Marketing automation
- [ ] Drip campaigns
- [ ] Segmentation and targeting
- [ ] Advanced analytics dashboard
- [ ] AI-powered subject line optimization
- [ ] Bounce handling and list hygiene

---

## 14. Success Criteria

### Technical Requirements
- [ ] Email delivery rate > 99%
- [ ] Average send time < 5 seconds
- [ ] Queue processing handles 1000 emails/minute
- [ ] Zero data loss in queue
- [ ] Provider failover works automatically

### Business Requirements
- [ ] Reduce manual email sending time by 90%
- [ ] Professional, branded emails across all modules
- [ ] Client approval emails sent automatically
- [ ] Email tracking provides actionable insights
- [ ] Template management accessible to non-technical users

---

**Document Version:** 1.0  
**Created:** October 15, 2025  
**Owner:** Engineering Team  
**Related Documents:**
- [Change Orders Specification](../1.%20critical/SPEC_change_orders.md)
- Change Orders Approval Workflow (to be created)

**Next Actions:**
1. Review and approve architecture
2. Set up SendGrid account and test credentials
3. Begin Phase 4A implementation
4. Create email templates with marketing/design team
