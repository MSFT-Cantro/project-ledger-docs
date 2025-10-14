# Deployment Checklist - Version 1.5.1

**Release**: v1.5.1  
**Type**: Patch Release  
**Date**: January 2025

---

## Pre-Deployment Checklist

### Code Review
- [x] Mobile menu fix reviewed and tested locally
- [x] Changes committed to main branch
- [x] All changes pushed to GitHub
- [ ] Git tag v1.5.1 created and pushed

### Version Updates
- [ ] Root `package.json` updated to 1.5.1
- [ ] `apps/frontend/package.json` updated to 1.5.1
- [ ] `apps/backend/package.json` updated to 1.5.1

### Environment Preparation
- [ ] Azure CLI authenticated
- [ ] Docker running and authenticated to Azure Container Registry
- [ ] On main branch with latest changes pulled
- [ ] No uncommitted changes in working directory

### Documentation
- [x] Release notes created
- [x] Deployment checklist created
- [ ] Deployment script created

---

## Deployment Execution

### Automated Deployment (Recommended)
- [ ] Navigate to `tools/deployment/releases/`
- [ ] Run `bash release-v1.5.1.sh`
- [ ] Script completes without errors

### Build Process
- [ ] Backend Docker image built successfully
- [ ] Frontend Docker image built successfully
- [ ] Both images tagged as v1.5.1
- [ ] Both images pushed to Azure Container Registry

### Azure Deployment
- [ ] Backend container app updated with v1.5.1 image
- [ ] Backend revision created with suffix `v1-5-1`
- [ ] Frontend container app updated with v1.5.1 image
- [ ] Frontend revision created with suffix `v1-5-1`
- [ ] No deployment errors in Azure CLI output

---

## Post-Deployment Verification

### Application Health
- [ ] Frontend URL responds: `https://app.projectledger.ca`
- [ ] Frontend returns HTTP 200 OK
- [ ] No JavaScript errors in browser console
- [ ] Backend API endpoints responding

### Mobile Navigation Testing

#### Mobile Menu Structure
- [ ] Mobile menu icon visible and clickable
- [ ] Mobile drawer opens smoothly
- [ ] All menu items render correctly
- [ ] Menu styling matches design system

#### New Menu Items
- [ ] "Change Orders" menu item visible
- [ ] Change Orders submenu expands/collapses
- [ ] "All Change Orders" submenu item present
- [ ] "Create Change Order" submenu item present
- [ ] "Reports" menu item visible
- [ ] "Account Settings" visible for ADMIN users
- [ ] "Account Settings" hidden for non-ADMIN users

#### Navigation Functionality
- [ ] Dashboard link works
- [ ] Clients link works
- [ ] Projects link works
- [ ] Estimates link works
- [ ] Invoices link works
- [ ] Change Orders link works
- [ ] All Change Orders submenu works
- [ ] Create Change Order submenu works
- [ ] Reports link works
- [ ] Account Settings link works (ADMIN)

### Cross-Device Testing

#### iOS Testing
- [ ] Tested on iPhone (iOS Safari)
- [ ] Menu opens and closes correctly
- [ ] All navigation links work
- [ ] Submenu expansion works
- [ ] No visual glitches

#### Android Testing
- [ ] Tested on Android (Chrome)
- [ ] Menu opens and closes correctly
- [ ] All navigation links work
- [ ] Submenu expansion works
- [ ] No visual glitches

#### Tablet Testing
- [ ] Tested on tablet device
- [ ] Navigation behaves appropriately
- [ ] All features accessible

### Desktop Verification
- [ ] Desktop menu unchanged
- [ ] Desktop navigation still functional
- [ ] No regression in desktop experience

### Backend Verification (No Changes Expected)
- [ ] API health endpoint responding
- [ ] Database connections stable
- [ ] No backend errors in logs

---

## Azure Container Apps Status

### Revision Status
```bash
az containerapp revision list \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --query "[?properties.active].{Name:name, Active:properties.active, Traffic:properties.trafficWeight}" \
  --output table
```
- [ ] Revision `projectledger-frontend--v1-5-1` is active
- [ ] Receiving 100% of traffic

```bash
az containerapp revision list \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --query "[?properties.active].{Name:name, Active:properties.active, Traffic:properties.trafficWeight}" \
  --output table
```
- [ ] Revision `projectledger-backend--v1-5-1` is active
- [ ] Receiving 100% of traffic

### Container Health
- [ ] Frontend replicas all healthy
- [ ] Backend replicas all healthy
- [ ] No container restarts or crashes
- [ ] Resource utilization normal

---

## User Acceptance Testing

### Feature Verification
- [ ] ADMIN user can access all menu items on mobile
- [ ] STAFF user can access non-admin menu items
- [ ] Change Orders accessible from mobile menu
- [ ] Reports accessible from mobile menu
- [ ] Mobile experience matches desktop feature set

### Regression Testing
- [ ] All existing mobile features work
- [ ] No broken links
- [ ] Authentication still works
- [ ] Authorization rules still apply
- [ ] No visual regressions

---

## Monitoring & Observability

### Application Logs
- [ ] Frontend logs checked - no errors
- [ ] Backend logs checked - no errors
- [ ] No unusual error patterns

### Performance
- [ ] Page load times acceptable
- [ ] Mobile menu opens quickly
- [ ] Navigation responsive
- [ ] No performance degradation

---

## Documentation & Communication

### Internal Documentation
- [ ] Release notes reviewed by team
- [ ] Deployment checklist completed
- [ ] Any issues documented

### User Communication (Optional for Patch)
- [ ] Internal team notified of fix
- [ ] Users made aware if needed

---

## Rollback Plan

### Rollback Triggers
- Critical navigation failures
- Mobile menu not rendering
- Feature access broken
- Major browser compatibility issues

### Rollback Procedure
If rollback needed:
```bash
# Frontend rollback
az containerapp revision set-mode \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --mode single \
  --revision projectledger-frontend--v1-5-0

# Backend rollback (if needed)
az containerapp revision set-mode \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --mode single \
  --revision projectledger-backend--v1-5-0
```

- [ ] Rollback not required (check at end)

---

## Sign-Off

### Deployment Team
- **Deployed By**: _________________
- **Date**: _________________
- **Time**: _________________

### Verification
- **Verified By**: _________________
- **Date**: _________________
- **Status**: ☐ Success ☐ Success with Issues ☐ Rollback Required

### Issues Encountered
_Document any issues or deviations from expected deployment:_

---

### Notes
_Additional observations or comments:_

---

## Completion
- [ ] All checklist items completed
- [ ] Deployment successful
- [ ] Application stable
- [ ] Ready for monitoring
