# Release v1.7.0 - Quotes & Estimates Integration

**Release Date:** October 23, 2025  
**Status:** ✅ Complete

---

## 📦 What's Included

This release represents the complete implementation of the Quotes & Estimates Integration feature, delivered across 7 phases.

### Major Features

#### 1. **Complete Quotes & Estimates System**
- Unified system for managing both quotes and estimates
- Type differentiation with contingency support for estimates
- Quote-to-estimate and estimate-to-quote conversion
- Professional PDF generation for both document types

#### 2. **Client Portal Integration**
- Clients can view estimates in the portal
- Approve/decline workflow for estimates
- Seamless navigation between quotes and estimates

#### 3. **Change Order Support**
- Change orders now support estimates
- Comprehensive change tracking
- Status management and approval workflow

#### 4. **Enhanced Security**
- Authentication required for admin endpoints
- Authentication required for metrics endpoints
- Account-level security for Change Orders API
- Improved error handling for 401/403 responses

---

## 📄 Documentation

- [RELEASE_NOTES_v1.7.0.md](./RELEASE_NOTES_v1.7.0.md) - Complete release notes
- [DEPLOYMENT_CHECKLIST_v1.7.0.md](./DEPLOYMENT_CHECKLIST_v1.7.0.md) - Deployment verification checklist
- [../../HOW_TO_RELEASE.md](../../HOW_TO_RELEASE.md) - Release process guide

---

## 🚀 Deployment

### Deployment Script

The release can be deployed using the automated script:

```bash
bash tools/deployment/release-v1.7.0.sh
```

### Manual Deployment Steps

If you need to deploy manually:

1. **Tag the release:**
   ```bash
   git tag -a v1.7.0 -m "Release v1.7.0: Quotes & Estimates Integration Complete"
   git push origin v1.7.0
   ```

2. **Build and push images:**
   ```bash
   # Backend
   cd apps/backend
   docker build --target production -t projectledgerregistry.azurecr.io/projectledger-backend:v1.7.0 ../..
   docker push projectledgerregistry.azurecr.io/projectledger-backend:v1.7.0
   
   # Frontend
   cd ../frontend
   docker build --target azure -t projectledgerregistry.azurecr.io/projectledger-frontend:v1.7.0 ../..
   docker push projectledgerregistry.azurecr.io/projectledger-frontend:v1.7.0
   ```

3. **Deploy to Azure:**
   ```bash
   # Backend
   az containerapp update \
     --name projectledger-backend \
     --resource-group projectledger-poc \
     --image projectledgerregistry.azurecr.io/projectledger-backend:v1.7.0 \
     --revision-suffix v1-7-0
   
   # Frontend
   az containerapp update \
     --name projectledger-frontend \
     --resource-group projectledger-poc \
     --image projectledgerregistry.azurecr.io/projectledger-frontend:v1.7.0 \
     --revision-suffix v1-7-0
   ```

---

## ✅ Testing Priority

After deployment, focus testing on:

1. **Quotes & Estimates Core Functionality**
   - Create/edit quotes and estimates
   - Type differentiation and filtering
   - Conversion between types
   - Contingency support for estimates

2. **PDF Generation**
   - Generate PDFs for quotes
   - Generate PDFs for estimates
   - Verify formatting and content

3. **Client Portal**
   - View estimates in portal
   - Approve/decline functionality
   - Navigation between documents

4. **Change Orders**
   - Create change orders for estimates
   - Status tracking
   - Total calculations

5. **Security**
   - Admin endpoint authentication
   - Metrics endpoint authentication
   - Account boundary enforcement

---

## 🔄 Rollback

If issues are encountered, rollback to v1.6.0:

```bash
# Backend
az containerapp revision activate \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --revision projectledger-backend--v1-6-0

# Frontend
az containerapp revision activate \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --revision projectledger-frontend--v1-6-0
```

---

## 📊 Implementation Phases

This release was delivered across 7 phases:

- **Phase 1:** Database schema and core logic
- **Phase 2:** API endpoints and validation
- **Phase 3:** Frontend form UI
- **Phase 4:** List views, filtering, and conversion
- **Phase 5:** PDF generation
- **Phase 6:** Change order integration
- **Phase 7:** Client portal support

Each phase was thoroughly tested and documented before moving to the next.

---

## 🔗 Related Issues

- Item #1-33: Various UI and functionality improvements (all completed)
- Security enhancements for admin and metrics endpoints
- Enhanced error handling across the application

---

## 👥 Contributors

- Development Team
- QA Team
- Product Management

---

**Previous Release:** [v1.6.0](../v1.6.0/README.md)  
**Next Release:** v1.8.0 (TBD)
