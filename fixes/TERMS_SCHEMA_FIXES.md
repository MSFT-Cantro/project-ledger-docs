# Terms & Conditions Schema Fixes

## Issues Identified

### 1. ❌ Schema Mismatch: `title` vs `name`
**Problem:** Frontend code referenced `termsTemplate.title` but database schema uses `termsTemplate.name`

**Affected Files:**
- ✅ `apps/frontend/src/api/terms.ts` - Type definition
- ✅ `apps/frontend/src/components/TermsManagement/TermsList.tsx` - Display
- ✅ `apps/frontend/src/components/TermsManagement/TermsSelector.tsx` - Dropdown & Preview
- ✅ `apps/frontend/src/components/TermsManagement/TermsEditor.tsx` - Edit modal
- ✅ `apps/backend/src/services/pdfService.ts` - Quote PDF rendering (line 378)
- ✅ `apps/backend/src/services/pdfService.ts` - Invoice PDF rendering (line 648)

**Fix Applied:** Changed all references from `title` to `name` to match Prisma schema

---

### 2. ❌ Missing `accountId` Field
**Problem:** Frontend type definition included `accountId` but schema uses `createdById`

**Fix Applied:** Updated `TermsTemplate` interface in `terms.ts` to use `createdById` and `updatedById`

---

### 3. ❌ Type Mismatches in Template Interface
**Problem:** Several fields had incorrect types compared to Prisma schema

