# Technical Debt Tracking

*Last Updated: September 15, 2025*

This document tracks technical debt items identified during development and maintenance of Project Ledger. Items are prioritized and categorized for systematic resolution.

## Quick Stats

| Category | Count | Priority Distribution |
|----------|-------|---------------------|
| Code Quality | 146 | High: 0, Medium: 13, Low: 133 |
| Testing | 3 | High: 2, Medium: 1, Low: 0 |
| Dependencies | 1 | High: 0, Medium: 1, Low: 0 |
| **Total** | **150** | **High: 2, Medium: 15, Low: 133** |

---

## Code Quality Issues (146 items)

### ESLint Warnings - Unused Imports and Variables (133 items) 
*Priority: Low | Effort: 1-2 days*

**Impact**: Code bloat, potential confusion during development, larger bundle size

**Files Affected**: 41 files across the frontend codebase

**Top Offenders**:
- `src/components/auth/SignupPage.tsx` - 7 unused imports
- `src/components/quotes/QuoteForm.tsx` - 13 unused imports
- `src/components/projects/ProjectDetailView.tsx` - 8 unused imports
- `src/components/invoices/InvoiceForm.tsx` - 7 unused imports

**Resolution Strategy**:
1. Run `npm run lint:fix` to auto-remove unused imports
2. Manual review for complex cases
3. Add pre-commit hook to prevent future accumulation

**Example Issues**:
```typescript
// Unused imports to remove
import { List, ListItem, ListItemIcon, ListItemText } from '@mui/material';
import { FormLabel } from '@mui/material';
import { CheckIcon } from '@mui/icons-material';
```

### ESLint Errors - Testing Library Violations (13 items)
*Priority: Medium | Effort: 4-6 hours*

**Impact**: Test reliability, maintenance issues, potential false positives/negatives

**Files Affected**:
- `src/components/projects/ProjectDetailView.test.tsx` - 12 errors
- Other test files - 1 error

**Specific Issues**:
1. **Direct Node Access (12 occurrences)** - Lines 122, 123, 150, 151, 196
   ```typescript
   // Bad: Direct DOM access
   expect(container.querySelector('.some-class')).toBeInTheDocument();
   
   // Good: Use Testing Library queries
   expect(screen.getByRole('button')).toBeInTheDocument();
   ```

2. **Multiple Assertions in waitFor (1 occurrence)** - Line 196
   ```typescript
   // Bad: Multiple assertions in waitFor
   await waitFor(() => {
     expect(element1).toBeInTheDocument();
     expect(element2).toBeInTheDocument();
   });
   
   // Good: Single assertion per waitFor
   await waitFor(() => expect(element1).toBeInTheDocument());
   await waitFor(() => expect(element2).toBeInTheDocument());
   ```

**Resolution Strategy**:
1. Refactor direct DOM queries to use Testing Library methods
2. Split multiple assertions in waitFor callbacks
3. Update test documentation and guidelines

---

## Testing Issues (3 items)

### React Router DOM Mock Issues (2 items)
*Priority: High | Effort: 2-4 hours*

**Impact**: Tests failing, CI/CD pipeline issues, inability to test routing functionality

**Files Affected**:
- `src/App.test.tsx`
- `src/components/projects/ProjectDetailView.test.tsx`

**Root Cause**: Missing or incomplete mocking of `react-router-dom` in test environment

**Error Messages**:
```
Cannot find module 'react-router-dom' from 'src/App.tsx'
Cannot find module 'react-router-dom' from 'src/components/common/UserSettingsModal.tsx'
```

**Resolution Strategy**:
1. Add comprehensive react-router-dom mocks to setupTests.ts
2. Create test utilities for router-dependent components
3. Consider using @testing-library/react-router for better testing

**Implementation**:
```typescript
// Add to setupTests.ts
jest.mock('react-router-dom', () => ({
  ...jest.requireActual('react-router-dom'),
  useNavigate: () => jest.fn(),
  useLocation: () => ({ pathname: '/', search: '', hash: '', state: null }),
  useParams: () => ({}),
}));
```

### React Hook Dependencies (1 item)
*Priority: Medium | Effort: 1-2 hours*

**Impact**: Potential runtime bugs, stale closures, unexpected re-renders

**Files Affected**: Multiple files with useEffect and useCallback warnings

**Examples**:
- `src/components/feedback/Toast.tsx` - Missing 'handleClose' dependency
- `src/components/navigation/CommandPalette.tsx` - Missing 'handleCommandExecute' dependency
- `src/pages/BillingPage.tsx` - Missing 'handlePayPalApproval' dependency

**Resolution Strategy**:
1. Add missing dependencies to hook dependency arrays
2. Use useCallback for stable function references
3. Consider splitting complex effects

---

## Dependencies (1 item)

### TypeScript Version Compatibility Warning (1 item)
*Priority: Medium | Effort: 1 hour*

**Impact**: Potential compatibility issues, unsupported features, warning noise

**Current Version**: TypeScript 5.9.2  
**Supported Range**: >=3.3.1 <5.2.0 (by @typescript-eslint)

**Warning Message**:
```
WARNING: You are currently running a version of TypeScript which is not officially supported by @typescript-eslint/typescript-estree.
```

**Resolution Options**:
1. **Downgrade TypeScript** to 5.1.x (safer, immediate)
2. **Upgrade @typescript-eslint** packages (if available)
3. **Monitor for official support** in newer eslint versions

**Recommendation**: Option 1 (downgrade) for stability

---

## Resolution Roadmap

### Immediate (Next Sprint)
- [ ] **Fix React Router mocks** (High priority, blocks testing)
- [ ] **Fix Testing Library violations** (Medium priority, affects test quality)

### Short Term (Next 2 weeks)
- [ ] **Run automated lint cleanup** for unused imports
- [ ] **Fix React Hook dependencies** 
- [ ] **Resolve TypeScript version compatibility**

### Long Term (Next Month)
- [ ] **Implement pre-commit hooks** to prevent unused import accumulation
- [ ] **Create testing guidelines** and utilities for router-dependent components
- [ ] **Set up automated dependency scanning** for compatibility issues

---

## Monitoring and Prevention

### Automated Checks
1. **Pre-commit hooks**: ESLint with auto-fix for unused imports
2. **CI/CD pipeline**: Fail builds on high-priority tech debt
3. **Weekly reviews**: Automated tech debt reports

### Metrics to Track
- Number of ESLint warnings/errors
- Test coverage and reliability
- Build success rate
- Time spent on tech debt resolution

### Guidelines
1. **New code**: Must pass all ESLint checks
2. **Pull requests**: Should not introduce new unused imports
3. **Testing**: All new components must have proper Testing Library usage
4. **Dependencies**: Regular updates with compatibility verification

---

## Historical Context

### September 15, 2025 - Accessibility Implementation Review
**Discovered During**: Phase 1.1 accessibility tooling setup  
**Context**: While setting up jsx-a11y ESLint plugin, discovered existing code quality issues  
**Action Taken**: Documented for systematic resolution, did not block accessibility implementation  

### Notes
- Issues are primarily cosmetic and don't affect functionality
- Accessibility implementation can proceed without resolving these items
- Most issues can be resolved with automated tooling
- Testing issues should be prioritized as they affect development workflow