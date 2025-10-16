# Invoice Management AI Agent Specification

> **Implementation Status**: Updated 2025-10-16  
> Legend: ✅ Completed | ⚠️ Partial | ❌ Not Implemented

## 1. Scope & Definitions
- **Invoice**: A formal, legally binding request for payment based on delivered goods or services. ✅
- **Purpose**: To capture, issue, track, and reconcile invoices automatically with full auditability and compliance. ⚠️
- **Deliverable**: AI-enabled Invoice Management System that supports creation, sending, reminders, payments, reconciliation, and integrations. ⚠️

**Current Status**: Core invoice CRUD implemented with payment tracking. Missing automation, reminders, and integrations.

---

## 2. Roles & Permissions
**Roles:**
- **Sales/Owner** – Generate and send invoices. ⚠️
- **Finance/Admin** – Configure numbering, taxes, templates, approval rules. ⚠️
- **Support** – View, resend, and assist with payment issues. ❌
- **Client** – View, pay, or download invoice. ❌
- **System/Agent** – Automate reminders, expiry, and payment reconciliation. ❌

**RBAC Rules:**
- Only Sales/Admin can create or modify invoices. ✅ (Basic auth)
- Clients interact only through secure tokenized links. ❌
- System executes scheduled and event-driven actions. ❌

**Current Status**: Basic JWT authentication. Users can create/update invoices. No role-based permissions, no client portal, no automation.

---

## 3. Functional Requirements

### 3.1 Creation & Structure
1. Create **Invoices** from scratch or from accepted quotes. ⚠️ (Create works, quote linking exists, no auto-conversion)
2. Include **header** (seller identity), **client info**, **issue date**, **due date**, **currency**. ⚠️ (Client/dates yes, seller/currency not in schema)
3. Add **line items**: {description, quantity, unit, unitPrice, taxRate}. ✅
4. Compute subtotals, taxes, total, balance due. ✅
5. Configure **payment terms** (Net 15, Net 30, etc.). ❌ (No terms field in invoice schema)
6. Support multiple **payment methods** (ACH, card, PayPal, manual bank transfer). ⚠️ (Enum exists: CASH, CHECK, CREDIT_CARD, BANK_TRANSFER, OTHER)
7. Include **notes**, **payment instructions**, **terms**, and **attachments**. ⚠️ (Notes only, no instructions/terms/attachments)

**Current Status**: Basic creation with items and client. Missing: seller info, currency, payment terms, attachments.

### 3.2 Numbering & Versioning
8. Unique sequential identifiers (e.g., `INV-{YYYY}-{SEQ}`). ⚠️ (Pattern: INV{YY}{MM}{NNNN})
9. Immutable once issued; use **credit notes** for corrections. ❌ (Can update, no immutability, credit notes spec only)
10. Allow draft → sent → paid lifecycle; canceled invoices marked as **void**. ✅

**Current Status**: Numbering implemented. State machine exists (DRAFT, SENT, PAID, OVERDUE, VOID). No immutability enforcement.

### 3.3 Lifecycle & State Machine
| State | Trigger | Next State |
|--------|----------|-------------|
| Draft | Issued | Sent | ✅
| Sent | Viewed | Viewed | ❌ (No VIEWED state)
| Viewed | Payment received | Paid | ❌
| Sent/View | Due date passed | Overdue | ⚠️ (Status exists, manual only)
| Paid | Reconciled | Closed | ⚠️ (No CLOSED state)
| Any | Manual void | Canceled | ✅ (VOID status)

**Current Status**: Basic state machine implemented. Missing: VIEWED state, automated overdue transition, CLOSED state.

