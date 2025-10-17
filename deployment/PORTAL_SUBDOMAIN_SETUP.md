# 🌐 Portal Subdomain Setup Guide

## Overview

The client portal is accessible at `portal.projectledger.com` in production and `portal.localhost:3002` in development. This subdomain-based setup provides a clean separation between the main application and the client portal.

## 🏗️ Architecture

### Production Setup
- **Main App**: `app.projectledger.ca` (admin/user interface)
- **Portal**: `portal.projectledger.com` (client interface)
- **API**: `api.projectledger.ca` (shared backend)

### Development Setup
- **Main App**: `http://localhost:3000`
- **Portal**: `http://localhost:3002`
- **API**: `http://localhost:3001`

## 🚀 Development Usage

### Start Services
```bash
# Start all services including portal
docker-compose up -d

# Portal will be available at:
# http://localhost:3002
```

### Access Points
| Service | URL | Purpose |
|---------|-----|---------|
| Main App | http://localhost:3000 | Admin/user interface |
| Portal | http://localhost:3002 | Client portal |
| Backend API | http://localhost:3001 | Shared API |

### Test Portal
```bash
# Test portal health
curl http://localhost:3002/health

# Test portal login page
curl http://localhost:3002/login
```

## 🌍 Production Deployment

### DNS Configuration

Add DNS records for the portal subdomain:

#### For projectledger.com:
```
Type: CNAME
Name: portal
Value: projectledger-portal.azurecontainer.io
TTL: 300
```

### Deploy Portal
```bash
# Build and deploy portal service
docker-compose -f docker-compose.prod.yml up -d portal

# Verify deployment
curl https://portal.projectledger.com/health
```

### SSL Certificate

The portal subdomain automatically gets SSL certificates via Let's Encrypt through Traefik.

## 🔧 Configuration Files

### Key Files
- `apps/frontend/nginx.portal.prod.conf` - Portal nginx config for production
- `apps/frontend/nginx.portal.azure.conf` - Portal nginx config for Azure
- `apps/frontend/src/routes/portalRoutes.tsx` - Portal-specific routes
- `apps/frontend/src/PortalApp.tsx` - Portal application entry point
- `apps/frontend/src/utils/portalRouting.ts` - Portal routing utilities

### Environment Variables

#### Portal-specific variables:
```bash
REACT_APP_PORTAL_MODE=true          # Enables portal mode
REACT_APP_API_URL=https://api.projectledger.ca
```

#### Backend CORS update:
```bash
CORS_ORIGIN=https://app.projectledger.ca,https://portal.projectledger.com
```

## 🔒 Security Considerations

### CORS Configuration
The backend is configured to accept requests from:
- `https://app.projectledger.ca` (main app)
- `https://portal.projectledger.com` (portal)

### CSP Headers
Portal has its own Content Security Policy configured in nginx:
- Allows connections to the API backend
- Restricts frame embedding
- Allows necessary fonts and stylesheets

### Authentication
- Portal uses separate JWT tokens with `type: 'client-portal'`
- Tokens have shorter expiration (8 hours vs 24 hours for main app)
- Portal sessions are isolated from main app sessions

## 🛠️ Troubleshooting

### Portal Not Loading
```bash
# Check portal container status
docker ps | grep portal

# Check portal logs
docker logs projectledger-portal-1

# Check nginx config
docker exec projectledger-portal-1 nginx -t
```

### API Connection Issues
```bash
# Test backend connectivity from portal
docker exec projectledger-portal-1 curl http://backend:3001/api/health

# Check CORS configuration
curl -H "Origin: https://portal.projectledger.com" https://api.projectledger.ca/api/health
```

### DNS Issues
```bash
# Test DNS resolution
nslookup portal.projectledger.com

# Test from local machine
curl -I https://portal.projectledger.com
```

## 📝 Routing Changes

### Portal Routes (Subdomain-based)
- `/login` (instead of `/portal/login`)
- `/dashboard` (instead of `/portal/dashboard`)  
- `/projects` (instead of `/portal/projects`)
- `/projects/:id` (instead of `/portal/projects/:id`)

### Migration Notes
- Existing portal URLs with `/portal` prefix still work in the main app
- New subdomain uses clean URLs without `/portal` prefix
- `portalRouting.ts` utility handles conditional routing based on environment

## 🔄 Deployment Checklist

### DNS Setup
- [ ] Create CNAME record for `portal.projectledger.com`
- [ ] Verify DNS propagation

### Container Deployment  
- [ ] Build portal container with correct target
- [ ] Deploy to container registry
- [ ] Update container app configuration
- [ ] Verify health endpoints

### SSL & Security
- [ ] Verify Let's Encrypt certificate generation
- [ ] Test HTTPS redirects
- [ ] Validate CORS configuration
- [ ] Test CSP headers

### Functionality Testing
- [ ] Portal login flow
- [ ] Account selection (multi-account clients)
- [ ] Dashboard access
- [ ] Project viewing
- [ ] Password reset flow

## 📞 Support

For issues with portal subdomain setup:
1. Check container logs: `docker logs projectledger-portal-1`
2. Verify DNS resolution: `nslookup portal.projectledger.com`
3. Test API connectivity: `curl https://api.projectledger.ca/api/health`
4. Check nginx configuration: `docker exec projectledger-portal-1 nginx -t`