# ProjectLedger Reporting Functionality - Implementation Plan

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
- Apply filters: e.g. "Invoices where status = overdue"
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
- **"Create Custom Report" Wizard** → guides users through building a report

---

## Design System Integration

Following the existing ProjectLedger design system and patterns:

### Frontend Components
- **Use existing layout system**: `ResponsivePageLayout`, `PageHeader`, `PageGrid`, `PageGridItem`
- **Data tables**: Leverage the robust `EnhancedDataTable` component with mobile-responsive cards
- **Forms**: Use `BrandedFormComponents` (`BrandedTextField`, etc.) for filter inputs
- **Charts**: Integrate with existing charting patterns from dashboard 
- **Modals**: Use `EnhancedModal` and `ConfirmDialog` for report configuration
- **Empty states**: Utilize `BrandedEmptyState` for no-data scenarios
- **Loading states**: Apply consistent `LoadingStates`, `Skeleton` patterns
- **Design tokens**: Follow `designTokens` for spacing, colors, typography, shadows

### Navigation Integration
- Add "Reports" to main navigation menu (alongside Clients, Projects, Quotes, Invoices)
- Use existing breadcrumb system for navigation hierarchy
- Follow established routing patterns in `routes.tsx`

### Theme & Branding
- Apply existing Material-UI theme customizations
- Use `AuthComponents` styling patterns for consistent branded appearance
- Follow responsive breakpoints and mobile-first design principles
- Leverage gradient backgrounds and glass morphism effects where appropriate

---

## Technical Implementation Plan

### Backend Architecture

#### 1. Database Schema Extensions
**New Models** (add to `schema.prisma`):

```prisma
model Report {
  id          Int      @id @default(autoincrement())
  name        String
  description String?
  type        String   // 'standard' | 'custom'
  category    String   // 'financial' | 'operational' | 'compliance'
  config      Json     // Report configuration (filters, columns, etc.)
  accountId   Int
  createdById Int
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  isPublic    Boolean  @default(false)
  
  account     Account  @relation(fields: [accountId], references: [id])
  createdBy   User     @relation(fields: [createdById], references: [id])
  
  @@index([accountId])
  @@index([type, category])
}

model ReportSchedule {
  id         Int      @id @default(autoincrement())
  reportId   Int
  frequency  String   // 'daily' | 'weekly' | 'monthly'
  recipients Json     // Array of email addresses
  nextRun    DateTime
  active     Boolean  @default(true)
  createdAt  DateTime @default(now())
  
  report     Report   @relation(fields: [reportId], references: [id])
  
  @@index([nextRun, active])
}
```

#### 2. API Endpoints
**Following existing API patterns** (`/apps/backend/src/routes/reports.ts`):

```typescript
// Standard reports
GET    /api/reports                          // List all available reports
GET    /api/reports/standard                 // Get standard report templates
GET    /api/reports/custom                   // Get user's custom reports

// Report execution
GET    /api/reports/:id/data                 // Execute report and get data
POST   /api/reports/:id/export               // Export report (PDF/CSV/XLSX)

// Custom report management
POST   /api/reports                          // Create custom report
PUT    /api/reports/:id                      // Update custom report
DELETE /api/reports/:id                      // Delete custom report

// Report scheduling
GET    /api/reports/:id/schedules            // Get report schedules
POST   /api/reports/:id/schedules            // Create schedule
PUT    /api/reports/schedules/:scheduleId    // Update schedule
DELETE /api/reports/schedules/:scheduleId   // Delete schedule

// Analytics endpoints (similar to existing dashboard API)
GET    /api/reports/analytics/revenue        // Revenue analytics
GET    /api/reports/analytics/projects       // Project analytics
GET    /api/reports/analytics/invoices       // Invoice analytics
```

#### 3. Report Service Layer
**Following existing service patterns**:

```typescript
// /apps/backend/src/services/reportService.ts
export class ReportService {
  // Standard report generators
  generateInvoiceAgingReport(accountId: number, filters: ReportFilters)
  generateRevenueReport(accountId: number, dateRange: DateRange)
  generateProjectProfitabilityReport(accountId: number, filters: ReportFilters)
  
  // Custom report engine
  executeCustomReport(reportConfig: ReportConfig, accountId: number)
  
  // Export functionality
  exportToPDF(reportData: ReportData, template: string)
  exportToCSV(reportData: ReportData)
  exportToExcel(reportData: ReportData)
}
```

