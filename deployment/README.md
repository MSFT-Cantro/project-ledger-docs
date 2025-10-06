# 🚀 Deployment Documentation

This folder contains all documentation related to deploying and managing the ProjectLedger application on Azure.

---

## 📑 Quick Reference

### **⭐ Release Templates** (Start Here for New Releases)
- **[RELEASE_TEMPLATE.md](RELEASE_TEMPLATE.md)** - 🎯 Standard template for all production releases
- **[DEPLOYMENT_CHECKLIST_TEMPLATE.md](DEPLOYMENT_CHECKLIST_TEMPLATE.md)** - ✅ Printable checklist for any release
- **[release-template.sh](../../tools/deployment/release-template.sh)** - 🤖 Automated deployment script template

### **Getting Started**
- **[HOW_TO_RELEASE.md](HOW_TO_RELEASE.md)** - 🎯 Complete guide for releasing new versions
- **[DEPLOYMENT_PLAN.md](DEPLOYMENT_PLAN.md)** - Complete deployment strategy with zero-downtime updates
- **[DATABASE_SEEDING.md](DATABASE_SEEDING.md)** - ⚠️ CRITICAL: Database seeding guide (required after initial deployment)

### **Azure Deployment**
- **[AZURE_DEPLOYMENT_COMPLETE.md](AZURE_DEPLOYMENT_COMPLETE.md)** - ✅ Complete Azure deployment details and current configuration

### **Custom Domain Setup**
- **[GODADDY_DNS_SETUP.md](GODADDY_DNS_SETUP.md)** - Complete DNS configuration guide for GoDaddy

### **Production & Security**
- **[PRODUCTION_DEPLOYMENT_PAYPAL.md](PRODUCTION_DEPLOYMENT_PAYPAL.md)** - PayPal integration for production
- **[SECURITY_AUDIT_PAYPAL.md](SECURITY_AUDIT_PAYPAL.md)** - Security audit checklist for PayPal integration
- **[MONITORING_ALERTING_PAYPAL.md](MONITORING_ALERTING_PAYPAL.md)** - Monitoring and alerting setup

---

## 🎯 Common Tasks

### **Initial Deployment**
1. Read [AZURE_DEPLOYMENT_COMPLETE.md](AZURE_DEPLOYMENT_COMPLETE.md) for infrastructure setup
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
3. Follow [GODADDY_DNS_SETUP.md](GODADDY_DNS_SETUP.md) for custom domain
4. Review [SECURITY_AUDIT_PAYPAL.md](SECURITY_AUDIT_PAYPAL.md) before going live

### **Production Releases**

**For a New Release:**
1. Copy templates:
   ```bash
   # Copy release template
   cp docs/deployment/RELEASE_TEMPLATE.md docs/deployment/RELEASE_NOTES_v1.2.0.md
   
   # Copy checklist template
   cp docs/deployment/DEPLOYMENT_CHECKLIST_TEMPLATE.md docs/deployment/DEPLOYMENT_CHECKLIST_v1.2.0.md
   
   # Copy script template
   cp tools/deployment/release-template.sh tools/deployment/release-v1.2.0.sh
   chmod +x tools/deployment/release-v1.2.0.sh
   ```

2. Update all three files with:
   - Version number (e.g., 1.2.0)
   - Release-specific features/changes
   - Any custom deployment steps

3. Run the release script:
   ```bash
   bash tools/deployment/release-v1.2.0.sh
   ```

4. Follow verification checklist
5. Monitor using [MONITORING_ALERTING_PAYPAL.md](MONITORING_ALERTING_PAYPAL.md)

**Example Release:** See [PRODUCTION_RELEASE_PLAN.md](PRODUCTION_RELEASE_PLAN.md) for v1.1.0 as reference

### **Daily Updates**
1. Use the deployment scripts in `/tools/deployment/`
2. Follow the zero-downtime strategy in [DEPLOYMENT_PLAN.md](DEPLOYMENT_PLAN.md)
3. Monitor using [MONITORING_ALERTING_PAYPAL.md](MONITORING_ALERTING_PAYPAL.md)

### **Troubleshooting**
1. Check [AZURE_LOCAL_TEST_FIX_SUMMARY.md](AZURE_LOCAL_TEST_FIX_SUMMARY.md) for local testing
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

### **Current Release Documentation**
- RELEASE_SUMMARY_v1.1.0.md
- DEPLOYMENT_CHECKLIST_v1.1.0.md

### **Infrastructure & Configuration**
- AZURE_DEPLOYMENT_COMPLETE.md
- GODADDY_DNS_SETUP.md
- DATABASE_SEEDING.md

### **Production Operations**
- PRODUCTION_DEPLOYMENT_PAYPAL.md
- MONITORING_ALERTING_PAYPAL.md
- SECURITY_AUDIT_PAYPAL.md

### **Archived Documentation**
- **[archive/README.md](archive/README.md)** - Historical documents preserved for reference

---

## 🔗 Related Documentation

- **Deployment Scripts:** `/tools/deployment/`
- **Azure Scripts:** `/tools/azure/`
- **Quick Start:** `/NEXT_STEPS.md` (root)
- **Custom Domain Setup:** `/CUSTOM_DOMAIN_SETUP.md` (root)

---

## 📝 Notes

- All Azure Container Apps use revision-based deployments for zero downtime
- Database persists across all deployments (never redeployed)
- Static IP (20.1.213.250) is permanent for the Container Apps Environment
- SSL certificates are automatically managed by Azure

---

**Last Updated:** October 6, 2025  
**Latest Release:** v1.1.0 (Beamer Integration + UserMenu Updates)  
**Templates Version:** 1.0

---

**��� Recent Cleanup (October 6, 2025):** Archived 6 outdated documents to `archive/` folder for historical reference.
