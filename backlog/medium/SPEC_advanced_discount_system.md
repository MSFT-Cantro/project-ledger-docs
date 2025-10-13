# Advanced Discount System Specification

**Date:** October 12, 2025  
**Version:** 1.0  
**Status:** Not Implemented  

---

## 1. Overview

The Advanced Discount System provides comprehensive discount management capabilities including header-level discounts, line-item discounts, conditional discounts, and promotional discount codes. It supports percentage and fixed-amount discounts with complex business rules and approval workflows.

---

## 2. Business Requirements

### 2.1 Core Functionality
- **Header-level Discounts**: Apply discounts to entire documents (quotes/invoices)
- **Line-item Discounts**: Individual item-level discount application
- **Discount Codes**: Promotional codes with usage limits and expiration
- **Conditional Discounts**: Volume-based, client-specific, or time-based discounts
- **Approval Workflows**: Require approval for discounts above threshold
- **Discount Stacking**: Rules for combining multiple discounts

### 2.2 Discount Types
- **Percentage Discounts**: 5%, 10%, 15% off subtotal or line items
- **Fixed Amount Discounts**: $100, $500 off total or specific items
- **Volume Discounts**: Tiered pricing based on quantity or total amount
- **Early Payment Discounts**: 2/10 net 30 payment terms
- **Client-specific Discounts**: Custom pricing for preferred clients
- **Seasonal/Promotional**: Time-limited discount campaigns

### 2.3 Business Rules
- Discounts cannot exceed 100% of item/document value
- Maximum discount limits per user role and document type
- Discount approval requirements based on amount and type
- Tax calculations apply after discount deductions
- Discount audit trail for compliance and reporting
- Integration with existing quote and invoice totals calculation

---

## 3. Data Model

### 3.1 Database Schema