### Frontend Architecture

#### 1. Page Structure
**Following existing page patterns**:

```
/apps/frontend/src/pages/reports/
├── ReportsPage.tsx              // Main reports dashboard
├── ReportDetailPage.tsx         // View specific report
├── ReportCreatePage.tsx         // Create custom report wizard
├── ReportEditPage.tsx           // Edit custom report
└── StandardReportsPage.tsx      // Browse standard report templates
```

#### 2. Component Structure
**Leveraging existing component architecture**:

```
/apps/frontend/src/components/reports/
├── ReportsDashboard.tsx         // Dashboard with KPI cards
├── ReportLibrary.tsx            // List of available reports
├── ReportBuilder/               // Custom report builder components
│   ├── DataSourceSelector.tsx
│   ├── FilterBuilder.tsx
│   ├── ColumnSelector.tsx
│   └── ChartTypeSelector.tsx
├── ReportViewer.tsx             // Display report results
├── ReportExportButton.tsx       // Export functionality
└── ReportScheduleModal.tsx      // Schedule configuration
```

#### 3. API Integration
**Following existing API patterns** (`/apps/frontend/src/api/reports/`):

```typescript
// /apps/frontend/src/api/reports/index.ts
export const reportsApi = {
  getReports: () => axiosInstance.get('/api/reports'),
  getStandardReports: () => axiosInstance.get('/api/reports/standard'),
  getCustomReports: () => axiosInstance.get('/api/reports/custom'),
  executeReport: (id: number, filters?: ReportFilters) => 
    axiosInstance.get(`/api/reports/${id}/data`, { params: filters }),
  exportReport: (id: number, format: 'pdf' | 'csv' | 'xlsx') =>
    axiosInstance.post(`/api/reports/${id}/export`, { format }),
  createReport: (report: CreateReportRequest) =>
    axiosInstance.post('/api/reports', report),
  updateReport: (id: number, report: UpdateReportRequest) =>
    axiosInstance.put(`/api/reports/${id}`, report),
  deleteReport: (id: number) =>
    axiosInstance.delete(`/api/reports/${id}`)
};
```

#### 4. Route Integration
**Add to existing route structure** (`/apps/frontend/src/routes/routes.tsx`):

```typescript
{
  path: 'reports',
  children: [
    {
      index: true,
      element: <ReportsPage />
    },
    {
      path: 'new',
      element: <ReportCreatePage />
    },
    {
      path: ':id',
      element: <ReportDetailPage />
    },
    {
      path: ':id/edit',
      element: <ReportEditPage />
    },
    {
      path: 'standard',
      element: <StandardReportsPage />
    }
  ]
}
```

---

## Implementation Phases

### Phase 1: Foundation (2-3 weeks)
1. **Backend Setup**
   - Add Report and ReportSchedule models to Prisma schema
   - Create basic API endpoints structure
   - Implement authentication/authorization middleware (reusing existing patterns)

2. **Frontend Setup**
   - Add Reports section to main navigation
   - Create basic page structure using existing layout components
   - Set up routing and API integration layer

### Phase 2: Standard Reports (3-4 weeks)
1. **Financial Reports**
   - Invoice Aging Report using existing invoice data
   - Revenue by Period using dashboard analytics patterns
   - Quote vs Invoice Conversion Rate

2. **UI Implementation**
   - Reports dashboard using existing card components
   - Report viewer using `EnhancedDataTable`
   - Export functionality (PDF, CSV, Excel)

### Phase 3: Custom Report Builder (4-5 weeks)
1. **Report Builder Interface**
   - Data source selector using existing form components
   - Filter builder with dynamic form generation
   - Column selector with drag-and-drop (if needed)

2. **Chart Integration**
   - Integrate charting library (following dashboard patterns)
   - Chart type selector and configuration
   - Chart export functionality

### Phase 4: Advanced Features (2-3 weeks)
1. **Report Scheduling**
   - Schedule configuration modal
   - Email delivery system
   - Background job processing

