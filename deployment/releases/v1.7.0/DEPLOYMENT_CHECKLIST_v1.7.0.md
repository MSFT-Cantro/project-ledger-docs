````markdown
# Production Deployment Checklist - v1.7.0

> **Version:** 1.7.0  
> **Date:** October 23, 2025  
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
- [ ] Backend images pushed to ACR (latest + v1.7.0)
- [ ] Frontend images pushed to ACR (latest + v1.7.0)
- [ ] Git tag v1.7.0 created and pushed

---

## 🚀 Deployment

- [ ] Current revisions backed up
- [ ] Backend deployed with revision v1-7-0
- [ ] Frontend deployed with revision v1-7-0
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
- [ ] Quote/Estimate tables properly configured
- [ ] Change order tables updated for estimate support
- [ ] No orphaned records

### Core Functionality
- [ ] Login/logout works
- [ ] Dashboard loads
- [ ] Navigation functional
- [ ] Mobile view works
- [ ] Create operations work
- [ ] Read operations work
- [ ] Update operations work
- [ ] Delete operations work

### New Features - Quotes & Estimates Integration
- [ ] Create new quote
- [ ] Create new estimate with contingency
- [ ] Edit existing quote
- [ ] Edit existing estimate
- [ ] Convert quote to estimate
- [ ] Convert estimate to quote
- [ ] Filter quotes by type (All/Quotes/Estimates)
- [ ] Quote list displays correctly with type labels
- [ ] Estimate list displays correctly with type labels
- [ ] Quote detail page shows correct information
- [ ] Estimate detail page shows contingency

### PDF Generation
- [ ] Generate PDF for quote
- [ ] Generate PDF for estimate
- [ ] PDF includes all line items
- [ ] PDF formatting is correct
- [ ] PDF downloads successfully

### Change Orders for Estimates
- [ ] Create change order for estimate
- [ ] Edit change order for estimate
- [ ] View change order list for estimate
- [ ] Change order properly affects estimate total
- [ ] Change order status tracking works

### Client Portal - Estimates
- [ ] Client can view estimates in portal
- [ ] Estimate details display correctly
- [ ] Client can approve/decline estimate
- [ ] Estimate status updates correctly
- [ ] Navigation between quotes and estimates works

### Security Features
- [ ] Admin endpoints require authentication
- [ ] Metrics endpoints require authentication
- [ ] Change orders respect account boundaries
- [ ] 401 errors handled correctly
- [ ] 403 errors handled correctly

### Integration Points
- [ ] API calls successful
- [ ] Authentication working
- [ ] External integrations functional

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
- [ ] No authentication errors

---

## 📋 Post-Deployment

- [ ] Monitoring active (first hour)
- [ ] Documentation updated
- [ ] Bug list updated
- [ ] Feature list updated
- [ ] Changelog updated
- [ ] Team notified (if applicable)
- [ ] Release notes published
- [ ] Marketing post published (MARKETING_POST_v1.7.0.md)
- [ ] Social media announcements posted
- [ ] Customer email notifications sent
- [ ] Deployment notes documented
- [ ] Lessons learned captured

---

## 🔧 Quick Reference Commands

```bash
# Health checks
curl -I https://app.projectledger.ca
curl https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io/api/health

# Verify database migrations
docker run --rm postgres:17.0-alpine psql "postgresql://[DB_CONNECTION_STRING]" -c "SELECT version FROM _prisma_migrations ORDER BY finished_at DESC LIMIT 5;"

# Monitor logs
az containerapp logs show --name projectledger-backend --resource-group projectledger-poc --tail 50 --follow
az containerapp logs show --name projectledger-frontend --resource-group projectledger-poc --tail 50 --follow

# List revisions
az containerapp revision list --name projectledger-backend --resource-group projectledger-poc -o table
az containerapp revision list --name projectledger-frontend --resource-group projectledger-poc -o table

# Rollback (if needed)
az containerapp revision activate --name projectledger-frontend --resource-group projectledger-poc --revision projectledger-frontend--v1-6-0
az containerapp revision activate --name projectledger-backend --resource-group projectledger-poc --revision projectledger-backend--v1-6-0
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

````
