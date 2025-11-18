`````markdown
````markdown
# Release Notes - Version 1.8.0

> **Version:** v1.8.0  
> **Release Date:** November 18, 2025  
> **Deployer:** [Your Name]  
> **Revision Suffix:** v1-8-0

---

## 📋 Release Information

**Changes in this release:**

### ✨ Features & Improvements

#### Design System Migration (Phase 1)
- **Complete design system implementation** with reusable components:
  - AdminCard, EnhancedTable, StatusBadge, FormSection
  - TextField, SelectField, LoadingOverlay, TabsContainer
  - Consistent styling and theming across admin interface
- **Migrated all admin pages** to design system:
  - AdminLoginPage, AdminDashboardPage
  - AdminAccountsPage, AdminAccountDetailPage, AdminAccountEditPage
  - AdminUsersPage, AdminUserDetailPage
  - AdminSettingsPage
- **Migrated all portal pages** to design system:
  - PortalDashboardPage, PortalProjectsPage, PortalProjectDetailPage
  - PortalInvoicesPage, PortalInvoiceDetailPage
  - PortalQuoteDetailPage, PortalChangeOrderDetailPage
  - Consistent Grid, Pagination, and component usage

#### Internal Admin Dashboard
- **Phase 1: Foundation & Authentication System**
  - Internal admin authentication and role-based access
  - Secure admin portal separate from customer accounts
  - Internal user management
- **Phase 2: Account & User Management Dashboard**
  - Complete account management interface
  - User management with actions & security controls
  - Enhanced metrics and analytics for system administrators

#### Client Portal Enhancements
- **Portal design system implementation** with accessibility
- **Responsive layouts** for all portal pages
- **User menu and branded login page**
- **Fixed mobile authentication** issues
- **Consistent MetricCard statistics** across portal pages
- **Improved list views** with BrandedDataTable component
- **Standardized CSS Grid layouts** for statistics cards

#### Storybook Integration
- **Comprehensive Storybook design system** documentation
- **Enhanced Storybook setup** with dark theme support
- **Component library** for development and testing

#### Deployment & Operations
- **Discord release notifications** for production deployments
- **Portal deployment** added to release scripts
- **Automated notification system** for deployment phases

#### Analytics & Reporting
- **Comprehensive reporting & analytics** for quotes and estimates
- **Enhanced charts** with ComposedChart for mixed visualizations

### 🐛 Bug Fixes
- Fixed BigInt to Number conversion in system stats endpoint for JSON serialization
- Fixed ComposedChart implementation for mixed Bar/Line charts in analytics
- Fixed React Hook order violations (error #310) across multiple components
- Fixed EnhancedTable visibility and hover states for dark mode
- Fixed TabsContainer bounds checking and validation
- Fixed render function signatures in EnhancedTable components
- Added user role types (ADMIN, SUPERADMIN, USER, OWNER, VIEWER) to StatusBadge
- Removed deprecated contactInfo field from Client schema
- Fixed Node.js memory allocation for frontend builds
- Fixed missing status property in CustomerUser mappings
- Fixed statistics cards sizing consistency across portal pages
- Fixed database seeding with comprehensive business document support

### 🔒 Security Enhancements
- Internal admin authentication system with role-based access control
- Enhanced security controls for user management operations
- Improved account-level security boundaries

### 📚 Documentation
- Moved documentation to separate repository for better organization
- Updated README with documentation repository links
- Moved completed specs to Completed folder
- Added Storybook integration specification
- Removed obsolete development documentation files

---

## 🚀 Deployment Details

**Git Tag:** v1.8.0  
**Previous Version:** v1.7.0  
**Branch:** main

**Docker Images:**
- Backend: `projectledgerregistry.azurecr.io/projectledger-backend:v1.8.0`
- Frontend: `projectledgerregistry.azurecr.io/projectledger-frontend:v1.8.0`
- Portal: `projectledgerregistry.azurecr.io/projectledger-portal:v1.8.0`

**Azure Revisions:**
- Backend: `projectledger-backend--v1-8-0`
- Frontend: `projectledger-frontend--v1-8-0`
- Portal: `projectledger-portal--v1-8-0`

---

## ✅ Testing Checklist

### Core Functionality
- [ ] Login/logout works
- [ ] Dashboard loads correctly
- [ ] Navigation is functional
- [ ] Mobile view works

### New Features to Test
- [ ] Design system components render correctly across all pages
- [ ] Internal admin dashboard accessible and functional
- [ ] Admin authentication and role-based access working
- [ ] Account management operations work correctly
- [ ] User management features functional
- [ ] Portal pages display correctly with new design system
- [ ] Statistics cards display consistently
- [ ] Mobile portal authentication works
- [ ] Discord notifications sent for deployments
- [ ] Analytics charts render correctly
- [ ] Dark mode theming works across all components
- [ ] Storybook documentation accessible

### Integration Points
- [ ] API calls successful
- [ ] Authentication working
- [ ] Database connections stable
- [ ] External integrations functional

### Security Testing
- [ ] Authentication endpoints protected
- [ ] Authorization working correctly
- [ ] Proper error handling (401/403)

---

## 📊 Deployment Timeline

| Phase | Time | Status |
|-------|------|--------|
| Pre-deployment checks | [Time] | ⏳ |
| Docker image build | [Time] | ⏳ |
| Image push to ACR | [Time] | ⏳ |
| Backend deployment | [Time] | ⏳ |
| Frontend deployment | [Time] | ⏳ |
| Portal deployment | [Time] | ⏳ |
| Verification | [Time] | ⏳ |
| Post-deployment monitoring | [Time] | ⏳ |

**Total Duration:** [Duration]

---

## 🔍 Verification Results

### Automated Checks
- [ ] Frontend health check (200 OK)
- [ ] Backend health check (200 OK)
- [ ] Portal health check (200 OK)
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
- [DEPLOYMENT_CHECKLIST_v1.8.0.md](./DEPLOYMENT_CHECKLIST_v1.8.0.md)
- [DEPLOYMENT_PLAN.md](../../DEPLOYMENT_PLAN.md)

---

## 📞 Rollback Information

**Previous Revision:**
- Backend: `projectledger-backend--v1-7-0`
- Frontend: `projectledger-frontend--v1-7-0`
- Portal: `projectledger-portal--v1-7-0`

**Rollback Command:**
```bash
# Backend rollback
az containerapp revision activate \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --revision projectledger-backend--v1-7-0

# Frontend rollback
az containerapp revision activate \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --revision projectledger-frontend--v1-7-0

# Portal rollback
az containerapp revision activate \
  --name projectledger-portal \
  --resource-group projectledger-poc \
  --revision projectledger-portal--v1-7-0
```

---

## ✍️ Sign-off

**Deployment Status:** ☐ Success ☐ Partial ☐ Failed

**Deployed By:** ___________________________  
**Date & Time:** ___________________________  

**Verified By:** ___________________________  
**Date & Time:** ___________________________  

---

**Next Release:** v1.9.0 (TBD)

````

`````
