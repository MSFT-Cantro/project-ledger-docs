# Azure Manual Rollout Plan for Project Ledger

This document outlines a manual Azure deployment plan for Project Ledger that fits within a **$150/month budget** while providing a production-ready environment with proper security, scalability, and monitoring.

## 🎯 Overview

**Budget Target**: $150/month
**Estimated Monthly Cost**: ~$105-115/month
**Architecture**: Container-based deployment using Azure Container Instances with cost-optimized managed services

## 💰 Cost Breakdown

| Service | Configuration | Monthly Cost (USD) |
|---------|--------------|-------------------|
| **Azure Container Instances (Frontend)** | 1 vCPU, 1.5GB RAM | ~$35 |
| **Azure Container Instances (Backend)** | 1 vCPU, 2GB RAM | ~$40 |
| **Azure Database for PostgreSQL** | Basic, 1 vCore, 50GB | ~$25 |
| **Azure Load Balancer** | Basic (free with 5 rules) | ~$0 |
| **Azure Storage Account** | Standard LRS, 50GB | ~$3 |
| **Azure Key Vault** | Standard operations | ~$2 |
| **GitHub Container Registry** | Free for public repos | ~$0 |
| **DNS (Registrar)** | Using domain registrar | ~$0 |
| **Total Estimated Cost** | | **~$105/month** |

*Buffer: ~$45/month for unexpected usage, SSL certificates, or scaling*

## 🏗️ Architecture Overview

```
Internet → Domain Registrar DNS → Azure Load Balancer → Container Instances
                                        ↓
                               Load Balancing (Basic)
                                        ↓
                        Frontend (ACI) + Backend (ACI)
                                        ↓
                            Azure PostgreSQL Database
```

## 📋 Prerequisites

### Azure Account Setup
- Azure subscription with admin rights
- Azure CLI installed locally
- Docker installed for building images
- Domain name ready to configure

### Required Tools
```bash
# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Login to Azure
az login

# Install Docker (if not already installed)
# Follow Docker installation guide for your OS

# Login to GitHub Container Registry
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin
```

## 🚀 Step-by-Step Deployment Guide

### Phase 1: Initial Azure Setup (30 minutes)

#### 1.1 Create Resource Group and Basic Resources

```bash
# Set variables
RESOURCE_GROUP="projectledger-prod"
LOCATION="eastus"
GITHUB_USERNAME="your-github-username"
DOMAIN_NAME="projectledger.ca"

# Create resource group
az group create --name $RESOURCE_GROUP --location $LOCATION
```

#### 1.2 Create Storage Account for Backups

```bash
STORAGE_ACCOUNT="projectledgerstorage$(date +%s)"

az storage account create \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku Standard_LRS \
  --kind StorageV2
```

#### 1.3 Create Key Vault for Secrets

```bash
KEYVAULT_NAME="projectledger-kv-$(date +%s)"

az keyvault create \
  --name $KEYVAULT_NAME \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku standard
```

### Phase 2: Database Setup (20 minutes)

#### 2.1 Create Azure Database for PostgreSQL

```bash
DB_SERVER_NAME="projectledger-db-$(date +%s)"
DB_NAME="projectledger"
DB_USERNAME="projectledger_admin"
DB_PASSWORD="$(openssl rand -base64 32)"

# Create PostgreSQL server
az postgres server create \
  --resource-group $RESOURCE_GROUP \
  --name $DB_SERVER_NAME \
  --location $LOCATION \
  --admin-user $DB_USERNAME \
  --admin-password $DB_PASSWORD \
  --sku-name B_Gen5_1 \
  --storage-size 51200 \
  --version 13

# Create database
az postgres db create \
  --resource-group $RESOURCE_GROUP \
  --server-name $DB_SERVER_NAME \
  --name $DB_NAME

# Configure firewall to allow Azure services
az postgres server firewall-rule create \
  --resource-group $RESOURCE_GROUP \
  --server $DB_SERVER_NAME \
  --name "AllowAzureServices" \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0
```

#### 2.2 Store Database Secrets

