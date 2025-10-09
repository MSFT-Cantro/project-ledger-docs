# UserAccount Table Requirements

> **Critical:** Understanding the UserAccount table and why it's required for authentication

---

## 🎯 Overview

The `UserAccount` table is a **critical join table** in ProjectLedger's multi-tenant architecture. It bridges the relationship between Users and Accounts, enabling proper access control and role management.

**Without UserAccount records, users will receive 403 Forbidden errors when accessing protected routes.**

---

## 📊 Table Structure

```sql
CREATE TABLE "UserAccount" (
  id              SERIAL PRIMARY KEY,
  userId          INTEGER NOT NULL REFERENCES "User"(id),
  accountId       INTEGER NOT NULL REFERENCES "Account"(id),
  role            "Role" NOT NULL,
  isPrimary       BOOLEAN NOT NULL DEFAULT false,
  status          "UserAccountStatus" NOT NULL DEFAULT 'ACTIVE',
  joinedAt        TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(userId, accountId)
);
```

---

## 🔐 Why It's Required

### Multi-Tenant Architecture
ProjectLedger uses account-based multi-tenancy where:
- A **User** can belong to multiple **Accounts** (organizations)
- Each User-Account relationship has its own role and status
- The same user can be an ADMIN in one account and a USER in another

### Authentication Middleware
The authentication middleware (`apps/backend/src/middleware/auth.ts`) checks:
1. JWT token is valid ✅
2. User exists in database ✅
3. **UserAccount record exists for the user's current account** ✅
4. UserAccount status is 'ACTIVE' ✅

If step 3 or 4 fails, the middleware returns:
```json
{
  "error": "INVALID_ACCOUNT_ACCESS",
  "message": "User does not have access to this account"
}
```
HTTP Status: **403 Forbidden**

---

## ⚠️ Common Issues

### Issue #1: 403 Errors After Database Seeding

**Symptom:**
- Users can log in successfully
- JWT token is valid
- All protected routes return 403 Forbidden

**Cause:**
- `User` table has records
- `UserAccount` table is empty
- Authentication middleware fails at UserAccount check

**Solution:**
```bash
# Option 1: Run seed script (development)
cd apps/backend
npm run seed

# Option 2: Run UserAccount fix script
node apps/backend/fix-user-accounts.js

# Option 3: Manual SQL (production)
node apps/backend/fix-production-user-accounts.js
```

### Issue #2: New User Registration Without UserAccount

**Symptom:**
- User registration succeeds
- User can't access any protected routes
- 403 errors on all pages after login

**Cause:**
- Registration created User record only
- No UserAccount record created

**Solution:**
Ensure user registration code creates BOTH records:
```typescript
// ❌ WRONG - Only creates User
const user = await prisma.user.create({
  data: { email, password, name, accountId }
})

// ✅ CORRECT - Creates User and UserAccount
const user = await prisma.user.create({
  data: {
    email, password, name, accountId,
    userAccounts: {
      create: {
        accountId,
        role: 'USER',
        isPrimary: true,
        status: 'ACTIVE'
      }
    }
  }
})
```

---

## 🛠️ Verification Commands

### Check for Missing UserAccount Records

```bash
# Count total users
docker run --rm postgres:17.0-alpine psql "$DATABASE_URL" \
  -c "SELECT COUNT(*) as total_users FROM \"User\";"

# Count UserAccount records
docker run --rm postgres:17.0-alpine psql "$DATABASE_URL" \
  -c "SELECT COUNT(*) as total_useraccount FROM \"UserAccount\";"

# Find users WITHOUT UserAccount records (orphans)
docker run --rm postgres:17.0-alpine psql "$DATABASE_URL" \
  -c "SELECT u.id, u.email, u.role, u.\"accountId\" 
      FROM \"User\" u 
      LEFT JOIN \"UserAccount\" ua ON u.id = ua.\"userId\" 
      WHERE ua.id IS NULL;"
```

### Verify Specific User

```bash
# Check if a user has proper UserAccount records
docker run --rm postgres:17.0-alpine psql "$DATABASE_URL" \
  -c "SELECT u.email, u.role as user_role, 
             ua.role as account_role, ua.\"isPrimary\", ua.status
      FROM \"User\" u
      JOIN \"UserAccount\" ua ON u.id = ua.\"userId\"
      WHERE u.email = 'admin@projectledger.com';"
```

Expected output:
```
          email          | user_role | account_role | isPrimary | status 
-------------------------+-----------+--------------+-----------+--------
 admin@projectledger.com | ADMIN     | ADMIN        | t         | ACTIVE
```

---

## 📋 Deployment Checklist

When deploying or seeding a database, **ALWAYS verify UserAccount records**:

### Local Development
- [ ] Run `npm run seed` (automatically creates UserAccount records as of Oct 2025)
- [ ] Verify with: `SELECT COUNT(*) FROM "UserAccount";`
- [ ] Test login and navigation

