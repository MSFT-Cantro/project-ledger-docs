# ESLint Errors and Warnings

This document tracks all ESLint errors and warnings that need to be addressed in the codebase. Generated on September 19, 2025.

## Summary
- **Total Files with Issues**: 47
- **Main Issue Types**: 
  - Unused imports/variables
  - Missing dependencies in React hooks
  - React hooks exhaustive-deps violations

## Files and Issues

### Accessibility Components

#### `src/components/accessibility/AccessibleModal.tsx`
- Line 119: `'modalRef' is assigned a value but never used` (@typescript-eslint/no-unused-vars)
- Line 124: `'focusManagement' is assigned a value but never used` (@typescript-eslint/no-unused-vars)

### Client Components

#### `src/components/clients/ClientForm.tsx`
- Line 8: `'Stack' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 10: `'Paper' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 12: `'Divider' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 13: `'FormControl' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 14: `'InputLabel' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 15: `'Select' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 16: `'MenuItem' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 19: `'Email' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 19: `'Phone' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 19: `'LocationOn' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 19: `'Notes' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 21: `'AccessibleFormField' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 28: `'MobileChoiceGroup' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 83: `'getErrorMessages' is assigned a value but never used` (@typescript-eslint/no-unused-vars)

### Common Components

#### `src/components/common/FilterPanel.tsx`
- Line 11: `'Grid' is defined but never used` (@typescript-eslint/no-unused-vars)

### Dev Tools

#### `src/components/dev-tools/StorybookIntegration.tsx`
- Line 107: The 'mockStories' conditional could make dependencies of useMemo Hook change on every render (react-hooks/exhaustive-deps)

### Feedback Components

#### `src/components/feedback/Toast.tsx`
- Line 3: `'Alert' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 4: `'Snackbar' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 141: React Hook useEffect has missing dependency: 'handleClose' (react-hooks/exhaustive-deps)
- Line 149: React Hook useCallback has missing dependency: 'toast' (react-hooks/exhaustive-deps)

### Form Components

#### `src/components/forms/AutoComplete.tsx`
- Line 152: React Hook useCallback received a function whose dependencies are unknown (react-hooks/exhaustive-deps)

#### `src/components/forms/FileUpload.tsx`
- Line 30: `'Close' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 202: React Hook useCallback has missing dependency: 'handleUpload' (react-hooks/exhaustive-deps)

#### `src/components/forms/MobileFormValidation.tsx`
- Line 61: `'theme' is assigned a value but never used` (@typescript-eslint/no-unused-vars)

#### `src/components/forms/MobileOptimizedForm.tsx`
- Line 193: `'theme' is assigned a value but never used` (@typescript-eslint/no-unused-vars)

### Inventory Components

#### `src/components/inventory/InventoryForm.tsx`
- Line 13: `'Typography' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 16: `'UpdateInventoryItemInput' is defined but never used` (@typescript-eslint/no-unused-vars)

#### `src/components/inventory/InventorySelector.tsx`
- Line 1: `'useEffect' is defined but never used` (@typescript-eslint/no-unused-vars)

### Invoice Components

#### `src/components/invoices/InvoiceForm.tsx`
- Line 1: `'useMemo' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 14: `'IconButton' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 24: `'Inventory' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 25: `'DeleteIcon' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 26: `'InvoiceStatus' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 26: `'InvoiceItem' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 32: `'DebouncedNumberField' is defined but never used` (@typescript-eslint/no-unused-vars)

#### `src/components/invoices/PaymentHistory.tsx`
- Line 4: `'Paper' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 12: `'Payment' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 15: `'ResponsiveTableColumn' is defined but never used` (@typescript-eslint/no-unused-vars)

### Layout Components

#### `src/components/layout/MainLayout.tsx`
- Line 8: `'IconButton' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 20: `'MenuIcon' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 43: `'theme' is assigned a value but never used` (@typescript-eslint/no-unused-vars)
- Line 46: `'isMobile' is assigned a value but never used` (@typescript-eslint/no-unused-vars)

#### `src/components/layout/PageHeader.tsx`
- Line 16: `'HomeIcon' is defined but never used` (@typescript-eslint/no-unused-vars)

#### `src/components/layout/ResponsiveDataGrid.tsx`
- Line 15: `'Chip' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 24: `'Avatar' is defined but never used` (@typescript-eslint/no-unused-vars)

#### `src/components/layout/ResponsiveInfoCard.tsx`
- Line 9: `'Avatar' is defined but never used` (@typescript-eslint/no-unused-vars)

