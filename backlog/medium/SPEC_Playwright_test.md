# Playwright E2E Testing Suite Implementation Plan

## 📋 Overview

Implement a comprehensive end-to-end (E2E) testing suite using Playwright to test all CRUD operations (Create, Read, Update, Delete) across all pages and functionality in the Project Ledger application.

---

## 🎯 Objectives

1. **Comprehensive Coverage**: Test all major user workflows and CRUD operations
2. **Reliability**: Ensure tests are stable, fast, and maintainable
3. **CI/CD Integration**: Enable automated testing in deployment pipelines
4. **Cross-Browser Testing**: Validate functionality across Chrome, Firefox, and WebKit
5. **Mobile Testing**: Test responsive design and mobile-specific features
6. **Performance Monitoring**: Track and validate page load times
7. **Accessibility Testing**: Validate WCAG compliance

---

## 🔍 Current State Analysis

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

---

## 🏗️ Proposed Architecture

### Test Structure

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

## 📋 Implementation Phases

### Phase 1: Foundation & Setup (Week 1)

**Objective**: Set up Playwright infrastructure and authentication tests

#### Tasks:

**1.1 Install and Configure Playwright**
```bash
npm install --save-dev @playwright/test
npx playwright install
```

**1.2 Create Playwright Configuration**
```typescript
// apps/frontend/playwright.config.ts
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

**1.3 Create Base Page Object**
```typescript
// apps/frontend/e2e/page-objects/base.page.ts
import { Page, Locator } from '@playwright/test';

export class BasePage {
  constructor(protected page: Page) {}

  async navigate(path: string) {
    await this.page.goto(path);
  }

  async waitForPageLoad() {
    await this.page.waitForLoadState('networkidle');
  }

  async screenshot(name: string) {
    await this.page.screenshot({ path: `screenshots/${name}.png`, fullPage: true });
  }

  async getElement(selector: string): Promise<Locator> {
    return this.page.locator(selector);
  }

  async clickButton(text: string) {
    await this.page.getByRole('button', { name: text }).click();
  }

  async fillInput(label: string, value: string) {
    await this.page.getByLabel(label).fill(value);
  }

  async expectVisible(selector: string) {
    await this.page.locator(selector).waitFor({ state: 'visible' });
  }
}
```

**1.4 Create Authentication Fixture**
```typescript
// apps/frontend/e2e/fixtures/auth.fixture.ts
import { test as base, expect } from '@playwright/test';
import { LoginPage } from '../page-objects/auth/login.page';

type AuthFixtures = {
  authenticatedPage: Page;
  loginAsAdmin: () => Promise<void>;
  loginAsUser: () => Promise<void>;
};

export const test = base.extend<AuthFixtures>({
  authenticatedPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await loginPage.navigate();
    await loginPage.login('test-user@example.com', 'test123');
    await use(page);
  },

  loginAsAdmin: async ({ page }, use) => {
    const login = async () => {
      const loginPage = new LoginPage(page);
      await loginPage.navigate();
      await loginPage.login('admin@projectledger.com', 'admin123');
    };
    await use(login);
  },

  loginAsUser: async ({ page }, use) => {
    const login = async () => {
      const loginPage = new LoginPage(page);
      await loginPage.navigate();
      await loginPage.login('demo@projectledger.com', 'admin123');
    };
    await use(login);
  },
});

export { expect };
```

**1.5 Create Login Page Object**
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
    await this.page.waitForURL(/\/(dashboard|select-organization)?/, { timeout: 10000 });
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

  async clickSignUp() {
    await this.page.getByRole('link', { name: /sign up/i }).click();
  }

  async clickForgotPassword() {
    await this.page.getByRole('link', { name: /forgot password/i }).click();
  }
}
```

