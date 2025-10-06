# Deployment Checklist v1.2.0

**Version:** 1.2.0  
**Date:** October 6, 2025  
**Deployer:** ________________

---

## ✅ Pre-Deployment (5 min)

- [ ] All changes committed and pushed to main
- [ ] Git working directory is clean
- [ ] Docker Desktop is running
- [ ] Logged into Azure CLI (`az login`)
- [ ] Reviewed release notes (RELEASE_NOTES_v1.2.0.md)

---

## 🚀 Deployment (30-40 min)

### Option A: Automated Script (Recommended)
```bash
cd c:/Code/ProjectLedger2
bash tools/deployment/release-v1.2.0.sh
```

### Option B: Manual Steps
- [ ] Create git tag: `git tag v1.2.0 && git push origin v1.2.0`
- [ ] Backup current revisions
- [ ] Login to ACR: `az acr login --name projectledgerregistry`
- [ ] Build backend image
- [ ] Build frontend image
- [ ] Push backend image to ACR
- [ ] Push frontend image to ACR
- [ ] Deploy backend to Azure
- [ ] Wait 15 seconds
- [ ] Deploy frontend to Azure
- [ ] Wait 15 seconds

---

## ✅ Verification (15 min)

### Health Checks
- [ ] Frontend responds: `curl -I https://app.projectledger.ca`
- [ ] Backend health OK: `curl https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io/api/health`

### Bug #26: TextField Focus Indicator
- [ ] Navigate to Clients → Create Client
- [ ] Click "Client Name" field → focus border doesn't overlap label
- [ ] Click "Email Address" field → focus border doesn't overlap label
- [ ] Click "Phone Number" field → focus border doesn't overlap label
- [ ] Test in dark mode
- [ ] Test on mobile viewport

### Bug #27: Project Detail Accordions
- [ ] Open any Project Detail page
- [ ] All 6 sections visible as accordions (not tabs)
- [ ] All sections expanded by default
- [ ] Click "Collapse All" → all sections collapse
- [ ] Click "Expand All" → all sections expand
- [ ] Click individual headers to toggle sections
- [ ] Test Overview section content
- [ ] Test Tasks & Milestones section
- [ ] Test Budget & Finances section
- [ ] Test Quotes section
- [ ] Test Invoices section
- [ ] Test Timeline section
- [ ] Test on mobile viewport
- [ ] Test in dark mode

### General Application
- [ ] User login works
- [ ] Dashboard loads without errors
- [ ] Navigation works (all menu items)
- [ ] No console errors
- [ ] No 404 or 500 errors
- [ ] Page load time < 3 seconds

---

## 📊 Post-Deployment

### First Hour
- [ ] Monitor backend logs (no errors)
- [ ] Monitor frontend logs (no errors)
- [ ] Check Azure Portal for CPU/memory usage
- [ ] Test a few user workflows

### Documentation
- [ ] Update RELEASE_NOTES_v1.2.0.md with completion time
- [ ] Mark deployment status as complete
- [ ] Document any issues encountered
- [ ] Update CHANGELOG.md (if exists)

---

## 🔙 Rollback (if needed)

**Only if critical issues detected:**

1. List revisions:
```bash
az containerapp revision list --name projectledger-backend --resource-group projectledger-poc -o table
az containerapp revision list --name projectledger-frontend --resource-group projectledger-poc -o table
```

2. Activate previous revision (get names from backup files):
```bash
az containerapp revision activate \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --revision [previous-backend-revision-name]

az containerapp revision activate \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --revision [previous-frontend-revision-name]
```

3. Verify rollback worked
4. Document incident and root cause

---

## 📌 Sign-off

**Deployment Status:** ⬜ Not Started | ⏳ In Progress | ✅ Completed | ❌ Failed

**Start Time:** ________________  
**End Time:** ________________  
**Duration:** ________________  

**Issues Encountered:** ________________

**Deployed By:** ________________  
**Sign-off:** ________________

---

## 📞 Emergency Contacts

- **Azure Portal:** https://portal.azure.com
- **GitHub Repository:** https://github.com/MSFT-Cantro/project-ledger
- **Release Notes:** docs/deployment/RELEASE_NOTES_v1.2.0.md