```prisma
model Discount {
  id                  Int                   @id @default(autoincrement())
  code                String?               @unique // Optional discount code
  name                String                // Internal name
  description         String?               // User-facing description
  
  // Discount configuration
  discountType        DiscountType
  valueType           DiscountValueType     // PERCENTAGE or FIXED_AMOUNT
  value               Float                 // Percentage (0-100) or fixed amount
  
  // Application scope
  applicationType     DiscountApplication   // HEADER, LINE_ITEM, or BOTH
  maxUsage            Int?                  // Usage limit (null = unlimited)
  currentUsage        Int                   @default(0)
  
  // Validity
  startDate           DateTime?
  endDate             DateTime?
  isActive            Boolean               @default(true)
  
  // Conditions
  minOrderAmount      Float?                // Minimum order value
  maxOrderAmount      Float?                // Maximum order value
  applicableClients   Json?                 // Array of client IDs
  applicableProducts  Json?                 // Array of product/service types
  
  // Approval requirements
  requiresApproval    Boolean               @default(false)
  approvalThreshold   Float?                // Amount requiring approval
  maxDiscountPercent  Float?                // Maximum discount percentage
  
  // Organization context
  accountId           Int
  createdBy           Int
  createdAt           DateTime              @default(now())
  updatedAt           DateTime              @updatedAt
  
  // Relations
  account             Account               @relation(fields: [accountId], references: [id])
  createdByUser       User                  @relation(fields: [createdBy], references: [id])
  applications        DiscountApplication[]
  
  @@index([accountId])
  @@index([code])
  @@index([discountType])
  @@index([startDate, endDate])
}

model DiscountApplication {
  id                  Int                   @id @default(autoincrement())
  discountId          Int
  
  // Document reference
  documentType        DocumentType          // QUOTE, INVOICE, CHANGE_ORDER
  documentId          Int                   // Reference to document
  lineItemId          Int?                  // Specific line item (if applicable)
  
  // Application details
  originalValue       Float                 // Value before discount
  discountAmount      Float                 // Actual discount applied
  finalValue          Float                 // Value after discount
  discountPercent     Float?                // Effective discount percentage
  
  // Approval tracking
  requiresApproval    Boolean               @default(false)
  approvedAt          DateTime?
  approvedBy          Int?
  approvalNotes       String?
  rejectedAt          DateTime?
  rejectedBy          Int?
  rejectionReason     String?
  
  // Metadata
  appliedAt           DateTime              @default(now())
  appliedBy           Int
  notes               String?
  
  // Relations
  discount            Discount              @relation(fields: [discountId], references: [id])
  appliedByUser       User                  @relation("DiscountAppliedBy", fields: [appliedBy], references: [id])
  approvedByUser      User?                 @relation("DiscountApprovedBy", fields: [approvedBy], references: [id])
  rejectedByUser      User?                 @relation("DiscountRejectedBy", fields: [rejectedBy], references: [id])
  
  @@index([discountId])
  @@index([documentType, documentId])
  @@index([appliedAt])
}

model DiscountRule {
  id                  Int                   @id @default(autoincrement())
  discountId          Int
  
  // Rule definition
  ruleType            DiscountRuleType      // VOLUME, CLIENT_TIER, TIME_BASED, etc.
  condition           Json                  // Rule conditions (flexible JSON)
  value               Float                 // Discount value for this rule
  priority            Int                   @default(0) // Rule application order
  
  // Metadata
  isActive            Boolean               @default(true)
  createdAt           DateTime              @default(now())
  
  // Relations
  discount            Discount              @relation(fields: [discountId], references: [id], onDelete: Cascade)
  
  @@index([discountId])
  @@index([ruleType])
}

model DiscountSettings {
  id                    Int                 @id @default(autoincrement())
  accountId             Int                 @unique
  
  // Global discount limits
  maxDiscountPercent    Float               @default(50.0) // Maximum discount percentage
  maxDiscountAmount     Float?              // Maximum fixed discount amount
  
  // Approval requirements
  approvalRequired      Boolean             @default(true)
  approvalThreshold     Float               @default(1000.0) // Amount requiring approval
  discountApprovers     Json                // Array of user IDs who can approve
  
  // Stacking rules
  allowDiscountStacking Boolean             @default(false)
  maxStackedDiscounts   Int                 @default(3)
  stackingMethod        DiscountStackMethod @default(SEQUENTIAL)
  
  // Default settings
  defaultDiscountType   DiscountType        @default(PERCENTAGE)
  showDiscountOnPDF     Boolean             @default(true)
  requireDiscountReason Boolean             @default(false)
  
  // Audit settings
  logAllDiscountUsage   Boolean             @default(true)
  discountReportingEnabled Boolean          @default(true)
  
  createdAt             DateTime            @default(now())
  updatedAt             DateTime            @updatedAt
  
  // Relations
  account               Account             @relation(fields: [accountId], references: [id], onDelete: Cascade)
  
  @@index([accountId])
}

enum DiscountType {
  STANDARD          // Basic percentage or fixed discount
  VOLUME            // Quantity-based discount
  CLIENT_TIER       // Client classification discount
  EARLY_PAYMENT     // Payment term discount
  SEASONAL          // Time-based promotional discount
  LOYALTY           // Customer loyalty discount
  BULK_ORDER        // Large order discount
  FIRST_TIME        // New customer discount
  REFERRAL          // Referral program discount
  CLEARANCE         // Inventory clearance discount
}

enum DiscountValueType {
  PERCENTAGE        // Percentage-based discount (5%, 10%, etc.)
  FIXED_AMOUNT      // Fixed dollar amount ($100, $500, etc.)
}

enum DiscountApplication {
  HEADER           // Applied to entire document
  LINE_ITEM        // Applied to specific line items
  BOTH             // Can be applied at either level
}

enum DiscountRuleType {
  VOLUME_TIER      // Quantity thresholds
  AMOUNT_TIER      // Dollar amount thresholds
  CLIENT_TYPE      // Client classification
  TIME_WINDOW      // Date/time conditions
  PRODUCT_MIX      // Combination of products/services
  GEOGRAPHY        // Location-based
  CUSTOM           // Custom business logic
}

enum DiscountStackMethod {
  SEQUENTIAL       // Apply discounts one after another
  BEST_SINGLE      // Apply only the best single discount
  ADDITIVE         // Add discount percentages
  MULTIPLICATIVE   // Multiply discount effects
}

enum DocumentType {
  QUOTE
  INVOICE
  CHANGE_ORDER
}
```

