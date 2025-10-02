# 🚀 Azure Deployment & Update Plan

**Project:** Project Ledger  
**Environment:** Azure Container Apps (Production)  
**Last Updated:** October 2, 2025

---

## 📋 Overview

This document outlines the deployment strategy for updating the Project Ledger application on Azure Container Apps with **zero downtime**, **no data loss**, and **stable IP addresses** for daily deployments.

---

## 🎯 Deployment Objectives

- ✅ **Zero Downtime:** Application remains accessible during updates
- ✅ **No Data Loss:** PostgreSQL database persists across deployments
- ✅ **Stable IPs:** Static IP (`20.1.213.250`) never changes
- ✅ **Fast Rollouts:** Deploy new versions in under 5 minutes
- ✅ **Easy Rollbacks:** Revert to previous version instantly
- ✅ **Daily Deployments:** Support multiple updates per day

---

## 🏗️ Architecture Overview

### **Current Infrastructure**

```
┌─────────────────────────────────────────────────────┐
│           Container Apps Environment                 │
│         (Static IP: 20.1.213.250)                   │
│                                                      │
│  ┌─────────────────┐      ┌────────────────────┐  │
│  │  Frontend        │      │  Backend           │  │
│  │  (nginx + React) │ ───> │  (Node.js + API)   │  │
│  │  Replicas: 1-3   │      │  Replicas: 1-3     │  │
│  └─────────────────┘      └────────────────────┘  │
│           │                        │                │
│           │                        │                │
│           └────────────┬───────────┘                │
└────────────────────────┼──────────────────────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │  PostgreSQL Container │
              │  (Persistent Storage) │
              │  Always Running       │
              └──────────────────────┘
```

### **Key Components**

1. **Container Apps Environment** - Provides stable static IP
2. **Frontend Container App** - Auto-scaling, rolling updates
3. **Backend Container App** - Auto-scaling, rolling updates
4. **PostgreSQL Container** - Persistent, never redeployed
5. **Container Registry** - Stores all image versions with tags

---

## 🔄 Deployment Strategy: Blue-Green Revisions

Azure Container Apps uses **revision-based deployments** which provides:

- Multiple revisions can run simultaneously
- Traffic can be split between revisions
- Instant rollback by switching traffic
- Zero downtime updates

### **How It Works**

```
┌─────────────────────────────────────────────────┐
│  Traffic Management (100% control)              │
├─────────────────────────────────────────────────┤
│                                                  │
│  Revision 1 (v1.0.0) ◄──── 100% traffic        │
│  Status: Active                                  │
│                                                  │
│  [Deploy new version]                           │
│                                                  │
│  Revision 2 (v1.0.1) ◄──── 0% traffic          │
│  Status: Ready                                   │
│                                                  │
│  [Test Revision 2]                              │
│  [Switch traffic]                               │
│                                                  │
│  Revision 1 (v1.0.0) ◄──── 0% traffic          │
│  Revision 2 (v1.0.1) ◄──── 100% traffic ✓      │
│                                                  │
└─────────────────────────────────────────────────┘
```

---

## 📝 Standard Deployment Workflow

### **Step 1: Prepare Release**

```bash
# 1. Ensure you're on main branch with latest changes
cd /c/Code/ProjectLedger2
git checkout main
git pull origin main

# 2. Update version in package.json (optional but recommended)
# Edit version: "1.0.1" → "1.0.2"

# 3. Commit version bump
git add package.json apps/backend/package.json apps/frontend/package.json
git commit -m "chore: Bump version to 1.0.2"
git push origin main

# 4. Tag the release (for tracking)
git tag -a v1.0.2 -m "Release version 1.0.2"
git push origin v1.0.2
```

---

### **Step 2: Build & Push Container Images**

```bash
# Ensure Docker is running
docker ps

# Login to Azure Container Registry
az acr login --name projectledgerregistry

# Build and tag images with version
VERSION="1.0.2"

# Backend
docker build \
  -f apps/backend/Dockerfile \
  -t projectledgerregistry.azurecr.io/projectledger-backend:latest \
  -t projectledgerregistry.azurecr.io/projectledger-backend:${VERSION} \
  --target production \
  .

docker push projectledgerregistry.azurecr.io/projectledger-backend:latest
docker push projectledgerregistry.azurecr.io/projectledger-backend:${VERSION}

# Frontend
docker build \
  -f apps/frontend/Dockerfile \
  -t projectledgerregistry.azurecr.io/projectledger-frontend:latest \
  -t projectledgerregistry.azurecr.io/projectledger-frontend:${VERSION} \
  --target azure \
  .

docker push projectledgerregistry.azurecr.io/projectledger-frontend:latest
docker push projectledgerregistry.azurecr.io/projectledger-frontend:${VERSION}
```

