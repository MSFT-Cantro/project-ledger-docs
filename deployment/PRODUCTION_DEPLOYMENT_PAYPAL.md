# PayPal Integration Production Deployment Strategy

## 🚀 **Overview**

This document outlines the complete strategy for deploying the PayPal integration to production, including environment configuration, deployment scripts, monitoring setup, and rollback procedures.

---

## 🏗️ **1. Environment Configuration**

### Production Environment Variables

Create a `.env.production` file with the following variables:

```bash
# PayPal Configuration
PAYPAL_MODE=live
PAYPAL_CLIENT_ID=<PRODUCTION_CLIENT_ID>
PAYPAL_CLIENT_SECRET=<PRODUCTION_CLIENT_SECRET>
PAYPAL_WEBHOOK_SECRET=<PRODUCTION_WEBHOOK_SECRET>

# Database Configuration
DATABASE_URL=<PRODUCTION_DATABASE_URL>

# Application Configuration
NODE_ENV=production
JWT_SECRET=<PRODUCTION_JWT_SECRET>
API_BASE_URL=https://api.projectledger.com
APP_BASE_URL=https://app.projectledger.com

# Security Configuration
CORS_ORIGIN=https://app.projectledger.com
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100

# Monitoring Configuration
LOG_LEVEL=info
SENTRY_DSN=<PRODUCTION_SENTRY_DSN>
NEW_RELIC_LICENSE_KEY=<PRODUCTION_NEW_RELIC_KEY>
```

### Azure App Service Configuration

```bash
# Set application settings in Azure App Service
az webapp config appsettings set --resource-group ProjectLedger-Production \
  --name projectledger-api-prod \
  --settings \
  PAYPAL_MODE=live \
  PAYPAL_CLIENT_ID="$PAYPAL_CLIENT_ID" \
  PAYPAL_CLIENT_SECRET="$PAYPAL_CLIENT_SECRET" \
  PAYPAL_WEBHOOK_SECRET="$PAYPAL_WEBHOOK_SECRET" \
  DATABASE_URL="$DATABASE_URL" \
  NODE_ENV=production \
  JWT_SECRET="$JWT_SECRET"
```

---

## 📦 **2. Deployment Scripts**

### Production Deployment Script

Create `scripts/deploy-production.sh`:

```bash
#!/bin/bash

# PayPal Integration Production Deployment Script
# This script handles the complete deployment of PayPal integration to production

set -e

echo "🚀 Starting PayPal Integration Production Deployment..."

# Configuration
RESOURCE_GROUP="ProjectLedger-Production"
APP_SERVICE_NAME="projectledger-api-prod"
FRONTEND_APP_NAME="projectledger-app-prod"
DATABASE_NAME="projectledger-prod-db"

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Functions
log_info() {
    echo -e "${GREEN}[INFO]${NC} $1"
}

log_warn() {
    echo -e "${YELLOW}[WARN]${NC} $1"
}

log_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

# Check prerequisites
check_prerequisites() {
    log_info "Checking deployment prerequisites..."
    
    # Check Azure CLI
    if ! command -v az &> /dev/null; then
        log_error "Azure CLI is not installed"
        exit 1
    fi
    
    # Check if logged in to Azure
    if ! az account show &> /dev/null; then
        log_error "Not logged in to Azure. Run 'az login' first."
        exit 1
    fi
    
    # Check environment variables
    if [[ -z "$PAYPAL_CLIENT_ID" || -z "$PAYPAL_CLIENT_SECRET" ]]; then
        log_error "PayPal credentials not set. Please set PAYPAL_CLIENT_ID and PAYPAL_CLIENT_SECRET"
        exit 1
    fi
    
    log_info "Prerequisites check passed ✅"
}

# Backup current deployment
backup_current_deployment() {
    log_info "Creating backup of current deployment..."
    
    # Create backup slot
    az webapp deployment slot create \
        --resource-group "$RESOURCE_GROUP" \
        --name "$APP_SERVICE_NAME" \
        --slot "backup-$(date +%Y%m%d-%H%M%S)" \
        --configuration-source "$APP_SERVICE_NAME"
    
    log_info "Backup created ✅"
}

# Database migration
run_database_migration() {
    log_info "Running database migrations..."
    
    # Run Prisma migrations
    cd apps/backend
    npx prisma migrate deploy
    cd ../..
    
    log_info "Database migrations completed ✅"
}

# Build and test
build_and_test() {
    log_info "Building and testing PayPal integration..."
    
    # Install dependencies
    npm ci
    
    # Run tests
    cd apps/backend
    npm test -- --testPathPattern="paypalService|auth" --verbose
    cd ../..
    
    # Build applications
    npm run build
    
    log_info "Build and test completed ✅"
}

# Deploy backend
deploy_backend() {
    log_info "Deploying backend with PayPal integration..."
    
    # Create deployment package
    cd apps/backend
    zip -r ../../backend-deployment.zip . -x "node_modules/*" "coverage/*" "*.test.*"
    cd ../..
    
    # Deploy to Azure App Service
    az webapp deployment source config-zip \
        --resource-group "$RESOURCE_GROUP" \
        --name "$APP_SERVICE_NAME" \
        --src backend-deployment.zip
    
    # Wait for deployment to complete
    sleep 60
    
    log_info "Backend deployment completed ✅"
}

# Deploy frontend
deploy_frontend() {
    log_info "Deploying frontend with PayPal integration..."
    
    # Build frontend
    cd apps/frontend
    npm run build
    
    # Deploy to Azure Static Web Apps
    az staticwebapp create \
        --name "$FRONTEND_APP_NAME" \
        --resource-group "$RESOURCE_GROUP" \
        --source "./build" \
        --location "East US 2"
    
    cd ../..
    
    log_info "Frontend deployment completed ✅"
}

# Configure PayPal settings
configure_paypal() {
    log_info "Configuring PayPal settings..."
    
    # Set PayPal environment variables
    az webapp config appsettings set \
        --resource-group "$RESOURCE_GROUP" \
        --name "$APP_SERVICE_NAME" \
        --settings \
        PAYPAL_MODE=live \
        PAYPAL_CLIENT_ID="$PAYPAL_CLIENT_ID" \
        PAYPAL_CLIENT_SECRET="$PAYPAL_CLIENT_SECRET" \
        PAYPAL_WEBHOOK_SECRET="$PAYPAL_WEBHOOK_SECRET"
    
    # Configure PayPal webhook URLs
    WEBHOOK_URL="https://${APP_SERVICE_NAME}.azurewebsites.net/api/webhooks/paypal"
    log_info "PayPal webhook URL: $WEBHOOK_URL"
    log_warn "Please configure this webhook URL in your PayPal Developer Dashboard"
    
    log_info "PayPal configuration completed ✅"
}

# Health check
run_health_check() {
    log_info "Running health checks..."
    
    # Wait for app to be ready
    sleep 30
    
    # Check backend health
    BACKEND_URL="https://${APP_SERVICE_NAME}.azurewebsites.net"
    HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$BACKEND_URL/health")
    
    if [ "$HTTP_STATUS" = "200" ]; then
        log_info "Backend health check passed ✅"
    else
        log_error "Backend health check failed (HTTP $HTTP_STATUS)"
        exit 1
    fi
    
    # Check PayPal service initialization
    PAYPAL_STATUS=$(curl -s "$BACKEND_URL/api/status/paypal" | jq -r '.status')
    
    if [ "$PAYPAL_STATUS" = "ready" ]; then
        log_info "PayPal service health check passed ✅"
    else
        log_error "PayPal service health check failed"
        exit 1
    fi
}

# Setup monitoring
setup_monitoring() {
    log_info "Setting up monitoring and alerts..."
    
    # Configure Application Insights
    az monitor app-insights component create \
        --app "projectledger-paypal-insights" \
        --location "East US 2" \
        --resource-group "$RESOURCE_GROUP" \
        --kind web
    
    # Create PayPal-specific alerts
    az monitor metrics alert create \
        --name "PayPal API Errors" \
        --resource-group "$RESOURCE_GROUP" \
        --scopes "/subscriptions/$(az account show --query id -o tsv)/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Web/sites/$APP_SERVICE_NAME" \
        --condition "count 'requests/failed' > 10" \
        --description "High PayPal API error rate detected"
    
    log_info "Monitoring setup completed ✅"
}

# Main deployment flow
main() {
    log_info "🚀 PayPal Integration Production Deployment Started"
    
    check_prerequisites
    backup_current_deployment
    run_database_migration
    build_and_test
    deploy_backend
    deploy_frontend
    configure_paypal
    run_health_check
    setup_monitoring
    
    log_info "🎉 PayPal Integration Production Deployment Completed Successfully!"
    log_warn "Don't forget to:"
    log_warn "1. Configure PayPal webhook URL in PayPal Developer Dashboard"
    log_warn "2. Test subscription flow in production"
    log_warn "3. Monitor logs for the first 24 hours"
}

# Run main deployment
main "$@"
```

