# Design System Migration Specification

**Date:** November 10, 2025  
**Type:** Technical Specification  
**Status:** Ready for Implementation  
**Priority:** Critical

## Overview

This specification outlines the complete migration of all frontend components to use the established design system. The audit revealed critical gaps where 100% of admin pages and significant portions of portal/main app pages bypass the design system.

## Current State Analysis

### Design System Adoption Rates
- **Admin Pages**: 0 of 9 pages (0% adoption)
- **Portal Pages**: ~60% adoption (mixed usage)
- **Main App Pages**: ~70% adoption (mixed usage)
- **Critical Impact**: ~2000+ lines of direct MUI usage bypassing design system

## Implementation Strategy

### Phase 1: Design System Component Development (Week 1-2)
Build all missing components in the design system before any page migrations.

### Phase 2: Critical Page Migration (Week 3-4)  
Migrate admin pages and high-priority portal pages.

### Phase 3: Complete Migration (Week 5-6)
Finish remaining pages and components.

---

## Phase 1: Missing Design System Components

### 1.1 Core Layout Components

#### Grid Component
**File**: `src/components/common/Grid.tsx`
**Priority**: Critical
**Used by**: All pages with layout issues

```typescript
// Specification
interface GridProps extends MuiGridProps {
  variant?: 'standard' | 'portal' | 'admin';
  spacing?: 'xs' | 'sm' | 'md' | 'lg' | 'xl';
}

// Features Required:
- Responsive behavior with design system breakpoints
- Consistent spacing using design tokens
- Portal/admin specific variants
- TypeScript support with proper prop forwarding
```

#### Container Component
**File**: `src/components/common/Container.tsx`
**Priority**: High
**Used by**: All page layouts

```typescript
// Specification
interface ContainerProps extends MuiContainerProps {
  variant?: 'standard' | 'portal' | 'admin';
  padding?: 'none' | 'sm' | 'md' | 'lg';
}
```

### 1.2 Data Display Components

#### EnhancedTable Component  
**File**: `src/components/common/EnhancedTable.tsx`
**Priority**: Critical
**Used by**: AdminAccountsPage, AdminUsersPage, AdminUserDetailPage

```typescript
// Specification
interface EnhancedTableProps<T> {
  data: T[];
  columns: TableColumn<T>[];
  loading?: boolean;
  onRowClick?: (row: T) => void;
  pagination?: {
    page: number;
    rowsPerPage: number;
    totalRows: number;
    onPageChange: (page: number) => void;
    onRowsPerPageChange: (rows: number) => void;
  };
  filtering?: {
    searchValue: string;
    onSearchChange: (value: string) => void;
    filters?: FilterConfig[];
  };
  actions?: TableAction<T>[];
}

interface TableColumn<T> {
  key: keyof T | string;
  label: string;
  sortable?: boolean;
  render?: (value: any, row: T) => React.ReactNode;
  align?: 'left' | 'center' | 'right';
  width?: string | number;
}

interface TableAction<T> {
  label: string;
  icon?: React.ReactNode;
  onClick: (row: T) => void;
  disabled?: (row: T) => boolean;
  color?: 'primary' | 'secondary' | 'error' | 'warning';
}

// Features Required:
- Built-in pagination with design system styling
- Search/filtering integration
- Row actions menu
- Sorting capabilities
- Loading states with skeleton
- Empty states integration
- Responsive behavior
- Accessibility compliance
```

#### Pagination Component
**File**: `src/components/common/Pagination.tsx`
**Priority**: High
**Used by**: AdminAccountsPage, AdminUsersPage, PortalProjectsPage

```typescript
// Specification
interface PaginationProps {
  currentPage: number;
  totalPages: number;
  onPageChange: (page: number) => void;
  showFirstLast?: boolean;
  showRowsPerPage?: boolean;
  rowsPerPageOptions?: number[];
  currentRowsPerPage?: number;
  onRowsPerPageChange?: (rows: number) => void;
  variant?: 'standard' | 'compact';
}

// Features Required:
- Design system styling
- Responsive behavior
- Accessibility compliance
- Integration with EnhancedTable
```

### 1.3 Form Components

#### FormSection Component
**File**: `src/components/common/FormSection.tsx`
**Priority**: High
**Used by**: AdminAccountEditPage, AdminSettingsPage