### 3.4 Tax & Compliance
11. Multi-tax support (GST/HST/VAT/PST, compounding options). ⚠️ (TaxService integration, single tax field)
12. Show registration numbers (e.g., GST# 12345 6789 RT0001). ❌
13. Include delivery/service date if required by region. ❌
14. PDF and JSON archive of each issued invoice (immutable snapshot). ⚠️ (PDF yes, no hash/immutability)

**Current Status**: Tax calculated via TaxService. PDF generation functional. No tax registration display, no compliance features.

### 3.5 Payments & Reconciliation
15. Integrate with payment gateways (Stripe, PayPal, ACH). ⚠️ (Payment schema exists with Stripe fields, limited implementation)
16. Support manual payment marking (for checks or transfers). ✅
17. Update status automatically on payment confirmation. ⚠️ (Manual via recordPayment, auto-updates to PAID)
18. Partial payments tracked and shown in **balanceDue** field. ✅

**Current Status**: Payment recording works. Payment status tracking (UNPAID, PARTIALLY_PAID, PAID). Auto status update on full payment. Gateway integration partial.

### 3.6 Reminders & Automation
19. Automatic reminders:
   - T-3 days before due ❌
   - T+3, T+7, T+14 days overdue ❌
20. Auto-expire unpaid drafts after 90 days. ❌
21. Webhooks for invoice lifecycle: created, sent, paid, overdue, voided. ❌

**Current Status**: PaymentReminder schema exists but no automation. No webhooks.

### 3.7 Client Experience
22. Secure view via tokenized URL. ❌
23. Display company branding, total amount due, and due date. ❌
24. Include Pay Now CTA linked to payment processor. ❌
25. Downloadable PDF copy and digital receipt after payment. ⚠️ (PDF download exists for authenticated users)

**Current Status**: No client-facing portal. All features are admin/internal only.

### 3.8 Integration Points
26. Convert from accepted quote: inherit items, taxes, and payment terms. ⚠️ (quoteId field exists, no auto-conversion)
27. Sync to CRM: update deal or org record with invoice link and payment status. ❌
28. Sync to accounting (QuickBooks/Xero): push issued and paid invoices. ❌

**Current Status**: Database linkage to quotes. No external integrations.

### 3.9 Audit & Compliance
29. Immutable once issued; all edits logged. ⚠️ (InvoiceHistory exists, not immutable)
30. Retain for 7 years minimum. ❌ (No retention policy)
31. Credit notes must reference original invoice ID. ❌ (Credit note spec exists, not implemented)
32. Include digital hash of PDF snapshot. ❌

**Current Status**: History tracking implemented. No immutability enforcement or compliance features.

### 3.10 Validation & Error Handling
33. Validate client and seller info, currency, and totals. ⚠️ (Partial validation via Zod)
34. Disallow negative total amounts. ❌ (Not enforced)
35. Ensure dueDate > issueDate. ❌ (Not enforced)
36. Return clear error messages:
   - "Invoice must have at least one line item." ⚠️ (Zod validates positive quantity)
   - "Due date cannot precede issue date." ❌
   - "Tax rate must be between 0 and 1." ❌

**Current Status**: Basic Zod validation on API. Needs strengthening.

---

## 4. Non-Functional Requirements
- **Availability:** 99.9% uptime for client access and payment endpoints. ❌ (No client endpoints)
- **Performance:** <250ms p95 for calculation API. ⚠️ (Not measured)
- **Security:** HMAC-secured links, TLS enforced. ⚠️ (JWT auth, no HMAC links)
- **Auditability:** All state changes logged with timestamp and actor. ✅
- **Localization:** Currency and date formats regionalized. ❌

**Current Status**: History tracking works. Basic security. No performance monitoring or localization.

---

## 5. Shared Data Schema (JSON) — Standard v1.1
This schema is **shared with the Invoice spec**. Domain objects are reusable and versioned.

### 5.1 Core Types
```json
{
  "$id": "https://spec.trellis.ai/billing/shared-v1.1.json",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Billing Shared Types v1.1",
  "type": "object",
  "$defs": {
    "Money": {"type":"number","description":"Decimal currency amount; rounding HALF_EVEN to 2 fraction digits unless configured."},
    "Party": {
      "type":"object",
      "required":["name"],
      "properties":{
        "name":{"type":"string"},
        "email":{"type":"string","format":"email"},
        "address":{"type":"string"},
        "taxNumber":{"type":"string"}
      }
    },
    "Tax": {
      "type":"object",
      "required":["code","rate"],
      "properties":{
        "code":{"type":"string"},
        "label":{"type":"string"},
        "rate":{"type":"number","minimum":0,"maximum":1},
        "compound":{"type":"boolean","default":false},
        "appliesTo":{"type":"string","enum":["subtotal","subtotal_minus_discounts","subtotal_plus_fees"],"default":"subtotal_minus_discounts"}
      }
    },
    "LineItem": {
      "type":"object",
      "required":["id","description","quantity","unitPrice"],
      "properties":{
        "id":{"type":"string"},
        "description":{"type":"string"},
        "quantity":{"type":"number","minimum":0},
        "unit":{"type":"string"},
        "unitPrice":{"$ref":"#/$defs/Money"},
        "lineType":{"type":"string","enum":["standard","optional","discount","fee"],"default":"standard"},
        "selected":{"type":"boolean","description":"for optional items"},
        "metadata":{"type":"object"}
      }
    },
    "Totals": {
      "type":"object",
      "required":["subtotal","discounts","fees","contingency","tax","grandTotal"],
      "properties":{
        "subtotal":{"$ref":"#/$defs/Money"},
        "discounts":{"$ref":"#/$defs/Money"},
        "fees":{"$ref":"#/$defs/Money"},
        "contingency":{"$ref":"#/$defs/Money"},
        "tax":{"$ref":"#/$defs/Money"},
        "grandTotal":{"$ref":"#/$defs/Money"},
        "rounding":{"type":"object","properties":{"mode":{"type":"string"},"fractionDigits":{"type":"integer"}}}
      }
    }
  }
}
```

### 5.2 Quote/Estimate Resource (uses shared types)
```json
{
  "$id": "https://spec.trellis.ai/billing/quote-v1.1.json",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Quote/Estimate v1.1",
  "type": "object",
  "required": ["id","number","version","type","status","issueDate","validUntil","currency","seller","client","lines","totals"],
  "properties": {
    "id": {"type": "string", "description":"UUID"},
    "number": {"type": "string"},
    "version": {"type": "integer","minimum":1},
    "type": {"type":"string","enum":["quote","estimate"]},
    "status": {"type":"string","enum":["draft","pending_approval","approved","sent","viewed","accepted","declined","expired","converted"]},
    "issueDate": {"type": "string", "format": "date"},
    "validUntil": {"type": "string", "format": "date"},
    "currency": {"type": "string", "pattern": "^[A-Z]{3}$"},
    "seller": {"$ref": "https://spec.trellis.ai/billing/shared-v1.1.json#/$defs/Party"},
    "client": {"$ref": "https://spec.trellis.ai/billing/shared-v1.1.json#/$defs/Party"},
    "paymentTerms": {"type": "string"},
    "notes": {"type": "string"},
    "assumptions": {"type": "string"},
    "exclusions": {"type": "string"},
    "contingencyPct": {"type": "number","minimum":0,"maximum":1},
    "taxes": {"type": "array", "items": {"$ref": "https://spec.trellis.ai/billing/shared-v1.1.json#/$defs/Tax"}},
    "lines": {"type": "array", "items": {"$ref": "https://spec.trellis.ai/billing/shared-v1.1.json#/$defs/LineItem"}},
    "totals": {"$ref": "https://spec.trellis.ai/billing/shared-v1.1.json#/$defs/Totals"},
    "approvals": {"type":"array","items":{"type":"object","properties":{"approvedBy":{"type":"string"},"timestamp":{"type":"string","format":"date-time"},"note":{"type":"string"}}}},
    "clientActions": {"type":"object","properties":{ "viewedAt":{"type":"string","format":"date-time"}, "acceptedAt":{"type":"string","format":"date-time"}, "declinedAt":{"type":"string","format":"date-time"}, "signature":{"type":"object","properties":{"name":{"type":"string"},"title":{"type":"string"},"ip":{"type":"string"},"blob":{"type":"string"}}} }},
    "audit": {"type":"array","items":{"type":"object","properties":{"actor":{"type":"string"},"action":{"type":"string"},"timestamp":{"type":"string","format":"date-time"},"diff":{"type":"object"}}}},
    "share": {"type":"object","properties":{"token":{"type":"string"},"expiresAt":{"type":"string","format":"date-time"}}},
    "integration": {"type":"object","properties":{"crm":{"type":"object"},"accounting":{"type":"object"}}}
  }
}
```

### 5.3 Deterministic Calculation Rules (Shared)
```
lineTotal = quantity * unitPrice (standard & optional where selected=true)
subtotal = Σ(lineTotal for standard + selected optional)
lineDiscounts = Σ(|discount line totals|) as positive value
headerDiscount = configured percent/absolute applied to subtotal
fees = Σ(fee lines)
contingency = (type == "estimate") ? contingencyPct * (subtotal - (lineDiscounts + headerDiscount)) : 0
// Taxes computed in declared order; if tax.compound == true, prior tax is included in base
base = choose(appliesTo)
tax = Σ(round(base * tax.rate, 2))
grandTotal = subtotal - (lineDiscounts + headerDiscount) + fees + contingency + tax
```
---

## 6. Calculation Logic
```pseudo
function computeInvoiceTotals(invoice):
  subtotal = Σ(line.quantity * line.unitPrice)
  taxTotal = Σ(subtotal * tax.rate)
  total = subtotal + taxTotal
  balanceDue = total - paidAmount
  return {subtotal, taxTotal, total, balanceDue}
```

---

## 7. API Endpoints
- `POST /api/invoices` – Create new invoice ✅
- `PATCH /api/invoices/:id` – Update (draft only) ⚠️ (Updates any status)
- `POST /api/invoices/:id/send` – Send to client ⚠️ (Updates status, no actual email)
- `POST /api/invoices/:id/pay` – Register payment (manual or gateway) ⚠️ (Via /payments endpoint, manual only)
- `POST /api/invoices/:id/remind` – Send payment reminder ❌
- `POST /api/invoices/:id/void` – Cancel invoice ⚠️ (Via voidInvoice service method, no dedicated endpoint)
- `GET /api/invoices/:id/pdf` – Retrieve immutable snapshot ⚠️ (PDF yes, not immutable)
- `GET /api/invoices/:id` – Fetch invoice with payment history ✅

**Additional Implemented Endpoints**:
- `GET /api/invoices` – List invoices with filters ✅
- `POST /api/invoices/:id/payments` – Record payment ✅
- `GET /api/invoices/:id/payments` – Get payments ✅
- `DELETE /api/invoices/:id` – Delete draft invoices ✅

**Location**: `apps/backend/src/routes/invoices.ts`

**Missing**: Dedicated void endpoint, remind endpoint, gateway payment processing

---

## 8. Automation Tasks
- **Nightly job:** mark invoices overdue if past dueDate. ❌
- **Reminder scheduler:** run on schedule (T-3, T+3, T+7, T+14). ❌
- **Payment reconciliation:** auto-update status from gateway events. ❌
- **Archival task:** lock and hash invoices older than 7 years. ❌

**Current Status**: No automation implemented. PaymentReminder schema exists but unused. RecurringInvoice schema exists.

---

## 9. Validation Rules
- At least one nonzero line item. ⚠️ (Zod validates positive quantity)
- Valid email for client and seller. ⚠️ (Client exists in DB, not validated)
- Tax rates within [0, 1]. ❌
- Due date after issue date. ❌
- Positive totals only. ❌

**Current Status**: Basic Zod validation in routes. Backend needs strengthening.

---

## 10. Test Coverage
- Calculation with single and multiple tax rates. ❌
- Payment marking and partial payments. ❌
- Reminder triggers and status transitions. ❌
- Conversion from quote to invoice. ❌
- Immutable snapshot and audit trail verification. ❌

**Current Status**: No test files found for invoice functionality.

---

## 11. Integration Points
- **CRM (HubSpot):** sync invoice activity and payment updates. ❌
- **Accounting (QuickBooks/Xero):** export invoice with tax and line mappings. ❌
- **Payment Gateways (Stripe/ACH):** handle reconciliation. ⚠️ (Stripe schema fields exist, minimal implementation)
- **Notifications:** Slack/email on send, payment, overdue. ❌

**Current Status**: No external integrations active. Payment gateway infrastructure partial.

---

## 12. Acceptance Criteria
- ✅ Invoice numbering unique and sequential. ✅
- ✅ Totals computed per deterministic rules. ✅
- ✅ Overdue transitions occur automatically. ❌
- ✅ Paid invoices locked and archived. ❌
- ✅ All changes logged with timestamps. ✅
- ✅ Webhooks emitted for major lifecycle events. ❌

**Overall Status**: Core CRUD and payment tracking functional. Missing automation, client portal, and integrations.

---

## 13. AI Agent Prompts
**Code Generation Prompt:**
> Implement the Invoice Management Service per the spec, using TypeScript, Express, and Prisma. Include validation, state machine transitions, and audit logging. Generate OpenAPI 3.1 documentation.

**Review Prompt:**
> Perform a compliance and security review of the Invoice Service implementation. Validate correct tax handling, payment reconciliation, state immutability, and webhook consistency.

---

**File Version:** v1.1 – Invoice Management AI Agent Specification (with Implementation Review)  
**Last Updated:** 2025-10-16  
**Review Date:** 2025-10-16

---

## IMPLEMENTATION SUMMARY

### What's Implemented ✅

**Core Features:**
- Invoice CRUD operations (Create, Read, Update, Delete)
- Invoice numbering (pattern: INV{YY}{MM}{NNNN})
- Status management (DRAFT, SENT, PAID, OVERDUE, VOID)
- Payment recording with partial payment support
- Payment status tracking (UNPAID, PARTIALLY_PAID, PAID)
- History logging (InvoiceHistory table)
- Line items with grouping support
- Tax calculation via TaxService integration
- PDF generation
- Balance calculation (total - amountPaid)
- Quote linking (quoteId field)
- Recurring invoice schema
- Template system (user invoice templates)

**Technical Stack:**
- Backend: TypeScript + Express + Prisma ✅
- Validation: Zod schemas ✅
- Database: PostgreSQL with proper relations ✅
- Frontend: React + MUI with invoice forms ✅

**API Endpoints:**
- `POST /api/invoices` - Create invoice
- `GET /api/invoices` - List invoices with filters
- `GET /api/invoices/:id` - Get single invoice
- `PATCH /api/invoices/:id` - Update invoice
- `POST /api/invoices/:id/send` - Mark as sent (status update only)
- `POST /api/invoices/:id/payments` - Record payment
- `GET /api/invoices/:id/payments` - Get payment history
- `DELETE /api/invoices/:id` - Delete draft invoices
- `GET /api/invoices/:id/pdf` - Generate PDF

**Service Methods:**
- `markAsPaid()`, `markAsOverdue()`, `voidInvoice()`
- `recordPayment()` with auto-status updates
- `calculateBalance()` and payment status tracking

### What's Partially Implemented ⚠️

- **Payment Gateway**: Stripe schema fields exist (paymentIntentId, receiptUrl), minimal integration
- **Send Feature**: Endpoint exists but only updates status (no actual email)
- **Tax System**: TaxService integrated but single tax field in invoice
- **PDF**: Generated but no hash/digital signature
- **Validation**: Zod validation present, needs strengthening (date validation, negative amounts)
- **Immutability**: History tracked but invoices can be updated
- **Quote Conversion**: quoteId field exists, no auto-conversion logic

### What's Missing ❌

**Critical Features:**
- Client portal (secure tokenized links for viewing/payment)
- Payment gateway integration (Stripe/PayPal/ACH processing)
- Email notifications (send, reminders, receipts)
- Automated reminders (T-3, T+3, T+7, T+14 days)
- Automated overdue marking (nightly job)
- Webhooks for lifecycle events
- Quote-to-invoice conversion endpoint
- Credit notes implementation (spec exists in backlog)
- Immutability enforcement for issued invoices
- Seller/company information in schema
- Currency field and multi-currency support
- Payment terms field (Net 15, Net 30, etc.)
- Delivery/service date fields
- Attachments
- Digital hash/signature for PDFs
- Tax registration number display
- External integrations (CRM, accounting)
- Data retention policies
- Compliance features (GDPR, 7-year retention)

**Data Model Gaps:**
- No `seller` or `company` object
- No `currency` field (assumed single currency)
- No `paymentTerms` field in Invoice schema
- No `viewed` state in status enum
- No `closed` state (after reconciliation)
- Missing `deliveryDate` or `serviceDate`
- No attachments support

**Validation Gaps:**
- No dueDate > issueDate enforcement
- No negative total prevention
- No tax rate range validation
- No immutability checks for issued invoices

### Change Orders Section

**Implementation Status**: ⚠️ Partial
- ChangeOrder schema exists in database ✅
- Routes exist (`/api/change-orders`) ✅
- Status enum: DRAFT, SENT, VIEWED, ACCEPTED, DECLINED, EXPIRED, CONVERTED ✅
- Financial tracking: originalTotal, newTotal, deltaAmount ✅
- Client approval token fields ✅
- Missing: Full API implementation, conversion logic, webhooks ❌

**Database Schema**: Lines 813-870 in schema.prisma

### Priority Recommendations

**High Priority (Core Functionality):**
1. Implement quote-to-invoice conversion endpoint
2. Add email sending capability (send invoices, reminders)
3. Implement payment gateway integration (Stripe at minimum)
4. Add client portal with secure tokenized links
5. Strengthen validation (dates, amounts, immutability)
6. Implement automated overdue marking
7. Add payment terms field to schema
8. Enforce immutability for issued invoices

**Medium Priority (Business Value):**
1. Automated reminder system
2. Webhook infrastructure for lifecycle events
3. Credit notes implementation
4. Currency support and localization
5. Recurring invoice automation
6. External integrations (accounting sync)
7. Digital PDF signatures/hashing
8. Tax registration display

**Low Priority (Nice to Have):**
1. CRM integration
2. Advanced reporting
3. Multi-language support
4. Bulk operations
5. Invoice templates with branding
6. Analytics dashboard

### Files to Review/Update

**Backend:**
- `apps/backend/prisma/schema.prisma` - Add seller, currency, paymentTerms, viewed state
- `apps/backend/src/services/invoiceService.ts` - Add conversion, email, immutability checks
- `apps/backend/src/routes/invoices.ts` - Add void, remind, convert endpoints
- `apps/backend/src/services/emailService.ts` - NEW: Email sending
- `apps/backend/src/services/paymentGatewayService.ts` - NEW: Stripe/PayPal integration
- `apps/backend/src/services/invoiceAutomationService.ts` - NEW: Reminders, overdue marking
- `packages/shared-types/invoices/types.ts` - Update TypeScript types

**Frontend:**
- `apps/frontend/src/api/invoices/index.ts` - Add new API methods
- `apps/frontend/src/pages/InvoiceDetailPage.tsx` - Add send/void/payment actions
- NEW: Client portal pages for viewing and paying invoices

**New Infrastructure:**
- Background job scheduler for reminders and overdue marking
- Webhook system for lifecycle events
- Email template system
- Credit note service and routes
- Test suite for invoice functionality

---

## 14. Change Orders (Shared Module)

> **Implementation Status**: ⚠️ Partially Implemented  
> Change Order database schema exists with approval workflow fields. Routes exist but full API and conversion logic not complete.

A **Change Order (CO)** is a scoped amendment to an accepted quote/contract that modifies scope, schedule, and **price**. COs reuse the **Shared Schema v1.1** types (`Money`, `Party`, `Tax`, `LineItem`, `Totals`) and the deterministic calculation rules. This module is **shared** by both the *Quote/Estimate* and *Invoice* specs.

### 14.1 When to Use
1. Before original quote acceptance → reissue **Quote vN+1** (no CO). ✅ (Quote versioning exists)
2. After acceptance, before invoicing → create **CO**, and on acceptance create **incremental invoice**. ⚠️ (Schema exists, conversion logic missing)
3. After invoice issued/unpaid → **separate delta invoice**; avoid modifying the issued invoice. ❌
4. After payment/partial delivery → if delta negative, issue **credit note** referencing original invoice; if positive, issue **new incremental invoice**. ❌

### 14.2 States & Reminders
`draft → sent → viewed → accepted | declined → converted` ✅ (States in enum)
Expiry and reminder cadence align with quotes (T−7, T−2). ❌ (No automation)

**Implementation**: ChangeOrderStatus enum exists: DRAFT, SENT, VIEWED, ACCEPTED, DECLINED, EXPIRED, CONVERTED

### 14.3 Data Model (JSON Schema)

**Implementation Status**: ⚠️ Database schema exists with most fields. Missing seller/client Party objects.

**Database Schema** (Prisma): See `schema.prisma` lines 813-870
- Fields: id, number, quoteId, version, status, changeType, originalTotal, newTotal, deltaAmount
- Approval fields: clientApprovedAt, clientApprovedBy, contractorApprovedAt, contractorApprovedBy
- Token fields: approvalToken, approvalTokenExpiry, approvalIpAddress, approvalUserAgent
- Relations: quote, createdBy, contractorApprover, items (ChangeOrderItem), history (ChangeOrderHistory)
- ChangeOrderType enum: ADDITIVE, DEDUCTIVE, NEUTRAL

**Missing from Schema**: Separate seller/client objects, taxes array, contingencyPct, scheduleImpact details, share token

```json
{
  "$id": "https://spec.trellis.ai/billing/change-order-v1.1.json",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Change Order v1.1",
  "type": "object",
  "required": ["id","number","version","status","currency","seller","client","lines","totals","parentQuoteId"],
  "properties": {
    "id": {"type":"string"},
    "number": {"type":"string"},
    "version": {"type":"integer","minimum":1},
    "status": {"type":"string","enum":["draft","sent","viewed","accepted","declined","expired","converted"]},
    "parentQuoteId": {"type":"string"},
    "parentContractRef": {"type":"string"},
    "issueDate": {"type":"string","format":"date"},
    "validUntil": {"type":"string","format":"date"},
    "currency": {"type":"string","pattern":"^[A-Z]{3}$"},
    "seller": {"$ref":"https://spec.trellis.ai/billing/shared-v1.1.json#/$defs/Party"},
    "client": {"$ref":"https://spec.trellis.ai/billing/shared-v1.1.json#/$defs/Party"},
    "reason": {"type":"string"},
    "rationale": {"type":"string"},
    "lines": {"type":"array","items":{"$ref":"https://spec.trellis.ai/billing/shared-v1.1.json#/$defs/LineItem"}},
    "taxes": {"type":"array","items":{"$ref":"https://spec.trellis.ai/billing/shared-v1.1.json#/$defs/Tax"}},
    "contingencyPct": {"type":"number","minimum":0,"maximum":1},
    "totals": {"$ref":"https://spec.trellis.ai/billing/shared-v1.1.json#/$defs/Totals"},
    "scheduleImpact": {"type":"object","properties":{"daysAdded":{"type":"integer"},"milestoneAdjustments":{"type":"array","items":{"type":"object","properties":{"milestone":{"type":"string"},"from":{"type":"string"},"to":{"type":"string"}}}}}},
    "paymentTerms": {"type":"string"},
    "approvals": {"type":"array","items":{"type":"object","properties":{"approvedBy":{"type":"string"},"timestamp":{"type":"string","format":"date-time"},"note":{"type":"string"}}}},
    "clientActions": {"type":"object","properties":{"viewedAt":{"type":"string","format":"date-time"},"acceptedAt":{"type":"string","format":"date-time"},"declinedAt":{"type":"string","format":"date-time"},"signature":{"type":"object","properties":{"name":{"type":"string"},"title":{"type":"string"},"ip":{"type":"string"},"blob":{"type":"string"}}}}},
    "audit": {"type":"array","items":{"type":"object","properties":{"actor":{"type":"string"},"action":{"type":"string"},"timestamp":{"type":"string","format":"date-time"},"diff":{"type":"object"}}}},
    "share": {"type":"object","properties":{"token":{"type":"string"},"expiresAt":{"type":"string","format":"date-time"}}},
    "links": {"type":"object","properties":{"originalQuoteNumber":{"type":"string"},"relatedInvoiceIds":{"type":"array","items":{"type":"string"}},"creditNoteIds":{"type":"array","items":{"type":"string"}}}}
  }
}
```

### 14.4 Calculation Rules (Delta)
- Reuse shared rules. ⚠️ (Basic calculation logic exists)
- **Negative lines** represent removals/credits. ⚠️ (ChangeOrderItem schema exists)
- `grandTotal` in CO is the **net delta** (may be positive or negative). ✅ (deltaAmount field)

### 14.5 API (Minimal)
- `POST /api/change-orders` (include `parentQuoteId`) ⚠️ (Route exists)
- `PATCH /api/change-orders/:id` (draft only) ⚠️ (Route exists)
- `POST /api/change-orders/:id/send` ❌
- `POST /api/change-orders/:id/accept` → returns **incremental invoice** *(+delta)* or **credit note** *(-delta)* payload ❌
- `POST /api/change-orders/:id/decline` ❌
- `GET /api/change-orders/:id/pdf` ❌
- Webhooks: `change_order.created|sent|accepted|declined|expired|converted` ❌

**Location**: `apps/backend/src/routes/change-orders.ts`

### 14.6 Acceptance Criteria
- Accepting positive delta → **incremental invoice** with correct tax and references. ❌
- Accepting negative delta → **credit note** tied to original invoice(s). ❌
- Original quote and prior invoices remain unchanged. ⚠️ (Schema supports, logic incomplete)
- Client view shows **delta**, **new project total**, and **schedule impact**; PDF snapshot hashed. ❌
- All events logged; webhooks emitted with references and amounts. ⚠️ (History exists, no webhooks)

**Current Status**: Database schema complete. Basic routes exist. Missing: conversion logic, client portal, PDF generation, webhooks.

---