# PayPal Integration Monitoring & Alerting Configuration

## 📊 **Overview**

This document provides comprehensive monitoring and alerting configuration for the PayPal integration, ensuring high availability, performance, and security.

---

## 🔧 **1. Application Insights Configuration**

### TypeScript Monitoring Service

```typescript
// src/monitoring/paypalMonitoring.ts
import { TelemetryClient } from 'applicationinsights';

export interface PayPalMetrics {
  subscriptionCreated: number;
  subscriptionApproved: number;
  subscriptionCancelled: number;
  paymentsProcessed: number;
  webhooksReceived: number;
  webhookFailures: number;
  apiErrors: number;
  responseTime: number;
}

export class PayPalMonitoring {
  private client: TelemetryClient;
  private metrics: PayPalMetrics;

  constructor() {
    this.client = new TelemetryClient(process.env.APPLICATIONINSIGHTS_CONNECTION_STRING);
    this.metrics = this.initializeMetrics();
  }

  private initializeMetrics(): PayPalMetrics {
    return {
      subscriptionCreated: 0,
      subscriptionApproved: 0,
      subscriptionCancelled: 0,
      paymentsProcessed: 0,
      webhooksReceived: 0,
      webhookFailures: 0,
      apiErrors: 0,
      responseTime: 0
    };
  }

  // Subscription Events
  trackSubscriptionCreated(subscriptionId: string, accountId: string, planId: number) {
    this.metrics.subscriptionCreated++;
    
    this.client.trackEvent({
      name: 'PayPal_Subscription_Created',
      properties: {
        subscriptionId,
        accountId,
        planId: planId.toString(),
        timestamp: new Date().toISOString(),
        environment: process.env.NODE_ENV
      },
      measurements: {
        subscriptionCreatedCount: this.metrics.subscriptionCreated
      }
    });

    this.client.trackMetric({
      name: 'PayPal_Subscriptions_Created_Total',
      value: this.metrics.subscriptionCreated
    });
  }

  trackSubscriptionApproved(subscriptionId: string, accountId: string) {
    this.metrics.subscriptionApproved++;
    
    this.client.trackEvent({
      name: 'PayPal_Subscription_Approved',
      properties: {
        subscriptionId,
        accountId,
        timestamp: new Date().toISOString()
      },
      measurements: {
        subscriptionApprovedCount: this.metrics.subscriptionApproved
      }
    });
  }

  trackSubscriptionCancelled(subscriptionId: string, reason?: string) {
    this.metrics.subscriptionCancelled++;
    
    this.client.trackEvent({
      name: 'PayPal_Subscription_Cancelled',
      properties: {
        subscriptionId,
        reason: reason || 'user_request',
        timestamp: new Date().toISOString()
      },
      measurements: {
        subscriptionCancelledCount: this.metrics.subscriptionCancelled
      }
    });
  }

  // Payment Events
  trackPaymentProcessed(paymentId: string, amount: number, currency: string = 'USD') {
    this.metrics.paymentsProcessed++;
    
    this.client.trackEvent({
      name: 'PayPal_Payment_Processed',
      properties: {
        paymentId,
        currency,
        timestamp: new Date().toISOString()
      },
      measurements: {
        paymentAmount: amount,
        paymentsProcessedCount: this.metrics.paymentsProcessed
      }
    });

    this.client.trackMetric({
      name: 'PayPal_Revenue_Total',
      value: amount
    });
  }

  // Webhook Events
  trackWebhookReceived(eventType: string, eventId: string, success: boolean, processingTime: number) {
    this.metrics.webhooksReceived++;
    if (!success) {
      this.metrics.webhookFailures++;
    }

    this.client.trackEvent({
      name: 'PayPal_Webhook_Received',
      properties: {
        eventType,
        eventId,
        success: success.toString(),
        timestamp: new Date().toISOString()
      },
      measurements: {
        processingTime,
        webhooksReceivedCount: this.metrics.webhooksReceived,
        webhookFailureCount: this.metrics.webhookFailures
      }
    });

    this.client.trackMetric({
      name: 'PayPal_Webhook_Processing_Time',
      value: processingTime
    });
  }

  // API Performance
  trackApiCall(endpoint: string, method: string, statusCode: number, responseTime: number) {
    const isError = statusCode >= 400;
    if (isError) {
      this.metrics.apiErrors++;
    }

    this.client.trackDependency({
      name: 'PayPal_API_Call',
      data: `${method} ${endpoint}`,
      duration: responseTime,
      success: !isError,
      properties: {
        endpoint,
        method,
        statusCode: statusCode.toString()
      }
    });

    this.client.trackMetric({
      name: 'PayPal_API_Response_Time',
      value: responseTime
    });

    if (isError) {
      this.client.trackMetric({
        name: 'PayPal_API_Errors_Total',
        value: this.metrics.apiErrors
      });
    }
  }

  // Error Tracking
  trackError(error: Error, context: string, subscriptionId?: string) {
    this.metrics.apiErrors++;
    
    this.client.trackException({
      exception: error,
      properties: {
        context,
        subscriptionId: subscriptionId || 'unknown',
        timestamp: new Date().toISOString(),
        environment: process.env.NODE_ENV
      },
      measurements: {
        errorCount: this.metrics.apiErrors
      }
    });
  }

  // Custom Metrics
  trackCustomMetric(name: string, value: number, properties?: Record<string, string>) {
    this.client.trackMetric({
      name: `PayPal_${name}`,
      value,
      properties
    });
  }

  // Batch flush for performance
  flush() {
    this.client.flush();
  }

  // Get current metrics
  getMetrics(): PayPalMetrics {
    return { ...this.metrics };
  }
}

// Singleton instance
export const paypalMonitoring = new PayPalMonitoring();
```

