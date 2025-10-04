# 🚀 Production Release Plan - Beamer Integration & UserMenu Updates

**Release Version:** v1.1.0  
**Release Date:** October 3, 2025  
**Environment:** Azure Production (app.projectledger.ca)  
**Release Manager:** [Your Name]

---

## 📋 Release Overview

### **What's Being Deployed**

This release includes two major features completed from the bug tracker:

✅ **Item #20: Beamer Notification System Integration**
- Integrated Beamer notification platform (Product ID: SbTnvhZr79497)
- Monthly Active User (MAU) tracking via user email
- Dynamic theme synchronization (light/dark mode)
- Custom notification buttons on desktop and mobile

✅ **Item #21: UserMenu Restructuring**
- Desktop menu: Removed Profile/Notifications, renamed to "Quick Settings"
- Mobile navigation: Removed Profile/Notifications, kept "Account Settings"
- Streamlined user experience

### **Technical Changes**
- 7 files modified
- 1 new file created (window.d.ts)
- +141 lines of code
- -22 lines of code
- Bundle size: +143 bytes

---

## 🎯 Pre-Release Checklist

### **1. Code Verification**
- [x] All changes committed to GitHub (commit: 913d22e)
- [x] Changes pushed to main branch
- [x] Build successful locally (371.47 kB)
- [ ] All tests passing
- [ ] No breaking changes identified

### **2. Environment Preparation**
- [ ] Azure CLI installed and logged in
- [ ] Docker Desktop running
- [ ] Access to Azure Container Registry confirmed
- [ ] Database backup completed
- [ ] Current production version documented

### **3. Configuration Verification**
- [ ] Environment variables reviewed
- [ ] Beamer Product ID confirmed (SbTnvhZr79497)
- [ ] No new environment variables needed
- [ ] No database migrations required

### **4. Communication**
- [ ] Stakeholders notified of deployment window
- [ ] Team members available for support
- [ ] Rollback plan communicated
- [ ] Monitoring dashboard open

---

## 🔄 Deployment Steps

### **Phase 1: Pre-Deployment (5 minutes)**

#### 1.1 Backup Current State
```bash
# Document current revision
az containerapp revision list \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --query "[?properties.active].name" -o tsv > current-frontend-revision.txt

az containerapp revision list \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --query "[?properties.active].name" -o tsv > current-backend-revision.txt

# Backup database (optional but recommended)
kubectl exec -it projectledger-db -- \
  pg_dump -U postgres projectledger > \
  backup-$(date +%Y%m%d-%H%M%S).sql
```

#### 1.2 Tag Release in Git
```bash
cd c:/Code/ProjectLedger2

# Create release tag
git tag -a v1.1.0 -m "Release v1.1.0: Beamer integration + UserMenu updates"
git push origin v1.1.0
```

#### 1.3 Verify Application Health
```bash
# Check frontend
curl -I https://app.projectledger.ca

# Check backend health endpoint
curl https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io/api/health

# Check database connectivity
az container logs \
  --resource-group projectledger-poc \
  --name projectledger-db \
  --tail 20
```

---

### **Phase 2: Build & Push Images (10-15 minutes)**

#### 2.1 Login to Azure Container Registry
```bash
az acr login --name projectledgerregistry
```

#### 2.2 Build Backend Image
```bash
cd c:/Code/ProjectLedger2

# Build production backend
docker build \
  -f apps/backend/Dockerfile \
  -t projectledgerregistry.azurecr.io/projectledger-backend:latest \
  -t projectledgerregistry.azurecr.io/projectledger-backend:v1.1.0 \
  --target production \
  .

# Verify image built successfully
docker images | grep projectledger-backend
```

#### 2.3 Build Frontend Image
```bash
# Build production frontend with Beamer integration
docker build \
  -f apps/frontend/Dockerfile \
  -t projectledgerregistry.azurecr.io/projectledger-frontend:latest \
  -t projectledgerregistry.azurecr.io/projectledger-frontend:v1.1.0 \
  --target azure \
  .

# Verify image built successfully
docker images | grep projectledger-frontend
```

