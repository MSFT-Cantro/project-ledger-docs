# Production Release v1.3.0

> **Multi-Organization Support + Invoice Enhancement + Documentation Cleanup**  
> **Release Date:** October 8, 2025  
> **Deployer:** GitHub Copilot Assistant  
> **Revision Suffix:** `v1-3-0`

---

## Release Information

**Version:** `v1.3.0`  
**Release Date:** `October 8, 2025`  
**Deployer:** `GitHub Copilot Assistant`  
**Revision Suffix:** `v1-3-0`

**🎯 Major Changes in this release:**
- ✅ **NEW MAJOR FEATURE**: Multi-Organization Support - Users can now belong to multiple organizations and switch between them
- ✅ Enhanced Invoice Creation - Quote item selector for selective invoice creation (Bug #30)
- ✅ Improved User Experience - Removed confusing Beamer default button (Bug #31) 
- ✅ Documentation Cleanup - Organized root directory and comprehensive specification documents
- ✅ Organization Selector Implementation - Complete with database migration, API endpoints, and UI components

---

## 📋 Pre-Release Checklist

### Code & Documentation
- [x] All features tested locally
- [x] All tests passing (`npm test` in backend and frontend)
- [x] Code reviewed and approved
- [x] Changelog updated with release notes
- [x] Documentation updated (comprehensive spec docs added)
- [x] Bug list updated (mark completed items #30, #31)
- [x] No TypeScript errors
- [x] No console errors in browser

### Git & Version Control
- [x] All changes committed to `main` branch
- [x] Git status is clean (no uncommitted changes)
- [x] All commits pushed to GitHub
- [x] Version numbers updated in `package.json` files (if applicable)

### Environment & Access
- [ ] Logged into Azure CLI (`az login`)
- [ ] Docker Desktop running
- [ ] Access to Azure Container Registry verified
- [ ] Environment variables verified (no changes needed, or changes documented)

### Database
- [x] Database migrations created (UserAccount table and migration)
- [x] Migration scripts tested locally
- [x] Rollback migration scripts prepared (if applicable)
- [ ] Database backup confirmed recent

---

## 🚀 Deployment Process

### Phase 1: Pre-Deployment (5 minutes)

#### 1.1 Final Verification
```bash
# Navigate to project
cd /path/to/ProjectLedger2

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
git tag v1.3.0
git push origin v1.3.0
```

#### 1.3 Backup Current Revisions
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
  -t projectledgerregistry.azurecr.io/projectledger-backend:v1.3.0 \
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
  -t projectledgerregistry.azurecr.io/projectledger-frontend:v1.3.0 \
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
docker push projectledgerregistry.azurecr.io/projectledger-backend:v1.3.0

# Push frontend images
docker push projectledgerregistry.azurecr.io/projectledger-frontend:latest
docker push projectledgerregistry.azurecr.io/projectledger-frontend:v1.3.0
```

---

### Phase 4: Deploy to Azure (5-10 minutes)

#### 4.1 Deploy Backend
```bash
az containerapp update \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --image projectledgerregistry.azurecr.io/projectledger-backend:v1.3.0 \
  --revision-suffix v1-3-0
```

**Wait 15 seconds for stabilization**

#### 4.2 Deploy Frontend
```bash
az containerapp update \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --image projectledgerregistry.azurecr.io/projectledger-frontend:v1.3.0 \
  --revision-suffix v1-3-0
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

**New Features (v1.3.0):**
- [ ] **Multi-Organization Support**: Users with multiple orgs see organization selector after login
- [ ] **Organization Switching**: Organization switcher dropdown appears in navigation for multi-org users
- [ ] **Single-Org Auto-Select**: Users with single organization automatically selected and go to dashboard
- [ ] **Invoice Quote Selector**: When creating invoice from quote, popup allows selecting specific line items
- [ ] **Clean Interface**: Beamer button no longer appears in bottom-right corner (only toolbar button)
- [ ] **UserAccount Database**: Migration completed, UserAccount table functioning properly

**Data Operations:**
- [ ] Can create new records
- [ ] Can read existing records
- [ ] Can update records
- [ ] Can delete records
- [ ] Search/filter functionality works

**Multi-Organization Specific Tests:**
- [ ] Test users can select from multiple organizations
- [ ] Organization switching works correctly
- [ ] Current organization displays in navigation
- [ ] Data isolation working (no cross-org data leaks)
- [ ] API endpoints return correct organization-specific data

**Integration Points:**
- [ ] API calls succeed
- [ ] Authentication tokens include correct accountId
- [ ] External integrations functional (PayPal, Beamer toolbar button, etc.)

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
- [ ] Organization switch audit logs (marked with [AUDIT])

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
# Rollback backend (use previous revision name from backup)
az containerapp revision activate \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --revision [previous-backend-revision-name]

# Rollback frontend (use previous revision name from backup)
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
- [ ] Watch for organization switching activity in audit logs

### First Day
- [ ] Review error logs (morning, afternoon, evening)
- [ ] Check user feedback channels
- [ ] Monitor database performance
- [ ] Verify scheduled jobs/cron tasks
- [ ] Check external integration logs
- [ ] Monitor organization selector usage

### First Week
- [ ] Daily log reviews
- [ ] Performance metrics analysis
- [ ] User satisfaction check
- [ ] Bug reports triage
- [ ] Plan any hotfixes if needed
- [ ] Analyze multi-org feature adoption

---

## 📈 Success Metrics

### Technical Metrics
- [ ] Deployment completed within estimated time
- [ ] Zero downtime achieved
- [ ] No rollback required
- [ ] Error rate < 1%
- [ ] API response time < 1s (p95)
- [ ] Page load time < 3s

### Business Metrics
- [ ] All new features functional
- [ ] Multi-organization support working correctly
- [ ] Invoice enhancement improving user workflow
- [ ] No critical bugs reported
- [ ] User satisfaction maintained/improved
- [ ] No data loss or corruption

---

## 🎯 Key Features Delivered

### 1. Multi-Organization Support (Major Feature)
**Complete implementation includes:**
- **Database Layer**: UserAccount junction table with many-to-many User ↔ Account relationship
- **Backend API**: 
  - `GET /api/user/organizations` - List user's organizations
  - `POST /api/user/select-organization` - Select active organization
  - `GET /api/user/current-organization` - Get current organization
  - Enhanced login endpoint with multi-org detection
- **Frontend UI**: 
  - OrganizationSelectorPage - Full-page selector for multi-org users
  - OrganizationSwitcher - Navigation dropdown for quick switching
  - OrganizationContext - State management for organization data
- **User Experience**:
  - Single org users: Auto-selected → Dashboard (no change in UX)
  - Multi org users: Login → Organization selector → Choose → Dashboard
  - Organization switching via navigation dropdown
- **Security**: Cross-organization access prevention, audit logging, JWT with organization context

### 2. Enhanced Invoice Creation (Bug #30)
- **Quote Item Selector**: When creating invoice from quote, popup allows selecting specific line items to include
- **Improved Workflow**: Reduces manual work when creating partial invoices from quotes
- **Better UX**: Clear interface for choosing which quote items to invoice

### 3. UI Improvements (Bug #31)
- **Beamer Button Removal**: Removed confusing default Beamer button from bottom-right corner
- **Cleaner Interface**: Notification access only through toolbar button (consistent UX)

### 4. Documentation & Code Organization
- **Comprehensive Specifications**: Added detailed spec documents for all major features
- **Clean Root Directory**: Organized project structure and removed security vulnerabilities
- **Complete Implementation Docs**: Multi-org feature fully documented with implementation guide

---

## 📝 Post-Release Tasks

### Immediate (Day 0)
- [ ] Update deployment documentation
- [ ] Mark completed items in bug/feature lists (Bug #30, #31)
- [ ] Update CHANGELOG.md with release notes
- [ ] Send release notification (if applicable)
- [ ] Archive deployment logs
- [ ] Update releases/README.md with v1.3.0 entry

### Short-term (Week 1)
- [ ] Review deployment process for improvements
- [ ] Update this template if needed
- [ ] Document any lessons learned
- [ ] Plan next release (if applicable)
- [ ] Monitor multi-org feature usage and user feedback

### Long-term
- [ ] Review metrics after 2 weeks
- [ ] Collect user feedback on multi-org experience
- [ ] Update roadmap based on learnings
- [ ] Consider next phase multi-org enhancements (user invitations to orgs)

---

## 🚨 Emergency Contacts

**Azure Support:** [Azure Portal](https://portal.azure.com)  
**GitHub Issues:** [Repository Issues](https://github.com/MSFT-Cantro/project-ledger/issues)  
**Database Admin:** [Contact Info]  
**Project Lead:** [Contact Info]

---

## 📚 Related Documentation

- [Multi-Org Implementation Complete](../../MULTI_ORG_IMPLEMENTATION_COMPLETE.md) - Complete implementation guide
- [Organization Selector Spec](../../Completed/SPEC_organization-selector.md) - Full specification (moved to completed)
- [Azure Deployment Guide](../AZURE_DEPLOYMENT_COMPLETE.md) - Infrastructure setup
- [Database Seeding](../DATABASE_SEEDING.md) - Database management
- [Monitoring & Alerting](../MONITORING_ALERTING_PAYPAL.md) - Monitoring setup

---

## 📌 Notes & Lessons Learned

**What went well:**
- Multi-org implementation was comprehensive and well-documented
- Zero breaking changes for single-org users
- Clean database migration with zero data loss
- Good separation of phases (database → backend → frontend)

**What could be improved:**
- [Add after deployment]

**Action items:**
- [Add after deployment]

---

**Deployment Status:** ⬜ Not Started | ⏳ In Progress | ✅ Completed | ❌ Failed

**Deployed By:** ________________  
**Date & Time:** ________________  
**Sign-off:** ________________