### Integration with PayPal Service

```typescript
// Update src/services/paypalService.ts to include monitoring
import { paypalMonitoring } from '../monitoring/paypalMonitoring';

export class PayPalService {
  // ... existing code ...

  async createSubscription(request: PayPalSubscriptionRequest): Promise<PayPalSubscriptionResponse> {
    const startTime = Date.now();
    
    try {
      // ... existing implementation ...
      
      const responseTime = Date.now() - startTime;
      paypalMonitoring.trackApiCall('/v1/billing/subscriptions', 'POST', 201, responseTime);
      paypalMonitoring.trackSubscriptionCreated(result.subscriptionId, request.accountId.toString(), request.planId);
      
      return result;
    } catch (error) {
      const responseTime = Date.now() - startTime;
      paypalMonitoring.trackApiCall('/v1/billing/subscriptions', 'POST', 500, responseTime);
      paypalMonitoring.trackError(error as Error, 'createSubscription', request.accountId?.toString());
      throw error;
    }
  }

  async handleWebhook(event: PayPalWebhookEvent): Promise<void> {
    const startTime = Date.now();
    
    try {
      // ... existing implementation ...
      
      const processingTime = Date.now() - startTime;
      paypalMonitoring.trackWebhookReceived(event.event_type, event.id, true, processingTime);
    } catch (error) {
      const processingTime = Date.now() - startTime;
      paypalMonitoring.trackWebhookReceived(event.event_type, event.id, false, processingTime);
      paypalMonitoring.trackError(error as Error, 'handleWebhook', event.resource?.id);
      throw error;
    }
  }
}
```

---

## 🚨 **2. Alert Rules Configuration**

### Azure Monitor Alert Rules Script

