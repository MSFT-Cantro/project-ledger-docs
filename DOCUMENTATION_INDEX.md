# 📚 Project Ledger Documentation Index

**Last Updated:** October 1, 2025  
**Location:** `docs/DOCUMENTATION_INDEX.md`

---

## 🚀 Quick Start

Start here if you're new to the project:

- **[QUICK_START.md](guides/QUICK_START.md)** - Local development and Azure testing setup
  - How to start services
  - Access points and URLs
  - Common commands
  - Troubleshooting guide

---

## 🔧 Technical Fixes & Solutions

### Frontend-Backend Communication
- **[AZURE_LOCAL_TEST_FIX_SUMMARY.md](fixes/AZURE_LOCAL_TEST_FIX_SUMMARY.md)** - Complete fix summary
  - All 4 issues resolved (SubscriptionService, RecurringInvoiceService, SubscriptionContext, Database)
  - Technical details for each fix
  - Before/After comparisons
  - Verification steps

- **[FRONTEND_API_FIX_COMPLETE.md](fixes/FRONTEND_API_FIX_COMPLETE.md)** - Detailed API configuration
  - axiosInstance vs raw axios explanation
  - Request flow diagrams
  - Complete troubleshooting guide
  - Deployment checklist

### Business Logic Fixes
- **[TAX_ISSUE_RESOLUTION.md](fixes/TAX_ISSUE_RESOLUTION.md)** - Tax calculation fixes
  - Canadian tax calculation issues
  - Company location validation
  - Tax service improvements

### Documentation Maintenance
- **[CLEANUP_SUMMARY.md](fixes/CLEANUP_SUMMARY.md)** - Documentation cleanup record
  - Files moved and organized
  - Cleanup process
  - Maintenance guidelines

---

## 📖 Project Documentation

### Architecture & Planning
Located in `docs/` directory:

- **[architecture.md](docs/Completed/architecture.md)** - System architecture overview
- **[BRANDING_APPLICATION_PLAN.md](docs/Completed/BRANDING_APPLICATION_PLAN.md)** - Brand guidelines
- **[ACCESSIBILITY_MOBILE_IMPROVEMENTS.md](docs/Completed/ACCESSIBILITY_MOBILE_IMPROVEMENTS.md)** - UX improvements

### Feature Specifications
- **[SPEC_payment-integration.md](docs/SPEC_payment-integration.md)** - Payment system design
- **[SPEC_pricing-plan-integration.md](docs/SPEC_pricing-plan-integration.md)** - Subscription plans
- **[SPEC_ReportingFunctionality.md](docs/SPEC_ReportingFunctionality.md)** - Reporting features
- **[SPEC_tax-configuration.md](docs/Completed/SPEC_tax-configuration.md)** - Tax system design

### Deployment & Operations
- **[AZURE_DEPLOYMENT_PLAN.md](docs/AZURE_DEPLOYMENT_PLAN.md)** - Azure deployment strategy
- **[PRODUCTION_DEPLOYMENT_PAYPAL.md](docs/PRODUCTION_DEPLOYMENT_PAYPAL.md)** - PayPal production setup
- **[SECURITY_AUDIT_PAYPAL.md](docs/SECURITY_AUDIT_PAYPAL.md)** - Security considerations
- **[MONITORING_ALERTING_PAYPAL.md](docs/MONITORING_ALERTING_PAYPAL.md)** - Monitoring setup

### OAuth & Authentication
- **[SPEC_OAUTH_SETUP.md](docs/SPEC_OAUTH_SETUP.md)** - OAuth integration guide
- **[SPEC_ACCOUNT_SUBSCRIPTION_MIGRATION.md](docs/SPEC_ACCOUNT_SUBSCRIPTION_MIGRATION.md)** - Account system

---

## 🛠️ Tools & Scripts

### Testing Tools
Located in `tools/testing/`:
- `validate-azure-deployment.sh` - Comprehensive deployment validation
- `test-azure-local.sh` - Local Azure simulation
- `quick-test.sh` - Fast health checks

### Deployment Tools
Located in `tools/deployment/`:
- `deploy-azure-infrastructure.sh` - Azure resource deployment
- `deploy-containers-poc.sh` - Container deployment
- `test-deployment.sh` - Post-deployment validation

### Utility Scripts
Located in `tools/utilities/`:
- Database management scripts
- Configuration helpers
- Temporary/test scripts in `temp-scripts/`

**See [tools/README.md](tools/README.md) for detailed tool documentation**

