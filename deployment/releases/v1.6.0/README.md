# Release v1.6.0 Documentation

**Release Date:** October 15, 2025  
**Deployment Time:** 22:16 PDT  
**Status:** ✅ Successfully Deployed  
**Branch:** main (via feature/change-order-edit-implementation)

---

## 📁 Contents

This folder contains all documentation related to the v1.6.0 release:

- **[RELEASE_NOTES_v1.6.0.md](./RELEASE_NOTES_v1.6.0.md)** - Complete technical release notes
- **[DEPLOYMENT_CHECKLIST_v1.6.0.md](./DEPLOYMENT_CHECKLIST_v1.6.0.md)** - Deployment verification checklist
- **[MARKETING_ANNOUNCEMENT.md](./MARKETING_ANNOUNCEMENT.md)** - Marketing materials and announcements

---

## 🚀 Release Summary

### Features Delivered

1. **Enhanced Change Order Management**
   - Redesigned editing interface
   - Standardized branded layout across all Change Order pages
   - Improved user experience and workflow

2. **Main Layout Improvements**
   - Fixed content positioning and sizing issues
   - Better responsive behavior
   - Optimized screen space utilization

3. **Terms & Conditions Phase 2**
   - Advanced features for terms management
   - Template library integration
   - Enhanced contract handling

### Technical Details

- **Git Tag:** v1.6.0
- **Previous Version:** v1.5.1
- **Docker Images:**
  - Backend: `projectledgerregistry.azurecr.io/projectledger-backend:v1.6.0`
  - Frontend: `projectledgerregistry.azurecr.io/projectledger-frontend:v1.6.0`
- **Azure Revisions:**
  - Backend: `projectledger-backend--v1-6-0`
  - Frontend: `projectledger-frontend--v1-6-0`

### Deployment Statistics

- **Build Time:** ~2.5 minutes
- **Push Time:** ~1 minute
- **Deployment Time:** ~4 minutes
- **Total Time:** ~8 minutes
- **Downtime:** 0 seconds ✅
- **Migrations Applied:** 2 new migrations
- **Health Status:** All systems healthy

---

## ✅ Deployment Verification

### Automated Checks
- ✅ Frontend responding at https://app.projectledger.ca
- ✅ Backend health check passed
- ✅ Database connectivity verified
- ✅ Migrations applied successfully

### Database State
- ✅ 29 migrations applied (2 new)
- ✅ Users: 1
- ✅ Accounts: 2
- ✅ Subscription Plans: 3
- ✅ UserAccount relationships: 1
- ✅ Business data intact

### Production Status
- **Frontend:** Running, Healthy, 1 replica
- **Backend:** Running, Healthy, 1 replica
- **Database:** PostgreSQL container running
- **DNS:** app.projectledger.ca resolving correctly

---

## 🔗 Related Resources

### Documentation
- [HOW_TO_RELEASE.md](../../HOW_TO_RELEASE.md) - Release process guide
- [DEPLOYMENT_PLAN.md](../../DEPLOYMENT_PLAN.md) - Deployment strategy

### Scripts
- [release-v1.6.0.sh](../../../tools/deployment/release-v1.6.0.sh) - Deployment script

### GitHub
- **Repository:** MSFT-Cantro/project-ledger
- **Tag:** [v1.6.0](https://github.com/MSFT-Cantro/project-ledger/releases/tag/v1.6.0)
- **Commit:** 942ec9b

---

## 📊 Post-Deployment Notes

### Issues Encountered
- ⚠️ Login requires correct email domain (admin@projectledger.ca not .com)
- Note: OAuth providers intentionally not configured

### Resolutions
- User account verified in database
- Correct email documented for support team

### Monitoring
- First hour: No errors reported
- Health checks: All passing
- Performance: Normal

---

## 🎯 Success Criteria

All success criteria were met:

- ✅ Zero downtime deployment
- ✅ All migrations applied successfully
- ✅ Database integrity maintained
- ✅ Frontend and backend healthy
- ✅ New features accessible
- ✅ No rollback required
- ✅ Documentation complete
- ✅ Marketing materials prepared

---

## 📞 Rollback Information

**Previous Stable Version:** v1.5.1

**Rollback Commands:**
```bash
# Backend
az containerapp revision activate \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --revision projectledger-backend--v1-5-1

# Frontend
az containerapp revision activate \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --revision projectledger-frontend--v1-5-1
```

**Backup Location:** `.deployment-backup/`

---

## 🎉 Team Acknowledgments

Special thanks to everyone who contributed to this release:
- Development team for feature implementation
- QA team for testing
- DevOps for smooth deployment
- Product team for requirements

---

## 📅 Next Release

**Version:** v1.7.0 (TBD)

**Planned Features:**
- Additional reporting capabilities
- Enhanced dashboard widgets
- More template options
- Performance optimizations

See [backlog](../../../backlog/) for detailed planning.

---

**Release Manager:** [Your Name]  
**Deployment Status:** ✅ Complete  
**Document Version:** 1.0  
**Last Updated:** October 15, 2025