```bash
#!/bin/bash

# PayPal Integration Alert Rules Setup
# This script creates comprehensive monitoring alerts for PayPal integration

RESOURCE_GROUP="ProjectLedger-Production"
APP_SERVICE_NAME="projectledger-api-prod"
SUBSCRIPTION_ID=$(az account show --query id -o tsv)
SCOPE="/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.Web/sites/$APP_SERVICE_NAME"

echo "🚨 Setting up PayPal integration alerts..."

# 1. High Error Rate Alert
az monitor metrics alert create \
    --name "PayPal-High-Error-Rate" \
    --resource-group "$RESOURCE_GROUP" \
    --scopes "$SCOPE" \
    --condition "count 'requests/failed' > 10" \
    --window-size 5m \
    --evaluation-frequency 1m \
    --severity 2 \
    --description "PayPal API error rate exceeds threshold" \
    --action-group "paypal-critical-alerts"

# 2. Webhook Processing Failures
az monitor metrics alert create \
    --name "PayPal-Webhook-Failures" \
    --resource-group "$RESOURCE_GROUP" \
    --scopes "$SCOPE" \
    --condition "count 'customEvents/PayPal_Webhook_Received' where success == 'false' > 3" \
    --window-size 10m \
    --evaluation-frequency 5m \
    --severity 2 \
    --description "Multiple PayPal webhook processing failures" \
    --action-group "paypal-critical-alerts"

# 3. Slow API Response Times
az monitor metrics alert create \
    --name "PayPal-Slow-API-Response" \
    --resource-group "$RESOURCE_GROUP" \
    --scopes "$SCOPE" \
    --condition "average 'customMetrics/PayPal_API_Response_Time' > 5000" \
    --window-size 15m \
    --evaluation-frequency 5m \
    --severity 3 \
    --description "PayPal API response times are degraded" \
    --action-group "paypal-warning-alerts"

# 4. No Webhook Activity (Possible PayPal Outage)
az monitor metrics alert create \
    --name "PayPal-No-Webhook-Activity" \
    --resource-group "$RESOURCE_GROUP" \
    --scopes "$SCOPE" \
    --condition "count 'customEvents/PayPal_Webhook_Received' < 1" \
    --window-size 1h \
    --evaluation-frequency 15m \
    --severity 3 \
    --description "No PayPal webhook activity detected - possible outage" \
    --action-group "paypal-warning-alerts"

# 5. High Subscription Cancellation Rate
az monitor metrics alert create \
    --name "PayPal-High-Cancellation-Rate" \
    --resource-group "$RESOURCE_GROUP" \
    --scopes "$SCOPE" \
    --condition "count 'customEvents/PayPal_Subscription_Cancelled' > 10" \
    --window-size 1h \
    --evaluation-frequency 15m \
    --severity 3 \
    --description "High PayPal subscription cancellation rate" \
    --action-group "paypal-business-alerts"

# 6. Payment Processing Failures
az monitor metrics alert create \
    --name "PayPal-Payment-Failures" \
    --resource-group "$RESOURCE_GROUP" \
    --scopes "$SCOPE" \
    --condition "count 'exceptions/any' where problemId contains 'PayPal' > 5" \
    --window-size 10m \
    --evaluation-frequency 5m \
    --severity 2 \
    --description "PayPal payment processing failures detected" \
    --action-group "paypal-critical-alerts"

# 7. Revenue Drop Alert
az monitor metrics alert create \
    --name "PayPal-Revenue-Drop" \
    --resource-group "$RESOURCE_GROUP" \
    --scopes "$SCOPE" \
    --condition "sum 'customMetrics/PayPal_Revenue_Total' < 1000" \
    --window-size 24h \
    --evaluation-frequency 1h \
    --severity 3 \
    --description "PayPal revenue below expected threshold" \
    --action-group "paypal-business-alerts"

echo "✅ PayPal alert rules configured successfully"
```

### Action Groups Configuration