**Changes:**
- `version`: Changed from `number` to `string` (schema uses String)
- `expirationDate`: Renamed to `expiryDate` (schema uses expiryDate)
- `tags`: Removed (schema doesn't have this field)
- Added `description`, `jurisdictions`, `governingLaw`, `isDefault`, `requiresAcceptance` (all exist in schema)

---

### 4. ❌ "All available terms added" Message - No Visibility
**Problem:** When all terms templates are added, UI shows message but doesn't display which ones are attached

**Current Behavior:**
```
"All available terms have been added to this document."
```
But user can't see what terms are attached!

**Root Cause:** The `TermsList` component is rendered, but it's not immediately visible or clear that terms exist below the selector.

**Fix Applied:** 
- Removed the confusing "tags" chip section from TermsList (schema doesn't have tags)
- Added description display in TermsList for better context
- The terms list already shows all attached terms with position, title, content preview

**User Experience Now:**
1. If no terms added → Shows "No terms added yet"
2. If some terms added → Shows dropdown with remaining templates + list of attached terms
3. If all terms added → Shows "All available terms added" message + full list of attached terms below

---

### 5. ❌ Terms Not Appearing in PDFs
**Problem:** Even when terms were added to documents, they didn't appear in generated PDFs

**Root Causes:**
1. PDF rendering code used `termsTemplate.title` instead of `termsTemplate.name` ✅ FIXED
2. Backend rebuilt to apply the changes ✅ DONE

**Testing Required:**
1. Add a term to a quote
2. Generate PDF
3. Verify "Terms and Conditions" section appears with the term

---

## Files Modified

### Frontend Files (6 files)
```
apps/frontend/src/
├── api/terms.ts (Type definitions fixed)
├── components/TermsManagement/
│   ├── TermsList.tsx (name fix, removed tags, added description)
│   ├── TermsSelector.tsx (name fix, description display)
│   └── TermsEditor.tsx (name fix)
```

### Backend Files (1 file)
```
apps/backend/src/
└── services/pdfService.ts (Quote & Invoice PDF: title → name)
```

---

## Verification Checklist

### ✅ Schema Alignment
- [x] All TypeScript interfaces match Prisma schema
- [x] No references to non-existent fields (title, tags, accountId)
- [x] Correct field types (version string, expiryDate)

### ⏳ Frontend Display (Needs Testing)
- [ ] Templates dropdown shows template names correctly
- [ ] Terms list displays term names (not "undefined")
- [ ] Edit modal shows correct term name in header
- [ ] Category chips display properly
- [ ] Description shows when available

### ⏳ PDF Generation (Needs Testing)
- [ ] Quote PDF includes "Terms and Conditions" section
- [ ] Invoice PDF includes terms
- [ ] Change Order PDF includes terms
- [ ] Terms show correct title/name
- [ ] Custom content is used when present
- [ ] Position numbers display correctly (1, 2, 3...)

---

## Next Steps

### Step 1: Rebuild Frontend (Required)
```bash
docker compose restart frontend
# OR
docker compose exec frontend npm run build
```

### Step 2: Test Terms Display
1. Navigate to `/quotes/1/edit` (or any existing quote)
2. Verify templates dropdown shows:
   - ✓ Payment Terms
   - ✓ Warranty Terms
   - ✓ Cancellation Policy
   - ✓ Liability Limitations
   - ✓ Change Order Process
   - ✓ Intellectual Property Rights
   - ✓ Confidentiality Agreement
3. Select "Payment Terms" and click "Add"
4. Verify term appears in list below with:
   - Position badge (1)
   - Name: "Payment Terms"
   - Category chip
   - Content preview
5. Click edit icon, verify modal shows "Payment Terms" in header

### Step 3: Test PDF Generation
1. Save the quote (with term attached)
2. Navigate to quote detail view
3. Click "Download PDF"
4. Open PDF and scroll to bottom
5. Verify section titled "Terms and Conditions"
6. Verify "1. Payment Terms" appears with full content

### Step 4: Test Other Document Types
Repeat Steps 2-3 for:
- Invoice edit page → Invoice PDF
- Change Order edit page → Change Order PDF

---

## Known Issues Remaining

### Issue: Terms on Create Pages
**Problem:** Cannot add terms when creating a new quote/invoice/change order

**Why:** TermsManagement component requires `documentId`, which doesn't exist until document is created

**Solution Options:**
1. **Option A (Recommended):** Keep current behavior - terms can only be added after creation
   - Simple, no database changes
   - User creates document, then edits to add terms
   
2. **Option B:** Add terms field to create forms
   - Allow selecting templates during creation
   - Store as temporary state
   - Attach terms in backend after document creation
   - More complex, requires form changes

**Recommendation:** Stick with Option A. Adding terms during creation is a nice-to-have, not critical. Current workflow is acceptable:
1. Create quote/invoice/change order
2. Edit to add terms
3. Generate PDF

**Future Enhancement:** If needed, can implement Option B in Phase 4 or later.

---

## Database Status

### Terms Templates (7 in database)
```sql
SELECT id, name, category, isActive FROM TermsTemplate;
```
```
id | name                        | category              | isActive
---+----------------------------+-----------------------+---------
1  | Payment Terms               | PAYMENT_TERMS         | true
2  | Intellectual Property Rights| INTELLECTUAL_PROPERTY | true
3  | Warranty Terms              | SERVICE_TERMS         | true
4  | Cancellation Policy         | CANCELLATION          | true
5  | Liability Limitations       | LEGAL_TERMS           | true
6  | Change Order Process        | CUSTOM                | true
7  | Confidentiality Agreement   | PRIVACY_DATA          | true
```

### Document Terms (None yet)
```sql
SELECT * FROM QuoteTerms;  -- Empty
SELECT * FROM InvoiceTerms;  -- Empty
SELECT * FROM ChangeOrderTerms;  -- Empty
```

**Expected after testing:** At least 1 QuoteTerms record linking a quote to a template

---

## Summary

**Root Issue:** Frontend and backend code was written assuming a `title` field that doesn't exist in the Prisma schema. The correct field is `name`.

**Impact:** 
- Templates weren't displaying correctly
- PDFs couldn't render terms (field undefined)
- TypeScript types were misleading

**Resolution:**
- ✅ Updated all 6 files to use correct schema fields
- ✅ Backend rebuilt with PDF fix
- ⏳ Frontend rebuild pending
- ⏳ End-to-end testing pending

**Time to Resolution:** ~15 minutes of code changes + testing time

**Prevention:** 
- Always generate TypeScript types from Prisma schema
- Consider using Prisma Client types directly instead of manual interfaces
- Add integration tests that verify schema field names
