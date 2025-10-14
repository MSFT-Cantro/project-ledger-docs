# Change Orders - Approval Workflow (Phase 4)

**Date:** October 14, 2025  
**Priority:** High  
**Status:** In Progress  
**Dependencies:** Phase 3 Complete  
**Estimated Timeline:** 4-5 days  
**Branch:** `feature/phase-4-change-order-approval-workflow`  
**PR:** [#19](https://github.com/MSFT-Cantro/project-ledger/pull/19)

---

## Overview

Implement the complete client-facing approval workflow for change orders, including secure token-based authentication, email notifications, and professional PDF generation. This enables clients to approve or decline change orders without logging into the system.

---

## Objectives

1. **Enable Client Self-Service** - Allow clients to approve/decline change orders via secure links
2. **Automate Communication** - Send professional email notifications at key workflow points
3. **Ensure Security** - Implement token-based authentication with expiration
4. **Generate Documents** - Create professional PDF change orders with branding
5. **Maintain Audit Trail** - Track all approval actions with timestamps and metadata

---

## Business Value

### Current Pain Points
- Manual approval process requires phone calls or in-person meetings
- Paper-based signatures slow down project timelines
- No automated notifications when change orders are ready for review
- Difficult to track approval status and history

### Expected Benefits
- **75% faster approval time** - From days to hours
- **Reduced admin overhead** - Automated email workflow
- **Better client experience** - Self-service approval from any device
- **Complete audit trail** - Legal compliance and transparency
- **Mobile-friendly** - Approve on-the-go from phone or tablet

---

## Features

### 4.1 Client Approval Interface (Public Pages)

**Requirements:**
- [ ] Create public approval page accessible without login
- [ ] Token-based authentication for secure access
- [ ] Display change order details and financial impact
- [ ] Approve/decline buttons with confirmation
- [ ] Responsive design for mobile approvals
- [ ] Success/failure feedback messages
- [ ] Expired token handling

**User Story:**
> As a **client**, I want to **review and approve change orders from a secure link** so that I can **quickly authorize project changes without logging in or scheduling meetings**.

**Acceptance Criteria:**
- Link format: `https://app.projectledger.com/approve/change-orders/{token}`
- Token valid for 30 days by default (configurable)
- Page shows: CO details, financial impact, original vs new totals
- Clear approve/decline buttons with confirmation dialogs
- Works on mobile devices (iOS/Android)
- Expired tokens show clear error message with contact info

**Technical Details:**
```typescript
// Public route (no auth required)
GET /approve/change-orders/:token

// Token validation
- Check token exists in database
- Verify not expired (validUntil > now)
- Verify not already used
- Load associated change order
```

**UI Wireframe:**
```
┌─────────────────────────────────────────┐
│  [Company Logo]                         │
│                                         │
│  Change Order Approval Required         │
│                                         │
│  CO-2025-004                           │
│  Website Redesign - Scope Addition     │
│                                         │
│  Original Total:    $15,000.00         │
│  Change Amount:     +$3,500.00         │
│  New Total:         $18,500.00         │
│                                         │
│  ┌─────────────────────────────────┐  │
│  │ Additional Features:            │  │
│  │ • Mobile responsive design      │  │
│  │ • Blog integration             │  │
│  │ • Contact form                 │  │
│  └─────────────────────────────────┘  │
│                                         │
│  [Decline]          [Approve ✓]        │
│                                         │
│  Questions? Contact: sales@company.com │
└─────────────────────────────────────────┘
```

---

### 4.2 Email Notification System

**Requirements:**
- [ ] Send change order to client via email
- [ ] Include approval link with secure token
- [ ] Professional HTML email templates
- [ ] Reminder emails for pending approvals
- [ ] Approval confirmation emails
- [ ] Decline notification emails
- [ ] Expiry warning emails

**Email Templates Needed:**

1. **Initial Send Email**
   - Subject: "Change Order CO-{number} - Approval Required"
   - Body: Summary + Approval link + Expiry date
   - CTA: "Review Change Order"

2. **Reminder Email (T-7 days before expiry)**
   - Subject: "Reminder: Change Order CO-{number} Expires in 7 Days"
   - Body: Gentle reminder + link
   - CTA: "Review Now"

3. **Reminder Email (T-2 days before expiry)**
   - Subject: "Urgent: Change Order CO-{number} Expires in 2 Days"
   - Body: Urgent tone + consequences of expiry
   - CTA: "Approve Now"

4. **Approval Confirmation (to client)**
   - Subject: "Change Order CO-{number} Approved - Thank You!"
   - Body: Confirmation + next steps + PDF attachment
   - CTA: "Download PDF"

5. **Approval Notification (to contractor)**
   - Subject: "Change Order CO-{number} Approved by Client"
   - Body: Client details + approval timestamp + next actions
   - CTA: "Execute Change Order"

6. **Decline Notification (to contractor)**
   - Subject: "Change Order CO-{number} Declined by Client"
   - Body: Decline reason (if provided) + client contact info
   - CTA: "Contact Client"

7. **Expired Notification (to contractor)**
   - Subject: "Change Order CO-{number} Expired"
   - Body: Expiry notice + option to reissue
   - CTA: "Reissue Change Order"

**Email Service Options:**
- SendGrid (recommended)
- AWS SES
- Mailgun
- SMTP relay

**Technical Implementation:**
```typescript
interface EmailTemplate {
  templateId: string;
  subject: string;
  to: string[];
  cc?: string[];
  bcc?: string[];
  data: {
    changeOrderNumber: string;
    clientName: string;
    approvalUrl: string;
    expiryDate: string;
    deltaAmount: number;
    // ... other dynamic data
  };
  attachments?: {
    filename: string;
    content: Buffer;
    contentType: string;
  }[];
}

async function sendChangeOrderEmail(
  type: 'send' | 'reminder' | 'approved' | 'declined' | 'expired',
  changeOrder: ChangeOrder
): Promise<void> {
  const template = getEmailTemplate(type);
  await emailService.send(template);
}
```

---

### 4.3 Approval Token Management

**Requirements:**
- [ ] Generate secure, unique approval tokens
- [ ] Token expiration logic (default: 30 days)
- [ ] Single-use token validation
- [ ] Token revocation support
- [ ] Audit trail for token usage

**Token Structure:**
```typescript
interface ApprovalToken {
  id: string;                    // UUID
  changeOrderId: number;         // Foreign key
  token: string;                 // Secure random token (32 chars)
  createdAt: Date;              
  expiresAt: Date;               // Default: now + 30 days
  usedAt?: Date;                 // Set when token is used
  revokedAt?: Date;              // Set if token is revoked
  ipAddress?: string;            // IP that used the token
  userAgent?: string;            // Browser/device info
}
```

**Token Generation:**
```typescript
import crypto from 'crypto';

function generateApprovalToken(): string {
  return crypto.randomBytes(32).toString('hex');
}

async function createApprovalToken(
  changeOrderId: number,
  expiryDays: number = 30
): Promise<ApprovalToken> {
  const token = generateApprovalToken();
  const expiresAt = new Date();
  expiresAt.setDate(expiresAt.getDate() + expiryDays);
  
  return await prisma.approvalToken.create({
    data: {
      changeOrderId,
      token,
      expiresAt
    }
  });
}
```

**Token Validation:**
```typescript
async function validateToken(token: string): Promise<{
  valid: boolean;
  changeOrderId?: number;
  error?: string;
}> {
  const approvalToken = await prisma.approvalToken.findUnique({
    where: { token }
  });
  
  if (!approvalToken) {
    return { valid: false, error: 'Invalid token' };
  }
  
  if (approvalToken.usedAt) {
    return { valid: false, error: 'Token already used' };
  }
  
  if (approvalToken.revokedAt) {
    return { valid: false, error: 'Token has been revoked' };
  }
  
  if (approvalToken.expiresAt < new Date()) {
    return { valid: false, error: 'Token has expired' };
  }
  
  return { 
    valid: true, 
    changeOrderId: approvalToken.changeOrderId 
  };
}
```

**Security Considerations:**
- Use cryptographically secure random token generation
- Store tokens hashed in database (optional)
- Rate limit token validation endpoint (prevent brute force)
- Log all token usage attempts
- Invalidate token after first use
- Support manual revocation by contractor

---

### 4.4 PDF Generation

**Requirements:**
- [ ] Professional change order PDF documents
- [ ] Include company branding and logo
- [ ] Display all change items with calculations
- [ ] Financial impact summary
- [ ] Terms and conditions
- [ ] Approval signature section
- [ ] Download link in emails

**PDF Library Options:**
- Puppeteer (Chrome headless - most flexible)
- PDFKit (Node.js library - faster)
- jsPDF (client-side - lighter weight)

**Recommended:** Puppeteer for professional formatting

**PDF Structure:**
```
┌─────────────────────────────────────────┐
│  [Company Logo]          CO-2025-004    │
│  Company Name                           │
│  Address Line 1                         │
│  City, State, ZIP                       │
│                                         │
│  CHANGE ORDER                           │
│                                         │
│  Date: October 14, 2025                │
│  Project: Website Redesign              │
│  Original Quote: Q-2025-001            │
│                                         │
│  Bill To:                               │
│  Client Name                            │
│  Client Address                         │
│                                         │
│  ─────────────────────────────────────  │
│                                         │
│  Change Description                     │
│  Additional scope requested by client   │
│                                         │
│  ─────────────────────────────────────  │
│                                         │
│  CHANGE ITEMS                           │
│  ┌───────────────────────────────────┐ │
│  │ Description    Qty  Price   Total │ │
│  │ Mobile Design   1   $2,000  $2,000│ │
│  │ Blog Setup      1   $1,000  $1,000│ │
│  │ Contact Form    1   $500    $500  │ │
│  └───────────────────────────────────┘ │
│                                         │
│  FINANCIAL SUMMARY                      │
│  Original Contract Total:   $15,000.00 │
│  Change Order Amount:       +$3,500.00 │
│  Tax (GST 5%):              +$175.00   │
│  ─────────────────────────────────────  │
│  Revised Contract Total:    $18,675.00 │
│                                         │
│  ─────────────────────────────────────  │
│                                         │
│  APPROVAL                               │
│                                         │
│  Client Signature: _________________   │
│  Name: John Smith                       │
│  Date: October 14, 2025                │
│  IP: 192.168.1.1                       │
│                                         │
│  ─────────────────────────────────────  │
│                                         │
│  Terms and Conditions                   │
│  [Standard T&Cs text here...]          │
│                                         │
│  Page 1 of 1                            │
└─────────────────────────────────────────┘
```

**Implementation:**
```typescript
import puppeteer from 'puppeteer';

async function generateChangeOrderPDF(
  changeOrderId: number
): Promise<Buffer> {
  const changeOrder = await getChangeOrder(changeOrderId);
  
  // Generate HTML from template
  const html = renderChangeOrderHTML(changeOrder);
  
  // Launch headless browser
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  
  // Set content and generate PDF
  await page.setContent(html);
  const pdfBuffer = await page.pdf({
    format: 'letter',
    margin: {
      top: '0.5in',
      right: '0.5in',
      bottom: '0.5in',
      left: '0.5in'
    },
    printBackground: true
  });
  
  await browser.close();
  
  // Store PDF in database or S3
  await storeChangeOrderPDF(changeOrderId, pdfBuffer);
  
  return pdfBuffer;
}
```

---

### 4.5 Approval Tracking

**Requirements:**
- [ ] Record approval timestamp and user
- [ ] Track decline reasons
- [ ] History log for all approval actions
- [ ] Contractor approval (internal)
- [ ] Dual approval workflow support

**Database Schema:**
```prisma
model ChangeOrder {
  // ... existing fields
  
  // Client Approval
  clientApprovedAt   DateTime?
  clientApprovedBy   String?      // Name entered by client
  clientApprovalIp   String?
  clientDeclinedAt   DateTime?
  clientDeclineReason String?
  
  // Contractor Approval (internal)
  contractorApprovedAt DateTime?
  contractorApprovedBy Int?       // User ID
  
  approvalToken      ApprovalToken?
}

model ApprovalToken {
  id              Int           @id @default(autoincrement())
  changeOrderId   Int           @unique
  token           String        @unique
  createdAt       DateTime      @default(now())
  expiresAt       DateTime
  usedAt          DateTime?
  revokedAt       DateTime?
  ipAddress       String?
  userAgent       String?
  
  changeOrder     ChangeOrder   @relation(fields: [changeOrderId], references: [id], onDelete: Cascade)
  
  @@index([token])
  @@index([changeOrderId])
}
```

**Approval History:**
```typescript
async function recordApproval(
  changeOrderId: number,
  approvedBy: string,
  metadata: {
    ipAddress: string;
    userAgent: string;
  }
): Promise<void> {
  await prisma.changeOrder.update({
    where: { id: changeOrderId },
    data: {
      status: 'APPROVED',
      clientApprovedAt: new Date(),
      clientApprovedBy: approvedBy,
      clientApprovalIp: metadata.ipAddress
    }
  });
  
  await prisma.changeOrderHistory.create({
    data: {
      changeOrderId,
      status: 'APPROVED',
      changes: {
        action: 'client_approved',
        approvedBy,
        timestamp: new Date().toISOString(),
        metadata
      },
      userId: 1 // System user
    }
  });
}
```

---

## Implementation Plan

### Day 1-2: Token Management & Backend
- [ ] Create ApprovalToken model migration
- [ ] Implement token generation service
- [ ] Add token validation middleware
- [ ] Create approval/decline API endpoints
- [ ] Add token cleanup job (expired tokens)

### Day 2-3: Email System
- [ ] Set up email service (SendGrid)
- [ ] Create HTML email templates
- [ ] Implement email sending service
- [ ] Add reminder scheduling logic
- [ ] Test email delivery

### Day 3-4: Public Approval Page
- [ ] Create public route (no auth)
- [ ] Build approval page UI
- [ ] Implement token validation on page load
- [ ] Add approve/decline form handlers
- [ ] Mobile responsive design
- [ ] Error handling (expired, invalid tokens)

### Day 4-5: PDF Generation & Polish
- [ ] Set up Puppeteer
- [ ] Create PDF template
- [ ] Implement PDF generation endpoint
- [ ] Add PDF attachment to emails
- [ ] Store PDFs for historical record
- [ ] End-to-end testing
- [ ] Documentation

---

## API Endpoints

### Send Change Order
```typescript
POST /api/change-orders/:id/send
Authorization: Bearer {token}

Request:
{
  "email": "client@example.com",
  "message": "Optional message to client",
  "expiryDays": 30  // Optional, defaults to 30
}

Response:
{
  "success": true,
  "changeOrder": { ... },
  "approvalToken": {
    "token": "abc123...",
    "expiresAt": "2025-11-13T00:00:00Z",
    "approvalUrl": "https://app.projectledger.com/approve/change-orders/abc123..."
  }
}
```

### Client Approval (Public Endpoint)
```typescript
POST /api/public/change-orders/approve
No Authorization Required

Request:
{
  "token": "abc123...",
  "approvedBy": "John Smith",
  "title": "CEO",  // Optional
  "comments": "Approved. Please proceed."  // Optional
}

Response:
{
  "success": true,
  "message": "Change order approved successfully",
  "changeOrder": {
    "id": 123,
    "number": "CO-2025-004",
    "status": "APPROVED",
    "clientApprovedAt": "2025-10-14T15:30:00Z"
  }
}
```

### Client Decline (Public Endpoint)
```typescript
POST /api/public/change-orders/decline
No Authorization Required

Request:
{
  "token": "abc123...",
  "reason": "Budget constraints"  // Required
}

Response:
{
  "success": true,
  "message": "Change order declined",
  "changeOrder": {
    "id": 123,
    "number": "CO-2025-004",
    "status": "DECLINED",
    "clientDeclinedAt": "2025-10-14T15:30:00Z"
  }
}
```

### Get Approval Status (Public Endpoint)
```typescript
GET /api/public/change-orders/:token

Response:
{
  "valid": true,
  "changeOrder": {
    "number": "CO-2025-004",
    "title": "Additional Features",
    "description": "...",
    "originalTotal": 15000,
    "newTotal": 18500,
    "deltaAmount": 3500,
    "expiresAt": "2025-11-13T00:00:00Z",
    "items": [...]
  }
}

Error Response (expired/invalid):
{
  "valid": false,
  "error": "Token has expired",
  "message": "This approval link is no longer valid. Please contact us for assistance."
}
```

### Generate PDF
```typescript
GET /api/change-orders/:id/pdf
Authorization: Bearer {token}

Response: Binary PDF file
Content-Type: application/pdf
Content-Disposition: attachment; filename="CO-2025-004.pdf"
```

---

## Success Criteria

- [ ] Client can approve change order from email link without login
- [ ] Secure tokens expire after 30 days
- [ ] Single-use tokens prevent duplicate approvals
- [ ] Professional email templates render correctly
- [ ] PDF documents include all required information
- [ ] Mobile devices can approve change orders
- [ ] Complete audit trail maintained for all actions
- [ ] Reminder emails sent automatically
- [ ] Expired change orders marked correctly
- [ ] Contractors notified of approval/decline immediately

---

## Testing Checklist

### Unit Tests
- [ ] Token generation creates unique tokens
- [ ] Token validation catches expired tokens
- [ ] Token validation catches used tokens
- [ ] Email service sends to correct recipients
- [ ] PDF generation includes all data
- [ ] Approval updates change order status correctly
- [ ] Decline records reason properly

### Integration Tests
- [ ] Send endpoint creates token and sends email
- [ ] Public approval endpoint validates token
- [ ] Approval triggers email notifications
- [ ] PDF attached to approval emails
- [ ] History records all actions

### End-to-End Tests
- [ ] Complete workflow: send → approve → execute
- [ ] Complete workflow: send → decline → resend
- [ ] Token expiry handling
- [ ] Email delivery verification
- [ ] Mobile device approval
- [ ] Invalid token error handling

---

## Security Considerations

1. **Token Security**
   - Use cryptographically secure random generation
   - 32+ character tokens
   - Store hashed (optional but recommended)
   - Rate limit validation endpoint

2. **CSRF Protection**
   - Tokens are single-use
   - IP address logging
   - User agent logging

3. **Data Privacy**
   - Don't expose sensitive data in URLs
   - HTTPS required for all endpoints
   - No PII in logs

4. **Rate Limiting**
   - Limit approval attempts per IP
   - Limit email sending per user
   - Prevent token brute force

---

## Monitoring & Observability

**Metrics to Track:**
- Approval rate (approved / sent)
- Average time to approval
- Token expiry rate
- Email delivery rate
- Email open rate
- Mobile vs desktop approvals
- Decline reasons (categorized)

**Alerts:**
- Email delivery failures
- PDF generation errors
- High decline rate
- Token validation errors
- Unusual approval patterns

---

## Documentation Requirements

- [ ] API documentation (OpenAPI)
- [ ] Email template customization guide
- [ ] Client approval user guide
- [ ] Troubleshooting guide
- [ ] Security best practices
- [ ] Admin configuration guide

---

## Future Enhancements (Post-Phase 4)

1. **Multi-level Approvals**
   - Require multiple approvers
   - Sequential or parallel approval flows
   - Approval thresholds by amount

2. **Digital Signatures**
   - Integration with DocuSign
   - Integration with Adobe Sign
   - Legal compliance features

3. **SMS Notifications**
   - Send approval link via SMS
   - Reminder texts
   - Two-factor authentication

4. **White-label Branding**
   - Custom email templates per account
   - Custom PDF styling
   - Custom domain for approval links

5. **Approval Analytics**
   - Dashboard for approval metrics
   - Trend analysis
   - Bottleneck identification

---

**Document Version:** 1.0 – Phase 4 Approval Workflow  
**Last Updated:** October 14, 2025  
**Status:** Ready for Implementation  
**Related Documents:**
- [SPEC_change_orders.md](../../critical/SPEC_change_orders.md) - Main specification
- [CHANGE_ORDERS_FUTURE_ENHANCEMENTS.md](../../medium/SPEC_CHANGE_ORDERS_FUTURE_ENHANCEMENTS.md) - Phase 5 & 6