### Environment-Specific Configuration Script

Create `scripts/configure-environment.sh`:

```bash
#!/bin/bash

# Environment Configuration Script for PayPal Integration

set -e

ENVIRONMENT=${1:-production}

echo "🔧 Configuring $ENVIRONMENT environment for PayPal integration..."

case $ENVIRONMENT in
    "production")
        PAYPAL_MODE="live"
        APP_SERVICE_NAME="projectledger-api-prod"
        RESOURCE_GROUP="ProjectLedger-Production"
        ;;
    "staging")
        PAYPAL_MODE="sandbox"
        APP_SERVICE_NAME="projectledger-api-staging"
        RESOURCE_GROUP="ProjectLedger-Staging"
        ;;
    "development")
        PAYPAL_MODE="sandbox"
        echo "Development environment - using local configuration"
        exit 0
        ;;
    *)
        echo "❌ Unknown environment: $ENVIRONMENT"
        echo "Usage: $0 [production|staging|development]"
        exit 1
        ;;
esac

# Configure Azure App Service settings
az webapp config appsettings set \
    --resource-group "$RESOURCE_GROUP" \
    --name "$APP_SERVICE_NAME" \
    --settings \
    PAYPAL_MODE="$PAYPAL_MODE" \
    NODE_ENV="$ENVIRONMENT"

echo "✅ Environment configuration completed for $ENVIRONMENT"
```

---

## 🔄 **3. Blue-Green Deployment Strategy**

### Blue-Green Deployment Script

Create `scripts/blue-green-deploy.sh`:

```bash
#!/bin/bash

# Blue-Green Deployment for PayPal Integration

set -e

RESOURCE_GROUP="ProjectLedger-Production"
APP_SERVICE_NAME="projectledger-api-prod"
DEPLOYMENT_SLOT="green"

echo "🔄 Starting Blue-Green deployment..."

# Create green slot if it doesn't exist
az webapp deployment slot create \
    --resource-group "$RESOURCE_GROUP" \
    --name "$APP_SERVICE_NAME" \
    --slot "$DEPLOYMENT_SLOT" \
    --configuration-source "$APP_SERVICE_NAME"

# Deploy to green slot
az webapp deployment source config-zip \
    --resource-group "$RESOURCE_GROUP" \
    --name "$APP_SERVICE_NAME" \
    --slot "$DEPLOYMENT_SLOT" \
    --src backend-deployment.zip

# Wait for deployment
sleep 60

# Health check on green slot
GREEN_URL="https://${APP_SERVICE_NAME}-${DEPLOYMENT_SLOT}.azurewebsites.net"
HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$GREEN_URL/health")

if [ "$HTTP_STATUS" = "200" ]; then
    echo "✅ Green slot health check passed"
    
    # Swap slots
    az webapp deployment slot swap \
        --resource-group "$RESOURCE_GROUP" \
        --name "$APP_SERVICE_NAME" \
        --slot "$DEPLOYMENT_SLOT" \
        --target-slot production
    
    echo "🎉 Blue-Green deployment completed successfully!"
else
    echo "❌ Green slot health check failed"
    exit 1
fi
```

---

## 📊 **4. Monitoring Configuration**

### Application Insights Configuration