```bash
#!/bin/bash

# PayPal Alert Action Groups Configuration

RESOURCE_GROUP="ProjectLedger-Production"

# Critical Alerts Action Group (PagerDuty, SMS, Email)
az monitor action-group create \
    --name "paypal-critical-alerts" \
    --resource-group "$RESOURCE_GROUP" \
    --short-name "PayPalCrit" \
    --email-receivers \
        devops@projectledger.com "DevOps Team" \
        oncall@projectledger.com "On-Call Engineer" \
    --sms-receivers \
        "+1-555-0123" "DevOps Lead" \
    --webhook-receivers \
        "PagerDuty" "https://events.pagerduty.com/integration/paypal-critical/enqueue"

# Warning Alerts Action Group (Email, Slack)
az monitor action-group create \
    --name "paypal-warning-alerts" \
    --resource-group "$RESOURCE_GROUP" \
    --short-name "PayPalWarn" \
    --email-receivers \
        engineering@projectledger.com "Engineering Team" \
    --webhook-receivers \
        "Slack" "https://hooks.slack.com/services/paypal-warnings"

# Business Alerts Action Group (Email to Business Team)
az monitor action-group create \
    --name "paypal-business-alerts" \
    --resource-group "$RESOURCE_GROUP" \
    --short-name "PayPalBiz" \
    --email-receivers \
        business@projectledger.com "Business Team" \
        finance@projectledger.com "Finance Team"

echo "✅ PayPal action groups configured"
```

---

## 📊 **3. Custom Dashboard Configuration**

### Azure Dashboard JSON

```json
{
  "lenses": {
    "0": {
      "order": 0,
      "parts": {
        "0": {
          "position": { "x": 0, "y": 0, "colSpan": 6, "rowSpan": 4 },
          "metadata": {
            "inputs": [
              {
                "name": "resourceTypeMode",
                "isOptional": true
              },
              {
                "name": "ComponentId",
                "value": {
                  "SubscriptionId": "[subscription-id]",
                  "ResourceGroup": "ProjectLedger-Production",
                  "Name": "projectledger-paypal-insights"
                }
              }
            ],
            "type": "Extension/AppInsightsExtension/PartType/AppMapGalPt"
          }
        },
        "1": {
          "position": { "x": 6, "y": 0, "colSpan": 6, "rowSpan": 4 },
          "metadata": {
            "inputs": [
              {
                "name": "query",
                "value": "customEvents\n| where name == 'PayPal_Subscription_Created'\n| summarize count() by bin(timestamp, 1h)\n| render timechart"
              }
            ],
            "type": "Extension/AppInsightsExtension/PartType/AnalyticsGridPt"
          }
        },
        "2": {
          "position": { "x": 0, "y": 4, "colSpan": 4, "rowSpan": 3 },
          "metadata": {
            "inputs": [
              {
                "name": "query",
                "value": "customMetrics\n| where name == 'PayPal_API_Response_Time'\n| summarize avg(value) by bin(timestamp, 5m)\n| render timechart"
              }
            ],
            "type": "Extension/AppInsightsExtension/PartType/AnalyticsGridPt"
          }
        },
        "3": {
          "position": { "x": 4, "y": 4, "colSpan": 4, "rowSpan": 3 },
          "metadata": {
            "inputs": [
              {
                "name": "query",
                "value": "customEvents\n| where name == 'PayPal_Webhook_Received'\n| summarize Success = countif(tostring(customDimensions.success) == 'true'), Failed = countif(tostring(customDimensions.success) == 'false') by bin(timestamp, 1h)\n| render timechart"
              }
            ],
            "type": "Extension/AppInsightsExtension/PartType/AnalyticsGridPt"
          }
        },
        "4": {
          "position": { "x": 8, "y": 4, "colSpan": 4, "rowSpan": 3 },
          "metadata": {
            "inputs": [
              {
                "name": "query",
                "value": "customMetrics\n| where name == 'PayPal_Revenue_Total'\n| summarize sum(value) by bin(timestamp, 1d)\n| render barchart"
              }
            ],
            "type": "Extension/AppInsightsExtension/PartType/AnalyticsGridPt"
          }
        }
      }
    }
  },
  "metadata": {
    "model": {
      "timeRange": {
        "value": {
          "relative": {
            "duration": 24,
            "timeUnit": 1
          }
        },
        "type": "MsPortalFx.Composition.Configuration.ValueTypes.TimeRange"
      }
    }
  }
}
```

---

## 📈 **4. Health Check Endpoints**

### PayPal Health Check Service

