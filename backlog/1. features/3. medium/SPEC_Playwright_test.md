# Playwright E2E Testing Suite Specification

**Date:** November 12, 2025  
**Version:** 1.0  
**Status:** Ready for Implementation  
**Priority:** High - Critical for ensuring application quality, preventing regressions, and enabling confident deployments  
**Location:** `backlog/1. features/3. medium/SPEC_Playwright_test.md`  
**Completed:** TBD

---

## 📊 Implementation Progress Summary

### Overall Completion: 0% Complete (0/8 phases)

```
Phase 1: Foundation & Auth       ░░░░░░░░░░░░░░░░░░░░  0% 🔴
Phase 2: Clients & Projects      ░░░░░░░░░░░░░░░░░░░░  0% 🔴
Phase 3: Quotes & Invoices       ░░░░░░░░░░░░░░░░░░░░  0% 🔴
Phase 4: Inventory & Reports     ░░░░░░░░░░░░░░░░░░░░  0% 🔴
Phase 5: Settings & Admin        ░░░░░░░░░░░░░░░░░░░░  0% 🔴
Phase 6: Multi-Org & Auth        ░░░░░░░░░░░░░░░░░░░░  0% 🔴
Phase 7: A11y & Performance      ░░░░░░░░░░░░░░░░░░░░  0% 🔴
Phase 8: CI/CD Integration       ░░░░░░░░░░░░░░░░░░░░  0% 🔴
```

### 🎯 Current Status
**Phase:** Pre-Implementation Planning  
**Started:** Not Started  
**Status:** Specification complete, awaiting review and approval to begin Phase 1

### Quick Reference
- **Authentication Tests**: 🔴 Not Started (Target: 15+ tests)
- **CRUD Tests**: 🔴 Not Started (Target: 110+ tests)
- **Business Logic Tests**: 🔴 Not Started (Target: 40+ tests)
- **Accessibility Tests**: 🔴 Not Started (Target: 15+ tests)
- **Performance Tests**: 🔴 Not Started (Target: 10+ tests)
- **Mobile Tests**: 🔴 Not Started (Target: 20+ tests)
- **CI/CD Pipeline**: 🔴 Not Started

### Progress Indicators

- 🔴 Not Started
- 🟡 In Progress
- 🟢 Complete
- ⚠️ Blocked
- ⏸️ Paused

---

## 📋 Overview

### Problem Statement
Currently, the Project Ledger application has only unit tests (Jest + React Testing Library) but no end-to-end testing. This creates significant risk:
- Manual testing is time-consuming and error-prone
- Regressions go undetected until production
- Cross-browser compatibility issues are discovered late
- Accessibility violations are not systematically caught
- Complex user workflows remain untested
- CI/CD pipeline lacks comprehensive validation

### Solution
Implement a comprehensive Playwright E2E testing suite covering all major user workflows, CRUD operations, and business logic across the entire application.

### Key Benefits
1. **Quality Assurance**: Catch bugs before production deployment
2. **Regression Prevention**: Automated detection of breaking changes
3. **Cross-Browser Validation**: Test across Chrome, Firefox, Safari (WebKit)
4. **Mobile Testing**: Validate responsive design and mobile interactions
5. **Accessibility Compliance**: Automated WCAG validation
6. **Performance Monitoring**: Track page load and API response times
7. **Confident Deployments**: Comprehensive test coverage enables faster releases
8. **Documentation**: Tests serve as living documentation of user workflows

### Scope
**In Scope:**
- 230+ E2E tests covering all application features
- Page Object Model architecture for maintainability
- CI/CD integration with GitHub Actions
- Cross-browser testing (Chrome, Firefox, Safari)
- Mobile device testing (iOS, Android)
- Accessibility testing (keyboard navigation, WCAG compliance)
- Performance monitoring (page load, API response times)
- Test data management and cleanup utilities

**Implementation Status:**
- **Phase 1 (Foundation & Auth)**: 🔴 Not Started - 15+ tests
- **Phase 2 (Clients & Projects)**: 🔴 Not Started - 30+ tests
- **Phase 3 (Quotes & Invoices)**: 🔴 Not Started - 40+ tests
- **Phase 4 (Inventory & Reports)**: 🔴 Not Started - 30+ tests
- **Phase 5 (Settings & Admin)**: 🔴 Not Started - 45+ tests
- **Phase 6 (Multi-Org & Auth)**: 🔴 Not Started - 25+ tests
- **Phase 7 (A11y & Performance)**: 🔴 Not Started - 45+ tests
- **Phase 8 (CI/CD Integration)**: 🔴 Not Started - 5+ tests

---

## 🎯 Scope & Requirements

### Functional Requirements

