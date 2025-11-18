# Release Notes - Version 1.7.1

> **Version:** v1.7.1  
> **Release Date:** November 18, 2025  
> **Deployer:** Warren Lovell  
> **Revision Suffix:** v1-7-1

---

## 📋 Release Information

**Changes in this release:**

### ✨ Features & Improvements
- **Discord Notifications**: Release deployment notifications integration
  - Notifications sent at 5 deployment phases: Start, Backend, Frontend, Portal, Success/Failed
  - Rich Discord embeds with deployment details, version, changes, and deployer
  - Non-blocking design - webhook failures don't affect deployments
  - Python-based JSON generation for robust payload formatting
- **Discord Notifications**: Documentation change notifications
  - Automatic notifications when docs are merged to main branch
  - Smart emoji selection based on file type (📋 specs, 🚀 deployment, 📖 guides, 📝 general)
  - Changed files list in notifications
  - Commit author and SHA links
- **Testing Tools**: Created test-discord-notifications.sh script
  - Interactive menu to test individual notification phases
  - Complete deployment flow simulation
  - Validates webhook configuration
  - No actual deployment required for testing
- **Documentation**: Discord setup guides
  - Added `.github/DISCORD_SETUP.md` for documentation notifications
  - Added `.github/DISCORD_RELEASE_SETUP.md` for release notifications
  - Comprehensive troubleshooting sections
  - Security best practices

### 📚 Documentation
- Created comprehensive Discord notifications spec (SPEC_DISCORD_NOTIFICATIONS.md)
- Documented GitHub Actions workflow for docs notifications
- Release notification implementation guide
- Test results and verification procedures

---

## 🚀 Deployment Details

**Git Tag:** v1.7.1  
**Previous Version:** v1.7.0  
**Branch:** main

**Docker Images:**
- Backend: `projectledgerregistry.azurecr.io/projectledger-backend:v1-7-1`
- Frontend: `projectledgerregistry.azurecr.io/projectledger-frontend:v1-7-1`
- Portal: `projectledgerregistry.azurecr.io/projectledger-portal:latest`

**Azure Revisions:**
- Backend: `projectledger-backend--v1-7-1`
- Frontend: `projectledger-frontend--v1-7-1`
- Portal: `projectledger-portal--portal-v1-7-1`

---

## ✅ Testing Checklist

### Core Functionality
- [ ] Login/logout works
- [ ] Dashboard loads correctly
- [ ] All navigation items accessible
- [ ] No console errors on homepage
- [ ] Mobile view renders correctly

### New Features - Discord Notifications
- [ ] Deployment started notification sent to Discord
- [ ] Backend deployed notification sent
- [ ] Frontend deployed notification sent
- [ ] Portal deployed notification sent
- [ ] Deployment complete notification sent with all details
- [ ] Notifications show correct version (v1.7.1)
- [ ] Notifications show deployer name
- [ ] Notifications show changes list
- [ ] Application URLs in notifications are clickable
- [ ] All notifications formatted correctly (embeds)

### Backend Health
- [ ] Backend health endpoint returns 200
- [ ] Database connections working
- [ ] API endpoints responding
- [ ] No errors in backend logs

### Frontend Health
- [ ] Frontend loads without errors
- [ ] Static assets loading
- [ ] API calls succeeding
- [ ] No errors in browser console

### Portal Health
- [ ] Portal loads at portal.projectledger.ca
- [ ] Portal login works
- [ ] Portal displays correctly
- [ ] No console errors

### Regression Testing
- [ ] Quotes & Estimates functionality still works
- [ ] PDF generation working
- [ ] Client portal functioning
- [ ] Change orders working
- [ ] All existing features functional

---

## 🔍 Post-Deployment Monitoring

### First Hour
- [ ] Monitor Discord for all 5 deployment notifications
- [ ] Watch backend logs for errors
- [ ] Watch frontend logs for errors
- [ ] Check application performance
- [ ] Verify no user-reported issues

### First 24 Hours
- [ ] Review error logs
- [ ] Check notification delivery success rate
- [ ] Verify all services stable
- [ ] Monitor user activity

---

## 📊 Rollback Plan

If issues are encountered:

### 1. Identify Previous Revisions
```bash
az containerapp revision list \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  -o table
```

### 2. Activate Previous Revision
```bash
# Backend
az containerapp revision activate \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --revision projectledger-backend--v1-7-0

# Frontend
az containerapp revision activate \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --revision projectledger-frontend--v1-7-0

# Portal
az containerapp revision activate \
  --name projectledger-portal \
  --resource-group projectledger-poc \
  --revision projectledger-portal--portal-v1-7-0
```

### 3. Verify Rollback
```bash
curl -I https://app.projectledger.ca
curl https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io/api/health
```

---

## 📝 Notes

### Known Issues
- None at time of release

### Breaking Changes
- None

### Migration Notes
- No database migrations in this release
- No environment variable changes required
- Discord webhooks are optional (deployment continues without them)

### Configuration Changes
- **Optional**: Set `DISCORD_RELEASE_WEBHOOK` environment variable for release notifications
- **Optional**: Set `DISCORD_DOCS_WEBHOOK` GitHub secret for documentation notifications

---

## 👥 Team Communication

**Announcement Message:**
```
🎉 Project Ledger v1.7.1 has been deployed to production!

New Features:
✅ Discord notifications for releases - you'll now see notifications here when we deploy!
✅ Discord notifications for documentation changes
✅ Test script for verifying notifications without deployments

This is a small infrastructure release focused on improving team communication and deployment visibility.

No user-facing changes. All existing functionality remains the same.

Questions? Feel free to ask!
```

**Social Media:**
Not required for this infrastructure-focused release.

---

## ✅ Sign-Off

**Deployment Completed:**
- [ ] Date: _______________
- [ ] Time: _______________
- [ ] Deployed by: _______________
- [ ] All tests passed: Yes / No
- [ ] Issues encountered: _______________
- [ ] Rollback required: Yes / No

**Post-Deployment:**
- [ ] Team notified
- [ ] Documentation updated
- [ ] Monitoring active
- [ ] No critical issues after 24 hours

---

**Approvals:**
- Deployer: _______________ Date: _______________
- Technical Lead: _______________ Date: _______________
