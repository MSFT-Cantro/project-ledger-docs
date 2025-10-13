# Change Orders System Specification

**Date:** October 12, 2025  
**Version:** 1.0  
**Status:** Not Implemented  

---

## 1. Overview

Change Orders enable clients and contractors to modify accepted quotes/contracts with scope or price changes while maintaining full audit trails and legal compliance. This system handles both additive and subtractive changes with automatic invoice/credit note generation.

---

## 2. Business Requirements

### 2.1 Core Functionality
- **Scope Modifications**: Add/remove line items from accepted quotes
- **Price Adjustments**: Modify quantities, unit prices, or add fees/discounts  
- **Approval Workflow**: Client acceptance required before execution
- **Financial Impact**: Automatic calculation of net change (positive/negative)
- **Document Generation**: Professional PDF change orders with approval signatures
- **Audit Trail**: Complete history of all modifications and approvals

### 2.2 Business Rules
- Change Orders can only be created for quotes with status `ACCEPTED`
- Each Change Order must reference the original quote/contract
- Positive deltas generate supplemental invoices
- Negative deltas generate credit notes
- Change Orders require client approval before becoming effective
- Original quote remains immutable; changes tracked separately

---

## 3. Data Model

### 3.1 Database Schema (Prisma)

```prisma
model ChangeOrder {
  id                  Int                    @id @default(autoincrement())
  number              String                 @unique // CO-YYYY-XXX format
  quoteId             Int                    // Reference to original quote
  version             Int                    @default(1)
  status              ChangeOrderStatus
  changeType          ChangeOrderType
  
  // Financial Impact
  originalTotal       Float                  // Original quote total
  newTotal           Float                  // New total after changes
  deltaAmount        Float                  // Net change (can be negative)
  
  // Content
  title              String?
  description        String?
  reason             String?                // Reason for change
  
  // Approval
  clientApprovedAt   DateTime?
  clientApprovedBy   String?                // Client user who approved
  contractorApprovedAt DateTime?
  contractorApprovedBy Int?                 // Internal user ID
  
  // Metadata  
  validUntil         DateTime               // Expiry date for approval
  createdAt          DateTime               @default(now())
  updatedAt          DateTime               @updatedAt
  createdById        Int
  
  // Relations
  quote              Quote                  @relation(fields: [quoteId], references: [id])
  createdBy          User                   @relation(fields: [createdById], references: [id])
  items              ChangeOrderItem[]
  history            ChangeOrderHistory[]
  invoices           Invoice[]              // Generated invoices/credits
  
  @@index([quoteId])
  @@index([status])
  @@index([createdById])
}

model ChangeOrderItem {
  id              Int           @id @default(autoincrement())
  changeOrderId   Int
  changeType      ItemChangeType // ADD, REMOVE, MODIFY
  
  // Original item reference (for MODIFY/REMOVE)
  originalQuoteItemId Int?
  
  // New/Modified item data
  description     String
  quantity        Float
  unitPrice       Float
  total           Float
  
  // Change tracking
  quantityChange  Float?        // For MODIFY operations
  priceChange     Float?        // For MODIFY operations
  
  sortOrder       Int
  createdAt       DateTime      @default(now())
  
  changeOrder     ChangeOrder   @relation(fields: [changeOrderId], references: [id], onDelete: Cascade)
  originalItem    QuoteItem?    @relation(fields: [originalQuoteItemId], references: [id])
  
  @@index([changeOrderId])
}

model ChangeOrderHistory {
  id              Int           @id @default(autoincrement())
  changeOrderId   Int
  status          String
  changes         Json          // Detailed change log
  notes           String?
  userId          Int
  createdAt       DateTime      @default(now())
  
  changeOrder     ChangeOrder   @relation(fields: [changeOrderId], references: [id], onDelete: Cascade)
  user            User          @relation(fields: [userId], references: [id])
  
  @@index([changeOrderId])
}

enum ChangeOrderStatus {
  DRAFT
  SENT
  UNDER_REVIEW
  APPROVED
  DECLINED
  EXPIRED
  EXECUTED
}

enum ChangeOrderType {
  ADDITIVE      // Increases total cost
  DEDUCTIVE     // Decreases total cost  
  NEUTRAL       // No net cost change
}

enum ItemChangeType {
  ADD           // New line item
  REMOVE        // Remove existing item
  MODIFY        // Change quantity/price of existing item
}
```