| Requirement ID | Description | Priority | Status |
|----------------|-------------|----------|--------|
| FR-001 | Test all authentication flows (login, signup, OAuth, logout) | Critical | 🔴 Not Started |
| FR-002 | Test CRUD operations for all entities (Clients, Projects, Quotes, Invoices, Inventory) | Critical | 🔴 Not Started |
| FR-003 | Test business workflows (Quote→Invoice, Payment recording, Report generation) | High | 🔴 Not Started |
| FR-004 | Test multi-organization features (org selector, org switching, data isolation) | High | 🔴 Not Started |
| FR-005 | Test navigation and search functionality | High | 🔴 Not Started |
| FR-006 | Test settings and configuration pages | Medium | 🔴 Not Started |
| FR-007 | Test admin panel features (user management, integrations) | Medium | 🔴 Not Started |
| FR-008 | Test form validation and error handling | High | 🔴 Not Started |
| FR-009 | Test PDF generation for quotes and invoices | Medium | 🔴 Not Started |
| FR-010 | Test email notifications and templates | Low | 🔴 Not Started |

### Non-Functional Requirements

| Requirement ID | Description | Target | Status |
|----------------|-------------|--------|--------|
| NFR-001 | Test Execution Time | < 30 minutes for full suite | 🔴 Not Started |
| NFR-002 | Test Stability | < 5% flaky tests | 🔴 Not Started |
| NFR-003 | Cross-Browser Coverage | 100% tests pass on Chrome, Firefox, Safari | 🔴 Not Started |
| NFR-004 | Mobile Coverage | 80% critical tests pass on iOS/Android | 🔴 Not Started |
| NFR-005 | Page Load Performance | All pages < 3 seconds | 🔴 Not Started |
| NFR-006 | API Response Time | All endpoints < 500ms | 🔴 Not Started |
| NFR-007 | Test Code Maintainability | Page Object Model with DRY principles | 🔴 Not Started |
| NFR-008 | CI/CD Integration | Tests run automatically on every PR | 🔴 Not Started |
| NFR-009 | Test Coverage | 85%+ coverage of critical user flows | 🔴 Not Started |
| NFR-010 | Accessibility Compliance | 70%+ WCAG 2.1 AA validation | 🔴 Not Started |

### Out of Scope

- **Load Testing**: Use Artillery or similar tools (separate initiative)
- **Visual Regression Testing**: Percy integration (future enhancement)
- **API-Only Testing**: Use Postman/Newman for isolated API tests
- **Security Testing**: Penetration testing (separate security audit)
- **Internationalization Testing**: Multi-language validation (future phase)
- **Third-Party Integrations**: Stripe, OAuth providers (mock in tests)
- **Database Performance Testing**: Query optimization (separate DB audit)

---

## 🏗️ Architecture & Design

### System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      Playwright Test Suite                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │  Test Files  │  │ Page Objects │  │   Fixtures   │         │
│  │  (*.spec.ts) │  │   (.page.ts) │  │ (auth, data) │         │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘         │
│         │                  │                  │                 │
│         └──────────────────┴──────────────────┘                 │
│                            │                                    │
│                    ┌───────▼────────┐                          │
│                    │ Playwright API │                          │
│                    └───────┬────────┘                          │
└────────────────────────────┼─────────────────────────────────┘
                             │
              ┌──────────────┴──────────────┐
              │                             │
    ┌─────────▼─────────┐         ┌────────▼────────┐
    │   Frontend App    │         │   Backend API   │
    │ (React + Router)  │◄────────┤  (Express + DB) │
    └───────────────────┘         └─────────────────┘
              │                             │
    ┌─────────▼─────────┐         ┌────────▼────────┐
    │  Chrome/Firefox   │         │   PostgreSQL    │
    │     /WebKit       │         │    Database     │
    └───────────────────┘         └─────────────────┘
