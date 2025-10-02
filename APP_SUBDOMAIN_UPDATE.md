# 🔄 Update to app.projectledger.ca - Implementation Plan

**Date:** October 2, 2025  
**Change:** Update domain from `projectledger.ca` to `app.projectledger.ca`

---

## 📋 Overview

The application will be accessed via `app.projectledger.ca` subdomain instead of the root domain. This is a better practice for web applications and allows flexibility for other services on the root domain.

---

## 🌐 DNS Changes Required in GoDaddy

### **Current DNS Records (Remove/Update):**
```
Type: A
Name: @
Value: 20.1.213.250
❌ DELETE (or keep for landing page)
```

```
Type: A
Name: www
Value: 20.1.213.250
❌ DELETE (or keep for landing page)
```

### **New DNS Records (Add):**

#### **1. A Record for app subdomain:**
```
Type: A
Name: app
Value: 20.1.213.250
TTL: 600
✅ ADD THIS
```

#### **2. TXT Record for domain verification:**
```
Type: TXT
Name: asuid.app.projectledger.ca
Value: C5DEF9282B87920374F788BD02411986D1F0299CD5705064A504EB3C778958F4
TTL: 600
✅ ADD THIS
```

### **Optional - Redirect root to app:**
If you want `projectledger.ca` to redirect to `app.projectledger.ca`:

**Option A: Using GoDaddy Forwarding**
1. In GoDaddy, go to Domain Settings
2. Set up forwarding: `projectledger.ca` → `https://app.projectledger.ca`
3. Enable "Forward with masking" if desired

**Option B: Using Azure Static Web App (Future)**
- Host a simple landing/marketing page on root domain
- Link to `app.projectledger.ca` for the application

---

## 🔧 Azure Configuration Changes

### **1. Update Backend Environment Variable**

```bash
az containerapp update \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --set-env-vars "FRONTEND_URL=https://app.projectledger.ca"
```

### **2. Add Custom Domain to Frontend Container App**

After DNS propagation (10-15 minutes):

```bash
# Add app.projectledger.ca
az containerapp hostname add \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --hostname app.projectledger.ca

# Enable SSL certificate
az containerapp hostname bind \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --hostname app.projectledger.ca \
  --environment projectledger-env \
  --validation-method HTTP
```

---

## 📝 Code/Configuration Changes

### **Files to Update:**

1. **Nginx Configuration** - `apps/frontend/nginx.azure.conf`
   - Update `server_name` directive
   - Update CSP headers if domain-specific

2. **Backend CORS Configuration** - Check if hardcoded
   - Should use `FRONTEND_URL` environment variable

3. **Documentation Files:**
   - CUSTOM_DOMAIN_SETUP.md
   - docs/GODADDY_DNS_SETUP.md
   - docs/DEPLOYMENT_PLAN.md
   - docs/fixes/AZURE_DEPLOYMENT_COMPLETE.md
   - tools/deployment/deploy-update.sh
   - tools/deployment/rollback.sh

---

## ✅ Implementation Steps

### **Step 1: Update DNS in GoDaddy** (10 minutes)
1. Login to GoDaddy
2. Go to DNS Management for projectledger.ca
3. Add A record: `app` → `20.1.213.250`
4. Add TXT record: `asuid.app.projectledger.ca` → verification ID
5. Wait 10-15 minutes for propagation

### **Step 2: Verify DNS Propagation** (5 minutes)
```bash
# Check if DNS is resolving
nslookup app.projectledger.ca

# Check TXT record
nslookup -type=TXT asuid.app.projectledger.ca

# Use online tool
# https://www.whatsmydns.net/#A/app.projectledger.ca
```

### **Step 3: Update Azure Custom Domain** (10 minutes)
```bash
# Add custom domain
az containerapp hostname add \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --hostname app.projectledger.ca

# Bind SSL certificate (wait 10-15 min for provisioning)
az containerapp hostname bind \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --hostname app.projectledger.ca \
  --environment projectledger-env \
  --validation-method HTTP
```

### **Step 4: Update Backend Configuration** (2 minutes)
```bash
# Update frontend URL environment variable
az containerapp update \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --set-env-vars "FRONTEND_URL=https://app.projectledger.ca"
```

### **Step 5: Update Code and Documentation** (10 minutes)
- Update Nginx configuration
- Update all documentation references
- Commit and push changes

### **Step 6: Test Application** (5 minutes)
```bash
# Test app access
curl -I https://app.projectledger.ca

# Test backend health
curl https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io/api/health

# Open in browser
# https://app.projectledger.ca
```

---

## 🧪 Testing Checklist

After implementation, verify:

- [ ] `app.projectledger.ca` resolves to `20.1.213.250`
- [ ] HTTPS works with valid SSL certificate (green padlock)
- [ ] Application loads correctly
- [ ] Login functionality works
- [ ] API calls succeed
- [ ] No CORS errors in browser console
- [ ] Backend receives correct origin headers

---

## 🔙 Rollback Plan

If issues occur:

### **Quick Rollback - Keep Old Domain Working:**
The old Azure Container Apps URLs will continue to work:
- Frontend: `projectledger-frontend.orangeplant-da913b57.eastus2.azurecontainerapps.io`
- Backend: `projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io`

### **Revert Custom Domain:**
```bash
# Remove custom domain if needed
az containerapp hostname delete \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --hostname app.projectledger.ca

# Revert backend URL
az containerapp update \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --set-env-vars "FRONTEND_URL=https://projectledger-frontend.orangeplant-da913b57.eastus2.azurecontainerapps.io"
```

---

## 📊 DNS Records Summary

After implementation, your GoDaddy DNS should look like:

| Type | Name | Value | Purpose |
|------|------|-------|---------|
| A | app | 20.1.213.250 | Application access |
| TXT | asuid.app.projectledger.ca | C5DEF9282B87920374F788BD02411986D1F0299CD5705064A504EB3C778958F4 | Domain verification |
| CNAME | @ | (optional - for landing page) | Root domain |

---

## ⏱️ Timeline

| Step | Time | Wait Time |
|------|------|-----------|
| Update DNS | 5 min | 10-15 min |
| Verify DNS | 5 min | - |
| Add to Azure | 5 min | - |
| Enable SSL | 2 min | 10-20 min |
| Update Backend | 2 min | - |
| Update Code | 10 min | - |
| Test | 5 min | - |
| **Total** | **~35 min** | **~30 min wait** |

---

## 📞 Support

If you encounter issues:
- Check DNS propagation: https://www.whatsmydns.net
- Verify certificate status in Azure Portal
- Review container logs: `az containerapp logs show --name projectledger-frontend --resource-group projectledger-poc`

---

**Ready to implement? Start with Step 1: Update DNS in GoDaddy!**
