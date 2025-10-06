# Azure Deployment Plan - Project Ledger
## Cost-Optimized Production Deployment Strategy

### 🎯 **Objective**
Deploy Project Ledger to Azure as a **Proof of Concept (PoC)** with the lowest possible cost while maintaining basic reliability. Target: **$36-49/month** using existing `projectledger.ca` domain.

### 🎯 **PoC Specific Requirements**
- ✅ Use existing `projectledger.ca` domain from GoDaddy
- ✅ Minimal monitoring (basic Azure Portal metrics only)
- ✅ Skip CDN (not needed for PoC traffic levels)
- ✅ Focus on core functionality testing
- ✅ Cost optimization prioritized over enterprise features

### 📊 **Current Application Analysis**

**Architecture:**
- **Frontend:** React TypeScript SPA with Material-UI
- **Backend:** Node.js/Express API with Prisma ORM
- **Database:** PostgreSQL 17
- **Authentication:** JWT-based with OAuth (Google/Microsoft)
- **Containerized:** Docker with multi-stage builds

**Resource Requirements (from docker-compose analysis):**
- Frontend: 1-2GB RAM, 1-2 CPU cores
- Backend: 1-2GB RAM, 1 CPU core  
- Database: 512MB-2GB RAM, shared CPU

---

## 🏗️ **Deployment Strategy: Azure Container Apps (Recommended)**

### **Why Container Apps over alternatives:**
- ✅ **Cheapest option** - Pay only for actual usage (can scale to zero)
- ✅ **Simplest deployment** - Native container support
- ✅ **Built-in ingress** - No need for load balancers or app gateways
- ✅ **Automatic HTTPS** - Free SSL certificates
- ✅ **Integrated secrets** - Built-in Key Vault integration
- ✅ **GitHub Actions ready** - Easy CI/CD setup

### **Architecture Overview:**
```
Internet → Container App Environment → [Frontend Container] + [Backend Container]
                                               ↓
                                    Azure Database for PostgreSQL
```

---

## 💰 **Cost Breakdown (Estimated Monthly)**

### **Primary Services:**
| Service | Configuration | Monthly Cost | Notes |
|---------|---------------|--------------|-------|
| **Container Apps Environment** | Consumption plan | $0 | Free tier available |
| **Frontend Container App** | 0.25 CPU, 0.5GB RAM | $8-12 | Auto-scales, pay-per-use |
| **Backend Container App** | 0.25 CPU, 0.5GB RAM | $8-12 | Auto-scales, pay-per-use |
| **Azure Database PostgreSQL** | Flexible Server B1ms | $12-15 | Burstable, 1 vCore, 2GB RAM |
| **Container Registry** | Basic tier | $5 | For storing Docker images |
| **Key Vault** | Standard | $1-2 | For secrets management |
| **Storage Account** | Standard LRS | $2-3 | For backups and static assets |
| **Custom Domain** | projectledger.ca (owned) | $0 | Using existing GoDaddy domain |

### **Total Estimated Cost: $36-49/month**

### **Optional Services (Not needed for PoC):**
| Service | Monthly Cost | Purpose | Status |
|---------|--------------|---------|--------|
| Application Gateway | $20-25 | Advanced routing, WAF protection | ❌ Skip for PoC |
| CDN Profile | $5-10 | Global content delivery | ❌ Not needed (existing domain) |
| Log Analytics | $2-5 | Enhanced monitoring | ❌ Skip for PoC |
| **PoC Total Cost: $36-49/month**

---

## 🚀 **Deployment Steps**

### **Phase 1: Prerequisites**
```bash
# Required tools
az --version          # Azure CLI
docker --version      # Docker
git --version        # Git

# Login to Azure
az login

# Set default subscription (if needed)
az account set --subscription "your-subscription-id"
```

### **Phase 2: Resource Creation**
```bash
# 1. Create Resource Group
az group create \
  --name "projectledger-prod" \
  --location "East US"

# 2. Create Container Registry
az acr create \
  --resource-group "projectledger-prod" \
  --name "projectledgerregistry" \
  --sku Basic \
  --admin-enabled true

# 3. Create PostgreSQL Database
az postgres flexible-server create \
  --resource-group "projectledger-prod" \
  --name "projectledger-db" \
  --location "East US" \
  --admin-user "postgres" \
  --admin-password "SECURE_PASSWORD_HERE" \
  --sku-name "Standard_B1ms" \
  --tier "Burstable" \
  --version "13" \
  --storage-size 32

# 4. Create Container Apps Environment
az containerapp env create \
  --name "projectledger-env" \
  --resource-group "projectledger-prod" \
  --location "East US"
```