```

### Current Application State Analysis

### Application Structure

**Core Pages (CRUD Functionality)**:
- **Clients**: List, Create, View, Edit, Delete
- **Projects**: List, Create, View, Edit, Delete
- **Quotes**: List, Create, View, Edit, Delete
- **Invoices**: List, Create, View, Edit, Delete
- **Inventory**: List, Create, View, Edit, Delete
- **Reports**: List, Create, View, Edit (Custom Reports)

**Supporting Pages**:
- **Dashboard**: Overview, metrics, recent activity
- **Settings**: User, Security, Notifications, Appearance, Billing, Company
- **Admin Panel**: User management, integrations
- **Authentication**: Login, Signup, OAuth callbacks
- **Organization Selector**: Multi-org selection and switching

**Authentication Flow**:
- Standard email/password login
- OAuth (Google, Microsoft)
- Multi-organization support
- Role-based access (ADMIN, USER)

### Technology Stack

**Frontend**:
- React 19.1.1 with TypeScript
- React Router 7.9.1 for routing
- Material-UI 7.3.1 for components
- React Hook Form 7.62.0 for forms
- Axios for API calls

**Backend**:
- Node.js with Express
- PostgreSQL database
- JWT authentication
- RESTful API

**Testing Infrastructure** (Current):
- Jest with React Testing Library (unit tests)
- No E2E tests currently implemented

### Test Structure

```
apps/frontend/
├── e2e/                          # Playwright E2E tests
│   ├── fixtures/                 # Test fixtures and helpers
│   │   ├── auth.fixture.ts       # Authentication helpers
│   │   ├── test-data.ts          # Seed data for tests
│   │   └── api-helpers.ts        # API interaction helpers
│   ├── page-objects/             # Page Object Model
│   │   ├── base.page.ts          # Base page class
│   │   ├── auth/                 # Authentication pages
│   │   ├── clients/              # Client pages
│   │   ├── projects/             # Project pages
│   │   ├── quotes/               # Quote pages
│   │   ├── invoices/             # Invoice pages
│   │   ├── inventory/            # Inventory pages
│   │   ├── reports/              # Report pages
│   │   ├── settings/             # Settings pages
│   │   ├── admin/                # Admin panel pages
│   │   └── common/               # Shared components (nav, search)
│   ├── tests/                    # Test files organized by feature
│   │   ├── auth/                 # Authentication tests
│   │   ├── clients/              # Client CRUD tests
│   │   ├── projects/             # Project CRUD tests
│   │   ├── quotes/               # Quote tests
│   │   ├── invoices/             # Invoice tests
│   │   ├── inventory/            # Inventory tests
│   │   ├── reports/              # Report tests
│   │   ├── settings/             # Settings tests
│   │   ├── admin/                # Admin tests
│   │   ├── navigation/           # Navigation tests
│   │   ├── accessibility/        # A11y tests
│   │   └── performance/          # Performance tests
│   ├── utils/                    # Utility functions
│   │   ├── database.ts           # Database seeding/cleanup
│   │   ├── test-users.ts         # Test user management
│   │   ├── screenshot.ts         # Screenshot helpers
│   │   └── wait-for.ts           # Custom wait utilities
│   └── playwright.config.ts      # Playwright configuration
├── playwright-report/            # Test reports (gitignored)
└── test-results/                 # Test artifacts (gitignored)
```

### Component Specifications

#### Component 1: Base Page Object
**File/Location:** `apps/frontend/e2e/page-objects/base.page.ts`  
**Priority:** Critical  
**Dependencies:** @playwright/test

**Purpose:**
Base class providing common functionality for all page objects, including navigation, element interaction, and waiting utilities.

**Specification:**
```typescript
import { Page, Locator } from '@playwright/test';

export class BasePage {
  constructor(protected page: Page) {}

  async navigate(path: string): Promise<void>;
  async waitForPageLoad(): Promise<void>;
  async screenshot(name: string): Promise<void>;
  async getElement(selector: string): Promise<Locator>;
  async clickButton(text: string): Promise<void>;
  async fillInput(label: string, value: string): Promise<void>;
  async expectVisible(selector: string): Promise<void>;
}
```

**Features Required:**
- Navigation with automatic wait
- Common element interactions (click, fill, select)
- Screenshot capture for debugging
- Custom wait strategies
- Error handling and retries

#### Component 2: Authentication Fixture
**File/Location:** `apps/frontend/e2e/fixtures/auth.fixture.ts`  
**Priority:** Critical  
**Dependencies:** @playwright/test, BasePage

**Purpose:**
Provides reusable authentication states for tests, enabling fast test execution by reusing login sessions.

**Specification:**
```typescript
import { test as base } from '@playwright/test';

type AuthFixtures = {
  authenticatedPage: Page;
  loginAsAdmin: () => Promise<void>;
  loginAsUser: () => Promise<void>;
};

export const test = base.extend<AuthFixtures>({
  authenticatedPage: async ({ page }, use) => {
    // Login and provide authenticated page
  },
  loginAsAdmin: async ({ page }, use) => {
    // Admin login helper
  },
  loginAsUser: async ({ page }, use) => {
    // User login helper
  }
});
```

**Features Required:**
- Reusable authentication states
- Role-based login helpers (admin, user)
- Storage state management
- Session persistence

#### Component 3: Page Object Classes
**File/Location:** `apps/frontend/e2e/page-objects/[feature]/`  
**Priority:** High  
**Dependencies:** BasePage

**Purpose:**
Encapsulate page-specific interactions and selectors using the Page Object Model pattern for maintainability.

**Features Required:**
- One page object per page/component
- Semantic method names (e.g., `clickNewClient()`)
- Stable selectors (ARIA roles, labels, data-testid)
- Type-safe method signatures
- Reusable across multiple tests

#### Component 4: Test Data Management
**File/Location:** `apps/frontend/e2e/utils/database.ts`  
**Priority:** High  
**Dependencies:** Prisma, test database

**Purpose:**
Manage test data creation, seeding, and cleanup to ensure test isolation and prevent data pollution.

**Features Required:**
- Database seeding utilities
- Unique data generation (timestamps, UUIDs)
- Automatic cleanup after tests
- Test user creation/deletion
- Transaction rollback support

---

## 🔧 Technical Implementation

### Test Framework Configuration

#### Playwright Configuration
**File:** `apps/frontend/playwright.config.ts`

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e/tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  
  reporter: [
    ['html'],
    ['json', { outputFile: 'test-results/results.json' }],
    ['junit', { outputFile: 'test-results/junit.xml' }]
  ],
  
  use: {
    baseURL: process.env.PLAYWRIGHT_BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],

  webServer: {
    command: 'npm run start',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120000,
  },
});
```

