# Release Notes - Version 1.6.0

> **Version:** v1.6.0  
> **Release Date:** October 15, 2025  
> **Deployer:** [Your Name]  
> **Revision Suffix:** v1-6-0

---

## 📋 Release Information

**Changes in this release:**

### ✨ Features & Improvements
- **Change Order Edit Implementation**: Enhanced Change Order editing functionality with improved user experience
- **Main Layout Fix**: Corrected main content positioning and sizing issues in MainLayout component
- **Branded Layout Standardization**: Standardized Change Order pages with consistent branded layout
- **Terms & Conditions Phase 2**: Advanced features for terms and conditions management

### 🐛 Bug Fixes
- Fixed main layout resize behavior
- Corrected content positioning in MainLayout component
- Resolved Change Order page layout inconsistencies

### 📚 Documentation
- Updated PR documentation for Change Order improvements
- Enhanced deployment checklist templates

---

## 🚀 Deployment Details

**Git Tag:** v1.6.0  
**Previous Version:** v1.5.1  
**Branch:** main

**Docker Images:**
- Backend: `projectledgerregistry.azurecr.io/projectledger-backend:v1.6.0`
- Frontend: `projectledgerregistry.azurecr.io/projectledger-frontend:v1.6.0`

**Azure Revisions:**
- Backend: `projectledger-backend--v1-6-0`
- Frontend: `projectledger-frontend--v1-6-0`

---

## ✅ Testing Checklist

### Core Functionality
- [ ] Login/logout works
- [ ] Dashboard loads correctly
- [ ] Navigation is functional
- [ ] Mobile view works

### New Features to Test
- [ ] Change Order editing functionality
- [ ] Main layout resizing behavior
- [ ] Change Order page layouts display correctly
- [ ] Terms & Conditions Phase 2 features

### Integration Points
- [ ] API calls successful
- [ ] Authentication working
- [ ] Database connections stable

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

---

## 📝 Deployment Notes

**Issues Encountered:**
- [None / List any issues]

**Resolution:**
- [N/A / How issues were resolved]

---

## 🔗 Related Documentation

- [HOW_TO_RELEASE.md](../../HOW_TO_RELEASE.md)
- [DEPLOYMENT_CHECKLIST_v1.6.0.md](./DEPLOYMENT_CHECKLIST_v1.6.0.md)
- [DEPLOYMENT_PLAN.md](../../DEPLOYMENT_PLAN.md)

---

## 📞 Rollback Information

**Previous Revision:**
- Backend: `projectledger-backend--v1-5-1`
- Frontend: `projectledger-frontend--v1-5-1`

**Rollback Command:**
```bash
# Backend rollback
az containerapp revision activate \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --revision projectledger-backend--v1-5-1

# Frontend rollback
az containerapp revision activate \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --revision projectledger-frontend--v1-5-1
```

---

## ✍️ Sign-off

**Deployment Status:** ☐ Success ☐ Partial ☐ Failed

**Deployed By:** ___________________________  
**Date & Time:** ___________________________  

**Verified By:** ___________________________  
**Date & Time:** ___________________________  

---

**Next Release:** v1.7.0 (TBD)