#### 2.4 Push Images to Registry
```bash
# Push backend
docker push projectledgerregistry.azurecr.io/projectledger-backend:latest
docker push projectledgerregistry.azurecr.io/projectledger-backend:v1.1.0

# Push frontend
docker push projectledgerregistry.azurecr.io/projectledger-frontend:latest
docker push projectledgerregistry.azurecr.io/projectledger-frontend:v1.1.0
```

---

### **Phase 3: Deploy to Azure (5-10 minutes)**

#### 3.1 Deploy Backend (No Changes in This Release)
```bash
# Backend deployment (mainly for consistency, no code changes)
az containerapp update \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --image projectledgerregistry.azurecr.io/projectledger-backend:v1.1.0 \
  --revision-suffix v1-1-0

# Wait for backend to be ready
echo "Waiting for backend deployment..."
sleep 30
```

#### 3.2 Deploy Frontend (Primary Changes)
```bash
# Frontend deployment with Beamer integration
az containerapp update \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --image projectledgerregistry.azurecr.io/projectledger-frontend:v1.1.0 \
  --revision-suffix v1-1-0

# Wait for frontend to be ready
echo "Waiting for frontend deployment..."
sleep 30
```

---

### **Phase 4: Verification & Testing (10 minutes)**

#### 4.1 Check Deployment Status
```bash
# Check frontend revision status
az containerapp revision list \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  -o table

# Check backend revision status
az containerapp revision list \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  -o table
```

#### 4.2 Health Checks
```bash
# Frontend health check
curl -I https://app.projectledger.ca
# Expected: HTTP/1.1 200 OK

# Backend health check
curl https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io/api/health
# Expected: {"status":"healthy"}

# Check if Beamer script is loaded
curl -s https://app.projectledger.ca | grep -o "getbeamer.com"
# Expected: getbeamer.com
```

#### 4.3 Functional Testing
**Manual Testing Checklist:**

- [ ] **Frontend loads successfully**
  - Visit: https://app.projectledger.ca
  - Verify: Page loads without errors

- [ ] **User can login**
  - Test: Login with existing account
  - Verify: Authentication successful

- [ ] **Beamer notification button appears**
  - Desktop: Check top toolbar for notification bell icon
  - Mobile: Check mobile toolbar for notification bell icon
  - Verify: Button visible and styled correctly

- [ ] **Beamer panel opens/closes**
  - Click: Notification bell icon
  - Verify: Beamer panel slides in from right
  - Click again: Panel closes

