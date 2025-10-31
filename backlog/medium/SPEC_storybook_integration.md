# Storybook Integration Specification

## Overview

This specification outlines the implementation plan for integrating Storybook into the ProjectLedger frontend application to create a comprehensive component library and design system documentation.

**Status**: Planning  
**Priority**: Medium  
**Effort Estimate**: 3-5 days for initial setup + ongoing story creation  
**Owner**: Frontend Team  
**Date**: October 31, 2025  
**Implementation Plan**: [IMPLEMENTATION_storybook_integration.md](./IMPLEMENTATION_storybook_integration.md)

---

## 📊 Implementation Progress Summary

### Overall Status: ⬜ Not Started (0% Complete)

| Phase | Status | Progress | Deliverables | Notes |
|-------|--------|----------|--------------|-------|
| Phase 1: Initial Setup | ⬜ Not Started | 0% | Storybook installation, theme config | Est. 0.5 days |
| Phase 2: Core Components | ⬜ Not Started | 0% | 20-25 component stories | Est. 2 days |
| Phase 3: Feature Components | ⬜ Not Started | 0% | 15-20 feature stories | Est. 2 days |
| Phase 4: Documentation & Deploy | ⬜ Not Started | 0% | Docs, deployment, training | Est. 0.5 days |

**Legend**: ⬜ Not Started | 🟡 In Progress | ✅ Complete | ⛔ Blocked

### Quick Stats

- **Total Components**: ~45 stories planned
- **Stories Created**: 0
- **Coverage**: 0%
- **Deployment Status**: Not deployed
- **Team Training**: Not started

### Recent Updates

*No updates yet - implementation not started*

### Next Actions

1. Review specification with frontend team
2. Allocate 5-day implementation window
3. Run `npx storybook@latest init` in `apps/frontend/`
4. Follow [Implementation Plan](./IMPLEMENTATION_storybook_integration.md)

---

## Table of Contents