### 3.2 TypeScript Interfaces

```typescript
export interface Discount {
  id: string;
  code?: string;
  name: string;
  description?: string;
  
  discountType: DiscountType;
  valueType: DiscountValueType;
  value: number;
  
  applicationType: DiscountApplication;
  maxUsage?: number;
  currentUsage: number;
  
  startDate?: string;
  endDate?: string;
  isActive: boolean;
  
  minOrderAmount?: number;
  maxOrderAmount?: number;
  applicableClients?: string[];
  applicableProducts?: string[];
  
  requiresApproval: boolean;
  approvalThreshold?: number;
  maxDiscountPercent?: number;
  
  accountId: string;
  createdBy: string;
  createdAt: string;
  updatedAt: string;
  
  applications: DiscountApplication[];
  rules: DiscountRule[];
}

export interface DiscountApplication {
  id: string;
  discountId: string;
  documentType: DocumentType;
  documentId: string;
  lineItemId?: string;
  
  originalValue: number;
  discountAmount: number;
  finalValue: number;
  discountPercent?: number;
  
  requiresApproval: boolean;
  approvedAt?: string;
  approvedBy?: string;
  approvalNotes?: string;
  
  appliedAt: string;
  appliedBy: string;
  notes?: string;
  
  discount: Discount;
}

export interface ApplyDiscountInput {
  discountId?: string;
  discountCode?: string;
  documentType: DocumentType;
  documentId: string;
  lineItemId?: string;
  customValue?: number; // For ad-hoc discounts
  reason?: string;
}

export interface DiscountCalculationResult {
  originalAmount: number;
  discountAmount: number;
  finalAmount: number;
  effectiveDiscountPercent: number;
  appliedDiscounts: DiscountApplication[];
  requiresApproval: boolean;
  warnings: string[];
}
```

---

## 4. API Endpoints

### 4.1 Discount Management

```typescript
// Create discount
POST /api/discounts
Body: CreateDiscountInput
Response: Discount

// Get discount
GET /api/discounts/:id
Response: Discount

// Update discount
PATCH /api/discounts/:id
Body: Partial<CreateDiscountInput>
Response: Discount

// Delete discount
DELETE /api/discounts/:id
Response: { success: boolean }

// List discounts
GET /api/discounts
Query: {
  active?: boolean,
  discountType?: DiscountType,
  search?: string
}
Response: Discount[]

// Validate discount code
GET /api/discounts/validate/:code
Query: {
  documentType: DocumentType,
  documentId: string,
  amount?: number
}
Response: {
  valid: boolean,
  discount?: Discount,
  applicableAmount?: number,
  message?: string
}
```

### 4.2 Discount Application

```typescript
// Apply discount
POST /api/discounts/apply
Body: ApplyDiscountInput
Response: DiscountCalculationResult

// Calculate discount preview
POST /api/discounts/calculate
Body: {
  discountId: string,
  amount: number,
  lineItems?: LineItem[]
}
Response: DiscountCalculationResult

// Remove discount application
DELETE /api/discount-applications/:id
Response: { success: boolean }

// Approve discount application
POST /api/discount-applications/:id/approve
Body: {
  approvedBy: string,
  approvalNotes?: string
}
Response: DiscountApplication

// Get discount applications for document
GET /api/documents/:type/:id/discounts
Response: DiscountApplication[]
```

### 4.3 Discount Reporting

```typescript
// Discount usage report
GET /api/discounts/reports/usage
Query: {
  dateFrom?: string,
  dateTo?: string,
  discountType?: DiscountType
}
Response: {
  totalDiscounts: number,
  totalAmount: number,
  averageDiscount: number,
  topDiscounts: Array<{
    discount: Discount,
    usage: number,
    amount: number
  }>
}

// Client discount summary
GET /api/clients/:id/discounts
Query: {
  dateFrom?: string,
  dateTo?: string
}
Response: {
  totalDiscountsReceived: number,
  totalAmountSaved: number,
  discountHistory: DiscountApplication[]
}
```

