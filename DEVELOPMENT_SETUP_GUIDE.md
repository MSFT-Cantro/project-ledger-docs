# Development Environment Setup Guide

## 🚀 Quick Start

Your ProjectLedger development environment is fully configured and ready to use!

## 📋 Environment Status

### ✅ Applications Running
- **Main Application**: http://localhost:3000
- **Backend API**: http://localhost:3001
- **Client Portal**: http://localhost:3002
- **PostgreSQL Database**: localhost:5432

### ✅ Database Status
All tables created and seeded with reference data:
- **Users**: 1 regular user
- **Admin Users**: 3 internal admin users
- **Clients**: 3 sample clients
- **Inventory**: 7 sample items
- **Countries**: 2 (US, Canada)
- **States/Provinces**: 64 (All US states + Canadian provinces)
- **Tax Types**: 6 (Sales Tax, GST, PST, HST, QST, VAT)
- **Tax Rates**: 4 basic rates
- **Subscription Plans**: 3 (Free, Professional, Enterprise)

## 🔐 Login Credentials

### Regular Application Login
**URL**: http://localhost:3000/login
- **Email**: `testuser@example.com`
- **Password**: `password123`
- **Role**: Admin (full access to main application)
- **Account**: Test Company

### Internal Admin Dashboard Login
**URL**: http://localhost:3000/admin

#### Admin Users Available:
1. **System Administrator**
   - **Email**: `admin@projectledger.com`
   - **Password**: `admin123`
   - **Role**: ADMIN

2. **Super Administrator**
   - **Email**: `superadmin@projectledger.com`
   - **Password**: `admin123`
   - **Role**: SUPERADMIN

3. **Customer Support**
   - **Email**: `support@projectledger.com`
   - **Password**: `admin123`
   - **Role**: SUPPORT

## 🌐 Application URLs

### Main Application (`localhost:3000`)
- **Login**: http://localhost:3000/login
- **Dashboard**: http://localhost:3000/dashboard
- **Projects**: http://localhost:3000/projects
- **Clients**: http://localhost:3000/clients
- **Quotes**: http://localhost:3000/quotes
- **Invoices**: http://localhost:3000/invoices
- **Inventory**: http://localhost:3000/inventory
- **Reports**: http://localhost:3000/reports
- **User Settings**: http://localhost:3000/usersettings
- **Company Settings**: http://localhost:3000/companysettings
- **Billing**: http://localhost:3000/billing

### Internal Admin Dashboard (`localhost:3000/admin`)
- **Login**: http://localhost:3000/admin/login
- **Dashboard**: http://localhost:3000/admin/dashboard
- **Accounts Management**: http://localhost:3000/admin/accounts
- **Users Management**: http://localhost:3000/admin/users

### Client Portal (`localhost:3002`)
- **Login**: http://localhost:3002/portal/login
- **Dashboard**: http://localhost:3002/portal/dashboard
- **Projects**: http://localhost:3002/portal/projects
- **Quotes**: http://localhost:3002/portal/quotes

### Backend API (`localhost:3001`)
- **Regular Login**: `POST /api/auth/login`
- **Admin Login**: `POST /api/login-admin`
- **Health Check**: `GET /health`

## 📊 Sample Data Available

### Clients
1. **Acme Construction Corp**
   - Email: contact@acmeconstruction.com
   - Phone: 555-0123
   - Contact: John Smith

2. **Downtown Office Building**
   - Email: facility@downtownoffice.com
   - Phone: 555-0456
   - Contact: Sarah Johnson

3. **Smith Residence**
   - Email: homeowner@smithfamily.com
   - Phone: 555-0789
   - Contact: Michael Smith

### Inventory Items
1. **Standard Labor Hour** - $75.00/hour (SKU: LABOR-STD)
2. **Premium Labor Hour** - $125.00/hour (SKU: LABOR-PREM)
3. **Consultation Hour** - $150.00/hour (SKU: CONSULT)
4. **Project Management** - $100.00/hour (SKU: PM-SERVICE)
5. **Material - Lumber 2x4** - $8.50/linear foot (SKU: LUMBER-2X4)
6. **Material - Drywall Sheet** - $15.00/sheet (SKU: DRYWALL-4X8)
7. **Equipment Rental - Scissor Lift** - $200.00/day (SKU: RENT-SCISSOR)

## 🔧 API Testing

### Test Regular User Login
```bash
curl -X POST http://localhost:3001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"testuser@example.com","password":"password123"}'
```

### Test Admin Login
```bash
curl -X POST http://localhost:3001/api/login-admin \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@projectledger.com","password":"admin123"}'
```

## 🐳 Docker Commands

### Start All Services
```bash
cd /c/Code/ProjectLedger2
docker-compose up -d
```

### View Running Containers
```bash
docker-compose ps
```

### View Logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f postgres
```

### Stop All Services
```bash
docker-compose down
```

### Rebuild and Start
```bash
docker-compose down
docker-compose build
docker-compose up -d
```

## 📁 Project Structure

```
ProjectLedger2/
├── apps/
│   ├── backend/           # Express.js API server
│   ├── frontend/          # React.js main application
│   └── portal/            # React.js client portal
├── docs/                  # Documentation
├── packages/              # Shared packages
└── tools/                 # Development tools
```

## 🔍 Troubleshooting

### Admin Login Issues
If you can't log into the admin dashboard:

1. **Check the correct URL**: http://localhost:3000/admin
2. **Use correct credentials**:
   - Email: `admin@projectledger.com`
   - Password: `admin123`
3. **Check if backend is running**: http://localhost:3001/health
4. **Check admin login endpoint directly**:
   ```bash
   curl -X POST http://localhost:3001/api/login-admin \
     -H "Content-Type: application/json" \
     -d '{"email":"admin@projectledger.com","password":"admin123"}'
   ```

### Database Access
```bash
# Access PostgreSQL directly
docker-compose exec postgres psql -U postgres -d projectledger

# Check users
SELECT id, email, name, role FROM "InternalUser";
SELECT id, email, name, role FROM "User";
```

### Reset Environment
If you need to start completely fresh:
```bash
# Stop and remove everything
docker-compose down -v

# Start fresh
docker-compose up -d

# Re-run setup scripts
cd apps/backend
node complete-setup.js
```

## 📚 Development Features

### Phase 1 & 2 Completed ✅
- Complete Internal Admin Dashboard
- User authentication & authorization
- Account management system
- User management interface
- Material-UI responsive design
- Role-based access control

### Available for Development
- Quote creation and management
- Invoice generation
- Project tracking
- Client management
- Inventory management
- Reporting system
- Payment processing
- Tax calculations

## 🎯 Next Steps

1. **Login to admin dashboard**: http://localhost:3000/admin
2. **Explore account management**: Create/edit accounts
3. **Manage users**: Add/modify internal users
4. **Test main application**: Login with testuser@example.com
5. **Create quotes/invoices**: Use the sample clients and inventory

## 📞 Support

If you encounter any issues:
1. Check this documentation first
2. Verify all containers are running: `docker-compose ps`
3. Check application logs: `docker-compose logs -f`
4. Ensure correct URLs and credentials are being used

---

**Last Updated**: October 29, 2025  
**Environment**: Development  
**Status**: ✅ Ready for Development