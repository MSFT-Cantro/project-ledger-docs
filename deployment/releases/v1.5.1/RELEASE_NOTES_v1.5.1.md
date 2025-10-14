# Release Notes - Version 1.5.1

**Release Date**: January 2025  
**Release Type**: Patch Release  
**Deployment Target**: Azure Container Apps (Production)

## Overview
Version 1.5.1 is a patch release that fixes mobile navigation menu synchronization with the desktop menu. This ensures complete feature parity across all devices.

## What's Fixed

### Mobile Navigation Menu Sync
- **Issue**: Mobile menu was missing "Change Orders" and "Reports" navigation items
- **Fix**: Added all missing menu items to mobile navigation drawer
- **Impact**: Users on mobile devices now have access to all features available on desktop

## Changes

### Frontend Updates
- Updated `MobileNavigationDrawer.tsx` component:
  - Added "Change Orders" menu item with submenu (All Change Orders, Create Change Order)
  - Added "Reports" menu item
  - Added "Account Settings" menu item (ADMIN users only)
  - Menu order now matches desktop navigation exactly

## Deployment Information

### Version Numbers
- Root: `1.5.1`
- Frontend: `1.5.1`
- Backend: `1.5.1` (no changes, version bump for consistency)

### Docker Images
- `projectledgerregistry.azurecr.io/projectledger-frontend:v1.5.1`
- `projectledgerregistry.azurecr.io/projectledger-backend:v1.5.1`

### Azure Container Apps
- **Frontend Revision**: `projectledger-frontend--v1-5-1`
- **Backend Revision**: `projectledger-backend--v1-5-1`

## Deployment Steps

### Prerequisites
- Azure CLI logged in with appropriate permissions
- Docker running locally
- Git repository on main branch with latest changes

### Automated Deployment
```bash
cd tools/deployment/releases
bash release-v1.5.1.sh
```

### Manual Deployment (if needed)

1. **Update Version Numbers**
   ```bash
   # Already updated in package.json files to 1.5.1
   git tag v1.5.1
   git push origin v1.5.1
   ```

2. **Build and Push Images**
   ```bash
   cd apps/backend
   docker build --platform linux/amd64 -t projectledgerregistry.azurecr.io/projectledger-backend:v1.5.1 -f Dockerfile .
   docker push projectledgerregistry.azurecr.io/projectledger-backend:v1.5.1
   
   cd ../frontend
   docker build --platform linux/amd64 -t projectledgerregistry.azurecr.io/projectledger-frontend:v1.5.1 -f Dockerfile .
   docker push projectledgerregistry.azurecr.io/projectledger-frontend:v1.5.1
   ```

3. **Deploy to Azure**
   ```bash
   # Backend
   az containerapp update \
     --name projectledger-backend \
     --resource-group projectledger-poc \
     --image projectledgerregistry.azurecr.io/projectledger-backend:v1.5.1 \
     --revision-suffix v1-5-1
   
   # Frontend
   az containerapp update \
     --name projectledger-frontend \
     --resource-group projectledger-poc \
     --image projectledgerregistry.azurecr.io/projectledger-frontend:v1.5.1 \
     --revision-suffix v1-5-1
   ```

4. **Verify Deployment**
   ```bash
   # Check revision status
   az containerapp revision list \
     --name projectledger-frontend \
     --resource-group projectledger-poc \
     --query "[?properties.active].{Name:name, Active:properties.active, Traffic:properties.trafficWeight}" \
     --output table
   
   # Test application
   curl -I https://app.projectledger.ca
   ```

## Verification Checklist

### Frontend Verification
- [ ] Mobile menu opens without errors
- [ ] "Change Orders" menu item visible on mobile
- [ ] Change Orders submenu shows "All Change Orders" and "Create Change Order"
- [ ] "Reports" menu item visible and functional
- [ ] "Account Settings" visible for ADMIN users only
- [ ] Menu order matches desktop navigation
- [ ] All navigation links work correctly
- [ ] Menu styling consistent with existing design

### Backend Verification
- [ ] Backend health endpoint responding (no changes, but verify)
- [ ] API endpoints functional (no changes expected)

### Cross-Device Testing
- [ ] Test on iOS Safari
- [ ] Test on Android Chrome
- [ ] Test on tablet devices
- [ ] Verify desktop menu unchanged

## Rollback Procedure

If issues are discovered, rollback to v1.5.0:

```bash
# Switch traffic back to v1.5.0 revisions
az containerapp revision set-mode \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --mode single \
  --revision projectledger-frontend--v1-5-0

az containerapp revision set-mode \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --mode single \
  --revision projectledger-backend--v1-5-0
```

## Support Information

### Monitoring
- Frontend URL: https://app.projectledger.ca
- Frontend health: `curl -I https://app.projectledger.ca`
- Backend health: Check via frontend API calls

### Known Issues
None at this time.

## Post-Deployment Tasks

1. Monitor application logs for any errors
2. Verify mobile navigation on multiple devices
3. Confirm user access to Change Orders and Reports on mobile
4. Update internal documentation if needed
5. Monitor user feedback channels

## Notes
- This is a frontend-only fix, but backend version was bumped for consistency
- No database migrations required
- No configuration changes needed
- No breaking changes
- Zero-downtime deployment maintained
