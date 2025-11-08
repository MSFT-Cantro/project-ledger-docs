# Contributing to Project Ledger Documentation

Thank you for contributing to Project Ledger documentation!

## � Repository Structure

```
project-ledger-docs/
├── backlog/
│   ├── 0. Completed/          # Completed specifications and features
│   ├── 1. features/           # Planned feature specifications
│   └── 2. bugs/              # Bug reports and technical debt
├── deployment/               # Deployment guides and checklists
├── guides/                   # Development and setup guides
├── DEVELOPMENT_SETUP_GUIDE.md # Primary setup documentation
└── README.md                 # Repository overview
```

## �📝 Documentation Types

### Specifications (`SPEC_*.md`)
Technical specifications for features, including requirements, design, and implementation details.
- **Location**: `backlog/1. features/` for planned features, `backlog/0. Completed/` for implemented features
- **Format**: Requirements, design decisions, implementation notes, acceptance criteria

### Development Guides
Step-by-step instructions for development, deployment, and troubleshooting.
- **Location**: `guides/` directory
- **Key Files**: 
  - `DEVELOPMENT_SETUP_GUIDE.md` - Complete environment setup with sample data
  - `current_prompts.md` - AI prompts and development context

### Deployment Documentation
Deployment processes, checklists, and infrastructure guides.
- **Location**: `deployment/` directory
- **Includes**: Release procedures, environment configuration, troubleshooting

### Bug Reports (`_bugs.md`, `_techdebt.md`)
Documentation for bug tracking and technical debt management.
- **Location**: `backlog/2. bugs/`
- **Purpose**: Track issues, technical debt, and fixes across the project

## 🔄 Workflow

### For New Features
1. **Planning Phase**: Create specification in `backlog/1. features/SPEC_[feature-name].md`
2. **Implementation**: Track progress and update specification with implementation details
3. **Completion**: Move completed specification to `backlog/0. Completed/`
4. **Update Tracking**: Update `_features.md` with completion status

### For Bug Fixes and Technical Debt
1. **Documentation**: Record in `backlog/2. bugs/_bugs.md` or `_techdebt.md`
2. **Investigation**: Add root cause analysis and solution details
3. **Resolution**: Document fix and verification steps
4. **Update Status**: Mark as resolved in tracking files

### For Development Guides
1. **Create/Update**: Work in `guides/` directory
2. **Test Instructions**: Verify all setup steps work in fresh environment
3. **Cross-Reference**: Link to related specifications and deployment docs
4. **Keep Current**: Update with environment changes and new sample data

### For Deployment Documentation
1. **Document Process**: Create clear step-by-step procedures in `deployment/`
2. **Include Checklists**: Add verification steps and rollback procedures  
3. **Environment Specific**: Separate dev, staging, and production processes
4. **Update Regularly**: Keep in sync with infrastructure changes

## 🎯 Documentation Standards

### File Naming Conventions
- **Specifications**: `SPEC_[feature-name].md` 
- **Guides**: Descriptive names like `DEVELOPMENT_SETUP_GUIDE.md`
- **Tracking Files**: `_features.md`, `_bugs.md`, `_techdebt.md`
- **Deployment**: `[PROCESS]_[ENVIRONMENT].md` or `[PURPOSE]_GUIDE.md`

### Content Structure
- **Clear Title**: Descriptive and specific
- **Table of Contents**: For documents longer than 200 lines
- **Status Indicators**: Use ✅ for completed items, 🚧 for in progress
- **Code Blocks**: Properly formatted with language specification
- **Screenshots**: Include for UI changes or complex setup steps
- **Cross-References**: Link to related documentation and issues

### Sample Data Documentation
- **Environment State**: Keep `DEVELOPMENT_SETUP_GUIDE.md` current with actual database state
- **Credentials**: Document test accounts and access details
- **Business Documents**: List sample quotes, invoices, change orders with amounts
- **Verification Steps**: Include commands to verify environment setup

