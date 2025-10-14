# Quotes & Estimates AI Agent Specification

## 1. Scope & Definitions
- **Quote**: Fixed-price offer for specific scope; binding until expiry.
- **Estimate**: Non-binding approximation; actual costs may vary.
- **Deliverable**: Unified system to manage Quotes and Estimates with shared logic for creation, approval, calculation, and conversion.

---

## 2. Roles & Permissions
**Roles:**
- **Sales/Owner** – Create, edit, send, convert to invoice.
- **Finance/Admin** – Configure templates, taxes, approval rules.
- **Support** – Read-only, resend, annotate.
- **Client** – View, accept/decline, select optional items.
- **System/Agent** – Handle automation: reminders, expiry, audits.

**RBAC Rules:**
- Creation/editing: Sales, Admin.
- Client access: secure tokenized link only.
- System actions: scheduled or event-based automation.

---

## 3. Functional Requirements

### 3.1 Creation & Structure
1. Support both **Quote** and **Estimate** types.
2. Required sections: header, client info, issue date, validity.
3. Itemized lines with types: standard | optional | discount | fee.
4. Optional items toggleable by client before acceptance.
5. Tax configuration: multi-rate, multi-jurisdiction.
6. Explicit currency using ISO-4217.
7. Contingency % for estimates (default 5–15%).
8. Notes/assumptions/exclusions (rich text).
9. Payment terms definition.
10. Attachments (e.g., SOW, proposal).

### 3.2 Calculations
11. Subtotal = Σ(lineTotal)
12. Discounts: line-level or header-level.
13. Taxes applied after discounts with configurable compounding.
14. Contingency applied only to estimates.
15. Grand total = subtotal − discounts + fees + contingency + tax.

### 3.3 Validity & Versioning
16. Validity default 30 days.
17. Auto-generated quote numbers: pattern `Q-{YYYY}-{SEQ}-vN`.
18. Immutable version history with diff logging.

### 3.4 Approvals & Signatures
19. Internal approval workflow based on thresholds.
20. Client signature with metadata (name, title, IP, timestamp).
21. Locked snapshot post-acceptance.

### 3.5 Client Experience
22. Secure share link (token-based).
23. Dynamic total updates for optional items.
24. Expiry countdown + CTA (Accept / Request Change).
25. Comment thread pre-acceptance.

### 3.6 Conversion & Notifications
26. Accepted quotes convert to invoice payload.
27. Trigger Slack/email notifications.
28. CRM integration for status updates.

### 3.7 Reminders & Expiry
29. Automated reminders (T−7, T−2 days).
30. Auto-expire past validity; allow reissue.

### 3.8 Templates & Localization
31. Configurable templates for layout and terms.
32. Currency, date, and tax localization.

### 3.9 Audit & Compliance
33. Full audit trail (actor, action, timestamp, diff).
34. PDF snapshot with hash signature.
35. GDPR compliance for client data.

### 3.10 Validation & Error Handling
36. Block send if: missing email, zero items, invalid taxes, or unapproved quote.
37. Enforce rounding strategy.

---

## 4. Non-Functional Requirements
- **Uptime**: 99.9% for acceptance endpoint.
- **Performance**: <200ms p95 for calculation API.
- **Security**: HMAC tokens, scoped access, short TTL.
- **Observability**: Structured logs, metrics (views, accepts).
- **Accessibility**: WCAG 2.1 AA for client UI.

---

## 5. Data Schema (JSON)
```json
{
  "id": "UUID",
  "number": "Q-2025-0001-v1",
  "type": "quote",
  "status": "draft",
  "issueDate": "2025-10-11",
  "validUntil": "2025-11-10",
  "currency": "CAD",
  "seller": {"name": "Trellis", "email": "quotes@trellis.org"},
  "client": {"name": "Acme Foundation", "email": "ops@acme.org"},
  "lines": [
    {"description": "Setup", "quantity": 1, "unitPrice": 5000, "lineType": "standard"},
    {"description": "Discount", "quantity": 1, "unitPrice": -300, "lineType": "discount"}
  ],
  "taxes": [{"code": "GST", "rate": 0.05}],
  "totals": {
    "subtotal": 5000,
    "discounts": 300,
    "tax": 235,
    "grandTotal": 4935
  }
}
```

---

## 6. Calculation Logic
```pseudo
function computeTotals(quote):
  subtotal = sum(line.quantity * line.unitPrice for standard/optional selected)
  discounts = sum(line.unitPrice for discounts)
  contingency = quote.type == 'estimate' ? subtotal * quote.contingencyPct : 0
  taxBase = subtotal - discounts + contingency
  tax = taxBase * taxRate
  total = subtotal - discounts + contingency + tax
  return totals
```

---

## 7. API Endpoints
- `POST /api/quotes` – create
- `PATCH /api/quotes/:id` – update
- `POST /api/quotes/:id/send` – email/share
- `POST /api/quotes/:id/accept` – client acceptance
- `POST /api/quotes/:id/decline` – client decline
- `POST /api/quotes/:id/convert` – convert to invoice
- `POST /api/quotes/:id/remind` – send reminder
- `GET /api/quotes/:id/pdf` – get signed snapshot

---

## 8. Automation Tasks
- **Nightly expiry check:** mark expired quotes.
- **Reminder scheduler:** trigger notifications.
- **Post-acceptance hooks:** invoice generation, CRM sync.

---

## 9. Validation Rules
- Client email valid.
- ≥1 billable item.
- Tax rate 0–1.
- Validity date > issue date.
- Error messages:
  - “Add at least one billable item.”
  - “Set a valid expiry date.”
  - “Invalid tax configuration.”

---

## 10. Test Coverage
- Totals with/without optional items.
- Multi-tax compound validation.
- Expiry auto-transition.
- Full acceptance → conversion flow.

---

## 11. Integration Points
- **CRM (HubSpot)**: log activity, close deal on acceptance.
- **Accounting (Stripe/QuickBooks)**: push invoices.
- **Notifications**: Slack + email triggers.

---

## 12. Acceptance Criteria
✅ Create/edit/send flows follow schema.
✅ Totals match deterministic calculation rules.
✅ Acceptance locks document; versioned reissues.
✅ Expiry handled automatically.
✅ PDF snapshot generated and hashed.

---

## 13. AI Agent Prompts
**Code Generation Prompt:**
> Implement the Quotes/Estimates API per the schema and endpoints. Use TypeScript + Express + Prisma. Validate inputs with Zod and return computed totals. Include versioning, audit logging, and OpenAPI documentation.

**Review Prompt:**
> Review implementation for compliance with spec. Verify access control, rounding, tax logic, version immutability, and API response consistency.

---

**File Version:** v1.0 – Quotes & Estimates AI Agent Spec  
**Last Updated:** 2025-10-11