**Configuration Features:**
- Multi-browser testing (Chrome, Firefox, Safari)
- Mobile device emulation (iOS, Android)
- Automatic server startup
- Multiple report formats (HTML, JSON, JUnit)
- Video/screenshot capture on failure
- Trace recording for debugging
- Parallel test execution

### Test Data Strategy

**Approach:** Isolated test data per test with automatic cleanup

**Data Management:**
1. **Unique Identifiers**: Use timestamps or UUIDs for test data
2. **Test Isolation**: Each test creates its own data
3. **Parallel-Safe**: No dependencies between tests
4. **Automatic Cleanup**: Cleanup hooks in fixtures
5. **Dedicated Test DB**: Separate database for testing

**Example Test Data Utility:**
```typescript
// apps/frontend/e2e/utils/test-data.ts
export function generateTestClient() {
  return {
    name: `Test Client ${Date.now()}`,
    email: `client-${Date.now()}@test.com`,
    phone: '555-1234',
    address: '123 Test St',
    city: 'Test City',
    province: 'BC',
    postalCode: 'V1V 1V1',
    country: 'Canada'
  };
}

export async function cleanupTestData(clientIds: number[]) {
  // Cleanup logic
}
```

### Example Test Implementation

#### Login Page Object
```typescript
// apps/frontend/e2e/page-objects/auth/login.page.ts
import { Page } from '@playwright/test';
import { BasePage } from '../base.page';

export class LoginPage extends BasePage {
  constructor(page: Page) {
    super(page);
  }

  async navigate() {
    await this.page.goto('/login');
    await this.waitForPageLoad();
  }

  async login(email: string, password: string) {
    await this.fillInput('Email', email);
    await this.fillInput('Password', password);
    await this.clickButton('Log In');
    
    // Wait for redirect to dashboard or org selector
    await this.page.waitForURL(/\/(dashboard|select-organization)?/, { 
      timeout: 10000 
    });
  }

  async loginAndExpectError(email: string, password: string) {
    await this.fillInput('Email', email);
    await this.fillInput('Password', password);
    await this.clickButton('Log In');
    
    // Should stay on login page with error
    await this.expectVisible('[role="alert"]');
  }

  async getErrorMessage() {
    return await this.page.locator('[role="alert"]').textContent();
  }
}
```

#### Login Test Suite
```typescript
// apps/frontend/e2e/tests/auth/login.spec.ts
import { test, expect } from '../../fixtures/auth.fixture';
import { LoginPage } from '../../page-objects/auth/login.page';

test.describe('Login Flow', () => {
  test('should successfully log in with valid credentials', async ({ page }) => {
    const loginPage = new LoginPage(page);
    await loginPage.navigate();
    await loginPage.login('demo@projectledger.com', 'admin123');
    
    // Should be redirected to dashboard or org selector
    await expect(page).toHaveURL(/\/(dashboard|select-organization)?/);
  });

  test('should show error with invalid credentials', async ({ page }) => {
    const loginPage = new LoginPage(page);
    await loginPage.navigate();
    await loginPage.loginAndExpectError('invalid@example.com', 'wrong');
    
    const error = await loginPage.getErrorMessage();
    expect(error).toContain('Invalid');
  });

  test('should validate required fields', async ({ page }) => {
    const loginPage = new LoginPage(page);
    await loginPage.navigate();
    await loginPage.clickButton('Log In');
    
    // Should show validation errors
    await loginPage.expectVisible('text=Email is required');
    await loginPage.expectVisible('text=Password is required');
  });
});
```

---

## 🔐 Security & Compliance

### Authentication & Authorization

**Authentication Method:** JWT tokens stored in localStorage/cookies
- Tests use dedicated test accounts (demo@projectledger.com, admin@projectledger.com)
- Storage state persisted for fast test execution
- Session expiry testing included

**Authorization Rules:**
- **ADMIN role**: Access to admin panel, company settings, user management
- **USER role**: Access to own organization data only
- Data isolation verified in multi-org tests

### Data Security

**Test Environment Security:**
- Dedicated test database (not production)
- Test accounts with fake/generated data
- No real user PII in test data
- Environment variables for sensitive config