```typescript
// monitoring/paypal-monitoring.ts
import { TelemetryClient } from 'applicationinsights';

export class PayPalMonitoring {
    private client: TelemetryClient;

    constructor() {
        this.client = new TelemetryClient(process.env.APPLICATIONINSIGHTS_CONNECTION_STRING);
    }

    trackSubscriptionCreated(subscriptionId: string, accountId: string) {
        this.client.trackEvent({
            name: 'PayPal_Subscription_Created',
            properties: {
                subscriptionId,
                accountId,
                timestamp: new Date().toISOString()
            }
        });
    }

    trackPaymentProcessed(paymentId: string, amount: number) {
        this.client.trackEvent({
            name: 'PayPal_Payment_Processed',
            properties: {
                paymentId,
                amount: amount.toString(),
                timestamp: new Date().toISOString()
            }
        });
    }

    trackWebhookReceived(eventType: string, success: boolean) {
        this.client.trackEvent({
            name: 'PayPal_Webhook_Received',
            properties: {
                eventType,
                success: success.toString(),
                timestamp: new Date().toISOString()
            }
        });
    }

    trackError(error: Error, context: string) {
        this.client.trackException({
            exception: error,
            properties: {
                context,
                timestamp: new Date().toISOString()
            }
        });
    }
}
```

### Alert Configuration Script

Create `scripts/setup-alerts.sh`:

```bash
#!/bin/bash

# Setup monitoring alerts for PayPal integration

RESOURCE_GROUP="ProjectLedger-Production"
APP_SERVICE_NAME="projectledger-api-prod"

# PayPal API Error Rate Alert
az monitor metrics alert create \
    --name "PayPal-High-Error-Rate" \
    --resource-group "$RESOURCE_GROUP" \
    --scopes "/subscriptions/$(az account show --query id -o tsv)/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Web/sites/$APP_SERVICE_NAME" \
    --condition "count 'requests/failed' > 5" \
    --window-size 5m \
    --evaluation-frequency 1m \
    --description "PayPal API error rate is high"

# PayPal Webhook Failure Alert
az monitor metrics alert create \
    --name "PayPal-Webhook-Failures" \
    --resource-group "$RESOURCE_GROUP" \
    --scopes "/subscriptions/$(az account show --query id -o tsv)/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Web/sites/$APP_SERVICE_NAME" \
    --condition "count 'customEvents/PayPal_Webhook_Received' where success == 'false' > 3" \
    --window-size 10m \
    --evaluation-frequency 5m \
    --description "Multiple PayPal webhook failures detected"

# PayPal Response Time Alert
az monitor metrics alert create \
    --name "PayPal-Slow-Response" \
    --resource-group "$RESOURCE_GROUP" \
    --scopes "/subscriptions/$(az account show --query id -o tsv)/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Web/sites/$APP_SERVICE_NAME" \
    --condition "average 'requests/duration' > 5000" \
    --window-size 15m \
    --evaluation-frequency 5m \
    --description "PayPal API response time is slow"

echo "✅ PayPal monitoring alerts configured"
```

---

## 🔙 **5. Rollback Strategy**

### Rollback Script

Create `scripts/rollback-paypal.sh`:

```bash
#!/bin/bash

# PayPal Integration Rollback Script

set -e

RESOURCE_GROUP="ProjectLedger-Production"
APP_SERVICE_NAME="projectledger-api-prod"

echo "🔙 Starting PayPal integration rollback..."

# List available deployment slots
echo "Available deployment slots:"
az webapp deployment slot list \
    --resource-group "$RESOURCE_GROUP" \
    --name "$APP_SERVICE_NAME" \
    --query "[].name" -o table

# Prompt for rollback target
read -p "Enter the deployment slot to rollback to (or 'previous' for last deployment): " ROLLBACK_TARGET

if [ "$ROLLBACK_TARGET" = "previous" ]; then
    # Swap back to previous slot
    az webapp deployment slot swap \
        --resource-group "$RESOURCE_GROUP" \
        --name "$APP_SERVICE_NAME" \
        --slot production \
        --target-slot green
else
    # Swap to specific slot
    az webapp deployment slot swap \
        --resource-group "$RESOURCE_GROUP" \
        --name "$APP_SERVICE_NAME" \
        --slot production \
        --target-slot "$ROLLBACK_TARGET"
fi

# Wait for rollback to complete
sleep 30

# Health check after rollback
BACKEND_URL="https://${APP_SERVICE_NAME}.azurewebsites.net"
HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$BACKEND_URL/health")

if [ "$HTTP_STATUS" = "200" ]; then
    echo "✅ Rollback completed successfully"
else
    echo "❌ Rollback health check failed"
    exit 1
fi

echo "🎉 PayPal integration rollback completed!"
```

