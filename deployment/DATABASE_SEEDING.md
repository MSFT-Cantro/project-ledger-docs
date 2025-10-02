# 🌱 Database Seeding Guide

**Purpose:** Populate the production database with required initial data  
**When to Run:** After initial deployment or when adding new required data  
**Safety:** All seed scripts are idempotent (safe to run multiple times)

---

## ⚠️ CRITICAL: Subscription Plans

**Must be run after initial deployment or signup will fail!**

The signup page requires subscription plans to be available. Without seeding, users will see an empty pricing page.

### **Seed Subscription Plans**

```bash
cd apps/backend

DATABASE_URL="postgresql://postgres:HzxHJdKgjLaamQSqYUUdD8oE8@projectledger-db.eastus2.azurecontainer.io:5432/projectledger" \
npx ts-node prisma/seed-subscriptions.ts
```

**What this creates:**

| Plan | Price | Limits | Features |
|------|-------|--------|----------|
| **Free** | $0/month | 25 clients, 25 projects, 25 quotes, 25 invoices, 1 user | Basic support |
| **Professional** | $99.99/month | Unlimited everything | Inventory, multiple users, priority support, advanced reporting |

**Verify it worked:**
```bash
curl https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io/api/subscriptions/plans/public
```

Expected output:
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Free",
      "slug": "free",
      "price": 0,
      ...
    },
    {
      "id": 2,
      "name": "Professional",
      "slug": "professional",
      "price": 99.99,
      ...
    }
  ]
}
```

---

## 🇨🇦 Canadian Tax Rates

**Required for Canadian customers to see correct tax calculations**

### **Seed Tax Rates**

```bash
cd apps/backend

DATABASE_URL="postgresql://postgres:HzxHJdKgjLaamQSqYUUdD8oE8@projectledger-db.eastus2.azurecontainer.io:5432/projectledger" \
node seed-canada.js
```

**What this creates:**

- **GST:** 5% federal tax (applies to all provinces)
- **HST:** Harmonized sales tax for ON, NS, NB, PE, NL (13-15%)
- **PST:** Provincial sales tax for BC, SK, MB, QC (6-10%)
- **Combined rates:** Automatic calculation of total tax per province

**Province Tax Rates:**

| Province | GST | PST | HST | Total |
|----------|-----|-----|-----|-------|
| Alberta (AB) | 5% | - | - | 5% |
| British Columbia (BC) | 5% | 7% | - | 12% |
| Manitoba (MB) | 5% | 7% | - | 12% |
| New Brunswick (NB) | - | - | 15% | 15% |
| Newfoundland (NL) | - | - | 15% | 15% |
| Nova Scotia (NS) | - | - | 15% | 15% |
| Ontario (ON) | - | - | 13% | 13% |
| Prince Edward Island (PE) | - | - | 15% | 15% |
| Quebec (QC) | 5% | 9.975% | - | 14.975% |
| Saskatchewan (SK) | 5% | 6% | - | 11% |

**Verify it worked:**
```bash
# Check how many tax rates were created
curl https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io/api/tax/rates
```

---

## 🔍 Checking Database Status

Use this script to verify all seeding:

```bash
cd apps/backend

DATABASE_URL="postgresql://postgres:HzxHJdKgjLaamQSqYUUdD8oE8@projectledger-db.eastus2.azurecontainer.io:5432/projectledger" \
npx ts-node check-db-status.ts
```

Expected output:
```
Checking database status...

📋 Tables in database: 35
  - Account
  - SubscriptionPlan
  - TaxRate
  ... (and 32 more)

📊 Checking key tables:
  SubscriptionPlan: 2 records ✅
  TaxRate: 20+ records ✅
  Account: X records
  User: X records

🔄 Migrations applied: 21

✅ Database check complete!
```

---

## 🛡️ Safety & Best Practices

### **Idempotent Operations**

All seed scripts use `INSERT ... ON CONFLICT DO UPDATE` or similar patterns:
- ✅ Safe to run multiple times
- ✅ Updates existing records instead of creating duplicates
- ✅ Won't break if data already exists

### **When to Re-run Seeds**

**Subscription Plans:**
- After database reset
- When adding new subscription tiers
- When updating plan features or pricing

**Tax Rates:**
- After database reset
- When tax rates change (update the seed script first)
- When adding support for new provinces/states

### **Rollback**

If you need to remove seeded data:

```bash
# Remove subscription plans (will cascade to related records)
DATABASE_URL="..." npx prisma db execute --stdin << 'EOF'
DELETE FROM "SubscriptionPlan" WHERE slug IN ('free', 'professional');
EOF

# Remove tax rates
DATABASE_URL="..." npx prisma db execute --stdin << 'EOF'
DELETE FROM "TaxRate" WHERE country = 'CA';
EOF
```

⚠️ **Warning:** Only do this if you're sure! This will affect all users and accounts.

---

## 📋 Deployment Checklist

After initial Azure deployment:

- [ ] Run database migrations: `npx prisma migrate deploy`
- [ ] **Seed subscription plans** (CRITICAL!)
- [ ] Seed Canadian tax rates
- [ ] Verify plans endpoint returns data
- [ ] Test signup page shows pricing
- [ ] Test creating an account with Free plan
- [ ] Verify tax calculations work for Canadian address

---

## 🔗 Related Documentation

- **Deployment Plan:** [DEPLOYMENT_PLAN.md](DEPLOYMENT_PLAN.md) - See Step 3.5
- **Azure Deployment:** [AZURE_DEPLOYMENT_COMPLETE.md](AZURE_DEPLOYMENT_COMPLETE.md)
- **Database Status Check:** `apps/backend/check-db-status.ts`

---

## 💡 Tips

1. **Always check if seeding is needed** before running (use check-db-status.ts)
2. **Seed scripts are safe** - they won't duplicate data
3. **Subscription plans are required** - signup will fail without them
4. **Tax rates are optional** - but needed for accurate calculations
5. **Keep seed scripts in version control** - they're part of your infrastructure

---

**Need help?** Check the seed script source code:
- `apps/backend/prisma/seed-subscriptions.ts`
- `apps/backend/seed-canada.js`
- `apps/backend/check-db-status.ts`
