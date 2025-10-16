# Quotes - Partially Implemented Features Completion Spec

> **Document Type**: Implementation Guide  
> **Priority**: High  
> **Created**: 2025-10-16  
> **Status**: Active

## Overview

This document outlines features that are partially implemented in the Quote system and provides detailed specifications for completing them. These features have foundational work done but need additional implementation to meet the full specification requirements.

---

## 1. Tax Calculation Enhancement

### Current State
- ✅ TaxService integration exists
- ✅ Single tax rate calculated
- ⚠️ Compound tax field exists but not utilized
- ⚠️ Multi-jurisdiction capability not leveraged

### What's Missing
- Multi-rate tax calculation
- Compound tax computation
- Tax jurisdiction selection
- Tax breakdown display

### Implementation Tasks

#### 1.1 Update Quote Schema
**File**: `apps/backend/prisma/schema.prisma`

```prisma
model Quote {
  // ... existing fields
  taxDetails       Json?           // Store array of tax applications
  
  // Add new fields
  jurisdiction     String?         // Tax jurisdiction (e.g., "ON", "CA", "NY")
  compoundTax      Boolean         @default(false)
}
```

#### 1.2 Enhance Quote Service Tax Calculation
**File**: `apps/backend/src/services/quoteService.ts`

```typescript
interface TaxLineItem {
  code: string;
  label: string;
  rate: number;
  taxableAmount: number;
  taxAmount: number;
  isCompound: boolean;
}

private async calculateQuoteTotals(
  data: Record<string, any>, 
  userId: number, 
  clientId?: number
): Promise<QuoteTotals> {
  const items = (data.items || []) as LineItem[];
  
  // Calculate subtotal
  const subtotal = items.reduce((sum, item) => 
    sum + (item.quantity * item.unitPrice), 0
  );

  // Get multi-rate tax calculation
  let taxDetails: TaxLineItem[] = [];
  let totalTax = 0;
  
  if (clientId) {
    const taxCalc = await this.taxService.calculateTax({
      userId,
      clientId,
      lineItems: items.map((item, idx) => ({
        id: idx + 1,
        description: item.description,
        quantity: item.quantity,
        unitPrice: item.unitPrice,
        amount: item.quantity * item.unitPrice,
        taxable: item.taxable || true,
        category: 'service'
      })),
      overrides: {}
    });

    taxDetails = taxCalc.appliedTaxes.map(tax => ({
      code: tax.taxType,
      label: tax.taxLabel || tax.taxType,
      rate: tax.rate,
      taxableAmount: tax.taxableAmount,
      taxAmount: tax.amount,
      isCompound: tax.isCompound || false
    }));

    totalTax = taxCalc.totalTaxAmount;
  }

  return {
    subtotal,
    tax: totalTax,
    total: subtotal + totalTax,
    items: processedItems,
    taxDetails // Include for storage
  };
}
```

#### 1.3 Update Frontend to Display Tax Breakdown
**File**: `apps/frontend/src/components/quotes/QuoteForm.tsx`

```typescript
// Add tax breakdown display section
<Box sx={{ mt: 3, p: 2, bgcolor: 'grey.50', borderRadius: 1 }}>
  <Typography variant="subtitle2" gutterBottom>Tax Breakdown</Typography>
  {taxBreakdown.map((tax, idx) => (
    <Box key={idx} sx={{ display: 'flex', justifyContent: 'space-between', mb: 1 }}>
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
```

### Acceptance Criteria
- [ ] Quote can have multiple tax rates applied
- [ ] Compound taxes calculated correctly (tax on tax)
- [ ] Tax jurisdiction stored and displayed
- [ ] Tax breakdown shows each tax component
- [ ] PDF includes detailed tax breakdown

---

## 2. Discount Calculation

### Current State
- ✅ Discount field exists on QuoteItemBase interface
- ❌ Discount not applied in calculation
- ❌ No header-level discount support

### What's Missing
- Line-level discount application
- Header-level discount
- Discount types (percentage vs fixed)
- Discount display on quotes

### Implementation Tasks

#### 2.1 Update Schema for Header Discount
**File**: `apps/backend/prisma/schema.prisma`

```prisma
model Quote {
  // ... existing fields
  
  // Add discount fields
  discountType     String?         // "PERCENTAGE" | "FIXED"
  discountValue    Float?          @default(0)
  discountAmount   Float?          @default(0)
  
  // Update totals
  subtotal         Float
  discounts        Float           @default(0)
  fees             Float           @default(0)
  tax              Float
  total            Float
}
```

