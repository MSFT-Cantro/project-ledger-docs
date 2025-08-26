# Project Ledger Deployment Guide

## System Requirements

### Core Requirements
- Node.js >= 16.0.0
- PostgreSQL >= 12.0
- npm or yarn package manager
- Git

### Frontend Dependencies
- React 19.1.x
- Material-UI (MUI) 7.3.x
- React Query 5.85.x
- TypeScript 4.9.x

### Backend Dependencies
- Express 4.18.x
- Prisma ORM 5.3.x
- TypeScript 5.2.x
- Node.js >= 16.0.0

### Development Tools
- Visual Studio Code (recommended)
- PostgreSQL client (e.g., pgAdmin, DBeaver)
- Postman or similar API testing tool
- Git client

### Required Services
- PostgreSQL database server
- SMTP server for email notifications
- Stripe account for payment processing

### Environment Requirements

#### Frontend Environment Variables
```
REACT_APP_STRIPE_PUBLISHABLE_KEY - Stripe public key
REACT_APP_API_URL - Backend API URL
```

#### Backend Environment Variables
```
PORT - Server port (default: 3001)
NODE_ENV - Environment (development/production)
DATABASE_URL - PostgreSQL connection string
STRIPE_SECRET_KEY - Stripe secret key
STRIPE_WEBHOOK_SECRET - Stripe webhook signing secret
CORS_ORIGIN - Allowed CORS origin
SMTP_* - Email server configuration
FRONTEND_URL - Frontend application URL
```

### Hardware Requirements

#### Development
- CPU: Dual-core processor or better
- RAM: 8GB minimum, 16GB recommended
- Storage: 1GB free space for project files and dependencies

#### Production (Recommended)
- CPU: 4+ cores
- RAM: 16GB minimum
- Storage: 10GB+ for application, logs, and database
- Network: Reliable internet connection with minimum 10Mbps

## Environment Setup

### Project Structure
```
project-ledger/
├── docs/                    # Project documentation
├── project-ledger/         # Frontend application
│   ├── public/             # Static assets
│   ├── src/
│   │   ├── api/           # API integration layers
│   │   ├── components/    # React components
│   │   ├── context/       # React context providers
│   │   ├── routes/        # Application routing
│   │   ├── services/      # Business logic services
│   │   ├── theme/         # Styling and theming
│   │   └── types/         # TypeScript definitions
│   ├── package.json       # Frontend dependencies
│   └── tsconfig.json      # TypeScript configuration
└── server/                # Backend application
    ├── prisma/           # Database schema and migrations
    ├── src/
    │   ├── handlers/     # Request handlers
    │   ├── services/     # Business logic
    │   └── types/        # TypeScript definitions
    ├── package.json      # Backend dependencies
    └── tsconfig.json     # TypeScript configuration
```

### Frontend Setup

1. Navigate to the frontend directory:
   ```bash
   cd project-ledger
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Create environment file:
   ```bash
   cp .env.example .env
   ```

4. Configure environment variables in `.env`:
   - Set `REACT_APP_API_URL` to your backend server URL
   - Add Stripe publishable key if using payments

5. Start development server:
   ```bash
   npm start
   ```

### Backend Setup

1. Navigate to the server directory:
   ```bash
   cd server
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Create environment file:
   ```bash
   cp .env.example .env
   ```

4. Configure environment variables in `.env`:
   - Set up database connection URL
   - Configure email settings
   - Add Stripe secret key and webhook secret

5. Generate Prisma client:
   ```bash
   npm run prisma:generate
   ```

6. Run database migrations:
   ```bash
   npm run prisma:migrate
   ```

7. Start development server:
   ```bash
   npm run dev
   ```

### Development Environment

1. VSCode Extensions (recommended):
   - ESLint
   - Prettier
   - TypeScript and JavaScript Language Features
   - Prisma
   - GitLens

2. Browser Extensions:
   - React Developer Tools
   - Redux DevTools (if using Redux)

3. Development Tools:
   - Git for version control
   - Node.js package manager (npm or yarn)
   - PostgreSQL database client

4. Local Services:
   - PostgreSQL database server running
   - SMTP server configured (or use a service like Mailtrap for testing)
   - Stripe CLI for webhook testing

