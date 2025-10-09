# Production Deployment Checklist - v1.4.0

> **Version:** 1.4.0  
> **Date:** October 9, 2025  
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
- [ ] 🚨 **MANDATORY: Production database backup created**
- [ ] 🚨 **MANDATORY: Database state verified (no schema drift)**
- [ ] 🚨 **MANDATORY: Migration status confirmed as "up to date"**

---

## 🏗️ Build & Push

- [ ] Backend Docker image built
- [ ] Frontend Docker image built
- [ ] Backend images pushed to ACR (latest + v1.4.0)
- [ ] Frontend images pushed to ACR (latest + v1.4.0)
- [ ] Git tag v1.4.0 created and pushed

---

## 🚀 Deployment

- [ ] Current revisions backed up
- [ ] Backend deployed with revision v1-4-0
- [ ] Frontend deployed with revision v1-4-0
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
- [ ] UserInvoiceTemplate table created (20251009201915 migration)
- [ ] UserQuoteTemplate table created (20251009183518 migration)
- [ ] UserAccount table has records (critical for authentication)
- [ ] SubscriptionPlan table populated
- [ ] AccountSubscription records exist
- [ ] No orphaned User records (all have UserAccount entries)

### Core Functionality
- [ ] Login/logout works
- [ ] Dashboard loads
- [ ] Navigation functional
- [ ] Mobile view works
- [ ] Create operations work
- [ ] Read operations work
- [ ] Update operations work
- [ ] Delete operations work

### New Features - Invoice Templates (Item#33)
- [ ] Can save invoice as template from invoice detail page
- [ ] "More" menu (⋮) appears on invoice detail page
- [ ] "Save as Template" option in more menu
- [ ] Template save modal opens with name/description fields
- [ ] Template saves successfully to database
- [ ] Success toast notification appears
- [ ] "Use Template" option appears on invoice create page
- [ ] Template manager modal opens from create page
- [ ] List of templates displays correctly
- [ ] Can select template and it loads invoice data
- [ ] Template data includes: client, project, items, notes, tax settings
- [ ] Can set a template as default
- [ ] Default indicator (★) shows correctly
- [ ] Can delete templates
- [ ] Delete confirmation works
- [ ] All users in account can see the same templates
- [ ] Back button and more button grouped together on create page

### New Features - Quote Templates (Item#32)
- [ ] Can save quote as template from quote detail page
- [ ] "More" menu (⋮) appears on quote detail page
- [ ] "Save as Template" option in more menu
- [ ] Template save modal opens with name/description fields
- [ ] Template saves successfully to database
- [ ] Success toast notification appears
- [ ] "Use Template" option appears on quote create page
- [ ] Template manager modal opens from create page
- [ ] List of templates displays correctly
- [ ] Can select template and it loads quote data
- [ ] Template data includes: client, project, items, notes, tax settings
- [ ] Can set a template as default
- [ ] Default indicator (★) shows correctly
- [ ] Can delete templates
- [ ] Delete confirmation works
- [ ] All users in account can see the same templates

### API Endpoints
- [ ] GET /api/user-invoice-templates returns 200
- [ ] POST /api/user-invoice-templates creates template
- [ ] GET /api/user-invoice-templates/:id returns template
- [ ] PATCH /api/user-invoice-templates/:id updates template
- [ ] DELETE /api/user-invoice-templates/:id deletes template
- [ ] GET /api/user-quote-templates returns 200
- [ ] POST /api/user-quote-templates creates template
- [ ] GET /api/user-quote-templates/:id returns template
- [ ] PATCH /api/user-quote-templates/:id updates template
- [ ] DELETE /api/user-quote-templates/:id deletes template

### Integration Points
- [ ] API calls successful
- [ ] Authentication working
- [ ] External integrations functional (PayPal, Beamer)
- [ ] Template data persists across sessions

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
- [ ] Template list loads quickly
- [ ] Template modal opens/closes smoothly

---

## 📊 Log Monitoring

- [ ] Backend logs show no errors
- [ ] Frontend logs show no errors
- [ ] Successful startup messages seen
- [ ] Database connection confirmed
- [ ] Migration logs show successful application
- [ ] No 404 errors for /api/user-invoice-templates
- [ ] No 404 errors for /api/user-quote-templates
- [ ] No 400 validation errors
- [ ] No 500 server errors

---

## 📋 Post-Deployment

- [ ] Monitoring active (first hour)
- [ ] Documentation updated
- [ ] Bug list updated (Item#32 and Item#33 marked complete)
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

# Test new endpoints
curl -H "Authorization: Bearer YOUR_TOKEN" https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io/api/user-invoice-templates
curl -H "Authorization: Bearer YOUR_TOKEN" https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io/api/user-quote-templates

# Verify UserAccount records (CRITICAL - prevents 403 errors)
docker run --rm postgres:17.0-alpine psql "postgresql://postgres:HzxHJdKgjLaamQSqYUUdD8oE8@projectledger-db.eastus2.azurecontainer.io:5432/projectledger" -c "SELECT COUNT(*) FROM \"UserAccount\";"

# Verify new tables exist
docker run --rm postgres:17.0-alpine psql "postgresql://postgres:HzxHJdKgjLaamQSqYUUdD8oE8@projectledger-db.eastus2.azurecontainer.io:5432/projectledger" -c "SELECT COUNT(*) FROM \"UserInvoiceTemplate\";"
docker run --rm postgres:17.0-alpine psql "postgresql://postgres:HzxHJdKgjLaamQSqYUUdD8oE8@projectledger-db.eastus2.azurecontainer.io:5432/projectledger" -c "SELECT COUNT(*) FROM \"UserQuoteTemplate\";"

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

**Database Migrations Applied:**
- 20251009183518_add_user_quote_templates
- 20251009201915_add_user_invoice_templates

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
- [ ] Track template adoption metrics
- [ ] Gather user feedback on template features
- [ ] Plan next release (if applicable)

**End Time:** [Time]  
**Total Duration:** [Duration]
