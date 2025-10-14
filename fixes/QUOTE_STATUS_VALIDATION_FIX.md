# Quote Status Validation Fix

## Issue
When attempting to mark a quote as "Sent" (or change to any status), the API was returning a validation error.

## Root Cause
Mismatch between frontend enum values and backend validation schema:

**Frontend (shared-types):**
```typescript
export enum QuoteStatus {
  DRAFT = 'DRAFT',
  SENT = 'SENT',
  ACCEPTED = 'ACCEPTED',
  REJECTED = 'REJECTED'  // ← Frontend uses REJECTED
}
```

**Backend (before fix):**
```typescript
const updateStatusSchema = z.object({
  status: z.enum(['DRAFT', 'SENT', 'ACCEPTED', 'DECLINED', 'EXPIRED']),  // ← Backend expected DECLINED
  notes: z.string().optional()
});
```

The frontend was sending `REJECTED` but the backend validation schema only accepted `DECLINED`.

## Fix Applied
Updated the backend validation schema in `apps/backend/src/routes/quotes.ts`:

```typescript
const updateStatusSchema = z.object({
  status: z.enum(['DRAFT', 'SENT', 'ACCEPTED', 'REJECTED']),  // ← Now matches frontend
  notes: z.string().optional()
});
```

Also removed `EXPIRED` status as it's not used in the current implementation.

## Testing
After fix:
1. Navigate to any quote detail page
2. Click "Status" button
3. Select "Mark as Sent"
4. Confirm dialog
5. ✅ Status should update successfully with success toast

## Files Modified
- `apps/backend/src/routes/quotes.ts` - Updated validation schema

## Related
- Part of Status Component Standardization work
- See: `docs/STATUS_COMPONENT_STANDARDIZATION.md`

## Deployment
- Backend restart required (completed)
- No database migration needed
- No frontend changes required
