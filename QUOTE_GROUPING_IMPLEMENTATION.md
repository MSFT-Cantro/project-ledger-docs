# Quote Line Item Grouping Implementation Guide

This document outlines all the changes made to implement line item grouping functionality for quotes. These same patterns can be applied to implement grouping for invoices.

## Overview

The grouping feature allows users to organize line items into logical groups (e.g., by room like "Kitchen", "Bathroom" or by service type like "Electrical Work", "Plumbing"). Items can be grouped or left ungrouped, and groups can be ordered and display subtotals.

## Database Schema Changes

### 1. Prisma Schema Updates (`apps/backend/prisma/schema.prisma`)

Added grouping fields to both QuoteItem and InvoiceItem models:

```prisma
model QuoteItem {
  id          Int     @id @default(autoincrement())
  quoteId     Int
  description String
  quantity    Float
  unitPrice   Float
  total       Float
  taxable     Boolean @default(true)
  sortOrder   Int     @default(0)
  
  // Grouping fields
  groupName   String? // Group name like "Kitchen", "Living Room", "Electrical Work"
  groupType   String? // Group type like "room" or "service"
  groupOrder  Int?    // Order of the group for sorting

  quote       Quote   @relation(fields: [quoteId], references: [id], onDelete: Cascade)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@index([quoteId])
  @@index([groupName])
  @@index([groupOrder])
}

model InvoiceItem {
  id          Int     @id @default(autoincrement())
  invoiceId   String
  description String
  quantity    Float
  unitPrice   Float
  total       Float
  taxable     Boolean @default(true)
  sortOrder   Int     @default(0)
  
  // Grouping fields (to be added)
  groupName   String?
  groupType   String?
  groupOrder  Int?

  invoice     Invoice @relation(fields: [invoiceId], references: [id], onDelete: Cascade)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@index([invoiceId])
  @@index([groupName])
  @@index([groupOrder])
}
```

### 2. Database Migration

After updating the schema, run:
```bash
npx prisma db push
```

### 3. Seed Data Updates (`apps/backend/prisma/seed.ts`)

Added default quote templates with flexible validation:

```typescript
// Create default quote template
await prisma.quoteTemplate.upsert({
  where: { type: 'default' },
  update: {},
  create: {
    type: 'default',
    name: 'Default Quote Template',
    description: 'Standard quote template for all types of quotes',
    fields: [
      {
        name: 'title',
        type: 'text',
        label: 'Quote Title',
        required: true
      },
      {
        name: 'items',
        type: 'text',
        label: 'Line Items',
        required: false
      },
      // ... other fields with flexible validation
    ]
  }
})
```

## Backend API Changes

### 1. Route Validation (`apps/backend/src/routes/quotes.ts`)

Updated validation schemas to accept grouping fields:

```typescript
const createQuoteSchema = z.object({
  // ... existing fields
  items: z.array(z.object({
    description: z.string(),
    quantity: z.number(),
    unitPrice: z.number(),
    taxable: z.boolean().optional().default(true),
    discount: z.any().optional(),
    notes: z.string().optional(),
    // Add grouping fields
    groupName: z.string().optional(),
    groupType: z.string().optional(),
    groupOrder: z.number().optional()
  })).optional().default([]),
  // ... rest of schema
});
```

### 2. Service Layer Updates (`apps/backend/src/services/quoteService.ts`)

Updated the createQuote method to handle grouping fields:

```typescript
// In createQuote method, the item insertion now includes:
await tx.$executeRaw`
  INSERT INTO "QuoteItem" (
    "quoteId", description, quantity, "unitPrice",
    total, taxable, "sortOrder", "groupName", "groupType", "groupOrder", "updatedAt"
  ) VALUES (
    ${createdQuote.id}, ${item.description}, ${item.quantity}, ${item.unitPrice},
    ${item.total}, ${item.taxable}, ${index}, ${item.groupName || null}, ${item.groupType || null}, ${item.groupOrder || null}, ${new Date()}
  )
`;
```

The `calculateQuoteTotals` method preserves grouping fields automatically through spread operator.

### 3. Template Service Updates (`apps/backend/src/services/quoteTemplateService.ts`)

Made validation more flexible to handle object/array data types:

```typescript
private validateTemplateData(templateType: string, data: Record<string, any>): ValidationResult {
  // ... existing validation logic
  
  // Made validation more permissive for complex data types
  if (field.type === 'text') {
    // Accept strings, objects, arrays, etc.
    if (value !== null && value !== undefined) {
      // Convert to string if needed
      validatedData[field.name] = typeof value === 'string' ? value : JSON.stringify(value);
    }
  }
  // ... rest of validation
}
```

### 4. PDF Service Updates (`apps/backend/src/services/pdfService.ts`)

Enhanced PDF generation to handle grouped items:

```typescript
// In generateQuotePDFFromData method:
if (hasGroups) {
  // Group items by groupName and groupType
  const groupedItems: Record<string, any[]> = {};
  const ungroupedItems: any[] = [];
  
  quote.items.forEach((item: any) => {
    if (item.groupName && item.groupName.trim()) {
      const groupKey = `${item.groupType || 'General'}: ${item.groupName}`;
      if (!groupedItems[groupKey]) {
        groupedItems[groupKey] = [];
      }
      groupedItems[groupKey].push(item);
    } else {
      ungroupedItems.push(item);
    }
  });
  
  // Sort groups by groupOrder
  const sortedGroups = Object.entries(groupedItems).sort(([, itemsA], [, itemsB]) => {
    const orderA = itemsA[0]?.groupOrder || 0;
    const orderB = itemsB[0]?.groupOrder || 0;
    return orderA - orderB;
  });
  
  // Render ungrouped items first (at the top)
  if (ungroupedItems.length > 0) {
    ungroupedItems.forEach((item: any) => {
      // Render item...
    });
    
    // Add separator line if there are groups
    if (sortedGroups.length > 0) {
      doc.moveTo(50, currentY).lineTo(550, currentY).stroke();
    }
  }
  
  // Render grouped items with headers and subtotals
  sortedGroups.forEach(([groupName, items]) => {
    // Group header
    doc.fontSize(14).font('Helvetica-Bold');
    doc.text(groupName, 50, currentY);
    
    // Group items
    items.forEach((item: any) => {
      // Render item...
    });
    
    // Group subtotal
    const groupSubtotal = items.reduce((sum, item) => sum + item.total, 0);
    doc.text(`${groupName} Subtotal: ${formatCurrency(groupSubtotal)}`, ...);
  });
}
```

## Frontend Changes

### 1. Type Definitions

#### Shared Types (`packages/shared-types/quotes/types.ts`)

```typescript
export interface QuoteItemBase {
  description: string;
  quantity: number;
  unitPrice: number;
  taxable?: boolean;
  discount?: {
    type: 'percentage' | 'fixed';
    value: number;
  };
  notes?: string;
  // Grouping fields
  groupName?: string;  // Group name like "Kitchen", "Bathroom", "Electrical Work"
  groupType?: string;  // Group type like "room" or "service"
  groupOrder?: number; // Order of the group for sorting
}

export interface QuoteItem extends Required<Omit<QuoteItemBase, 'discount' | 'notes' | 'groupName' | 'groupType' | 'groupOrder'>> {
  id: string;
  quoteId: string;
  total: number;
  // Optional grouping fields
  groupName?: string;
  groupType?: string;
  groupOrder?: number;
}
```

#### Frontend Types (`apps/frontend/src/types/quotes/types.ts`)

Same structure as shared types to maintain consistency.

### 2. Form Components (`apps/frontend/src/components/quotes/QuoteForm.tsx`)

#### Grouping UI Implementation

```tsx
// Group items by groupName for accordion display
const groupedItems = useMemo(() => {
  const groups: Record<string, any[]> = {};
  const ungrouped: any[] = [];
  
  fields.forEach((field, index) => {
    const item = { ...field, index };
    if (item.groupName && item.groupName.trim()) {
      const groupKey = `${item.groupType || 'general'}-${item.groupName}`;
      if (!groups[groupKey]) {
        groups[groupKey] = [];
      }
      groups[groupKey].push(item);
    } else {
      ungrouped.push(item);
    }
  });
  
  return { groups, ungrouped };
}, [fields]);

// Get available group names for autocomplete
const availableGroups = useMemo(() => {
  const groups = new Set<string>();
  fields.forEach(item => {
    if (item.groupName && item.groupName.trim()) {
      groups.add(item.groupName);
    }
  });
  return Array.from(groups);
}, [fields]);
```

#### Accordion-Based Group Display

```tsx
{/* Render grouped items */}
{Object.entries(groupedItems.groups).map(([groupKey, items]) => {
  const firstItem = items[0];
  return (
    <Accordion key={groupKey} defaultExpanded>
      <AccordionSummary expandIcon={<ExpandMore />}>
        <Typography variant="h6">
          {firstItem.groupType ? `${firstItem.groupType}: ` : ''}{firstItem.groupName}
        </Typography>
        <Chip 
          label={`${items.length} item${items.length !== 1 ? 's' : ''}`}
          size="small"
          sx={{ ml: 2 }}
        />
      </AccordionSummary>
      <AccordionDetails>
        {items.map((item, itemIndex) => (
          <ItemFormSection key={item.index} item={item} />
        ))}
      </AccordionDetails>
    </Accordion>
  );
})}
```

#### Fixed Autocomplete Behavior

```tsx
<Controller
  name={`items.${item.index}.groupName`}
  control={control}
  render={({ field }) => (
    <Autocomplete
      freeSolo
      options={availableGroups}
      value={field.value || ''}
      onChange={(_, value) => {
        // Only update when user selects from dropdown or confirms input
        field.onChange(value || '');
      }}
      onBlur={() => {
        field.onBlur();
      }}
      renderInput={(params) => (
        <TextField
          {...params}
          label="Group Name"
          placeholder="e.g., Kitchen, Living Room, Electrical Work"
          helperText="Optional: Group this item with others"
          onBlur={(e) => {
            // Update the field value when user tabs out or clicks away
            field.onChange(e.target.value);
            field.onBlur();
          }}
          onKeyDown={(e) => {
            // Update the field value when user presses Enter
            if (e.key === 'Enter') {
              field.onChange((e.target as HTMLInputElement).value);
            }
          }}
        />
      )}
    />
  )}
/>
```