**Data Privacy:**
- Test data automatically cleaned up
- No persistent storage of test artifacts in VCS
- Screenshots/videos excluded from git (gitignored)

### Audit Trail

**Logged Events:**
- Test execution start/end times
- Test failures with screenshots/videos
- Performance metrics (page load, API response times)

**Logged Information:**
- Test name and suite
- Browser/device used
- Timestamp
- Pass/fail status
- Error messages and stack traces
- Screenshot/video URLs

---

## 📋 Implementation Plan

### Phase 1: Foundation & Setup (Week 1)

#### Day 1-2: Playwright Installation & Configuration
**Developer:** QA Engineer (Lead)  
**Tasks:**
1. Install Playwright and dependencies
   - Run: `npm install --save-dev @playwright/test`
   - Install browsers: `npx playwright install`
2. Create Playwright configuration file
   - Configure browsers (Chrome, Firefox, Safari)
   - Configure mobile devices (Pixel 5, iPhone 12)
   - Set up reporters (HTML, JSON, JUnit)
3. Create base directory structure
   - `/e2e/tests`, `/e2e/page-objects`, `/e2e/fixtures`, `/e2e/utils`

**Success Criteria:**
- [ ] Playwright installed and `npx playwright --version` works
- [ ] playwright.config.ts created and validates
- [ ] Sample test runs successfully
- [ ] Test reports generate correctly

#### Day 3-5: Base Infrastructure & Authentication
**Developer:** QA Engineer (Lead) + Frontend Developer  
**Tasks:**
1. Create BasePage class with common utilities
2. Implement authentication fixture (reusable login states)
3. Create LoginPage page object
4. Write 15+ authentication tests
   - Valid login, invalid credentials, validation errors
   - OAuth flows (Google, Microsoft)
   - Logout, session expiry
5. Set up test data management utilities
6. Configure test database and cleanup hooks

**Success Criteria:**
- [ ] BasePage class created with essential methods
- [ ] Authentication fixture working (reusable login)
- [ ] LoginPage page object complete
- [ ] 15+ authentication tests passing
- [ ] All tests pass on Chrome, Firefox, Safari
- [ ] Test cleanup working (no data pollution)

---

### Phase 2: Core CRUD Tests - Clients & Projects (Week 2)

#### Day 1-3: Client CRUD Tests
**Developer:** QA Engineer (Lead)  
**Tasks:**
1. Create Client page objects
   - ClientsListPage, ClientCreatePage, ClientDetailPage, ClientEditPage
2. Write 15+ Client CRUD tests
   - Create client (valid data, validation errors)
   - View client details
   - Edit client (update name, email, address)
   - Delete client (with confirmation)
   - Search clients
   - Pagination and sorting

**Success Criteria:**
- [ ] 4 Client page objects created
- [ ] 15+ Client tests passing
- [ ] Tests validate form errors
- [ ] Tests verify data persistence
- [ ] Tests clean up created data

#### Day 4-5: Project CRUD Tests
**Developer:** QA Engineer (Lead)  
**Tasks:**
1. Create Project page objects
   - ProjectsListPage, ProjectCreatePage, ProjectDetailPage, ProjectEditPage
2. Write 15+ Project CRUD tests
   - Create project (with client association)
   - View project details
   - Edit project (status, dates, budget)
   - Delete project
   - Search and filter projects

**Success Criteria:**
- [ ] 4 Project page objects created
- [ ] 15+ Project tests passing
- [ ] Tests verify client-project association
- [ ] Tests validate project status changes
- [ ] Tests clean up created data

---

### Phase 3: Business Logic Tests - Quotes & Invoices (Week 3)

#### Day 1-3: Quote Tests
**Developer:** QA Engineer (Lead) + Backend Developer  
**Tasks:**
1. Create Quote page objects
   - QuotesListPage, QuoteCreatePage, QuoteDetailPage, QuoteEditPage
2. Write 20+ Quote tests
   - Create quote with line items
   - Calculate totals (subtotal, tax, total)
   - Quote status lifecycle (Draft → Sent → Accepted/Rejected)
   - Convert quote to invoice
   - Export quote to PDF
   - Send quote to client (email)

**Success Criteria:**
- [ ] 4 Quote page objects created
- [ ] 20+ Quote tests passing
- [ ] Tests verify calculations (subtotal, tax, total)
- [ ] Tests verify quote→invoice conversion
- [ ] Tests verify PDF generation

#### Day 4-5: Invoice Tests
**Developer:** QA Engineer (Lead) + Backend Developer  
**Tasks:**
1. Create Invoice page objects
   - InvoicesListPage, InvoiceCreatePage, InvoiceDetailPage, InvoiceEditPage
2. Write 20+ Invoice tests
   - Create invoice with line items
   - Apply tax rates
   - Record payments
   - Invoice status lifecycle (Draft → Sent → Paid → Overdue)
   - Export invoice to PDF
   - Send invoice to client