**Why Two Tags?**
- `latest` - Always points to newest version (easy for quick deploys)
- `1.0.2` - Specific version tag (enables precise rollbacks)

---

### **Step 3: Database Migrations (If Needed)**

**IMPORTANT:** Run migrations BEFORE deploying new code!

```bash
# Option A: Run migrations via backend container
az containerapp exec \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --command "/bin/sh -c 'npx prisma migrate deploy'"

# Option B: Connect to database directly (requires psql client)
PGPASSWORD=HzxHJdKgjLaamQSqYUUdD8oE8 psql \
  -h projectledger-db.eastus2.azurecontainer.io \
  -U postgres \
  -d projectledger \
  -f path/to/migration.sql
```

**Migration Safety Checklist:**
- ✅ Backup database before major schema changes
- ✅ Test migrations on local environment first
- ✅ Use backward-compatible migrations when possible
- ✅ Never drop columns in production without grace period
- ✅ Add new columns as nullable initially

---

### **Step 4: Deploy Backend (Zero Downtime)**

```bash
# Create new revision with updated image
az containerapp update \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --image projectledgerregistry.azurecr.io/projectledger-backend:latest \
  --revision-suffix v1-0-2

# Container Apps automatically:
# 1. Creates new revision
# 2. Starts new containers
# 3. Waits for health checks to pass
# 4. Gradually shifts traffic (rolling update)
# 5. Keeps old revision running until traffic switches
# 6. Deactivates old revision after successful deployment
```

**Health Check:** Backend must respond to health endpoint within 30 seconds or deployment fails.

---

### **Step 5: Deploy Frontend (Zero Downtime)**

```bash
# Create new revision with updated image
az containerapp update \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --image projectledgerregistry.azurecr.io/projectledger-frontend:latest \
  --revision-suffix v1-0-2

# Same zero-downtime process as backend
```

---

### **Step 6: Verify Deployment**

```bash
# Check revision status
az containerapp revision list \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  -o table

az containerapp revision list \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  -o table

# Test application
curl -I https://projectledger.ca
curl https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io/api/health

# Check logs for errors
az containerapp logs show \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --tail 50

az containerapp logs show \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --tail 50
```

---

### **Step 7: Monitor & Validate**

**Immediate Checks (first 5 minutes):**
- ✅ Health endpoints responding (200 OK)
- ✅ Login functionality works
- ✅ API calls succeeding
- ✅ No error spikes in logs
- ✅ Database queries working

**Extended Monitoring (first 24 hours):**
- Monitor error rates
- Check performance metrics
- Review user feedback
- Watch for memory/CPU issues

---

## 🔙 Rollback Procedure

If issues are detected, rollback is instant:

### **Quick Rollback (Under 1 minute)**

```bash
# List revisions to find previous version
az containerapp revision list \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  -o table

# Activate previous revision and deactivate current
az containerapp revision activate \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --revision projectledger-backend--v1-0-1

az containerapp revision deactivate \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --revision projectledger-backend--v1-0-2

# Repeat for frontend
az containerapp revision activate \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --revision projectledger-frontend--v1-0-1

az containerapp revision deactivate \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --revision projectledger-frontend--v1-0-2
```

### **Traffic Split Rollback (Gradual)**

```bash
# Gradually shift traffic back to previous version
az containerapp ingress traffic set \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --revision-weight \
    projectledger-backend--v1-0-1=80 \
    projectledger-backend--v1-0-2=20

# Then shift to 100% if all is well
az containerapp ingress traffic set \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --revision-weight \
    projectledger-backend--v1-0-1=100 \
    projectledger-backend--v1-0-2=0
```

---

## 🗄️ Database Backup Strategy

### **Automated Backups**

**Currently:** No automated backups configured ⚠️

**Recommended Setup:**

```bash
# Create backup script
cat > /c/Code/ProjectLedger2/tools/backup/backup-db.sh << 'EOF'
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="./backups"
mkdir -p $BACKUP_DIR

PGPASSWORD=HzxHJdKgjLaamQSqYUUdD8oE8 pg_dump \
  -h projectledger-db.eastus2.azurecontainer.io \
  -U postgres \
  -d projectledger \
  -F c \
  -f "$BACKUP_DIR/projectledger_$DATE.backup"

echo "Backup created: $BACKUP_DIR/projectledger_$DATE.backup"

# Upload to Azure Storage (optional)
az storage blob upload \
  --account-name projectledger10020919 \
  --container-name backups \
  --name "projectledger_$DATE.backup" \
  --file "$BACKUP_DIR/projectledger_$DATE.backup"

# Keep only last 30 days
find $BACKUP_DIR -name "*.backup" -mtime +30 -delete
EOF

chmod +x /c/Code/ProjectLedger2/tools/backup/backup-db.sh
```