```bash
# Store database connection info in Key Vault
az keyvault secret set \
  --vault-name $KEYVAULT_NAME \
  --name "database-url" \
  --value "postgresql://${DB_USERNAME}:${DB_PASSWORD}@${DB_SERVER_NAME}.postgres.database.azure.com:5432/${DB_NAME}?sslmode=require"

az keyvault secret set \
  --vault-name $KEYVAULT_NAME \
  --name "jwt-secret" \
  --value "$(openssl rand -base64 64)"
```

### Phase 3: Container Images (45 minutes)

#### 3.1 Build and Push Frontend Image

```bash
# Build and push frontend to GitHub Container Registry
docker build -f apps/frontend/Dockerfile -t ghcr.io/$GITHUB_USERNAME/projectledger-frontend:latest .
docker push ghcr.io/$GITHUB_USERNAME/projectledger-frontend:latest
```

#### 3.2 Build and Push Backend Image

```bash
# Build and push backend to GitHub Container Registry
docker build -f apps/backend/Dockerfile -t ghcr.io/$GITHUB_USERNAME/projectledger-backend:latest .
docker push ghcr.io/$GITHUB_USERNAME/projectledger-backend:latest
```

### Phase 4: Container Instances (30 minutes)

#### 4.1 Create Backend Container Instance

Create `backend-deploy.yaml`:
```yaml
apiVersion: '2019-12-01'
location: eastus
name: projectledger-backend
properties:
  containers:
  - name: backend
    properties:
      image: ghcr.io/your-github-username/projectledger-backend:latest
      resources:
        requests:
          cpu: 1.0
          memoryInGb: 2.0
      ports:
      - port: 3001
        protocol: TCP
      environmentVariables:
      - name: NODE_ENV
        value: 'production'
      - name: PORT
        value: '3001'
      - name: DATABASE_URL
        secureValue: 'YOUR_DATABASE_URL_FROM_KEYVAULT'
      - name: JWT_SECRET
        secureValue: 'YOUR_JWT_SECRET_FROM_KEYVAULT'
  osType: Linux
  ipAddress:
    type: Public
    ports:
    - protocol: TCP
      port: 3001
  imageRegistryCredentials:
  - server: ghcr.io
    username: 'YOUR_GITHUB_USERNAME'
    password: 'YOUR_GITHUB_TOKEN'
  restartPolicy: Always
tags: {}
type: Microsoft.ContainerInstance/containerGroups
```

#### 4.2 Deploy Backend Container

```bash
# Deploy backend container using GitHub Container Registry
az container create \
  --resource-group $RESOURCE_GROUP \
  --name projectledger-backend \
  --image ghcr.io/$GITHUB_USERNAME/projectledger-backend:latest \
  --cpu 1 \
  --memory 2 \
  --registry-login-server ghcr.io \
  --registry-username $GITHUB_USERNAME \
  --registry-password $GITHUB_TOKEN \
  --ip-address Public \
  --ports 3001 \
  --environment-variables \
    NODE_ENV=production \
    PORT=3001 \
  --secure-environment-variables \
    DATABASE_URL="$(az keyvault secret show --vault-name $KEYVAULT_NAME --name database-url --query value -o tsv)" \
    JWT_SECRET="$(az keyvault secret show --vault-name $KEYVAULT_NAME --name jwt-secret --query value -o tsv)"
```

#### 4.3 Deploy Frontend Container

```bash
# Get backend IP
BACKEND_IP=$(az container show --resource-group $RESOURCE_GROUP --name projectledger-backend --query ipAddress.ip -o tsv)

# Deploy frontend container
az container create \
  --resource-group $RESOURCE_GROUP \
  --name projectledger-frontend \
  --image ghcr.io/$GITHUB_USERNAME/projectledger-frontend:latest \
  --cpu 1 \
  --memory 1.5 \
  --registry-login-server ghcr.io \
  --registry-username $GITHUB_USERNAME \
  --registry-password $GITHUB_TOKEN \
  --ip-address Public \
  --ports 80 443 \
  --environment-variables \
    REACT_APP_API_URL=https://$DOMAIN_NAME/api \
    REACT_APP_ENVIRONMENT=production
```