---

## 🐛 Issue Tracking

### Known Issues
- **[_bugs.md](docs/_bugs.md)** - Current bug list and status

### Feature Requests
- **[_features.md](docs/_features.md)** - Planned features and enhancements

### Technical Debt
- **[_techdebt.md](docs/_techdebt.md)** - Areas needing refactoring

---

## 📋 Development Prompts

- **[__prompts.md](docs/__prompts.md)** - Common development tasks and prompts

---

## 🏗️ Project Structure

```
ProjectLedger2/
├── apps/
│   ├── backend/              # Node.js/Express API
│   └── frontend/             # React application
├── packages/
│   └── shared-types/         # Shared TypeScript types
├── docs/                     # All documentation
│   ├── Completed/           # Completed specs
│   ├── SPEC_*.md            # Feature specifications
│   ├── _bugs.md             # Issue tracking
│   ├── _features.md         # Feature requests
│   └── _techdebt.md         # Technical debt
├── tools/                    # Automation scripts
│   ├── testing/             # Test utilities
│   ├── deployment/          # Deploy scripts
│   ├── monitoring/          # Monitoring tools
│   └── utilities/           # Helper scripts
├── QUICK_START.md           # ⭐ Start here
├── AZURE_LOCAL_TEST_FIX_SUMMARY.md  # Fix documentation
├── FRONTEND_API_FIX_COMPLETE.md     # API configuration
└── TAX_ISSUE_RESOLUTION.md          # Tax fixes
```

---

## 🔍 Finding What You Need

### "I want to..."

**...set up the project locally**
→ [QUICK_START.md](QUICK_START.md)

**...understand the recent fixes**
→ [AZURE_LOCAL_TEST_FIX_SUMMARY.md](AZURE_LOCAL_TEST_FIX_SUMMARY.md)

**...troubleshoot API issues**
→ [FRONTEND_API_FIX_COMPLETE.md](FRONTEND_API_FIX_COMPLETE.md)

**...deploy to Azure**
→ [docs/AZURE_DEPLOYMENT_PLAN.md](docs/AZURE_DEPLOYMENT_PLAN.md)

**...understand the architecture**
→ [docs/Completed/architecture.md](docs/Completed/architecture.md)

**...add a new feature**
→ [docs/SPEC_*.md](docs/) + [tools/README.md](tools/README.md)

**...fix a tax calculation bug**
→ [TAX_ISSUE_RESOLUTION.md](TAX_ISSUE_RESOLUTION.md)

**...set up PayPal**
→ [docs/PRODUCTION_DEPLOYMENT_PAYPAL.md](docs/PRODUCTION_DEPLOYMENT_PAYPAL.md)

---

## ✅ Status Summary

### ✅ Working & Tested
- Frontend production build
- Backend API with all fixes applied
- Database with migrations and seeding
- Docker containerization
- Nginx proxy configuration
- Local Azure simulation environment
- PayPal integration (sandbox mode)

### ⚠️ Needs Configuration
- OAuth providers (Google, Microsoft, GitHub)
- Stripe payment integration
- Production PayPal credentials
- Azure deployment resources

### 📝 Documentation Status
- ✅ Quick start guide complete
- ✅ Fix documentation complete
- ✅ Architecture documented
- ✅ API configuration documented
- ✅ Deployment plan documented
- ⚠️ API endpoint documentation needs updating

---

## 🤝 Contributing

When adding new documentation:

1. **Create files in appropriate locations:**
   - Technical fixes: Root level (`*_FIX.md`, `*_RESOLUTION.md`)
   - Specifications: `docs/SPEC_*.md`
   - Completed work: `docs/Completed/`
   - Issue tracking: `docs/_bugs.md`, `docs/_features.md`, `docs/_techdebt.md`

2. **Update this index** with links to new documentation

3. **Follow naming conventions:**
   - Fixes: `{ISSUE}_FIX.md` or `{ISSUE}_RESOLUTION.md`
   - Specs: `SPEC_{feature}.md`
   - Plans: `{SERVICE}_PLAN.md` or `{SERVICE}_DEPLOYMENT.md`

4. **Include in README.md** support section if it's a major reference

---

## 📞 Need Help?

Can't find what you're looking for?

1. Check this index
2. Search within documentation files
3. Check [tools/README.md](tools/README.md) for scripts
4. Review [README.md](README.md) for project overview
5. Create an issue if documentation is missing

---

**Last Review:** October 1, 2025  
**Maintained By:** Development Team
