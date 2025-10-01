# Quick Start - Azure Local Test Environment

## 🚀 Start Services

```bash
docker-compose -f docker-compose.azure-test.yml up -d
```

## 🌐 Access Points

| Service | URL | Purpose |
|---------|-----|---------|
| Frontend | http://localhost:8080 | Main application |
| Backend API | http://localhost:8081 | Direct API access |
| Nginx Proxy | http://localhost:8082 | Azure Gateway simulation |
| Database | localhost:5433 | PostgreSQL |

## 🧪 Test Endpoints

### Public (No Auth Required)
```bash
# Subscription plans
curl http://localhost:8080/api/subscriptions/plans/public

# Signup page
curl http://localhost:8080/signup

# Login page
curl http://localhost:8080/login
```

### Protected (Auth Required)
```bash
# Current subscription (needs auth token)
curl -H "Authorization: Bearer YOUR_TOKEN" http://localhost:8080/api/subscriptions/current
```

## 📊 Database Access

```bash
# Connect to database
docker-compose -f docker-compose.azure-test.yml exec postgres psql -U postgres -d projectledger_azure_test

# Check subscription plans
docker-compose -f docker-compose.azure-test.yml exec postgres psql -U postgres -d projectledger_azure_test -c "SELECT * FROM \"SubscriptionPlan\";"
```

## 🔧 Common Commands

### View Logs
```bash
# All services
docker-compose -f docker-compose.azure-test.yml logs -f

# Specific service
docker-compose -f docker-compose.azure-test.yml logs -f frontend
docker-compose -f docker-compose.azure-test.yml logs -f backend
```

### Restart Services
```bash
# All services
docker-compose -f docker-compose.azure-test.yml restart

# Specific service
docker-compose -f docker-compose.azure-test.yml restart frontend
```

### Rebuild After Code Changes
```bash
# Frontend
docker-compose -f docker-compose.azure-test.yml build --no-cache frontend
docker-compose -f docker-compose.azure-test.yml up -d frontend

# Backend
docker-compose -f docker-compose.azure-test.yml build --no-cache backend
docker-compose -f docker-compose.azure-test.yml up -d backend
```

### Stop Services
```bash
# Stop all
docker-compose -f docker-compose.azure-test.yml down

# Stop and remove volumes (fresh start)
docker-compose -f docker-compose.azure-test.yml down -v
```

## 🐛 Troubleshooting

### Check Service Status
```bash
docker-compose -f docker-compose.azure-test.yml ps
```

### Check Service Health
```bash
# Backend health
curl http://localhost:8081/health

# Database health
docker-compose -f docker-compose.azure-test.yml exec postgres pg_isready -U postgres
```

### Common Issues

**Issue:** Frontend returns 502 Bad Gateway
```bash
# Check backend logs
docker-compose -f docker-compose.azure-test.yml logs backend --tail=50

# Restart backend
docker-compose -f docker-compose.azure-test.yml restart backend
```

**Issue:** API returns empty subscription plans
```bash
# Seed database
docker-compose -f docker-compose.azure-test.yml exec backend npx ts-node prisma/seed-subscriptions.ts
```

**Issue:** 401 Unauthorized errors on signup page
```bash
# This was fixed - verify you have the latest build
docker-compose -f docker-compose.azure-test.yml build --no-cache frontend
docker-compose -f docker-compose.azure-test.yml up -d frontend
```

## 📝 Testing Signup Flow

1. Open browser: http://localhost:8080/signup
2. Fill out account information:
   - Email: test@example.com
   - Password: Test1234!
   - Company: Test Company
   - Address details
3. Click "Continue to Plan Selection"
4. Verify plans display:
   - ✅ Free ($0/month)
   - ✅ Professional ($99.99/month)
5. Select a plan
6. Click "Create Account"

## 🔍 Monitoring

### Watch Backend Requests
```bash
docker-compose -f docker-compose.azure-test.yml logs -f backend | grep -E "POST|GET|401|500"
```

### Monitor Database Queries
```bash
docker-compose -f docker-compose.azure-test.yml exec postgres psql -U postgres -d projectledger_azure_test -c "SELECT pid, query FROM pg_stat_activity WHERE datname = 'projectledger_azure_test';"
```

## 📚 Documentation

- Full fix details: `AZURE_LOCAL_TEST_FIX_SUMMARY.md`
- Deployment guide: `docs/AZURE_DEPLOYMENT_PLAN.md`
- Frontend-backend fixes: `FRONTEND_BACKEND_FIX.md`

## ✅ Status

All systems operational:
- ✅ Frontend (React + Nginx)
- ✅ Backend (Node.js + Express)
- ✅ Database (PostgreSQL)
- ✅ Subscription plans seeded
- ✅ API endpoints working
- ✅ No authentication errors on public pages

**Ready for production deployment!**