### Phase 5: Load Balancer Setup (30 minutes)

#### 5.1 Create Public IPs

```bash
# Create public IPs for frontend and backend
az network public-ip create \
  --resource-group $RESOURCE_GROUP \
  --name projectledger-frontend-ip \
  --allocation-method Static \
  --sku Basic

az network public-ip create \
  --resource-group $RESOURCE_GROUP \
  --name projectledger-backend-ip \
  --allocation-method Static \
  --sku Basic
```

#### 5.2 Create Load Balancer

```bash
# Create Load Balancer (Basic tier is free)
az network lb create \
  --resource-group $RESOURCE_GROUP \
  --name projectledger-lb \
  --sku Basic \
  --public-ip-address projectledger-frontend-ip \
  --frontend-ip-name FrontendPool \
  --backend-pool-name BackendPool

# Create backend address pool for API
az network lb address-pool create \
  --resource-group $RESOURCE_GROUP \
  --lb-name projectledger-lb \
  --name BackendPoolAPI

# Create load balancer rules
az network lb rule create \
  --resource-group $RESOURCE_GROUP \
  --lb-name projectledger-lb \
  --name HTTPRule \
  --protocol Tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name FrontendPool \
  --backend-pool-name BackendPool

az network lb rule create \
  --resource-group $RESOURCE_GROUP \
  --lb-name projectledger-lb \
  --name HTTPSRule \
  --protocol Tcp \
  --frontend-port 443 \
  --backend-port 443 \
  --frontend-ip-name FrontendPool \
  --backend-pool-name BackendPool

az network lb rule create \
  --resource-group $RESOURCE_GROUP \
  --lb-name projectledger-lb \
  --name APIRule \
  --protocol Tcp \
  --frontend-port 3001 \
  --backend-port 3001 \
  --frontend-ip-name FrontendPool \
  --backend-pool-name BackendPoolAPI
```

### Phase 6: DNS Configuration (15 minutes)

#### 6.1 Configure DNS at Your Registrar

```bash
# Get Load Balancer public IP
LB_IP=$(az network public-ip show --resource-group $RESOURCE_GROUP --name projectledger-frontend-ip --query ipAddress -o tsv)

echo "Configure the following DNS records at your domain registrar:"
echo "A Record: @ (root domain) -> $LB_IP"
echo "A Record: www -> $LB_IP"
echo "A Record: api -> $LB_IP"
echo ""
echo "Example DNS configuration:"
echo "projectledger.ca        A    $LB_IP"
echo "www.projectledger.ca    A    $LB_IP"
echo "api.projectledger.ca    A    $LB_IP"
```

#### 6.2 SSL Certificate Setup

Since we're not using Application Gateway, you'll need to handle SSL certificates differently:

**Option 1: Let's Encrypt with Certbot (Recommended)**
```bash
# Install certbot in a container or on the host
# This is a manual process that needs to be set up on your containers

# Example for nginx in frontend container:
# 1. Install certbot in your frontend Dockerfile
# 2. Configure nginx to handle ACME challenges
# 3. Set up automatic renewal
```

**Option 2: Use Cloudflare (Free SSL)**
```bash
# Alternative: Use Cloudflare as a proxy
# 1. Add your domain to Cloudflare
# 2. Update DNS to point to Cloudflare
# 3. Enable "Flexible" or "Full" SSL in Cloudflare
echo "Consider using Cloudflare for free SSL termination"
```

### Phase 7: Final Configuration (20 minutes)

#### 7.1 Update Container Environment Variables

