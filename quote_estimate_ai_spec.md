# Quotes & Estimates AI Agent Specification

> **Implementation Status**: Updated 2025-10-16  
> Legend: ✅ Completed | ⚠️ Partial | ❌ Not Implemented

## 1. Scope & Definitions
- **Quote**: Fixed-price offer for specific scope; binding until expiry. ✅
- **Estimate**: Non-binding approximation; actual costs may vary. ❌
- **Deliverable**: Unified system to manage Quotes and Estimates with shared logic for creation, approval, calculation, and conversion. ⚠️

**Current Status**: System supports Quotes only. Estimate type not implemented.

---

## 2. Roles & Permissions
**Roles:**
- **Sales/Owner** – Create, edit, send, convert to invoice. ⚠️
- **Finance/Admin** – Configure templates, taxes, approval rules. ⚠️
- **Support** – Read-only, resend, annotate. ❌
- **Client** – View, accept/decline, select optional items. ❌
- **System/Agent** – Handle automation: reminders, expiry, audits. ❌

**RBAC Rules:**
- Creation/editing: Sales, Admin. ✅ (Basic auth implemented)
- Client access: secure tokenized link only. ❌
- System actions: scheduled or event-based automation. ❌

**Current Status**: Basic authentication implemented. User can create/edit quotes. No role-based access control, no client-facing features, no automation.

---

## 3. Functional Requirements

### 3.1 Creation & Structure
1. Support both **Quote** and **Estimate** types. ❌ (Only Quote type implemented)
2. Required sections: header, client info, issue date, validity. ✅
3. Itemized lines with types: standard | optional | discount | fee. ❌ (No line type differentiation)
4. Optional items toggleable by client before acceptance. ❌
5. Tax configuration: multi-rate, multi-jurisdiction. ⚠️ (Single rate, compound tax field exists but not fully utilized)
6. Explicit currency using ISO-4217. ⚠️ (Currency field exists, hardcoded to USD)
7. Contingency % for estimates (default 5–15%). ❌
8. Notes/assumptions/exclusions (rich text). ⚠️ (Notes field exists, plain text)
9. Payment terms definition. ⚠️ (Field exists in templates, not in main schema)
10. Attachments (e.g., SOW, proposal). ❌

**Current Status**: Basic quote creation with items, client, validity date. Missing: estimate type, line item types, optional items, contingency, attachments, full currency support.

### 3.2 Calculations
11. Subtotal = Σ(lineTotal) ✅
12. Discounts: line-level or header-level. ⚠️ (Discount field on items, not implemented in calculation)
13. Taxes applied after discounts with configurable compounding. ⚠️ (Tax calculated, compound flag exists but not used)
14. Contingency applied only to estimates. ❌
15. Grand total = subtotal − discounts + fees + contingency + tax. ⚠️ (Basic calculation: subtotal + tax)

**Current Status**: Basic calculation logic exists (subtotal, tax, total). Missing: discount application, contingency, fees, compound tax calculation.


### 3.3 Validity & Versioning
16. Validity default 30 days. ✅
17. Auto-generated quote numbers: pattern `Q-{YYYY}-{SEQ}-vN`. ⚠️ (Pattern: Q{YY}{MM}{NNNN}, no version suffix)
18. Immutable version history with diff logging. ✅

**Current Status**: Versioning and history tracking implemented. Quote number format differs from spec.

### 3.4 Approvals & Signatures
19. Internal approval workflow based on thresholds. ❌ (Status field exists: INTERNAL_REVIEW, but no workflow)
20. Client signature with metadata (name, title, IP, timestamp). ❌ (Approval fields in ChangeOrder schema only)
21. Locked snapshot post-acceptance. ❌ (Can update only DRAFT status)

**Current Status**: Basic status management (DRAFT, SENT, ACCEPTED, REJECTED). No approval workflow, no signature capture.

### 3.5 Client Experience
22. Secure share link (token-based). ❌
23. Dynamic total updates for optional items. ❌
24. Expiry countdown + CTA (Accept / Request Change). ❌
25. Comment thread pre-acceptance. ❌

**Current Status**: No client-facing features implemented. All features are internal/admin only.

### 3.6 Conversion & Notifications
26. Accepted quotes convert to invoice payload. ⚠️ (Invoice schema has quoteId field, no conversion endpoint)
27. Trigger Slack/email notifications. ❌
28. CRM integration for status updates. ❌

**Current Status**: Infrastructure for quote-to-invoice linking exists, but no conversion logic or notifications.

### 3.7 Reminders & Expiry
29. Automated reminders (T−7, T−2 days). ❌
30. Auto-expire past validity; allow reissue. ❌

**Current Status**: No automation implemented. Manual status updates only.

### 3.8 Templates & Localization
31. Configurable templates for layout and terms. ✅ (QuoteTemplate and UserQuoteTemplate)
32. Currency, date, and tax localization. ❌ (Hardcoded USD/English)

**Current Status**: Template system exists for quote structure and user-saved templates. No localization.

### 3.9 Audit & Compliance
33. Full audit trail (actor, action, timestamp, diff). ✅ (QuoteHistory table)
34. PDF snapshot with hash signature. ⚠️ (PDF generation exists, no hash)
35. GDPR compliance for client data. ❌