---

## 5. Business Logic Services

### 5.1 Discount Service

```typescript
export class DiscountService {
  async applyDiscount(input: ApplyDiscountInput): Promise<DiscountCalculationResult> {
    // Validate discount eligibility
    const discount = await this.validateDiscountEligibility(input);
    
    // Calculate discount amount
    const calculation = await this.calculateDiscount(discount, input);
    
    // Check approval requirements
    const requiresApproval = this.checkApprovalRequired(discount, calculation);
    
    // Apply discount if no approval needed
    if (!requiresApproval) {
      await this.recordDiscountApplication(discount, input, calculation);
    }
    
    return {
      originalAmount: calculation.originalAmount,
      discountAmount: calculation.discountAmount,
      finalAmount: calculation.finalAmount,
      effectiveDiscountPercent: (calculation.discountAmount / calculation.originalAmount) * 100,
      appliedDiscounts: calculation.applications,
      requiresApproval,
      warnings: calculation.warnings
    };
  }
  
  async calculateDiscount(
    discount: Discount,
    input: ApplyDiscountInput
  ): Promise<DiscountCalculationResult> {
    const originalAmount = await this.getDocumentAmount(input.documentType, input.documentId, input.lineItemId);
    
    let discountAmount = 0;
    
    switch (discount.valueType) {
      case 'PERCENTAGE':
        discountAmount = originalAmount * (discount.value / 100);
        break;
      case 'FIXED_AMOUNT':
        discountAmount = Math.min(discount.value, originalAmount);
        break;
    }
    
    // Apply business rules and limitations
    discountAmount = await this.applyDiscountRules(discount, discountAmount, originalAmount, input);
    
    // Check for discount stacking
    const existingDiscounts = await this.getExistingDiscounts(input.documentType, input.documentId);
    if (existingDiscounts.length > 0) {
      discountAmount = await this.handleDiscountStacking(discountAmount, existingDiscounts);
    }
    
    const finalAmount = originalAmount - discountAmount;
    
    return {
      originalAmount,
      discountAmount,
      finalAmount,
      appliedDiscounts: [],
      warnings: []
    };
  }
  
  private async applyDiscountRules(
    discount: Discount,
    discountAmount: number,
    originalAmount: number,
    input: ApplyDiscountInput
  ): Promise<number> {
    // Apply maximum discount limits
    const settings = await this.getDiscountSettings();
    
    // Percentage limit
    if (settings.maxDiscountPercent) {
      const maxAmount = originalAmount * (settings.maxDiscountPercent / 100);
      discountAmount = Math.min(discountAmount, maxAmount);
    }
    
    // Fixed amount limit
    if (settings.maxDiscountAmount) {
      discountAmount = Math.min(discountAmount, settings.maxDiscountAmount);
    }
    
    // Apply discount-specific rules
    if (discount.rules) {
      for (const rule of discount.rules) {
        discountAmount = await this.applyDiscountRule(rule, discountAmount, originalAmount, input);
      }
    }
    
    return discountAmount;
  }
  
  async validateDiscountCode(
    code: string,
    documentType: DocumentType,
    documentId: string,
    amount?: number
  ): Promise<{ valid: boolean; discount?: Discount; message?: string }> {
    const discount = await prisma.discount.findFirst({
      where: {
        code,
        isActive: true,
        OR: [
          { startDate: null },
          { startDate: { lte: new Date() } }
        ],
        AND: [
          { OR: [
            { endDate: null },
            { endDate: { gte: new Date() } }
          ]}
        ]
      }
    });
    
    if (!discount) {
      return { valid: false, message: 'Invalid or expired discount code' };
    }
    
    // Check usage limits
    if (discount.maxUsage && discount.currentUsage >= discount.maxUsage) {
      return { valid: false, message: 'Discount code usage limit exceeded' };
    }
    
    // Check minimum order amount
    if (discount.minOrderAmount && amount && amount < discount.minOrderAmount) {
      return { 
        valid: false, 
        message: `Minimum order amount of $${discount.minOrderAmount} required` 
      };
    }
    
    // Check maximum order amount
    if (discount.maxOrderAmount && amount && amount > discount.maxOrderAmount) {
      return { 
        valid: false, 
        message: `Maximum order amount of $${discount.maxOrderAmount} exceeded` 
      };
    }
    
    return { valid: true, discount };
  }
  
  async generateDiscountReport(
    accountId: string,
    dateFrom?: Date,
    dateTo?: Date
  ): Promise<DiscountReport> {
    const applications = await prisma.discountApplication.findMany({
      where: {
        discount: { accountId: parseInt(accountId) },
        appliedAt: {
          gte: dateFrom,
          lte: dateTo
        }
      },
      include: {
        discount: true
      }
    });
    
    const totalDiscounts = applications.length;
    const totalAmount = applications.reduce((sum, app) => sum + app.discountAmount, 0);
    const averageDiscount = totalAmount / totalDiscounts || 0;
    
    // Group by discount type
    const byType = applications.reduce((acc, app) => {
      const type = app.discount.discountType;
      acc[type] = (acc[type] || 0) + app.discountAmount;
      return acc;
    }, {} as Record<string, number>);
    
    return {
      totalDiscounts,
      totalAmount,
      averageDiscount,
      discountsByType: byType,
      applications
    };
  }
}
```

