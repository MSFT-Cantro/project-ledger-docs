# Project Ledger Features Documentation

## Project Overview
Project Ledger is a comprehensive project management system designed to streamline business operations. The system integrates client management, project tracking, quote generation, and invoice processing into a unified platform.

### Core Features
- **Client Management**: Track and manage client information, contact details, and project history
- **Project Tracking**: Monitor project status, assign team members, and track progress
- **Quote Generation**: Create and manage professional quotes with comprehensive features
- **Invoice Processing**: Generate and track invoices linked to projects and quotes

### Quote Management
- **Status Workflow**
  - Draft: Initial quote creation and editing
  - Sent: Quote has been sent to client for review
  - Accepted: Client has approved the quote
  - Rejected: Client has declined the quote

- **Line Item Management**
  - Detailed item descriptions
  - Quantity and unit price tracking
  - Per-item tax settings
  - Automatic total calculations
  - Support for discounts and adjustments

- **Financial Features**
  - Multi-currency support (USD, EUR, GBP, JPY)
  - Configurable tax rates and labels
  - Early payment discount options
  - Automatic currency conversion
  - Compound tax calculations

- **Validation and Rules**
  - Required line items validation
  - Valid date range enforcement
  - Tax rate boundaries (0-100%)
  - Automatic quote number generation
  - Client and project association checks

- **Document Generation**
  - Professional PDF quote generation
  - Customizable quote templates
  - Company branding support
  - Digital signature fields
  - Terms and conditions section

- **Integration Features**
  - Direct project linking
  - Invoice generation from quotes
  - Client history tracking
  - Payment terms configuration
  - Audit trail with metadata

### UI Components

- **DataTable Component**
  - Responsive design with mobile optimization
  - Column configuration with priority levels
  - Sortable columns with custom formatters
  - Row selection capabilities
  - Pagination with configurable page sizes
  - Mobile card view for better UX on small screens
  - Customizable row actions
  - Empty state handling

- **FilterPanel Component**
  - Collapsible filter interface
  - Multiple filter types supported:
    - Text search
    - Select dropdowns
    - Date pickers
    - Date range selectors
    - Number inputs
  - Responsive grid layout
  - Real-time filter updates
  - Clear and reset capabilities
  - Customizable filter groups

- **StatusBadge Component**
  - Predefined status types:
    - Draft, Sent, Accepted, Rejected
    - Created, In Progress, Completed
    - Paid, Pending
    - Active, Inactive
  - Color-coded status indicators
  - Configurable sizes
  - Custom styling support
  - Consistent status visualization

- **Form Components**
  - Standardized input validation
  - Error message handling
  - Real-time field validation
  - Form state management
  - Custom field components
  - Responsive layouts

### API Endpoints

- **Quote Management**
  - `GET /api/quotes`: List all quotes with optional filters
  - `GET /api/quotes/{id}`: Get quote details
  - `POST /api/quotes`: Create new quote
  - `PUT /api/quotes/{id}`: Update existing quote
  - `DELETE /api/quotes/{id}`: Delete quote
  - `GET /api/quotes/{id}/pdf`: Generate quote PDF
  - Additional operations:
    - Currency conversion
    - Quote number generation
    - Quote validation

- **Project Management**
  - `GET /api/projects`: List all projects
  - `GET /api/projects/{id}`: Get project details
  - `POST /api/projects`: Create new project
  - `PATCH /api/projects/{id}`: Update project
  - `DELETE /api/projects/{id}`: Delete project
  - `POST /api/projects/convert-quote/{quoteId}`: Convert quote to project

- **Invoice Management**
  - `GET /api/invoices`: List all invoices with filters
  - `GET /api/invoices/{id}`: Get invoice details
  - `POST /api/invoices`: Create new invoice
  - `PUT /api/invoices/{id}`: Update invoice
  - `DELETE /api/invoices/{id}`: Delete invoice
  - `GET /api/invoices/{id}/pdf`: Generate invoice PDF
  - `GET /api/invoices/{id}/payments`: Get payment history

- **Recurring Invoices**
  - `POST /api/invoices/recurring`: Create recurring invoice
  - `PUT /api/invoices/recurring/{id}`: Update recurring schedule
  - `POST /api/invoices/recurring/{id}/pause`: Pause recurring invoice
  - `POST /api/invoices/recurring/{id}/resume`: Resume recurring invoice
  - `POST /api/invoices/recurring/{id}/cancel`: Cancel recurring invoice