#### 2.2 Enhance Calculation Logic
**File**: `apps/backend/src/services/quoteService.ts`

```typescript
interface QuoteTotals {
  subtotal: number;
  lineDiscounts: number;
  headerDiscount: number;
  totalDiscounts: number;
  fees: number;
  tax: number;
  total: number;
  items: LineItem[];
  taxDetails?: any;
}

private async calculateQuoteTotals(
  data: Record<string, any>, 
  userId: number, 
  clientId?: number
): Promise<QuoteTotals> {
  const items = (data.items || []) as LineItem[];

  // Calculate line item totals and discounts
  const processedItems = items.map(item => {
    const baseTotal = item.quantity * item.unitPrice;
    let discountAmount = 0;

    if (item.discount) {
      if (item.discount.type === 'percentage') {
        discountAmount = baseTotal * (item.discount.value / 100);
      } else {
        discountAmount = item.discount.value;
      }
    }

    return {
      ...item,
      total: baseTotal - discountAmount,
      discountAmount,
      baseTotal
    };
  });

  // Calculate subtotal (after line discounts)
  const subtotal = processedItems.reduce((sum, item) => sum + item.total, 0);
  const lineDiscounts = processedItems.reduce((sum, item) => sum + (item.discountAmount || 0), 0);

  // Calculate header-level discount
  let headerDiscount = 0;
  if (data.discount) {
    if (data.discount.type === 'percentage') {
      headerDiscount = subtotal * (data.discount.value / 100);
    } else {
      headerDiscount = data.discount.value;
    }
  }

  const subtotalAfterDiscounts = subtotal - headerDiscount;

  // Calculate fees (if any)
  const fees = data.fees || 0;

  // Calculate tax on discounted amount
  const taxableAmount = subtotalAfterDiscounts + fees;
  const tax = await this.calculateTax(taxableAmount, clientId, userId);

  const total = taxableAmount + tax;

  return {
    subtotal: processedItems.reduce((sum, item) => sum + item.baseTotal, 0),
    lineDiscounts,
    headerDiscount,
    totalDiscounts: lineDiscounts + headerDiscount,
    fees,
    tax,
    total,
    items: processedItems
  };
}
```

#### 2.3 Update Frontend Discount UI
**File**: `apps/frontend/src/components/quotes/QuoteForm.tsx`

Add discount configuration section:

```typescript
// Add to form state
const [headerDiscount, setHeaderDiscount] = useState({
  type: 'percentage' as 'percentage' | 'fixed',
  value: 0
});

// Add UI section
<Box sx={{ mt: 2 }}>
  <Typography variant="subtitle2" gutterBottom>Quote Discount</Typography>
  <Grid container spacing={2}>
    <Grid item xs={6}>
      <FormControl fullWidth>
        <InputLabel>Discount Type</InputLabel>
        <Select
          value={headerDiscount.type}
          onChange={(e) => setHeaderDiscount({...headerDiscount, type: e.target.value as any})}
        >
          <MenuItem value="percentage">Percentage</MenuItem>
          <MenuItem value="fixed">Fixed Amount</MenuItem>
        </Select>
      </FormControl>
    </Grid>
    <Grid item xs={6}>
      <TextField
        label={headerDiscount.type === 'percentage' ? 'Percentage (%)' : 'Amount ($)'}
        type="number"
        value={headerDiscount.value}
        onChange={(e) => setHeaderDiscount({...headerDiscount, value: parseFloat(e.target.value)})}
        fullWidth
      />
    </Grid>
  </Grid>
</Box>
```

### Acceptance Criteria
- [ ] Line items can have individual discounts (percentage or fixed)
- [ ] Header-level discount can be applied to entire quote
- [ ] Discounts calculated before tax
- [ ] Total discounts displayed separately in breakdown
- [ ] PDF shows discount details

---

## 3. Currency Support

### Current State
- ✅ Currency field exists in types
- ⚠️ Hardcoded to USD in implementation
- ❌ No currency selection or conversion

### What's Missing
- Currency selection in UI
- Multi-currency support
- Exchange rate handling
- Currency display formatting

### Implementation Tasks

#### 3.1 Add Currency to Quote Schema
**File**: `apps/backend/prisma/schema.prisma`

```prisma
model Quote {
  // ... existing fields
  currency         String          @default("USD")
  exchangeRate     Float?          @default(1.0)
}
```

#### 3.2 Update Quote Service
**File**: `apps/backend/src/services/quoteService.ts`