### 5.2 Integration with Existing Systems

```typescript
// Extend existing quote/invoice calculation services
export class EnhancedTotalsCalculator {
  async calculateDocumentTotals(
    lineItems: LineItem[],
    discountApplications: DiscountApplication[] = [],
    taxRates: TaxRate[] = []
  ): Promise<DocumentTotals> {
    // Calculate base subtotal
    const subtotal = lineItems.reduce((sum, item) => {
      return sum + (item.quantity * item.unitPrice);
    }, 0);
    
    // Apply line-item discounts
    const lineItemDiscounts = discountApplications
      .filter(app => app.lineItemId)
      .reduce((sum, app) => sum + app.discountAmount, 0);
    
    // Apply header-level discounts
    const headerDiscounts = discountApplications
      .filter(app => !app.lineItemId)
      .reduce((sum, app) => sum + app.discountAmount, 0);
    
    const totalDiscounts = lineItemDiscounts + headerDiscounts;
    const discountedSubtotal = subtotal - totalDiscounts;
    
    // Calculate tax on discounted amount
    const tax = this.calculateTax(discountedSubtotal, taxRates);
    
    const grandTotal = discountedSubtotal + tax;
    
    return {
      subtotal,
      discounts: totalDiscounts,
      discountedSubtotal,
      tax,
      grandTotal,
      appliedDiscounts: discountApplications
    };
  }
}
```

---

## 6. User Interface Components

### 6.1 Discount Management Interface

```typescript
interface DiscountManagerProps {
  onDiscountCreated: (discount: Discount) => void;
}

export function DiscountManager({ onDiscountCreated }: DiscountManagerProps) {
  const [discounts, setDiscounts] = useState<Discount[]>([]);
  const [createDialogOpen, setCreateDialogOpen] = useState(false);

  const { data: discountList } = useQuery({
    queryKey: ['discounts'],
    queryFn: () => getDiscounts()
  });

  return (
    <Container maxWidth="xl">
      <Box sx={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center', mb: 4 }}>
        <Typography variant="h4">
          Discount Management
        </Typography>
        <Button
          variant="contained"
          startIcon={<Add />}
          onClick={() => setCreateDialogOpen(true)}
        >
          Create Discount
        </Button>
      </Box>

      {/* Discount List */}
      <DataGrid
        rows={discountList || []}
        columns={[
          { field: 'name', headerName: 'Name', width: 200 },
          { field: 'code', headerName: 'Code', width: 120 },
          { field: 'discountType', headerName: 'Type', width: 120 },
          { 
            field: 'value', 
            headerName: 'Value', 
            width: 100,
            renderCell: (params) => (
              params.row.valueType === 'PERCENTAGE' 
                ? `${params.value}%` 
                : `$${params.value}`
            )
          },
          { field: 'currentUsage', headerName: 'Usage', width: 80 },
          { 
            field: 'isActive', 
            headerName: 'Status', 
            width: 100,
            renderCell: (params) => (
              <Chip 
                label={params.value ? 'Active' : 'Inactive'} 
                color={params.value ? 'success' : 'default'}
                size="small"
              />
            )
          },
          {
            field: 'actions',
            headerName: 'Actions',
            width: 120,
            renderCell: (params) => (
              <Box>
                <IconButton onClick={() => editDiscount(params.row)}>
                  <Edit />
                </IconButton>
                <IconButton onClick={() => deleteDiscount(params.row.id)} color="error">
                  <Delete />
                </IconButton>
              </Box>
            )
          }
        ]}
        pageSize={25}
        checkboxSelection
        disableSelectionOnClick
      />

      {/* Create/Edit Dialog */}
      {createDialogOpen && (
        <CreateDiscountDialog
          open={createDialogOpen}
          onClose={() => setCreateDialogOpen(false)}
          onSubmit={(discount) => {
            onDiscountCreated(discount);
            setCreateDialogOpen(false);
          }}
        />
      )}
    </Container>
  );
}
```