**Current Status**: Audit history implemented. PDF generation functional. No compliance features.

### 3.10 Validation & Error Handling
36. Block send if: missing email, zero items, invalid taxes, or unapproved quote. ⚠️ (Frontend validation exists, partial backend)
37. Enforce rounding strategy. ❌

**Current Status**: Basic Zod validation on API. Error messages implemented. No explicit rounding strategy.


---

## 4. Non-Functional Requirements
- **Uptime**: 99.9% for acceptance endpoint. ❌ (No acceptance endpoint)
- **Performance**: <200ms p95 for calculation API. ⚠️ (Not measured)
- **Security**: HMAC tokens, scoped access, short TTL. ❌ (JWT auth only, no client tokens)
- **Observability**: Structured logs, metrics (views, accepts). ⚠️ (Console logs only)
- **Accessibility**: WCAG 2.1 AA for client UI. ⚠️ (Accessibility hooks exist, no client UI)

**Current Status**: Basic security via JWT. Logging present but not structured. No client-facing features to test accessibility.

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

**Implementation Status**: 
- ✅ Fields: id, number, status, issueDate, validUntil, client
- ❌ type field (no quote vs estimate distinction)
- ❌ currency (exists but hardcoded to USD)
- ❌ seller object (not in schema)
- ❌ lineType on items
- ⚠️ taxes (single tax field, not array)
- ⚠️ totals (subtotal, tax, total; no discounts breakdown)

**Database Schema**: Quote table with clientId, projectId, items in QuoteItem table. See `schema.prisma` lines 266-348.

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

**Implementation Status**: ⚠️ Partially implemented
- ✅ Subtotal calculation from items
- ✅ Tax calculation (via TaxService integration)
- ❌ Discount application
- ❌ Contingency (no estimate type)
- ❌ Optional item handling
- **Location**: `apps/backend/src/services/quoteService.ts` lines 485-530

**Current Logic**:
```typescript
subtotal = sum(item.quantity * item.unitPrice)
tax = calculated via TaxService (jurisdiction-based)
total = subtotal + tax
```

---

## 7. API Endpoints
- `POST /api/quotes` – create ✅
- `PATCH /api/quotes/:id` – update ✅
- `POST /api/quotes/:id/send` – email/share ❌
- `POST /api/quotes/:id/accept` – client acceptance ❌
- `POST /api/quotes/:id/decline` – client decline ❌
- `POST /api/quotes/:id/convert` – convert to invoice ❌
- `POST /api/quotes/:id/remind` – send reminder ❌
- `GET /api/quotes/:id/pdf` – get signed snapshot ⚠️ (PDF exists, no signature/hash)

**Additional Implemented Endpoints**:
- `GET /api/quotes` – list with filters ✅
- `GET /api/quotes/:id` – get single quote ✅
- `POST /api/quotes/:id/status` – update status ✅
- `DELETE /api/quotes/:id` – delete (draft only) ✅

**Location**: `apps/backend/src/routes/quotes.ts`

**Missing Endpoints**: send, accept, decline, convert, remind

---

## 8. Automation Tasks
- **Nightly expiry check:** mark expired quotes. ❌
- **Reminder scheduler:** trigger notifications. ❌
- **Post-acceptance hooks:** invoice generation, CRM sync. ❌

**Current Status**: No automation infrastructure for quotes. Report scheduling exists in system but not used for quotes.

---

## 9. Validation Rules
- Client email valid. ⚠️ (Client exists in DB, email not validated on quote creation)
- ≥1 billable item. ⚠️ (Frontend validates, backend allows empty)
- Tax rate 0–1. ⚠️ (Frontend validates)
- Validity date > issue date. ⚠️ (Not enforced)
- Error messages:
  - "Add at least one billable item." ⚠️ (Frontend only)
  - "Set a valid expiry date." ⚠️ (Frontend only)
  - "Invalid tax configuration." ❌

**Location**: 
- Frontend: `apps/frontend/src/components/quotes/QuoteForm.tsx` (Zod schema)
- Backend: `apps/backend/src/routes/quotes.ts` (Zod validation)

**Status**: Basic validation exists, needs strengthening on backend.

---

## 10. Test Coverage
- Totals with/without optional items. ❌
- Multi-tax compound validation. ❌
- Expiry auto-transition. ❌
- Full acceptance → conversion flow. ❌

**Current Status**: No test files found for quote functionality.

---

## 11. Integration Points
- **CRM (HubSpot)**: log activity, close deal on acceptance. ❌
- **Accounting (Stripe/QuickBooks)**: push invoices. ❌
- **Notifications**: Slack + email triggers. ❌

**Current Status**: No external integrations implemented for quotes.

---

## 12. Acceptance Criteria
- ⚠️ Create/edit/send flows follow schema. (Create/edit work, send not implemented)
- ⚠️ Totals match deterministic calculation rules. (Basic calculation only)
- ❌ Acceptance locks document; versioned reissues. (No acceptance, versioning exists)
- ❌ Expiry handled automatically.
- ⚠️ PDF snapshot generated and hashed. (PDF yes, hash no)

