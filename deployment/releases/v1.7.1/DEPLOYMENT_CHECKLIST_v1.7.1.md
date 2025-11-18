# Production Deployment Checklist - v1.7.1

> **Version:** 1.7.1  
> **Date:** November 18, 2025  
> **Deployer:** Warren Lovell  
> **Start Time:** _______

---

## ☑️ Pre-Deployment

- [ ] All changes committed and pushed to GitHub
- [ ] Git status is clean (only new files: test script and release script)
- [ ] All tests passing locally
- [ ] Docker Desktop running
- [ ] Logged into Azure CLI (`az login`)
- [ ] ACR access verified
- [ ] Discord webhook configured (`DISCORD_RELEASE_WEBHOOK` set)

---

## 🏗️ Build & Push

- [ ] Backend Docker image built
- [ ] Frontend Docker image built
- [ ] Portal Docker image built
- [ ] Backend images pushed to ACR (latest + v1-7-1)
- [ ] Frontend images pushed to ACR (latest + v1-7-1)
- [ ] Portal image pushed to ACR (latest)
- [ ] Git tag v1.7.1 created and pushed

---

## 🚀 Deployment

- [ ] Current revisions backed up
- [ ] Backend deployed with revision v1-7-1
- [ ] Frontend deployed with revision v1-7-1
- [ ] Portal deployed with revision portal-v1-7-1
- [ ] 30-second stabilization wait after each deployment
- [ ] Deployment completed without errors

---

## 🔔 Discord Notifications

- [ ] "🚀 Deployment Started" notification received in Discord
- [ ] "⚙️ Backend Deployed" notification received
- [ ] "🎨 Frontend Deployed" notification received
- [ ] "🌐 Portal Deployed" notification received
- [ ] "✅ Deployment Complete" notification received
- [ ] All notifications show correct version (v1.7.1)
- [ ] All notifications show deployer name
- [ ] All notifications show changes list
- [ ] Application URLs in notifications work

---

## ✅ Verification & Testing

### Health Checks
- [ ] Frontend health check (200 OK)
- [ ] Backend health check (200 OK, DB connected)
- [ ] Portal health check (200 OK)
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

### Regression Testing (Ensure Previous Features Still Work)
- [ ] Quotes & Estimates module functioning
- [ ] PDF generation working
- [ ] Client portal accessible
- [ ] Change orders operational
- [ ] Invoice creation/management working
- [ ] Terms & conditions system working
- [ ] Organization selector functioning
- [ ] Approval workflows operational

### New Features (v1.7.1 - Discord Notifications)
- [ ] Test script works: `./tools/deployment/test-discord-notifications.sh`
- [ ] Documentation webhook (DISCORD_DOCS_WEBHOOK) configured in GitHub
- [ ] Release webhook (DISCORD_RELEASE_WEBHOOK) configured locally
- [ ] Setup documentation accessible (DISCORD_SETUP.md, DISCORD_RELEASE_SETUP.md)

### Browser Testing
- [ ] Chrome desktop
- [ ] Firefox desktop  
- [ ] Mobile browser (responsive view)

### Performance
- [ ] Page load time < 3 seconds
- [ ] API response time < 1 second
- [ ] No console errors
- [ ] No network errors

---

## 📊 Log Monitoring

- [ ] Backend logs show no errors
- [ ] Frontend logs show no errors
- [ ] Portal logs show no errors
- [ ] Successful startup messages seen
- [ ] Database connection confirmed
- [ ] All 5 Discord notifications logged successfully

---

## 📋 Post-Deployment

- [ ] Monitoring active (first hour)
- [ ] Documentation updated
- [ ] Spec updated (SPEC_DISCORD_NOTIFICATIONS.md - already at 100%)
- [ ] Team notified via Discord
- [ ] Release notes published (RELEASE_NOTES_v1.7.1.md)
- [ ] Deployment notes documented
- [ ] Lessons learned captured

---

## 🔧 Quick Reference Commands

```bash
# Health checks
curl -I https://app.projectledger.ca
curl https://portal.projectledger.ca
curl https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io/api/health

# Test Discord notifications
export DISCORD_RELEASE_WEBHOOK="your-webhook-url"
./tools/deployment/test-discord-notifications.sh

# Monitor logs
az containerapp logs show --name projectledger-backend --resource-group projectledger-poc --tail 50 --follow
az containerapp logs show --name projectledger-frontend --resource-group projectledger-poc --tail 50 --follow
az containerapp logs show --name projectledger-portal --resource-group projectledger-poc --tail 50 --follow

# List revisions
az containerapp revision list --name projectledger-backend --resource-group projectledger-poc -o table
az containerapp revision list --name projectledger-frontend --resource-group projectledger-poc -o table
az containerapp revision list --name projectledger-portal --resource-group projectledger-poc -o table

# Rollback (if needed)
az containerapp revision activate --name projectledger-backend --resource-group projectledger-poc --revision projectledger-backend--v1-7-0
az containerapp revision activate --name projectledger-frontend --resource-group projectledger-poc --revision projectledger-frontend--v1-7-0
az containerapp revision activate --name projectledger-portal --resource-group projectledger-poc --revision projectledger-portal--portal-v1-7-0
```

---

## 📝 Deployment Notes

**Issues Encountered:**
- [None / List any issues]

**Resolution:**
- [N/A / How issues were resolved]

**Discord Notification Results:**
- Deployment Started: ☐ Success ☐ Failed
- Backend Deployed: ☐ Success ☐ Failed
- Frontend Deployed: ☐ Success ☐ Failed
- Portal Deployed: ☐ Success ☐ Failed
- Deployment Complete: ☐ Success ☐ Failed

**Time Taken:**
- Pre-deployment: _____ min
- Build: _____ min
- Push: _____ min
- Deploy: _____ min
- Verification: _____ min
- **Total:** _____ min

**Rollback Required:** ☐ Yes ☐ No

---

## ✍️ Sign-off

**Deployment Status:** ☐ Success ☐ Partial ☐ Failed

**Deployed By:** Warren Lovell  
**Date & Time:** ___________________________  
**Signature:** _____________________________

**Verified By:** ___________________________  
**Date & Time:** ___________________________  
**Signature:** _____________________________

---

## 🎯 Next Steps

- [ ] Monitor for next hour
- [ ] Review logs at end of day
- [ ] Confirm all 5 Discord notifications appeared correctly
- [ ] Test next documentation merge triggers notification
- [ ] Plan next release (if applicable)

**End Time:** _______  
**Total Duration:** _______