```typescript
// Specification
interface FormSectionProps {
  title: string;
  description?: string;
  children: React.ReactNode;
  collapsible?: boolean;
  defaultExpanded?: boolean;
  actions?: React.ReactNode;
}

// Features Required:
- Consistent section styling
- Optional collapsible behavior
- Action area for buttons
- Proper spacing and typography
```

#### SelectField Component
**File**: `src/components/common/SelectField.tsx`
**Priority**: High
**Used by**: AdminAccountsPage, AdminUsersPage, AdminAccountEditPage

```typescript
// Specification
interface SelectFieldProps<T = any> extends Omit<MuiSelectProps, 'onChange'> {
  label: string;
  options: SelectOption<T>[];
  value: T;
  onChange: (value: T) => void;
  placeholder?: string;
  loading?: boolean;
  error?: string;
  helperText?: string;
  searchable?: boolean;
}

interface SelectOption<T> {
  value: T;
  label: string;
  disabled?: boolean;
  group?: string;
}

// Features Required:
- Consistent with TextField styling
- Loading states
- Error handling
- Optional search functionality
- Grouped options support
```

#### InputField Component Enhancement
**File**: `src/components/common/InputField.tsx` (enhance existing TextField)
**Priority**: Medium
**Used by**: All form pages

```typescript
// Add InputAdornment support to existing TextField
interface InputFieldProps extends MuiTextFieldProps {
  startAdornment?: React.ReactNode;
  endAdornment?: React.ReactNode;
  loading?: boolean;
  onClear?: () => void;
  showClearButton?: boolean;
}

// Features Required:
- InputAdornment wrapper integration
- Clear button functionality
- Loading state within input
- Search icon integration
```

### 1.4 Navigation Components

#### TabPanel Component
**File**: `src/components/common/TabPanel.tsx`
**Priority**: Medium
**Used by**: AdminAccountDetailPage, AdminUserDetailPage

```typescript
// Specification
interface TabPanelProps {
  children: React.ReactNode;
  index: number;
  value: number;
  keepMounted?: boolean;
}

interface TabsContainerProps {
  value: number;
  onChange: (value: number) => void;
  tabs: TabConfig[];
  variant?: 'standard' | 'scrollable' | 'fullWidth';
}

interface TabConfig {
  label: string;
  icon?: React.ReactNode;
  disabled?: boolean;
  content: React.ReactNode;
}

// Features Required:
- Consistent tab styling
- Icon support
- Lazy loading options
- Responsive behavior
```

#### ToggleButtonGroup Component
**File**: `src/components/common/ToggleButtonGroup.tsx`
**Priority**: Medium
**Used by**: PortalProjectsPage

```typescript
// Specification
interface ToggleButtonGroupProps<T = string> {
  value: T;
  onChange: (value: T) => void;
  options: ToggleOption<T>[];
  exclusive?: boolean;
  size?: 'small' | 'medium' | 'large';
  orientation?: 'horizontal' | 'vertical';
}

interface ToggleOption<T> {
  value: T;
  label: string;
  icon?: React.ReactNode;
  disabled?: boolean;
}

// Features Required:
- Design system styling
- Icon support
- Multiple selection mode
- Responsive behavior
```

### 1.5 Feedback Components

#### ConfirmationDialog Component
**File**: `src/components/common/ConfirmationDialog.tsx`
**Priority**: High
**Used by**: AdminUsersPage, AdminAccountsPage

```typescript
// Specification
interface ConfirmationDialogProps {
  open: boolean;
  onClose: () => void;
  onConfirm: () => void;
  title: string;
  message: string;
  confirmText?: string;
  cancelText?: string;
  severity?: 'info' | 'warning' | 'error';
  loading?: boolean;
}

// Features Required:
- Consistent dialog styling
- Loading states
- Severity-based styling
- Accessible keyboard navigation
```

#### LoadingOverlay Component
**File**: `src/components/common/LoadingOverlay.tsx`
**Priority**: Medium

```typescript
// Specification
interface LoadingOverlayProps {
  loading: boolean;
  children: React.ReactNode;
  message?: string;
  backdrop?: boolean;
}

// Features Required:
- Non-blocking loading states
- Optional backdrop
- Custom loading messages
- Smooth transitions
```

### 1.6 Admin-Specific Components