#### `src/components/layout/ResponsivePageLayout.tsx`
- Line 14: `'alpha' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 68: `'theme' is assigned a value but never used` (@typescript-eslint/no-unused-vars)

### Navigation Components

#### `src/components/navigation/CommandPalette.tsx`
- Line 22: `'History' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 23: `'TrendingUp' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 24: `'Keyboard' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 25: `'Launch' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 74: `'navigate' is assigned a value but never used` (@typescript-eslint/no-unused-vars)
- Line 166: React Hook useCallback has missing dependency: 'handleCommandExecute' (react-hooks/exhaustive-deps)

#### `src/components/navigation/EnhancedBreadcrumbs.tsx`
- Line 64: `'isOverflowing' is assigned a value but never used` (@typescript-eslint/no-unused-vars)

#### `src/components/navigation/MobileAppBar.tsx`
- Line 10: `'alpha' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 45: `'isTouchDevice' is assigned a value but never used` (@typescript-eslint/no-unused-vars)

### Project Components

#### `src/components/projects/ProjectDetailView.tsx`
- Line 38: `'Fab' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 39: `'Menu' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 40: `'ListItemButton' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 62: `'MoreVert' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 67: `'Pause' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 118: `'loading' is assigned a value but never used` (@typescript-eslint/no-unused-vars)
- Line 170: `'handleCreateMilestone' is assigned a value but never used` (@typescript-eslint/no-unused-vars)
- Line 249: `'handleCreateTask' is assigned a value but never used` (@typescript-eslint/no-unused-vars)
- Line 453: `'completedMilestones' is assigned a value but never used` (@typescript-eslint/no-unused-vars)

### Quote Components

#### `src/components/quotes/QuoteForm.tsx`
- Line 16: `'IconButton' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 18: `'Chip' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 19: `'Accordion' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 20: `'AccordionSummary' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 21: `'AccordionDetails' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 22: `'Autocomplete' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 27: `'Divider' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 29: `'Add' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 29: `'Delete' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 29: `'ExpandMore' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 29: `'Groups' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 34: `'PageGrid' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 34: `'PageGridItem' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 35: `'DebouncedNumberField' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 36: `'Client' is defined but never used` (@typescript-eslint/no-unused-vars)

### Settings Components

#### `src/components/settings/CollapsibleSettingsSection.tsx`
- Line 13: `'ExpandLess' is defined but never used` (@typescript-eslint/no-unused-vars)

#### `src/components/settings/MobileBottomNavigation.tsx`
- Line 6: `'Box' is defined but never used` (@typescript-eslint/no-unused-vars)

### Shared Components

#### `src/components/shared/LineItemSection.tsx`
- Line 20: `'useFieldArray' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 20: `'useWatch' is defined but never used` (@typescript-eslint/no-unused-vars)

### Subscription Components

#### `src/components/subscriptions/UsageDashboard.tsx`
- Line 9: `'Alert' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 11: `'Grid' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 18: `'TrendingUp' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 33: `'isFreePlan' is assigned a value but never used` (@typescript-eslint/no-unused-vars)

### Table Components

#### `src/components/tables/ResponsiveTable.tsx`
- Line 14: `'Chip' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 17: `'useMediaQuery' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 86: `'theme' is assigned a value but never used` (@typescript-eslint/no-unused-vars)
- Line 89: `'setVisibleColumns' is assigned a value but never used` (@typescript-eslint/no-unused-vars)

### Context

#### `src/context/SubscriptionContext.tsx`
- Line 8: `'PlanLimits' is defined but never used` (@typescript-eslint/no-unused-vars)

### Hooks

#### `src/hooks/useKeyboardShortcuts.ts`
- Line 46: The 'globalShortcuts' array makes dependencies of useEffect Hook change on every render (react-hooks/exhaustive-deps)

#### `src/hooks/useMobileNavigation.ts`
- Line 94: `'startX' is assigned a value but never used` (@typescript-eslint/no-unused-vars)

### Pages

#### `src/pages/AdminPanelPage.tsx`
- Line 58: `'CreateUserRequest' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 107: `'currentSubscription' is assigned a value but never used` (@typescript-eslint/no-unused-vars)