**Success Criteria:**
- [ ] 4 Invoice page objects created
- [ ] 20+ Invoice tests passing
- [ ] Tests verify payment recording
- [ ] Tests verify status transitions
- [ ] Tests verify PDF generation

---

### Phase 4: Inventory & Reports (Week 4)

#### Day 1-2: Inventory Tests
**Developer:** QA Engineer (Lead)  
**Tasks:**
1. Create Inventory page objects
2. Write 15+ Inventory tests
   - Create inventory item (name, SKU, price, stock)
   - Update inventory (stock levels, price changes)
   - Delete inventory item
   - Track low stock warnings
   - Categories and tags

**Success Criteria:**
- [ ] Inventory page objects created
- [ ] 15+ Inventory tests passing
- [ ] Tests verify stock tracking
- [ ] Tests verify low stock alerts

#### Day 3-5: Reports Tests
**Developer:** QA Engineer (Lead)  
**Tasks:**
1. Create Report page objects
2. Write 15+ Report tests
   - Standard reports (Invoice Aging, Revenue, Profitability)
   - Custom report builder
   - Report filters and date ranges
   - Export reports (PDF, CSV, Excel)
   - Save custom reports

**Success Criteria:**
- [ ] Report page objects created
- [ ] 15+ Report tests passing
- [ ] Tests verify report calculations
- [ ] Tests verify export functionality

---

### Phase 5: Settings, Admin, and Navigation (Week 5)

#### Day 1-2: Settings Tests
**Developer:** QA Engineer (Lead)  
**Tasks:**
1. Create Settings page objects (all tabs)
2. Write 20+ Settings tests
   - User Profile (update name, email, avatar)
   - Security (change password, 2FA)
   - Notifications (email preferences)
   - Appearance (theme, language)
   - Company Settings (ADMIN only)
   - Billing (subscription, payment methods)

**Success Criteria:**
- [ ] Settings page objects created (6 tabs)
- [ ] 20+ Settings tests passing
- [ ] Tests verify role-based access (ADMIN vs USER)

#### Day 3-4: Admin Panel Tests
**Developer:** QA Engineer (Lead)  
**Tasks:**
1. Create Admin page objects
2. Write 15+ Admin tests
   - User management (create, edit, delete users)
   - User roles (ADMIN, USER)
   - Integrations (OAuth setup)
   - Organization settings

**Success Criteria:**
- [ ] Admin page objects created
- [ ] 15+ Admin tests passing
- [ ] Tests verify ADMIN-only access

#### Day 5: Navigation Tests
**Developer:** QA Engineer (Lead)  
**Tasks:**
1. Create Navigation page objects
2. Write 10+ Navigation tests
   - Main navigation menu
   - Mobile navigation drawer
   - Breadcrumbs
   - Global search
   - Notifications dropdown
   - User menu
   - Organization switcher

**Success Criteria:**
- [ ] Navigation page objects created
- [ ] 10+ Navigation tests passing
- [ ] Tests work on desktop and mobile

---

### Phase 6: Multi-Organization & Authentication Edge Cases (Week 6)

#### Day 1-3: Multi-Organization Tests
**Developer:** QA Engineer (Lead) + Backend Developer  
**Tasks:**
1. Create Organization Selector page object
2. Write 15+ Multi-org tests
   - Login with multi-org user → Org selector displayed
   - Login with single-org user → Direct to dashboard
   - Select organization from selector
   - Switch organization from dropdown
   - Verify data isolation between orgs
   - Org-specific roles (ADMIN in org1, USER in org2)

**Success Criteria:**
- [ ] Org selector page objects created
- [ ] 15+ Multi-org tests passing
- [ ] Tests verify data isolation
- [ ] Tests verify role switching across orgs

#### Day 4-5: Authentication Edge Cases
**Developer:** QA Engineer (Lead)  
**Tasks:**
1. Write 10+ Auth edge case tests
   - Session expiry and refresh
   - OAuth callbacks (Google, Microsoft)
   - Remember me functionality
   - Logout from all devices
   - Password reset flow
   - Email verification

**Success Criteria:**
- [ ] 10+ Auth edge case tests passing
- [ ] OAuth flow tests passing
- [ ] Session management tests passing

---

### Phase 7: Accessibility, Performance, and Mobile (Week 7)

#### Day 1-2: Accessibility Tests
**Developer:** QA Engineer (Lead) + Frontend Developer  
**Tasks:**
1. Integrate axe-core for automated accessibility scanning
2. Write 15+ Accessibility tests
   - Keyboard navigation (Tab, Enter, Esc, Arrow keys)
   - Screen reader compatibility (ARIA labels, roles)
   - Focus management
   - Color contrast
   - Form labels and error messages
   - Skip links

