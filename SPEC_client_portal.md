# Client Portal Feature Specification

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