#### `src/pages/BillingPage.tsx`
- Line 6: `'Paper' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 21: `'Divider' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 34: `'PlanBadge' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 49: `'refreshBillingHistory' is assigned a value but never used` (@typescript-eslint/no-unused-vars)
- Line 68: React Hook useEffect has missing dependency: 'handlePayPalApproval' (react-hooks/exhaustive-deps)
- Line 83: `'handleOpenUpgradeDialog' is assigned a value but never used` (@typescript-eslint/no-unused-vars)

#### `src/pages/ClientDetailPage.tsx`
- Line 6: `'Typography' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 9: `'Button' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 10: `'Card' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 11: `'CardContent' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 13: `'Divider' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 24: `'Assignment' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 40: `'isSmallScreen' is assigned a value but never used` (@typescript-eslint/no-unused-vars)

#### `src/pages/ClientsPage.tsx`
- Line 9: `'Tooltip' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 24: `'Business' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 38: `'FilterButton' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 63: `'setSearchTerm' is assigned a value but never used` (@typescript-eslint/no-unused-vars)

#### `src/pages/InventoryPage.tsx`
- Line 17: `'Tooltip' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 23: `'IconButton' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 33: `'Close' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 37: `'MoreVert' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 39: `'TransitionProps' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 46: `'LoadingSpinner' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 48: `'FilterButton' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 86: `'setSearchTerm' is assigned a value but never used` (@typescript-eslint/no-unused-vars)
- Line 87: `'setCategoryFilter' is assigned a value but never used` (@typescript-eslint/no-unused-vars)
- Line 88: `'setActiveFilter' is assigned a value but never used` (@typescript-eslint/no-unused-vars)
- Line 229: `'categories' is assigned a value but never used` (@typescript-eslint/no-unused-vars)

#### `src/pages/InvoicesPage.tsx`
- Line 4: `'Container' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 8: `'IconButton' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 11: `'Card' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 12: `'CardContent' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 13: `'Tooltip' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 14: `'Breadcrumbs' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 15: `'Link' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 31: `'Home' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 34: `'MoreVert' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 36: `'RouterLink' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 47: `'FilterButton' is defined but never used` (@typescript-eslint/no-unused-vars)

#### `src/pages/LoginPage.tsx`
- Line 10: `'Divider' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 22: `'OAuthButtons' is defined but never used` (@typescript-eslint/no-unused-vars)

#### `src/pages/Phase3CDemo.tsx`
- Line 25: `'SmartBreadcrumbs' is defined but never used` (@typescript-eslint/no-unused-vars)

#### `src/pages/ProjectsPage.tsx`
- Line 6: `'FilterButton' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 11: `'IconButton' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 23: `'Folder' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 43: `'UpdateProjectMutationVariables' is defined but never used` (@typescript-eslint/no-unused-vars)

#### `src/pages/QuoteCreatePage.tsx`
- Line 9: `'Currency' is defined but never used` (@typescript-eslint/no-unused-vars)

#### `src/pages/QuotesPage.tsx`
- Line 7: `'IconButton' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 17: `'MoreVert' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 36: `'FilterButton' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 196: `'handleMenuOpen' is assigned a value but never used` (@typescript-eslint/no-unused-vars)

#### `src/pages/SettingsPage.tsx`
- Line 16: `'MenuItem' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 92: `'isMobile' is assigned a value but never used` (@typescript-eslint/no-unused-vars)

#### `src/pages/SignupPage.tsx`
- Line 13: `'Divider' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 16: `'CardActions' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 17: `'Grid' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 27: `'FormLabel' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 41: `'OAuthButtons' is defined but never used` (@typescript-eslint/no-unused-vars)

#### `src/pages/SubscriptionCancelPage.tsx`
- Line 5: `'Box' is defined but never used` (@typescript-eslint/no-unused-vars)

#### `src/pages/SubscriptionSuccessPage.tsx`
- Line 5: `'Box' is defined but never used` (@typescript-eslint/no-unused-vars)
- Line 58: React Hook useEffect has missing dependency: 'handlePayPalApproval' (react-hooks/exhaustive-deps)

## Recommended Fixes

### High Priority
1. **React Hooks Dependencies** - Fix all `react-hooks/exhaustive-deps` violations as these can cause bugs
2. **Unused State Variables** - Remove or implement unused state setters that indicate incomplete functionality

### Medium Priority
1. **Unused Imports** - Remove unused imports to reduce bundle size
2. **Unused Variables** - Clean up unused variables and functions

### Low Priority
1. **Unused Type Imports** - Remove unused type definitions

## Fix Strategy

1. **Automated Cleanup**: Use ESLint auto-fix where possible
2. **Batch Processing**: Group similar files and fix by category
3. **Functionality Review**: For unused handlers/functions, determine if they should be implemented or removed
4. **Dependency Review**: Add missing dependencies to useEffect/useCallback hooks

## Notes
- Some unused imports may be intentionally kept for future development
- React hooks dependency issues should be prioritized as they can cause runtime bugs
- Consider adding ESLint rules to pre-commit hooks to prevent future accumulation