### 3. Page Components

#### Quote Creation (`apps/frontend/src/pages/QuoteCreatePage.tsx`)

Updated lineItems mapping to include grouping fields:

```typescript
lineItems: data.items.map((item, index) => ({
  id: `item-${index + 1}`,
  description: item.description,
  quantity: item.quantity,
  unitPrice: item.unitPrice,
  taxable: item.taxable ?? true,
  discount: item.discount,
  notes: item.notes,
  // Include grouping fields
  groupName: item.groupName,
  groupType: item.groupType,
  groupOrder: item.groupOrder
}))
```

#### Quote Editing (`apps/frontend/src/pages/QuoteEditPage.tsx`)

Updated data transformation to preserve grouping fields:

```typescript
items: quote.items.map(item => ({
  description: item.description,
  quantity: item.quantity,
  unitPrice: item.unitPrice,
  taxable: item.taxable,
  // Preserve grouping fields with type casting
  groupName: (item as any).groupName || '',
  groupType: (item as any).groupType || '',
  groupOrder: (item as any).groupOrder || 0
}))
```

#### Quote Detail Page (`apps/frontend/src/pages/QuoteDetailPage.tsx`)

Enhanced to display grouped items and improved status management:

```tsx
// Manual status management with dropdown
<FormControl size="small" sx={{ minWidth: 120 }}>
  <InputLabel>Status</InputLabel>
  <Select
    value={quote.status}
    label="Status"
    onChange={(e) => handleStatusChange(e.target.value as QuoteStatus)}
    disabled={isUpdatingStatus || statusUpdateMutation.isPending}
  >
    <MenuItem value={QuoteStatus.DRAFT}>Draft</MenuItem>
    <MenuItem value={QuoteStatus.SENT}>Sent</MenuItem>
    <MenuItem value={QuoteStatus.ACCEPTED}>Accepted</MenuItem>
    <MenuItem value={QuoteStatus.REJECTED}>Rejected</MenuItem>
  </Select>
</FormControl>

// Status update mutation
const statusUpdateMutation = useMutation({
  mutationFn: ({ status }: { status: QuoteStatus }) => quotesApi.updateStatus(id!, status),
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['quote', id] });
    queryClient.invalidateQueries({ queryKey: ['quotes'] });
    toast.success('Quote status updated successfully');
  }
});
```

## Key Implementation Patterns for Invoices

### 1. Database Schema Pattern
- Add `groupName`, `groupType`, `groupOrder` fields to InvoiceItem model
- Add indexes for efficient querying
- Use same nullable approach for backward compatibility

### 2. API Validation Pattern
- Update validation schemas to accept grouping fields
- Use `z.string().optional()` for nullable string fields
- Use `z.number().optional()` for nullable number fields

### 3. Service Layer Pattern
- Preserve grouping fields in data transformation methods
- Include grouping fields in database insert/update operations
- Use spread operator to maintain all fields during calculations

### 4. PDF Generation Pattern
- Group items by `groupName` and `groupType`
- Sort groups by `groupOrder`
- Render ungrouped items first
- Add group headers and subtotals
- Include visual separators between sections

### 5. Frontend Form Pattern
- Use accordion-based UI for grouped item display
- Implement autocomplete with proper event handling
- Group items using `useMemo` for performance
- Provide visual feedback with chips showing item counts

### 6. Data Flow Pattern
- Preserve grouping fields throughout the entire data pipeline
- Use type casting where necessary for TypeScript compatibility
- Ensure API request/response includes all grouping fields
- Maintain backward compatibility with existing data

## Testing Checklist

When implementing for invoices:

### Backend Testing
- [ ] Create invoice with grouped items
- [ ] Update invoice with mixed grouped/ungrouped items
- [ ] Verify database stores grouping fields correctly
- [ ] Test PDF generation with various grouping scenarios
- [ ] Validate API endpoints accept grouping data

### Frontend Testing
- [ ] Create invoice with grouped items in form
- [ ] Edit existing invoice and verify grouping data loads
- [ ] Test autocomplete behavior (no premature submissions)
- [ ] Verify grouped items display correctly in details view
- [ ] Test status management dropdown functionality

### Integration Testing
- [ ] End-to-end quote creation to PDF generation
- [ ] Data persistence across create/edit/view cycle
- [ ] Group ordering and sorting functionality
- [ ] Template validation with complex data types

## Migration Strategy

1. **Database Migration**: Update schema and run migration
2. **Backend Implementation**: Update services and routes
3. **Frontend Implementation**: Update components and pages
4. **Testing**: Comprehensive testing of all scenarios
5. **Documentation**: Update API documentation and user guides

## Notes

- All grouping fields are optional to maintain backward compatibility
- Group ordering is handled at the UI level and in PDF generation
- The same patterns work for both quotes and invoices
- Type casting may be needed during transition period
- Consider adding group validation rules if needed (e.g., max group name length)