### **Phase 3: Build and Push Containers**
```bash
# Login to registry
az acr login --name projectledgerregistry

# Build and push backend
docker build -f apps/backend/Dockerfile -t projectledgerregistry.azurecr.io/backend:latest .
docker push projectledgerregistry.azurecr.io/backend:latest

# Build and push frontend
docker build -f apps/frontend/Dockerfile -t projectledgerregistry.azurecr.io/frontend:latest .
docker push projectledgerregistry.azurecr.io/frontend:latest
```

### **Phase 4: Deploy Container Apps**
```bash
# Deploy Backend
az containerapp create \
  --name "projectledger-backend" \
  --resource-group "projectledger-prod" \
  --environment "projectledger-env" \
  --image "projectledgerregistry.azurecr.io/backend:latest" \
  --target-port 3001 \
  --ingress external \
  --registry-server "projectledgerregistry.azurecr.io" \
  --env-vars NODE_ENV=production PORT=3001 \
  --cpu 0.25 --memory 0.5Gi \
  --min-replicas 0 --max-replicas 3

# Deploy Frontend  
az containerapp create \
  --name "projectledger-frontend" \
  --resource-group "projectledger-prod" \
  --environment "projectledger-env" \
  --image "projectledgerregistry.azurecr.io/frontend:latest" \
  --target-port 80 \
  --ingress external \
  --registry-server "projectledgerregistry.azurecr.io" \
  --cpu 0.25 --memory 0.5Gi \
  --min-replicas 0 --max-replicas 5
```

---

## 🔧 **Configuration Management**

### **Environment Variables (Backend):**
```bash
# Required environment variables
NODE_ENV=production
PORT=3001
DATABASE_URL=postgresql://username:password@server:5432/database
JWT_SECRET=your-super-secret-jwt-key
CORS_ORIGIN=https://yourdomain.com
FRONTEND_URL=https://yourdomain.com

# Optional integrations
PAYPAL_CLIENT_ID=your-paypal-client-id
PAYPAL_CLIENT_SECRET=your-paypal-client-secret
GOOGLE_CLIENT_ID=your-google-oauth-client-id
GOOGLE_CLIENT_SECRET=your-google-oauth-secret
```

### **Secrets Management:**
```bash
# Create Key Vault
az keyvault create \
  --name "projectledger-kv" \
  --resource-group "projectledger-prod" \
  --location "East US"

# Store secrets
az keyvault secret set --vault-name "projectledger-kv" --name "database-url" --value "your-db-connection-string"
az keyvault secret set --vault-name "projectledger-kv" --name "jwt-secret" --value "your-jwt-secret"
```

---

## 🌍 **Custom Domain & SSL**

### **GoDaddy DNS Configuration:**
```dns
# A Records to configure in GoDaddy
projectledger.ca        →  Container App Frontend IP
www.projectledger.ca    →  Container App Frontend IP  
api.projectledger.ca    →  Container App Backend IP
```

**GoDaddy Setup Steps:**
1. Login to GoDaddy DNS management
2. Create/update A records with the IPs from deployed Container Apps
3. TTL: Set to 600 seconds (10 minutes) for faster propagation during setup
4. Test DNS propagation with `nslookup projectledger.ca`

### **SSL Setup:**
Azure Container Apps provides **automatic HTTPS** with free certificates. Simply add your custom domain through the Azure Portal or CLI:

```bash
# Add custom domain (free SSL included)
az containerapp hostname add \
  --hostname "projectledger.ca" \
  --name "projectledger-frontend" \
  --resource-group "projectledger-prod"

az containerapp hostname add \
  --hostname "www.projectledger.ca" \
  --name "projectledger-frontend" \
  --resource-group "projectledger-prod"

az containerapp hostname add \
  --hostname "api.projectledger.ca" \
  --name "projectledger-backend" \
  --resource-group "projectledger-prod"
```

---

## 📈 **Performance Optimizations**

### **Cost Optimization Features:**
1. **Auto-scaling to zero** - No charges when idle
2. **Consumption billing** - Pay only for actual resource usage
3. **Shared compute** - Multiple containers share underlying resources
4. **Built-in load balancing** - No extra load balancer costs

### **Performance Settings:**
```yaml
# Container App Scaling Rules
replicas:
  min: 0          # Scale to zero when idle
  max: 10         # Scale up under load
cpu: 0.25         # Quarter CPU core
memory: 0.5Gi     # 512MB RAM
```

