# 🚀 Deployment Documentation

This folder contains all documentation related to deploying and managing the ProjectLedger application on Azure.

---

## 📑 Quick Reference

### **⭐ Release Templates** (Start Here for New Releases)
- **[DEPLOYMENT_CHECKLIST_TEMPLATE.md](DEPLOYMENT_CHECKLIST_TEMPLATE.md)** - ✅ Printable checklist for any release
- **[MARKETING_POST_TEMPLATE.md](MARKETING_POST_TEMPLATE.md)** - 📣 Social media announcement template

### **Getting Started**
- **[HOW_TO_RELEASE.md](HOW_TO_RELEASE.md)** - 🎯 Complete guide for releasing new versions
- **[DEPLOYMENT_PLAN.md](DEPLOYMENT_PLAN.md)** - Complete deployment strategy with zero-downtime updates

### **Release History**
- **[releases/](releases/)** - 📦 Archived releases with documentation and scripts
  - **[v1.2.0/](releases/v1.2.0/)** - TextField focus fix & Project Detail accordions (October 6, 2025)
  - **[v1.1.0/](releases/v1.1.0/)** - Beamer Integration + UserMenu Updates (October 2, 2025)

### **Azure Deployment**
- **[Azure Deployment Archive](archive/)** - Historical Azure deployment documentation

### **Production & Security**
- See archived documentation for production deployment details

---

## 🎯 Common Tasks

### **Initial Deployment**
1. Review infrastructure setup documentation (see archive folder for historical reference)
2. **IMPORTANT:** Seed the database after deployment:
   ```bash
   # Seed subscription plans (REQUIRED - signup won't work without this!)
   cd apps/backend
   DATABASE_URL="postgresql://postgres:PASSWORD@projectledger-db.eastus2.azurecontainer.io:5432/projectledger" \
   npx ts-node prisma/seed-subscriptions.ts
   
   # Seed Canadian tax rates
   DATABASE_URL="postgresql://postgres:PASSWORD@projectledger-db.eastus2.azurecontainer.io:5432/projectledger" \
   node seed-canada.js
   ```
3. Configure custom domain settings
4. Review security audit procedures before going live

### **Production Releases**

**For a New Release:**
1. Copy templates:
   ```bash
   # Copy release template
   cp docs/deployment/RELEASE_TEMPLATE.md docs/deployment/releases/v1.X.X/RELEASE_NOTES_v1.X.X.md
   
   # Copy checklist template
   cp docs/deployment/DEPLOYMENT_CHECKLIST_TEMPLATE.md docs/deployment/releases/v1.X.X/DEPLOYMENT_CHECKLIST_v1.X.X.md
   
   # Copy script template
   cp tools/deployment/release-template.sh tools/deployment/releases/release-v1.X.X.sh
   chmod +x tools/deployment/releases/release-v1.X.X.sh
   ```

2. Update all three files with:
   - Version number (e.g., 1.X.X)
   - Release-specific features/changes
   - Any custom deployment steps

3. Run the release script:
   ```bash
   bash tools/deployment/releases/release-v1.X.X.sh
   ```

4. Follow verification checklist
5. Monitor application health and performance

**Example Releases:** 
- See [releases/v1.2.0/](releases/v1.2.0/) for latest release
- See [releases/v1.1.0/](releases/v1.1.0/) for previous release

### **Daily Updates**
1. Use the deployment scripts in `/tools/deployment/`
2. Follow the zero-downtime strategy in [DEPLOYMENT_PLAN.md](DEPLOYMENT_PLAN.md)
3. Monitor application health and performance

### **Troubleshooting**
1. Check [AZURE_LOCAL_TEST_FIX_SUMMARY.md](archive/AZURE_LOCAL_TEST_FIX_SUMMARY.md) for local testing (archived)
2. Review container logs using Azure CLI commands
3. Consult rollback procedures in [DEPLOYMENT_PLAN.md](DEPLOYMENT_PLAN.md)

---

## 🏗️ Infrastructure Overview

### **Current Production Setup**
- **Platform:** Azure Container Apps
- **Domain:** app.projectledger.ca
- **Static IP:** 20.1.213.250
- **Database:** PostgreSQL on Azure Container Instance
- **Registry:** Azure Container Registry (projectledgerregistry)

### **Key Services**
- **Frontend:** projectledger-frontend (React + Nginx)
- **Backend:** projectledger-backend (Node.js + Prisma)
- **Database:** projectledger-db (PostgreSQL 16)

---

## 📚 Document Categories

### **Active Templates & Guides**
- RELEASE_TEMPLATE.md
- DEPLOYMENT_CHECKLIST_TEMPLATE.md
- HOW_TO_RELEASE.md
- DEPLOYMENT_PLAN.md

### **Release History**
- **[releases/](releases/)** - All production releases organized by version
  - **[v1.2.0/](releases/v1.2.0/)** - Latest: TextField fixes & Project accordions (Oct 6, 2025)
  - **[v1.1.0/](releases/v1.1.0/)** - Beamer Integration + UserMenu Updates (Oct 2, 2025)

### **Infrastructure & Configuration**
- See [archive/](archive/) for historical infrastructure documentation

### **Production Operations**
- Deployment scripts located in app repository
- Monitoring and security procedures documented in app repository

### **Archived Documentation**
- **[archive/README.md](archive/README.md)** - Historical documents preserved for reference

---

## 🔗 Related Documentation

- **Deployment Scripts:** `/tools/deployment/`
  - **Release Scripts:** `/tools/deployment/releases/` - Version-specific deployment scripts
- **Azure Scripts:** `/tools/azure/`
- **Quick Start:** `/NEXT_STEPS.md` (root)
- **Custom Domain Setup:** `/CUSTOM_DOMAIN_SETUP.md` (root)

---

## 📝 Notes

- All Azure Container Apps use revision-based deployments for zero downtime
- Database persists across all deployments (never redeployed)
- Static IP (20.1.213.250) is permanent for the Container Apps Environment
- SSL certificates are automatically managed by Azure
- Release documentation is organized in `releases/` folder by version number

---

**Last Updated:** October 6, 2025  
**Latest Release:** v1.2.0 (TextField Focus Fix + Project Detail Accordions)  
**Templates Version:** 1.0

---

**🧹 Recent Updates (October 6, 2025):** 
- Reorganized release documentation into `releases/` folder for better organization
- v1.2.0 deployed: TextField focus indicators and Project Detail accordion conversion
- All release-specific files now stored in version-specific subfolders

