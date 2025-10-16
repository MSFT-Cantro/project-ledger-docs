# Client Portal Feature Specification

**Date:** October 16, 2025  
**Version:** 1.0  
**Status:** Not Started - Planning Phase  
**Priority:** High  

---

## 📊 Implementation Progress Summary

### Overall Completion: 0% (Not Started)

```
Phase 1: Foundation & Auth      ░░░░░░░░░░░░░░░░░░░░   0% ⏳
Phase 2: Data Viewing           ░░░░░░░░░░░░░░░░░░░░   0% ⏳
Phase 3: Quote Approval         ░░░░░░░░░░░░░░░░░░░░   0% ⏳
Phase 4: Invoice Enhancement    ░░░░░░░░░░░░░░░░░░░░   0% ⏳
Phase 5: Polish & Security      ░░░░░░░░░░░░░░░░░░░░   0% ⏳
```

### 🎯 Current Status
**Phase:** Pre-Development Planning  
**Next Milestone:** Phase 1 Kickoff  
**Estimated Start:** TBD  
**Estimated Completion:** 8-11 weeks from start

### Quick Reference
- **Total Duration**: 8-11 weeks
- **Total Effort**: 290-420 hours (780 hours with overhead)
- **Primary Risk**: Data security and authorization boundaries
- **Success Metric**: >80% quote approval rate via portal within 3 months
- **Launch Strategy**: Phased rollout starting with pilot group

---

### 🎯 Phase 1: Foundation & Authentication (2-3 weeks) ⚡ CRITICAL - ⏳ NOT STARTED
**Priority:** HIGHEST - Nothing else can start without this  
**Status:** Pending - Awaiting resource allocation and approval  
**Estimated Effort:** 80-120 hours

#### Pre-Development Checklist
- [ ] Review and approve specification
- [ ] Allocate backend developer (full-time, weeks 1-8)
- [ ] Allocate frontend developer (full-time, weeks 2-10)
- [ ] Allocate QA engineer (part-time, weeks 4-11)
- [ ] Request DNS access for portal.projectledger.ca subdomain
- [ ] Order SSL certificate for portal subdomain
- [ ] Set up staging environment with portal subdomain
- [ ] Create feature flag for portal access in admin panel

#### Backend Tasks
- [ ] Write database migration for Client portal fields
- [ ] Create QuoteApproval model and migration
- [ ] Create ClientPortalActivity model and migration
- [ ] Test migrations on staging database
- [ ] Implement client portal JWT authentication endpoints
  - [ ] POST /api/portal/auth/login - Client login endpoint
  - [ ] POST /api/portal/auth/logout - Client logout
  - [ ] GET /api/portal/auth/me - Get authenticated client info
  - [ ] POST /api/portal/auth/reset-password-request - Request password reset
  - [ ] POST /api/portal/auth/reset-password - Complete password reset
- [ ] Create password hashing and validation utilities
- [ ] Build authenticateClientPortal middleware
- [ ] Implement activity logging for all portal actions
- [ ] Build Client Management API (Admin)
  - [ ] PATCH /api/clients/:id/portal-access - Enable/disable portal
  - [ ] POST /api/clients/:id/set-portal-password - Set client password
  - [ ] GET /api/clients/:id/portal-activity - View client activity log

#### Frontend Tasks (Admin App)
- [ ] Add "Enable Portal Access" toggle to Client edit form
- [ ] Add "Set Portal Password" section to Client form
- [ ] Show portal activation status in Client details
- [ ] Display last login date in Client view
- [ ] Add button to view portal activity log
- [ ] Show portal activation history

#### Frontend Tasks (Client Portal)
- [ ] Configure DNS for portal.projectledger.ca
- [ ] Create portal subdomain setup (route-based or separate app)
- [ ] Design portal-specific branding/theme
- [ ] Create portal login page (email/password)
- [ ] Build password reset request page
- [ ] Build password reset confirmation page
- [ ] Implement logout functionality
- [ ] Create portal layout structure
- [ ] Build navigation for portal
- [ ] Add header with client name/logout
- [ ] Ensure responsive design for mobile

#### Testing & Validation
- [ ] Deploy Phase 1 to staging environment
- [ ] Security review of authentication system
- [ ] Test JWT token generation and validation
- [ ] Test password reset flow
- [ ] Test activity logging
- [ ] Enable portal for 2 internal test clients
- [ ] Verify data isolation between clients