**1.6 Write Authentication Tests**
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
    await loginPage.loginAndExpectError('invalid@example.com', 'wrongpassword');
    
    const error = await loginPage.getErrorMessage();
    expect(error).toContain('Invalid');
  });

  test('should navigate to signup page', async ({ page }) => {
    const loginPage = new LoginPage(page);
    await loginPage.navigate();
    await loginPage.clickSignUp();
    
    await expect(page).toHaveURL('/signup');
  });

  test('should show validation errors for empty fields', async ({ page }) => {
    const loginPage = new LoginPage(page);
    await loginPage.navigate();
    await loginPage.clickButton('Log In');
    
    // Should show validation errors
    await loginPage.expectVisible('text=Email is required');
    await loginPage.expectVisible('text=Password is required');
  });
});
```

**Deliverables**:
- ✅ Playwright installed and configured
- ✅ Base page object class created
- ✅ Authentication fixture created
- ✅ Login page object implemented
- ✅ 10+ authentication tests passing
- ✅ Test reports generated (HTML, JSON, JUnit)

---

### Phase 2: Core CRUD Tests - Clients & Projects (Week 2)

**Objective**: Implement comprehensive CRUD tests for Clients and Projects

#### Tasks:

**2.1 Create Client Page Objects**
```typescript
// apps/frontend/e2e/page-objects/clients/clients-list.page.ts
export class ClientsListPage extends BasePage {
  async navigate() {
    await this.page.goto('/clients');
  }

  async clickNewClient() {
    await this.clickButton('New Client');
  }

  async searchClient(name: string) {
    await this.page.getByPlaceholder('Search clients').fill(name);
    await this.page.waitForTimeout(500); // Debounce
  }

  async getClientRow(name: string) {
    return this.page.getByRole('row', { name: new RegExp(name, 'i') });
  }

  async clickClient(name: string) {
    await this.getClientRow(name).click();
  }

  async getClientCount() {
    const rows = await this.page.getByRole('row').count();
    return rows - 1; // Subtract header row
  }
}

// apps/frontend/e2e/page-objects/clients/client-create.page.ts
export class ClientCreatePage extends BasePage {
  async navigate() {
    await this.page.goto('/clients/new');
  }

  async fillClientForm(data: {
    name: string;
    email: string;
    phone?: string;
    address?: string;
    city?: string;
    province?: string;
    postalCode?: string;
    country?: string;
  }) {
    await this.fillInput('Client Name', data.name);
    await this.fillInput('Email', data.email);
    
    if (data.phone) await this.fillInput('Phone', data.phone);
    if (data.address) await this.fillInput('Address', data.address);
    if (data.city) await this.fillInput('City', data.city);
    if (data.province) await this.fillInput('Province', data.province);
    if (data.postalCode) await this.fillInput('Postal Code', data.postalCode);
    if (data.country) {
      await this.page.getByLabel('Country').click();
      await this.page.getByRole('option', { name: data.country }).click();
    }
  }

  async submitForm() {
    await this.clickButton('Save Client');
    await this.page.waitForURL(/\/clients\/\d+/);
  }

  async submitAndExpectError() {
    await this.clickButton('Save Client');
    await this.expectVisible('[role="alert"]');
  }
}

// apps/frontend/e2e/page-objects/clients/client-detail.page.ts
export class ClientDetailPage extends BasePage {
  async getClientName() {
    return await this.page.locator('h1').textContent();
  }

  async clickEdit() {
    await this.clickButton('Edit');
  }

  async clickDelete() {
    await this.clickButton('Delete');
  }

  async confirmDelete() {
    await this.page.getByRole('button', { name: /confirm|yes|delete/i }).click();
  }

  async getClientEmail() {
    return await this.page.getByText(/email/i).locator('..').textContent();
  }
}

// apps/frontend/e2e/page-objects/clients/client-edit.page.ts
export class ClientEditPage extends ClientCreatePage {
  async navigate(clientId: number) {
    await this.page.goto(`/clients/${clientId}/edit`);
  }

  async updateClientName(newName: string) {
    const nameInput = this.page.getByLabel('Client Name');
    await nameInput.clear();
    await nameInput.fill(newName);
  }

