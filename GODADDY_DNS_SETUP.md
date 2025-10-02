# 🌐 GoDaddy DNS Setup Guide for projectledger.ca

**Date:** October 2, 2025  
**Domain:** projectledger.ca  
**Azure Static IP:** 20.1.213.250

---

## 📋 Overview

This guide will help you configure your GoDaddy domain `projectledger.ca` to point to your Azure Container Apps deployment.

---

## Step 1: Configure DNS Records in GoDaddy

### **Login to GoDaddy**
1. Go to https://www.godaddy.com
2. Login to your account
3. Navigate to **My Products** → **Domains**
4. Click **DNS** next to `projectledger.ca`

### **Add/Update DNS Records**

You need to configure the following DNS records:

#### **A Record (Root Domain)**
Configure the root domain (@) to point to Azure:

```
Type: A
Name: @
Value: 20.1.213.250
TTL: 600 seconds (10 minutes)
```

#### **A Record (www subdomain)**
Configure www subdomain:

```
Type: A
Name: www
Value: 20.1.213.250
TTL: 600 seconds (10 minutes)
```

#### **TXT Record (Domain Verification) - Required**
Azure needs to verify domain ownership. We'll add this in Step 2.

```
Type: TXT
Name: asuid.projectledger.ca
Value: [Will be provided by Azure in Step 2]
TTL: 600 seconds (10 minutes)
```

### **Remove Conflicting Records**
- Delete any existing CNAME records for @ or www that point elsewhere
- Delete any existing A records that conflict with the new ones

---

## Step 2: Add Custom Domain to Azure Container App

Now we need to configure Azure to accept your custom domain.

### **Option A: Using Azure CLI (Recommended)**

#### **Step 2.1: Get Domain Verification ID**

Run this command to get your verification ID:

```bash
az containerapp show \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --query "properties.customDomainVerificationId" \
  -o tsv
```

**Save this verification ID** - you'll need it for the TXT record.

#### **Step 2.2: Add TXT Record in GoDaddy**

Go back to GoDaddy DNS settings and add:

```
Type: TXT
Name: asuid.projectledger.ca
Value: [Paste the verification ID from Step 2.1]
TTL: 600 seconds
```

**Wait 5-10 minutes** for DNS propagation before proceeding.

#### **Step 2.3: Add Custom Domain to Container App**

For **projectledger.ca** (root domain):

```bash
az containerapp hostname add \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --hostname projectledger.ca
```

For **www.projectledger.ca**:

```bash
az containerapp hostname add \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --hostname www.projectledger.ca
```

#### **Step 2.4: Bind SSL Certificate**

Azure Container Apps automatically provisions a free managed SSL certificate:

```bash
# Enable managed certificate for root domain
az containerapp hostname bind \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --hostname projectledger.ca \
  --environment projectledger-env \
  --validation-method HTTP

# Enable managed certificate for www subdomain
az containerapp hostname bind \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --hostname www.projectledger.ca \
  --environment projectledger-env \
  --validation-method HTTP
```

---

### **Option B: Using Azure Portal**

#### **Step 2.1: Get Domain Verification ID**

1. Go to Azure Portal: https://portal.azure.com
2. Navigate to: **Resource Groups** → **projectledger-poc** → **projectledger-frontend**
3. In the left menu, click **Custom domains**
4. Copy the **Custom Domain Verification ID** at the top

#### **Step 2.2: Add TXT Record in GoDaddy**

Add the TXT record with the verification ID (as described above).
Wait 5-10 minutes for DNS propagation.

#### **Step 2.3: Add Custom Domain**

1. In Azure Portal, still on the Custom domains page
2. Click **+ Add custom domain**
3. Enter `projectledger.ca`
4. Click **Validate**
5. If validation succeeds, click **Add**
6. Repeat for `www.projectledger.ca`

#### **Step 2.4: Configure SSL**

1. After adding the domain, click on it
2. Select **Managed Certificate** (free)
3. Click **Add binding**
4. Wait 5-15 minutes for certificate provisioning

---

## Step 3: Verify Configuration

### **Check DNS Propagation**

Use online tools to verify DNS is working:
- https://www.whatsmydns.net/#A/projectledger.ca
- https://dnschecker.org/#A/projectledger.ca

You should see **20.1.213.250** as the resolved IP.

### **Test Domain Access**

After DNS propagates (can take up to 48 hours, usually 5-30 minutes):

```bash
# Test root domain
curl -I https://projectledger.ca

# Test www subdomain
curl -I https://www.projectledger.ca
```

Both should return **200 OK** with SSL certificate.

