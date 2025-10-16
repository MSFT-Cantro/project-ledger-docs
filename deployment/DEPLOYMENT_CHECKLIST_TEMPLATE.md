# Production Deployment Checklist - vX.Y.Z

> **Version:** X.Y.Z  
> **Date:** [Deployment Date]  
> **Deployer:** [Your Name]  
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
- [ ] Backend images pushed to ACR (latest + vX.Y.Z)
- [ ] Frontend images pushed to ACR (latest + vX.Y.Z)
- [ ] Git tag vX.Y.Z created and pushed

---

## 🚀 Deployment

- [ ] Current revisions backed up
- [ ] Backend deployed with revision vX-Y-Z
- [ ] Frontend deployed with revision vX-Y-Z
- [ ] 15-second stabilization wait after each deployment
- [ ] Deployment completed without errors

---

## ✅ Verification & Testing

### Health Checks
- [ ] Frontend health check (200 OK)
- [ ] Backend health check (200 OK, DB connected)
- [ ] No 404 or 500 errors

### Database Verification
- [ ] All migrations applied successfully
- [ ] UserAccount table has records (critical for authentication)
- [ ] SubscriptionPlan table populated
- [ ] AccountSubscription records exist
- [ ] No orphaned User records (all have UserAccount entries)
- [ ] TermsTemplate table has 7 standard templates (auto-seeded by migration)

### Core Functionality
- [ ] Login/logout works
- [ ] Dashboard loads
- [ ] Navigation functional
- [ ] Mobile view works
- [ ] Create operations work
- [ ] Read operations work
- [ ] Update operations work
- [ ] Delete operations work

### New Features
- [ ] [Feature 1 - describe]
- [ ] [Feature 2 - describe]
- [ ] [Feature 3 - describe]

### Integration Points
- [ ] API calls successful
- [ ] Authentication working
- [ ] External integrations functional
- [ ] [Add specific integrations]

### Browser Testing
- [ ] Chrome desktop
- [ ] Firefox desktop
- [ ] Safari (if available)
- [ ] Mobile browser (iOS/Android)

### Performance
- [ ] Page load time < 3 seconds
- [ ] API response time < 1 second
- [ ] No console errors
- [ ] No network errors

---

## 📊 Log Monitoring

- [ ] Backend logs show no errors
- [ ] Frontend logs show no errors
- [ ] Successful startup messages seen
- [ ] Database connection confirmed

---

## 📋 Post-Deployment

- [ ] Monitoring active (first hour)
- [ ] Documentation updated
- [ ] Bug list updated
- [ ] Feature list updated
- [ ] Changelog updated
- [ ] Team notified (if applicable)
- [ ] Release notes published
- [ ] Deployment notes documented
- [ ] Lessons learned captured

---

## 🔧 Quick Reference Commands

```bash
# Health checks
curl -I https://app.projectledger.ca
curl https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io/api/health

# Verify UserAccount records (CRITICAL - prevents 403 errors)
docker run --rm postgres:17.0-alpine psql "postgresql://[DB_CONNECTION_STRING]" -c "SELECT COUNT(*) FROM \"UserAccount\";"
docker run --rm postgres:17.0-alpine psql "postgresql://[DB_CONNECTION_STRING]" -c "SELECT u.email, ua.role, ua.status FROM \"User\" u LEFT JOIN \"UserAccount\" ua ON u.id = ua.\"userId\" WHERE ua.id IS NULL;"

# Monitor logs
az containerapp logs show --name projectledger-backend --resource-group projectledger-poc --tail 50 --follow
az containerapp logs show --name projectledger-frontend --resource-group projectledger-poc --tail 50 --follow

# List revisions
az containerapp revision list --name projectledger-backend --resource-group projectledger-poc -o table
az containerapp revision list --name projectledger-frontend --resource-group projectledger-poc -o table

# Rollback (if needed)
az containerapp revision activate --name projectledger-frontend --resource-group projectledger-poc --revision [previous-revision]
az containerapp revision activate --name projectledger-backend --resource-group projectledger-poc --revision [previous-revision]
```

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

- [ ] Monitor for next hour
- [ ] Review logs at end of day
- [ ] Check user feedback tomorrow
- [ ] Plan next release (if applicable)

**End Time:** [Time]  
**Total Duration:** [Duration]
