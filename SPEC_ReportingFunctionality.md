# ProjectLedger Reporting Specification

## Context
ProjectLedger is an application that allows small contractors to create and manage projects, quotes, and invoices.  
We want to add **reporting functionality** that covers both standard (pre-built) reports and custom user-defined reports.  
The reporting system should be simple enough for small contractors to use, but flexible enough to grow with their needs.

---

## Requirements

### 1. Standard Reporting
Provide out-of-the-box reports that cover the most common contractor needs:

**Financial Reports**
- Invoice Aging Report (unpaid, overdue, upcoming payments)
- Revenue by Period (monthly/quarterly revenue trends)
- Quote vs. Invoice Conversion Rate

**Operational Reports**
- Project Profitability (estimated vs. actual cost/revenue)
- Top Customers (by revenue, frequency, or margin)
- Workload Overview (active projects, pipeline, completion status)

**Compliance / Administrative**
- Tax Summary (totals by tax code: GST/HST, VAT, etc.)
- Expenses Report (if expense tracking is enabled)
- Cash Flow Forecast (based on invoice due dates)

---

### 2. Custom Reporting
Allow users to create their own reports by defining parameters.

**Capabilities**
- Select data sources: projects, quotes, invoices, customers, payments
- Apply filters: e.g. “Invoices where status = overdue”
- Group data: by customer, by project, by month, etc.
- Choose output format: table, bar/line chart, pie chart
- Export results: CSV, PDF, XLSX
- Save & reuse reports
- Schedule & share (email/PDF exports on weekly/monthly basis)

---

### 3. User Experience (UX)
- **Reports Dashboard** → shows key KPIs at a glance (e.g., revenue, outstanding invoices, jobs in progress)
- **Report Library** → library of standard + saved custom reports
- **Quick Filters** → date ranges, customers, project status
- **“Create Custom Report” Wizard** → guides users through building a report

---

### 4. Technical Considerations
- **Backend**:  
  - Reporting endpoints should query across entities efficiently  
  - Use pre-aggregated data or caching for performance  
- **Frontend**:  
  - Use a table/grid component with sorting/filtering (e.g., AG Grid, MUI DataGrid)  
  - Charts via Recharts or Chart.js  
- **Export**:  
  - PDF (reportlab), CSV (pandas), XLSX (openpyxl)  
- **Permissions**:  
  - Ensure users only access their own account data  

---

## Open Questions
- Should custom reporting be a **simple filter-and-export tool** or a **full drag-and-drop builder**?  
- Should reports be **interactive dashboards** (real-time) or **static exports** (for sharing)?  

---

## Implementation Status

### Phase 1: Foundation & Basic Infrastructure ✅ COMPLETED

**Database Schema (✅ Completed)**
- Added `Report` model with fields: type, name, description, config, accountId
- Added `ReportSchedule` model with fields: frequency, recipients, enabled, nextRun
- Implemented proper indexes and foreign key relationships
- Migration successfully applied to Docker database environment

**Backend API (✅ Completed)**
- Created comprehensive REST API in `src/routes/reports.ts`:
  - `GET /api/reports` - List account reports
  - `GET /api/reports/:id` - Get specific report
  - `POST /api/reports` - Create new report  
  - `PUT /api/reports/:id` - Update report
  - `DELETE /api/reports/:id` - Delete report
  - `POST /api/reports/:id/generate` - Generate report data
  - Report scheduling endpoints for automated reports
- Applied JWT authentication and account-scoped authorization
- Comprehensive error handling and validation

**Frontend Scaffolding (✅ Completed)**
- Created Reports page with Material-UI responsive design
- Added navigation menu integration
- Implemented report list view with edit/delete actions
- Created modal dialog for report creation
- Established API client integration following existing patterns

**Environment Compatibility (✅ Completed)**
- Resolved Prisma client generation differences between local and Docker environments
- Fixed relation naming issues across all services (Client, Project relations)
- Verified successful compilation and operation in Docker (production target)
- Report and ReportSchedule models confirmed working in deployment environment

**Status**: Phase 1 foundation is complete and ready for approval. Docker environment compiles successfully with all Report functionality operational.

### Phase 2: Standard Reports (🔄 Planned)
- Invoice Aging Report implementation
- Revenue by Period reports  
- Quote vs Invoice Conversion tracking
- Project Profitability analysis
- Top Customers reporting
- Tax Summary reports

### Phase 3: Custom Report Builder (🔄 Planned)
- Data source selection interface
- Filter configuration wizard
- Grouping and aggregation options
- Chart visualization integration
- Export functionality (CSV, PDF, XLSX)
- Report scheduling and sharing

---