### **Manual Backup Before Major Changes**

```bash
# Quick manual backup
PGPASSWORD=HzxHJdKgjLaamQSqYUUdD8oE8 pg_dump \
  -h projectledger-db.eastus2.azurecontainer.io \
  -U postgres \
  -d projectledger \
  -F c \
  -f "backup_before_v1.0.2_$(date +%Y%m%d).backup"
```

### **Restore from Backup**

```bash
# Restore database from backup
PGPASSWORD=HzxHJdKgjLaamQSqYUUdD8oE8 pg_restore \
  -h projectledger-db.eastus2.azurecontainer.io \
  -U postgres \
  -d projectledger \
  -c \
  backup_before_v1.0.2_20251002.backup
```

---

## 🔒 IP Address Stability

### **Why IPs Never Change**

Your static IP (`20.1.213.250`) is bound to the **Container Apps Environment**, not individual apps:

```
Container Apps Environment (projectledger-env)
├── Static IP: 20.1.213.250 (PERMANENT)
├── Default Domain: orangeplant-da913b57.eastus2.azurecontainerapps.io
├── Frontend Container App (updates don't change IP)
├── Backend Container App (updates don't change IP)
└── Custom Domain: projectledger.ca (points to static IP)
```

**IP Remains Stable When:**
- ✅ Updating container images
- ✅ Creating new revisions
- ✅ Scaling up/down replicas
- ✅ Changing environment variables
- ✅ Restarting containers

**IP Changes Only When:**
- ❌ Deleting the entire Container Apps Environment
- ❌ Moving to a different Azure region
- ❌ Recreating the environment from scratch

**Action Required:** None! Your DNS records in GoDaddy never need to change.

---

## 📊 Deployment Checklist

### **Pre-Deployment**
- [ ] Code reviewed and tested locally
- [ ] All tests passing
- [ ] Version number updated
- [ ] Git tag created
- [ ] Database migrations prepared (if needed)
- [ ] Backup created (for major changes)
- [ ] Deployment window scheduled (if needed)

### **Deployment**
- [ ] Docker images built with version tags
- [ ] Images pushed to Azure Container Registry
- [ ] Database migrations applied (if needed)
- [ ] Backend updated to new revision
- [ ] Frontend updated to new revision
- [ ] Health checks passing
- [ ] No errors in logs

### **Post-Deployment**
- [ ] Application accessible at projectledger.ca
- [ ] Login functionality tested
- [ ] Critical features verified
- [ ] Performance metrics normal
- [ ] Error rates within acceptable range
- [ ] Team notified of deployment
- [ ] Documentation updated (if needed)

---

## 🚀 Quick Deploy Script

Save time with this automated deployment script:

```bash
# Create deploy script
cat > /c/Code/ProjectLedger2/tools/deployment/deploy-update.sh << 'EOF'
#!/bin/bash
set -e

echo "🚀 Project Ledger Deployment Script"
echo "===================================="
echo ""

# Get version from argument or prompt
VERSION=${1:-$(git describe --tags --abbrev=0 2>/dev/null || echo "latest")}
echo "📦 Deploying version: $VERSION"
echo ""

# Confirm deployment
read -p "Continue with deployment? (y/n) " -n 1 -r
echo
if [[ ! $REPLY =~ ^[Yy]$ ]]; then
    echo "❌ Deployment cancelled"
    exit 1
fi

# Login to ACR
echo "🔐 Logging into Azure Container Registry..."
az acr login --name projectledgerregistry

# Build and push backend
echo "🏗️  Building backend..."
docker build \
  -f apps/backend/Dockerfile \
  -t projectledgerregistry.azurecr.io/projectledger-backend:latest \
  -t projectledgerregistry.azurecr.io/projectledger-backend:${VERSION} \
  --target production \
  .

echo "📤 Pushing backend..."
docker push projectledgerregistry.azurecr.io/projectledger-backend:latest
docker push projectledgerregistry.azurecr.io/projectledger-backend:${VERSION}

# Build and push frontend
echo "🎨 Building frontend..."
docker build \
  -f apps/frontend/Dockerfile \
  -t projectledgerregistry.azurecr.io/projectledger-frontend:latest \
  -t projectledgerregistry.azurecr.io/projectledger-frontend:${VERSION} \
  --target azure \
  .

echo "📤 Pushing frontend..."
docker push projectledgerregistry.azurecr.io/projectledger-frontend:latest
docker push projectledgerregistry.azurecr.io/projectledger-frontend:${VERSION}

# Deploy backend
echo "🚀 Deploying backend..."
REVISION_SUFFIX=$(echo $VERSION | tr '.' '-')
az containerapp update \
  --name projectledger-backend \
  --resource-group projectledger-poc \
  --image projectledgerregistry.azurecr.io/projectledger-backend:latest \
  --revision-suffix ${REVISION_SUFFIX}

# Deploy frontend
echo "🚀 Deploying frontend..."
az containerapp update \
  --name projectledger-frontend \
  --resource-group projectledger-poc \
  --image projectledgerregistry.azurecr.io/projectledger-frontend:latest \
  --revision-suffix ${REVISION_SUFFIX}

# Verify deployment
echo "✅ Verifying deployment..."
sleep 5
curl -I https://projectledger.ca
curl https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io/api/health

echo ""
echo "✅ Deployment complete!"
echo "🔗 Frontend: https://projectledger.ca"
echo "🔗 Backend: https://projectledger-backend.orangeplant-da913b57.eastus2.azurecontainerapps.io"
echo ""
echo "📊 Monitor logs with:"
echo "  az containerapp logs show --name projectledger-backend --resource-group projectledger-poc --tail 50"
echo "  az containerapp logs show --name projectledger-frontend --resource-group projectledger-poc --tail 50"
EOF

chmod +x /c/Code/ProjectLedger2/tools/deployment/deploy-update.sh
```