---

## 🧪 **6. Production Testing Script**

Create `scripts/test-production.sh`:

```bash
#!/bin/bash

# Production PayPal Integration Testing Script

set -e

BACKEND_URL="https://projectledger-api-prod.azurewebsites.net"
FRONTEND_URL="https://app.projectledger.com"

echo "🧪 Testing PayPal integration in production..."

# Test 1: Health Check
echo "Testing health endpoint..."
HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$BACKEND_URL/health")
if [ "$HTTP_STATUS" = "200" ]; then
    echo "✅ Health check passed"
else
    echo "❌ Health check failed"
    exit 1
fi

# Test 2: PayPal Service Status
echo "Testing PayPal service status..."
PAYPAL_STATUS=$(curl -s "$BACKEND_URL/api/status/paypal" | jq -r '.status')
if [ "$PAYPAL_STATUS" = "ready" ]; then
    echo "✅ PayPal service ready"
else
    echo "❌ PayPal service not ready"
    exit 1
fi

# Test 3: Webhook Endpoint
echo "Testing webhook endpoint..."
HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" -X POST "$BACKEND_URL/api/webhooks/paypal" -H "Content-Type: application/json" -d '{}')
if [ "$HTTP_STATUS" = "400" ]; then
    echo "✅ Webhook endpoint responding (400 expected for empty payload)"
else
    echo "❌ Webhook endpoint not responding correctly"
fi

echo "🎉 Production testing completed!"
```

---

## 📋 **7. Deployment Checklist**

### Pre-Deployment Checklist

- [ ] **Environment Variables**: All production environment variables configured
- [ ] **Database**: Production database migrations tested
- [ ] **PayPal Account**: Production PayPal account configured and verified
- [ ] **SSL Certificates**: Valid SSL certificates installed
- [ ] **Monitoring**: Application Insights and alerts configured
- [ ] **Security**: Security audit checklist completed
- [ ] **Testing**: All tests passing in staging environment
- [ ] **Backup**: Current production backup created
- [ ] **Team Notification**: Team notified of deployment window

### Post-Deployment Checklist

- [ ] **Health Checks**: All health checks passing
- [ ] **PayPal Integration**: PayPal service responding correctly
- [ ] **Webhook URL**: PayPal webhook URL configured in PayPal dashboard
- [ ] **Monitoring**: Monitoring dashboards showing green status
- [ ] **Error Rates**: Error rates within acceptable limits
- [ ] **Performance**: Response times within SLA
- [ ] **User Testing**: Basic user flow tested in production
- [ ] **Documentation**: Deployment documented
- [ ] **Team Notification**: Team notified of successful deployment

---

## 🚨 **8. Emergency Procedures**

### Immediate Rollback (Critical Issues)

```bash
# Emergency rollback command
az webapp deployment slot swap \
    --resource-group "ProjectLedger-Production" \
    --name "projectledger-api-prod" \
    --slot production \
    --target-slot green
```

### Emergency Contacts

- **DevOps Team**: devops@projectledger.com
- **Engineering Team**: engineering@projectledger.com
- **PayPal Technical Support**: PayPal Business Account Portal
- **Azure Support**: Azure Support Portal

### Incident Response

1. **Immediate**: Execute rollback if critical issues detected
2. **Short-term**: Investigate root cause and implement fix
3. **Long-term**: Update deployment process to prevent recurrence

---

**Document Version**: 1.0  
**Last Updated**: September 16, 2025  
**Next Review**: After each deployment