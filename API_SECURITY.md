# API Security Documentation

**Last Updated:** October 21, 2025  
**Version:** 1.0

## Overview

This document describes the authentication and authorization mechanisms implemented across the ProjectLedger2 API.

## Authentication Methods

### 1. JWT Token Authentication

**Used For:** Most user-facing endpoints  
**Header:** `Authorization: Bearer <token>`

#### Token Structure
```json
{
  "userId": 123,
  "accountId": 456,
  "role": "ADMIN",
  "iat": 1634567890,
  "exp": 1634571490
}
```

#### Token Expiration
- **Access Token:** 1 hour
- **Refresh Token:** 30 days (if implemented)

#### How to Obtain Token
```bash
POST /api/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password"
}

Response:
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": { ... }
}
```

### 2. API Key Authentication

**Used For:** Metrics endpoints (monitoring tools)  
**Header:** `x-metrics-api-key: <api_key>`

#### How to Generate API Key
```bash
# Using OpenSSL
openssl rand -base64 32

# Using Node.js
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"
```

#### Environment Configuration
```bash
# .env
METRICS_API_KEY=your-generated-key-here
```

## Authorization Roles

### Role Hierarchy

| Role | Access Level | Permissions |
|------|-------------|-------------|
| **ADMIN** | Full access | All operations including admin endpoints |
| **USER** | Standard access | CRUD on own resources |
| **VIEWER** | Read-only | View access only |

### Role-Based Endpoints

#### Admin Endpoints (ADMIN role required)
```
POST /api/admin/seed-database
GET  /api/admin/check-plans
```

## Protected Endpoints

### By JWT Authentication

#### Invoices
```
GET    /api/invoices              - List invoices
GET    /api/invoices/:id          - Get invoice
POST   /api/invoices              - Create invoice
PUT    /api/invoices/:id          - Update invoice
DELETE /api/invoices/:id          - Delete invoice
```

#### Quotes
```
GET    /api/quotes                - List quotes
GET    /api/quotes/:id            - Get quote
POST   /api/quotes                - Create quote
PUT    /api/quotes/:id            - Update quote
DELETE /api/quotes/:id            - Delete quote
```

#### Clients
```
GET    /api/clients               - List clients
GET    /api/clients/:id           - Get client
POST   /api/clients               - Create client
PUT    /api/clients/:id           - Update client
DELETE /api/clients/:id           - Delete client
```

#### Projects
```
GET    /api/projects              - List projects
GET    /api/projects/:id          - Get project
POST   /api/projects              - Create project
PUT    /api/projects/:id          - Update project
DELETE /api/projects/:id          - Delete project
```

#### Admin (ADMIN role required)
```
POST /api/admin/seed-database     - Seed subscription plans
GET  /api/admin/check-plans       - View all subscription plans
```

### By API Key Authentication

#### Metrics (API key required)
```
GET  /metrics/                    - Full application metrics
GET  /metrics/system              - System metrics (CPU, memory)
GET  /metrics/requests            - Request statistics
GET  /metrics/errors              - Error statistics
POST /metrics/reset               - Reset metrics
GET  /metrics/prometheus          - Prometheus format metrics
```

## Public Endpoints

### Authentication
```
POST /api/auth/login              - User login
POST /api/auth/signup             - User registration
POST /api/auth/google             - Google OAuth
POST /api/auth/microsoft          - Microsoft OAuth
GET  /api/auth/google/callback    - Google OAuth callback
GET  /api/auth/microsoft/callback - Microsoft OAuth callback
```

### Health Checks
```
GET  /health                      - Basic health check
GET  /ready                       - Readiness probe
GET  /alive                       - Liveness probe
```

## Error Responses

### 401 Unauthorized
**Reason:** Missing or invalid authentication token/API key

```json
{
  "error": "Authentication required",
  "message": "No authentication token provided"
}
```

**Common Causes:**
- Missing `Authorization` header
- Expired token
- Invalid token signature
- Missing `x-metrics-api-key` header (for metrics endpoints)

### 403 Forbidden
**Reason:** Insufficient permissions for the requested resource

```json
{
  "error": "Insufficient permissions",
  "message": "You do not have permission to access this resource. Required role: ADMIN"
}
```

**Common Causes:**
- USER role attempting to access ADMIN endpoints
- VIEWER role attempting to modify resources
- User accessing another account's resources

## Security Best Practices

### For API Consumers

1. **Store Tokens Securely**
   - Never store tokens in localStorage
   - Use httpOnly cookies when possible
   - Clear tokens on logout

2. **Handle Token Expiration**
   ```typescript
   if (error.response?.status === 401) {
     // Clear stored token
     localStorage.removeItem('authToken');
     // Redirect to login
     window.location.href = '/login?session=expired';
   }
   ```