2. **Performance & Polish**
   - Report caching and optimization
   - Mobile responsiveness refinement
   - Loading states and error handling

---

## Design Decisions Based on Current Architecture

### 1. Simple Filter-and-Export Tool
**Decision**: Start with a simple, guided approach rather than full drag-and-drop builder
**Rationale**: Aligns with the application's focus on simplicity for small contractors

### 2. Interactive Dashboards with Export Options
**Decision**: Provide both interactive viewing and static export capabilities
**Rationale**: Supports both real-time decision making and formal reporting needs

### 3. Component Reuse Strategy
**Leverage existing components**:
- `EnhancedDataTable` for report results display
- `BrandedFormComponents` for filters and configuration
- `ResponsivePageLayout` for consistent page structure
- Dashboard card patterns for KPI displays
- Existing modal and navigation patterns

### 4. Mobile-First Responsive Design
**Follow established patterns**:
- Mobile-responsive report cards (similar to data table mobile view)
- Touch-optimized interactions
- Progressive enhancement for desktop features

---

## Technical Considerations

### Performance
- **Caching**: Implement report result caching using existing patterns
- **Pagination**: Use established pagination patterns for large datasets
- **Query Optimization**: Leverage existing Prisma optimization techniques

### Security
- **Authorization**: Follow existing role-based access patterns
- **Data Isolation**: Ensure account-level data separation (existing pattern)
- **Input Validation**: Use existing Zod validation patterns

### Scalability
- **Background Processing**: Implement for export generation and scheduling
- **Database Indexing**: Add appropriate indexes for report queries
- **API Rate Limiting**: Follow existing API protection patterns

---

## Success Metrics

### User Adoption
- Monthly active users of reporting features
- Number of custom reports created
- Export frequency and formats used

### Performance
- Report generation time < 5 seconds for standard reports
- Export generation time < 30 seconds for large datasets
- Page load performance maintaining existing standards

### Business Value
- Reduction in manual reporting time for users
- Increased user engagement with financial data
- Feature adoption rate among existing customers

---

## Detailed Component Examples

### Example: Reports Dashboard Component

```typescript
// /apps/frontend/src/components/reports/ReportsDashboard.tsx
import React from 'react';
import { 
  PageGrid, 
  PageGridItem, 
  Card, 
  CardContent, 
  Typography, 
  Box 
} from '../common';
import { 
  TrendingUp, 
  AttachMoney, 
  Receipt, 
  Business 
} from '@mui/icons-material';
import { designTokens } from '../../theme/tokens';

interface KPICardProps {
  title: string;
  value: string | number;
  trend?: number;
  icon: React.ReactNode;
  onClick?: () => void;
}

const KPICard: React.FC<KPICardProps> = ({ title, value, trend, icon, onClick }) => (
  <Card 
    sx={{ 
      cursor: onClick ? 'pointer' : 'default',
      transition: designTokens.transitions.normal,
      '&:hover': onClick ? {
        boxShadow: designTokens.shadows.md,
        transform: 'translateY(-2px)'
      } : {}
    }}
    onClick={onClick}
  >
    <CardContent>
      <Box sx={{ display: 'flex', alignItems: 'center', gap: 2 }}>
        <Box sx={{ color: 'primary.main' }}>
          {icon}
        </Box>
        <Box sx={{ flex: 1 }}>
          <Typography variant="h4" fontWeight={600}>
            {value}
          </Typography>
          <Typography variant="body2" color="text.secondary">
            {title}
          </Typography>
          {trend && (
            <Box sx={{ display: 'flex', alignItems: 'center', gap: 0.5, mt: 0.5 }}>
              <TrendingUp 
                fontSize="small" 
                color={trend > 0 ? 'success' : 'error'} 
              />
              <Typography 
                variant="caption" 
                color={trend > 0 ? 'success.main' : 'error.main'}
              >
                {trend > 0 ? '+' : ''}{trend}%
              </Typography>
            </Box>
          )}
        </Box>
      </Box>
    </CardContent>
  </Card>
);

export const ReportsDashboard: React.FC = () => {
  const kpis = [
    {
      title: 'Monthly Revenue',
      value: '$12,450',
      trend: 8.2,
      icon: <AttachMoney />
    },
    {
      title: 'Outstanding Invoices',
      value: 7,
      icon: <Receipt />
    },
    {
      title: 'Active Projects',
      value: 12,
      trend: -2.1,
      icon: <Business />
    }
  ];

  return (
    <PageGrid spacing={3}>
      {kpis.map((kpi, index) => (
        <PageGridItem key={index} xs={12} sm={6} md={4}>
          <KPICard {...kpi} />
        </PageGridItem>
      ))}
    </PageGrid>
  );
};
```