#### AdminCard Component
**File**: `src/components/common/AdminCard.tsx`
**Priority**: High
**Used by**: All admin pages

```typescript
// Specification
interface AdminCardProps {
  title: string;
  subtitle?: string;
  actions?: React.ReactNode;
  children: React.ReactNode;
  loading?: boolean;
  error?: string;
  icon?: React.ReactNode;
  variant?: 'standard' | 'outlined' | 'elevated';
}

// Features Required:
- Admin-specific styling
- Loading states
- Error states
- Action area
- Consistent spacing
```

#### StatusChip Component Enhancement
**File**: `src/components/common/StatusChip.tsx` (enhance existing StatusBadge)
**Priority**: Medium

```typescript
// Add more status types for admin usage
type AdminStatusType = 
  | 'ACTIVE' | 'INACTIVE' | 'TRIAL' | 'SUSPENDED' 
  | 'CANCELLED' | 'EXPIRED' | 'PENDING' | 'BLOCKED';

// Features Required:
- Admin status color mapping
- Size variants
- Icon integration
- Tooltip support
```

---

## Phase 2: Critical Page Migrations

### 2.1 Admin Login Page (Priority 1)
**File**: `src/pages/admin/AdminLoginPage.tsx`
**Timeline**: 1 day after Phase 1 components
**Dependencies**: FormSection, InputField, AdminCard

**Migration Tasks:**
1. Replace all MUI imports with design system components
2. Use AdminCard for login container
3. Use InputField for email/password inputs
4. Use Button from design system
5. Use Alert from design system
6. Ensure responsive behavior matches design system

### 2.2 Admin Dashboard Page (Priority 2)  
**File**: `src/pages/admin/AdminDashboardPage.tsx`
**Timeline**: 1 day
**Dependencies**: Grid, AdminCard, StatusChip

**Migration Tasks:**
1. Replace MuiGrid with design system Grid
2. Use AdminCard for dashboard widgets
3. Update status displays to use StatusChip
4. Ensure responsive layout

### 2.3 Admin Data Pages (Priority 3)
**Files**: 
- `src/pages/admin/AdminAccountsPage.tsx`
- `src/pages/admin/AdminUsersPage.tsx`  
**Timeline**: 2-3 days each
**Dependencies**: EnhancedTable, Pagination, SelectField, ConfirmationDialog

**Migration Tasks:**
1. Replace table implementation with EnhancedTable
2. Integrate Pagination component
3. Replace form controls with design system equivalents
4. Add ConfirmationDialog for actions
5. Use InputField for search functionality

### 2.4 Admin Detail Pages (Priority 4)
**Files**:
- `src/pages/admin/AdminAccountDetailPage.tsx`
- `src/pages/admin/AdminUserDetailPage.tsx`
- `src/pages/admin/AdminAccountEditPage.tsx`
**Timeline**: 1-2 days each  
**Dependencies**: TabPanel, FormSection, AdminCard

**Migration Tasks:**
1. Replace tabs with TabPanel component
2. Use FormSection for form organization
3. Replace all MUI components with design system equivalents
4. Ensure proper error handling and loading states

---

## Phase 3: Portal and Main App Migration

### 3.1 Portal Pages Grid Migration
**Timeline**: 3-4 days
**Files**: All portal pages with `Grid as MuiGrid` usage

**Migration Tasks:**
1. Replace all MuiGrid usage with design system Grid
2. Update responsive behavior
3. Test layout on all screen sizes

### 3.2 Missing Component Integration
**Timeline**: 2-3 days

**Migration Tasks:**
1. Replace ToggleButtonGroup usage in PortalProjectsPage
2. Update pagination in portal pages
3. Replace InputAdornment usage with enhanced InputField
4. Update loading indicators to use LoadingSpinner

---

## Implementation Guidelines

### Development Standards

1. **Component Development Order**:
   - Build components in dependency order
   - Test each component in isolation (Storybook)
   - Add TypeScript definitions
   - Include accessibility features

2. **Migration Standards**:
   - Never mix MUI and design system imports in the same file
   - Always import from design system first
   - Test responsive behavior
   - Verify accessibility compliance

3. **Testing Requirements**:
   - Unit tests for all new components
   - Integration tests for migrated pages
   - Visual regression testing
   - Accessibility testing

### Code Review Checklist