### 3.2 TypeScript Interfaces

```typescript
export interface ChangeOrder {
  id: string;
  number: string;
  quoteId: string;
  version: number;
  status: ChangeOrderStatus;
  changeType: ChangeOrderType;
  
  // Financial
  originalTotal: number;
  newTotal: number;
  deltaAmount: number;
  
  // Content
  title?: string;
  description?: string;  
  reason?: string;
  
  // Approval
  clientApprovedAt?: string;
  clientApprovedBy?: string;
  contractorApprovedAt?: string;
  contractorApprovedBy?: string;
  
  // Metadata
  validUntil: string;
  createdAt: string;
  updatedAt: string;
  createdById: string;
  
  // Relations
  quote: Quote;
  items: ChangeOrderItem[];
  history: ChangeOrderHistory[];
}

export interface ChangeOrderItem {
  id: string;
  changeType: ItemChangeType;
  originalQuoteItemId?: string;
  
  description: string;
  quantity: number;
  unitPrice: number;
  total: number;
  
  quantityChange?: number;
  priceChange?: number;
  
  sortOrder: number;
}

export interface CreateChangeOrderInput {
  quoteId: string;
  title?: string;
  description?: string;
  reason?: string;
  validUntil?: string;
  items: CreateChangeOrderItemInput[];
}

export interface CreateChangeOrderItemInput {
  changeType: ItemChangeType;
  originalQuoteItemId?: string;
  description: string;
  quantity: number;
  unitPrice: number;
}
```

---

## 4. API Endpoints

### 4.1 Core CRUD Operations

```typescript
// Create change order
POST /api/change-orders
Body: CreateChangeOrderInput
Response: ChangeOrder

// Get change order
GET /api/change-orders/:id
Response: ChangeOrder

// Update change order (draft only)
PATCH /api/change-orders/:id
Body: Partial<CreateChangeOrderInput>
Response: ChangeOrder

// Delete change order (draft only)
DELETE /api/change-orders/:id
Response: { success: boolean }

// List change orders
GET /api/change-orders
Query: quoteId?, status?, clientId?
Response: ChangeOrder[]
```

### 4.2 Workflow Operations

```typescript
// Send to client for approval
POST /api/change-orders/:id/send
Body: { email?: string, message?: string }
Response: ChangeOrder

// Client approval (public endpoint with token)
POST /api/change-orders/:id/approve
Body: { token: string, approvedBy: string }
Response: ChangeOrder

// Client decline
POST /api/change-orders/:id/decline  
Body: { token: string, reason?: string }
Response: ChangeOrder

// Execute approved change order
POST /api/change-orders/:id/execute
Response: { 
  changeOrder: ChangeOrder,
  generatedInvoice?: Invoice,
  generatedCreditNote?: Invoice
}
```

### 4.3 Document Generation

```typescript
// Generate PDF
GET /api/change-orders/:id/pdf
Response: Binary PDF

// Get approval link (for client)
GET /api/change-orders/:id/approval-link
Response: { approvalUrl: string, expiresAt: string }
```

---

## 5. Business Logic

### 5.1 Change Order Creation

```typescript
async function createChangeOrder(input: CreateChangeOrderInput): Promise<ChangeOrder> {
  // 1. Validate quote exists and is ACCEPTED
  const quote = await getQuote(input.quoteId);
  if (quote.status !== 'ACCEPTED') {
    throw new Error('Can only create change orders for accepted quotes');
  }
  
  // 2. Calculate financial impact
  const originalTotal = quote.total;
  const { newTotal, deltaAmount, changeType } = calculateChangeImpact(quote, input.items);
  
  // 3. Generate change order number
  const number = await generateChangeOrderNumber();
  
  // 4. Create change order
  const changeOrder = await prisma.changeOrder.create({
    data: {
      number,
      quoteId: input.quoteId,
      status: 'DRAFT',
      changeType,
      originalTotal,
      newTotal,
      deltaAmount,
      title: input.title,
      description: input.description,
      reason: input.reason,
      validUntil: input.validUntil || addDays(new Date(), 30),
      createdById: currentUser.id,
      items: {
        create: input.items.map((item, index) => ({
          ...item,
          total: item.quantity * item.unitPrice,
          sortOrder: index
        }))
      }
    },
    include: { items: true, quote: true }
  });
  
  return changeOrder;
}
```