## ✅ Quality Checklist

### Before Submitting Documentation
- [ ] **Clear, descriptive title** that explains the purpose
- [ ] **Table of contents** for documents longer than 200 lines  
- [ ] **Code examples** properly formatted with syntax highlighting
- [ ] **Environment verification** - all setup steps tested in clean environment
- [ ] **Cross-references** to related specifications and guides
- [ ] **Status tracking** - updated relevant tracking files in `backlog/`
- [ ] **Current information** - reflects actual state of application/environment
- [ ] **Screenshots** included for UI changes or complex processes

### For Specifications
- [ ] **Requirements** clearly defined with acceptance criteria
- [ ] **Implementation details** with technical approach
- [ ] **Database changes** documented if applicable
- [ ] **API changes** documented with examples
- [ ] **UI/UX mockups** or descriptions included

### For Setup Guides
- [ ] **Prerequisites** clearly listed
- [ ] **Step-by-step instructions** that can be followed exactly
- [ ] **Verification commands** to confirm each step
- [ ] **Troubleshooting section** for common issues
- [ ] **Sample data descriptions** with actual values and credentials

### For Deployment Documentation  
- [ ] **Environment requirements** specified
- [ ] **Rollback procedures** documented
- [ ] **Verification steps** for successful deployment
- [ ] **Monitoring and alerting** setup documented

## 🔗 Cross-Repository References

### Main Application Repository
Link to issues and code in the main [project-ledger](https://github.com/MSFT-Cantro/project-ledger) repository when:
- Referencing specific code implementations
- Linking to related issues or pull requests
- Documenting API endpoints or database schemas

### Development Environment
Always reference the current state documented in:
- `guides/DEVELOPMENT_SETUP_GUIDE.md` - for environment setup
- Sample data and credentials for testing
- Container configurations and requirements

## 🚀 Recent Major Updates

### November 2025 - Database Seeding & Sample Data
- ✅ Fixed comprehensive database seeding with business documents
- ✅ Added sample quotes, estimates, invoices, and change orders
- ✅ Updated development guide with actual sample data listings
- ✅ Enhanced environment documentation with complete setup process

### Key Completed Features (moved to `backlog/0. Completed/`)
- Account subscription system with multi-tenant support
- User management with role-based permissions
- PayPal integration for subscription payments
- Tax configuration system
- Quote/estimate workflow with approval process

## 📧 Questions and Support

### For Documentation Issues
- Create an issue in this [project-ledger-docs](https://github.com/MSFT-Cantro/project-ledger-docs) repository
- Use appropriate labels: `documentation`, `guide`, `specification`, `bug`

### For Application Issues
- Open an issue in the main [project-ledger](https://github.com/MSFT-Cantro/project-ledger) repository
- Reference relevant documentation when reporting bugs or requesting features

### For Development Environment Issues
1. Check `guides/DEVELOPMENT_SETUP_GUIDE.md` first
2. Verify Docker containers are running: `docker-compose ps`
3. Check application logs: `docker-compose logs -f`
4. Test with provided sample credentials and data

## 📋 Priority Areas for Contribution

### High Priority
- [ ] API documentation for all endpoints
- [ ] Frontend component documentation
- [ ] Database schema documentation
- [ ] Security and authentication guides

### Medium Priority  
- [ ] Advanced configuration guides
- [ ] Performance optimization documentation
- [ ] Integration examples and tutorials
- [ ] Troubleshooting expansion

### Ongoing Maintenance
- [ ] Keep sample data current with seeding scripts
- [ ] Update screenshots when UI changes
- [ ] Verify all setup instructions work in latest environment
- [ ] Maintain cross-references between related documents

---

**Last Updated**: November 7, 2025  
**Repository Structure**: Current as of comprehensive documentation reorganization  
**Development Environment**: Verified with working sample data and complete setup