**Overall Status**: Core CRUD operations functional. Client-facing features and automation missing.

---

## 13. AI Agent Prompts
**Code Generation Prompt:**
> Implement the Quotes/Estimates API per the schema and endpoints. Use TypeScript + Express + Prisma. Validate inputs with Zod and return computed totals. Include versioning, audit logging, and OpenAPI documentation.

**Review Prompt:**
> Review implementation for compliance with spec. Verify access control, rounding, tax logic, version immutability, and API response consistency.

---

**File Version:** v1.1 – Quotes & Estimates AI Agent Spec (with Implementation Review)  
**Last Updated:** 2025-10-16  
**Review Date:** 2025-10-16

---

## IMPLEMENTATION SUMMARY

### What's Implemented ✅

**Core Features:**
- Quote CRUD operations (Create, Read, Update, Delete)
- Quote number generation (pattern: Q{YY}{MM}{NNNN})
- Version tracking and history (QuoteHistory table)
- Status management (DRAFT, INTERNAL_REVIEW, SENT, ACCEPTED, REJECTED)
- Basic calculation (subtotal, tax via TaxService, total)
- PDF generation
- Line items with grouping support
- Template system (QuoteTemplate + UserQuoteTemplate)
- Client and project associations
- 30-day default validity period

**Technical Stack:**
- Backend: TypeScript + Express + Prisma ✅
- Validation: Zod schemas ✅
- Database: PostgreSQL with proper relations ✅
- Frontend: React + MUI with quote forms ✅

**API Endpoints:**
- `POST /api/quotes` - Create quote
- `GET /api/quotes` - List quotes with filters
- `GET /api/quotes/:id` - Get single quote
- `PATCH /api/quotes/:id` - Update quote
- `POST /api/quotes/:id/status` - Update status
- `DELETE /api/quotes/:id` - Delete draft quotes
- `GET /api/quotes/:id/pdf` - Generate PDF

### What's Partially Implemented ⚠️

- **Tax Calculation**: TaxService integration exists, but multi-rate/jurisdiction not fully leveraged
- **Currency**: Field exists, hardcoded to USD
- **Discounts**: Field on items, not applied in calculations
- **Compound Tax**: Flag exists, not used in calculation
- **Validation**: Frontend validates, backend needs strengthening
- **Quote Numbering**: Pattern differs from spec (no version suffix)

### What's Missing ❌

**Critical Features:**
- Estimate type (only Quote supported)
- Line item types (standard, optional, discount, fee)
- Optional items toggleable by client
- Contingency percentage for estimates
- Client-facing features (secure links, acceptance, decline)
- Email/send functionality
- Quote to invoice conversion endpoint
- Approval workflow
- Client signature capture
- Expiry automation
- Reminder system
- Notifications (Slack, email)
- External integrations (CRM, accounting)
- Seller/company information in schema
- PDF hash/signature
- Localization
- Attachments
- GDPR compliance features
- Test coverage

**Data Model Gaps:**
- No `type` field (quote vs estimate)
- No `lineType` on items
- No `seller` object
- No multi-tax array (single tax field)
- No explicit discount/fee tracking in totals
- Payment terms not in main Quote schema (only in templates)

**Missing Endpoints:**
- `POST /api/quotes/:id/send`
- `POST /api/quotes/:id/accept`
- `POST /api/quotes/:id/decline`
- `POST /api/quotes/:id/convert`
- `POST /api/quotes/:id/remind`

### Priority Recommendations

**High Priority (Core Spec Compliance):**
1. Implement line item types (standard, optional, discount, fee)
2. Add discount and fee calculation logic
3. Implement quote-to-invoice conversion
4. Add estimate type support with contingency
5. Strengthen backend validation
6. Add PDF hash/signature for compliance

**Medium Priority (Client Experience):**
1. Secure client share links with tokens
2. Client acceptance/decline endpoints
3. Email send functionality
4. Expiry automation
5. Optional item selection by client

**Low Priority (Nice to Have):**
1. Reminder scheduler
2. External integrations (CRM, accounting)
3. Multi-currency support
4. Localization
5. Automated testing

### Files to Review/Update

**Backend:**
- `apps/backend/prisma/schema.prisma` - Add type, lineType, seller fields
- `apps/backend/src/services/quoteService.ts` - Enhanced calculation logic
- `apps/backend/src/routes/quotes.ts` - Add missing endpoints
- `packages/shared-types/quotes/types.ts` - Update TypeScript types

**Frontend:**
- `apps/frontend/src/components/quotes/QuoteForm.tsx` - Support new fields
- `apps/frontend/src/api/quotes/index.ts` - Add new API methods
- `apps/frontend/src/pages/QuoteDetailPage.tsx` - Add send/convert actions

**New Files Needed:**
- `apps/backend/src/routes/quotes-client.ts` - Client-facing endpoints
- `apps/backend/src/services/quoteNotificationService.ts` - Email/notifications
- `apps/backend/src/services/quoteAutomationService.ts` - Expiry/reminders
- Test files for all quote functionality

---