### 5.2 Financial Impact Calculation

```typescript
function calculateChangeImpact(quote: Quote, items: ChangeOrderItemInput[]) {
  let totalChange = 0;
  
  for (const item of items) {
    switch (item.changeType) {
      case 'ADD':
        totalChange += item.quantity * item.unitPrice;
        break;
        
      case 'REMOVE':
        const originalItem = quote.items.find(qi => qi.id === item.originalQuoteItemId);
        if (originalItem) {
          totalChange -= originalItem.total;
        }
        break;
        
      case 'MODIFY':
        const modifyItem = quote.items.find(qi => qi.id === item.originalQuoteItemId);
        if (modifyItem) {
          const newItemTotal = item.quantity * item.unitPrice;
          totalChange += (newItemTotal - modifyItem.total);
        }
        break;
    }
  }
  
  const newTotal = quote.total + totalChange;
  
  const changeType: ChangeOrderType = 
    totalChange > 0 ? 'ADDITIVE' :
    totalChange < 0 ? 'DEDUCTIVE' : 'NEUTRAL';
    
  return {
    newTotal,
    deltaAmount: totalChange,
    changeType
  };
}
```

### 5.3 Change Order Execution

```typescript
async function executeChangeOrder(changeOrderId: string) {
  const changeOrder = await getChangeOrder(changeOrderId);
  
  if (changeOrder.status !== 'APPROVED') {
    throw new Error('Change order must be approved before execution');
  }
  
  const results: any = { changeOrder };
  
  // Generate financial documents based on delta
  if (changeOrder.deltaAmount > 0) {
    // Positive change: create supplemental invoice
    results.generatedInvoice = await createSupplementalInvoice(changeOrder);
  } else if (changeOrder.deltaAmount < 0) {
    // Negative change: create credit note
    results.generatedCreditNote = await createCreditNote(changeOrder);
  }
  
  // Update change order status
  await updateChangeOrderStatus(changeOrderId, 'EXECUTED');
  
  return results;
}
```

---

## 6. User Interface

### 6.1 Change Order Creation Form

```typescript
interface ChangeOrderFormProps {
  quoteId: string;
  onSubmit: (data: CreateChangeOrderInput) => void;
}

export function ChangeOrderForm({ quoteId, onSubmit }: ChangeOrderFormProps) {
  const [items, setItems] = useState<ChangeOrderItemInput[]>([]);
  const [originalQuote, setOriginalQuote] = useState<Quote>();
  
  // Component for adding/modifying items
  const renderItemEditor = (item: ChangeOrderItemInput, index: number) => (
    <Box key={index} sx={{ border: 1, borderColor: 'divider', p: 2, mb: 2 }}>
      <FormControl fullWidth sx={{ mb: 2 }}>
        <InputLabel>Change Type</InputLabel>
        <Select
          value={item.changeType}
          onChange={(e) => updateItem(index, 'changeType', e.target.value)}
        >
          <MenuItem value="ADD">Add New Item</MenuItem>
          <MenuItem value="MODIFY">Modify Existing Item</MenuItem>
          <MenuItem value="REMOVE">Remove Item</MenuItem>
        </Select>
      </FormControl>
      
      {item.changeType !== 'ADD' && (
        <FormControl fullWidth sx={{ mb: 2 }}>
          <InputLabel>Original Item</InputLabel>
          <Select
            value={item.originalQuoteItemId || ''}
            onChange={(e) => updateItem(index, 'originalQuoteItemId', e.target.value)}
          >
            {originalQuote?.items.map(qi => (
              <MenuItem key={qi.id} value={qi.id}>
                {qi.description} - ${qi.total}
              </MenuItem>
            ))}
          </Select>
        </FormControl>
      )}
      
      <TextField
        fullWidth
        label="Description"
        value={item.description}
        onChange={(e) => updateItem(index, 'description', e.target.value)}
        sx={{ mb: 2 }}
      />
      
      <Box sx={{ display: 'flex', gap: 2 }}>
        <TextField
          label="Quantity"
          type="number"
          value={item.quantity}
          onChange={(e) => updateItem(index, 'quantity', parseFloat(e.target.value))}
        />
        <TextField
          label="Unit Price"
          type="number"
          value={item.unitPrice}
          onChange={(e) => updateItem(index, 'unitPrice', parseFloat(e.target.value))}
        />
        <TextField
          label="Total"
          value={`$${(item.quantity * item.unitPrice).toFixed(2)}`}
          disabled
        />
      </Box>
    </Box>
  );
  
  return (
    <Paper sx={{ p: 3 }}>
      <Typography variant="h5" gutterBottom>
        Create Change Order
      </Typography>
      
      {/* Change order details form */}
      <Box sx={{ mb: 4 }}>
        <TextField
          fullWidth
          label="Title"
          sx={{ mb: 2 }}
        />
        <TextField
          fullWidth
          multiline
          rows={3}
          label="Description"
          sx={{ mb: 2 }}
        />
        <TextField
          fullWidth
          label="Reason for Change"
          sx={{ mb: 2 }}
        />
      </Box>
      
      {/* Items editor */}
      <Typography variant="h6" gutterBottom>
        Change Items
      </Typography>
      
      {items.map((item, index) => renderItemEditor(item, index))}
      
      <Button 
        startIcon={<Add />}
        onClick={() => setItems([...items, { 
          changeType: 'ADD', 
          description: '', 
          quantity: 1, 
          unitPrice: 0 
        }])}
        sx={{ mb: 3 }}
      >
        Add Change Item
      </Button>
      
      {/* Financial impact summary */}
      <FinancialImpactSummary 
        originalTotal={originalQuote?.total || 0}
        items={items}
      />
      
      <Box sx={{ display: 'flex', gap: 2, justifyContent: 'flex-end' }}>
        <Button variant="outlined">
          Save Draft
        </Button>
        <Button variant="contained" onClick={() => onSubmit({
          quoteId,
          items,
          // ... other form data
        })}>
          Create Change Order
        </Button>
      </Box>
    </Paper>
  );
}
```