### **Test in Browser**

Open in your browser:
- https://projectledger.ca
- https://www.projectledger.ca

Both should load your application with a valid SSL certificate (green padlock).

---

## Step 4: Update Application Configuration

After custom domain is working, update your backend environment variables:

```bash
az containerapp update \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --set-env-vars "FRONTEND_URL=https://projectledger.ca"
```

---

## 📊 DNS Records Summary

After setup, your GoDaddy DNS should look like this:

| Type | Name                  | Value                              | TTL |
|------|-----------------------|-----------------------------------|-----|
| A    | @                     | 20.1.213.250                      | 600 |
| A    | www                   | 20.1.213.250                      | 600 |
| TXT  | asuid.projectledger.ca| [Azure verification ID]           | 600 |

---

## 🔍 Troubleshooting

### **Domain Not Resolving**
- **Issue:** Domain doesn't point to Azure IP
- **Solution:** 
  - Check DNS propagation using whatsmydns.net
  - Verify A records in GoDaddy are correct
  - Wait up to 48 hours for full DNS propagation
  - Clear your DNS cache: `ipconfig /flushdns` (Windows)

### **Certificate Error in Browser**
- **Issue:** "Your connection is not private" error
- **Solution:**
  - Wait 15-30 minutes for Azure to provision certificate
  - Verify TXT record is correctly configured
  - Check domain binding in Azure Portal
  - Try certificate provisioning again

### **Domain Verification Fails**
- **Issue:** Azure can't verify domain ownership
- **Solution:**
  - Ensure TXT record `asuid.projectledger.ca` is added correctly
  - Wait 10-15 minutes after adding TXT record
  - Use `nslookup -type=TXT asuid.projectledger.ca` to verify
  - Check there are no conflicting DNS records

### **Site Loads but API Calls Fail**
- **Issue:** Frontend loads but can't reach backend
- **Solution:**
  - Update backend FRONTEND_URL to https://projectledger.ca
  - Check Nginx configuration allows your domain
  - Verify CORS settings in backend

### **HTTP Instead of HTTPS**
- **Issue:** Site loads on http:// but not https://
- **Solution:**
  - Ensure SSL certificate binding is complete
  - Check certificate status in Azure Portal
  - Wait for certificate provisioning (can take 15-30 min)

---

## 🚀 Quick Setup Commands

For quick reference, here are all the commands in sequence:

```bash
# 1. Get verification ID
VERIFICATION_ID=$(az containerapp show \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --query "properties.customDomainVerificationId" \
  -o tsv)

echo "Verification ID: $VERIFICATION_ID"
echo "Add this as TXT record: asuid.projectledger.ca"
echo ""
echo "After adding TXT record and waiting 10 minutes, run:"
echo ""

# 2. Add custom domains
echo "az containerapp hostname add --name projectledger-frontend --resource-group projectledger-poc --hostname projectledger.ca"
echo "az containerapp hostname add --name projectledger-frontend --resource-group projectledger-poc --hostname www.projectledger.ca"
echo ""

# 3. Bind SSL certificates
echo "az containerapp hostname bind --name projectledger-frontend --resource-group projectledger-poc --hostname projectledger.ca --environment projectledger-env --validation-method HTTP"
echo "az containerapp hostname bind --name projectledger-frontend --resource-group projectledger-poc --hostname www.projectledger.ca --environment projectledger-env --validation-method HTTP"
echo ""

# 4. Update backend
echo "az containerapp update --name projectledger-backend --resource-group projectledger-poc --set-env-vars 'FRONTEND_URL=https://projectledger.ca'"
```

---

## 📞 Support Resources

- **Azure Container Apps Custom Domains:** https://learn.microsoft.com/en-us/azure/container-apps/custom-domains-certificates
- **GoDaddy DNS Management:** https://www.godaddy.com/help/manage-dns-records-680
- **DNS Propagation Checker:** https://www.whatsmydns.net
- **SSL Certificate Checker:** https://www.sslshopper.com/ssl-checker.html

---

## ✅ Post-Setup Checklist

After completing the setup:

- [ ] DNS A records added in GoDaddy (@ and www)
- [ ] TXT record for domain verification added
- [ ] Custom domains added to Azure Container App
- [ ] SSL certificates bound and active
- [ ] DNS propagation verified (whatsmydns.net)
- [ ] HTTPS working for both projectledger.ca and www.projectledger.ca
- [ ] Backend FRONTEND_URL updated
- [ ] Application loads correctly on custom domain
- [ ] API calls working from custom domain

---

**Next Step:** Go to GoDaddy and configure your DNS records using the information above!