---

## 🔒 **Security Considerations**

### **Database Security:**
- ✅ SSL enforced connections
- ✅ Firewall rules (Azure services only)
- ✅ Admin password in Key Vault
- ✅ Regular automated backups

### **Application Security:**
- ✅ HTTPS enforced (automatic)
- ✅ Environment variables via Key Vault
- ✅ Container image scanning (ACR)
- ✅ Network isolation (Container Apps Environment)

### **Essential Security (PoC Minimum):**
```bash
# Configure database firewall (essential)
az postgres flexible-server firewall-rule create \
  --resource-group "projectledger-prod" \
  --name "projectledger-db" \
  --rule-name "AllowAzureServices" \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0

# Note: Advanced monitoring skipped for PoC - can be added later
```

---

## 📊 **Monitoring & Maintenance (PoC Minimal Setup)**

### **Basic Monitoring (Free/Included):**
- ✅ Container Apps basic metrics (available in Azure Portal)
- ✅ Database basic performance metrics
- ✅ Application container logs (basic retention)
- ✅ Cost analysis and budget alerts (recommended for cost control)

### **Backup Strategy:**
```bash
# Automated PostgreSQL backups (7-day retention included)
# Manual backup script for critical data
az postgres flexible-server backup create \
  --resource-group "projectledger-prod" \
  --name "projectledger-db" \
  --backup-name "manual-backup-$(date +%Y%m%d)"
```

---

## 🎯 **Alternative Deployment Options**

### **Option 2: Azure App Service (Higher Cost)**
- **Cost:** $55-75/month
- **Pros:** Simpler management, built-in scaling
- **Cons:** Always-on billing, more expensive

### **Option 3: AKS (Kubernetes) - Not Recommended for Small Apps**
- **Cost:** $100+/month
- **Pros:** Maximum flexibility and scalability  
- **Cons:** Complex setup, overkill for this size application

### **Option 4: Azure Static Web Apps + Functions**
- **Cost:** $25-40/month
- **Pros:** Serverless, very cheap
- **Cons:** Requires application architecture changes

---

## ✅ **Final Deployment Checklist**

### **Pre-deployment:**
- [ ] Azure subscription ready
- [ ] projectledger.ca domain access (GoDaddy)
- [ ] PayPal/Stripe accounts configured (if needed for PoC)
- [ ] OAuth applications created (Google/Microsoft) - optional for PoC

### **Deployment:**
- [ ] Resource group created
- [ ] Container registry setup
- [ ] PostgreSQL database provisioned
- [ ] Container apps environment created
- [ ] Applications deployed and tested
- [ ] Custom domain configured (if applicable)
- [ ] SSL certificates verified

### **Post-deployment (PoC Focus):**
- [ ] Database seeded with initial data
- [ ] Basic functionality tested (login, create project, invoice)
- [ ] Domain DNS propagation verified
- [ ] SSL certificates active
- [ ] Cost monitoring alerts configured

---

## 🚦 **Go/No-Go Decision Points**

### **✅ Proceed if:**
- Budget allows $40-80/month
- Team comfortable with Docker/containers
- Basic Azure CLI experience available
- Domain management access (if using custom domain)

### **🛑 Consider alternatives if:**
- Budget under $30/month (consider shared hosting)
- Zero container experience (consider Azure App Service)
- Need enterprise-grade SLA (consider premium tiers)

---

## 🚀 **PoC Quick Start Summary**

### **Simplified Deployment Steps:**
1. **Azure Setup** (30 min): Create resource group, container registry, PostgreSQL
2. **Build & Deploy** (60 min): Build containers, deploy to Container Apps  
3. **DNS Configuration** (15 min): Point `projectledger.ca` A records to Container App IPs
4. **SSL Setup** (30 min): Add custom domains to Container Apps (auto SSL)
5. **Testing** (30 min): Verify functionality and domain access

### **Total Estimated Time: 2.5-3 hours**

### **Final PoC Configuration:**
- **URL**: `https://projectledger.ca` (frontend)
- **API**: `https://api.projectledger.ca` (backend)  
- **Cost**: $36-49/month
- **Monitoring**: Basic Azure Portal metrics
- **Scaling**: Auto-scale to zero when idle
- **SSL**: Automatic with custom domain

---

**💡 Ready to proceed?** This PoC setup will give you a fully functional, cost-effective deployment to test Project Ledger with real users and gather feedback before scaling up.

Would you like me to create the automated deployment scripts and step-by-step commands to implement this simplified plan?