```bash
# Get Load Balancer IP for API configuration
LB_IP=$(az network public-ip show --resource-group $RESOURCE_GROUP --name projectledger-frontend-ip --query ipAddress -o tsv)

# Update backend container with proper CORS settings
az container create \
  --resource-group $RESOURCE_GROUP \
  --name projectledger-backend-updated \
  --image ghcr.io/$GITHUB_USERNAME/projectledger-backend:latest \
  --cpu 1 \
  --memory 2 \
  --registry-login-server ghcr.io \
  --registry-username $GITHUB_USERNAME \
  --registry-password $GITHUB_TOKEN \
  --ip-address Public \
  --ports 3001 \
  --environment-variables \
    NODE_ENV=production \
    PORT=3001 \
    CORS_ORIGIN=https://$DOMAIN_NAME,https://www.$DOMAIN_NAME \
    FRONTEND_URL=https://$DOMAIN_NAME \
  --secure-environment-variables \
    DATABASE_URL="$(az keyvault secret show --vault-name $KEYVAULT_NAME --name database-url --query value -o tsv)" \
    JWT_SECRET="$(az keyvault secret show --vault-name $KEYVAULT_NAME --name jwt-secret --query value -o tsv)"

# Delete old backend container
az container delete --resource-group $RESOURCE_GROUP --name projectledger-backend --yes
```

## 🔧 Configuration Files

### Environment Configuration

Create `.env.azure.production`:
```bash
# Database
DATABASE_URL=postgresql://projectledger_admin:PASSWORD@projectledger-db.postgres.database.azure.com:5432/projectledger?sslmode=require

# Security
JWT_SECRET=your-jwt-secret-from-keyvault
NODE_ENV=production
PORT=3001

# CORS
CORS_ORIGIN=https://projectledger.ca,https://www.projectledger.ca
FRONTEND_URL=https://projectledger.ca

# GitHub Container Registry
GITHUB_USERNAME=your-github-username
GITHUB_TOKEN=your-github-personal-access-token

# Optional integrations (add as needed)
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
STRIPE_SECRET_KEY=your_stripe_secret_key
```

### Deployment Scripts

Create `deploy-azure.sh`:
```bash
#!/bin/bash
set -e

echo "🚀 Starting Azure deployment..."

# Build and push images to GitHub Container Registry
echo "📦 Building and pushing container images..."
echo $GITHUB_TOKEN | docker login ghcr.io -u $GITHUB_USERNAME --password-stdin

docker build -f apps/frontend/Dockerfile -t ghcr.io/$GITHUB_USERNAME/projectledger-frontend:latest .
docker push ghcr.io/$GITHUB_USERNAME/projectledger-frontend:latest

docker build -f apps/backend/Dockerfile -t ghcr.io/$GITHUB_USERNAME/projectledger-backend:latest .
docker push ghcr.io/$GITHUB_USERNAME/projectledger-backend:latest

# Update container instances
echo "🔄 Updating container instances..."
az container restart --resource-group $RESOURCE_GROUP --name projectledger-backend-updated
az container restart --resource-group $RESOURCE_GROUP --name projectledger-frontend

echo "✅ Deployment completed successfully!"
```

## 📊 Monitoring & Maintenance

### Health Check Endpoints

Configure health checks for both services:
- Frontend: `https://projectledger.ca/health`
- Backend: `https://api.projectledger.ca/health`

**Note**: Without Application Insights, you'll need to implement basic monitoring through:
1. Azure Monitor (basic metrics included)
2. Container logs via Azure CLI
3. Custom logging in your application

### Backup Strategy

Create `backup-azure.sh`:
```bash
#!/bin/bash
DATE=$(date +"%Y%m%d_%H%M%S")
BACKUP_NAME="projectledger_backup_$DATE"

# Create database backup
az postgres db export \
  --resource-group $RESOURCE_GROUP \
  --server-name $DB_SERVER_NAME \
  --name $DB_NAME \
  --blob-url "https://$STORAGE_ACCOUNT.blob.core.windows.net/backups/$BACKUP_NAME.sql"

echo "Backup created: $BACKUP_NAME.sql"
```

### Cost Monitoring

Set up budget alerts:
```bash
# Create budget alert for $150/month
az consumption budget create \
  --resource-group $RESOURCE_GROUP \
  --budget-name "ProjectLedgerBudget" \
  --amount 150 \
  --time-grain Monthly \
  --category Cost
```