```typescript
async createQuote(userId: number, input: CreateQuoteInput): Promise<Quote> {
  // ... existing validation

  // Add currency handling
  const currency = input.data.currency?.code || 'USD';
  const exchangeRate = input.data.currency?.rate || 1.0;

  return this.db.$transaction(async (tx: PrismaTransaction) => {
    const [createdQuote] = await tx.$queryRaw<[Quote & { id: number }]>`
      INSERT INTO "Quote" (
        number, version, status, "templateType",
        "clientId", "projectId", "createdById", subtotal, tax,
        total, "validUntil", notes, currency, "exchangeRate", "updatedAt"
      ) VALUES (
        ${number}, 1, ${QuoteStatus.DRAFT}, ${input.templateType},
        ${input.clientId}, ${input.projectId || null}, ${userId}, ${subtotal}, ${tax},
        ${total}, ${input.validUntil || new Date(Date.now() + 30 * 24 * 60 * 60 * 1000)}, 
        ${input.notes || ''}, ${currency}, ${exchangeRate}, ${new Date()}
      )
      RETURNING *
    `;
    
    // ... rest of creation logic
  });
}
```

#### 3.3 Add Currency Selector to Frontend
**File**: `apps/frontend/src/components/quotes/QuoteForm.tsx`

```typescript
const SUPPORTED_CURRENCIES = [
  { code: 'USD', symbol: '$', name: 'US Dollar' },
  { code: 'CAD', symbol: '$', name: 'Canadian Dollar' },
  { code: 'EUR', symbol: '€', name: 'Euro' },
  { code: 'GBP', symbol: '£', name: 'British Pound' },
];

// Add to form
<FormControl fullWidth>
  <InputLabel>Currency</InputLabel>
  <Select
    value={selectedCurrency}
    onChange={(e) => setSelectedCurrency(e.target.value)}
  >
    {SUPPORTED_CURRENCIES.map(curr => (
      <MenuItem key={curr.code} value={curr.code}>
        {curr.code} - {curr.name} ({curr.symbol})
      </MenuItem>
    ))}
  </Select>
</FormControl>
```

### Acceptance Criteria
- [ ] User can select currency when creating quote
- [ ] Currency symbol displayed correctly throughout quote
- [ ] Exchange rate stored for historical accuracy
- [ ] PDF shows currency code and symbol
- [ ] Reports can filter by currency

---

## 4. Quote Number Format

### Current State
- ✅ Auto-generation implemented
- ⚠️ Format differs from spec: `Q{YY}{MM}{NNNN}` vs spec `Q-{YYYY}-{SEQ}-vN`
- ❌ No version suffix in number

### What's Missing
- Spec-compliant format
- Version suffix support
- Configurable format

### Implementation Tasks

#### 4.1 Update Number Generation Logic
**File**: `apps/backend/src/services/quoteService.ts`

```typescript
private async generateQuoteNumber(version: number = 1): Promise<string> {
  const now = new Date();
  const year = now.getFullYear();

  // Get latest quote number for this year
  const latestQuote = await this.db.$queryRaw<Array<{ number: string }>>`
    SELECT number FROM "Quote"
    WHERE number LIKE ${`Q-${year}-%`}
    ORDER BY number DESC
    LIMIT 1
  `;

  let sequence = 1;
  if (latestQuote.length > 0) {
    // Extract sequence from Q-2025-0001-v1 format
    const match = latestQuote[0].number.match(/Q-\d{4}-(\d+)-v\d+/);
    if (match) {
      sequence = parseInt(match[1]) + 1;
    }
  }

  const sequenceStr = String(sequence).padStart(4, '0');
  return `Q-${year}-${sequenceStr}-v${version}`;
}
```

#### 4.2 Handle Versioning on Updates
**File**: `apps/backend/src/services/quoteService.ts`

```typescript
async updateQuote(quoteId: number, userId: number, input: UpdateQuoteInput): Promise<Quote> {
  const quote = await this.getQuote(quoteId);
  if (!quote) {
    throw new Error('Quote not found');
  }

  // Increment version
  const newVersion = quote.version + 1;
  
  // Generate new version number if needed (for major changes)
  const shouldGenerateNewNumber = input.data && JSON.stringify(input.data) !== JSON.stringify(quote.history[0].changes);
  
  const updateData: any = {
    version: { increment: 1 },
    // ... other fields
  };

  if (shouldGenerateNewNumber) {
    const baseNumber = quote.number.replace(/-v\d+$/, '');
    updateData.number = `${baseNumber}-v${newVersion}`;
  }

  // ... rest of update logic
}
```