## Configuration Guide

### Configuration Files

1. Frontend Configuration Files
   - `.env` - Environment variables
   - `src/theme/theme.ts` - UI theme configuration
   - `package.json` - Project dependencies and scripts
   - `tsconfig.json` - TypeScript configuration

2. Backend Configuration Files
   - `.env` - Environment variables
   - `prisma/schema.prisma` - Database schema
   - `package.json` - Project dependencies and scripts
   - `tsconfig.json` - TypeScript configuration

### Environment Variables

#### Frontend Variables
```dotenv
# API Configuration
REACT_APP_API_URL=http://localhost:3001
# Default: http://localhost:3001
# Production: Your production API URL

# Stripe Configuration
REACT_APP_STRIPE_PUBLISHABLE_KEY=pk_test_your_key
# Required for payment processing
# Get from Stripe Dashboard
```

#### Backend Variables
```dotenv
# Server Configuration
PORT=3001
NODE_ENV=development
# Options: development, production, test

# Database Configuration
DATABASE_URL=postgresql://user:password@localhost:5432/project_ledger
# Format: postgresql://USER:PASSWORD@HOST:PORT/DATABASE

# Security
CORS_ORIGIN=http://localhost:3000
# Frontend URL for CORS
# Multiple origins: Use comma-separated values

# Email Configuration
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_SECURE=false
SMTP_USER=your_email@example.com
SMTP_PASS=your_smtp_password
SMTP_FROM=noreply@example.com
# Required for email notifications
# Use Mailtrap for development

# Frontend URL
FRONTEND_URL=http://localhost:3000
# Used for email links and redirects
```

### Configuration Options

1. Database Options
   - Connection pool size (default: 10)
   - SSL mode (required for some providers)
   - Schema name (default: public)

2. Email Settings
   - SMTP or API-based providers
   - TLS/SSL security options
   - HTML/Text email templates
   - Rate limiting settings

3. Security Options
   - CORS configuration
   - Rate limiting
   - Request timeouts
   - Maximum request size

4. Application Settings
   - Pagination limits
   - File upload restrictions
   - Session timeouts
   - Cache settings

### Configuration Best Practices

1. Environment Variables
   - Never commit .env files to version control
   - Use different values for development/staging/production
   - Regularly rotate sensitive credentials
   - Use strong, unique values for security keys

2. Security
   - Enable CORS only for trusted origins
   - Use HTTPS in production
   - Set secure headers (implemented via Helmet)
   - Rate limit sensitive endpoints

3. Performance
   - Configure appropriate connection pool sizes
   - Set reasonable request timeouts
   - Enable compression for responses
   - Configure caching headers

4. Monitoring
   - Set up logging levels
   - Configure error reporting
   - Enable performance monitoring
   - Set up health check endpoints

## Stripe Integration Setup

