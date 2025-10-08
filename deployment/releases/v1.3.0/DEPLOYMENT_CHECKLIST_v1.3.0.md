# Production Deployment Checklist - v1.3.0

> **Version:** 1.3.0  
> **Date:** October 8, 2025  
> **Deployer:** GitHub Copilot Assistant  
> **Start Time:** [Time]

---

## ☑️ Pre-Deployment

- [ ] All changes committed and pushed to GitHub
- [ ] Git status is clean (no uncommitted files)
- [ ] All tests passing locally
- [ ] Docker Desktop running
- [ ] Logged into Azure CLI (`az login`)
- [ ] ACR access verified
- [ ] Version numbers updated (if applicable)

---

## 🏗️ Build & Push

- [ ] Backend Docker image built
- [ ] Frontend Docker image built
- [ ] Backend images pushed to ACR (latest + v1.3.0)
- [ ] Frontend images pushed to ACR (latest + v1.3.0)
- [ ] Git tag v1.3.0 created and pushed

---

## 🚀 Deployment

- [ ] Current revisions backed up
- [ ] Backend deployed with revision v1-3-0
- [ ] Frontend deployed with revision v1-3-0
- [ ] 15-second stabilization wait after each deployment
- [ ] Deployment completed without errors

---

## ✅ Verification & Testing

### Health Checks
- [ ] Frontend health check (200 OK)
- [ ] Backend health check (200 OK, DB connected)
- [ ] No 404 or 500 errors

### Core Functionality
- [ ] Login/logout works
- [ ] Dashboard loads
- [ ] Navigation functional
- [ ] Mobile view works
- [ ] Create operations work
- [ ] Read operations work
- [ ] Update operations work
- [ ] Delete operations work

### New Features (v1.3.0)
- [ ] **Multi-Organization Support**: Users with multiple orgs see organization selector after login
- [ ] **Organization Auto-Selection**: Single-org users automatically selected and go to dashboard (no UX change)
- [ ] **Organization Switcher**: Dropdown appears in navigation for multi-org users
- [ ] **Current Organization Display**: Shows active organization name and role in navigation
- [ ] **Organization Data Isolation**: No cross-organization data leaks (test with different orgs)
- [ ] **Quote Item Selector**: When creating invoice from quote, popup allows selecting specific line items
- [ ] **Beamer Button Removal**: Default Beamer button no longer appears in bottom-right corner
- [ ] **Clean Navigation**: Only toolbar notification button remains for Beamer access

### Multi-Organization Specific Tests
- [ ] **Test User: demo@projectledger.com** (2 orgs) - Shows organization selector
- [ ] **Test User: lovell@microsoft.com** (2 orgs) - Shows organization selector with different roles
- [ ] **Test User: warren@microsoft.com** (1 org) - Auto-selected, no selector shown
- [ ] Organization switching works from navigation dropdown
- [ ] JWT tokens contain correct accountId after organization selection
- [ ] API calls return organization-specific data only
- [ ] Organization switcher hidden for single-org users

### Integration Points
- [ ] API calls successful
- [ ] Authentication working (JWT includes accountId)
- [ ] External integrations functional
- [ ] PayPal integration still works
- [ ] Beamer toolbar button works
- [ ] Organization API endpoints functional:
  - [ ] GET /api/user/organizations
  - [ ] POST /api/user/select-organization  
  - [ ] GET /api/user/current-organization

### Browser Testing
- [ ] Chrome desktop
- [ ] Firefox desktop
- [ ] Safari (if available)
- [ ] Mobile browser (iOS/Android)

### Performance
- [ ] Page load time < 3 seconds
- [ ] API response time < 1 second
- [ ] Organization selector loads quickly
- [ ] Organization switching is fast
- [ ] No console errors
- [ ] No network errors

---

## 📊 Log Monitoring

- [ ] Backend logs show no errors
- [ ] Frontend logs show no errors
- [ ] Successful startup messages seen
- [ ] Database connection confirmed
- [ ] Organization switch audit logs visible (marked with [AUDIT])
- [ ] UserAccount table functioning properly

---

## 📋 Post-Deployment

- [ ] Monitoring active (first hour)
- [ ] Documentation updated
- [ ] Bug list updated (mark #30, #31 complete)
- [ ] Feature list updated (mark multi-org TODO complete)
- [ ] Changelog updated
- [ ] Team notified (if applicable)
- [ ] Release notes published
- [ ] Deployment notes documented
- [ ] Lessons learned captured
- [ ] Update releases/README.md with v1.3.0 entry

---

## 🔧 Quick Reference Commands

```bash
# Health checks
curl -I https://app.projectledger.ca
curl https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io/api/health

# Test multi-org API endpoints
curl -H \"Authorization: Bearer [token]\" https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io/api/user/organizations

# Monitor logs
az containerapp logs show --name projectledger-backend --resource-group projectledger-poc --tail 50 --follow
az containerapp logs show --name projectledger-frontend --resource-group projectledger-poc --tail 50 --follow

# Look for organization switch audit logs
az containerapp logs show --name projectledger-backend --resource-group projectledger-poc --tail 100 | grep \"[AUDIT]\"

# List revisions
az containerapp revision list --name projectledger-backend --resource-group projectledger-poc -o table
az containerapp revision list --name projectledger-frontend --resource-group projectledger-poc -o table

# Rollback (if needed)
az containerapp revision activate --name projectledger-frontend --resource-group projectledger-poc --revision [previous-revision]
az containerapp revision activate --name projectledger-backend --resource-group projectledger-poc --revision [previous-revision]
```

---

## 🧪 Test Credentials for Multi-Org Testing

**Multi-Org Users (will see organization selector):**
- **demo@projectledger.com** - 2 organizations (Demo Account, Secondary Demo Account)
- **lovell@microsoft.com** - 2 organizations (Microsoft Corporation as ADMIN, Legacy Account as USER)

**Single-Org Users (will auto-select, no selector):**
- **warren@microsoft.com** - 1 organization (Microsoft Corporation as USER)

---

## 📝 Deployment Notes

**Issues Encountered:**
- [None / List any issues]

**Resolution:**
- [N/A / How issues were resolved]

**Time Taken:**
- Pre-deployment: _____ min
- Build: _____ min
- Push: _____ min
- Deploy: _____ min
- Verification: _____ min
- **Total:** _____ min

**Rollback Required:** ☐ Yes ☑ No

**Key Features Deployed:**
- ✅ Multi-Organization Support (complete database, backend, frontend)
- ✅ Invoice Quote Item Selector (Bug #30)
- ✅ Beamer Button Cleanup (Bug #31)
- ✅ Documentation Organization
- ✅ Zero breaking changes for existing single-org users

---

## ✍️ Sign-off

**Deployment Status:** ☐ Success ☐ Partial ☐ Failed

**Deployed By:** ___________________________  
**Date & Time:** ___________________________  
**Signature:** _____________________________

**Verified By:** ___________________________  
**Date & Time:** ___________________________  
**Signature:** _____________________________

---

## 🎯 Next Steps

- [ ] Monitor for next hour (especially organization switching)
- [ ] Review logs at end of day for organization-related activity
- [ ] Check user feedback tomorrow on multi-org experience
- [ ] Plan next release (consider advanced multi-org features like user invitations)
- [ ] Monitor multi-org feature adoption metrics

**End Time:** [Time]  
**Total Duration:** [Duration]