````markdown
# Release Notes - Version 1.7.0

> **Version:** v1.7.0  
> **Release Date:** October 23, 2025  
> **Deployer:** GitHub Copilot  
> **Revision Suffix:** v1-7-0

---

## 📋 Release Information

**Changes in this release:**

### ✨ Features & Improvements
- **Quotes & Estimates Integration - Phase 7**: Complete quotes and estimates system with client portal support
  - Added estimate support to client portal
  - Clients can view and interact with estimates
  - Full quote/estimate lifecycle management
- **Quotes & Estimates Integration - Phase 6**: Change Order Integration for Estimates
  - Change orders now support estimates
  - Comprehensive change order workflow
- **Quotes & Estimates Integration - Phase 5**: PDF Generation for Quotes & Estimates
  - Professional PDF generation for both quotes and estimates
  - Standardized document formatting
- **Quotes & Estimates Integration - Phase 4**: Frontend list/display with type filtering and conversion
  - Quote to estimate conversion functionality
  - Advanced filtering by document type
  - Improved list views
- **Quotes & Estimates Integration - Phase 3**: Frontend form UI implementation
  - Unified quote/estimate creation forms
  - Enhanced user experience
- **Quotes & Estimates Integration - Phase 2**: API Endpoints for Quotes & Estimates
  - RESTful API endpoints for all quote/estimate operations
  - Robust validation and error handling
- **Quotes & Estimates Integration - Phase 1**: Database & Core Logic
  - Enhanced database schema for quotes/estimates
  - Contingency support for estimates
  - Type differentiation between quotes and estimates

### 🐛 Bug Fixes
- Fixed estimate creation not passing type and contingency to backend
- Fixed missing change-orders report route and database entry
- Enhanced frontend error handling for 401/403 responses
- Updated UI labels from 'Quotes' to 'Quotes & Estimates' across application
- Removed unused title field from quote/estimate forms

### 🔒 Security Enhancements
- Added authentication to admin and metrics endpoints
- Enhanced account-level security for Change Orders API
- Improved authorization checks across endpoints

### 📚 Documentation
- Comprehensive implementation documentation for quotes/estimates integration
- Updated specifications with completion status
- Enhanced security documentation
- Detailed phase-by-phase implementation reports

---

## 🚀 Deployment Details

**Git Tag:** v1.7.0  
**Previous Version:** v1.6.0  
**Branch:** main

**Docker Images:**
- Backend: `projectledgerregistry.azurecr.io/projectledger-backend:v1.7.0`
- Frontend: `projectledgerregistry.azurecr.io/projectledger-frontend:v1.7.0`

**Azure Revisions:**
- Backend: `projectledger-backend--v1-7-0`
- Frontend: `projectledger-frontend--v1-7-0`

---

## ✅ Testing Checklist

### Core Functionality
- [ ] Login/logout works
- [ ] Dashboard loads correctly
- [ ] Navigation is functional
- [ ] Mobile view works

### New Features to Test
- [ ] Quote creation and editing
- [ ] Estimate creation with contingency support
- [ ] Quote to estimate conversion
- [ ] Estimate to quote conversion
- [ ] PDF generation for quotes
- [ ] PDF generation for estimates
- [ ] Change orders for estimates
- [ ] Client portal estimate viewing
- [ ] Estimate filtering and list views
- [ ] Quote/estimate type differentiation

### Integration Points
- [ ] API calls successful for quotes/estimates
- [ ] Authentication working
- [ ] Database connections stable
- [ ] PDF generation service operational
- [ ] Client portal integration working

### Security Testing
- [ ] Admin endpoints require authentication
- [ ] Metrics endpoints protected
- [ ] Change orders respect account boundaries
- [ ] Proper 401/403 error handling

---

## 📊 Deployment Timeline

| Phase | Time | Status |
|-------|------|--------|
| Pre-deployment checks | [Time] | ⏳ |
| Docker image build | [Time] | ⏳ |
| Image push to ACR | [Time] | ⏳ |
| Backend deployment | [Time] | ⏳ |
| Frontend deployment | [Time] | ⏳ |
| Verification | [Time] | ⏳ |
| Post-deployment monitoring | [Time] | ⏳ |

**Total Duration:** [Duration]

---

## 🔍 Verification Results

### Automated Checks
- [ ] Frontend health check (200 OK)
- [ ] Backend health check (200 OK)
- [ ] Database connectivity verified
- [ ] No console errors

### Manual Testing
- [ ] Login functionality tested
- [ ] New features verified
- [ ] Browser compatibility checked
- [ ] Mobile responsiveness confirmed
- [ ] Client portal functionality tested

---

## 📝 Deployment Notes

**Issues Encountered:**
- [None / List any issues]

**Resolution:**
- [N/A / How issues were resolved]

---

## 🔗 Related Documentation

- [HOW_TO_RELEASE.md](../../HOW_TO_RELEASE.md)
- [DEPLOYMENT_CHECKLIST_v1.7.0.md](./DEPLOYMENT_CHECKLIST_v1.7.0.md)
- [DEPLOYMENT_PLAN.md](../../DEPLOYMENT_PLAN.md)
- [SPEC_quotes_estimates_integration.md](../../../backlog/critical/SPEC_quotes_estimates_integration.md)

---

## 📞 Rollback Information

**Previous Revision:**
- Backend: `projectledger-backend--v1-6-0`
- Frontend: `projectledger-frontend--v1-6-0`

**Rollback Command:**
```bash
# Backend rollback
az containerapp revision activate \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --revision projectledger-backend--v1-6-0

# Frontend rollback
az containerapp revision activate \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --revision projectledger-frontend--v1-6-0
```

---

## ✍️ Sign-off

**Deployment Status:** ☐ Success ☐ Partial ☐ Failed

**Deployed By:** ___________________________  
**Date & Time:** ___________________________  

**Verified By:** ___________________________  
**Date & Time:** ___________________________  

---

**Next Release:** v1.8.0 (TBD)

````