```typescript
// src/services/healthCheck.ts
import { PayPalService } from './paypalService';
import { paypalMonitoring } from '../monitoring/paypalMonitoring';

export interface HealthCheckResult {
  status: 'healthy' | 'degraded' | 'unhealthy';
  timestamp: string;
  checks: {
    paypal: HealthCheck;
    database: HealthCheck;
    webhooks: HealthCheck;
  };
  metrics: {
    responseTime: number;
    uptime: number;
  };
}

export interface HealthCheck {
  status: 'pass' | 'warn' | 'fail';
  message: string;
  responseTime?: number;
}

export class PayPalHealthService {
  private paypalService: PayPalService;

  constructor() {
    this.paypalService = new PayPalService();
  }

  async performHealthCheck(): Promise<HealthCheckResult> {
    const startTime = Date.now();
    
    const checks = await Promise.allSettled([
      this.checkPayPalConnection(),
      this.checkDatabaseConnection(),
      this.checkWebhookEndpoint()
    ]);

    const [paypalCheck, dbCheck, webhookCheck] = checks.map(result => 
      result.status === 'fulfilled' ? result.value : { 
        status: 'fail' as const, 
        message: 'Check failed to execute',
        responseTime: 0
      }
    );

    const responseTime = Date.now() - startTime;
    const overallStatus = this.determineOverallStatus([paypalCheck, dbCheck, webhookCheck]);

    const result: HealthCheckResult = {
      status: overallStatus,
      timestamp: new Date().toISOString(),
      checks: {
        paypal: paypalCheck,
        database: dbCheck,
        webhooks: webhookCheck
      },
      metrics: {
        responseTime,
        uptime: process.uptime()
      }
    };

    // Track health check results
    paypalMonitoring.trackCustomMetric('HealthCheck_ResponseTime', responseTime);
    paypalMonitoring.trackCustomMetric('HealthCheck_Status', overallStatus === 'healthy' ? 1 : 0);

    return result;
  }

  private async checkPayPalConnection(): Promise<HealthCheck> {
    const startTime = Date.now();
    
    try {
      // Simple PayPal API connectivity test
      const response = await fetch(`${this.paypalService['paypalBaseUrl']}/v1/oauth2/token`, {
        method: 'POST',
        headers: {
          'Accept': 'application/json',
          'Accept-Language': 'en_US',
        },
        body: 'grant_type=client_credentials'
      });

      const responseTime = Date.now() - startTime;

      if (response.status === 401) {
        // 401 is expected without credentials - means PayPal is reachable
        return {
          status: 'pass',
          message: 'PayPal API accessible',
          responseTime
        };
      } else {
        return {
          status: 'warn',
          message: `PayPal API returned unexpected status: ${response.status}`,
          responseTime
        };
      }
    } catch (error) {
      return {
        status: 'fail',
        message: `PayPal API connection failed: ${(error as Error).message}`,
        responseTime: Date.now() - startTime
      };
    }
  }

  private async checkDatabaseConnection(): Promise<HealthCheck> {
    const startTime = Date.now();
    
    try {
      // Simple database connectivity test
      await prisma.$queryRaw`SELECT 1`;
      
      return {
        status: 'pass',
        message: 'Database connection healthy',
        responseTime: Date.now() - startTime
      };
    } catch (error) {
      return {
        status: 'fail',
        message: `Database connection failed: ${(error as Error).message}`,
        responseTime: Date.now() - startTime
      };
    }
  }

  private async checkWebhookEndpoint(): Promise<HealthCheck> {
    const startTime = Date.now();
    
    try {
      // Check if webhook endpoint is responsive
      const webhookUrl = `${process.env.API_BASE_URL}/api/webhooks/paypal`;
      const response = await fetch(webhookUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({})
      });

      const responseTime = Date.now() - startTime;

      if (response.status === 400) {
        // 400 is expected for empty webhook - means endpoint is responsive
        return {
          status: 'pass',
          message: 'Webhook endpoint responsive',
          responseTime
        };
      } else {
        return {
          status: 'warn',
          message: `Webhook endpoint returned unexpected status: ${response.status}`,
          responseTime
        };
      }
    } catch (error) {
      return {
        status: 'fail',
        message: `Webhook endpoint check failed: ${(error as Error).message}`,
        responseTime: Date.now() - startTime
      };
    }
  }

  private determineOverallStatus(checks: HealthCheck[]): 'healthy' | 'degraded' | 'unhealthy' {
    const failedChecks = checks.filter(check => check.status === 'fail').length;
    const warnChecks = checks.filter(check => check.status === 'warn').length;

    if (failedChecks > 0) {
      return 'unhealthy';
    } else if (warnChecks > 0) {
      return 'degraded';
    } else {
      return 'healthy';
    }
  }
}

export const paypalHealthService = new PayPalHealthService();
```