- [ ] **Beamer theme matches app theme**
  - Toggle: Switch between light/dark mode
  - Verify: Beamer panel colors update automatically
  - Light mode: Blue top bar (#2563eb)
  - Dark mode: Dark slate top bar (#1e293b)

- [ ] **UserMenu updated correctly**
  - Desktop: Open user dropdown
  - Verify: Shows "Quick Settings" (not "Account Settings")
  - Verify: No "Profile" or "Notifications" options

- [ ] **Mobile navigation updated**
  - Mobile: Open side drawer
  - Verify: Shows "Account Settings" (not "Quick Settings")
  - Verify: No "Profile" or "Notifications" options

- [ ] **MAU tracking works**
  - Login: Check browser localStorage
  - Verify: `userEmail` key contains user's email
  - Check: Beamer dashboard for new user activity (later)

- [ ] **All existing features work**
  - Test: Dashboard, Clients, Projects, Quotes, Invoices
  - Verify: No regressions in existing functionality

#### 4.4 Check Application Logs
```bash
# Frontend logs
az containerapp logs show \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --tail 50 \
  --follow

# Backend logs
az containerapp logs show \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --tail 50 \
  --follow

# Look for any errors or warnings
```

#### 4.5 Performance Check
```bash
# Check response times
time curl -s -o /dev/null -w "Total: %{time_total}s\n" https://app.projectledger.ca

# Check bundle size (should be ~371 kB gzipped)
curl -s -H "Accept-Encoding: gzip" https://app.projectledger.ca/static/js/main.*.js \
  | wc -c
```

---

### **Phase 5: Traffic Management (Optional)**

If you want gradual rollout (recommended for first-time deployment):

#### 5.1 Split Traffic (Optional)
```bash
# Route 10% traffic to new revision for testing
az containerapp revision set-mode \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --mode multiple

az containerapp ingress traffic set \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --revision-weight projectledger-frontend--v1-1-0=10 \
  --revision-weight projectledger-frontend--<previous-revision>=90

# Monitor for 10-15 minutes, check for errors

# If all good, increase to 50%
az containerapp ingress traffic set \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --revision-weight projectledger-frontend--v1-1-0=50 \
  --revision-weight projectledger-frontend--<previous-revision>=50

# Monitor for 10-15 minutes, check for errors

# If all good, route 100% to new revision
az containerapp ingress traffic set \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --revision-weight projectledger-frontend--v1-1-0=100
```

#### 5.2 Switch to Single-Revision Mode
```bash
# Once stable, deactivate old revision
az containerapp revision deactivate \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --revision <previous-revision>
```

---

## 🔙 Rollback Procedure

### **If Issues Are Detected**

#### Quick Rollback (Instant)
```bash
# List all revisions
az containerapp revision list \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  -o table

# Activate previous revision
az containerapp revision activate \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --revision <previous-revision-name>

# Route 100% traffic back to previous version
az containerapp ingress traffic set \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --revision-weight <previous-revision-name>=100
```

#### Verify Rollback
```bash
# Check active revision
az containerapp revision list \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --query "[?properties.active].name" -o tsv

# Test application
curl -I https://app.projectledger.ca
```

---

## 📊 Post-Deployment Monitoring

### **Immediate Monitoring (First Hour)**

#### Check Metrics
```bash
# View container app metrics in Azure Portal
az monitor metrics list \
  --resource /subscriptions/<subscription-id>/resourceGroups/projectledger-poc/providers/Microsoft.App/containerApps/projectledger-frontend \
  --metric "Requests" \
  --interval PT1M

# Monitor error rates
az monitor metrics list \
  --resource /subscriptions/<subscription-id>/resourceGroups/projectledger-poc/providers/Microsoft.App/containerApps/projectledger-frontend \
  --metric "RequestsWithError" \
  --interval PT1M
```

#### Check Beamer Dashboard
- [ ] Login to Beamer dashboard: https://app.getbeamer.com
- [ ] Verify MAU tracking is working
- [ ] Check for user activities
- [ ] Verify theme settings applied correctly

### **24-Hour Monitoring**

- [ ] **User Activity**: Monitor user login/signup rates
- [ ] **Error Rates**: Check for increased errors
- [ ] **Performance**: Monitor response times
- [ ] **Beamer Engagement**: Track notification button clicks
- [ ] **User Feedback**: Monitor for user-reported issues

### **Week 1 Monitoring**

- [ ] **MAU Growth**: Track Monthly Active Users in Beamer
- [ ] **Feature Adoption**: Monitor notification engagement
- [ ] **Performance Trends**: Ensure no degradation
- [ ] **User Satisfaction**: Collect feedback on new features

---

## 📈 Success Criteria

### **Deployment Success**
- [x] All containers deployed without errors
- [x] Application accessible at app.projectledger.ca
- [x] No increase in error rates
- [x] Response times within acceptable range (<2s)
- [x] All functional tests passed

### **Feature Success**
- [ ] Beamer notification button visible on all pages
- [ ] Notification panel opens/closes correctly
- [ ] Theme synchronization working (light/dark)
- [ ] MAU tracking active in Beamer dashboard
- [ ] UserMenu updated per specifications
- [ ] No user complaints about missing features

---

## 🚨 Known Issues & Mitigation

### **Potential Issues**

1. **Beamer Script Loading**
   - **Risk**: External script may fail to load
   - **Mitigation**: Added error handling and console logging
   - **Fallback**: App works without Beamer if script fails

2. **Theme Detection**
   - **Risk**: localStorage may not be available
   - **Mitigation**: Defaults to 'light' theme if unavailable
   - **Impact**: Minimal - Beamer still works with default theme

3. **MAU Tracking**
   - **Risk**: Email may not be stored in some edge cases
   - **Mitigation**: Defaults to 'anonymous' if email unavailable
   - **Impact**: User still sees notifications, just not tracked

4. **UserMenu Changes**
   - **Risk**: Users may look for removed menu items
   - **Mitigation**: Beamer notification button prominent
   - **Impact**: Low - Quick Settings still accessible

---

## 📞 Support Plan

### **Incident Response**

#### Severity 1 (Critical - App Down)
- **Response Time**: Immediate
- **Action**: Execute rollback procedure
- **Escalation**: All hands on deck

#### Severity 2 (Major Feature Broken)
- **Response Time**: 15 minutes
- **Action**: Assess impact, decide rollback vs. hotfix
- **Escalation**: Development team lead

#### Severity 3 (Minor Issue)
- **Response Time**: 1 hour
- **Action**: Log issue, schedule fix in next release
- **Escalation**: None

### **Contact Information**
- **DevOps Lead**: [Your Name/Email]
- **Backend Lead**: [Name/Email]
- **Frontend Lead**: [Name/Email]
- **On-Call**: [Phone/Slack]

---

## 📝 Deployment Timeline

### **Estimated Timeline**

| Phase | Duration | Description |
|-------|----------|-------------|
| Pre-Deployment | 5 min | Backup, tag release, verify health |
| Build & Push | 10-15 min | Build Docker images, push to registry |
| Deploy | 5-10 min | Update container apps |
| Verification | 10 min | Health checks, functional testing |
| **Total** | **30-40 min** | Complete deployment window |

### **Recommended Deployment Window**

- **Best Time**: Outside business hours (evening or early morning)
- **Suggested**: 7:00 PM - 8:00 PM EST (low traffic period)
- **Backup Window**: 6:00 AM - 7:00 AM EST (before business hours)

---

## ✅ Post-Deployment Tasks

### **Immediate (Day 1)**
- [ ] Update deployment log with version and timestamp
- [ ] Notify team of successful deployment
- [ ] Monitor application for first 2 hours
- [ ] Document any issues encountered
- [ ] Update release notes documentation

### **Week 1**
- [ ] Review Beamer analytics dashboard
- [ ] Collect user feedback on new features
- [ ] Monitor error logs for patterns
- [ ] Schedule retrospective meeting
- [ ] Plan next release features

### **Documentation Updates**
- [ ] Update AZURE_DEPLOYMENT_COMPLETE.md with new version
- [ ] Add release notes to CHANGELOG.md
- [ ] Update README.md if needed
- [ ] Archive this deployment plan

---

## 📚 Related Documentation

- [DEPLOYMENT_PLAN.md](./DEPLOYMENT_PLAN.md) - General deployment strategy
- [AZURE_DEPLOYMENT_COMPLETE.md](./AZURE_DEPLOYMENT_COMPLETE.md) - Infrastructure details
- [ROLLBACK.md](./ROLLBACK.md) - Detailed rollback procedures
- [MONITORING_ALERTING_PAYPAL.md](./MONITORING_ALERTING_PAYPAL.md) - Monitoring setup

---

## 🎉 Release Notes (v1.1.0)

### **New Features**

#### Beamer Notification System
- Integrated Beamer notification platform for product updates and announcements
- Monthly Active User (MAU) tracking via user email
- Dynamic theme synchronization (light/dark mode)
- Custom notification buttons on desktop and mobile interfaces
- Product ID: SbTnvhZr79497

#### UserMenu Improvements
- Streamlined desktop menu: Removed Profile/Notifications, renamed to "Quick Settings"
- Updated mobile navigation: Removed Profile/Notifications
- Improved user experience with cleaner navigation

### **Technical Improvements**
- Added TypeScript definitions for Beamer API
- Enhanced theme context with real-time Beamer updates
- Improved auth context with MAU tracking on login/logout
- Added initialization checks and error handling

### **Bundle Size**
- Main bundle: 371.47 kB (gzipped)
- Change: +143 bytes (negligible impact)

---

**Deployment Plan Version:** 1.0  
**Last Updated:** October 3, 2025  
**Status:** Ready for Execution