1. [Goals & Objectives](#goals--objectives)
2. [Current State Analysis](#current-state-analysis)
3. [Technical Architecture](#technical-architecture)
4. [Implementation Phases](#implementation-phases)
5. [Component Story Guidelines](#component-story-guidelines)
6. [Configuration Details](#configuration-details)
7. [Integration with Existing Systems](#integration-with-existing-systems)
8. [Testing Strategy](#testing-strategy)
9. [Documentation Standards](#documentation-standards)
10. [Deployment & Hosting](#deployment--hosting)
11. [Maintenance Plan](#maintenance-plan)
12. [Success Metrics](#success-metrics)
13. [Implementation Guide](#implementation-guide)

---

## Goals & Objectives

### Primary Goals

1. **Component Documentation**: Create a living documentation system for all UI components
2. **Design System Reference**: Establish single source of truth for design patterns and usage
3. **Developer Experience**: Improve development workflow with isolated component testing
4. **Quality Assurance**: Enable visual regression testing and accessibility auditing
5. **Stakeholder Communication**: Provide interactive component demos for non-technical stakeholders

### Secondary Goals

1. Enable faster onboarding for new developers
2. Facilitate design-development collaboration
3. Support component-driven development practices
4. Create reusable component examples and patterns
5. Document component props, variants, and edge cases

---

## Current State Analysis

### Existing Assets

✅ **Well-Structured Components**
- 20+ component categories in `apps/frontend/src/components/`
- Clean component hierarchy with logical grouping
- Already exported through index files

✅ **Design System Foundation**
- Custom branded theme (`brandedTheme.ts`)
- Design tokens (`tokens.ts`)
- Light/dark theme support
- Comprehensive typography and spacing system

✅ **Material-UI Integration**
- MUI v7.3.1 (latest version with excellent Storybook support)
- Emotion styling engine
- Custom component wrappers

✅ **TypeScript Configuration**
- Full TypeScript coverage
- Type-safe component props
- Automatic prop documentation capability

✅ **Accessibility Focus**
- WCAG-compliant components
- `jsx-a11y` ESLint plugin
- Accessible form components

### Preparatory Work Detected

```
.gitignore includes:
- storybook-static
- .storybook-out

jest.config.js excludes:
- *.stories.{ts,tsx}

Components exist:
- StorybookIntegration.tsx (UI mockup)
```

**Analysis**: Infrastructure preparation suggests Storybook was previously planned but not implemented.

### Missing Elements

❌ Storybook packages not installed  
❌ No `.storybook/` configuration directory  
❌ No story files (`.stories.tsx`)  
❌ No Storybook scripts in package.json  

---

## Technical Architecture

### Technology Stack

```json
{
  "storybook": "^8.3.0",
  "dependencies": [
    "@storybook/react",
    "@storybook/react-webpack5",
    "@storybook/addon-essentials",
    "@storybook/addon-interactions",
    "@storybook/addon-a11y",
    "@storybook/addon-links",
    "@storybook/addon-themes",
    "@storybook/addon-viewport",
    "@storybook/testing-library",
    "@storybook/test"
  ]
}
```

### Directory Structure

```
apps/frontend/
├── .storybook/
│   ├── main.ts                 # Storybook configuration
│   ├── preview.tsx             # Global decorators and parameters
│   ├── manager.ts              # UI customization
│   └── theme.ts                # Storybook theme customization
├── src/
│   ├── components/
│   │   ├── common/
│   │   │   ├── Button.tsx
│   │   │   └── Button.stories.tsx
│   │   ├── forms/
│   │   │   ├── FormField.tsx
│   │   │   └── FormField.stories.tsx
│   │   └── [other components]/
│   ├── theme/
│   │   ├── brandedTheme.ts
│   │   └── storybook.ts       # Storybook-specific theme config
│   └── .storybook-helpers/
│       ├── decorators.tsx      # Reusable decorators
│       ├── mockData.ts         # Mock data for stories
│       └── RouterDecorator.tsx # React Router wrapper
└── storybook-static/           # Built Storybook (gitignored)
```

---

## Implementation Phases

### Phase 1: Initial Setup (Day 1) 🚀

**Objective**: Get Storybook running with basic configuration

#### Tasks

1. **Install Storybook**
   ```bash
   cd apps/frontend
   npx storybook@latest init
   ```

2. **Update package.json scripts**
   ```json
   {
     "scripts": {
       "storybook": "storybook dev -p 6006",
       "storybook:build": "storybook build",
       "storybook:test": "test-storybook"
     }
   }
   ```

3. **Configure main.ts**
   - Set up TypeScript support
   - Configure Emotion/MUI webpack config
   - Enable essential addons

4. **Configure preview.tsx**
   - Integrate branded theme
   - Add global decorators
   - Set up viewport configurations

5. **Test Installation**
   - Run `npm run storybook`
   - Verify localhost:6006 loads
   - Ensure theme applies correctly

**Deliverables**:
- Storybook running locally
- Basic configuration files
- Theme integration working

**Time**: 2-4 hours

---

### Phase 2: Core Components (Days 2-3) ⭐

**Objective**: Create stories for essential, high-traffic components

#### Priority 1 Components (Day 2)

**Common Components** (5-6 hours)
- [ ] `Button.stories.tsx` - All variants, sizes, states
- [ ] `LoadingSpinner.stories.tsx` - Different sizes and contexts
- [ ] `Badge.stories.tsx` / `StatusBadge.stories.tsx` - All status types
- [ ] `DataTable.stories.tsx` - With mock data, pagination
- [ ] `ErrorBoundary.stories.tsx` - Error state demonstrations

**Form Components** (3-4 hours)
- [ ] `FormField.stories.tsx` - Text, password, validation states
- [ ] `FormSelect.stories.tsx` - Single/multi-select
- [ ] `DatePicker.stories.tsx` - Different date formats
- [ ] `AutoComplete.stories.tsx` - Async loading example
- [ ] `FileUpload.stories.tsx` - Upload states

#### Priority 2 Components (Day 3)

**Layout Components** (2-3 hours)
- [ ] `MainLayout.stories.tsx` - With/without sidebar
- [ ] `PortalMainLayout.stories.tsx` - Portal variant
- [ ] `Card` variations - Different elevations

**Navigation Components** (2-3 hours)
- [ ] `Sidebar.stories.tsx` - Expanded/collapsed states
- [ ] `OrganizationSwitcher.stories.tsx` - Multiple orgs
- [ ] `UserMenu.stories.tsx` - Different user states

**Charts** (2-3 hours)
- [ ] `LineChart.stories.tsx` - Revenue data example
- [ ] `BarChart.stories.tsx` - Comparison data
- [ ] `PieChart.stories.tsx` - Distribution data
- [ ] `MetricCard.stories.tsx` - Dashboard metrics

**Deliverables**:
- 20-25 story files covering core components
- Mock data utilities created
- Component variations documented

**Time**: 12-16 hours (2 working days)

---

### Phase 3: Feature Components (Days 4-5) 📦

**Objective**: Document domain-specific components

#### Feature Modules

**Invoice Components** (3-4 hours)
- [ ] `InvoiceForm.stories.tsx`
- [ ] `InvoiceList.stories.tsx`
- [ ] `InvoiceTemplateManager.stories.tsx`
- [ ] `PaymentReceipt.stories.tsx`

**Client Management** (2-3 hours)
- [ ] `ClientForm.stories.tsx`
- [ ] `ClientList.stories.tsx`
- [ ] `ClientSelector.stories.tsx`

**Project Components** (2-3 hours)
- [ ] `ProjectForm.stories.tsx`
- [ ] `ProjectList.stories.tsx`
- [ ] `ProjectTimeline.stories.tsx`

**Quote Components** (2-3 hours)
- [ ] `QuoteForm.stories.tsx`
- [ ] `QuoteList.stories.tsx`
- [ ] `QuotePreview.stories.tsx`

**Authentication** (1-2 hours)
- [ ] `LoginForm.stories.tsx`
- [ ] `RegisterForm.stories.tsx`
- [ ] `PasswordReset.stories.tsx`

**Deliverables**:
- 15-20 feature component stories
- Domain-specific mock data
- Complex interaction examples

**Time**: 10-15 hours (2 working days)

---

### Phase 4: Advanced Features (Ongoing) 🎨

**Objective**: Polish and enhance Storybook experience

#### Enhancement Tasks

1. **Interaction Testing** (2-3 hours)
   - Add `play` functions for user interactions
   - Test form validation flows
   - Test modal open/close behaviors

2. **Accessibility Testing** (2-3 hours)
   - Configure a11y addon for all stories
   - Document ARIA patterns
   - Add keyboard navigation examples

3. **Responsive Testing** (1-2 hours)
   - Configure viewport addon
   - Add mobile/tablet/desktop viewports
   - Document responsive behaviors

4. **Documentation Pages** (3-4 hours)
   - Create introduction page
   - Document design principles
   - Add usage guidelines
   - Create contribution guide

5. **Theme Switcher** (1-2 hours)
   - Add light/dark mode toggle
   - Add branded theme variants
   - Document theming approach

**Deliverables**:
- Interactive component tests
- Accessibility audit reports
- Complete design system documentation

**Time**: 10-15 hours (ongoing)

---

## Component Story Guidelines

### Story File Template

```tsx
// Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';
import { fn } from '@storybook/test';

const meta = {
  title: 'Common/Button',
  component: Button,
  parameters: {
    layout: 'centered',
    docs: {
      description: {
        component: 'Custom Button component wrapping MUI Button with loading states and icons.',
      },
    },
  },
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['contained', 'outlined', 'text'],
    },
    size: {
      control: 'select',
      options: ['small', 'medium', 'large'],
    },
    color: {
      control: 'select',
      options: ['primary', 'secondary', 'error', 'warning', 'info', 'success'],
    },
  },
  args: {
    onClick: fn(),
  },
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

// Default story
export const Primary: Story = {
  args: {
    children: 'Primary Button',
    variant: 'contained',
    color: 'primary',
  },
};

// Loading state
export const Loading: Story = {
  args: {
    children: 'Loading...',
    loading: true,
  },
};

// With icon
export const WithIcon: Story = {
  args: {
    children: 'Save',
    icon: <SaveIcon />,
  },
};

// Disabled
export const Disabled: Story = {
  args: {
    children: 'Disabled',
    disabled: true,
  },
};

// All sizes
export const Sizes: Story = {
  render: () => (
    <Box sx={{ display: 'flex', gap: 2, alignItems: 'center' }}>
      <Button size="small">Small</Button>
      <Button size="medium">Medium</Button>
      <Button size="large">Large</Button>
    </Box>
  ),
};
```

### Best Practices

1. **Naming Conventions**
   - File: `ComponentName.stories.tsx`
   - Meta title: `Category/ComponentName`
   - Story names: Descriptive PascalCase

2. **Story Organization**
   - Default/Primary variant first
   - Common use cases next
   - Edge cases and states last
   - Composite stories showing multiple variants

3. **Documentation**
   - Add component descriptions
   - Document each prop
   - Include usage examples
   - Note accessibility considerations

4. **Mock Data**
   - Create reusable mock data in `.storybook-helpers/mockData.ts`
   - Use realistic data that reflects actual use cases
   - Provide variety for different scenarios

5. **Args vs. Render**
   - Use `args` for simple prop variations
   - Use `render` for complex layouts or multiple components

---

## Configuration Details

### .storybook/main.ts

```typescript
import type { StorybookConfig } from '@storybook/react-webpack5';
import path from 'path';

const config: StorybookConfig = {
  stories: [
    '../src/**/*.mdx',
    '../src/**/*.stories.@(js|jsx|mjs|ts|tsx)',
  ],
  
  addons: [
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
    '@storybook/addon-links',
    '@storybook/addon-a11y',
    '@storybook/addon-themes',
    '@storybook/addon-viewport',
    {
      name: '@storybook/addon-styling-webpack',
      options: {
        rules: [
          {
            test: /\.css$/,
            use: [
              'style-loader',
              'css-loader',
            ],
          },
        ],
      },
    },
  ],
  
  framework: {
    name: '@storybook/react-webpack5',
    options: {
      builder: {
        useSWC: true,
      },
    },
  },
  
  typescript: {
    check: true,
    reactDocgen: 'react-docgen-typescript',
    reactDocgenTypescriptOptions: {
      shouldExtractLiteralValuesFromEnum: true,
      propFilter: (prop) => {
        if (prop.parent) {
          return !prop.parent.fileName.includes('node_modules');
        }
        return true;
      },
    },
  },
  
  webpackFinal: async (config) => {
    // Resolve aliases
    config.resolve = config.resolve || {};
    config.resolve.alias = {
      ...config.resolve.alias,
      '@': path.resolve(__dirname, '../src'),
    };
    
    return config;
  },
  
  staticDirs: ['../public'],
  
  docs: {
    autodocs: 'tag',
  },
};

export default config;
```

### .storybook/preview.tsx

```tsx
import React from 'react';
import type { Preview } from '@storybook/react';
import { ThemeProvider } from '@mui/material/styles';
import { CssBaseline, Box } from '@mui/material';
import { withThemeFromJSXProvider } from '@storybook/addon-themes';
import { MemoryRouter } from 'react-router-dom';

import { brandedTheme } from '../src/theme/brandedTheme';
import { createLightTheme, createDarkTheme } from '../src/theme/themes';

// Global decorator for theme
const withMuiTheme = withThemeFromJSXProvider({
  themes: {
    light: createLightTheme(),
    dark: createDarkTheme(),
    branded: brandedTheme,
  },
  defaultTheme: 'branded',
  Provider: ThemeProvider,
  GlobalStyles: CssBaseline,
});

// Decorator for React Router
const withRouter = (Story) => (
  <MemoryRouter>
    <Story />
  </MemoryRouter>
);

// Decorator for consistent padding
const withPadding = (Story, context) => {
  const needsPadding = context.parameters.layout !== 'fullscreen';
  
  return (
    <Box sx={{ p: needsPadding ? 3 : 0 }}>
      <Story />
    </Box>
  );
};

const preview: Preview = {
  parameters: {
    actions: { argTypesRegex: '^on[A-Z].*' },
    controls: {
      matchers: {
        color: /(background|color)$/i,
        date: /Date$/i,
      },
      expanded: true,
    },
    backgrounds: {
      default: 'light',
      values: [
        {
          name: 'light',
          value: '#f5f5f5',
        },
        {
          name: 'dark',
          value: '#1e1e1e',
        },
        {
          name: 'white',
          value: '#ffffff',
        },
      ],
    },
    viewport: {
      viewports: {
        mobile: {
          name: 'Mobile',
          styles: { width: '375px', height: '667px' },
        },
        tablet: {
          name: 'Tablet',
          styles: { width: '768px', height: '1024px' },
        },
        laptop: {
          name: 'Laptop',
          styles: { width: '1366px', height: '768px' },
        },
        desktop: {
          name: 'Desktop',
          styles: { width: '1920px', height: '1080px' },
        },
      },
    },
    a11y: {
      config: {
        rules: [
          {
            id: 'color-contrast',
            enabled: true,
          },
          {
            id: 'label',
            enabled: true,
          },
        ],
      },
    },
  },
  
  decorators: [
    withMuiTheme,
    withRouter,
    withPadding,
  ],
  
  tags: ['autodocs'],
};

export default preview;
```

### .storybook/manager.ts

```typescript
import { addons } from '@storybook/manager-api';
import { create } from '@storybook/theming';

const theme = create({
  base: 'light',
  brandTitle: 'ProjectLedger Design System',
  brandUrl: 'https://projectledger.com',
  brandTarget: '_self',
  
  colorPrimary: '#2563eb',
  colorSecondary: '#7c3aed',
  
  appBg: '#f8fafc',
  appContentBg: '#ffffff',
  appBorderColor: '#e2e8f0',
  appBorderRadius: 8,
  
  fontBase: '"Inter", "Roboto", "Helvetica", "Arial", sans-serif',
  fontCode: 'monospace',
  
  textColor: '#1e293b',
  textInverseColor: '#ffffff',
  
  barTextColor: '#64748b',
  barSelectedColor: '#2563eb',
  barBg: '#ffffff',
  
  inputBg: '#ffffff',
  inputBorder: '#e2e8f0',
  inputTextColor: '#1e293b',
  inputBorderRadius: 8,
});

addons.setConfig({
  theme,
  panelPosition: 'right',
  showPanel: true,
});
```

---

## Integration with Existing Systems

### Theme Integration

**Challenge**: Storybook needs to use the same theme system as the main application.

**Solution**:
```tsx
// .storybook/preview.tsx
import { brandedTheme } from '../src/theme/brandedTheme';
import { createLightTheme, createDarkTheme } from '../src/theme/themes';

// Use addon-themes to provide theme switching
```

**Benefits**:
- Components render identically in Storybook and app
- Theme changes are automatically reflected
- Design tokens are accessible

### React Router Integration

**Challenge**: Components using `useNavigate`, `Link`, or `useParams` will fail.

**Solution**:
```tsx
// .storybook-helpers/decorators.tsx
import { MemoryRouter } from 'react-router-dom';

export const RouterDecorator = (Story) => (
  <MemoryRouter initialEntries={['/']}>
    <Story />
  </MemoryRouter>
);
```

### React Query Integration

**Challenge**: Components using `useQuery` or `useMutation` need QueryClient.

**Solution**:
```tsx
// .storybook-helpers/decorators.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: { retry: false, staleTime: Infinity },
  },
});

export const QueryDecorator = (Story) => (
  <QueryClientProvider client={queryClient}>
    <Story />
  </QueryClientProvider>
);
```

### Context Providers

**Challenge**: Components depend on AuthContext, OrganizationContext, etc.

**Solution**:
```tsx
// .storybook-helpers/decorators.tsx
export const MockAuthDecorator = (Story) => (
  <AuthProvider value={mockAuthValue}>
    <OrganizationProvider value={mockOrgValue}>
      <Story />
    </OrganizationProvider>
  </AuthProvider>
);
```

### Mock Data

**Centralized Mock Data**:
```typescript
// .storybook-helpers/mockData.ts
export const mockUser = {
  id: '1',
  email: 'demo@projectledger.com',
  firstName: 'John',
  lastName: 'Doe',
  role: 'ADMIN',
};

export const mockClient = {
  id: '1',
  name: 'Acme Corporation',
  email: 'contact@acme.com',
  // ... other fields
};

export const mockInvoice = {
  id: '1',
  invoiceNumber: 'INV-2025-001',
  client: mockClient,
  // ... other fields
};
```

---

## Testing Strategy

### Visual Regression Testing

**Tool**: Chromatic (recommended) or Percy

```bash
# Install Chromatic
npm install --save-dev chromatic

# Add script to package.json
"chromatic": "chromatic --project-token=<token>"
```

**Benefits**:
- Automatic visual diff detection
- Catch unintended UI changes
- Review UI changes in PRs

### Interaction Testing

```tsx
// Button.stories.tsx
import { userEvent, within } from '@storybook/testing-library';
import { expect } from '@storybook/test';

export const Clickable: Story = {
  play: async ({ canvasElement, args }) => {
    const canvas = within(canvasElement);
    const button = canvas.getByRole('button');
    
    await userEvent.click(button);
    await expect(args.onClick).toHaveBeenCalled();
  },
};
```

### Accessibility Testing

**Automatic with a11y addon**:
- WCAG compliance checks
- Color contrast validation
- ARIA attribute verification
- Keyboard navigation testing

**Manual checks**:
- Screen reader testing
- Keyboard-only navigation
- Focus management

### Test Commands

```json
{
  "scripts": {
    "test:storybook": "test-storybook",
    "test:storybook:watch": "test-storybook --watch",
    "test:chromatic": "chromatic --exit-zero-on-changes"
  }
}
```

---

## Documentation Standards

### Component Documentation Template

Each component story should include:

1. **Description**: What the component does
2. **Usage**: When and where to use it
3. **Props**: All available props with descriptions
4. **Examples**: Common use cases
5. **Accessibility**: ARIA requirements and keyboard support
6. **Variants**: All visual variations
7. **States**: Loading, error, empty, disabled, etc.

### MDX Documentation Pages

```mdx
// Introduction.mdx
import { Meta } from '@storybook/blocks';

<Meta title="Introduction" />

# ProjectLedger Design System

Welcome to the ProjectLedger component library and design system documentation.

## Getting Started

This Storybook contains all UI components used in the ProjectLedger application...

## Design Principles

1. **Consistency**: Unified visual language
2. **Accessibility**: WCAG AA compliance
3. **Responsiveness**: Mobile-first approach
4. **Performance**: Optimized components

## Using This Documentation

- Browse components in the sidebar
- Try different props in the Controls panel
- View accessibility reports in the A11y panel
- Test responsive layouts in the Viewport toolbar
```

### Categories

Organize stories into logical categories:

```
├── Introduction
├── Design Tokens
│   ├── Colors
│   ├── Typography
│   └── Spacing
├── Common
│   ├── Button
│   ├── Badge
│   └── LoadingSpinner
├── Forms
│   ├── FormField
│   ├── FormSelect
│   └── DatePicker
├── Layout
│   ├── MainLayout
│   ├── Card
│   └── Grid
├── Navigation
│   ├── Sidebar
│   └── UserMenu
├── Charts
│   ├── LineChart
│   ├── BarChart
│   └── PieChart
├── Domain
│   ├── Invoices
│   ├── Clients
│   └── Projects
└── Patterns
    ├── Form Patterns
    ├── Data Display
    └── User Feedback
```

---

## Deployment & Hosting

### Static Build

```bash
# Build Storybook for production
npm run storybook:build

# Output directory: storybook-static/
```

### Hosting Options

#### Option 1: Chromatic (Recommended)

**Pros**:
- Automatic deployment on push
- Built-in visual regression testing
- Free for open source
- Shareable URLs

**Setup**:
```bash
npm install --save-dev chromatic
npx chromatic --project-token=<token>
```

#### Option 2: Azure Static Web Apps

**Pros**:
- Already using Azure infrastructure
- Custom domain support
- Free tier available

**Setup**:
```yaml
# .github/workflows/storybook-deploy.yml
name: Deploy Storybook

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run storybook:build
      - uses: azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          action: "upload"
          app_location: "apps/frontend/storybook-static"
```

#### Option 3: GitHub Pages

**Pros**:
- Free
- Simple setup
- Good for public projects

**Setup**:
```bash
npm install --save-dev @storybook/storybook-deployer

# Add to package.json
"deploy-storybook": "storybook-deployer"
```

### CI/CD Integration

```yaml
# .github/workflows/storybook-ci.yml
name: Storybook CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run storybook:build
      - run: npm run test:storybook
```

---

## Maintenance Plan

### Regular Tasks

#### Weekly
- [ ] Review new components added to codebase
- [ ] Create stories for new components
- [ ] Update existing stories if props changed

#### Monthly
- [ ] Audit story coverage (aim for 80%+)
- [ ] Review and update documentation
- [ ] Check for Storybook updates
- [ ] Review accessibility audit results

#### Quarterly
- [ ] Major Storybook version upgrades
- [ ] Comprehensive accessibility audit
- [ ] Performance optimization review
- [ ] Gather developer feedback

### Governance

**Story Review Process**:
1. Developer creates component story
2. PR includes story file and updated docs
3. Reviewer checks:
   - All variants covered
   - Props documented
   - Accessibility notes added
   - Examples are clear
4. Merge triggers Storybook deployment

**Component Definition of Done**:
- [ ] Component implemented
- [ ] TypeScript types defined
- [ ] Unit tests written
- [ ] **Story file created** ⭐
- [ ] Accessibility verified
- [ ] Responsive behavior tested
- [ ] Documentation complete

### Metrics to Track

1. **Story Coverage**: % of components with stories
2. **Documentation Completeness**: % of stories with full docs
3. **Accessibility Score**: Average a11y addon score
4. **Usage**: Storybook page views (if tracking enabled)
5. **Developer Satisfaction**: Survey feedback

---

## Success Metrics

### Quantitative Metrics

| Metric | Target | Timeline |
|--------|--------|----------|
| Core components documented | 100% | End of Week 1 |
| Total component coverage | 80% | End of Month 1 |
| Accessibility compliance | 95% WCAG AA | Ongoing |
| Story build time | < 2 minutes | Optimized |
| Developer adoption | 100% team | End of Month 2 |

### Qualitative Metrics

- [ ] Developers prefer Storybook for component development
- [ ] Designers can review components independently
- [ ] Onboarding time reduced for new developers
- [ ] Component reusability increased
- [ ] Fewer UI bugs in production

### ROI Indicators

**Time Savings**:
- Faster component development (no app navigation)
- Reduced debugging time (isolated testing)
- Quicker design reviews (shared URLs)

**Quality Improvements**:
- Earlier bug detection
- Better accessibility compliance
- More consistent UI patterns

**Collaboration Benefits**:
- Better design-dev handoff
- Clearer component requirements
- Living documentation

---

## Migration Path

### For Existing Components

1. **Assess Priority**: Use component usage analytics
2. **Create Story**: Follow template above
3. **Add Tests**: Include interaction tests
4. **Document**: Add MDX documentation
5. **Review**: Team review for completeness

### For New Components

1. **Create Component**: TDD approach with story first
2. **Develop in Storybook**: Build in isolation
3. **Test in Storybook**: Interaction and a11y tests
4. **Integrate**: Add to main application
5. **Maintain**: Update story with changes

---

## Common Challenges & Solutions

### Challenge: Component Depends on API Data

**Solution**: Use MSW (Mock Service Worker) addon
```bash
npm install --save-dev msw msw-storybook-addon
```

### Challenge: Component Uses Environment Variables

**Solution**: Configure webpack in main.ts
```typescript
webpackFinal: async (config) => {
  config.plugins.push(
    new webpack.DefinePlugin({
      'process.env.REACT_APP_API_URL': JSON.stringify('http://localhost:3001'),
    })
  );
  return config;
};
```

### Challenge: Large Bundle Size

**Solution**: 
- Enable code splitting in webpack
- Lazy load heavy dependencies
- Optimize story imports

### Challenge: Slow Build Times

**Solutions**:
- Use SWC instead of Babel (already configured)
- Limit stories during development with `--filter` flag
- Use webpack caching

---

## Next Steps

### Immediate Actions (Week 1)

1. [ ] Run Storybook init: `npx storybook@latest init`
2. [ ] Configure theme integration in preview.tsx
3. [ ] Create 5 core component stories
4. [ ] Verify CI/CD pipeline
5. [ ] Share initial Storybook with team

### Short Term (Month 1)

1. [ ] Complete Phase 2 (core components)
2. [ ] Set up deployment pipeline
3. [ ] Create documentation pages
4. [ ] Train team on Storybook workflow
5. [ ] Integrate into PR review process

### Long Term (Quarter 1)

1. [ ] Achieve 80%+ component coverage
2. [ ] Implement visual regression testing
3. [ ] Create comprehensive design system docs
4. [ ] Gather and act on team feedback
5. [ ] Establish maintenance routine

---

## Resources

### Documentation
- [Storybook Official Docs](https://storybook.js.org/docs/react/get-started/introduction)
- [Material-UI + Storybook](https://storybook.js.org/recipes/@mui/material)
- [Storybook Best Practices](https://storybook.js.org/docs/react/writing-stories/introduction)

### Tools
- [Chromatic](https://www.chromatic.com/) - Visual testing
- [Storybook Addon A11y](https://storybook.js.org/addons/@storybook/addon-a11y)
- [Storybook Test Runner](https://storybook.js.org/docs/react/writing-tests/test-runner)

### Community
- [Storybook Discord](https://discord.gg/storybook)
- [GitHub Discussions](https://github.com/storybookjs/storybook/discussions)

---

## Implementation Guide

For detailed implementation steps, timeline, and task breakdowns, see the companion document:

📋 **[Implementation Plan](./IMPLEMENTATION_storybook_integration.md)**

The implementation plan provides:
- **Day-by-day task breakdown** with time estimates
- **Detailed acceptance criteria** for each task
- **Ready-to-execute commands** and code snippets
- **Daily checklists** for tracking progress
- **Risk management** strategies
- **Testing & QA procedures**
- **Rollback plan** if needed
- **Resource allocation** and team requirements

### Quick Start

To begin implementation immediately:

1. **Review the Implementation Plan**: Read [IMPLEMENTATION_storybook_integration.md](./IMPLEMENTATION_storybook_integration.md)
2. **Allocate Resources**: Assign 1-2 frontend developers for 5 days
3. **Phase 1 - Day 1 Morning**: Run `npx storybook@latest init` in `apps/frontend/`
4. **Follow Daily Checklists**: Track progress with the daily task lists
5. **Review at Phase Gates**: Ensure quality gates are met before proceeding

### Implementation Timeline

| Phase | Duration | Focus |
|-------|----------|-------|
| Phase 1 | 0.5 days | Setup & Configuration |
| Phase 2 | 2 days | Core Components (20-25 stories) |
| Phase 3 | 2 days | Feature Components (15-20 stories) |
| Phase 4 | 0.5 days | Documentation & Deployment |
| **Total** | **5 days** | **Complete implementation** |

### Success Indicators

After completing the implementation:
- ✅ Storybook accessible at localhost:6006
- ✅ 35+ component stories documented
- ✅ Deployed to hosting platform
- ✅ Team trained and onboarded
- ✅ Integrated into development workflow

---

## Appendix A: Package Installation

```bash
cd apps/frontend

# Initialize Storybook (automated)
npx storybook@latest init

# Additional recommended addons
npm install --save-dev @storybook/addon-a11y
npm install --save-dev @storybook/addon-themes
npm install --save-dev @storybook/addon-viewport
npm install --save-dev @storybook/test
npm install --save-dev chromatic
```

---

## Appendix B: Example Stories

### Simple Component Story

```tsx
// StatusBadge.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { StatusBadge } from './StatusBadge';

const meta: Meta<typeof StatusBadge> = {
  title: 'Common/StatusBadge',
  component: StatusBadge,
  tags: ['autodocs'],
};

export default meta;
type Story = StoryObj<typeof StatusBadge>;

export const Draft: Story = {
  args: { status: 'draft' },
};

export const Pending: Story = {
  args: { status: 'pending' },
};

export const Paid: Story = {
  args: { status: 'paid' },
};
```

### Complex Component Story

```tsx
// DataTable.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { DataTable } from './DataTable';
import { mockInvoices } from '../../.storybook-helpers/mockData';

const meta: Meta<typeof DataTable> = {
  title: 'Common/DataTable',
  component: DataTable,
  parameters: {
    layout: 'padded',
  },
  tags: ['autodocs'],
};

export default meta;
type Story = StoryObj<typeof DataTable>;

export const Default: Story = {
  args: {
    columns: [
      { id: 'invoiceNumber', label: 'Invoice #' },
      { id: 'client', label: 'Client' },
      { id: 'amount', label: 'Amount', format: (val) => `$${val}` },
      { id: 'status', label: 'Status' },
    ],
    rows: mockInvoices,
    getRowId: (row) => row.id,
  },
};

export const WithSelection: Story = {
  args: {
    ...Default.args,
    selectable: true,
    onSelectionChange: (ids) => console.log('Selected:', ids),
  },
};

export const Empty: Story = {
  args: {
    ...Default.args,
    rows: [],
  },
};
```

---

## Appendix C: Checklist

### Pre-Implementation Checklist

- [ ] Review this specification with team
- [ ] Allocate time for Phase 1 (3-5 days)
- [ ] Identify component priorities
- [ ] Set up deployment target
- [ ] Create mock data strategy

### Implementation Checklist

- [ ] Install Storybook
- [ ] Configure theme integration
- [ ] Create first 5 stories
- [ ] Set up CI/CD pipeline
- [ ] Document workflow
- [ ] Train team

### Post-Implementation Checklist

- [ ] Achieve core component coverage
- [ ] Deploy to hosting
- [ ] Share with stakeholders
- [ ] Gather feedback
- [ ] Iterate and improve
- [ ] Establish maintenance routine

---

## Changelog

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-10-31 | Frontend Team | Initial specification |

---

**Document Status**: ✅ Ready for Review  
**Next Review Date**: After Phase 1 completion  
**Maintained By**: Frontend Team Lead
