# Production Release v1.5.0

> **Release Version:** 1.5.0  
> **Last Updated:** October 14, 2025  
> **Purpose:** Change Orders System - Complete Implementation (Phases 3 & 4)

---

## Release Information

**Version:** `v1.5.0`  
**Release Date:** `October 14, 2025`  
**Deployer:** `[Your Name]`  
**Revision Suffix:** `v1-5-0`

**Changes in this release:**
- ✅ **Change Orders List & Detail Pages** - Complete UI with filtering, sorting, and search
- ✅ **Change Order Creation Form** - Full-featured form with item management (add/modify/remove)
- ✅ **Client Approval Workflow** - Token-based approval system with email notifications
- ✅ **PDF Generation** - Professional change order PDFs with branding
- ✅ **Project Integration** - Change orders display on project detail pages
- ✅ **Reporting & Analytics** - Comprehensive change order reports with financial metrics
- ✅ **Status Management** - Complete workflow (DRAFT → SENT → APPROVED → EXECUTED)
- ✅ **Email Service** - Automated notifications for approval requests and confirmations
- ✅ **Financial Impact Calculations** - Automatic delta calculations (additive/deductive)
- ✅ **Audit Trail** - Complete history tracking for all change order actions

---

## 📋 Pre-Release Checklist

### Code & Documentation
- [x] All features tested locally
- [x] All tests passing (`npm test` in backend and frontend)
- [x] Code reviewed and approved
- [ ] Changelog updated with release notes
- [x] Documentation updated (SPEC_change_orders.md)
- [x] Bug list updated (mark completed items)
- [x] No TypeScript errors
- [x] No console errors in browser
- [ ] **Marketing post created** (see Marketing Content section below)

### Git & Version Control
- [x] All changes committed to `main` branch
- [x] Git status is clean (no uncommitted changes)
- [x] All commits pushed to GitHub
- [ ] Version numbers updated in `package.json` files

### Environment & Access
- [ ] Logged into Azure CLI (`az login`)
- [ ] Docker Desktop running
- [ ] Access to Azure Container Registry verified
- [ ] Environment variables verified (no changes needed, or changes documented)

### Database (CRITICAL - October 8, 2025 Data Loss Prevention)
- [ ] 🚨 **MANDATORY: Production database backup created** (`./tools/backup/backup-database.sh deployment`)
- [ ] 🚨 **MANDATORY: Database state verified** (`DATABASE_URL=... node scripts/verify-production-data.js`)
- [ ] 🚨 **MANDATORY: Migration status checked** (not "out of sync" or empty)
- [x] Database migrations created (if needed) - One migration: 20251013000002_add_change_order_approval_token
- [x] Migration scripts tested locally
- [x] Rollback migration scripts prepared (if applicable)
- [ ] 🚨 **MANDATORY: Recovery plan documented** (location of backup, restore commands)

---

## 📣 Marketing Content Creation

### Create Marketing Post (MANDATORY)

Before deployment, create a user-facing marketing post to announce the release.

#### **Location:**
`docs/deployment/releases/v1.5.0/MARKETING_POST_v1.5.0.md`

#### **Key Features to Highlight:**

1. **Change Order Management** - Track and manage scope changes professionally
2. **Client Approval Workflow** - Secure token-based approval system
3. **Financial Transparency** - Automatic calculation of additive/deductive changes
4. **PDF Generation** - Professional branded documents for client approval
5. **Complete Audit Trail** - Track every change with full history
6. **Project Integration** - See all change orders related to each project

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
git tag v1.5.0
git push origin v1.5.0
```

#### 1.3 🚨 CRITICAL: Database Safety Checks
```bash
# Step 1: Create database backup (MANDATORY)
./tools/backup/backup-database.sh deployment "Pre-deployment backup for v1.5.0"

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
docker build \
  --target production \
  -t projectledgerregistry.azurecr.io/projectledger-backend:latest \
  -t projectledgerregistry.azurecr.io/projectledger-backend:v1.5.0 \
  -f apps/backend/Dockerfile \
  .
```

**Expected Output:** 
- Build should complete without errors
- Image size typically 400-600 MB

#### 2.3 Build Frontend Image
```bash
docker build \
  --target azure \
  -t projectledgerregistry.azurecr.io/projectledger-frontend:latest \
  -t projectledgerregistry.azurecr.io/projectledger-frontend:v1.5.0 \
  -f apps/frontend/Dockerfile \
  .
```

**Expected Output:**
- Build should complete without errors
- Image size typically 50-100 MB

---

### Phase 3: Push Images (3-5 minutes)

```bash
# Push backend images
docker push projectledgerregistry.azurecr.io/projectledger-backend:latest
docker push projectledgerregistry.azurecr.io/projectledger-backend:v1.5.0