**Success Criteria:**
- [ ] axe-core integrated
- [ ] 15+ Accessibility tests passing
- [ ] Tests verify WCAG 2.1 AA compliance

#### Day 3: Performance Tests
**Developer:** QA Engineer (Lead) + DevOps Engineer  
**Tasks:**
1. Write 10+ Performance tests
   - Page load times (< 3 seconds)
   - API response times (< 500ms)
   - Bundle size monitoring
   - Image optimization
   - Lazy loading

**Success Criteria:**
- [ ] 10+ Performance tests passing
- [ ] Tests track metrics over time
- [ ] Performance thresholds enforced

#### Day 4-5: Mobile Tests
**Developer:** QA Engineer (Lead)  
**Tasks:**
1. Write 20+ Mobile tests
   - Mobile navigation drawer
   - Touch interactions
   - Responsive layouts (breakpoints)
   - Mobile forms and inputs
   - Mobile-specific features

**Success Criteria:**
- [ ] 20+ Mobile tests passing
- [ ] Tests run on iOS (iPhone 12) and Android (Pixel 5)
- [ ] Tests verify touch gestures

---

### Phase 8: CI/CD Integration and Documentation (Week 8)

#### Day 1-2: CI/CD Pipeline
**Developer:** DevOps Engineer + QA Engineer (Lead)  
**Tasks:**
1. Create GitHub Actions workflow
2. Configure test execution on PR and push
3. Set up test artifact upload (reports, screenshots, videos)
4. Configure test failure notifications

**Success Criteria:**
- [ ] GitHub Actions workflow created
- [ ] Tests run automatically on every PR
- [ ] Test reports published and accessible
- [ ] Failing tests block PR merges

#### Day 3-4: Documentation
**Developer:** QA Engineer (Lead)  
**Tasks:**
1. Write `e2e/README.md` - Setup and running instructions
2. Write `e2e/CONTRIBUTING.md` - How to add new tests
3. Write `e2e/TROUBLESHOOTING.md` - Common issues and solutions
4. Document test data management strategy
5. Create maintenance guidelines

**Success Criteria:**
- [ ] Documentation complete and reviewed
- [ ] Team trained on running/writing tests
- [ ] Maintenance schedule established

#### Day 5: Final Review & Handoff
**Developer:** QA Engineer (Lead) + Team  
**Tasks:**
1. Run full test suite and verify all tests pass
2. Review test coverage metrics
3. Demo test suite to stakeholders
4. Handoff to team with training session

**Success Criteria:**
- [ ] All 230+ tests passing
- [ ] Test coverage targets met (85%+)
- [ ] Team trained and confident
- [ ] Documentation approved

---

## � Quality Gates

### Phase 1 Completion Gate
**Required before moving to Phase 2:**
- [ ] Playwright installed and configured
- [ ] BasePage class created and tested
- [ ] Authentication fixture working
- [ ] 15+ authentication tests passing on all browsers
- [ ] Test data cleanup working
- [ ] Test reports generating correctly

### Phase 2 Completion Gate
**Required before moving to Phase 3:**
- [ ] Client and Project page objects created
- [ ] 30+ CRUD tests passing (15 clients + 15 projects)
- [ ] Tests verify data persistence
- [ ] Tests clean up all created data
- [ ] No flaky tests (< 5% failure rate)

### Phase 3 Completion Gate
**Required before moving to Phase 4:**
- [ ] Quote and Invoice page objects created
- [ ] 40+ business logic tests passing
- [ ] Tests verify quote→invoice conversion
- [ ] Tests verify payment recording
- [ ] PDF generation tests passing

### Phase 4 Completion Gate
**Required before moving to Phase 5:**
- [ ] Inventory and Report page objects created
- [ ] 30+ tests passing (15 inventory + 15 reports)
- [ ] Tests verify stock tracking
- [ ] Tests verify report calculations

### Phase 5 Completion Gate
**Required before moving to Phase 6:**
- [ ] Settings, Admin, and Navigation page objects created
- [ ] 45+ tests passing (20 settings + 15 admin + 10 nav)
- [ ] Tests verify role-based access
- [ ] Mobile navigation tests passing

### Phase 6 Completion Gate
**Required before moving to Phase 7:**
- [ ] Multi-org and auth edge case tests passing
- [ ] 25+ tests passing (15 multi-org + 10 auth edge cases)
- [ ] Tests verify data isolation
- [ ] OAuth flow tests passing

### Phase 7 Completion Gate
**Required before moving to Phase 8:**
- [ ] Accessibility, performance, and mobile tests passing
- [ ] 45+ tests passing (15 a11y + 10 performance + 20 mobile)
- [ ] WCAG compliance verified
- [ ] Performance thresholds met
- [ ] Mobile tests passing on iOS and Android