### Acceptance Criteria
- [ ] Quote numbers follow format `Q-YYYY-NNNN-vN`
- [ ] Sequence increments correctly
- [ ] Version increments on significant changes
- [ ] Old quote numbers remain valid (migration needed)
- [ ] Number format configurable per account

---

## 5. Validation Strengthening

### Current State
- ✅ Frontend Zod validation
- ⚠️ Backend validation partial
- ❌ No date validation
- ❌ No business rule validation

### What's Missing
- Comprehensive backend validation
- Date range validation
- Business rule enforcement
- Consistent error messages

### Implementation Tasks

#### 5.1 Create Comprehensive Validation Schema
**File**: `apps/backend/src/routes/quotes.ts`

```typescript
const createQuoteSchema = z.object({
  title: z.string().min(1, 'Title is required').max(200, 'Title too long'),
  clientId: z.union([z.string(), z.number()])
    .transform(val => typeof val === 'string' ? parseInt(val) : val)
    .refine(val => val > 0, 'Valid client required'),
  projectId: z.union([z.string(), z.number()])
    .transform(val => typeof val === 'string' ? parseInt(val) : val)
    .optional(),
  items: z.array(z.object({
    description: z.string().min(1, 'Item description required'),
    quantity: z.number().positive('Quantity must be positive'),
    unitPrice: z.number().min(0, 'Unit price cannot be negative'),
    taxable: z.boolean().optional().default(true),
    discount: z.object({
      type: z.enum(['percentage', 'fixed']),
      value: z.number().min(0)
    }).optional(),
    groupName: z.string().optional(),
    groupType: z.string().optional(),
    groupOrder: z.number().optional()
  }))
    .min(1, 'At least one line item required')
    .refine(items => items.some(item => item.quantity * item.unitPrice > 0), 
      'At least one billable item required'),
  validUntil: z.string()
    .datetime()
    .refine(date => new Date(date) > new Date(), 
      'Valid until date must be in the future'),
  currency: z.object({
    code: z.string().length(3, 'Currency code must be 3 characters'),
    symbol: z.string(),
    rate: z.number().positive().optional().default(1.0)
  }).optional(),
  taxSettings: z.object({
    taxRate: z.number().min(0).max(1, 'Tax rate must be between 0 and 1'),
    taxLabel: z.string().optional(),
    isCompoundTax: z.boolean().optional()
  }).optional(),
  discount: z.object({
    type: z.enum(['percentage', 'fixed']),
    value: z.number().min(0).refine(
      (val, ctx) => {
        if (ctx.type === 'percentage') return val <= 100;
        return true;
      },
      'Percentage discount cannot exceed 100%'
    )
  }).optional(),
  notes: z.string().max(5000, 'Notes too long').optional()
});
```

#### 5.2 Add Business Rule Validation
**File**: `apps/backend/src/services/quoteService.ts`

```typescript
private async validateQuoteBusinessRules(input: CreateQuoteInput): Promise<void> {
  // Check client exists and is active
  const client = await this.db.client.findUnique({
    where: { id: input.clientId }
  });
  
  if (!client) {
    throw new Error('Client not found');
  }

  // Check project exists if provided
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

  // Validate validity period
  const validUntil = new Date(input.validUntil || Date.now() + 30 * 24 * 60 * 60 * 1000);
  const now = new Date();
  
  if (validUntil <= now) {
    throw new Error('Valid until date must be in the future');
  }

  const maxValidityDays = 90;
  const daysDiff = Math.ceil((validUntil.getTime() - now.getTime()) / (1000 * 60 * 60 * 24));
  
  if (daysDiff > maxValidityDays) {
    throw new Error(`Quote validity cannot exceed ${maxValidityDays} days`);
  }

  // Validate totals are reasonable
  const { subtotal, total } = await this.calculateQuoteTotals(input.data, input.clientId);
  
  if (total <= 0) {
    throw new Error('Quote total must be greater than zero');
  }

  if (total > 10000000) { // $10M limit
    throw new Error('Quote total exceeds maximum allowed amount');
  }
}

async createQuote(userId: number, input: CreateQuoteInput): Promise<Quote> {
  // Validate business rules
  await this.validateQuoteBusinessRules(input);
  
  // ... rest of creation logic
}
```