### 6.2 Discount Application Component

```typescript
interface DiscountSelectorProps {
  documentType: DocumentType;
  documentId: string;
  currentAmount: number;
  onDiscountApplied: (application: DiscountApplication) => void;
}

export function DiscountSelector({ 
  documentType, 
  documentId, 
  currentAmount, 
  onDiscountApplied 
}: DiscountSelectorProps) {
  const [discountCode, setDiscountCode] = useState('');
  const [selectedDiscount, setSelectedDiscount] = useState<Discount | null>(null);
  const [customDiscount, setCustomDiscount] = useState({ type: 'PERCENTAGE', value: 0 });

  const { data: availableDiscounts } = useQuery({
    queryKey: ['available-discounts', documentType, currentAmount],
    queryFn: () => getAvailableDiscounts(documentType, currentAmount)
  });

  const handleApplyDiscountCode = async () => {
    if (!discountCode) return;

    try {
      const validation = await validateDiscountCode(discountCode, documentType, documentId, currentAmount);
      
      if (!validation.valid) {
        toast.error(validation.message);
        return;
      }

      const result = await applyDiscount({
        discountCode,
        documentType,
        documentId
      });

      if (result.requiresApproval) {
        toast.info('Discount requires approval');
      } else {
        toast.success('Discount applied successfully');
        onDiscountApplied(result.appliedDiscounts[0]);
      }
    } catch (error) {
      toast.error('Failed to apply discount: ' + error.message);
    }
  };

  const calculateDiscountPreview = (discount: Discount) => {
    if (discount.valueType === 'PERCENTAGE') {
      return currentAmount * (discount.value / 100);
    }
    return Math.min(discount.value, currentAmount);
  };

  return (
    <Card sx={{ p: 3 }}>
      <Typography variant="h6" gutterBottom>
        Apply Discount
      </Typography>

      {/* Discount Code Input */}
      <Box sx={{ mb: 3 }}>
        <TextField
          fullWidth
          label="Discount Code"
          value={discountCode}
          onChange={(e) => setDiscountCode(e.target.value.toUpperCase())}
          placeholder="Enter discount code"
          InputProps={{
            endAdornment: (
              <Button
                variant="contained"
                size="small"
                onClick={handleApplyDiscountCode}
                disabled={!discountCode}
              >
                Apply
              </Button>
            )
          }}
        />
      </Box>

      {/* Available Discounts */}
      {availableDiscounts && availableDiscounts.length > 0 && (
        <Box>
          <Typography variant="subtitle1" gutterBottom>
            Available Discounts
          </Typography>
          
          {availableDiscounts.map(discount => (
            <Card key={discount.id} variant="outlined" sx={{ mb: 2, p: 2 }}>
              <Box sx={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center' }}>
                <Box>
                  <Typography variant="subtitle2">
                    {discount.name}
                  </Typography>
                  <Typography variant="body2" color="text.secondary">
                    {discount.description}
                  </Typography>
                  <Typography variant="caption" color="success.main">
                    Save ${calculateDiscountPreview(discount).toFixed(2)}
                  </Typography>
                </Box>
                <Button
                  variant="outlined"
                  size="small"
                  onClick={() => applySelectedDiscount(discount)}
                >
                  Apply
                </Button>
              </Box>
            </Card>
          ))}
        </Box>
      )}

      {/* Custom Discount (Admin only) */}
      <Box sx={{ mt: 3, pt: 3, borderTop: 1, borderColor: 'divider' }}>
        <Typography variant="subtitle1" gutterBottom>
          Custom Discount
        </Typography>
        
        <Grid container spacing={2} alignItems="center">
          <Grid item xs={4}>
            <FormControl fullWidth>
              <Select
                value={customDiscount.type}
                onChange={(e) => setCustomDiscount(prev => ({ ...prev, type: e.target.value }))}
              >
                <MenuItem value="PERCENTAGE">Percentage</MenuItem>
                <MenuItem value="FIXED_AMOUNT">Fixed Amount</MenuItem>
              </Select>
            </FormControl>
          </Grid>
          <Grid item xs={4}>
            <TextField
              fullWidth
              type="number"
              label={customDiscount.type === 'PERCENTAGE' ? 'Percent' : 'Amount'}
              value={customDiscount.value}
              onChange={(e) => setCustomDiscount(prev => ({ ...prev, value: parseFloat(e.target.value) }))}
              inputProps={{ 
                min: 0, 
                max: customDiscount.type === 'PERCENTAGE' ? 100 : currentAmount 
              }}
            />
          </Grid>
          <Grid item xs={4}>
            <Button
              variant="contained"
              fullWidth
              onClick={() => applyCustomDiscount()}
              disabled={customDiscount.value <= 0}
            >
              Apply Custom
            </Button>
          </Grid>
        </Grid>
      </Box>
    </Card>
  );
}
```