### Production Deployment
- [ ] Run migrations: `npx prisma migrate deploy`
- [ ] Run production seed: `node seed-production-initial.js`
- [ ] Verify UserAccount records exist
- [ ] Test production login
- [ ] Monitor logs for 403 errors

### Emergency Fix (Production)
If UserAccount records are missing in production:
```bash
# 1. Review what needs to be fixed
node apps/backend/fix-production-user-accounts.js

# 2. Type 'yes' to confirm when prompted

# 3. Verify the fix
docker run --rm postgres:17.0-alpine psql "$PROD_DATABASE_URL" \
  -c "SELECT COUNT(*) FROM \"UserAccount\";"
```

---

## 🔍 How to Debug 403 Errors

### Step 1: Check Authentication Middleware Logs
Look for these patterns in backend logs:
```
❌ 403 /api/user/organizations
❌ 403 /api/organizations/current
❌ 403 /api/subscriptions/plans
```

### Step 2: Verify JWT Token
```bash
# Decode JWT token (copy from browser localStorage or network tab)
echo "YOUR_JWT_TOKEN" | cut -d'.' -f2 | base64 -d | jq
```

Should show:
```json
{
  "userId": 1,
  "accountId": 2,
  "email": "admin@projectledger.com",
  "role": "ADMIN"
}
```

### Step 3: Check UserAccount Table
```sql
SELECT * FROM "UserAccount" 
WHERE "userId" = 1 AND "accountId" = 2;
```

Should return a record with:
- `status`: 'ACTIVE'
- `role`: 'ADMIN' (or appropriate role)

### Step 4: Check Authentication Middleware Code
File: `apps/backend/src/middleware/auth.ts`

The middleware queries:
```typescript
const userAccount = await prisma.userAccount.findFirst({
  where: {
    userId: user.id,
    accountId: req.accountId, // From JWT token
    status: 'ACTIVE'
  }
})

if (!userAccount) {
  // Returns 403 INVALID_ACCOUNT_ACCESS
}
```

---

## 🚀 Fixed in Seed Scripts

As of **October 2025**, the following scripts have been updated to create UserAccount records:

### ✅ apps/backend/prisma/seed.ts (Development)
```typescript
// Now creates UserAccount records for all users
for (const user of users) {
  await prisma.$executeRaw`
    INSERT INTO "UserAccount" ("userId", "accountId", "role", "isPrimary", "status", "joinedAt")
    VALUES (${user.id}, ${user.accountId}, ${user.role}::"Role", true, 'ACTIVE', NOW())
    ON CONFLICT ("userId", "accountId") DO UPDATE SET
    "role" = EXCLUDED."role",
    "status" = EXCLUDED."status"
  `
}
```

### ✅ apps/backend/seed-production-initial.js (Production)
```javascript
// Creates admin user WITH UserAccount record
await prisma.userAccount.create({
  data: {
    userId: adminUser.id,
    accountId: account.id,
    role: 'ADMIN',
    isPrimary: true,
    status: 'ACTIVE'
  }
})
```

### ✅ Fix Scripts Available
- `apps/backend/fix-user-accounts.js` - Local development
- `apps/backend/fix-production-user-accounts.js` - Production safe

---

## 📚 Related Documentation

- [403 Error Fix](../fixes/403_ERROR_FIX.md) - Original incident report
- [Production UserAccount Fix](./PRODUCTION_USERACCOUNT_FIX.md) - Production deployment guide
- [Authentication Middleware](../../apps/backend/src/middleware/auth.ts) - Code reference
- [Deployment Checklist](./DEPLOYMENT_CHECKLIST_TEMPLATE.md) - Includes UserAccount checks

---

## 💡 Best Practices

1. **Always create User and UserAccount together**
   - Use Prisma nested creates
   - Never create User without UserAccount

2. **Verify after every database operation**
   - Count UserAccount records
   - Check for orphaned users

3. **Include in deployment checklists**
   - Add UserAccount verification step
   - Test authentication after deployment

4. **Monitor 403 errors**
   - Set up alerts for 403 patterns
   - Check UserAccount table first

5. **Document any manual fixes**
   - Record when UserAccount records were manually created
   - Include in incident reports

---

## 🎓 Historical Context

### October 9, 2025 - Incident
- **Issue**: Fresh development database seeded without UserAccount records
- **Impact**: All protected routes returned 403 Forbidden
- **Root Cause**: `seed.ts` created Users but not UserAccount records
- **Fix**: Updated `seed.ts` to create UserAccount records
- **Prevention**: 
  - Added to deployment checklist
  - Created this documentation
  - Updated all seed scripts

### Lessons Learned
1. Multi-tenant architecture requires join table records
2. Authentication depends on more than just User table
3. Seed scripts must mirror production requirements
4. Verification steps are critical

---

**Last Updated:** October 9, 2025  
**Maintainer:** Development Team  
**Status:** ✅ Issue Resolved & Documented