  async saveChanges() {
    await this.clickButton('Save Changes');
    await this.page.waitForURL(/\/clients\/\d+$/);
  }
}
```

**2.2 Write Client CRUD Tests**
```typescript
// apps/frontend/e2e/tests/clients/client-crud.spec.ts
import { test, expect } from '../../fixtures/auth.fixture';
import { ClientsListPage } from '../../page-objects/clients/clients-list.page';
import { ClientCreatePage } from '../../page-objects/clients/client-create.page';
import { ClientDetailPage } from '../../page-objects/clients/client-detail.page';
import { ClientEditPage } from '../../page-objects/clients/client-edit.page';

test.describe('Client CRUD Operations', () => {
  test.use({ storageState: 'auth-state.json' }); // Reuse authentication

  const testClient = {
    name: `Test Client ${Date.now()}`,
    email: `client-${Date.now()}@test.com`,
    phone: '555-1234',
    address: '123 Test St',
    city: 'Test City',
    province: 'BC',
    postalCode: 'V1V 1V1',
    country: 'Canada'
  };

  test('should create a new client', async ({ page }) => {
    const listPage = new ClientsListPage(page);
    const createPage = new ClientCreatePage(page);
    const detailPage = new ClientDetailPage(page);

    // Navigate to clients list
    await listPage.navigate();
    await listPage.clickNewClient();

    // Fill and submit form
    await createPage.fillClientForm(testClient);
    await createPage.submitForm();

    // Verify client created
    const clientName = await detailPage.getClientName();
    expect(clientName).toContain(testClient.name);
  });

  test('should view client details', async ({ page }) => {
    const listPage = new ClientsListPage(page);
    const detailPage = new ClientDetailPage(page);

    await listPage.navigate();
    await listPage.searchClient(testClient.name);
    await listPage.clickClient(testClient.name);

    // Verify details displayed
    const name = await detailPage.getClientName();
    expect(name).toContain(testClient.name);
  });

  test('should edit an existing client', async ({ page }) => {
    const listPage = new ClientsListPage(page);
    const detailPage = new ClientDetailPage(page);
    const editPage = new ClientEditPage(page);

    // Navigate to client
    await listPage.navigate();
    await listPage.searchClient(testClient.name);
    await listPage.clickClient(testClient.name);
    
    // Click edit
    await detailPage.clickEdit();

    // Update client name
    const newName = `${testClient.name} - Updated`;
    await editPage.updateClientName(newName);
    await editPage.saveChanges();

    // Verify update
    const updatedName = await detailPage.getClientName();
    expect(updatedName).toContain('Updated');
  });

  test('should delete a client', async ({ page }) => {
    const listPage = new ClientsListPage(page);
    const detailPage = new ClientDetailPage(page);

    // Navigate to client
    await listPage.navigate();
    const initialCount = await listPage.getClientCount();
    
    await listPage.searchClient(testClient.name);
    await listPage.clickClient(testClient.name);

    // Delete client
    await detailPage.clickDelete();
    await detailPage.confirmDelete();

    // Verify deleted (redirected to list)
    await expect(page).toHaveURL('/clients');
    
    // Verify count decreased
    const finalCount = await listPage.getClientCount();
    expect(finalCount).toBeLessThan(initialCount);
  });

  test('should validate required fields', async ({ page }) => {
    const createPage = new ClientCreatePage(page);

    await createPage.navigate();
    await createPage.submitAndExpectError();

    // Should show validation errors
    await expect(page.getByText(/client name is required/i)).toBeVisible();
    await expect(page.getByText(/email is required/i)).toBeVisible();
  });

  test('should search for clients', async ({ page }) => {
    const listPage = new ClientsListPage(page);

    await listPage.navigate();
    await listPage.searchClient(testClient.name);

    // Should find the client
    const row = await listPage.getClientRow(testClient.name);
    await expect(row).toBeVisible();
  });
});
```

**2.3 Create Project Page Objects** (Similar structure to Clients)

**2.4 Write Project CRUD Tests** (Similar to Client tests)

**Deliverables**:
- ✅ Client page objects (List, Create, Detail, Edit)
- ✅ 15+ Client CRUD tests passing
- ✅ Project page objects (List, Create, Detail, Edit)
- ✅ 15+ Project CRUD tests passing
- ✅ Test data cleanup utilities

---

### Phase 3: Business Logic Tests - Quotes & Invoices (Week 3)

**Objective**: Test Quotes and Invoices with complex workflows

#### Test Coverage:

**Quotes**:
- Create quote from scratch
- Create quote from template
- Add line items to quote
- Calculate totals (subtotal, tax, total)
- Convert quote to invoice
- Send quote to client
- Export quote to PDF
- Quote status lifecycle (Draft → Sent → Accepted/Rejected)

**Invoices**:
- Create invoice from scratch
- Create invoice from quote
- Add line items and calculate totals
- Apply tax rates
- Record payments
- Mark as paid
- Send invoice to client
- Export invoice to PDF
- Invoice status lifecycle (Draft → Sent → Paid → Overdue)

**3.1 Example Quote Conversion Test**
```typescript
// apps/frontend/e2e/tests/quotes/quote-conversion.spec.ts
test('should convert quote to invoice', async ({ page }) => {
  const quotesPage = new QuotesListPage(page);
  const quoteDetailPage = new QuoteDetailPage(page);
  const invoiceDetailPage = new InvoiceDetailPage(page);

  // Create a quote first
  await quotesPage.navigate();
  await quotesPage.clickNewQuote();
  // ... create quote logic

  // Convert to invoice
  await quoteDetailPage.clickConvertToInvoice();
  await quoteDetailPage.confirmConversion();

  // Verify invoice created
  await expect(page).toHaveURL(/\/invoices\/\d+/);
  
  const invoiceNumber = await invoiceDetailPage.getInvoiceNumber();
  expect(invoiceNumber).toBeTruthy();
  
  const status = await invoiceDetailPage.getStatus();
  expect(status).toBe('Draft');
});
```

**Deliverables**:
- ✅ Quote page objects (List, Create, Detail, Edit)
- ✅ 20+ Quote tests (CRUD, conversion, PDF, status)
- ✅ Invoice page objects (List, Create, Detail, Edit)
- ✅ 20+ Invoice tests (CRUD, payments, PDF, status)

---

### Phase 4: Inventory & Reports (Week 4)

**Objective**: Test Inventory management and Reporting functionality

#### Test Coverage:

**Inventory**:
- Create inventory item
- View inventory item
- Update inventory item (name, SKU, price, stock)
- Delete inventory item
- Track stock levels
- Low stock warnings
- Categories and tags

**Reports**:
- Standard reports (Invoice Aging, Revenue, Project Profitability)
- Custom report builder
- Report filters and date ranges
- Export reports (PDF, CSV, Excel)
- Save custom reports
- Report permissions (Pro plan check)

**Deliverables**:
- ✅ Inventory page objects
- ✅ 15+ Inventory CRUD tests
- ✅ Report page objects
- ✅ 15+ Report tests (standard + custom)

---

### Phase 5: Settings, Admin, and Navigation (Week 5)

**Objective**: Test Settings, Admin Panel, and Navigation features

#### Test Coverage:

**Settings**:
- User Profile (update name, email, avatar)
- Security (change password, 2FA)
- Notifications (email preferences)
- Appearance (theme, language)
- Company Settings (ADMIN only)
- Billing (subscription, payment methods)

**Admin Panel** (ADMIN role only):
- User management (create, edit, delete, roles)
- Integrations (OAuth setup)
- Organization settings

**Navigation**:
- Main navigation menu
- Mobile navigation drawer
- Breadcrumbs
- Global search
- Notifications dropdown
- User menu
- Organization switcher (multi-org)

**5.1 Example Admin Test**
```typescript
// apps/frontend/e2e/tests/admin/user-management.spec.ts
test('admin should create a new user', async ({ page, loginAsAdmin }) => {
  await loginAsAdmin();
  
  const adminPage = new AdminPanelPage(page);
  await adminPage.navigate();
  await adminPage.clickUsersTab();
  await adminPage.clickCreateUser();

  await adminPage.fillUserForm({
    email: `newuser-${Date.now()}@test.com`,
    name: 'New Test User',
    role: 'USER'
  });
  await adminPage.submitUserForm();

  // Verify user created
  const successMessage = await adminPage.getSuccessMessage();
  expect(successMessage).toContain('User created successfully');
});
```

**Deliverables**:
- ✅ Settings page objects
- ✅ 20+ Settings tests (all tabs)
- ✅ Admin page objects
- ✅ 15+ Admin tests (user management, integrations)
- ✅ Navigation page objects
- ✅ 10+ Navigation tests (menu, search, mobile)

---

### Phase 6: Multi-Organization & Authentication Edge Cases (Week 6)

**Objective**: Test multi-org features and complex auth scenarios

#### Test Coverage:

**Multi-Organization**:
- Login with multi-org user → Organization selector displayed
- Login with single-org user → No selector, direct to dashboard
- Select organization from selector
- Switch organization from navigation dropdown
- Verify data isolation between organizations
- Organization-specific roles (ADMIN in org1, USER in org2)

**Authentication Edge Cases**:
- Session expiry and refresh
- OAuth login (Google, Microsoft)
- Remember me functionality
- Logout from all devices
- Password reset flow
- Email verification

**6.1 Example Multi-Org Test**
```typescript
// apps/frontend/e2e/tests/auth/multi-org.spec.ts
test('should switch between organizations', async ({ page }) => {
  const loginPage = new LoginPage(page);
  const orgSelectorPage = new OrganizationSelectorPage(page);
  const dashboardPage = new DashboardPage(page);
  const navigation = new NavigationPage(page);

  // Login with multi-org user
  await loginPage.navigate();
  await loginPage.login('lovell@microsoft.com', 'admin123');

  // Should see organization selector
  await expect(page).toHaveURL('/select-organization');
  
  const orgs = await orgSelectorPage.getOrganizations();
  expect(orgs.length).toBe(2);

  // Select first organization
  await orgSelectorPage.selectOrganization('Microsoft');
  await expect(page).toHaveURL('/dashboard');

  // Switch to second organization
  await navigation.openOrganizationSwitcher();
  await navigation.selectOrganization('Legacy Account');

  // Verify switched (page should reload)
  await expect(page).toHaveURL('/dashboard');
  
  const currentOrg = await navigation.getCurrentOrganization();
  expect(currentOrg).toBe('Legacy Account');
});
```

**Deliverables**:
- ✅ Organization selector page objects
- ✅ 15+ Multi-org tests
- ✅ 10+ Authentication edge case tests
- ✅ OAuth callback tests

---

### Phase 7: Accessibility, Performance, and Mobile (Week 7)

**Objective**: Test accessibility, performance, and mobile responsiveness

#### Test Coverage:

**Accessibility**:
- Keyboard navigation (Tab, Enter, Esc, Arrow keys)
- Screen reader compatibility (ARIA labels, roles)
- Focus management
- Color contrast
- Form labels and error messages
- Skip links

**Performance**:
- Page load times (< 3 seconds)
- API response times (< 500ms)
- Bundle size monitoring
- Image optimization
- Lazy loading

**Mobile Testing**:
- Mobile navigation drawer
- Touch interactions
- Responsive layouts (breakpoints)
- Mobile forms and inputs
- Mobile-specific features (bottom navigation)

**7.1 Example Accessibility Test**
```typescript
// apps/frontend/e2e/tests/accessibility/keyboard-navigation.spec.ts
test('should navigate menu with keyboard', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.navigate();
  await loginPage.login('demo@projectledger.com', 'admin123');

  // Focus first menu item
  await page.keyboard.press('Tab');
  await page.keyboard.press('Tab'); // Skip to main nav

  // Navigate through menu with arrows
  await page.keyboard.press('ArrowDown');
  await page.keyboard.press('ArrowDown');
  
  // Activate with Enter
  await page.keyboard.press('Enter');

  // Should navigate to page
  await expect(page).toHaveURL(/\/projects/);
});
```

**7.2 Example Performance Test**
```typescript
// apps/frontend/e2e/tests/performance/page-load-times.spec.ts
test('dashboard should load within 3 seconds', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.navigate();
  await loginPage.login('demo@projectledger.com', 'admin123');

  const startTime = Date.now();
  await page.waitForURL('/dashboard');
  await page.waitForLoadState('networkidle');
  const loadTime = Date.now() - startTime;

  expect(loadTime).toBeLessThan(3000);
});
```

**Deliverables**:
- ✅ 15+ Accessibility tests (keyboard, ARIA, focus)
- ✅ 10+ Performance tests (load times, API)
- ✅ 20+ Mobile tests (all devices)
- ✅ Automated accessibility scanning (axe-core integration)

---

### Phase 8: CI/CD Integration and Maintenance (Week 8)

**Objective**: Integrate tests into CI/CD pipeline and establish maintenance practices

#### Tasks:

**8.1 GitHub Actions Workflow**
```yaml
# .github/workflows/e2e-tests.yml
name: E2E Tests

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - uses: actions/setup-node@v3
      with:
        node-version: 20
    
    - name: Install dependencies
      run: |
        cd apps/frontend
        npm ci
    
    - name: Install Playwright Browsers
      run: |
        cd apps/frontend
        npx playwright install --with-deps
    
    - name: Start services
      run: |
        docker compose up -d
        # Wait for services to be ready
        sleep 30
    
    - name: Run Playwright tests
      run: |
        cd apps/frontend
        npx playwright test
      env:
        PLAYWRIGHT_BASE_URL: http://localhost:3000
    
    - uses: actions/upload-artifact@v3
      if: always()
      with:
        name: playwright-report
        path: apps/frontend/playwright-report/
        retention-days: 30
    
    - uses: actions/upload-artifact@v3
      if: always()
      with:
        name: test-results
        path: apps/frontend/test-results/
        retention-days: 30