- **Payment Processing**
  - `POST /api/payments/create-intent`: Create Stripe payment intent
  - `POST /api/payments/confirm`: Confirm payment
  - `GET /api/payments/intent/{id}`: Get payment intent status
  - `GET /api/payments/methods/{customerId}`: List payment methods
  - `POST /api/payments/webhook`: Handle Stripe webhooks

### Invoice Management

- **Invoice Status Workflow**
  - Draft: Initial invoice creation phase
  - Sent: Invoice has been sent to client
  - Overdue: Payment deadline has passed
  - Paid: Full payment received
  - Void: Invoice has been cancelled

- **Payment Status Tracking**
  - Unpaid: No payments received
  - PartiallyPaid: Some payments received
  - Paid: Full payment completed
  - Processing: Payment is being processed
  - Succeeded: Payment successfully processed
  - Failed: Payment attempt failed
  - Refunded: Payment has been refunded

- **Recurring Invoice Features**
  - Frequency options (Monthly, Quarterly, Yearly)
  - Customizable start and end dates
  - Optional occurrence limits
  - Automatic next invoice generation
  - Pause/Resume functionality
  - Schedule management and tracking
  - Activity history

- **Payment Processing**
  - Stripe integration for online payments
  - Multiple payment method support
  - Payment intent creation and confirmation
  - Webhook handling for payment updates
  - Payment receipt generation
  - Refund processing capabilities

- **Line Item Management**
  - Detailed item descriptions
  - Quantity and unit price tracking
  - Automatic total calculations
  - Tax handling
  - Subtotal and total computation
  - Running balance tracking

- **Document Generation**
  - Professional PDF invoice generation
  - Customizable invoice templates
  - Company branding support
  - Payment history inclusion
  - Dynamic calculations and summaries

- **Client Integration**
  - Direct client association
  - Client contact information
  - Project and quote linking
  - Payment history tracking
  - Communication history

### Project Management

- **Project Status Workflow**
  - New: Initial project setup phase
  - In Progress: Active development/work ongoing
  - On Hold: Temporarily paused projects
  - Completed: Finished projects ready for invoicing

- **Timeline Management**
  - Planned start and end dates
  - Actual start and end date tracking
  - Timeline comparison and variance monitoring
  - Visual timeline display
  - Date-based filtering and reporting

- **Budget Control**
  - Quote amount tracking
  - Actual cost monitoring
  - Remaining budget calculations
  - Over-budget warnings
  - Budget variance analysis
  - Visual budget indicators (color-coded)

- **Client Integration**
  - Direct client association
  - Client contact information display
  - Project-client relationship management
  - Client communication history

- **Quote and Invoice Integration**
  - Quote to project conversion
  - Project-based invoice generation
  - Automatic invoice data population
  - Multiple invoices per project support
  - Quote reference maintenance

- **Project Details**
  - Comprehensive project descriptions
  - Status tracking and updates
  - Team member assignments
  - Project documentation storage
  - Activity logging and history

- **Project Actions**
  - Project creation from quotes
  - Status updates and modifications
  - Invoice generation for completed projects
  - Project archiving and deletion
  - Bulk operations support

### Integration with Existing Client Management
- Seamless client data integration across projects, quotes, and invoices
- Unified client view with complete project and financial history
- Multi-tenant architecture with account-based data separation

## Project Structure
```
project-ledger/
├── src/
│   ├── api/           # API integration layers
│   ├── components/    # React components
│   │   ├── auth/      # Authentication components
│   │   ├── clients/   # Client management
│   │   ├── common/    # Shared components
│   │   ├── invoices/  # Invoice management
│   │   ├── layout/    # Layout components
│   │   ├── projects/  # Project management
│   │   └── quotes/    # Quote management
│   ├── context/       # React context providers
│   ├── routes/        # Application routing
│   ├── theme/         # Styling and theming
│   └── types/         # TypeScript type definitions
```

### Major Dependencies
- **UI Framework**: MUI (Material-UI) v7.3.1
- **State Management**: React Query v5.85.5
- **Form Handling**: React Hook Form v7.62.0 with Yup validation
- **Data Display**: MUI X-Data Grid v7.0.0
- **PDF Generation**: @react-pdf/renderer v3.3.8
- **Date Handling**: date-fns v3.3.1
- **Decimal Operations**: decimal.js v10.4.3
- **HTTP Client**: Axios v1.11.0

### Testing Infrastructure
- Jest and React Testing Library for unit and integration tests
- Dedicated test directories (`__tests__/`) for each major feature
- Support for component, integration, and unit testing
- Testing utilities and mock data providers