#### Deliverables
- ✅ Clients can be enabled for portal access by admins
- ✅ Clients can log in with email and password
- ✅ JWT-based authentication for portal (separate from admin)
- ✅ Activity logging infrastructure in place
- ✅ Admin can manage client portal access and passwords
- ✅ Password reset flow functional

**🚧 GATE:** Security review of authentication before Phase 2

---

### 🎯 Phase 2: Project & Data Viewing (2-3 weeks) - ⏳ NOT STARTED
**Priority:** HIGH - Core value proposition  
**Status:** Pending - Dependent on Phase 1 completion  
**Estimated Effort:** 80-120 hours  
**Dependencies:** Phase 1 complete

#### Backend Tasks
- [ ] Build Client Portal Data API endpoints
  - [ ] GET /api/portal/dashboard - Dashboard summary data
  - [ ] GET /api/portal/projects - List client's projects
  - [ ] GET /api/portal/projects/:id - Get project details
  - [ ] GET /api/portal/projects/:id/quotes - Get project quotes
  - [ ] GET /api/portal/projects/:id/invoices - Get project invoices
  - [ ] GET /api/portal/projects/:id/tasks - Get project tasks
  - [ ] GET /api/portal/projects/:id/milestones - Get project milestones
- [ ] Implement client data scoping middleware (verify clientId match)
- [ ] Add authorization checks on every portal API request
- [ ] Implement activity logging for all view actions
- [ ] Add database indexes on clientId for performance
- [ ] Test data isolation (client A cannot see client B's data)

#### Frontend Tasks (Client Portal)
- [ ] Create dashboard/home page
  - [ ] Welcome message with client name
  - [ ] Project summary cards
  - [ ] Quick stats (active projects, pending quotes, outstanding invoices)
- [ ] Build projects list page
  - [ ] Grid/list view of all projects
  - [ ] Status badges (Active, Completed, On Hold)
  - [ ] Search functionality
  - [ ] Filter by status
- [ ] Create project detail page
  - [ ] Full project information display
  - [ ] Tabs for: Overview, Quotes, Invoices, Tasks, Milestones
  - [ ] Responsive design for mobile
- [ ] Build tasks & milestones view
  - [ ] List of tasks with status indicators
  - [ ] Milestone timeline visualization
  - [ ] Filter by status (Todo, In Progress, Done)
  - [ ] Due date highlighting
- [ ] Add loading states and error handling throughout
- [ ] Implement pagination for large datasets

#### Testing & Validation
- [ ] Deploy Phase 2 to staging
- [ ] Internal UAT with test clients
- [ ] Test data scoping thoroughly
- [ ] Performance testing (API response times < 500ms)
- [ ] Mobile responsiveness testing
- [ ] Cross-browser compatibility testing

#### Deliverables
- ✅ Clients can view all their projects
- ✅ Clients can view project details (overview, tasks, milestones)
- ✅ Data properly scoped to authenticated client only
- ✅ Activity logging for all view actions
- ✅ Dashboard provides useful project overview
- ✅ Responsive design works on mobile devices

**🚧 GATE:** Data authorization audit before Phase 3

---

### 🎯 Phase 3: Quote Approval System (2 weeks) ⭐ - ⏳ NOT STARTED
**Priority:** HIGH - Highest ROI feature  
**Status:** Pending - Dependent on Phase 2 completion  
**Estimated Effort:** 60-80 hours  
**Dependencies:** Phase 2 complete

#### Backend Tasks
- [ ] Build Quote Approval API
  - [ ] POST /api/portal/quotes/:id/approve - Approve quote
  - [ ] POST /api/portal/quotes/:id/reject - Reject quote
  - [ ] GET /api/portal/quotes/:id - Quote details
  - [ ] GET /api/portal/quotes/:id/history - Get approval history
- [ ] Implement quote status validation (only 'Sent' quotes can be approved)
- [ ] Auto-update quote status when client approves/rejects
- [ ] Create QuoteApproval records on client action
- [ ] Build email notification service for admins
  - [ ] Email on quote approval
  - [ ] Email on quote rejection (include client notes)
- [ ] Capture IP address and user agent for audit trail
- [ ] Validate quote belongs to authenticated client

#### Frontend Tasks (Client Portal)
- [ ] Create quote detail view
  - [ ] Display full quote information
  - [ ] Show line items with descriptions and amounts
  - [ ] Display tax and total calculations
  - [ ] Show status badge (Draft, Sent, Approved, Rejected)
- [ ] Build quote approval interface
  - [ ] "Approve" button with confirmation dialog
  - [ ] "Reject" button with confirmation dialog
  - [ ] Optional notes field for client feedback
  - [ ] Success/error feedback messages
- [ ] Create quote history view
  - [ ] Show all quotes for project
  - [ ] Display status timeline/history
  - [ ] Filter by status (Draft, Sent, Approved, Rejected)
- [ ] Add quote summary to dashboard
  - [ ] Pending quotes widget
  - [ ] Recent quote activity

#### Frontend Tasks (Admin App)
- [ ] Add quote approval notifications
  - [ ] In-app notification when client acts on quote
  - [ ] Real-time quote status updates
  - [ ] Display client notes/feedback
- [ ] Show approval history in quote details
- [ ] Add filter for client-approved quotes

#### Testing & Validation
- [ ] Deploy Phase 3 to staging
- [ ] Test complete approval workflow end-to-end
- [ ] Verify email notifications are sent
- [ ] Test with pilot group (3-5 clients)
- [ ] Collect pilot client feedback
- [ ] Verify audit trail captures all required data

#### Deliverables
- ✅ Clients can approve quotes with confirmation
- ✅ Clients can reject quotes with optional notes
- ✅ Quote status updates automatically on client action
- ✅ Admins receive email notifications immediately
- ✅ Approval history tracked and visible to admins
- ✅ Audit trail includes IP, timestamp, and user agent

**🚧 GATE:** Pilot group feedback review before Phase 4

---

### 🎯 Phase 4: Invoice Enhancement (1-2 weeks) - ⏳ NOT STARTED
**Priority:** MEDIUM - Adds transparency  
**Status:** Pending - Can run parallel to Phase 3  
**Estimated Effort:** 40-60 hours  
**Dependencies:** Phase 2 complete

#### Backend Tasks
- [ ] Build Invoice API enhancements
  - [ ] GET /api/portal/invoices - List all client invoices
  - [ ] GET /api/portal/invoices/:id - Get invoice details
  - [ ] GET /api/portal/invoices/:id/pdf - Download invoice PDF
- [ ] Implement invoice filtering
  - [ ] Filter by status (Paid, Pending, Overdue)
  - [ ] Filter by date range
  - [ ] Sort by due date, amount, status
- [ ] Add overdue calculation logic
- [ ] Optimize invoice queries for performance

#### Frontend Tasks (Client Portal)
- [ ] Create invoices list page
  - [ ] Table view of all invoices
  - [ ] Status badges (Paid, Pending, Overdue)
  - [ ] Highlight overdue invoices
  - [ ] Amount owed column
  - [ ] Filter and sort controls
- [ ] Build invoice detail view
  - [ ] Full invoice display
  - [ ] Line items breakdown
  - [ ] Payment history (if applicable)
  - [ ] Download PDF button
  - [ ] Payment status indicator
- [ ] Add dashboard invoice widget
  - [ ] Show outstanding balance
  - [ ] List recent/upcoming invoices
  - [ ] Quick link to invoices page
- [ ] Implement invoice PDF download

#### Testing & Validation
- [ ] Deploy Phase 4 to staging
- [ ] Test invoice filtering and sorting
- [ ] Verify PDF downloads work across browsers
- [ ] Test with various invoice statuses
- [ ] Performance testing with large invoice lists

#### Deliverables
- ✅ Clients can view all their invoices
- ✅ Clients can filter invoices by status and date
- ✅ Clients can view detailed invoice information
- ✅ Clients can download invoice PDFs
- ✅ Dashboard shows invoice summary and outstanding balance
- ✅ Overdue invoices clearly highlighted

**🚧 GATE:** Performance testing (API response times < 500ms)

---

### 🎯 Phase 5: Polish & Security Hardening (1 week) ⚡ CRITICAL - ⏳ NOT STARTED
**Priority:** CRITICAL - Must complete before full rollout  
**Status:** Pending - Dependent on all previous phases  
**Estimated Effort:** 30-40 hours  
**Dependencies:** All previous phases complete

#### Security Tasks
- [ ] Conduct comprehensive security audit
  - [ ] Review authentication implementation
  - [ ] Test authorization boundaries thoroughly
  - [ ] Verify data scoping is correct everywhere
  - [ ] Check for SQL injection vulnerabilities
  - [ ] Test for XSS vulnerabilities
- [ ] Implement rate limiting on all portal endpoints
- [ ] Add account lockout after 5 failed login attempts
- [ ] Add CAPTCHA to login form (Google reCAPTCHA)
- [ ] Enforce strong password requirements
  - [ ] Minimum 8 characters
  - [ ] Must include uppercase, lowercase, number
  - [ ] Password complexity validation
- [ ] Set up alerts for suspicious activity
  - [ ] Multiple failed login attempts
  - [ ] Unusual access patterns
  - [ ] Data access outside normal hours
- [ ] Penetration testing (external if budget allows)

#### UX/Polish Tasks
- [ ] Mobile optimization
  - [ ] Test all pages on iOS and Android
  - [ ] Ensure responsive design works perfectly
  - [ ] Optimize touch interactions
  - [ ] Test landscape and portrait modes
- [ ] Add loading states
  - [ ] Skeleton loaders for content
  - [ ] Proper loading spinners
  - [ ] Implement progress indicators
- [ ] Implement error states
  - [ ] User-friendly error messages
  - [ ] Retry mechanisms for failed requests
  - [ ] Offline state handling
- [ ] Add help & documentation
  - [ ] Help tooltips throughout portal
  - [ ] Create PDF user guide for clients
  - [ ] Build FAQ section
  - [ ] Add contextual help buttons

#### Testing Tasks
- [ ] Write E2E test suite
  - [ ] Login/logout flow
  - [ ] Project viewing workflow
  - [ ] Quote approval workflow
  - [ ] Invoice viewing workflow
  - [ ] Password reset flow
- [ ] Cross-browser testing
  - [ ] Chrome (latest)
  - [ ] Firefox (latest)
  - [ ] Safari (latest)
  - [ ] Edge (latest)
- [ ] Performance optimization
  - [ ] Database query optimization
  - [ ] Implement caching where appropriate
  - [ ] API response time tuning
- [ ] Load testing
  - [ ] Test with 50+ concurrent clients
  - [ ] Verify performance under load
  - [ ] Identify and fix bottlenecks

#### Deliverables
- ✅ Security audit complete with all issues resolved
- ✅ Mobile experience optimized and tested
- ✅ Help documentation created and accessible
- ✅ E2E tests passing for all critical workflows
- ✅ Performance benchmarks met (< 500ms API responses)
- ✅ Rate limiting and security hardening in place
- ✅ Production-ready for full client rollout

**🚧 GATE:** Final security + performance review before full rollout

---

### Post-Deployment Plan
- [ ] Deploy to production with feature flag OFF
- [ ] Gradual client enablement (10% per week)
- [ ] Monitor error rates and performance metrics
- [ ] Collect client feedback via in-portal survey
- [ ] Track success metrics
  - [ ] Login frequency
  - [ ] Quote approval rate
  - [ ] Time to quote approval
  - [ ] Client satisfaction scores
  - [ ] Support ticket volume
- [ ] Iteration based on feedback (2-week sprints)
- [ ] Plan Phase 6 (payment integration) based on adoption

---

### ✅ Completed Tasks
*No tasks completed yet - project not started*

---

---

### Key Decision Points

**Decision 1: Portal Architecture (Week 1)**
- ❓ Separate React app vs. route-based in existing app?
- **Recommendation**: Route-based initially, migrate to separate app if needed
- **Rationale**: Faster development, code reuse, single deployment

**Decision 2: Pilot Group Selection (End of Phase 1)**
- ❓ Which clients to enable first for UAT?
- **Recommendation**: 3-5 engaged clients with active projects and pending quotes
- **Rationale**: Get real feedback on quote approval flow early

**Decision 3: Quote Auto-Status Updates (Phase 3)**
- ❓ Should client approval automatically change quote status to "Accepted"?
- **Recommendation**: Yes, with admin override capability
- **Rationale**: Reduces manual work, but admins retain control

---

## Overview
Enable existing clients to access a dedicated client portal where they can view their projects, quotes, invoices, tasks, and milestones. This provides transparency and allows clients to take actions like approving/rejecting quotes without requiring direct communication.

---

## Feature Requirements

### User Story
> *As an existing client in the system, I want to log into a dedicated portal where I can view my projects, approve quotes, track invoices, and monitor project progress in real-time.*

### Core Capabilities

1. **Client Portal Authentication**
   - Clients can be enabled for portal access by account administrators
   - Each enabled client will have a secure password set
   - Portal accessible at `portal.projectledger.ca`
   - Clients log in using their email and password

2. **Project Viewing**
   - Clients can view all their associated projects
   - Full project details visible (name, description, status, dates, budget info)
   - Project list shows current status and progress

3. **Quote Management** ⭐
   - View all quotes attached to their projects
   - See quote details (line items, totals, dates, status)
   - **Approve or Reject quotes** (client action)
   - View quote history and status changes

4. **Invoice Tracking**
   - View all invoices (paid and unpaid)
   - See invoice details (items, amounts, due dates)
   - Filter by status (Paid, Pending, Overdue)
   - *Future enhancement: Pay invoices directly in portal*

5. **Task & Milestone Visibility**
   - View tasks associated with their projects
   - See milestone progress and completion status
   - Filter by status (Todo, In Progress, Done)
   - View due dates and descriptions

---

## System Architecture Proposal

### Database Schema Changes

#### 1. Extend Client Model
Add portal access fields to existing `Client` table:

```prisma
model Client {
  // ... existing fields ...
  
  // New fields for portal access
  portalEnabled       Boolean   @default(false)
  portalPassword      String?   // Hashed password
  portalLastLogin     DateTime?
  portalActivatedAt   DateTime?
  portalActivatedBy   Int?      // User who enabled portal
  
  // Relations
  portalActivator     User?     @relation("ClientPortalActivator", fields: [portalActivatedBy], references: [id])
  quoteApprovals      QuoteApproval[]
  
  @@index([portalEnabled])
}
```

#### 2. Quote Approval Tracking
New table to track client actions on quotes:

```prisma
model QuoteApproval {
  id          Int       @id @default(autoincrement())
  quoteId     Int
  clientId    Int
  action      String    // 'APPROVED' | 'REJECTED'
  notes       String?   // Optional client notes
  ipAddress   String?   // Audit trail
  userAgent   String?   // Audit trail
  createdAt   DateTime  @default(now())
  
  quote       Quote     @relation(fields: [quoteId], references: [id])
  client      Client    @relation(fields: [clientId], references: [id])
  
  @@unique([quoteId, clientId])
  @@index([quoteId])
  @@index([clientId])
}
```

#### 3. Portal Activity Log
Track client portal activity for security and analytics:

```prisma
model ClientPortalActivity {
  id          Int       @id @default(autoincrement())
  clientId    Int
  action      String    // 'LOGIN', 'VIEW_PROJECT', 'APPROVE_QUOTE', etc.
  resourceId  Int?      // ID of project/quote/invoice viewed
  resourceType String?  // 'PROJECT', 'QUOTE', 'INVOICE'
  ipAddress   String?
  userAgent   String?
  createdAt   DateTime  @default(now())
  
  client      Client    @relation(fields: [clientId], references: [id])
  
  @@index([clientId, createdAt])
  @@index([action])
}
```

---

## Implementation Plan - Phased Approach

### **Phase 1: Foundation & Authentication** (2-3 weeks)

#### Backend Tasks
1. **Database Migration**
   - Add portal fields to `Client` table
   - Create `QuoteApproval` table
   - Create `ClientPortalActivity` table
   - Run migration and test

2. **Client Portal Authentication API**
   - `POST /api/portal/auth/login` - Client login endpoint
   - `POST /api/portal/auth/logout` - Client logout
   - `GET /api/portal/auth/me` - Get authenticated client info
   - JWT token generation for clients (separate from user tokens)
   - Password reset flow for clients

3. **Client Management API (Admin)**
   - `PATCH /api/clients/:id/portal-access` - Enable/disable portal
   - `POST /api/clients/:id/set-portal-password` - Set client password
   - `GET /api/clients/:id/portal-activity` - View client activity log

#### Frontend Tasks (Admin App)
1. **Client Edit Form Enhancement**
   - Add "Enable Portal Access" toggle
   - Add "Set Portal Password" section
   - Show portal activation status
   - Display last login date

2. **Client Details View**
   - Show portal access status
   - Add button to view portal activity log
   - Show portal activation history

#### Frontend Tasks (Client Portal)
1. **Portal Subdomain Setup**
   - Configure DNS for `portal.projectledger.ca`
   - Create separate React app for portal (or route-based separation)
   - Design portal-specific branding/theme

2. **Authentication Pages**
   - Login page (email/password)
   - Password reset request page
   - Password reset confirmation page
   - Logout functionality

3. **Portal Layout**
   - Navigation structure for portal
   - Header with client name/logout
   - Responsive design for mobile

#### Deliverables
- ✅ Clients can be enabled for portal access
- ✅ Clients can log in with password
- ✅ JWT-based authentication for portal
- ✅ Activity logging infrastructure
- ✅ Admin can manage client portal access

---

### **Phase 2: Project & Data Viewing** (2-3 weeks)

#### Backend Tasks
1. **Client Portal Data API**
   - `GET /api/portal/projects` - List client's projects
   - `GET /api/portal/projects/:id` - Get project details
   - `GET /api/portal/projects/:id/quotes` - Get project quotes
   - `GET /api/portal/projects/:id/invoices` - Get project invoices
   - `GET /api/portal/projects/:id/tasks` - Get project tasks
   - `GET /api/portal/projects/:id/milestones` - Get project milestones

2. **Authorization Middleware**
   - `authenticateClientPortal` middleware
   - Verify client only accesses their own data
   - Check portal is enabled for client

3. **Activity Logging**
   - Log all view actions (projects, quotes, invoices)
   - Track timestamps and resources accessed

#### Frontend Tasks (Client Portal)
1. **Dashboard/Home Page**
   - Welcome message with client name
   - Project summary cards
   - Quick stats (active projects, pending quotes, outstanding invoices)

2. **Projects List Page**
   - Grid/list view of all projects
   - Status badges
   - Search and filter functionality

3. **Project Detail Page**
   - Full project information display
   - Tabs for: Overview, Quotes, Invoices, Tasks, Milestones
   - Responsive design

4. **Tasks & Milestones View**
   - List of tasks with status indicators
   - Milestone timeline visualization
   - Filter by status (Todo, In Progress, Done)
   - Due date highlighting

#### Deliverables
- ✅ Clients can view their projects
- ✅ Clients can view project details
- ✅ Clients can view tasks and milestones
- ✅ Data properly scoped to authenticated client
- ✅ Activity logging for all views

---

### **Phase 3: Quote Approval System** (2 weeks)

#### Backend Tasks
1. **Quote Approval API**
   - `POST /api/portal/quotes/:id/approve` - Approve quote
   - `POST /api/portal/quotes/:id/reject` - Reject quote
   - `GET /api/portal/quotes/:id/history` - Get approval history

2. **Quote Status Management**
   - Auto-update quote status when client approves/rejects
   - Send notifications to account admins on approval/rejection
   - Validate quote is in correct status for approval

3. **Email Notifications**
   - Send email to admin when quote is approved
   - Send email to admin when quote is rejected
   - Include client notes in notification

#### Frontend Tasks (Client Portal)
1. **Quote Detail View**
   - Display full quote information
   - Line items with descriptions and amounts
   - Tax and total calculations
   - Status badge

2. **Quote Approval Interface**
   - "Approve" and "Reject" action buttons
   - Optional notes field for client feedback
   - Confirmation dialog before action
   - Success/error feedback

3. **Quote History View**
   - Show all quotes for project
   - Status timeline/history
   - Filter by status (Draft, Sent, Approved, Rejected)

#### Frontend Tasks (Admin App)
1. **Quote Approval Notifications**
   - In-app notification when client acts on quote
   - Update quote status in real-time
   - Show client notes/feedback

#### Deliverables
- ✅ Clients can approve quotes
- ✅ Clients can reject quotes with notes
- ✅ Quote status updates automatically
- ✅ Admins receive notifications
- ✅ Approval history tracked and visible

---

### **Phase 4: Invoice Viewing & Enhancement** (1-2 weeks)

#### Backend Tasks
1. **Invoice API Enhancements**
   - `GET /api/portal/invoices` - List all client invoices
   - `GET /api/portal/invoices/:id` - Get invoice details
   - `GET /api/portal/invoices/:id/pdf` - Download invoice PDF

2. **Invoice Filtering**
   - Filter by status (Paid, Pending, Overdue)
   - Filter by date range
   - Sort by due date, amount, etc.

#### Frontend Tasks (Client Portal)
1. **Invoices List Page**
   - Table view of all invoices
   - Status badges (Paid, Pending, Overdue)
   - Amount owed highlighted
   - Filter and sort controls

2. **Invoice Detail View**
   - Full invoice display
   - Line items breakdown
   - Payment history (if applicable)
   - Download PDF button
   - Payment status indicator

3. **Dashboard Invoice Widget**
   - Show outstanding balance
   - List recent/upcoming invoices
   - Quick link to invoices page

#### Deliverables
- ✅ Clients can view all their invoices
- ✅ Clients can filter invoices by status
- ✅ Clients can view invoice details
- ✅ Clients can download invoice PDFs
- ✅ Dashboard shows invoice summary

---

### **Phase 5: Polish & Security Hardening** (1 week)

#### Security Tasks
1. **Security Audit**
   - Review authentication implementation
   - Test authorization boundaries
   - Ensure data scoping is correct
   - Rate limiting on portal endpoints

2. **Password Security**
   - Enforce strong password requirements
   - Add password complexity validation
   - Implement account lockout after failed attempts
   - Add CAPTCHA to login form

3. **Activity Monitoring**
   - Set up alerts for suspicious activity
   - Monitor failed login attempts
   - Track unusual access patterns

#### UX/Polish Tasks
1. **Mobile Optimization**
   - Test all pages on mobile devices
   - Ensure responsive design works
   - Optimize touch interactions

2. **Loading States**
   - Add skeleton loaders
   - Implement proper error states
   - Add retry mechanisms

3. **Help & Documentation**
   - Add help tooltips
   - Create user guide for clients
   - Add FAQ section

#### Testing Tasks
1. **End-to-End Testing**
   - Test complete user journeys
   - Test quote approval flow
   - Test data access restrictions
   - Cross-browser testing

2. **Performance Testing**
   - Load testing with multiple clients
   - Optimize API response times
   - Check database query performance

#### Deliverables
- ✅ Security audit complete
- ✅ Mobile experience optimized
- ✅ Help documentation created
- ✅ E2E tests passing
- ✅ Performance benchmarks met

---

## Technical Considerations

### Authentication Strategy
- **Separate JWT tokens** for client portal (different secret/expiry)
- Client tokens include: `clientId`, `accountId`, `portalSession`
- Token expiry: 24 hours (configurable)
- Refresh token support for better UX

### Authorization Rules
- Clients can ONLY access data where `clientId` matches their ID
- Middleware checks on every portal API request
- Admin API endpoints separate from portal endpoints
- No cross-client data leakage

### Subdomain Setup
**Option A: Separate React App**
- Pros: Complete isolation, easier security boundary
- Cons: More deployment complexity, code duplication

**Option B: Route-Based**
- Pros: Shared components, single deployment
- Cons: Requires careful route/auth separation

**Recommendation**: Start with route-based (Option B), migrate to separate app later if needed.

### Database Performance
- Add indexes on `clientId` for all client-scoped queries
- Consider caching frequently accessed client data
- Monitor query performance as data grows

### Email Notifications
- Use existing email service (if in place)
- Template system for quote approval notifications
- Configurable notification preferences for admins

---

## API Endpoints Summary

### Client Portal Authentication
- `POST /api/portal/auth/login` - Client login
- `POST /api/portal/auth/logout` - Client logout
- `GET /api/portal/auth/me` - Get current client info
- `POST /api/portal/auth/reset-password-request` - Request password reset
- `POST /api/portal/auth/reset-password` - Complete password reset

### Client Portal Data
- `GET /api/portal/dashboard` - Dashboard summary data
- `GET /api/portal/projects` - List projects
- `GET /api/portal/projects/:id` - Project details
- `GET /api/portal/projects/:id/quotes` - Project quotes
- `GET /api/portal/projects/:id/invoices` - Project invoices
- `GET /api/portal/projects/:id/tasks` - Project tasks
- `GET /api/portal/projects/:id/milestones` - Project milestones

### Quote Actions
- `GET /api/portal/quotes/:id` - Quote details
- `POST /api/portal/quotes/:id/approve` - Approve quote
- `POST /api/portal/quotes/:id/reject` - Reject quote
- `GET /api/portal/quotes/:id/history` - Approval history

### Invoice Access
- `GET /api/portal/invoices` - List invoices
- `GET /api/portal/invoices/:id` - Invoice details
- `GET /api/portal/invoices/:id/pdf` - Download PDF

### Admin Portal Management
- `PATCH /api/clients/:id/portal-access` - Toggle portal access
- `POST /api/clients/:id/set-portal-password` - Set password
- `GET /api/clients/:id/portal-activity` - View activity log

---

## Future Enhancements (Post-MVP)

### Phase 6+: Payment Integration
- Direct invoice payment through portal
- Credit card processing integration
- Payment history and receipts
- Auto-payment setup

### Additional Features
- **File/Document Sharing**: Upload/download project documents
- **Messaging System**: Direct messaging between client and team
- **Notifications**: Email/SMS alerts for new quotes, invoices
- **Mobile App**: Native iOS/Android apps
- **Custom Branding**: White-label portal for agencies
- **Multi-Language Support**: Internationalization
- **Reporting**: Client-facing project reports and analytics

---

## Success Metrics

### Engagement Metrics
- % of clients enabled for portal access
- Portal login frequency
- Time spent in portal
- Most viewed sections

### Business Metrics
- Quote approval rate (portal vs. email)
- Time to quote approval (reduction)
- Invoice viewing rate
- Client satisfaction scores

### Technical Metrics
- API response times (<500ms)
- Portal uptime (>99.5%)
- Failed login rate
- Security incidents (target: 0)

---

## Risk Mitigation

### Security Risks
- **Risk**: Client data exposure
- **Mitigation**: Strict authorization checks, security audit, penetration testing

### Performance Risks
- **Risk**: Slow portal performance
- **Mitigation**: Database optimization, caching, performance monitoring

### Adoption Risks
- **Risk**: Clients don't use portal
- **Mitigation**: User training, clear value proposition, email reminders

### Technical Risks
- **Risk**: Integration issues with existing system
- **Mitigation**: Phased rollout, thorough testing, rollback plan

---

## Estimated Timeline

| Phase | Duration | Effort |
|-------|----------|--------|
| Phase 1: Foundation & Auth | 2-3 weeks | 80-120 hours |
| Phase 2: Data Viewing | 2-3 weeks | 80-120 hours |
| Phase 3: Quote Approval | 2 weeks | 60-80 hours |
| Phase 4: Invoice Enhancement | 1-2 weeks | 40-60 hours |
| Phase 5: Polish & Security | 1 week | 30-40 hours |
| **Total** | **8-11 weeks** | **290-420 hours** |

*Note: Timeline assumes full-time development effort. Adjust based on team availability.*

---

## Dependencies

### Technical Dependencies
- ✅ Existing Client model in database
- ✅ Existing Project, Quote, Invoice, Task, Milestone models
- ✅ JWT authentication infrastructure
- ✅ Email service for notifications
- ⚠️ DNS access for subdomain setup
- ⚠️ SSL certificate for portal subdomain

### Team Dependencies
- Backend developer (API development)
- Frontend developer (Portal UI)
- Designer (Portal UX/branding)
- QA engineer (Testing)
- DevOps (Deployment, subdomain setup)

---

## Deployment Strategy

### Staging Rollout
1. Deploy to staging environment
2. Internal testing with test clients
3. UAT with select real clients (pilot group)
4. Gather feedback and iterate

### Production Rollout
1. Deploy portal authentication (Phase 1)
2. Gradually enable portal for willing clients
3. Deploy viewing features (Phase 2)
4. Deploy quote approval (Phase 3)
5. Monitor and optimize
6. Full client enablement

### Rollback Plan
- Feature flags for portal access
- Database migrations reversible
- Ability to disable portal globally
- Backup authentication for critical clients

---

## Conclusion

This client portal feature will significantly enhance client experience and reduce administrative overhead. The phased approach ensures we deliver value incrementally while managing risk. Starting with authentication and viewing, then adding interactive features like quote approval, provides a solid foundation for future enhancements like payment processing.

**Next Steps:**
1. Review and approve this specification
2. Allocate development resources
3. Set up staging environment for portal
4. Begin Phase 1 implementation
5. Identify pilot clients for UAT

---

*Document Version: 1.0*  
*Last Updated: October 7, 2025*  
*Status: Proposed - Awaiting Approval*