```

**8.2 Test Maintenance Guidelines**

**Best Practices**:
1. **Page Object Pattern**: All interactions through page objects
2. **DRY Principle**: Reusable fixtures and utilities
3. **Test Isolation**: Each test creates/cleans its own data
4. **Descriptive Names**: Clear test and variable names
5. **Error Handling**: Proper waits and error messages
6. **Screenshots**: Capture on failure for debugging

**Maintenance Schedule**:
- Weekly: Review failing tests, update selectors
- Monthly: Review test coverage, add missing tests
- Quarterly: Performance audit, refactor duplicated code

**8.3 Documentation**

Create comprehensive documentation:
- `e2e/README.md` - Setup and running instructions
- `e2e/CONTRIBUTING.md` - How to add new tests
- `e2e/TROUBLESHOOTING.md` - Common issues and solutions

**Deliverables**:
- ✅ GitHub Actions workflow configured
- ✅ Tests running on every PR
- ✅ Test reports published
- ✅ Documentation complete
- ✅ Maintenance guidelines established

---

## 📊 Testing Metrics and Coverage Goals

### Test Coverage Targets

| Area | Target Coverage | Tests Required |
|------|----------------|----------------|
| Authentication | 100% | 15+ tests |
| Clients CRUD | 90% | 15+ tests |
| Projects CRUD | 90% | 15+ tests |
| Quotes CRUD + Workflows | 85% | 20+ tests |
| Invoices CRUD + Workflows | 85% | 20+ tests |
| Inventory CRUD | 85% | 15+ tests |
| Reports | 80% | 15+ tests |
| Settings | 75% | 20+ tests |
| Admin Panel | 80% | 15+ tests |
| Navigation | 85% | 10+ tests |
| Multi-Org | 90% | 15+ tests |
| Accessibility | 70% | 15+ tests |
| Performance | 100% | 10+ tests |
| Mobile | 80% | 20+ tests |

**Total Tests**: ~230+ E2E tests

### Success Criteria

**Test Execution**:
- ✅ All tests pass on 3 browsers (Chrome, Firefox, Safari)
- ✅ All tests pass on 2 mobile devices (iOS, Android)
- ✅ Test suite completes in < 30 minutes
- ✅ < 5% flaky tests (tests that fail intermittently)

**Code Quality**:
- ✅ All page objects follow consistent pattern
- ✅ No duplicate test code (DRY principle)
- ✅ Test data properly isolated and cleaned up
- ✅ Clear, descriptive test names

**CI/CD Integration**:
- ✅ Tests run automatically on every PR
- ✅ Test reports published and accessible
- ✅ Failing tests block PR merges
- ✅ Screenshot/video artifacts saved on failure

---

## 🛠️ Technology and Tools

### Core Testing Framework

- **Playwright 1.40+**: E2E testing framework
- **TypeScript**: Type-safe test code
- **Page Object Model**: Maintainable test structure
- **Test Fixtures**: Reusable test setup

### Additional Tools

- **axe-core**: Automated accessibility testing
- **Percy** (optional): Visual regression testing
- **Lighthouse**: Performance auditing
- **Artillery** (optional): Load testing

### Reporting

- **HTML Reporter**: Interactive test results
- **JUnit Reporter**: CI/CD integration
- **JSON Reporter**: Custom analytics
- **Allure** (optional): Advanced reporting

---

## 📋 Implementation Checklist

### Pre-Implementation
- [ ] Review and approve this specification
- [ ] Assign team members to phases
- [ ] Set up test environment (dedicated test database)
- [ ] Create test user accounts
- [ ] Document test data requirements

### Phase 1: Foundation (Week 1)
- [ ] Install Playwright and dependencies
- [ ] Configure Playwright (config file)
- [ ] Create base page object class
- [ ] Create authentication fixture
- [ ] Implement login page object
- [ ] Write 10+ authentication tests
- [ ] Verify tests pass on all browsers

### Phase 2: Clients & Projects (Week 2)
- [ ] Create client page objects (List, Create, Detail, Edit)
- [ ] Write 15+ client CRUD tests
- [ ] Create project page objects
- [ ] Write 15+ project CRUD tests
- [ ] Implement test data cleanup

### Phase 3: Quotes & Invoices (Week 3)
- [ ] Create quote page objects
- [ ] Write 20+ quote tests (CRUD, conversion, PDF)
- [ ] Create invoice page objects
- [ ] Write 20+ invoice tests (CRUD, payments, PDF)

### Phase 4: Inventory & Reports (Week 4)
- [ ] Create inventory page objects
- [ ] Write 15+ inventory tests
- [ ] Create report page objects
- [ ] Write 15+ report tests

### Phase 5: Settings & Admin (Week 5)
- [ ] Create settings page objects
- [ ] Write 20+ settings tests
- [ ] Create admin panel page objects
- [ ] Write 15+ admin tests
- [ ] Create navigation page objects
- [ ] Write 10+ navigation tests

### Phase 6: Multi-Org & Auth Edge Cases (Week 6)
- [ ] Create org selector page objects
- [ ] Write 15+ multi-org tests
- [ ] Write 10+ auth edge case tests
- [ ] Test OAuth flows

### Phase 7: Accessibility & Performance (Week 7)
- [ ] Write 15+ accessibility tests
- [ ] Write 10+ performance tests
- [ ] Write 20+ mobile tests
- [ ] Integrate axe-core scanning

### Phase 8: CI/CD Integration (Week 8)
- [ ] Create GitHub Actions workflow
- [ ] Configure test artifacts upload
- [ ] Write documentation (README, CONTRIBUTING, TROUBLESHOOTING)
- [ ] Establish maintenance guidelines
- [ ] Train team on test maintenance

---

## 🚨 Risks and Mitigation

### Risk 1: Flaky Tests
**Risk**: Tests fail intermittently due to timing issues, network conditions, or UI animations

**Mitigation**:
- Use Playwright's auto-waiting features
- Implement proper wait strategies (waitForSelector, waitForLoadState)
- Avoid hardcoded timeouts
- Retry failed tests automatically (2-3 retries)
- Use stable selectors (data-testid, ARIA roles)

### Risk 2: Test Data Pollution
**Risk**: Tests interfere with each other, causing failures

**Mitigation**:
- Isolated test data per test
- Automatic cleanup after each test
- Use unique identifiers (timestamps, UUIDs)
- Separate test database
- Database transaction rollback where possible

### Risk 3: Long Test Execution Time
**Risk**: Test suite takes too long to run, slowing down development

**Mitigation**:
- Run tests in parallel (Playwright workers)
- Use test tags to run subsets (smoke tests, full suite)
- Optimize test setup (reuse authentication)
- Use fast selectors
- Target: < 30 minutes for full suite

### Risk 4: Maintenance Burden
**Risk**: Tests require constant updates as UI changes

**Mitigation**:
- Page Object Model (centralize selectors)
- Use semantic selectors (ARIA roles, labels)
- Avoid brittle selectors (CSS classes, IDs)
- Regular refactoring and cleanup
- Team training on best practices

### Risk 5: CI/CD Integration Issues
**Risk**: Tests pass locally but fail in CI/CD

**Mitigation**:
- Use consistent environments (Docker)
- Lock dependency versions
- Use headless browsers in CI
- Save artifacts (screenshots, videos) on failure
- Monitor CI/CD performance

---

## 💰 Resource Requirements

### Personnel

- **QA Engineer (Lead)**: 1 full-time (8 weeks)
- **Frontend Developer**: 0.5 time (assist with page objects)
- **Backend Developer**: 0.25 time (test data setup)
- **DevOps Engineer**: 0.25 time (CI/CD integration)

### Infrastructure

- **Test Environment**: Dedicated staging server
- **Test Database**: PostgreSQL instance
- **CI/CD Minutes**: ~1000 minutes/month (GitHub Actions)
- **Storage**: Test artifacts (screenshots, videos) ~10GB/month

### Tools (Estimated Annual Cost)

- Playwright: **Free** (open source)
- Percy (optional): ~$300/month for visual testing
- Allure (optional): **Free** (self-hosted) or ~$50/month (cloud)
- GitHub Actions: **Free** (public repos) or included in plan

**Total Estimated Cost**: $300-600/month for optional premium tools

---

## 📈 Timeline Summary

| Phase | Duration | Tests | Status |
|-------|----------|-------|--------|
| Phase 1: Foundation & Auth | Week 1 | 10+ | 🟡 Planned |
| Phase 2: Clients & Projects | Week 2 | 30+ | 🟡 Planned |
| Phase 3: Quotes & Invoices | Week 3 | 40+ | 🟡 Planned |
| Phase 4: Inventory & Reports | Week 4 | 30+ | 🟡 Planned |
| Phase 5: Settings & Admin | Week 5 | 45+ | 🟡 Planned |
| Phase 6: Multi-Org & Auth | Week 6 | 25+ | 🟡 Planned |
| Phase 7: A11y & Performance | Week 7 | 45+ | 🟡 Planned |
| Phase 8: CI/CD & Docs | Week 8 | 5+ | 🟡 Planned |

**Total Duration**: 8 weeks (2 months)  
**Total Tests**: 230+ E2E tests

---

## 🎯 Next Steps

1. **Review and Approval**: Team reviews this specification
2. **Resource Allocation**: Assign team members to phases
3. **Environment Setup**: Create test environment and database
4. **Kickoff Meeting**: Align on goals, timeline, and responsibilities
5. **Phase 1 Start**: Begin with Foundation & Authentication tests

---

**Status**: 🟡 Awaiting Review & Approval  
**Next Steps**: Review specification → Approve → Begin Phase 1

**Document Version**: 1.0  
**Last Updated**: October 7, 2025  
**Author**: Development Team 