### Health Check Route

```typescript
// src/routes/health.ts
import { Router } from 'express';
import { paypalHealthService } from '../services/healthCheck';

const router = Router();

router.get('/health', async (req, res) => {
  try {
    const healthResult = await paypalHealthService.performHealthCheck();
    
    const statusCode = healthResult.status === 'healthy' ? 200 : 
                      healthResult.status === 'degraded' ? 200 : 503;
    
    res.status(statusCode).json(healthResult);
  } catch (error) {
    res.status(503).json({
      status: 'unhealthy',
      timestamp: new Date().toISOString(),
      error: 'Health check failed to execute'
    });
  }
});

router.get('/health/paypal', async (req, res) => {
  try {
    const healthResult = await paypalHealthService.performHealthCheck();
    res.json({
      status: healthResult.checks.paypal.status,
      message: healthResult.checks.paypal.message,
      responseTime: healthResult.checks.paypal.responseTime
    });
  } catch (error) {
    res.status(503).json({
      status: 'fail',
      message: 'PayPal health check failed'
    });
  }
});

export default router;
```

---

## 📋 **5. Monitoring Checklist**

### Pre-Production Monitoring Setup

- [ ] **Application Insights**: Configured with PayPal-specific tracking
- [ ] **Alert Rules**: All critical alerts configured and tested
- [ ] **Action Groups**: Notification channels configured
- [ ] **Dashboard**: PayPal monitoring dashboard created
- [ ] **Health Checks**: Health check endpoints implemented
- [ ] **Log Analytics**: PayPal-specific log queries created
- [ ] **Performance Counters**: Custom metrics implemented
- [ ] **Error Tracking**: Exception handling with context

### Production Monitoring Validation

- [ ] **Metrics Collection**: Verify metrics are being collected
- [ ] **Alert Testing**: Test alert rules with simulated failures
- [ ] **Dashboard Functionality**: Confirm dashboard displays data correctly
- [ ] **Health Endpoint**: Verify health checks are working
- [ ] **Log Aggregation**: Confirm logs are being aggregated
- [ ] **Notification Delivery**: Test alert notifications
- [ ] **Performance Baseline**: Establish performance baselines
- [ ] **SLA Monitoring**: Configure SLA monitoring and reporting

---

## 🚨 **6. Incident Response Procedures**

### PayPal Service Incident Runbook

#### High Error Rate (> 10 errors in 5 minutes)
1. **Immediate**: Check PayPal service status at status.paypal.com
2. **Investigate**: Review error logs in Application Insights
3. **Escalate**: If PayPal outage, notify business team
4. **Mitigate**: Consider enabling mock mode if extended outage

#### Webhook Processing Failures
1. **Immediate**: Check webhook signature validation
2. **Investigate**: Review webhook payload and headers
3. **Retry**: Manual webhook reprocessing if needed
4. **Fix**: Update webhook processing logic if bug identified

#### Performance Degradation
1. **Monitor**: Check API response times and dependencies
2. **Scale**: Consider scaling up app service resources
3. **Optimize**: Review database query performance
4. **Cache**: Implement caching for frequently accessed data

---

**Document Version**: 1.0  
**Last Updated**: September 16, 2025  
**Next Review**: Monthly or after incidents