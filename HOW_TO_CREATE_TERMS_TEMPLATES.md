# How to Create Terms Templates

## Quick Start: Templates Already Created!

✅ **7 standard terms templates have been seeded into your database**:
1. Payment Terms
2. Warranty Terms
3. Cancellation Policy
4. Liability Limitations
5. Change Order Process
6. Intellectual Property Rights
7. Confidentiality Agreement

You can now use these immediately when editing quotes, invoices, or change orders!

---

## Using Terms in Your Documents

### Step-by-Step Guide:

1. **Navigate to a Document Edit Page**
   - Go to `/quotes/1/edit` (or any quote ID)
   - Or `/invoices/1/edit`
   - Or `/change-orders/1/edit`

2. **Scroll Down to "Terms & Conditions" Section**
   - You'll see a section titled "Terms & Conditions"
   - Below the main form

3. **Select a Template**
   - Click the dropdown "Select a template"
   - Choose a terms template (e.g., "Payment Terms")
   - You'll see a preview of the content below the dropdown

4. **Add the Term**
   - Click the "Add" button
   - The term will appear in the "Attached Terms" list below

5. **Reorder Terms (Optional)**
   - Use the ↑ and ↓ arrow buttons to change the order
   - Terms will appear in PDFs in this order

6. **Edit Custom Content (Optional)**
   - Click the ✏️ (edit) icon on any term
   - Modify the content for this specific document
   - Click "Save Changes"
   - The term will show an "Customized" badge

7. **Generate PDF**
   - Save your document changes
   - Go back to the detail view
   - Click "Download PDF"
   - **Your terms will now appear in the PDF!**

---

## Creating New Terms Templates via API

If you need to create additional custom terms templates:

### Endpoint
```
POST /api/terms-templates
Authorization: Bearer <your-jwt-token>
Content-Type: application/json
```

### Request Body
```json
{
  "name": "Dispute Resolution",
  "description": "Standard arbitration clause",
  "category": "DISPUTE_RESOLUTION",
  "content": "Any disputes arising from this agreement shall be resolved through binding arbitration in accordance with the rules of the American Arbitration Association. The arbitration shall take place in [City, State]. Each party shall bear their own costs and fees.",
  "version": "1.0",
  "effectiveDate": "2025-10-15T00:00:00Z",
  "isActive": true,
  "isDefault": false,
  "requiresAcceptance": false
}
```

### Valid Categories
Choose one of these for the `category` field:
- `PAYMENT_TERMS` - Payment schedules, late fees, methods
- `SERVICE_TERMS` - Service delivery, warranties, limitations
- `LEGAL_TERMS` - Liability, indemnification, governing law
- `CANCELLATION` - Cancellation policies, refunds
- `INTELLECTUAL_PROPERTY` - IP ownership, usage rights
- `PRIVACY_DATA` - Data handling, privacy policies
- `DISPUTE_RESOLUTION` - Arbitration, mediation procedures
- `CUSTOM` - Client-specific or specialized terms

### Example with cURL
```bash
curl -X POST http://localhost:3001/api/terms-templates \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Dispute Resolution",
    "category": "DISPUTE_RESOLUTION",
    "content": "Any disputes shall be resolved through binding arbitration.",
    "version": "1.0",
    "effectiveDate": "2025-10-15T00:00:00Z",
    "isActive": true,
    "isDefault": false,
    "requiresAcceptance": false
  }'
```

---

## API Endpoints Reference

### Get All Active Templates
```
GET /api/terms-templates/active
```

### Get All Templates (including inactive)
```
GET /api/terms-templates
```

### Get Specific Template
```
GET /api/terms-templates/:id
```

### Create Template
```
POST /api/terms-templates
```

### Update Template
```
PATCH /api/terms-templates/:id
```

### Archive Template
```
POST /api/terms-templates/:id/archive
```

---

## Attaching Terms to Documents via API

If you prefer to use the API directly instead of the UI:

### Add Term to Quote
```bash
POST /api/quotes/:quoteId/terms
Content-Type: application/json

{
  "termsTemplateId": 1,
  "position": 1,
  "customContent": "Optional: custom content for this specific quote"
}
```

### Add Term to Invoice
```bash
POST /api/invoices/:invoiceId/terms
Content-Type: application/json

{
  "termsTemplateId": 1,
  "position": 1
}
```

### Add Term to Change Order
```bash
POST /api/change-orders/:changeOrderId/terms
Content-Type: application/json

{
  "termsTemplateId": 1,
  "position": 1
}
```

---

## Troubleshooting

### "No templates appear in the dropdown"

1. **Check if templates exist:**
   ```bash
   docker compose exec backend node -e "
   const { PrismaClient } = require('@prisma/client');
   const prisma = new PrismaClient();
   prisma.termsTemplate.findMany().then(t => {
     console.log('Templates:', t.length);
     t.forEach(x => console.log('  -', x.name));
     prisma.\$disconnect();
   });
   "
   ```

2. **Re-run the seed script:**
   ```bash
   docker compose exec backend node seed-terms-templates.js
   ```

### "Terms don't appear in PDF"

1. **Verify terms are attached:**
   - Check the "Attached Terms" section shows your terms
   - Ensure you saved the document after adding terms

2. **Check the API directly:**
   ```bash
   curl http://localhost:3001/api/quotes/1/terms \
     -H "Authorization: Bearer YOUR_TOKEN"
   ```

3. **Check backend logs:**
   ```bash
   docker compose logs backend | grep -i term
   ```

### "Frontend shows error when adding terms"

1. **Check browser console** (F12) for errors
2. **Verify you're authenticated** - JWT token must be valid
3. **Check backend logs** for API errors

---

## Next Steps

1. ✅ **Terms templates are seeded** (7 standard templates)
2. 🎯 **Try adding a term to a quote** using the UI
3. 📄 **Generate a PDF** and verify terms appear
4. 🎨 **Customize content** for specific documents as needed
5. ➕ **Create additional templates** via API if needed

---

## Need Help?

- Frontend Components: `apps/frontend/src/components/TermsManagement/`
- Backend Routes: `apps/backend/src/routes/terms-templates.ts`
- Database Schema: `apps/backend/prisma/schema.prisma` (search for "TermsTemplate")
- API Documentation: This file
