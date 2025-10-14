# Production Deployment Checklist - v1.5.0

> **Version:** 1.5.0  
> **Date:** October 14, 2025  
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
- [ ] Version numbers updated to 1.5.0

---

## 🏗️ Build & Push

- [ ] Backend Docker image built
- [ ] Frontend Docker image built
- [ ] Backend images pushed to ACR (latest + v1.5.0)
- [ ] Frontend images pushed to ACR (latest + v1.5.0)
- [ ] Git tag v1.5.0 created and pushed

---

## 🚀 Deployment

- [ ] Current revisions backed up
- [ ] Backend deployed with revision v1-5-0
- [ ] Frontend deployed with revision v1-5-0
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
- [ ] ChangeOrder table accessible
- [ ] approvalToken field added to ChangeOrder
- [ ] No orphaned records
- [ ] Foreign key constraints intact

### Core Functionality
- [ ] Login/logout works
- [ ] Dashboard loads
- [ ] Navigation functional (including "Change Orders" menu item)
- [ ] Mobile view works
- [ ] Create operations work
- [ ] Read operations work
- [ ] Update operations work
- [ ] Delete operations work

### New Features - Change Orders List & Detail
- [ ] Change Orders menu item visible and clickable
- [ ] List page loads without errors
- [ ] Can filter by status (DRAFT, SENT, APPROVED, DECLINED, EXECUTED)
- [ ] Can filter by type (ADDITIVE, DEDUCTIVE, NEUTRAL)
- [ ] Can filter by date range
- [ ] Search functionality works
- [ ] Stats bar displays accurate metrics
- [ ] "View Reports" button navigates correctly
- [ ] Detail page shows all change order information
- [ ] Status badges render correctly
- [ ] Financial impact summary calculates correctly
- [ ] Related quote information displays
- [ ] Change order items table displays
- [ ] History timeline shows (if available)

### New Features - Change Order Creation
- [ ] "Create Change Order" button works
- [ ] Create page loads without errors
- [ ] Quote selection dropdown populated
- [ ] Can add new line items
- [ ] Can modify existing quote items
- [ ] Can remove quote items
- [ ] Quantity and price fields update correctly
- [ ] Financial impact preview updates in real-time
- [ ] Can save as DRAFT
- [ ] Can send for approval
- [ ] Validation works (required fields)
- [ ] Error messages display appropriately

### New Features - Client Approval Workflow
- [ ] Can send change order to client from detail page
- [ ] Approval email sent successfully
- [ ] Approval link generates with valid token
- [ ] Public approval page loads without login
- [ ] Approval page displays change order correctly
- [ ] Client can approve change order
- [ ] Client can decline change order
- [ ] Approval status updates in database
- [ ] Confirmation email sent after approval
- [ ] Confirmation email sent after decline
- [ ] Token expiration works (if past validUntil date)

### New Features - PDF Generation
- [ ] "Generate PDF" button visible on detail page
- [ ] PDF downloads successfully
- [ ] PDF contains change order number
- [ ] PDF contains client information
- [ ] PDF contains project information
- [ ] PDF contains all change order items
- [ ] PDF displays financial impact correctly
- [ ] PDF includes approval section (if applicable)
- [ ] PDF formatting looks professional

### New Features - Project Integration
- [ ] Project detail page loads successfully
- [ ] Change Orders accordion section visible
- [ ] Count badge displays correctly (number of change orders)
- [ ] Change orders table shows related change orders
- [ ] Quote reference column displays quote number
- [ ] Can click to navigate to change order detail
- [ ] No errors in browser console

### New Features - Reporting
- [ ] Change Orders report accessible from /reports/change-orders
- [ ] Report page loads without errors
- [ ] Summary metrics display (total, approved, executed, etc.)
- [ ] Approval rate calculation correct
- [ ] Total delta amount displays correctly
- [ ] Status distribution chart renders
- [ ] Report table lists all change orders
- [ ] Financial data accurate (delta amounts)
- [ ] Can export report (if enabled)
- [ ] Report filters work (if applicable)

### Data Operations
- [ ] Can create new change orders
- [ ] Can read existing change orders
- [ ] Can update draft change orders
- [ ] Can delete draft change orders
- [ ] Search/filter functionality works
- [ ] Sorting works (by date, status, etc.)

### Integration Points
- [ ] API calls succeed (/api/change-orders endpoints)
- [ ] Authentication tokens work
- [ ] Quote integration works (change orders link to quotes correctly)
- [ ] Project integration works (change orders show on project pages)
- [ ] Email service functional (approval/confirmation emails)
- [ ] PDF service functional (document generation)

### Browser Testing
- [ ] Chrome (desktop)
- [ ] Firefox (desktop)
- [ ] Safari (if available)
- [ ] Mobile browser (iOS/Android)

### Performance
- [ ] Page load time < 3 seconds
- [ ] API response time < 1 second
- [ ] PDF generation < 5 seconds
- [ ] Email sending < 10 seconds
- [ ] No console errors
- [ ] No network errors

---

## 📊 Log Monitoring

- [ ] Backend logs show no errors
- [ ] Frontend logs show no errors
- [ ] Successful startup messages seen
- [ ] Database connection confirmed
- [ ] Migration applied successfully
- [ ] Email service initialized
- [ ] PDF service initialized
- [ ] Change order routes registered

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
- [ ] Marketing post published

---

## 🔧 Quick Reference Commands

```bash
# Health checks
curl -I https://app.projectledger.ca
curl https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io/api/health

# Test change orders API
curl https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io/api/change-orders

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
- [ ] Plan Phase 5 (Financial Integration)

**End Time:** [Time]  
**Total Duration:** [Duration]
