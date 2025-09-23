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
- Created comprehensive REST API in `src/routes/reports-simple.ts`:
  - `GET /api/reports` - List available standard reports
  - `GET /api/reports/standard` - Get standard report templates  
  - `GET /api/reports/:id/data` - Execute report and return data
- Applied JWT authentication and account-scoped authorization
- Comprehensive error handling and validation with fallback sample data

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

**Status**: Phase 1 foundation is complete and deployed to production.

### Phase 2: Standard Reports (✅ COMPLETED)

**Current Status**: All core financial and operational reports implemented and ready for production use

**Priority Reports (Implementation Order):**

1. **Invoice Aging Report** (✅ Implemented)
   - ✅ Display unpaid invoices categorized by age (0-30, 31-60, 61-90, 90+ days)
   - ✅ Show total amounts outstanding by aging category  
   - ✅ Include customer details and invoice numbers
   - ✅ Real-time data processing from existing invoice records
   - ✅ Comprehensive aging calculation logic with summary totals

2. **Revenue by Period Report** (✅ Implemented)
   - ✅ Monthly revenue trends with comprehensive growth analysis
   - ✅ Month-over-month and year-over-year growth calculations
   - ✅ Invoice count and summary statistics by period
   - ✅ 24-month data coverage for meaningful trend analysis
   - ✅ Integration with existing invoice data and client information

3. **Project Profitability Report** (✅ Implemented)
   - ✅ Compare estimated vs actual project revenue and costs
   - ✅ Comprehensive profit margin analysis by project
   - ✅ Project status tracking (billing status, payment status)
   - ✅ Portfolio-wide profitability metrics and summary statistics
   - ✅ Integration with quotes for estimated revenue and invoice data for actuals

**Implementation Results:**
- ✅ Successfully leveraged existing data models (Invoice, Project, Quote, Client)
- ✅ Used established API patterns from Phase 1 with proper authentication
- ✅ Integrated comprehensive error handling with fallback sample data
- ✅ Focused on actionable insights for small contractors with growth analysis and profitability metrics
- ✅ Ready for frontend integration with Material-UI reporting components

**Status**: Phase 2 implementation complete. All three priority reports are functional and ready for production use.

### Phase 3: Frontend Integration & UI/UX Enhancements (✅ COMPLETED)

**Current Status**: Reports page completely redesigned with modern table-based interface, mobile responsiveness, and enhanced user experience

**Major UI/UX Improvements Completed:**

1. **Reports Page Modernization** (✅ Implemented)
   - ✅ Converted from card-based layout to responsive BrandedDataTable interface
   - ✅ Implemented consistent design patterns matching ClientsPage and ProjectsPage
   - ✅ Added comprehensive search functionality for both standard and custom reports
   - ✅ Enhanced mobile responsiveness with automatic card view on smaller screens
   - ✅ Integrated glass morphism styling for professional appearance

2. **Standard Reports Section** (✅ Implemented)
   - ✅ Professional table layout with 7 standard reports including new entity-specific reports
   - ✅ Report icons and descriptions for better visual identification
   - ✅ Category-based organization with colored chips (Financial, Analysis, Clients, Projects, Sales, Accounting)
   - ✅ Fixed routing issues - all entity-specific reports now properly navigate
   - ✅ Action buttons with "View Report" functionality

3. **Custom Reports Management** (✅ Implemented)
   - ✅ Converted to consistent table format matching standard reports
   - ✅ Added comprehensive CRUD operations (Create, Read, Update, Delete)
   - ✅ Delete functionality with professional confirmation dialog
   - ✅ Report metadata display (creation date, category, description)
   - ✅ Action buttons for Run, Edit, and Delete operations
   - ✅ Empty state handling with call-to-action for report creation

4. **Enhanced Report Navigation** (✅ Implemented)
   - ✅ Fixed REPORT_CONFIG ID mismatches causing "Report not found" errors
   - ✅ Proper routing to entity-specific reports (Client Summary, Project Summary, Quote Summary, Invoice Summary)
   - ✅ Streamlined page header - removed unnecessary "Create Custom Report" button
   - ✅ Integrated navigation patterns consistent with application standards

5. **Data Table Enhancements** (✅ Implemented)
   - ✅ Search functionality across all report fields
   - ✅ Sortable columns with disabled sorting for action columns
   - ✅ Responsive design with mobile-optimized card views
   - ✅ Professional loading states and error handling
   - ✅ Pagination and performance optimization

**Technical Implementation:**
- ✅ Leveraged existing BrandedDataTable component for consistency
- ✅ Enhanced EnhancedColumn interface with proper TypeScript support
- ✅ Implemented ConfirmDialog for safe delete operations
- ✅ Used existing reportsApi with proper error handling and optimistic updates
- ✅ Maintained accessibility standards and keyboard navigation
- ✅ Glass morphism design integration following application design system

**User Experience Results:**
- ✅ **Mobile-First Design**: Seamless experience across all device sizes
- ✅ **Professional Interface**: Consistent with modern business application standards
- ✅ **Enhanced Searchability**: Users can quickly find specific reports
- ✅ **Intuitive Operations**: Clear visual feedback for all user actions
- ✅ **Safe Data Management**: Confirmation dialogs prevent accidental deletions
- ✅ **Performance Optimized**: Fast loading and responsive interactions

**Status**: Phase 3 UI/UX enhancements complete. Reports page now provides a professional, mobile-responsive interface that scales with user needs and maintains consistency with the application's design system.

### Phase 4: Custom Report Builder (🔄 Planned)
- Data source selection interface
- Filter configuration wizard
- Grouping and aggregation options
- Chart visualization integration
- Export functionality (CSV, PDF, XLSX)
- Report scheduling and sharing

---
