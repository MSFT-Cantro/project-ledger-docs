# Production Release Template

> **Template Version:** 1.0  
> **Last Updated:** October 6, 2025  
> **Purpose:** Standard template for all production releases

---

## Release Information

**Version:** `vX.Y.Z`  
**Release Date:** `[Date]`  
**Deployer:** `[Your Name]`  
**Revision Suffix:** `vX-Y-Z`

---

## 📋 Pre-Release Checklist

### Code & Documentation
- [ ] All features tested locally
- [ ] All tests passing (`npm test` in backend and frontend)
- [ ] Code reviewed and approved
- [ ] Changelog updated with release notes
- [ ] Documentation updated (if applicable)
- [ ] Bug list updated (mark completed items)
- [ ] No TypeScript errors
- [ ] No console errors in browser

### Git & Version Control
- [ ] All changes committed to `main` branch
- [ ] Git status is clean (no uncommitted changes)
- [ ] All commits pushed to GitHub
- [ ] Version numbers updated in `package.json` files (if applicable)

### Environment & Access
- [ ] Logged into Azure CLI (`az login`)
- [ ] Docker Desktop running
- [ ] Access to Azure Container Registry verified
- [ ] Environment variables verified (no changes needed, or changes documented)

### Database
- [ ] Database migrations created (if needed)
- [ ] Migration scripts tested locally
- [ ] Rollback migration scripts prepared (if applicable)
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
git tag vX.Y.Z
git push origin vX.Y.Z
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
  -t projectledgerregistry.azurecr.io/projectledger-backend:vX.Y.Z \
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
  -t projectledgerregistry.azurecr.io/projectledger-frontend:vX.Y.Z \
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
docker push projectledgerregistry.azurecr.io/projectledger-backend:vX.Y.Z

# Push frontend images
docker push projectledgerregistry.azurecr.io/projectledger-frontend:latest
docker push projectledgerregistry.azurecr.io/projectledger-frontend:vX.Y.Z
```

---

### Phase 4: Deploy to Azure (5-10 minutes)

#### 4.1 Deploy Backend
```bash
az containerapp update \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --image projectledgerregistry.azurecr.io/projectledger-backend:vX.Y.Z \
  --revision-suffix vX-Y-Z
```

**Wait 15 seconds for stabilization**

#### 4.2 Deploy Frontend
```bash
az containerapp update \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --image projectledgerregistry.azurecr.io/projectledger-frontend:vX.Y.Z \
  --revision-suffix vX-Y-Z
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

**New Features (if applicable):**
- [ ] [List specific features being released]
- [ ] [Add tests for each new feature]

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

### First Day
- [ ] Review error logs (morning, afternoon, evening)
- [ ] Check user feedback channels
- [ ] Monitor database performance
- [ ] Verify scheduled jobs/cron tasks
- [ ] Check external integration logs

### First Week
- [ ] Daily log reviews
- [ ] Performance metrics analysis
- [ ] User satisfaction check
- [ ] Bug reports triage
- [ ] Plan any hotfixes if needed

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
- [ ] No critical bugs reported
- [ ] User satisfaction maintained/improved
- [ ] No data loss or corruption

---

## 📝 Post-Release Tasks

### Immediate (Day 0)
- [ ] Update deployment documentation
- [ ] Mark completed items in bug/feature lists
- [ ] Update CHANGELOG.md with release notes
- [ ] Send release notification (if applicable)
- [ ] Archive deployment logs

### Short-term (Week 1)
- [ ] Review deployment process for improvements
- [ ] Update this template if needed
- [ ] Document any lessons learned
- [ ] Plan next release (if applicable)

### Long-term
- [ ] Review metrics after 2 weeks
- [ ] Collect user feedback
- [ ] Update roadmap based on learnings

---

## 🚨 Emergency Contacts

**Azure Support:** [Azure Portal](https://portal.azure.com)  
**GitHub Issues:** [Repository Issues](https://github.com/MSFT-Cantro/project-ledger/issues)  
**Database Admin:** [Contact Info]  
**Project Lead:** [Contact Info]

---

## 📚 Related Documentation

- [Production Release Plan](./PRODUCTION_RELEASE_PLAN.md) - Detailed procedures
- [Azure Deployment Guide](./AZURE_DEPLOYMENT_COMPLETE.md) - Infrastructure setup
- [Database Seeding](./DATABASE_SEEDING.md) - Database management
- [Monitoring & Alerting](./MONITORING_ALERTING_PAYPAL.md) - Monitoring setup

---

## 🔧 Automated Deployment Script

For automated deployments, use the script template:

```bash
#!/bin/bash
# File: tools/deployment/release-vX.Y.Z.sh

# Copy from tools/deployment/release-v1.1.0.sh as template
# Update version numbers and release-specific details
```

**Script should include:**
- Safety checks (Docker, Azure login, git status)
- Confirmation prompts
- Automatic revision backup
- Image building and pushing
- Container app updates
- Health verification
- Rollback instructions in output

---

## 📋 Release Checklist Summary

**Pre-Deployment:**
- [ ] All pre-release checklist items completed
- [ ] Git tag created and pushed
- [ ] Current revisions backed up

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