# Push frontend images
docker push projectledgerregistry.azurecr.io/projectledger-frontend:latest
docker push projectledgerregistry.azurecr.io/projectledger-frontend:v1.5.0
```

---

### Phase 4: Deploy to Azure (5-10 minutes)

#### 4.1 Deploy Backend
```bash
az containerapp update \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --image projectledgerregistry.azurecr.io/projectledger-backend:v1.5.0 \
  --revision-suffix v1-5-0
```

**Wait 15 seconds for stabilization**

#### 4.2 Deploy Frontend
```bash
az containerapp update \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --image projectledgerregistry.azurecr.io/projectledger-frontend:v1.5.0 \
  --revision-suffix v1-5-0
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
- [ ] Navigation works (all menu items including "Change Orders")
- [ ] Mobile view renders correctly

**New Features (Change Orders - List & Detail):**
- [ ] Change Orders menu item visible and functional
- [ ] Change orders list page loads successfully
- [ ] Can filter change orders by status, type, and date range
- [ ] Can search change orders
- [ ] Stats bar shows accurate metrics (Total Delta, Approved, Executed, Approval Rate)
- [ ] "View Reports" button navigates to Change Orders report
- [ ] Detail page displays all change order information
- [ ] Status badges display correctly
- [ ] Financial impact summary calculates correctly

**New Features (Change Order Creation):**
- [ ] "Create Change Order" button works
- [ ] Can select quote from dropdown
- [ ] Can add new line items
- [ ] Can modify existing quote items
- [ ] Can remove quote items
- [ ] Financial impact preview updates in real-time
- [ ] Can save as DRAFT
- [ ] Can send for approval

**New Features (Client Approval Workflow):**
- [ ] Can send change order to client
- [ ] Approval email is sent successfully
- [ ] Approval link generates with token
- [ ] Public approval page loads without login
- [ ] Client can approve change order
- [ ] Client can decline change order
- [ ] Approval status updates correctly
- [ ] Confirmation email sent after approval/decline

**New Features (PDF Generation):**
- [ ] "Generate PDF" button visible on detail page
- [ ] PDF downloads successfully
- [ ] PDF contains all change order details
- [ ] PDF displays financial impact correctly
- [ ] PDF includes approval section (if applicable)

**New Features (Project Integration):**
- [ ] Project detail page shows related change orders
- [ ] Count badges display on accordions
- [ ] Change orders table shows quote reference
- [ ] Can navigate to change order detail from project page

**New Features (Reporting):**
- [ ] Change Orders report accessible from /reports/change-orders
- [ ] Report shows summary metrics (total, approved, executed, approval rate)
- [ ] Status distribution chart displays correctly
- [ ] Report table lists all change orders with financial data
- [ ] Can export report (if feature enabled)

**Data Operations:**
- [ ] Can create new change orders
- [ ] Can read existing change orders
- [ ] Can update draft change orders
- [ ] Can delete draft change orders
- [ ] Search/filter functionality works

**Integration Points:**
- [ ] API calls succeed (/api/change-orders endpoints)
- [ ] Authentication tokens work
- [ ] Quote integration works (change orders link to quotes)
- [ ] Project integration works (change orders show on projects)
- [ ] Email service works (approval/confirmation emails)
- [ ] PDF service works (document generation)

**Browser Testing:**
- [ ] Chrome (desktop)
- [ ] Firefox (desktop)
- [ ] Safari (if available)
- [ ] Mobile browsers (iOS/Android)

**Performance:**
- [ ] Page load time < 3 seconds
- [ ] API response time < 1 second
- [ ] PDF generation < 5 seconds
- [ ] Email sending < 10 seconds
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
- [ ] Migration applied successfully (approvalToken field added)
- [ ] Email service initialized
- [ ] PDF service initialized
- [ ] Change order routes registered

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
- [ ] Verify change order API endpoints working
- [ ] Test email delivery
- [ ] Test PDF generation

### First Day
- [ ] Review error logs (morning, afternoon, evening)
- [ ] Check user feedback channels
- [ ] Monitor database performance
- [ ] Verify scheduled jobs/cron tasks
- [ ] Check external integration logs (email service)
- [ ] Monitor change order creation/approval activity
- [ ] Track PDF generation usage

### First Week
- [ ] Daily log reviews
- [ ] Performance metrics analysis
- [ ] User satisfaction check
- [ ] Bug reports triage
- [ ] Plan any hotfixes if needed
- [ ] Track change order adoption rate
- [ ] Monitor approval workflow completion rate
- [ ] Review email delivery success rate

---

## 📈 Success Metrics

### Technical Metrics
- [ ] Deployment completed within estimated time
- [ ] Zero downtime achieved
- [ ] No rollback required
- [ ] Error rate < 1%
- [ ] API response time < 1s (p95)
- [ ] Page load time < 3s
- [ ] Database migration applied successfully
- [ ] PDF generation < 5s average
- [ ] Email delivery > 95% success rate