**For New Components:**
- [ ] Follows design system patterns
- [ ] Includes proper TypeScript definitions
- [ ] Has Storybook stories
- [ ] Includes accessibility features
- [ ] Has unit tests
- [ ] Exports from common/index.ts

**For Page Migrations:**
- [ ] No direct `@mui/material` imports
- [ ] Uses design system components exclusively  
- [ ] Maintains existing functionality
- [ ] Responsive behavior verified
- [ ] Error states handled properly
- [ ] Loading states implemented

---

## Current Issues Reference

> This section provides a quick reference of current issues for context. Full implementation details are in the phases above.

### Portal Pages Issues
- **Grid Usage**: 7 pages use `Grid as MuiGrid` directly
- **Missing Components**: ToggleButton, Pagination, InputAdornment usage
- **Theme Providers**: 3 login/auth pages use ThemeProvider directly

### Admin Pages Issues  
- **Complete Bypass**: All 9 admin pages use direct MUI imports exclusively
- **Table Components**: Heavy usage of MUI Table, TableBody, TableCell, etc.
- **Form Components**: Direct usage of TextField, Select, FormControl
- **Layout Components**: Direct usage of Box, Card, Paper, Grid

### Main App Issues
- **Stepper Components**: CustomReportBuilder needs Stepper wrapper
- **Loading States**: Inconsistent usage of CircularProgress vs LoadingSpinner
- **Form Components**: Various pages use direct MUI form components

---

## Success Criteria

### Phase 1 Success Criteria
- [ ] All missing design system components implemented
- [ ] Components have comprehensive TypeScript definitions
- [ ] Storybook stories created for all new components
- [ ] Unit tests written and passing
- [ ] Components exported from design system index
- [ ] Documentation updated

### Phase 2 Success Criteria
- [ ] All admin pages use design system components exclusively
- [ ] No direct MUI imports in admin pages
- [ ] Responsive behavior maintained
- [ ] All existing functionality preserved
- [ ] Error and loading states properly implemented
- [ ] Accessibility compliance verified

### Phase 3 Success Criteria
- [ ] All portal pages use design system Grid component
- [ ] Missing component usage eliminated
- [ ] Main app pages migration completed
- [ ] Design system adoption rate reaches 95%+
- [ ] Performance impact assessed and optimized

### Overall Success Criteria
- [ ] Zero direct MUI imports in feature code (excluding theme/infrastructure)
- [ ] Consistent styling across all admin, portal, and main app interfaces
- [ ] Reduced bundle size through consistent imports
- [ ] Improved developer experience with design system
- [ ] Maintained or improved accessibility scores
- [ ] No regression in existing functionality

---

## Timeline Summary

| Phase | Duration | Components | Pages | Dependencies |
|-------|----------|------------|--------|--------------|
| **Phase 1** | 2 weeks | 11 new components | 0 | None |
| **Phase 2** | 2 weeks | 0 | 9 admin pages | Phase 1 complete |
| **Phase 3** | 2 weeks | 0 | 15+ portal/main pages | Phase 1 complete |
| **Total** | **6 weeks** | **11 components** | **24+ pages** | Sequential |

## Risk Assessment

### High Risk Items
- **EnhancedTable Component**: Complex requirements, multiple dependencies
- **Admin User/Account Pages**: Heavy table usage, complex interactions
- **Responsive Behavior**: Ensuring all components work across devices

### Mitigation Strategies
1. **Incremental Development**: Build components iteratively with feedback
2. **Early Testing**: Test components in isolation before page integration
3. **Rollback Plan**: Maintain ability to rollback individual components/pages
4. **Performance Monitoring**: Track bundle size and runtime performance

### Dependencies
- Design system team availability
- QA testing resources
- Stakeholder approval for UI changes
- Development environment setup

---

## Next Steps

1. **Immediate Action**: Review and approve this specification
2. **Resource Allocation**: Assign development team for 6-week project
3. **Environment Setup**: Ensure Storybook and testing infrastructure ready
4. **Stakeholder Communication**: Notify users of upcoming UI consistency improvements
5. **Start Phase 1**: Begin with Grid and EnhancedTable components

---

**Document Status**: Ready for Implementation  
**Estimated Effort**: 6 weeks (1 senior developer)  
**Business Impact**: Critical - Resolves major technical debt and UI inconsistency