### 6.2 Client Approval Interface

```typescript
export function ChangeOrderApproval({ changeOrder, token }: ChangeOrderApprovalProps) {
  const [approved, setApproved] = useState<boolean | null>(null);
  const [reason, setReason] = useState('');
  
  const handleApproval = async (approve: boolean) => {
    try {
      if (approve) {
        await approveChangeOrder(changeOrder.id, { 
          token, 
          approvedBy: 'Client Name' 
        });
        setApproved(true);
      } else {
        await declineChangeOrder(changeOrder.id, { 
          token, 
          reason 
        });
        setApproved(false);
      }
    } catch (error) {
      console.error('Approval failed:', error);
    }
  };
  
  if (approved !== null) {
    return (
      <Alert severity={approved ? "success" : "info"}>
        Change order has been {approved ? 'approved' : 'declined'}.
      </Alert>
    );
  }
  
  return (
    <Card sx={{ maxWidth: 800, mx: 'auto', mt: 4 }}>
      <CardContent>
        <Typography variant="h4" gutterBottom>
          Change Order Approval Required
        </Typography>
        
        <Typography variant="h6" gutterBottom>
          {changeOrder.title}
        </Typography>
        
        <Typography variant="body1" paragraph>
          {changeOrder.description}
        </Typography>
        
        {/* Change items display */}
        <TableContainer component={Paper} sx={{ mb: 3 }}>
          <Table>
            <TableHead>
              <TableRow>
                <TableCell>Change Type</TableCell>
                <TableCell>Description</TableCell>
                <TableCell align="right">Quantity</TableCell>
                <TableCell align="right">Unit Price</TableCell>
                <TableCell align="right">Total</TableCell>
              </TableRow>
            </TableHead>
            <TableBody>
              {changeOrder.items.map((item) => (
                <TableRow key={item.id}>
                  <TableCell>
                    <Chip 
                      label={item.changeType}
                      color={
                        item.changeType === 'ADD' ? 'success' :
                        item.changeType === 'REMOVE' ? 'error' : 'warning'
                      }
                      size="small"
                    />
                  </TableCell>
                  <TableCell>{item.description}</TableCell>
                  <TableCell align="right">{item.quantity}</TableCell>
                  <TableCell align="right">${item.unitPrice.toFixed(2)}</TableCell>
                  <TableCell align="right">${item.total.toFixed(2)}</TableCell>
                </TableRow>
              ))}
            </TableBody>
          </Table>
        </TableContainer>
        
        {/* Financial impact */}
        <Box sx={{ bgcolor: 'grey.50', p: 2, borderRadius: 1, mb: 3 }}>
          <Typography variant="h6" gutterBottom>
            Financial Impact
          </Typography>
          <Box sx={{ display: 'flex', justifyContent: 'space-between', mb: 1 }}>
            <Typography>Original Quote Total:</Typography>
            <Typography>${changeOrder.originalTotal.toFixed(2)}</Typography>
          </Box>
          <Box sx={{ display: 'flex', justifyContent: 'space-between', mb: 1 }}>
            <Typography>Change Amount:</Typography>
            <Typography color={changeOrder.deltaAmount >= 0 ? 'success.main' : 'error.main'}>
              {changeOrder.deltaAmount >= 0 ? '+' : ''}${changeOrder.deltaAmount.toFixed(2)}
            </Typography>
          </Box>
          <Box sx={{ display: 'flex', justifyContent: 'space-between', fontWeight: 'bold' }}>
            <Typography variant="h6">New Total:</Typography>
            <Typography variant="h6">${changeOrder.newTotal.toFixed(2)}</Typography>
          </Box>
        </Box>
        
        {/* Decline reason */}
        {approved === false && (
          <TextField
            fullWidth
            multiline
            rows={3}
            label="Reason for declining (optional)"
            value={reason}
            onChange={(e) => setReason(e.target.value)}
            sx={{ mb: 3 }}
          />
        )}
        
        {/* Action buttons */}
        <Box sx={{ display: 'flex', gap: 2, justifyContent: 'center' }}>
          <Button
            variant="outlined"
            color="error"
            size="large"
            onClick={() => handleApproval(false)}
          >
            Decline Change Order
          </Button>
          <Button
            variant="contained"
            color="success"
            size="large"
            onClick={() => handleApproval(true)}
          >
            Approve Change Order
          </Button>
        </Box>
      </CardContent>
    </Card>
  );
}
```

