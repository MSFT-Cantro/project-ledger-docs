# Terms Templates Migration - Deployment Guide

## Overview

Terms templates are now **automatically seeded via database migration** during deployment. This ensures that all standard terms and conditions are available in every environment (development, staging, production) without requiring manual seeding scripts.

## Migration Details

**Migration File:** `20251015120000_seed_terms_templates/migration.sql`

**What It Does:**
- Automatically creates 7 standard terms templates on first deployment
- Idempotent: Safe to run multiple times (uses `ON CONFLICT DO NOTHING`)
- Requires at least one User to exist (uses first user as creator)
- Runs as part of normal `prisma migrate deploy` command

## Standard Templates Included

The migration seeds these 7 templates:

| Template Name | Category | Use Case |
|--------------|----------|----------|
| Payment Terms | PAYMENT_TERMS | Payment schedules, late fees, methods |
| Warranty Terms | SERVICE_TERMS | Service warranties and guarantees |
| Cancellation Policy | CANCELLATION | Cancellation rules and fees |
| Liability Limitations | LEGAL_TERMS | Liability caps and indemnification |
| Change Order Process | CUSTOM | Change order procedures |
| Intellectual Property Rights | INTELLECTUAL_PROPERTY | IP ownership and usage |
| Confidentiality Agreement | PRIVACY_DATA | Confidentiality obligations |

## Deployment Process

### Production Deployment

When deploying to production, the migration runs automatically:

```bash
# Standard deployment process
docker compose up -d

# The backend entrypoint automatically runs:
npx prisma migrate deploy
```

**Result:** Terms templates are created if they don't exist.

### Fresh Database Setup

For a brand new database:

```bash
# 1. Run migrations (includes schema + seed data)
docker compose exec backend npx prisma migrate deploy

# 2. Create initial user (if needed)
docker compose exec backend npm run seed

# 3. Verify templates exist
docker compose exec backend node -e "
const { PrismaClient } = require('@prisma/client');
const prisma = new PrismaClient();
prisma.termsTemplate.count().then(count => {
  console.log('Terms templates:', count);
  prisma.\$disconnect();
});"
```

### Azure Deployment

The migration runs automatically during Azure deployment:

```bash
# In docker-entrypoint-production.sh:
echo "Running database migrations..."
npx prisma migrate deploy
```

**No manual intervention required!**

## Verification

After deployment, verify terms templates exist:

### Via API
```bash
curl http://localhost:3001/api/terms-templates/active \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

Should return 7 active templates.

### Via Database
```bash
docker compose exec postgres psql -U projectledger -d projectledger \
  -c "SELECT id, name, category FROM \"TermsTemplate\";"
```

### Via Frontend
1. Navigate to `/quotes/1/edit`
2. Scroll to "Terms & Conditions" section
3. Click dropdown "Select a template"
4. Should see all 7 templates listed

## Troubleshooting

### Issue: Migration runs but no templates created

**Cause:** No users exist in the database yet

**Solution:**
```bash
# The migration will log: "No users found - skipping..."
# Create users first, then run the manual seed script:
docker compose exec backend node seed-terms-templates.js
```

### Issue: Want to add more templates

**Option 1:** Use the API (Recommended)
```bash
POST /api/terms-templates
{
  "name": "New Template",
  "category": "CUSTOM",
  "content": "Template content...",
  "version": "1.0",
  "effectiveDate": "2025-10-15T00:00:00Z",
  "isActive": true
}
```

**Option 2:** Create a new migration
```bash
# Create migration file: 20251015130000_add_custom_terms/migration.sql
INSERT INTO "TermsTemplate" (...)
VALUES (...) ON CONFLICT DO NOTHING;
```

### Issue: Need to update existing templates

**Important:** Don't modify migration files after they've been deployed!

**Solution:** Create a new migration with updates:
```sql
-- 20251015140000_update_payment_terms/migration.sql
UPDATE "TermsTemplate"
SET 
  "content" = 'New content...',
  "version" = '1.1',
  "updatedAt" = CURRENT_TIMESTAMP
WHERE "name" = 'Payment Terms' AND "version" = '1.0';
```

## Migration File Structure

```sql
DO $$
DECLARE
    v_user_id INTEGER;
BEGIN
    -- Get first user as creator
    SELECT id INTO v_user_id FROM "User" ORDER BY id LIMIT 1;
    
    IF v_user_id IS NOT NULL THEN
        -- Insert each template
        INSERT INTO "TermsTemplate" (
            "name", "category", "content", ...
        ) VALUES (
            'Template Name', 'CATEGORY', 'Content...', ...
        ) ON CONFLICT ("name", "version") DO NOTHING;
        
        -- Repeat for each template...
        
        RAISE NOTICE 'Terms templates seeded successfully';
    ELSE
        RAISE NOTICE 'No users found - skipping';
    END IF;
END $$;
```

**Key Features:**
- ✅ Uses `ON CONFLICT DO NOTHING` for idempotency
- ✅ Checks for user existence
- ✅ Assigns first user as creator
- ✅ Sets all timestamps correctly
- ✅ Safe to run multiple times

## Rollback Procedure

If you need to remove the seeded templates:

```sql
-- Remove all terms templates (CAREFUL!)
DELETE FROM "QuoteTerms";
DELETE FROM "InvoiceTerms";
DELETE FROM "ChangeOrderTerms";
DELETE FROM "TermsTemplate" WHERE version = '1.0';
```

**Note:** This will also remove any terms attached to documents!

## Best Practices

### DO:
✅ Let the migration seed templates automatically
✅ Use the API to create custom templates
✅ Create new migrations for template updates
✅ Test migrations in staging before production
✅ Verify template count after deployment

### DON'T:
❌ Modify migration files after deployment
❌ Delete templates that are in use
❌ Manually INSERT templates (use API instead)
❌ Change the unique constraint (name + version)
❌ Run seed scripts in production (migration handles it)

## Related Files

**Migration:**
- `apps/backend/prisma/migrations/20251015120000_seed_terms_templates/migration.sql`

**Manual Seed Script** (for development only):
- `apps/backend/seed-terms-templates.js`

**API Routes:**
- `apps/backend/src/routes/terms-templates.ts`

**Frontend Components:**
- `apps/frontend/src/components/TermsManagement/`

**Documentation:**
- `docs/HOW_TO_CREATE_TERMS_TEMPLATES.md`
- `docs/QUICK_TEST_GUIDE_TERMS.md`
- `docs/fixes/TERMS_SCHEMA_FIXES.md`

## Migration History

| Migration | Date | Purpose |
|-----------|------|---------|
| 20251015104457_add_terms_and_conditions_system | 2025-10-15 | Created tables and schema |
| 20251015120000_seed_terms_templates | 2025-10-15 | Seeded standard templates |

## Summary

✅ **Automated:** Terms templates seed automatically during deployment
✅ **Idempotent:** Safe to run multiple times
✅ **Production-Ready:** No manual intervention required
✅ **Rollback-Safe:** Can be undone if needed
✅ **Documented:** Full deployment guide available

**Next Deployment:** Templates will be automatically available in all environments!