3. **Rotate API Keys Regularly**
   - Generate new keys every 90 days
   - Update all monitoring tools
   - Revoke old keys after transition

4. **Use HTTPS Only**
   - Never send tokens over HTTP
   - Enforce TLS 1.2 or higher

### For API Developers

1. **Validate All Inputs**
   ```typescript
   router.post('/endpoint', authenticateToken, validateRequest, handler);
   ```

2. **Log Security Events**
   ```typescript
   logger.warn('Authentication failure', {
     ip: req.ip,
     endpoint: req.path,
     timestamp: new Date().toISOString()
   });
   ```

3. **Rate Limit Sensitive Endpoints**
   ```typescript
   const limiter = rateLimit({
     windowMs: 15 * 60 * 1000, // 15 minutes
     max: 100 // limit each IP to 100 requests per windowMs
   });
   
   router.use('/api/auth', limiter);
   ```

## Examples

### Making Authenticated Requests

#### Using cURL
```bash
# JWT Authentication
curl -H "Authorization: Bearer YOUR_TOKEN" \
     http://localhost:3001/api/invoices

# API Key Authentication (metrics)
curl -H "x-metrics-api-key: YOUR_API_KEY" \
     http://localhost:3001/metrics/system
```

#### Using JavaScript (fetch)
```javascript
// JWT Authentication
const response = await fetch('/api/invoices', {
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  }
});

// API Key Authentication (metrics)
const metricsResponse = await fetch('/metrics/system', {
  headers: {
    'x-metrics-api-key': process.env.METRICS_API_KEY
  }
});
```

#### Using Axios
```javascript
// Create axios instance with JWT
const api = axios.create({
  baseURL: 'http://localhost:3001',
  headers: {
    'Authorization': `Bearer ${token}`
  }
});

// Create axios instance with API key
const metricsApi = axios.create({
  baseURL: 'http://localhost:3001/metrics',
  headers: {
    'x-metrics-api-key': process.env.METRICS_API_KEY
  }
});
```

## Troubleshooting

### "Authentication required" Error

**Check:**
1. Token is being sent in the `Authorization` header
2. Token is properly formatted: `Bearer <token>`
3. Token hasn't expired (check `exp` claim)
4. For metrics: API key is in `x-metrics-api-key` header

**Solution:**
```bash
# Verify token
curl -H "Authorization: Bearer YOUR_TOKEN" \
     http://localhost:3001/api/auth/verify

# Check token expiration
echo YOUR_TOKEN | cut -d. -f2 | base64 -d | jq .exp
```

### "Insufficient permissions" Error

**Check:**
1. User role matches endpoint requirement
2. User is accessing their own resources
3. Multi-tenant access is properly configured

**Solution:**
```bash
# Check current user role
curl -H "Authorization: Bearer YOUR_TOKEN" \
     http://localhost:3001/api/auth/me
```

### "Invalid API key" Error (Metrics)

**Check:**
1. `METRICS_API_KEY` environment variable is set
2. Key matches exactly (no extra whitespace)
3. Key is being sent in correct header

**Solution:**
```bash
# Verify API key is set
echo $METRICS_API_KEY

# Test with explicit key
curl -H "x-metrics-api-key: $METRICS_API_KEY" \
     http://localhost:3001/metrics/system
```

## Environment Variables

### Required Security Variables

```bash
# JWT Secret (required)
JWT_SECRET=your-secure-jwt-secret-minimum-32-characters

# Metrics API Key (required for metrics endpoints)
METRICS_API_KEY=your-secure-metrics-api-key

# OAuth (optional)
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
MICROSOFT_CLIENT_ID=your-microsoft-client-id
MICROSOFT_CLIENT_SECRET=your-microsoft-client-secret
```

### Generating Secure Values

```bash
# JWT Secret (64 bytes)
openssl rand -base64 64

# Metrics API Key (32 bytes)
openssl rand -base64 32
```

## Compliance & Auditing

### Audit Logging

All authentication events are logged:
- Successful logins
- Failed login attempts
- Token expiration
- Role-based access denials
- Admin operations (with user ID)
- Metrics access (with IP)

### Log Format
```json
{
  "timestamp": "2025-10-21T12:00:00.000Z",
  "level": "INFO",
  "event": "auth.login.success",
  "userId": 123,
  "ip": "192.168.1.1",
  "userAgent": "Mozilla/5.0..."
}
```

### GDPR Compliance

- Tokens include minimal user data
- Logs can be filtered by user ID for deletion
- Token revocation supported
- User data access via authenticated endpoint

## Support

For security issues or questions:
- **Email:** security@projectledger.com
- **Documentation:** https://docs.projectledger.com/security
- **GitHub Issues:** https://github.com/MSFT-Cantro/project-ledger/issues

**Security Vulnerability Reporting:**
Please report security vulnerabilities privately via email, not through public issues.
