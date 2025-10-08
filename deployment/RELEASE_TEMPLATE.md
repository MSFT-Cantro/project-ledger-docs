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
- [ ] **Marketing post created** (see Marketing Content section below)

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

### Database (CRITICAL - October 8, 2025 Data Loss Prevention)
- [ ] 🚨 **MANDATORY: Production database backup created** (`./tools/backup/backup-database.sh deployment`)
- [ ] 🚨 **MANDATORY: Database state verified** (`DATABASE_URL=... node scripts/verify-production-data.js`)
- [ ] 🚨 **MANDATORY: Migration status checked** (not "out of sync" or empty)
- [ ] Database migrations created (if needed)
- [ ] Migration scripts tested locally
- [ ] Rollback migration scripts prepared (if applicable)
- [ ] 🚨 **MANDATORY: Recovery plan documented** (location of backup, restore commands)

---

## � Marketing Content Creation

### Create Marketing Post (MANDATORY)

Before deployment, create a user-facing marketing post to announce the release.

#### **Location:**
`docs/deployment/releases/vX.Y.Z/MARKETING_POST_vX.Y.Z.md`

#### **Template Structure:**

```markdown
# 🚀 ProjectLedger vX.Y.Z — [Catchy Headline]

**Release Date:** [Date]
**Version:** X.Y.Z

## ✨ Overview
[2-3 paragraphs describing the release theme and main value proposition]

## 💡 What's New

### 🎯 **[Feature 1 Name]**
**[Benefit-focused subtitle]**

[Description of feature from user perspective]

**Why you'll love it:**
- [Benefit 1]
- [Benefit 2]
- [Benefit 3]

**How it works:**
- [Usage explanation]

### [Repeat for each major feature]

## ⚙️ Behind the Scenes
[Technical improvements without jargon]

## 🧭 What to Expect
[Reassurance about backward compatibility and user experience]

## ✅ Summary
[Bullet-point list of key improvements]

## 🎯 Perfect For:
- [Target audience 1]
- [Target audience 2]

## 🚀 Available Now
[Call to action]
```

#### **Writing Guidelines:**

1. **Focus on Benefits, Not Features**
   - ❌ "We added a new database table for organizations"
   - ✅ "Manage multiple businesses from one account"

2. **Use Clear, Non-Technical Language**
   - Write for business owners, not developers
   - Explain "what" and "why", not "how"
   - Avoid technical jargon (API, database, migration, etc.)

3. **Emphasize User Value**
   - How does this save time?
   - How does this reduce friction?
   - How does this help them grow?

4. **Include Reassurance**
   - Backward compatibility
   - Data safety
   - No disruption to existing workflows

5. **Match Brand Tone**
   - Professional but approachable
   - Confident but not boastful
   - Helpful and supportive

#### **Distribution Checklist:**
- [ ] Marketing post created and reviewed
- [ ] Post added to release documentation folder
- [ ] Ready for blog/website publication
- [ ] Ready for email newsletter
- [ ] Ready for social media adaptation
- [ ] Key talking points identified for support team

#### **Example Reference:**
See `docs/deployment/releases/v1.3.0/MARKETING_POST_v1.3.0.md` for a complete example.

---

## �🚀 Deployment Process

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

#### 1.3 🚨 CRITICAL: Database Safety Checks
```bash
# Step 1: Create database backup (MANDATORY)
./tools/backup/backup-database.sh deployment "Pre-deployment backup for vX.Y.Z"

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
- [ ] **Publish marketing post** to blog/website
- [ ] **Send release announcement** via email newsletter (if applicable)
- [ ] **Share on social media** (adapt marketing post for platform)
- [ ] **Update support team** with key talking points from marketing post
- [ ] Send release notification (if applicable)
- [ ] Archive deployment logs
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