---

## 7. Implementation Plan

### Phase 1: Core Infrastructure (Week 1-2)
- [ ] Database schema migration for discount tables
- [ ] Basic CRUD operations for discount management
- [ ] Discount validation and eligibility checking
- [ ] Integration with existing totals calculation system

### Phase 2: Discount Application (Week 2-3)
- [ ] Discount code validation and application system
- [ ] Line-item and header-level discount application
- [ ] Approval workflow for high-value discounts
- [ ] Discount stacking rules and calculations

### Phase 3: User Interface (Week 3-4)
- [ ] Discount management dashboard for admins
- [ ] Discount selector component for quotes/invoices
- [ ] Discount code input and validation UI
- [ ] Approval interface for discount approvers

### Phase 4: Advanced Features (Week 4-5)
- [ ] Volume and tiered discount rules
- [ ] Client-specific discount management
- [ ] Promotional discount campaigns
- [ ] Discount usage reporting and analytics

### Phase 5: Integration & Testing (Week 5-6)
- [ ] Integration with PDF generation for discount display
- [ ] Email notifications for discount approvals
- [ ] Comprehensive testing of discount calculations
- [ ] Performance optimization for discount queries

---

## 8. Success Criteria

### Functional Requirements
- [ ] Create and manage various discount types (percentage, fixed, volume, promotional)
- [ ] Apply discounts at header and line-item level with proper calculations
- [ ] Validate discount codes with usage limits and expiration dates
- [ ] Require approval for discounts exceeding configured thresholds
- [ ] Generate comprehensive discount usage reports
- [ ] Integrate seamlessly with existing quote and invoice systems

### Technical Requirements
- [ ] Discount calculation accuracy of 99.99%
- [ ] Support for 1000+ concurrent discount validations
- [ ] Discount application processing < 500ms
- [ ] Comprehensive audit trail for all discount activities
- [ ] Flexible rule engine for complex discount conditions

### Business Requirements
- [ ] Reduce manual discount processing time by 80%
- [ ] Increase promotional campaign effectiveness
- [ ] Improve sales team productivity with quick discount application
- [ ] Ensure accurate financial reporting with proper discount tracking
- [ ] Support complex pricing strategies and client negotiations

---

**File Version:** 1.0 – Advanced Discount System Specification  
**Last Updated:** October 12, 2025