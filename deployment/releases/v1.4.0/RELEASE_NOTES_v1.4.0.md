# Production Release v1.4.0

> **Release Version:** 1.4.0  
> **Last Updated:** October 9, 2025  
> **Purpose:** Invoice and Quote Template Functionality

---

## Release Information

**Version:** `v1.4.0`  
**Release Date:** `October 9, 2025`  
**Deployer:** `[Your Name]`  
**Revision Suffix:** `v1-4-0`

**Changes in this release:**
- ✅ Added invoice template functionality (Item#33)
- ✅ Added quote template functionality (Item#32)
- ✅ Users can save and reuse invoice templates
- ✅ Users can save and reuse quote templates
- ✅ Template management UI with create, view, edit, delete
- ✅ Set default templates for quick access
- ✅ All users in account can access shared templates
- ✅ Improved UX on invoice create page (grouped back/more buttons)

---

## 📋 Pre-Release Checklist

### Code & Documentation
- [x] All features tested locally
- [x] All tests passing (`npm test` in backend and frontend)
- [x] Code reviewed and approved
- [ ] Changelog updated with release notes
- [x] Documentation updated (if applicable)
- [x] Bug list updated (mark completed items)
- [x] No TypeScript errors
- [x] No console errors in browser
- [ ] **Marketing post created** (see Marketing Content section below)

### Git & Version Control
- [x] All changes committed to `main` branch
- [x] Git status is clean (no uncommitted changes)
- [x] All commits pushed to GitHub
- [ ] Version numbers updated in `package.json` files (if applicable)

### Environment & Access
- [ ] Logged into Azure CLI (`az login`)
- [ ] Docker Desktop running
- [ ] Access to Azure Container Registry verified
- [ ] Environment variables verified (no changes needed, or changes documented)

### Database (CRITICAL - October 8, 2025 Data Loss Prevention)
- [ ] 🚨 **MANDATORY: Production database backup created** (`./tools/backup/backup-database.sh deployment`)
- [ ] 🚨 **MANDATORY: Database state verified** (`DATABASE_URL=... node scripts/verify-production-data.js`)
- [ ] 🚨 **MANDATORY: Migration status checked** (not "out of sync" or empty)
- [x] Database migrations created (if needed) - Two migrations: 20251009183518_add_user_quote_templates and 20251009201915_add_user_invoice_templates
- [x] Migration scripts tested locally
- [x] Rollback migration scripts prepared (if applicable)
- [ ] 🚨 **MANDATORY: Recovery plan documented** (location of backup, restore commands)

---

## 📣 Marketing Content Creation

### Create Marketing Post (MANDATORY)

Before deployment, create a user-facing marketing post to announce the release.

#### **Location:**
`docs/deployment/releases/v1.4.0/MARKETING_POST_v1.4.0.md`

#### **Key Features to Highlight:**

1. **Invoice Templates** - Save time by creating reusable invoice templates
2. **Quote Templates** - Quickly generate quotes from saved templates
3. **Team Collaboration** - All team members can access shared templates
4. **Default Templates** - Set your most-used template as default for one-click access

#### **Distribution Checklist:**
- [ ] Marketing post created and reviewed
- [ ] Post added to release documentation folder
- [ ] Ready for blog/website publication
- [ ] Ready for email newsletter
- [ ] Ready for social media adaptation
- [ ] Key talking points identified for support team

---

## 🚀 Deployment Process

### Phase 1: Pre-Deployment (5 minutes)

#### 1.1 Final Verification
```bash
# Navigate to project
cd C:/Code/ProjectLedger2

# Check git status
git status

# Verify logged into Azure
az account show

# Verify Docker running
docker ps
```

#### 1.2 Create Git Tag
```bash
# Create and push tag
git tag v1.4.0
git push origin v1.4.0
```

#### 1.3 🚨 CRITICAL: Database Safety Checks
```bash
# Step 1: Create database backup (MANDATORY)
./tools/backup/backup-database.sh deployment "Pre-deployment backup for v1.4.0"

# Step 2: Verify database state (MANDATORY)
cd apps/backend
DATABASE_URL="postgresql://postgres:HzxHJdKgjLaamQSqYUUdD8oE8@projectledger-db.eastus2.azurecontainer.io:5432/projectledger" \
node scripts/verify-production-data.js

# Expected output: ✅ VERIFICATION PASSED - DATABASE HEALTHY
# If FAILED or WARNINGS: INVESTIGATE BEFORE PROCEEDING

# Step 3: Check migration status (MANDATORY)
DATABASE_URL="postgresql://postgres:HzxHJdKgjLaamQSqYUUdD8oE8@projectledger-db.eastus2.azurecontainer.io:5432/projectledger" \
npx prisma migrate status

# Expected: "Database schema is up to date!"
# If "out of sync": DO NOT DEPLOY - investigate immediately
```

#### 1.4 Backup Current Revisions
```bash
# Create backup directory
mkdir -p .deployment-backup

# Get current backend revision
az containerapp revision list \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --query "[0].name" -o tsv > .deployment-backup/backend-revision-$(date +%Y%m%d-%H%M%S).txt

# Get current frontend revision
az containerapp revision list \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --query "[0].name" -o tsv > .deployment-backup/frontend-revision-$(date +%Y%m%d-%H%M%S).txt
```

---

### Phase 2: Build Docker Images (10-15 minutes)

#### 2.1 Login to Azure Container Registry
```bash
az acr login --name projectledgerregistry
```

#### 2.2 Build Backend Image
```bash
cd apps/backend

docker build \
  --target production \
  -t projectledgerregistry.azurecr.io/projectledger-backend:latest \
  -t projectledgerregistry.azurecr.io/projectledger-backend:v1.4.0 \
  -f Dockerfile \
  ../..
```

**Expected Output:** 
- Build should complete without errors
- Image size typically 400-600 MB

#### 2.3 Build Frontend Image
```bash
cd ../frontend

docker build \
  --target azure \
  -t projectledgerregistry.azurecr.io/projectledger-frontend:latest \
  -t projectledgerregistry.azurecr.io/projectledger-frontend:v1.4.0 \
  -f Dockerfile \
  ../..
```

**Expected Output:**
- Build should complete without errors
- Image size typically 50-100 MB

---

### Phase 3: Push Images (3-5 minutes)

```bash
# Push backend images
docker push projectledgerregistry.azurecr.io/projectledger-backend:latest
docker push projectledgerregistry.azurecr.io/projectledger-backend:v1.4.0

# Push frontend images
docker push projectledgerregistry.azurecr.io/projectledger-frontend:latest
docker push projectledgerregistry.azurecr.io/projectledger-frontend:v1.4.0
```

---

### Phase 4: Deploy to Azure (5-10 minutes)

#### 4.1 Deploy Backend
```bash
az containerapp update \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --image projectledgerregistry.azurecr.io/projectledger-backend:v1.4.0 \
  --revision-suffix v1-4-0
```

**Wait 15 seconds for stabilization**

#### 4.2 Deploy Frontend
```bash
az containerapp update \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --image projectledgerregistry.azurecr.io/projectledger-frontend:v1.4.0 \
  --revision-suffix v1-4-0
```

**Wait 15 seconds for stabilization**

---

### Phase 5: Verification & Testing (10-15 minutes)

#### 5.1 Health Checks

```bash
# Test frontend
curl -I https://app.projectledger.ca
# Expected: HTTP/1.1 200 OK

# Test backend health
curl https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io/api/health
# Expected: {"status":"ok","database":"connected","timestamp":"..."}
```

#### 5.2 Manual Testing Checklist

**Core Functionality:**
- [ ] Frontend loads without errors
- [ ] User can log in successfully
- [ ] Dashboard displays correctly
- [ ] Navigation works (all menu items)
- [ ] Mobile view renders correctly

**New Features (Invoice Templates):**
- [ ] Can save invoice as template from invoice detail page
- [ ] Template manager opens from invoice create page
- [ ] Can select and load template when creating invoice
- [ ] Can set default template
- [ ] Can delete templates
- [ ] All users in account can see templates
- [ ] Template data loads correctly (client, project, items, notes)

**New Features (Quote Templates):**
- [ ] Can save quote as template from quote detail page
- [ ] Template manager opens from quote create page
- [ ] Can select and load template when creating quote
- [ ] Can set default template
- [ ] Can delete templates
- [ ] All users in account can see templates
- [ ] Template data loads correctly (client, project, items, notes)

**Data Operations:**
- [ ] Can create new records
- [ ] Can read existing records
- [ ] Can update records
- [ ] Can delete records
- [ ] Search/filter functionality works

**Integration Points:**
- [ ] API calls succeed
- [ ] Authentication tokens work
- [ ] External integrations functional (PayPal, Beamer, etc.)

**Browser Testing:**
- [ ] Chrome (desktop)
- [ ] Firefox (desktop)
- [ ] Safari (if available)
- [ ] Mobile browsers (iOS/Android)

**Performance:**
- [ ] Page load time < 3 seconds
- [ ] API response time < 1 second
- [ ] No console errors
- [ ] No 404 or 500 errors in Network tab

#### 5.3 Log Monitoring

```bash
# Watch backend logs (in separate terminal)
az containerapp logs show \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --tail 50 \
  --follow

# Watch frontend logs (in separate terminal)
az containerapp logs show \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --tail 50 \
  --follow
```

**Look for:**
- [ ] No error messages
- [ ] Successful startup logs
- [ ] Database connection confirmed
- [ ] No authentication failures
- [ ] Migrations applied successfully (UserInvoiceTemplate and UserQuoteTemplate tables created)

---

## 🔙 Rollback Procedure

**Time to Rollback:** < 2 minutes

### If Issues Detected

#### 1. List Available Revisions
```bash
# Backend revisions
az containerapp revision list \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  -o table

# Frontend revisions
az containerapp revision list \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  -o table
```

#### 2. Activate Previous Revision

```bash
# Rollback backend
az containerapp revision activate \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --revision [previous-backend-revision-name]

# Rollback frontend
az containerapp revision activate \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --revision [previous-frontend-revision-name]
```

#### 3. Verify Rollback
```bash
# Test that previous version is working
curl -I https://app.projectledger.ca
```

#### 4. Document Issue
- Create incident report in `docs/deployment/incidents/`
- Include: timestamp, symptoms, root cause, resolution
- Update rollback log

---

## 📊 Post-Deployment Monitoring

### First Hour
- [ ] Monitor logs continuously
- [ ] Check for error spikes
- [ ] Verify API response times
- [ ] Monitor user activity (if available)
- [ ] Check memory/CPU usage in Azure Portal
- [ ] Verify template API endpoints working (/api/user-invoice-templates, /api/user-quote-templates)

### First Day
- [ ] Review error logs (morning, afternoon, evening)
- [ ] Check user feedback channels
- [ ] Monitor database performance
- [ ] Verify scheduled jobs/cron tasks
- [ ] Check external integration logs
- [ ] Monitor template creation/usage

### First Week
- [ ] Daily log reviews
- [ ] Performance metrics analysis
- [ ] User satisfaction check
- [ ] Bug reports triage
- [ ] Plan any hotfixes if needed
- [ ] Track template adoption rate

---

## 📈 Success Metrics

### Technical Metrics
- [ ] Deployment completed within estimated time
- [ ] Zero downtime achieved
- [ ] No rollback required
- [ ] Error rate < 1%
- [ ] API response time < 1s (p95)
- [ ] Page load time < 3s
- [ ] Database migrations applied successfully

### Business Metrics
- [ ] All new features functional
- [ ] No critical bugs reported
- [ ] User satisfaction maintained/improved
- [ ] No data loss or corruption
- [ ] Template features accessible to all users

---

## 📝 Post-Release Tasks

### Immediate (Day 0)
- [ ] Update deployment documentation
- [ ] Mark completed items in bug/feature lists (Item#32 and Item#33)
- [ ] Update CHANGELOG.md with release notes
- [ ] **Publish marketing post** to blog/website
- [ ] **Send release announcement** via email newsletter (if applicable)
- [ ] **Share on social media** (adapt marketing post for platform)
- [ ] **Update support team** with key talking points from marketing post
- [ ] Send release notification (if applicable)
- [ ] Archive deployment logs

### Short-term (Week 1)
- [ ] Review deployment process for improvements
- [ ] Update template documentation if needed
- [ ] Document any lessons learned
- [ ] Plan next release (if applicable)
- [ ] Gather user feedback on template functionality

### Long-term
- [ ] Review metrics after 2 weeks
- [ ] Collect user feedback
- [ ] Update roadmap based on learnings
- [ ] Consider additional template features based on usage

---

## 🚨 Emergency Contacts

**Azure Support:** [Azure Portal](https://portal.azure.com)  
**GitHub Issues:** [Repository Issues](https://github.com/MSFT-Cantro/project-ledger/issues)  
**Database Admin:** [Contact Info]  
**Project Lead:** [Contact Info]

---

## 📚 Related Documentation

- [Production Release Plan](../PRODUCTION_RELEASE_PLAN.md) - Detailed procedures
- [Azure Deployment Guide](../AZURE_DEPLOYMENT_COMPLETE.md) - Infrastructure setup
- [Database Seeding](../DATABASE_SEEDING.md) - Database management
- [Monitoring & Alerting](../MONITORING_ALERTING_PAYPAL.md) - Monitoring setup
- [HOW_TO_RELEASE.md](../HOW_TO_RELEASE.md) - Release process guide

---

## 📌 Technical Details

### Database Changes
- **Migration:** `20251009183518_add_user_quote_templates`
  - Adds `UserQuoteTemplate` table
  - Relations to `Account` and `User` models
  - JSON field for template data storage

- **Migration:** `20251009201915_add_user_invoice_templates`
  - Adds `UserInvoiceTemplate` table
  - Relations to `Account` and `User` models
  - JSON field for template data storage

### New API Endpoints
- `GET /api/user-invoice-templates` - List invoice templates
- `POST /api/user-invoice-templates` - Create invoice template
- `GET /api/user-invoice-templates/:id` - Get invoice template
- `PATCH /api/user-invoice-templates/:id` - Update invoice template
- `DELETE /api/user-invoice-templates/:id` - Delete invoice template
- `GET /api/user-quote-templates` - List quote templates
- `POST /api/user-quote-templates` - Create quote template
- `GET /api/user-quote-templates/:id` - Get quote template
- `PATCH /api/user-quote-templates/:id` - Update quote template
- `DELETE /api/user-quote-templates/:id` - Delete quote template

### New Components
- `InvoiceTemplateManager.tsx` - Template selection modal
- `SaveInvoiceTemplateModal.tsx` - Save template dialog
- `QuoteTemplateManager.tsx` - Template selection modal
- `SaveQuoteTemplateModal.tsx` - Save template dialog

---

## 📋 Release Checklist Summary

**Pre-Deployment:**
- [ ] All pre-release checklist items completed
- [ ] Git tag created and pushed
- [ ] Current revisions backed up
- [ ] Database backup created
- [ ] Migration status verified

**Deployment:**
- [ ] Docker images built successfully
- [ ] Images pushed to ACR
- [ ] Backend deployed with new revision
- [ ] Frontend deployed with new revision

**Verification:**
- [ ] Health checks passing
- [ ] Manual testing completed
- [ ] Logs show no errors
- [ ] Performance metrics acceptable
- [ ] Template features working

**Post-Deployment:**
- [ ] Monitoring in place (first hour)
- [ ] Documentation updated
- [ ] Team notified
- [ ] Success metrics tracked

---

## 📌 Notes & Lessons Learned

**What went well:**
- [Add after deployment]

**What could be improved:**
- [Add after deployment]

**Action items:**
- [Add after deployment]

---

**Deployment Status:** ⬜ Not Started | ⏳ In Progress | ✅ Completed | ❌ Failed

**Deployed By:** ________________  
**Date & Time:** ________________  
**Sign-off:** ________________