### Acceptance Criteria
- [ ] All required fields validated on backend
- [ ] Date ranges validated (validUntil > today)
- [ ] Business rules enforced (client exists, project belongs to client)
- [ ] Consistent error messages returned
- [ ] Validation errors include field names
- [ ] Frontend displays validation errors clearly

---

## Testing Requirements

For each partially implemented feature, create tests:

### Unit Tests
```typescript
// File: apps/backend/src/services/__tests__/quoteService.test.ts

describe('QuoteService - Tax Calculation', () => {
  it('should calculate multi-rate taxes correctly', async () => {
    // Test implementation
  });

  it('should handle compound taxes', async () => {
    // Test implementation
  });
});

describe('QuoteService - Discounts', () => {
  it('should apply line-level percentage discount', async () => {
    // Test implementation
  });

  it('should apply header-level fixed discount', async () => {
    // Test implementation
  });

  it('should apply both line and header discounts', async () => {
    // Test implementation
  });
});

describe('QuoteService - Validation', () => {
  it('should reject quotes with invalid dates', async () => {
    // Test implementation
  });

  it('should reject quotes with non-existent clients', async () => {
    // Test implementation
  });
});
```

### Integration Tests
```typescript
// File: apps/backend/src/routes/__tests__/quotes.integration.test.ts

describe('POST /api/quotes', () => {
  it('should create quote with multi-currency', async () => {
    // Test implementation
  });

  it('should return proper validation errors', async () => {
    // Test implementation
  });
});
```

---

## Migration Plan

### Database Migration
Create migration for new fields:

```sql
-- File: apps/backend/prisma/migrations/YYYYMMDDHHMMSS_enhance_quotes/migration.sql

-- Add currency fields
ALTER TABLE "Quote" ADD COLUMN "currency" TEXT DEFAULT 'USD';
ALTER TABLE "Quote" ADD COLUMN "exchangeRate" DOUBLE PRECISION DEFAULT 1.0;

-- Add discount fields
ALTER TABLE "Quote" ADD COLUMN "discountType" TEXT;
ALTER TABLE "Quote" ADD COLUMN "discountValue" DOUBLE PRECISION DEFAULT 0;
ALTER TABLE "Quote" ADD COLUMN "discountAmount" DOUBLE PRECISION DEFAULT 0;
ALTER TABLE "Quote" ADD COLUMN "discounts" DOUBLE PRECISION DEFAULT 0;
ALTER TABLE "Quote" ADD COLUMN "fees" DOUBLE PRECISION DEFAULT 0;

-- Add jurisdiction
ALTER TABLE "Quote" ADD COLUMN "jurisdiction" TEXT;
ALTER TABLE "Quote" ADD COLUMN "compoundTax" BOOLEAN DEFAULT false;

-- Update existing quote numbers to new format (if needed)
-- This should be done carefully with a data migration script
```

---

## Rollout Plan

1. **Phase 1: Backend Updates** (Week 1)
   - Update database schema
   - Implement enhanced calculations
   - Add validation
   - Write unit tests

2. **Phase 2: API Enhancement** (Week 1)
   - Update API endpoints
   - Add new validation
   - Update documentation
   - Write integration tests

3. **Phase 3: Frontend Updates** (Week 2)
   - Add currency selector
   - Add discount UI
   - Add tax breakdown display
   - Update forms and validation

4. **Phase 4: Testing & QA** (Week 2)
   - End-to-end testing
   - User acceptance testing
   - Performance testing
   - Bug fixes

5. **Phase 5: Deployment** (Week 3)
   - Deploy to staging
   - Smoke testing
   - Deploy to production
   - Monitor

---

## Success Metrics

- [ ] All 5 partially implemented features completed
- [ ] Unit test coverage > 80%
- [ ] Integration test coverage > 70%
- [ ] All acceptance criteria met
- [ ] No regression in existing functionality
- [ ] Documentation updated
- [ ] User training materials created

---

## Dependencies

- TaxService must support multi-rate calculations
- Database migration must be tested thoroughly
- Frontend components must handle new data structures
- PDF generation must be updated for new fields

---

## Risk Mitigation

**Risk**: Breaking existing quotes
- **Mitigation**: Comprehensive migration script with rollback plan, test on staging first

**Risk**: Performance degradation with complex tax calculations
- **Mitigation**: Profile and optimize, consider caching, index jurisdiction field

**Risk**: User confusion with new features
- **Mitigation**: Tooltips, help text, training videos, gradual rollout

---

**Document Status**: Ready for Implementation  
**Estimated Effort**: 3 weeks  
**Team Size**: 2-3 developers