### Prerequisites
- Stripe account (create one at https://dashboard.stripe.com)
- Stripe CLI installed for webhook testing
- Access to Stripe Dashboard for API keys

### API Key Configuration

1. Obtain API Keys
   - Log in to Stripe Dashboard
   - Navigate to Developers > API keys
   - Copy the publishable key and secret key
   ```
   # Frontend (.env)
   REACT_APP_STRIPE_PUBLISHABLE_KEY=pk_test_...

   # Backend (.env)
   STRIPE_SECRET_KEY=sk_test_...
   ```

2. Testing vs Production Keys
   - Use `pk_test_` and `sk_test_` keys for development
   - Use `pk_live_` and `sk_live_` keys for production
   - Never commit real API keys to version control

### Webhook Setup

1. Install Stripe CLI
   ```bash
   # Download and install Stripe CLI
   brew install stripe/stripe-cli/stripe    # macOS
   # or download from https://stripe.com/docs/stripe-cli

   # Login to Stripe CLI
   stripe login
   ```

2. Configure Webhook Endpoints
   ```bash
   # Start webhook forwarding (development)
   stripe listen --forward-to localhost:3001/api/payments/webhook
   ```

3. Set Webhook Secret
   - Copy the webhook signing secret from CLI output
   - Add to backend environment variables:
   ```
   STRIPE_WEBHOOK_SECRET=whsec_...
   ```

4. Production Webhook Setup
   - Create webhook endpoint in Stripe Dashboard
   - Use your production API endpoint URL
   - Select relevant events to monitor
   - Copy signing secret to production environment

### Payment Integration Testing

1. Test Card Numbers
   ```
   Success: 4242 4242 4242 4242
   Decline: 4000 0000 0000 0002
   3D Secure: 4000 0000 0000 3220
   ```

2. Test Webhook Events
   ```bash
   # Trigger test events
   stripe trigger payment_intent.succeeded
   stripe trigger payment_intent.failed
   ```

3. Testing Process
   - Use test mode in Stripe Dashboard
   - Verify webhook delivery
   - Check payment processing
   - Validate email notifications
   - Test refund process

### Production Considerations

1. Security Measures
   - Use HTTPS for all endpoints
   - Validate webhook signatures
   - Implement idempotency keys
   - Monitor failed payments

2. Error Handling
   - Handle card declines gracefully
   - Implement retry logic
   - Log payment errors
   - Set up error notifications

3. Monitoring
   - Set up Stripe Dashboard alerts
   - Monitor webhook delivery
   - Track payment success rates
   - Monitor API response times

4. Compliance
   - Implement Strong Customer Authentication (SCA)
   - Follow PCI compliance guidelines
   - Store card data securely
   - Handle customer data properly

## Database Setup

### PostgreSQL Installation

1. Install PostgreSQL
   ```bash
   # Ubuntu/Debian
   sudo apt update
   sudo apt install postgresql postgresql-contrib

   # macOS with Homebrew
   brew install postgresql
   brew services start postgresql

   # Windows
   # Download installer from https://www.postgresql.org/download/windows/
   ```

2. Verify Installation
   ```bash
   # Check version
   psql --version

   # Connect to default database
   psql -U postgres
   ```

3. Create Database
   ```sql
   CREATE DATABASE project_ledger;
   CREATE USER project_user WITH ENCRYPTED PASSWORD 'your_password';
   GRANT ALL PRIVILEGES ON DATABASE project_ledger TO project_user;
   ```

### Schema Management

1. Review Schema
   - Examine `prisma/schema.prisma` for database structure
   - Check model relationships and field types
   - Verify indexes and constraints

2. Initialize Database
   ```bash
   # Generate Prisma client
   npm run prisma:generate

   # Run migrations
   npm run prisma:migrate

   # Seed database (if available)
   npm run prisma:seed
   ```

3. Migration Commands
   ```bash
   # Create a new migration
   npx prisma migrate dev --name describe_your_change

   # Apply migrations (development)
   npx prisma migrate dev

   # Apply migrations (production)
   npx prisma migrate deploy

   # Reset database (development only)
   npx prisma migrate reset
   ```

### Database Maintenance

1. Backup Procedures
   ```bash
   # Backup database
   pg_dump -U project_user project_ledger > backup.sql

   # Scheduled backups (cron job)
   0 0 * * * pg_dump -U project_user project_ledger > /path/to/backup/backup_$(date +\%Y\%m\%d).sql

   # Backup with compression
   pg_dump -U project_user project_ledger | gzip > backup.sql.gz
   ```

2. Restore Procedures
   ```bash
   # Restore from backup
   psql -U project_user project_ledger < backup.sql

   # Restore compressed backup
   gunzip -c backup.sql.gz | psql -U project_user project_ledger
   ```

3. Maintenance Tasks
   ```sql
   -- Analyze tables
   ANALYZE project_ledger;

   -- Vacuum database
   VACUUM FULL project_ledger;

   -- Update statistics
   ANALYZE VERBOSE;
   ```

### Monitoring and Optimization

1. Database Monitoring
   - Monitor connection pool usage
   - Track query performance
   - Monitor disk space usage
   - Check for long-running queries

2. Performance Optimization
   ```sql
   -- Add indexes for common queries
   CREATE INDEX idx_invoice_client ON invoices(client_id);
   CREATE INDEX idx_payment_invoice ON payments(invoice_id);

   -- Analyze query plans
   EXPLAIN ANALYZE SELECT * FROM invoices WHERE client_id = 1;
   ```

3. Connection Pool Settings
   ```typescript
   // prisma/schema.prisma
   datasource db {
     provider = "postgresql"
     url      = env("DATABASE_URL")
     // Adjust pool settings based on load
     poolTimeout = 20
     pool = { min = 2, max = 10 }
   }
   ```

4. Backup Verification
   ```bash
   # Test backup integrity
   pg_restore --list backup.sql

   # Restore to test database
   createdb test_restore
   pg_restore -d test_restore backup.sql

## Build and Deployment Procedures

### Build Process

1. Frontend Build
   ```bash
   # Install dependencies
   cd project-ledger
   npm install

   # Build for production
   npm run build

   # Output will be in the 'build' directory
   ```

2. Backend Build
   ```bash
   # Install dependencies
   cd server
   npm install

   # Build TypeScript
   npm run build

   # Output will be in the 'dist' directory
   ```

3. Environment Configuration
   ```bash
   # Create production environment files
   cp .env.example .env.production

   # Set production environment variables
   REACT_APP_API_URL=https://api.yoursite.com
   REACT_APP_STRIPE_PUBLIC_KEY=pk_live_...
   NODE_ENV=production
   ```

### Deployment Configuration

1. Web Server Setup (Nginx)
   ```nginx
   # /etc/nginx/sites-available/project-ledger
   server {
       listen 80;
       server_name yoursite.com;

       # Frontend
       location / {
           root /var/www/project-ledger;
           try_files $uri $uri/ /index.html;
           expires 1h;
           add_header Cache-Control "public, no-transform";
       }

       # Backend API
       location /api {
           proxy_pass http://localhost:3000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```

2. Process Management (PM2)
   ```bash
   # Install PM2
   npm install -g pm2

   # Start backend server
   pm2 start ecosystem.config.js

   # Monitor processes
   pm2 monit

   # View logs
   pm2 logs
   ```

3. SSL Configuration
   ```bash
   # Install Certbot
   sudo apt install certbot python3-certbot-nginx

   # Obtain SSL certificate
   sudo certbot --nginx -d yoursite.com

   # Auto-renewal
   sudo certbot renew --dry-run
   ```

### CI/CD Pipeline

1. GitHub Actions Workflow
   ```yaml
   # .github/workflows/deploy.yml
   name: Deploy Project Ledger

   on:
     push:
       branches: [ main ]

   jobs:
     build-and-deploy:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v2
         
         - name: Setup Node.js
           uses: actions/setup-node@v2
           with:
             node-version: '18'

         - name: Install Dependencies
           run: |
             npm ci
             cd server && npm ci

         - name: Build
           run: |
             npm run build
             cd server && npm run build

         - name: Deploy
           uses: appleboy/ssh-action@master
           with:
             host: ${{ secrets.HOST }}
             username: ${{ secrets.USERNAME }}
             key: ${{ secrets.SSH_KEY }}
             script: |
               cd /var/www/project-ledger
               git pull
               npm ci
               npm run build
               pm2 restart all
   ```

2. Deployment Scripts
   ```bash
   # deploy.sh
   #!/bin/bash
   
   echo "Deploying Project Ledger..."
   
   # Pull latest changes
   git pull origin main
   
   # Install dependencies
   npm ci
   cd server && npm ci
   
   # Build applications
   npm run build
   cd server && npm run build
   
   # Restart services
   pm2 restart all
   
   echo "Deployment complete!"
   ```

3. Rollback Procedures
   ```bash
   # rollback.sh
   #!/bin/bash
   
   # Usage: ./rollback.sh <commit-hash>
   
   if [ -z "$1" ]; then
     echo "Please provide a commit hash"
     exit 1
   fi
   
   echo "Rolling back to $1..."
   
   git checkout $1
   
   npm ci
   npm run build
   
   cd server
   npm ci
   npm run build
   
   pm2 restart all
   
   echo "Rollback complete!"
   ```

### Deployment Verification

1. Health Checks
   ```bash
   # Check backend health
   curl https://api.yoursite.com/health

   # Monitor error rates
   tail -f /var/log/nginx/error.log

   # Check database connectivity
   npx prisma db seed
   ```

2. Performance Monitoring
   ```bash
   # Install monitoring tools
   npm install -g clinic

   # Profile application
   clinic doctor -- node server/dist/index.js

   # Monitor memory usage
   clinic heap -- node server/dist/index.js
   ```

3. Load Testing
   ```bash
   # Install artillery
   npm install -g artillery

   # Run load tests
   artillery run load-tests/scenarios.yml

   # Generate report
   artillery report load-test-results.json
   ```

## Production Considerations

### Security Hardening

1. Network Security
  ```bash
  # Configure firewall rules
  sudo ufw allow 80,443,22
  sudo ufw enable

  # Set up rate limiting in Nginx
  limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
  ```

2. Application Security
  ```nginx
  # In Nginx configuration
  add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
  add_header X-Frame-Options "SAMEORIGIN" always;
  add_header X-XSS-Protection "1; mode=block" always;
  add_header X-Content-Type-Options "nosniff" always;
  add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline';" always;
  ```

3. Data Protection
  - Implement regular security audits
  - Use encryption at rest for sensitive data
  - Implement proper session management
  - Regular security patch updates

### Performance Optimization

1. Frontend Performance
  - Implement code splitting and lazy loading
  - Optimize bundle size
  - Use CDN for static assets
  - Enable browser caching
  ```nginx
  # Nginx caching configuration
  location /static/ {
      expires 1y;
      add_header Cache-Control "public, no-transform";
  }
  ```

2. Backend Performance
  - Implement response compression
  - Use connection pooling
  - Cache frequently accessed data
  - Optimize database queries
  ```typescript
  // Enable compression
  app.use(compression());

  // Configure caching
  const cache = new NodeCache({ stdTTL: 600 });
  ```

3. Database Optimization
  - Regular index maintenance
  - Query optimization
  - Connection pool tuning
  - Regular VACUUM and ANALYZE

### Infrastructure Management

1. High Availability
  ```nginx
  # Load balancer configuration
  upstream backend {
      server backend1.example.com:3000;
      server backend2.example.com:3000;
      keepalive 32;
  }
  ```

2. Backup Strategy
  - Daily automated backups
  - Cross-region backup storage
  - Regular backup testing
  - Point-in-time recovery setup

3. Monitoring Setup
  ```bash
  # Set up monitoring tools
  npm install -g pm2@latest
  pm2 install pm2-logrotate
  pm2 set pm2-logrotate:max_size 10M
  pm2 set pm2-logrotate:retain 7
  ```

### Disaster Recovery

1. Recovery Procedures
  - Document recovery steps
  - Regular recovery testing
  - Maintain backup restoration scripts
  - Define incident response plan

2. Failover Configuration
  ```nginx
  # Health checks for failover
  health_check interval=5s fails=3 passes=2;
  ```

3. Data Recovery
  - Regular backup verification
  - Documented restore procedures
  - Test recovery time objectives

### Scaling Strategy

1. Horizontal Scaling
  ```yaml
  # Docker Compose configuration
  version: '3'
  services:
    api:
      image: project-ledger-api
      deploy:
        replicas: 3
        update_config:
          parallelism: 1
          delay: 10s
  ```

2. Vertical Scaling
  - Monitor resource usage
  - Implement auto-scaling rules
  - Define scaling thresholds
  - Resource optimization

3. Database Scaling
  - Read replicas setup
  - Connection pool optimization
  - Sharding strategy
  - Caching implementation

### Compliance and Auditing

1. Access Control
  - Role-based access control
  - Regular access reviews
  - Audit logging
  - Session management

2. Data Compliance
  - Data retention policies
  - Privacy compliance
  - Regular compliance audits
  - Data encryption standards

3. Monitoring and Alerts
  ```javascript
  // Alert configuration
  const alerts = {
    highCPU: {
      threshold: 80,
      duration: '5m',
      action: 'notify-team'
    },
    errorRate: {
      threshold: 1,
      duration: '1m',
      action: 'page-oncall'
    }
  };
  ```

### Production Checklist

1. Pre-deployment Verification
  - Security scan complete
  - Performance testing done
  - Backup systems verified
  - Monitoring configured

2. Launch Procedures
  - Staged rollout plan
  - Rollback procedures ready
  - Communication plan prepared
  - Support team briefed

3. Post-launch Monitoring
  - Performance metrics tracking
  - Error rate monitoring
  - User experience metrics
  - Resource utilization checks
   ```