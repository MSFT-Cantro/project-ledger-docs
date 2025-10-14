# Invoice Management AI Agent Specification

## 1. Scope & Definitions
- **Invoice**: A formal, legally binding request for payment based on delivered goods or services.
- **Purpose**: To capture, issue, track, and reconcile invoices automatically with full auditability and compliance.
- **Deliverable**: AI-enabled Invoice Management System that supports creation, sending, reminders, payments, reconciliation, and integrations.

---

## 2. Roles & Permissions
**Roles:**
- **Sales/Owner** – Generate and send invoices.
- **Finance/Admin** – Configure numbering, taxes, templates, approval rules.
- **Support** – View, resend, and assist with payment issues.
- **Client** – View, pay, or download invoice.
- **System/Agent** – Automate reminders, expiry, and payment reconciliation.

**RBAC Rules:**
- Only Sales/Admin can create or modify invoices.
- Clients interact only through secure tokenized links.
- System executes scheduled and event-driven actions.

---

## 3. Functional Requirements

### 3.1 Creation & Structure
1. Create **Invoices** from scratch or from accepted quotes.
2. Include **header** (seller identity), **client info**, **issue date**, **due date**, **currency**.
3. Add **line items**: {description, quantity, unit, unitPrice, taxRate}.
4. Compute subtotals, taxes, total, balance due.
5. Configure **payment terms** (Net 15, Net 30, etc.).
6. Support multiple **payment methods** (ACH, card, PayPal, manual bank transfer).
7. Include **notes**, **payment instructions**, **terms**, and **attachments**.

### 3.2 Numbering & Versioning
8. Unique sequential identifiers (e.g., `INV-{YYYY}-{SEQ}`).
9. Immutable once issued; use **credit notes** for corrections.
10. Allow draft → sent → paid lifecycle; canceled invoices marked as **void**.

### 3.3 Lifecycle & State Machine
| State | Trigger | Next State |
|--------|----------|-------------|
| Draft | Issued | Sent |
| Sent | Viewed | Viewed |
| Viewed | Payment received | Paid |
| Sent/View | Due date passed | Overdue |
| Paid | Reconciled | Closed |
| Any | Manual void | Canceled |