### Example: Report Library Component

```typescript
// /apps/frontend/src/components/reports/ReportLibrary.tsx
import React from 'react';
import { EnhancedDataTable, EnhancedColumn } from '../common/EnhancedDataTable';
import { StatusBadge } from '../common/StatusBadge';
import { useNavigate } from 'react-router-dom';

interface Report {
  id: number;
  name: string;
  description: string;
  type: 'standard' | 'custom';
  category: string;
  lastRun?: string;
  createdAt: string;
}

export const ReportLibrary: React.FC = () => {
  const navigate = useNavigate();
  
  const columns: EnhancedColumn<Report>[] = [
    {
      id: 'name',
      label: 'Report Name',
      priority: 'high',
      mobileLabel: 'Report'
    },
    {
      id: 'type',
      label: 'Type',
      priority: 'medium',
      format: (value) => (
        <StatusBadge 
          status={value as string} 
          variant={value === 'standard' ? 'info' : 'primary'}
        />
      )
    },
    {
      id: 'category',
      label: 'Category',
      priority: 'medium'
    },
    {
      id: 'lastRun',
      label: 'Last Run',
      priority: 'low',
      format: (value) => value ? new Date(value as string).toLocaleDateString() : 'Never'
    }
  ];

  const mockReports: Report[] = [
    {
      id: 1,
      name: 'Invoice Aging Report',
      description: 'Shows outstanding invoices by age',
      type: 'standard',
      category: 'Financial',
      lastRun: '2024-03-15',
      createdAt: '2024-01-01'
    },
    // ... more reports
  ];

  return (
    <EnhancedDataTable
      columns={columns}
      rows={mockReports}
      title="Available Reports"
      getRowId={(row) => row.id.toString()}
      onRowClick={(report) => navigate(`/reports/${report.id}`)}
      searchable
      emptyMessage="No reports found"
      emptyActionLabel="Create Report"
      onEmptyAction={() => navigate('/reports/new')}
    />
  );
};
```

---

## Integration with Existing Codebase

### Navigation Menu Update
Add to `MainLayout.tsx`:

```typescript
const navigationItems = [
  { text: 'Dashboard', icon: <Dashboard />, path: '/' },
  { text: 'Clients', icon: <People />, path: '/clients' },
  { text: 'Projects', icon: <Business />, path: '/projects' },
  { text: 'Quotes', icon: <Assignment />, path: '/quotes' },
  { text: 'Invoices', icon: <Receipt />, path: '/invoices' },
  { text: 'Reports', icon: <Analytics />, path: '/reports' }, // NEW
  { text: 'Inventory', icon: <Inventory />, path: '/inventory' },
  // ... rest of items
];
```

### Type Definitions
Add to `packages/shared-types/`:

```typescript
// /packages/shared-types/reports.ts
export interface ReportConfig {
  dataSource: 'projects' | 'quotes' | 'invoices' | 'clients' | 'payments';
  filters: ReportFilter[];
  columns: string[];
  groupBy?: string;
  sortBy?: { field: string; direction: 'asc' | 'desc' };
  chartType?: 'table' | 'bar' | 'line' | 'pie';
}

export interface ReportFilter {
  field: string;
  operator: 'equals' | 'contains' | 'greater_than' | 'less_than' | 'between';
  value: any;
}

export interface ReportData {
  headers: string[];
  rows: any[][];
  metadata: {
    totalRows: number;
    generatedAt: string;
    filters: ReportFilter[];
  };
}
```

This implementation plan provides a comprehensive roadmap for adding reporting functionality to ProjectLedger while fully leveraging the existing design system, component architecture, and established patterns. The phased approach ensures manageable development cycles while maintaining consistency with the current codebase.