---

## 7. Implementation Plan

### Phase 1: Core Infrastructure (Week 1-2)
- [ ] Database schema migration for ChangeOrder tables
- [ ] Basic TypeScript interfaces and types
- [ ] Core API endpoints (CRUD operations)
- [ ] Change order number generation
- [ ] Basic validation and error handling

### Phase 2: Business Logic (Week 2-3)
- [ ] Financial impact calculation engine
- [ ] Change order creation workflow
- [ ] Status management and transitions
- [ ] Integration with existing Quote system
- [ ] Audit trail and history tracking

### Phase 3: User Interface (Week 3-4)
- [ ] Change order creation form
- [ ] Change order list and detail views
- [ ] Item editor with add/modify/remove options
- [ ] Financial impact visualization
- [ ] Status indicators and workflow UI

### Phase 4: Approval Workflow (Week 4-5)
- [ ] Client approval interface (public pages)
- [ ] Email notification system
- [ ] Approval token generation and validation
- [ ] PDF generation for change orders
- [ ] Document signing/approval tracking

### Phase 5: Financial Integration (Week 5-6)
- [ ] Supplemental invoice generation
- [ ] Credit note creation
- [ ] Integration with existing Invoice system
- [ ] Payment processing for change orders
- [ ] Accounting journal entries

### Phase 6: Testing & Polish (Week 6)
- [ ] Unit tests for business logic
- [ ] Integration tests for API endpoints
- [ ] E2E tests for approval workflow
- [ ] Performance optimization
- [ ] Documentation and training materials

---

## 8. Success Criteria

### Functional Requirements
- [ ] Create change orders from accepted quotes
- [ ] Calculate financial impact accurately  
- [ ] Client approval workflow functions end-to-end
- [ ] Generate appropriate invoices/credit notes
- [ ] Maintain complete audit trail
- [ ] Professional PDF generation

### Technical Requirements
- [ ] API response times < 500ms
- [ ] 99.9% uptime for approval workflows
- [ ] Secure token-based client approvals
- [ ] Database referential integrity maintained
- [ ] Error handling and validation comprehensive

### Business Requirements
- [ ] Reduces change order processing time by 75%
- [ ] Eliminates manual calculation errors
- [ ] Provides legal audit trail for compliance
- [ ] Integrates seamlessly with existing workflows
- [ ] Supports complex multi-item changes

---

**File Version:** 1.0 – Change Orders System Specification  
**Last Updated:** October 12, 2025