### 3.4 Tax & Compliance
11. Multi-tax support (GST/HST/VAT/PST, compounding options).
12. Show registration numbers (e.g., GST# 12345 6789 RT0001).
13. Include delivery/service date if required by region.
14. PDF and JSON archive of each issued invoice (immutable snapshot).

### 3.5 Payments & Reconciliation
15. Integrate with payment gateways (Stripe, PayPal, ACH).
16. Support manual payment marking (for checks or transfers).
17. Update status automatically on payment confirmation.
18. Partial payments tracked and shown in **balanceDue** field.

### 3.6 Reminders & Automation
19. Automatic reminders:
   - T-3 days before due
   - T+3, T+7, T+14 days overdue
20. Auto-expire unpaid drafts after 90 days.
21. Webhooks for invoice lifecycle: created, sent, paid, overdue, voided.

### 3.7 Client Experience
22. Secure view via tokenized URL.
23. Display company branding, total amount due, and due date.
24. Include Pay Now CTA linked to payment processor.
25. Downloadable PDF copy and digital receipt after payment.

### 3.8 Integration Points
26. Convert from accepted quote: inherit items, taxes, and payment terms.
27. Sync to CRM: update deal or org record with invoice link and payment status.
28. Sync to accounting (QuickBooks/Xero): push issued and paid invoices.

### 3.9 Audit & Compliance
29. Immutable once issued; all edits logged.
30. Retain for 7 years minimum.
31. Credit notes must reference original invoice ID.
32. Include digital hash of PDF snapshot.

### 3.10 Validation & Error Handling
33. Validate client and seller info, currency, and totals.
34. Disallow negative total amounts.
35. Ensure dueDate > issueDate.
36. Return clear error messages:
   - “Invoice must have at least one line item.”
   - “Due date cannot precede issue date.”
   - “Tax rate must be between 0 and 1.”

---

## 4. Non-Functional Requirements
- **Availability:** 99.9% uptime for client access and payment endpoints.
- **Performance:** <250ms p95 for calculation API.
- **Security:** HMAC-secured links, TLS enforced.
- **Auditability:** All state changes logged with timestamp and actor.
- **Localization:** Currency and date formats regionalized.

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
- `POST /api/invoices` – Create new invoice
- `PATCH /api/invoices/:id` – Update (draft only)
- `POST /api/invoices/:id/send` – Send to client
- `POST /api/invoices/:id/pay` – Register payment (manual or gateway)
- `POST /api/invoices/:id/remind` – Send payment reminder
- `POST /api/invoices/:id/void` – Cancel invoice
- `GET /api/invoices/:id/pdf` – Retrieve immutable snapshot
- `GET /api/invoices/:id` – Fetch invoice with payment history

---

## 8. Automation Tasks
- **Nightly job:** mark invoices overdue if past dueDate.
- **Reminder scheduler:** run on schedule (T-3, T+3, T+7, T+14).
- **Payment reconciliation:** auto-update status from gateway events.
- **Archival task:** lock and hash invoices older than 7 years.

---

## 9. Validation Rules
- At least one nonzero line item.
- Valid email for client and seller.
- Tax rates within [0, 1].
- Due date after issue date.
- Positive totals only.

---

## 10. Test Coverage
- Calculation with single and multiple tax rates.
- Payment marking and partial payments.
- Reminder triggers and status transitions.
- Conversion from quote to invoice.
- Immutable snapshot and audit trail verification.

---

## 11. Integration Points
- **CRM (HubSpot):** sync invoice activity and payment updates.
- **Accounting (QuickBooks/Xero):** export invoice with tax and line mappings.
- **Payment Gateways (Stripe/ACH):** handle reconciliation.
- **Notifications:** Slack/email on send, payment, overdue.

---

## 12. Acceptance Criteria
✅ Invoice numbering unique and sequential.
✅ Totals computed per deterministic rules.
✅ Overdue transitions occur automatically.
✅ Paid invoices locked and archived.
✅ All changes logged with timestamps.
✅ Webhooks emitted for major lifecycle events.

---

## 13. AI Agent Prompts
**Code Generation Prompt:**
> Implement the Invoice Management Service per the spec, using TypeScript, Express, and Prisma. Include validation, state machine transitions, and audit logging. Generate OpenAPI 3.1 documentation.

**Review Prompt:**
> Perform a compliance and security review of the Invoice Service implementation. Validate correct tax handling, payment reconciliation, state immutability, and webhook consistency.

---

**File Version:** v1.0 – Invoice Management AI Agent Specification  
**Last Updated:** 2025-10-11



---

## 14. Change Orders (Shared Module)
A **Change Order (CO)** is a scoped amendment to an accepted quote/contract that modifies scope, schedule, and **price**. COs reuse the **Shared Schema v1.1** types (`Money`, `Party`, `Tax`, `LineItem`, `Totals`) and the deterministic calculation rules. This module is **shared** by both the *Quote/Estimate* and *Invoice* specs.

### 14.1 When to Use
1. Before original quote acceptance → reissue **Quote vN+1** (no CO).
2. After acceptance, before invoicing → create **CO**, and on acceptance create **incremental invoice**.
3. After invoice issued/unpaid → **separate delta invoice**; avoid modifying the issued invoice.
4. After payment/partial delivery → if delta negative, issue **credit note** referencing original invoice; if positive, issue **new incremental invoice**.

### 14.2 States & Reminders
`draft → sent → viewed → accepted | declined → converted`  
Expiry and reminder cadence align with quotes (T−7, T−2).

### 14.3 Data Model (JSON Schema)
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
- Reuse shared rules.  
- **Negative lines** represent removals/credits.  
- `grandTotal` in CO is the **net delta** (may be positive or negative).

### 14.5 API (Minimal)
- `POST /api/change-orders` (include `parentQuoteId`)
- `PATCH /api/change-orders/:id` (draft only)
- `POST /api/change-orders/:id/send`
- `POST /api/change-orders/:id/accept` → returns **incremental invoice** *(+delta)* or **credit note** *(-delta)* payload
- `POST /api/change-orders/:id/decline`
- `GET /api/change-orders/:id/pdf`
- Webhooks: `change_order.created|sent|accepted|declined|expired|converted`

### 14.6 Acceptance Criteria
- Accepting positive delta → **incremental invoice** with correct tax and references.  
- Accepting negative delta → **credit note** tied to original invoice(s).  
- Original quote and prior invoices remain unchanged.  
- Client view shows **delta**, **new project total**, and **schedule impact**; PDF snapshot hashed.  
- All events logged; webhooks emitted with references and amounts.