### Phase 8 Completion Gate (Final)
**Required for project completion:**
- [ ] CI/CD pipeline functional
- [ ] Tests run automatically on every PR
- [ ] Test reports published
- [ ] Documentation complete
- [ ] Team trained
- [ ] All 230+ tests passing
- [ ] Test suite completes in < 30 minutes
- [ ] < 5% flaky tests

---

## 🧪 Testing Strategy

```
apps/frontend/
├── e2e/                          # Playwright E2E tests
│   ├── fixtures/                 # Test fixtures and helpers
│   │   ├── auth.fixture.ts       # Authentication helpers
│   │   ├── test-data.ts          # Seed data for tests
│   │   └── api-helpers.ts        # API interaction helpers
│   ├── page-objects/             # Page Object Model
│   │   ├── auth/
│   │   │   ├── login.page.ts
│   │   │   ├── signup.page.ts
│   │   │   └── org-selector.page.ts
│   │   ├── clients/
│   │   │   ├── clients-list.page.ts
│   │   │   ├── client-create.page.ts
│   │   │   ├── client-detail.page.ts
│   │   │   └── client-edit.page.ts
│   │   ├── projects/
│   │   │   ├── projects-list.page.ts
│   │   │   ├── project-create.page.ts
│   │   │   ├── project-detail.page.ts
│   │   │   └── project-edit.page.ts
│   │   ├── quotes/
│   │   ├── invoices/
│   │   ├── inventory/
│   │   ├── reports/
│   │   ├── settings/
│   │   ├── admin/
│   │   └── common/
│   │       ├── navigation.page.ts
│   │       ├── search.page.ts
│   │       └── notifications.page.ts
│   ├── tests/
│   │   ├── auth/
│   │   │   ├── login.spec.ts
│   │   │   ├── signup.spec.ts
│   │   │   ├── logout.spec.ts
│   │   │   └── multi-org.spec.ts
│   │   ├── clients/
│   │   │   ├── client-crud.spec.ts
│   │   │   ├── client-search.spec.ts
│   │   │   └── client-validation.spec.ts
│   │   ├── projects/
│   │   │   ├── project-crud.spec.ts
│   │   │   ├── project-client-association.spec.ts
│   │   │   └── project-validation.spec.ts
│   │   ├── quotes/
│   │   │   ├── quote-crud.spec.ts
│   │   │   ├── quote-conversion.spec.ts
│   │   │   └── quote-pdf.spec.ts
│   │   ├── invoices/
│   │   │   ├── invoice-crud.spec.ts
│   │   │   ├── invoice-payment.spec.ts
│   │   │   └── invoice-pdf.spec.ts
│   │   ├── inventory/
│   │   │   ├── inventory-crud.spec.ts
│   │   │   ├── inventory-stock.spec.ts
│   │   │   └── inventory-categories.spec.ts
│   │   ├── reports/
│   │   │   ├── standard-reports.spec.ts
│   │   │   ├── custom-reports.spec.ts
│   │   │   └── report-export.spec.ts
│   │   ├── settings/
│   │   │   ├── user-profile.spec.ts
│   │   │   ├── security.spec.ts
│   │   │   ├── company-settings.spec.ts
│   │   │   └── billing.spec.ts
│   │   ├── admin/
│   │   │   ├── user-management.spec.ts
│   │   │   └── integrations.spec.ts
│   │   ├── navigation/
│   │   │   ├── main-navigation.spec.ts
│   │   │   ├── mobile-navigation.spec.ts
│   │   │   └── global-search.spec.ts
│   │   ├── accessibility/
│   │   │   ├── keyboard-navigation.spec.ts
│   │   │   ├── screen-reader.spec.ts
│   │   │   └── wcag-compliance.spec.ts
│   │   └── performance/
│   │       ├── page-load-times.spec.ts
│   │       └── api-response-times.spec.ts
│   ├── utils/
│   │   ├── database.ts             # Database seeding/cleanup
│   │   ├── test-users.ts           # Test user management
│   │   ├── screenshot.ts           # Screenshot helpers
│   │   └── wait-for.ts             # Custom wait utilities
│   └── playwright.config.ts        # Playwright configuration
├── playwright-report/              # Test reports (gitignored)
└── test-results/                   # Test artifacts (gitignored)
```

### Test Data Strategy

**Approach**: Isolated test data per test suite with automatic cleanup

1. **Database Seeding**:
   - Create dedicated test accounts/organizations
   - Seed with minimal required data
   - Use unique identifiers per test run

2. **Test Isolation**:
   - Each test creates its own data
   - No dependencies between tests
   - Parallel execution safe

3. **Cleanup**:
   - Automatic cleanup after each test
   - Database transaction rollback (where possible)
   - API cleanup endpoints for test data

---

**Document Owner:** Engineering Team  
**Created:** November 12, 2025  
**Last Updated:** November 12, 2025  
**Next Review:** End of Phase 1 (Week 1)  
**Status:** Ready for Implementation
