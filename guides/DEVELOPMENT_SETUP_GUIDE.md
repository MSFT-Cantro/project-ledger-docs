# Development Environment Setup Guide

## 🚀 Quick Start

Your ProjectLedger development environment is fully configured and ready to use!

## 📋 Environment Status

### ✅ Applications Running
- **Main Application**: http://localhost:3000
- **Backend API**: http://localhost:3001
- **Client Portal**: http://localhost:3002
- **Storybook (Design System)**: http://localhost:6006
- **PostgreSQL Database**: localhost:5432

### ✅ Database Status
All tables created and seeded with reference data:
- **Users**: 4 user accounts
- **Accounts**: 3 company accounts
- **Clients**: 3 sample clients
- **Subscription Plans**: 3 (Free, Professional, Enterprise)
- **Projects**: Sample project data
- **Quote Templates**: Sample templates

## 🔐 Login Credentials

### Regular Application Login
**URL**: http://localhost:3000/login

#### Available User Accounts:
1. **Admin User**
   - **Email**: `admin@projectledger.com`
   - **Password**: `admin123`
   - **Role**: ADMIN
   - **Account**: Default Account

2. **Demo User**
   - **Email**: `demo@projectledger.com`
   - **Password**: `demo123`
   - **Role**: USER
   - **Account**: Default Account

3. **Test User**
   - **Email**: `test@company.com`
   - **Password**: `test123`
   - **Role**: ADMIN
   - **Account**: Test Company

4. **Test Manager**
   - **Email**: `manager@company.com`
   - **Password**: `manager123`
   - **Role**: USER
   - **Account**: Test Company

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

### Design System & Documentation
- **Storybook**: http://localhost:6006

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

### User Accounts
1. **Admin User** (admin@projectledger.com)
   - Full admin access to all features
   - Password: admin123
   - Account: Default Account

2. **Demo User** (demo@projectledger.com)
   - Standard user permissions
   - Password: demo123
   - Account: Default Account

3. **Test User** (test@company.com)
   - Admin access for Test Company
   - Password: test123
   - Account: Test Company

4. **Test Manager** (manager@company.com)
   - Manager role for Test Company
   - Password: manager123
   - Account: Test Company

### Company Accounts
1. **Default Account** - Main system account
2. **Test Company** - Sample company account
3. **Additional Test Account** - Third sample account

### Sample Business Documents

#### Quotes & Estimates
- **QUO-2024-001**: Website redesign quote for Acme Corporation ($50,850) - SENT
- **QUO-2024-002**: E-commerce setup quote for Small Business Co ($13,560) - DRAFT  
- **EST-2024-001**: Mobile app development estimate for Tech Innovations Inc ($79,100) - DRAFT with 10% contingency

#### Invoices
- **INV-2024-001**: Design phase invoice for Acme Corporation ($16,950) - PAID

#### Change Orders
- **CO-2024-001**: SEO optimization addition (+$5,000) for website project - DRAFT
- **CO-2024-002**: Extended mobile testing (+$1,500) for website project - SENT

### Sample Clients
- **Acme Corporation** (contact@acme.com) - Default Account
- **Tech Innovations Inc** (hello@techinnovations.com) - Default Account  
- **Small Business Co** (info@smallbiz.com) - Test Company

### Sample Projects
- **Website Redesign** - For Acme Corporation (In Progress, $50,000 budget)
- **Mobile App Development** - For Tech Innovations Inc (Planning, $75,000 budget)
- **E-commerce Setup** - For Small Business Co (Planning, $15,000 budget)

## 🔧 API Testing

### Test User Login
```bash
curl -X POST http://localhost:3001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@projectledger.com","password":"admin123"}'
```

### Test Demo User Login
```bash
curl -X POST http://localhost:3001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"demo@projectledger.com","password":"demo123"}'
```

## 🐳 Docker Commands

### Start All Services
```bash
cd /c/Code/project-ledger-app
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
project-ledger-app/
├── apps/
│   ├── backend/           # Express.js API server
│   └── frontend/          # React.js main application & portal
├── packages/              # Shared packages
│   └── shared-types/      # TypeScript type definitions
├── docker-compose.yml     # Docker services configuration
└── README.md             # Project documentation
```

## 🔍 Troubleshooting

### User Login Issues
If you can't log into the main application:

1. **Check the correct URL**: http://localhost:3000/login
2. **Use correct credentials**:
   - Email: `admin@projectledger.com`
   - Password: `admin123`
3. **Check if backend is running**: http://localhost:3001/health
4. **Test login endpoint directly**:
   ```bash
   curl -X POST http://localhost:3001/api/auth/login \
     -H "Content-Type: application/json" \
     -d '{"email":"admin@projectledger.com","password":"admin123"}'
   ```

### Database Access
```bash
# Access PostgreSQL directly
docker-compose exec postgres psql -U postgres -d projectledger

# Check users
SELECT id, email, name, role FROM "User";

# Check accounts
SELECT id, "companyName", "companyEmail" FROM "Account";
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
npx prisma db seed
```

## 📚 Development Features

### Phase 1 & 2 Completed ✅
- Complete user authentication & authorization system
- Multi-tenant account management
- User management interface
- Design system with Storybook integration
- Material-UI responsive design
- Role-based access control
- Project and client management foundation

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

1. **Login to main application**: http://localhost:3000/login
2. **Explore design system**: http://localhost:6006 (Storybook)
3. **Test user roles**: Try different user accounts (admin, demo, test, manager)
4. **Explore account management**: Create/edit accounts and users
5. **Test business documents**: 
   - View existing quotes: QUO-2024-001, QUO-2024-002
   - Review estimate: EST-2024-001 (with contingency)
   - Check invoice: INV-2024-001 (completed payment)
   - Examine change orders: CO-2024-001, CO-2024-002
6. **Create new sample data**: Use the interface to add more quotes, invoices, projects
7. **Test workflows**: Quote approval, invoice generation, change order processing

## 📞 Support

If you encounter any issues:
1. Check this documentation first
2. Verify all containers are running: `docker-compose ps`
3. Check application logs: `docker-compose logs -f`
4. Ensure correct URLs and credentials are being used

---

**Last Updated**: November 7, 2025  
**Environment**: Development  
**Status**: ✅ Ready for Development