# Release History

This folder contains all production release documentation organized by version number. Each release has its own subfolder with complete documentation and deployment scripts.

---

## 📦 Release Structure

Each release folder contains:
- **RELEASE_NOTES_vX.X.X.md** - Comprehensive release documentation including:
  - Features and bug fixes
  - Technical details
  - Deployment commands
  - Verification checklist
  - Rollback procedures
- **DEPLOYMENT_CHECKLIST_vX.X.X.md** - Quick checklist for deployment verification
- **Associated deployment script** - Located in `/tools/deployment/releases/release-vX.X.X.sh`

---

## 📋 Available Releases

### v1.3.0 - October 8, 2025
**Multi-Organization Support + Invoice Enhancement + Documentation Cleanup**

- **[Release Notes](v1.3.0/RELEASE_NOTES_v1.3.0.md)**
- **[Deployment Checklist](v1.3.0/DEPLOYMENT_CHECKLIST_v1.3.0.md)**
- **[Deployment Script](../../../tools/deployment/release-v1.3.0.sh)**

**Key Changes:**
- 🚀 **NEW MAJOR FEATURE**: Multi-Organization Support - Users can belong to multiple organizations and switch between them
- Organization Selector: Full-page selector for multi-org users after login
- Organization Switcher: Navigation dropdown for quick organization switching  
- Enhanced Invoice Creation: Quote item selector for partial invoicing (Bug #30)
- UI Improvement: Removed confusing Beamer default button (Bug #31)
- Documentation Cleanup: Organized specs and comprehensive documentation
- Database Migration: UserAccount table with many-to-many User ↔ Account relationship
- Zero Breaking Changes: Single-org users experience no change in workflow

**Status:** ✅ Successfully Deployed  
**Deployed By:** GitHub Copilot Assistant  
**Deployment Date:** October 8, 2025

---

### v1.2.0 - October 6, 2025
**TextField Focus Fix + Project Detail Accordions**

- **[Release Notes](v1.2.0/RELEASE_NOTES_v1.2.0.md)**
- **[Deployment Checklist](v1.2.0/DEPLOYMENT_CHECKLIST_v1.2.0.md)**
- **[Deployment Script](../../../tools/deployment/releases/release-v1.2.0.sh)**

**Key Changes:**
- Bug #26: Fixed TextField focus indicator overlapping labels in Client/Inventory forms
- Bug #27: Converted Project Detail page from tabs to collapsible accordions
- Mobile-first UX improvements
- Zero downtime deployment

**Status:** ✅ Successfully Deployed  
**Deployed By:** Warren Lovell  
**Deployment Date:** October 6, 2025

---

### v1.1.0 - October 2, 2025
**Beamer Integration + UserMenu Updates**

- **[Release Summary](v1.1.0/RELEASE_SUMMARY_v1.1.0.md)**
- **[Deployment Checklist](v1.1.0/DEPLOYMENT_CHECKLIST_v1.1.0.md)**
- **[Deployment Script](../../../tools/deployment/releases/release-v1.1.0.sh)**

**Key Changes:**
- Integrated Beamer notification system
- Updated UserMenu component
- Removed Profile and Notification from user preferences
- Added Quick Settings functionality

**Status:** ✅ Successfully Deployed  
**Deployment Date:** October 2, 2025

---

## 🔄 Creating a New Release

When creating a new release:

1. **Create version folder:**
   ```bash
   mkdir -p docs/deployment/releases/vX.X.X
   ```

2. **Copy templates:**
   ```bash
   # Copy and customize release notes
   cp docs/deployment/RELEASE_TEMPLATE.md \
      docs/deployment/releases/vX.X.X/RELEASE_NOTES_vX.X.X.md
   
   # Copy and customize checklist
   cp docs/deployment/DEPLOYMENT_CHECKLIST_TEMPLATE.md \
      docs/deployment/releases/vX.X.X/DEPLOYMENT_CHECKLIST_vX.X.X.md
   
   # Copy and customize deployment script
   cp tools/deployment/release-template.sh \
      tools/deployment/releases/release-vX.X.X.sh
   chmod +x tools/deployment/releases/release-vX.X.X.sh
   ```

3. **Update this README** with the new release information

4. **Update main [README.md](../README.md)** to reference the latest release

---

## 📚 Related Documentation

- **[Release Template](../RELEASE_TEMPLATE.md)** - Template for creating new releases
- **[Deployment Checklist Template](../DEPLOYMENT_CHECKLIST_TEMPLATE.md)** - Template for deployment verification
- **[How to Release Guide](../HOW_TO_RELEASE.md)** - Complete release process guide
- **[Deployment Tools](../../../tools/deployment/)** - Deployment scripts and utilities

---

## 📝 Notes

- All releases are tagged in Git (e.g., `v1.2.0`)
- Releases use Azure Container Apps revision-based deployments for zero downtime
- Each release preserves the complete state of documentation at deployment time
- Release scripts are preserved for historical reference and potential rollback scenarios

---

**Last Updated:** October 8, 2025  
**Latest Release:** v1.3.0