### Business Metrics
- [ ] All new features functional
- [ ] No critical bugs reported
- [ ] User satisfaction maintained/improved
- [ ] No data loss or corruption
- [ ] Change order features accessible to all users
- [ ] Client approval workflow completes successfully

---

## 📝 Post-Release Tasks

### Immediate (Day 0)
- [ ] Update deployment documentation
- [ ] Mark completed items in bug/feature lists
- [ ] Update CHANGELOG.md with release notes
- [ ] **Publish marketing post** to blog/website
- [ ] **Send release announcement** via email newsletter (if applicable)
- [ ] **Share on social media** (adapt marketing post for platform)
- [ ] **Update support team** with key talking points from marketing post
- [ ] Send release notification (if applicable)
- [ ] Archive deployment logs

### Short-term (Week 1)
- [ ] Review deployment process for improvements
- [ ] Update change order documentation if needed
- [ ] Document any lessons learned
- [ ] Plan next release (Phase 5: Financial Integration)
- [ ] Gather user feedback on change order functionality
- [ ] Monitor approval workflow usage patterns

### Long-term
- [ ] Review metrics after 2 weeks
- [ ] Collect user feedback
- [ ] Update roadmap based on learnings
- [ ] Consider additional change order features based on usage
- [ ] Plan Phase 5 implementation (Invoice/Credit Note generation)

---

## 🚨 Emergency Contacts

**Azure Support:** [Azure Portal](https://portal.azure.com)  
**GitHub Issues:** [Repository Issues](https://github.com/MSFT-Cantro/project-ledger/issues)  
**Database Admin:** [Contact Info]  
**Project Lead:** [Contact Info]

---

## 📚 Related Documentation

- [HOW_TO_RELEASE.md](../../HOW_TO_RELEASE.md) - Release process guide
- [DEPLOYMENT_PLAN.md](../../DEPLOYMENT_PLAN.md) - Technical deployment guide
- [SPEC_change_orders.md](../../../backlog/critical/SPEC_change_orders.md) - Feature specification
- [CHANGE_ORDERS_APPROVAL_WORKFLOW.md](../../../backlog/high/CHANGE_ORDERS_APPROVAL_WORKFLOW.md) - Approval workflow spec

---

## 📌 Technical Details

### Database Changes
- **Migration:** `20251013000002_add_change_order_approval_token`
  - Adds `approvalToken` field to ChangeOrder model
  - Adds `tokenExpiresAt` field for token expiration
  - Required for secure client approval workflow

### New API Endpoints
- `GET /api/change-orders` - List change orders
- `POST /api/change-orders` - Create change order
- `GET /api/change-orders/:id` - Get change order detail
- `PATCH /api/change-orders/:id` - Update change order
- `DELETE /api/change-orders/:id` - Delete change order
- `POST /api/change-orders/:id/send` - Send for approval
- `POST /api/change-orders/:id/approve` - Client approval (public with token)
- `POST /api/change-orders/:id/decline` - Client decline (public with token)
- `GET /api/change-orders/:id/pdf` - Generate PDF
- `POST /api/change-orders/:id/execute` - Execute approved change order
- `GET /api/reports/change-orders` - Generate change order report

### New Backend Services
- `ChangeOrderService.ts` - Complete business logic for change orders
- `approvalToken.ts` - Token generation and validation
- `emailService.ts` - Email sending for approvals and confirmations
- `pdfService.ts` - PDF generation for change orders

### New Frontend Components
- `ChangeOrdersPage.tsx` - List page with filtering and search
- `ChangeOrderDetailPage.tsx` - Detail view with status and actions
- `CreateChangeOrderPage.tsx` - Creation form with item management
- `ChangeOrderApprovalPage.tsx` - Public approval interface
- `ChangeOrderCreatePage.tsx` - Placeholder (redirects to Create)
- `ChangeOrderEditPage.tsx` - Placeholder (edit not yet implemented)

### New Frontend Routes
- `/change-orders` - List page
- `/change-orders/new` - Create new change order
- `/change-orders/:id` - Detail page
- `/change-orders/:id/edit` - Edit page (placeholder)
- `/change-orders/:id/approve/:token` - Public approval page
- `/reports/change-orders` - Change orders report

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
- [ ] Manual testing completed (all features)
- [ ] Logs show no errors
- [ ] Performance metrics acceptable
- [ ] Change order features working
- [ ] Approval workflow functional
- [ ] PDF generation working
- [ ] Email notifications sending

**Post-Deployment:**
- [ ] Monitoring in place (first hour)
- [ ] Documentation updated
- [ ] Team notified
- [ ] Success metrics tracked
- [ ] Marketing post published

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