## 🔄 Scaling & Optimization

### Cost Optimization Strategies

1. **Reserved Instances**: Consider 1-year reserved instances for 20-30% savings
2. **Auto-shutdown**: Implement development environment auto-shutdown
3. **Storage optimization**: Use cool/archive tiers for old backups
4. **Monitoring**: Regular cost analysis and optimization

### Performance Optimization

1. **Content Delivery**: Add Azure CDN for static assets (+$10/month)
2. **Database optimization**: Enable connection pooling
3. **Container optimization**: Right-size container resources based on usage

### High Availability Options

If budget allows expansion:
1. **Multi-region deployment**: +$80-120/month
2. **Load balancer upgrade**: +$15/month
3. **Database HA**: +$25/month

## 🚨 Troubleshooting

### Common Issues

1. **Container startup failures**:
   ```bash
   az container logs --resource-group $RESOURCE_GROUP --name projectledger-backend-updated
   az container logs --resource-group $RESOURCE_GROUP --name projectledger-frontend
   ```

2. **Database connection issues**:
   ```bash
   az postgres server show --resource-group $RESOURCE_GROUP --name $DB_SERVER_NAME
   ```

3. **Load balancer problems**:
   ```bash
   az network lb show --resource-group $RESOURCE_GROUP --name projectledger-lb
   ```

4. **DNS/SSL issues**:
   ```bash
   # Test DNS resolution
   nslookup projectledger.ca
   
   # Check SSL certificate (if using Let's Encrypt or Cloudflare)
   curl -I https://projectledger.ca
   ```

### Emergency Procedures

1. **Rollback deployment**:
   ```bash
   az container restart --resource-group $RESOURCE_GROUP --name projectledger-backend-updated
   az container restart --resource-group $RESOURCE_GROUP --name projectledger-frontend
   ```

2. **Scale up resources** (temporary):
   ```bash
   # Create new container with more resources
   az container create --cpu 2 --memory 4 --name projectledger-backend-scaled
   ```

## 📝 Security Checklist

- [ ] Enable Azure Security Center
- [ ] Configure Network Security Groups  
- [ ] Implement Azure Key Vault for secrets ✅
- [ ] Configure Load Balancer rules
- [ ] Set up SSL/TLS certificates (via Let's Encrypt or Cloudflare)
- [ ] Configure database firewall rules
- [ ] Enable container registry vulnerability scanning (GitHub)
- [ ] Set up basic Azure Monitor alerts
- [ ] Implement backup and disaster recovery
- [ ] Configure access management (RBAC)

## 📚 Next Steps

After successful deployment:

1. **SSL Certificates**: Set up Let's Encrypt or use Cloudflare for SSL termination
2. **CI/CD Pipeline**: Set up GitHub Actions for automated deployments
3. **Monitoring**: Configure basic Azure Monitor alerts and custom logging
4. **Performance Testing**: Load test the application
5. **Documentation**: Update operational procedures
6. **Backup Automation**: Set up automated backup scripts

## 💡 Cost-Saving Tips

1. **Use Azure Cost Management**: Monitor spending weekly
2. **Implement tagging**: Tag all resources for cost tracking
3. **Consider spot instances**: For non-production workloads
4. **Optimize storage**: Regular cleanup of unused resources
5. **Review monthly**: Regular architecture reviews for optimization

---

**Total Setup Time**: ~3-4 hours
**Monthly Cost**: ~$105-115/month
**Cost Savings**: ~$30/month compared to full Azure services
**Scalability**: Can handle 10,000+ users with this configuration

## 🔄 Upgrade Path

If you need more features later, you can easily upgrade:

1. **Add Application Gateway** (+$20/month): Better SSL management and WAF
2. **Add Application Insights** (+$3/month): Detailed application monitoring  
3. **Add Azure Container Registry** (+$5/month): Private registry with vulnerability scanning
4. **Add Azure DNS** (+$2/month): Integrated DNS management

For questions or issues with this deployment plan, refer to the Azure documentation or create an issue in the project repository.