**Usage:**
```bash
# Deploy with automatic version detection
./tools/deployment/deploy-update.sh

# Deploy with specific version
./tools/deployment/deploy-update.sh v1.0.3
```

---

## 🎯 Best Practices

### **Version Management**
1. **Semantic Versioning:** Use `MAJOR.MINOR.PATCH` (e.g., 1.0.2)
2. **Git Tags:** Tag every production release
3. **Image Tags:** Always push both `latest` and version-specific tags
4. **Changelog:** Maintain CHANGELOG.md for tracking changes

### **Database Migrations**
1. **Backward Compatible:** New code works with old schema
2. **Forward Compatible:** Old code works with new schema
3. **Staged Rollouts:** Deploy schema changes separately from code
4. **Test First:** Always test migrations on staging/local first

### **Monitoring**
1. **Health Checks:** Ensure `/health` endpoint is always responsive
2. **Log Aggregation:** Regularly review logs for warnings
3. **Alerts:** Set up alerts for error spikes (future todo)
4. **Performance:** Monitor response times and CPU/memory usage

### **Communication**
1. **Deployment Window:** Notify team of deployments
2. **Release Notes:** Document what changed in each release
3. **Rollback Plan:** Always have rollback commands ready
4. **Post-Mortem:** Document any issues that occur

---

## 📅 Daily Deployment Workflow

**Morning Deployment (Recommended):**

```bash
# 9:00 AM - Start deployment
git pull origin main
./tools/deployment/deploy-update.sh v1.0.3

# 9:05 AM - Verify deployment
curl https://projectledger.ca
# Test login, create project, verify features

# 9:10 AM - Monitor for issues
# Check logs, error rates, user feedback

# 9:30 AM - Mark deployment complete
# Update team, close deployment ticket
```

**Emergency Hotfix:**

```bash
# Fix critical bug
git checkout -b hotfix/critical-bug
# Make fix
git commit -m "fix: Critical bug in payment processing"
git checkout main
git merge hotfix/critical-bug
git tag -a v1.0.4 -m "Hotfix: Critical bug"
git push origin main --tags

# Deploy immediately
./tools/deployment/deploy-update.sh v1.0.4
```

---

## 📞 Support & Troubleshooting

### **Common Issues**

**"Deployment taking too long"**
- Check if health endpoint is responding
- Review container logs for startup errors
- Verify environment variables are correct

**"New revision not receiving traffic"**
- Check revision status: `az containerapp revision list`
- Manually activate revision if needed
- Verify image was actually updated

**"Database connection errors after deployment"**
- Verify DATABASE_URL is correct
- Check PostgreSQL container is running
- Ensure migrations didn't break anything

### **Getting Help**

- **Azure Container Apps Docs:** https://learn.microsoft.com/en-us/azure/container-apps/
- **Deployment Documentation:** [`docs/fixes/AZURE_DEPLOYMENT_COMPLETE.md`](../fixes/AZURE_DEPLOYMENT_COMPLETE.md)
- **Azure Portal:** https://portal.azure.com → projectledger-poc

---

## ✅ Summary

**Key Takeaways:**
- ✅ Zero downtime deployments using revision-based updates
- ✅ Static IP never changes (20.1.213.250)
- ✅ Instant rollbacks by switching revisions
- ✅ Database persists independently of application updates
- ✅ Support for multiple deployments per day
- ✅ Automated deployment script available

**Ready for Daily Deployments!** 🎉

---

**Next Steps:**
1. Test the deployment script with a minor update
2. Set up automated database backups
3. Configure monitoring alerts (Application Insights)
4. Document team deployment process
5. Create staging environment (optional)
