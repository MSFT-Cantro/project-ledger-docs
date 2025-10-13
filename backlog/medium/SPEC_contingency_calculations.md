# Contingency Calculations System Specification

**Date:** October 12, 2025  
**Version:** 1.0  
**Status:** Not Implemented  

---

## 1. Overview

The Contingency Calculations System provides sophisticated contingency management for estimates and project quotes. It handles percentage-based contingencies, fixed-amount reserves, category-specific contingencies, and risk-based calculations to ensure accurate project cost estimation and budget protection.

---

## 2. Business Requirements

### 2.1 Core Functionality
- **Percentage-based Contingencies**: Apply percentage of subtotal as contingency
- **Fixed-amount Contingencies**: Add specific dollar amounts for known risks
- **Category-specific Contingencies**: Different rates for materials, labor, equipment
- **Risk-based Calculations**: Contingency based on project risk assessment
- **Tiered Contingencies**: Multiple contingency levels for complex projects
- **Client-specific Rules**: Custom contingency rates per client or project type

### 2.2 Contingency Types
- **Standard Contingency**: General project uncertainty buffer (5-15%)
- **Design Contingency**: Changes during design phase (10-20%)
- **Construction Contingency**: Field condition variations (5-10%)
- **Material Escalation**: Price increase protection (3-8%)
- **Weather Contingency**: Weather delay costs (2-5%)
- **Permit/Regulatory**: Unknown regulatory requirements (5-15%)
- **Client Change**: Scope change protection (10-25%)

### 2.3 Calculation Rules
- Contingencies apply only to estimates, not final invoices
- Multiple contingencies can be combined based on business rules
- Contingencies calculated after discounts but before tax
- Different contingency rates for different line item categories
- Maximum contingency limits to prevent excessive estimates
- Contingency approval workflows for high percentages

---

## 3. Data Model

### 3.1 Database Schema

```prisma
model ContingencyTemplate {
  id                    Int                     @id @default(autoincrement())
  name                  String                  // Template name
  description           String?                 // Template description
  
  // Template configuration
  isDefault             Boolean                 @default(false)
  isActive              Boolean                 @default(true)
  applicableDocTypes    Json                    // Array of document types (ESTIMATE, QUOTE)
  
  // Organization context
  accountId             Int
  createdBy             Int
  createdAt             DateTime                @default(now())
  updatedAt             DateTime                @updatedAt
  
  // Relations
  account               Account                 @relation(fields: [accountId], references: [id])
  createdByUser         User                    @relation(fields: [createdBy], references: [id])
  rules                 ContingencyRule[]
  applications          ContingencyApplication[]
  
  @@index([accountId])
  @@index([isDefault, isActive])
}

model ContingencyRule {
  id                    Int                     @id @default(autoincrement())
  templateId            Int
  
  // Rule definition
  ruleName              String                  // "Material Contingency", "Weather Buffer", etc.
  ruleType              ContingencyRuleType
  calculationMethod     ContingencyCalculation
  
  // Percentage-based rules
  percentage            Float?                  // Contingency percentage (0-100)
  minAmount             Float?                  // Minimum contingency amount
  maxAmount             Float?                  // Maximum contingency amount
  
  // Fixed amount rules
  fixedAmount           Float?                  // Fixed contingency amount
  
  // Conditional rules
  conditions            Json?                   // Conditions for rule application
  priority              Int                     @default(0) // Rule application order
  
  // Category targeting
  applicableCategories  Json?                   // Line item categories this applies to
  excludeCategories     Json?                   // Categories to exclude
  
  // Risk factors
  riskMultiplier        Float                   @default(1.0) // Risk-based adjustment
  
  // Metadata
  isActive              Boolean                 @default(true)
  notes                 String?
  
  // Relations
  template              ContingencyTemplate     @relation(fields: [templateId], references: [id], onDelete: Cascade)
  
  @@index([templateId])
  @@index([ruleType])
}

model ContingencyApplication {
  id                    Int                     @id @default(autoincrement())
  templateId            Int?                    // Template used (if any)
  
  // Document reference
  documentType          DocumentType            // QUOTE, ESTIMATE
  documentId            Int                     // Reference to document
  
  // Application details
  baseAmount            Float                   // Amount contingency is calculated on
  totalContingency      Float                   // Total contingency amount applied
  
  // Breakdown by rule
  ruleApplications      Json                    // Array of applied rules with amounts
  
  // Override settings
  isCustom              Boolean                 @default(false) // True if manually overridden
  customPercentage      Float?                  // Custom percentage override
  customAmount          Float?                  // Custom fixed amount override
  overrideReason        String?                 // Reason for override
  
  // Approval tracking
  requiresApproval      Boolean                 @default(false)
  approvedAt            DateTime?
  approvedBy            Int?
  approvalNotes         String?
  
  // Metadata
  appliedAt             DateTime                @default(now())
  appliedBy             Int
  notes                 String?
  
  // Relations
  template              ContingencyTemplate?    @relation(fields: [templateId], references: [id])
  appliedByUser         User                    @relation("ContingencyAppliedBy", fields: [appliedBy], references: [id])
  approvedByUser        User?                   @relation("ContingencyApprovedBy", fields: [approvedBy], references: [id])
  
  @@index([documentType, documentId])
  @@index([appliedAt])
}

model ContingencySettings {
  id                      Int                   @id @default(autoincrement())
  accountId               Int                   @unique
  
  // Default settings
  defaultContingencyPercent Float               @default(10.0) // Default contingency percentage
  maxContingencyPercent     Float               @default(30.0) // Maximum allowed contingency
  minContingencyAmount      Float               @default(0.0)  // Minimum contingency amount
  
  // Approval requirements
  approvalRequired        Boolean               @default(true)
  approvalThreshold       Float                 @default(20.0) // Percentage requiring approval
  contingencyApprovers    Json                  // Array of user IDs who can approve
  
  // Calculation preferences
  applyAfterDiscounts     Boolean               @default(true)
  applyBeforeTax          Boolean               @default(true)
  roundingMethod          RoundingMethod        @default(ROUND_UP)
  
  // Display settings
  showContingencyOnPDF    Boolean               @default(true)
  contingencyLabel        String                @default("Contingency")
  showContingencyBreakdown Boolean              @default(false)
  
  // Risk assessment
  enableRiskAssessment    Boolean               @default(false)
  riskFactors             Json?                 // Risk assessment criteria
  
  // Automation
  autoApplyContingency    Boolean               @default(true)
  defaultTemplateId       Int?                  // Default template to use
  
  createdAt               DateTime              @default(now())
  updatedAt               DateTime              @updatedAt
  
  // Relations
  account                 Account               @relation(fields: [accountId], references: [id], onDelete: Cascade)
  defaultTemplate         ContingencyTemplate?  @relation(fields: [defaultTemplateId], references: [id])
  
  @@index([accountId])
}

enum ContingencyRuleType {
  STANDARD          // Basic percentage contingency
  DESIGN_PHASE      // Design development contingency
  CONSTRUCTION      // Construction phase contingency
  MATERIAL_ESCALATION // Material price increase protection
  WEATHER           // Weather-related delays
  PERMIT_REGULATORY // Regulatory and permit unknowns
  CLIENT_CHANGES    // Client-driven scope changes
  RISK_BASED        // Risk assessment driven
  CATEGORY_SPECIFIC // Category-based contingency
  FIXED_AMOUNT      // Fixed dollar amount
  CUSTOM            // Custom business logic
}

enum ContingencyCalculation {
  PERCENTAGE_OF_SUBTOTAL    // Percentage of line items subtotal
  PERCENTAGE_OF_CATEGORY    // Percentage of specific categories
  PERCENTAGE_AFTER_DISCOUNT // Percentage after discounts applied
  FIXED_AMOUNT             // Fixed dollar amount
  RISK_WEIGHTED            // Risk factor weighted calculation
  TIERED_PERCENTAGE        // Different percentages by amount tiers
  CUSTOM_FORMULA           // Custom calculation formula
}

enum RoundingMethod {
  ROUND_UP      // Always round up to next dollar
  ROUND_DOWN    // Always round down
  ROUND_NEAREST // Round to nearest dollar
  NO_ROUNDING   // Keep exact decimal
}

enum DocumentType {
  QUOTE
  ESTIMATE
  CHANGE_ORDER
}
```

### 3.2 TypeScript Interfaces

```typescript
export interface ContingencyTemplate {
  id: string;
  name: string;
  description?: string;
  isDefault: boolean;
  isActive: boolean;
  applicableDocTypes: DocumentType[];
  
  accountId: string;
  createdBy: string;
  createdAt: string;
  updatedAt: string;
  
  rules: ContingencyRule[];
}

export interface ContingencyRule {
  id: string;
  templateId: string;
  ruleName: string;
  ruleType: ContingencyRuleType;
  calculationMethod: ContingencyCalculation;
  
  percentage?: number;
  minAmount?: number;
  maxAmount?: number;
  fixedAmount?: number;
  
  conditions?: any;
  priority: number;
  applicableCategories?: string[];
  excludeCategories?: string[];
  riskMultiplier: number;
  
  isActive: boolean;
  notes?: string;
}

export interface ContingencyApplication {
  id: string;
  templateId?: string;
  documentType: DocumentType;
  documentId: string;
  
  baseAmount: number;
  totalContingency: number;
  ruleApplications: RuleApplication[];
  
  isCustom: boolean;
  customPercentage?: number;
  customAmount?: number;
  overrideReason?: string;
  
  requiresApproval: boolean;
  approvedAt?: string;
  approvedBy?: string;
  approvalNotes?: string;
  
  appliedAt: string;
  appliedBy: string;
  notes?: string;
}

export interface RuleApplication {
  ruleId: string;
  ruleName: string;
  baseAmount: number;
  percentage?: number;
  calculatedAmount: number;
  appliedAmount: number;
  notes?: string;
}

export interface ContingencyCalculationInput {
  documentType: DocumentType;
  documentId: string;
  lineItems: LineItem[];
  subtotal: number;
  discounts?: number;
  templateId?: string;
  customSettings?: {
    percentage?: number;
    amount?: number;
    reason?: string;
  };
}

export interface ContingencyCalculationResult {
  baseAmount: number;
  totalContingency: number;
  effectivePercentage: number;
  ruleBreakdown: RuleApplication[];
  requiresApproval: boolean;
  warnings: string[];
  grandTotalWithContingency: number;
}
```

---

## 4. API Endpoints

### 4.1 Contingency Template Management

```typescript
// Create contingency template
POST /api/contingency-templates
Body: CreateContingencyTemplateInput
Response: ContingencyTemplate

// Get contingency template
GET /api/contingency-templates/:id
Response: ContingencyTemplate

// Update contingency template
PATCH /api/contingency-templates/:id
Body: Partial<CreateContingencyTemplateInput>
Response: ContingencyTemplate

// Delete contingency template
DELETE /api/contingency-templates/:id
Response: { success: boolean }

// List contingency templates
GET /api/contingency-templates
Query: {
  active?: boolean,
  documentType?: DocumentType
}
Response: ContingencyTemplate[]

// Set default template
POST /api/contingency-templates/:id/set-default
Response: ContingencyTemplate
```

### 4.2 Contingency Calculation

```typescript
// Calculate contingency
POST /api/contingency/calculate
Body: ContingencyCalculationInput
Response: ContingencyCalculationResult

// Apply contingency to document
POST /api/contingency/apply
Body: {
  documentType: DocumentType,
  documentId: string,
  templateId?: string,
  customSettings?: object
}
Response: ContingencyApplication

// Remove contingency application
DELETE /api/contingency-applications/:id
Response: { success: boolean }

// Update contingency application
PATCH /api/contingency-applications/:id
Body: {
  customPercentage?: number,
  customAmount?: number,
  overrideReason?: string
}
Response: ContingencyApplication

// Approve contingency application
POST /api/contingency-applications/:id/approve
Body: {
  approvedBy: string,
  approvalNotes?: string
}
Response: ContingencyApplication
```

### 4.3 Contingency Analysis & Reporting

```typescript
// Get contingency breakdown for document
GET /api/documents/:type/:id/contingency
Response: {
  application: ContingencyApplication,
  breakdown: RuleApplication[],
  totalWithContingency: number
}

// Contingency usage report
GET /api/contingency/reports/usage
Query: {
  dateFrom?: string,
  dateTo?: string,
  documentType?: DocumentType
}
Response: {
  totalDocuments: number,
  documentsWithContingency: number,
  averageContingencyPercent: number,
  totalContingencyAmount: number,
  byTemplate: Array<{
    template: ContingencyTemplate,
    usage: number,
    averageAmount: number
  }>
}

// Project contingency tracking
GET /api/projects/:id/contingency-tracking
Response: {
  estimatedContingency: number,
  usedContingency: number,
  remainingContingency: number,
  contingencyHistory: ContingencyApplication[]
}
```

---

## 5. Business Logic Services

### 5.1 Contingency Calculation Service

```typescript
export class ContingencyCalculationService {
  async calculateContingency(input: ContingencyCalculationInput): Promise<ContingencyCalculationResult> {
    // Get template or use default
    const template = input.templateId 
      ? await this.getTemplate(input.templateId)
      : await this.getDefaultTemplate(input.documentType);
    
    if (!template) {
      return this.createEmptyResult(input.subtotal);
    }
    
    // Calculate base amount (after discounts if configured)
    const settings = await this.getContingencySettings();
    const baseAmount = settings.applyAfterDiscounts 
      ? input.subtotal - (input.discounts || 0)
      : input.subtotal;
    
    // Apply contingency rules
    const ruleApplications = await this.applyContingencyRules(
      template.rules,
      input.lineItems,
      baseAmount
    );
    
    // Calculate total contingency
    let totalContingency = ruleApplications.reduce(
      (sum, rule) => sum + rule.appliedAmount, 
      0
    );
    
    // Apply custom overrides
    if (input.customSettings) {
      totalContingency = await this.applyCustomSettings(
        totalContingency,
        baseAmount,
        input.customSettings
      );
    }
    
    // Apply limits and rounding
    totalContingency = await this.applyLimitsAndRounding(totalContingency, baseAmount);
    
    // Check approval requirements
    const effectivePercentage = (totalContingency / baseAmount) * 100;
    const requiresApproval = effectivePercentage > settings.approvalThreshold;
    
    return {
      baseAmount,
      totalContingency,
      effectivePercentage,
      ruleBreakdown: ruleApplications,
      requiresApproval,
      warnings: await this.generateWarnings(totalContingency, baseAmount, settings),
      grandTotalWithContingency: baseAmount + totalContingency
    };
  }
  
  private async applyContingencyRules(
    rules: ContingencyRule[],
    lineItems: LineItem[],
    baseAmount: number
  ): Promise<RuleApplication[]> {
    const applications: RuleApplication[] = [];
    
    // Sort rules by priority
    const sortedRules = rules
      .filter(rule => rule.isActive)
      .sort((a, b) => a.priority - b.priority);
    
    for (const rule of sortedRules) {
      const application = await this.applyContingencyRule(rule, lineItems, baseAmount);
      if (application.appliedAmount > 0) {
        applications.push(application);
      }
    }
    
    return applications;
  }
  
  private async applyContingencyRule(
    rule: ContingencyRule,
    lineItems: LineItem[],
    baseAmount: number
  ): Promise<RuleApplication> {
    let calculationBase = baseAmount;
    
    // Filter line items if category-specific
    if (rule.applicableCategories?.length || rule.excludeCategories?.length) {
      const filteredItems = this.filterLineItemsByCategory(
        lineItems,
        rule.applicableCategories,
        rule.excludeCategories
      );
      calculationBase = filteredItems.reduce(
        (sum, item) => sum + (item.quantity * item.unitPrice),
        0
      );
    }
    
    let calculatedAmount = 0;
    
    switch (rule.calculationMethod) {
      case 'PERCENTAGE_OF_SUBTOTAL':
      case 'PERCENTAGE_OF_CATEGORY':
      case 'PERCENTAGE_AFTER_DISCOUNT':
        calculatedAmount = calculationBase * (rule.percentage! / 100);
        break;
        
      case 'FIXED_AMOUNT':
        calculatedAmount = rule.fixedAmount!;
        break;
        
      case 'RISK_WEIGHTED':
        const riskFactor = await this.calculateRiskFactor(rule.conditions);
        calculatedAmount = calculationBase * (rule.percentage! / 100) * riskFactor;
        break;
        
      case 'TIERED_PERCENTAGE':
        calculatedAmount = await this.calculateTieredPercentage(
          calculationBase,
          rule.conditions
        );
        break;
        
      case 'CUSTOM_FORMULA':
        calculatedAmount = await this.executeCustomFormula(
          rule.conditions,
          calculationBase,
          lineItems
        );
        break;
    }
    
    // Apply risk multiplier
    calculatedAmount *= rule.riskMultiplier;
    
    // Apply min/max limits
    if (rule.minAmount && calculatedAmount < rule.minAmount) {
      calculatedAmount = rule.minAmount;
    }
    if (rule.maxAmount && calculatedAmount > rule.maxAmount) {
      calculatedAmount = rule.maxAmount;
    }
    
    return {
      ruleId: rule.id,
      ruleName: rule.ruleName,
      baseAmount: calculationBase,
      percentage: rule.percentage,
      calculatedAmount,
      appliedAmount: Math.round(calculatedAmount * 100) / 100, // Round to cents
      notes: rule.notes
    };
  }
  
  async applyContingencyToDocument(
    documentType: DocumentType,
    documentId: string,
    templateId?: string,
    customSettings?: any
  ): Promise<ContingencyApplication> {
    // Get document details
    const document = await this.getDocument(documentType, documentId);
    const lineItems = await this.getDocumentLineItems(documentType, documentId);
    
    // Calculate contingency
    const calculation = await this.calculateContingency({
      documentType,
      documentId,
      lineItems,
      subtotal: document.subtotal,
      discounts: document.discounts,
      templateId,
      customSettings
    });
    
    // Create application record
    const application = await prisma.contingencyApplication.create({
      data: {
        templateId: templateId ? parseInt(templateId) : null,
        documentType,
        documentId: parseInt(documentId),
        baseAmount: calculation.baseAmount,
        totalContingency: calculation.totalContingency,
        ruleApplications: calculation.ruleBreakdown,
        isCustom: !!customSettings,
        customPercentage: customSettings?.percentage,
        customAmount: customSettings?.amount,
        overrideReason: customSettings?.reason,
        requiresApproval: calculation.requiresApproval,
        appliedBy: this.getCurrentUserId(), // From context
        notes: customSettings?.notes
      }
    });
    
    // Update document totals
    await this.updateDocumentTotals(documentType, documentId, calculation.totalContingency);
    
    // Send approval notification if required
    if (calculation.requiresApproval) {
      await this.notifyContingencyApprovalRequired(application);
    }
    
    return application;
  }
  
  private async updateDocumentTotals(
    documentType: DocumentType,
    documentId: string,
    contingencyAmount: number
  ): Promise<void> {
    const updateData = {
      contingency: contingencyAmount,
      // Recalculate grand total: subtotal - discounts + fees + contingency + tax
      grandTotal: {
        increment: contingencyAmount
      }
    };
    
    switch (documentType) {
      case 'QUOTE':
        await prisma.quote.update({
          where: { id: parseInt(documentId) },
          data: updateData
        });
        break;
        
      case 'ESTIMATE':
        // Handle estimate updates if separate from quote
        break;
        
      case 'CHANGE_ORDER':
        await prisma.changeOrder.update({
          where: { id: parseInt(documentId) },
          data: updateData
        });
        break;
    }
  }
}
```

### 5.2 Risk Assessment Service

```typescript
export class RiskAssessmentService {
  async calculateProjectRisk(
    projectData: any,
    riskFactors: any[]
  ): Promise<number> {
    let totalRiskScore = 1.0; // Base multiplier
    
    for (const factor of riskFactors) {
      const factorScore = await this.evaluateRiskFactor(factor, projectData);
      totalRiskScore *= factorScore;
    }
    
    // Cap risk multiplier between 0.5 and 3.0
    return Math.max(0.5, Math.min(3.0, totalRiskScore));
  }
  
  private async evaluateRiskFactor(factor: any, projectData: any): Promise<number> {
    switch (factor.type) {
      case 'PROJECT_SIZE':
        return this.evaluateProjectSizeRisk(projectData.totalValue, factor.thresholds);
        
      case 'CLIENT_HISTORY':
        return this.evaluateClientHistoryRisk(projectData.clientId, factor.criteria);
        
      case 'PROJECT_COMPLEXITY':
        return this.evaluateComplexityRisk(projectData.lineItems, factor.complexity);
        
      case 'TIMELINE_PRESSURE':
        return this.evaluateTimelineRisk(projectData.duration, factor.thresholds);
        
      case 'LOCATION_RISK':
        return this.evaluateLocationRisk(projectData.location, factor.riskAreas);
        
      default:
        return 1.0;
    }
  }
}
```

---

## 6. User Interface Components

### 6.1 Contingency Configuration

```typescript
interface ContingencyTemplateEditorProps {
  template?: ContingencyTemplate;
  onSave: (template: ContingencyTemplate) => void;
  onCancel: () => void;
}

export function ContingencyTemplateEditor({ template, onSave, onCancel }: ContingencyTemplateEditorProps) {
  const [formData, setFormData] = useState({
    name: template?.name || '',
    description: template?.description || '',
    isDefault: template?.isDefault || false,
    applicableDocTypes: template?.applicableDocTypes || ['ESTIMATE'],
    rules: template?.rules || []
  });

  const addRule = () => {
    const newRule: Partial<ContingencyRule> = {
      ruleName: '',
      ruleType: 'STANDARD',
      calculationMethod: 'PERCENTAGE_OF_SUBTOTAL',
      percentage: 10,
      priority: formData.rules.length,
      isActive: true
    };
    
    setFormData(prev => ({
      ...prev,
      rules: [...prev.rules, newRule as ContingencyRule]
    }));
  };

  const updateRule = (index: number, updates: Partial<ContingencyRule>) => {
    setFormData(prev => ({
      ...prev,
      rules: prev.rules.map((rule, i) => i === index ? { ...rule, ...updates } : rule)
    }));
  };

  const removeRule = (index: number) => {
    setFormData(prev => ({
      ...prev,
      rules: prev.rules.filter((_, i) => i !== index)
    }));
  };

  return (
    <Dialog open maxWidth="lg" fullWidth>
      <DialogTitle>
        {template ? 'Edit' : 'Create'} Contingency Template
      </DialogTitle>
      
      <DialogContent>
        <Box sx={{ mb: 3 }}>
          <TextField
            fullWidth
            label="Template Name"
            value={formData.name}
            onChange={(e) => setFormData(prev => ({ ...prev, name: e.target.value }))}
            sx={{ mb: 2 }}
          />
          
          <TextField
            fullWidth
            label="Description"
            value={formData.description}
            onChange={(e) => setFormData(prev => ({ ...prev, description: e.target.value }))}
            multiline
            rows={2}
            sx={{ mb: 2 }}
          />

          <FormControlLabel
            control={
              <Checkbox
                checked={formData.isDefault}
                onChange={(e) => setFormData(prev => ({ ...prev, isDefault: e.target.checked }))}
              />
            }
            label="Default Template"
          />
        </Box>

        {/* Document Types */}
        <Typography variant="h6" gutterBottom>
          Applicable Document Types
        </Typography>
        <FormGroup row sx={{ mb: 3 }}>
          {['ESTIMATE', 'QUOTE', 'CHANGE_ORDER'].map(type => (
            <FormControlLabel
              key={type}
              control={
                <Checkbox
                  checked={formData.applicableDocTypes.includes(type as DocumentType)}
                  onChange={(e) => {
                    const types = e.target.checked
                      ? [...formData.applicableDocTypes, type as DocumentType]
                      : formData.applicableDocTypes.filter(t => t !== type);
                    setFormData(prev => ({ ...prev, applicableDocTypes: types }));
                  }}
                />
              }
              label={type}
            />
          ))}
        </FormGroup>

        {/* Contingency Rules */}
        <Box sx={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center', mb: 2 }}>
          <Typography variant="h6">
            Contingency Rules
          </Typography>
          <Button
            variant="outlined"
            startIcon={<Add />}
            onClick={addRule}
          >
            Add Rule
          </Button>
        </Box>

        {formData.rules.map((rule, index) => (
          <Card key={index} sx={{ mb: 2, p: 2 }}>
            <Grid container spacing={2} alignItems="center">
              <Grid item xs={12} sm={3}>
                <TextField
                  fullWidth
                  label="Rule Name"
                  value={rule.ruleName}
                  onChange={(e) => updateRule(index, { ruleName: e.target.value })}
                />
              </Grid>
              
              <Grid item xs={12} sm={2}>
                <FormControl fullWidth>
                  <InputLabel>Type</InputLabel>
                  <Select
                    value={rule.ruleType}
                    onChange={(e) => updateRule(index, { ruleType: e.target.value as ContingencyRuleType })}
                  >
                    <MenuItem value="STANDARD">Standard</MenuItem>
                    <MenuItem value="DESIGN_PHASE">Design Phase</MenuItem>
                    <MenuItem value="CONSTRUCTION">Construction</MenuItem>
                    <MenuItem value="MATERIAL_ESCALATION">Material Escalation</MenuItem>
                    <MenuItem value="WEATHER">Weather</MenuItem>
                    <MenuItem value="PERMIT_REGULATORY">Permit/Regulatory</MenuItem>
                    <MenuItem value="CLIENT_CHANGES">Client Changes</MenuItem>
                    <MenuItem value="RISK_BASED">Risk Based</MenuItem>
                  </Select>
                </FormControl>
              </Grid>
              
              <Grid item xs={12} sm={2}>
                <FormControl fullWidth>
                  <InputLabel>Method</InputLabel>
                  <Select
                    value={rule.calculationMethod}
                    onChange={(e) => updateRule(index, { calculationMethod: e.target.value as ContingencyCalculation })}
                  >
                    <MenuItem value="PERCENTAGE_OF_SUBTOTAL">% of Subtotal</MenuItem>
                    <MenuItem value="PERCENTAGE_OF_CATEGORY">% of Category</MenuItem>
                    <MenuItem value="FIXED_AMOUNT">Fixed Amount</MenuItem>
                    <MenuItem value="RISK_WEIGHTED">Risk Weighted</MenuItem>
                  </Select>
                </FormControl>
              </Grid>
              
              <Grid item xs={12} sm={2}>
                <TextField
                  fullWidth
                  label={rule.calculationMethod === 'FIXED_AMOUNT' ? 'Amount ($)' : 'Percentage (%)'}
                  type="number"
                  value={rule.calculationMethod === 'FIXED_AMOUNT' ? rule.fixedAmount : rule.percentage}
                  onChange={(e) => {
                    const value = parseFloat(e.target.value);
                    const update = rule.calculationMethod === 'FIXED_AMOUNT'
                      ? { fixedAmount: value }
                      : { percentage: value };
                    updateRule(index, update);
                  }}
                  inputProps={{ min: 0, step: 0.1 }}
                />
              </Grid>
              
              <Grid item xs={12} sm={1}>
                <TextField
                  fullWidth
                  label="Priority"
                  type="number"
                  value={rule.priority}
                  onChange={(e) => updateRule(index, { priority: parseInt(e.target.value) })}
                  inputProps={{ min: 0 }}
                />
              </Grid>
              
              <Grid item xs={12} sm={1}>
                <IconButton
                  color="error"
                  onClick={() => removeRule(index)}
                >
                  <Delete />
                </IconButton>
              </Grid>
            </Grid>
            
            {/* Advanced options */}
            <Accordion sx={{ mt: 2 }}>
              <AccordionSummary expandIcon={<ExpandMore />}>
                <Typography variant="body2">Advanced Options</Typography>
              </AccordionSummary>
              <AccordionDetails>
                <Grid container spacing={2}>
                  <Grid item xs={6}>
                    <TextField
                      fullWidth
                      label="Minimum Amount ($)"
                      type="number"
                      value={rule.minAmount || ''}
                      onChange={(e) => updateRule(index, { minAmount: parseFloat(e.target.value) || undefined })}
                    />
                  </Grid>
                  <Grid item xs={6}>
                    <TextField
                      fullWidth
                      label="Maximum Amount ($)"
                      type="number"
                      value={rule.maxAmount || ''}
                      onChange={(e) => updateRule(index, { maxAmount: parseFloat(e.target.value) || undefined })}
                    />
                  </Grid>
                  <Grid item xs={12}>
                    <TextField
                      fullWidth
                      label="Notes"
                      value={rule.notes || ''}
                      onChange={(e) => updateRule(index, { notes: e.target.value })}
                      multiline
                      rows={2}
                    />
                  </Grid>
                </Grid>
              </AccordionDetails>
            </Accordion>
          </Card>
        ))}
      </DialogContent>
      
      <DialogActions>
        <Button onClick={onCancel}>
          Cancel
        </Button>
        <Button
          variant="contained"
          onClick={() => onSave(formData as ContingencyTemplate)}
          disabled={!formData.name || formData.rules.length === 0}
        >
          Save Template
        </Button>
      </DialogActions>
    </Dialog>
  );
}
```

### 6.2 Contingency Application Component

```typescript
interface ContingencyCalculatorProps {
  documentType: DocumentType;
  documentId: string;
  subtotal: number;
  lineItems: LineItem[];
  onContingencyApplied: (application: ContingencyApplication) => void;
}

export function ContingencyCalculator({
  documentType,
  documentId,
  subtotal,
  lineItems,
  onContingencyApplied
}: ContingencyCalculatorProps) {
  const [selectedTemplate, setSelectedTemplate] = useState<string>('');
  const [customPercentage, setCustomPercentage] = useState<number>(0);
  const [customAmount, setCustomAmount] = useState<number>(0);
  const [calculationMode, setCalculationMode] = useState<'template' | 'custom'>('template');
  const [calculation, setCalculation] = useState<ContingencyCalculationResult | null>(null);

  const { data: templates } = useQuery({
    queryKey: ['contingency-templates', documentType],
    queryFn: () => getContingencyTemplates({ documentType })
  });

  const { data: defaultTemplate } = useQuery({
    queryKey: ['default-contingency-template', documentType],
    queryFn: () => getDefaultContingencyTemplate(documentType)
  });

  // Auto-select default template
  useEffect(() => {
    if (defaultTemplate && !selectedTemplate) {
      setSelectedTemplate(defaultTemplate.id);
    }
  }, [defaultTemplate, selectedTemplate]);

  // Calculate contingency when inputs change
  useEffect(() => {
    if (calculationMode === 'template' && selectedTemplate) {
      calculateTemplateContingency();
    } else if (calculationMode === 'custom' && (customPercentage > 0 || customAmount > 0)) {
      calculateCustomContingency();
    }
  }, [calculationMode, selectedTemplate, customPercentage, customAmount]);

  const calculateTemplateContingency = async () => {
    try {
      const result = await calculateContingency({
        documentType,
        documentId,
        lineItems,
        subtotal,
        templateId: selectedTemplate
      });
      setCalculation(result);
    } catch (error) {
      console.error('Contingency calculation failed:', error);
    }
  };

  const calculateCustomContingency = async () => {
    try {
      const result = await calculateContingency({
        documentType,
        documentId,
        lineItems,
        subtotal,
        customSettings: {
          percentage: customPercentage,
          amount: customAmount
        }
      });
      setCalculation(result);
    } catch (error) {
      console.error('Custom contingency calculation failed:', error);
    }
  };

  const applyContingency = async () => {
    if (!calculation) return;

    try {
      const application = await applyContingencyToDocument({
        documentType,
        documentId,
        templateId: calculationMode === 'template' ? selectedTemplate : undefined,
        customSettings: calculationMode === 'custom' ? {
          percentage: customPercentage,
          amount: customAmount,
          reason: 'Custom contingency application'
        } : undefined
      });

      onContingencyApplied(application);
      toast.success('Contingency applied successfully');
    } catch (error) {
      toast.error('Failed to apply contingency: ' + error.message);
    }
  };

  return (
    <Card sx={{ p: 3 }}>
      <Typography variant="h6" gutterBottom>
        Contingency Calculator
      </Typography>

      {/* Calculation Mode Selection */}
      <FormControl component="fieldset" sx={{ mb: 3 }}>
        <RadioGroup
          value={calculationMode}
          onChange={(e) => setCalculationMode(e.target.value as 'template' | 'custom')}
          row
        >
          <FormControlLabel value="template" control={<Radio />} label="Use Template" />
          <FormControlLabel value="custom" control={<Radio />} label="Custom Calculation" />
        </RadioGroup>
      </FormControl>

      {/* Template Selection */}
      {calculationMode === 'template' && (
        <Box sx={{ mb: 3 }}>
          <FormControl fullWidth>
            <InputLabel>Contingency Template</InputLabel>
            <Select
              value={selectedTemplate}
              onChange={(e) => setSelectedTemplate(e.target.value)}
            >
              {templates?.map(template => (
                <MenuItem key={template.id} value={template.id}>
                  <Box>
                    <Typography variant="body1">
                      {template.name}
                      {template.isDefault && <Chip label="Default" size="small" sx={{ ml: 1 }} />}
                    </Typography>
                    {template.description && (
                      <Typography variant="caption" color="text.secondary">
                        {template.description}
                      </Typography>
                    )}
                  </Box>
                </MenuItem>
              ))}
            </Select>
          </FormControl>
        </Box>
      )}

      {/* Custom Inputs */}
      {calculationMode === 'custom' && (
        <Box sx={{ mb: 3 }}>
          <Grid container spacing={2}>
            <Grid item xs={6}>
              <TextField
                fullWidth
                label="Percentage (%)"
                type="number"
                value={customPercentage}
                onChange={(e) => setCustomPercentage(parseFloat(e.target.value) || 0)}
                inputProps={{ min: 0, max: 100, step: 0.1 }}
              />
            </Grid>
            <Grid item xs={6}>
              <TextField
                fullWidth
                label="Fixed Amount ($)"
                type="number"
                value={customAmount}
                onChange={(e) => setCustomAmount(parseFloat(e.target.value) || 0)}
                inputProps={{ min: 0, step: 0.01 }}
              />
            </Grid>
          </Grid>
        </Box>
      )}

      {/* Calculation Results */}
      {calculation && (
        <Box sx={{ mb: 3 }}>
          <Alert 
            severity={calculation.requiresApproval ? 'warning' : 'info'}
            sx={{ mb: 2 }}
          >
            {calculation.requiresApproval 
              ? 'This contingency amount requires approval'
              : 'Contingency calculation ready to apply'
            }
          </Alert>

          <Card variant="outlined" sx={{ p: 2, bgcolor: 'grey.50' }}>
            <Typography variant="h6" gutterBottom>
              Contingency Summary
            </Typography>
            
            <Box sx={{ display: 'flex', justifyContent: 'space-between', mb: 1 }}>
              <Typography>Base Amount:</Typography>
              <Typography>${calculation.baseAmount.toLocaleString()}</Typography>
            </Box>
            
            <Box sx={{ display: 'flex', justifyContent: 'space-between', mb: 1 }}>
              <Typography>Contingency Amount:</Typography>
              <Typography>${calculation.totalContingency.toLocaleString()}</Typography>
            </Box>
            
            <Box sx={{ display: 'flex', justifyContent: 'space-between', mb: 1 }}>
              <Typography>Effective Percentage:</Typography>
              <Typography>{calculation.effectivePercentage.toFixed(2)}%</Typography>
            </Box>
            
            <Divider sx={{ my: 1 }} />
            
            <Box sx={{ display: 'flex', justifyContent: 'space-between', fontWeight: 'bold' }}>
              <Typography variant="h6">Total with Contingency:</Typography>
              <Typography variant="h6" color="primary.main">
                ${calculation.grandTotalWithContingency.toLocaleString()}
              </Typography>
            </Box>
          </Card>

          {/* Rule Breakdown */}
          {calculation.ruleBreakdown.length > 0 && (
            <Accordion sx={{ mt: 2 }}>
              <AccordionSummary expandIcon={<ExpandMore />}>
                <Typography variant="body1">Contingency Breakdown</Typography>
              </AccordionSummary>
              <AccordionDetails>
                {calculation.ruleBreakdown.map((rule, index) => (
                  <Box key={index} sx={{ mb: 1, p: 1, bgcolor: 'grey.100', borderRadius: 1 }}>
                    <Box sx={{ display: 'flex', justifyContent: 'space-between' }}>
                      <Typography variant="body2" fontWeight="medium">
                        {rule.ruleName}
                      </Typography>
                      <Typography variant="body2">
                        ${rule.appliedAmount.toLocaleString()}
                      </Typography>
                    </Box>
                    {rule.percentage && (
                      <Typography variant="caption" color="text.secondary">
                        {rule.percentage}% of ${rule.baseAmount.toLocaleString()}
                      </Typography>
                    )}
                  </Box>
                ))}
              </AccordionDetails>
            </Accordion>
          )}

          {/* Warnings */}
          {calculation.warnings.length > 0 && (
            <Alert severity="warning" sx={{ mt: 2 }}>
              <AlertTitle>Warnings</AlertTitle>
              <ul>
                {calculation.warnings.map((warning, index) => (
                  <li key={index}>{warning}</li>
                ))}
              </ul>
            </Alert>
          )}
        </Box>
      )}

      {/* Apply Button */}
      <Box sx={{ display: 'flex', gap: 2, justifyContent: 'flex-end' }}>
        <Button
          variant="contained"
          onClick={applyContingency}
          disabled={!calculation || calculation.totalContingency <= 0}
        >
          Apply Contingency
          {calculation?.requiresApproval && ' (Requires Approval)'}
        </Button>
      </Box>
    </Card>
  );
}
```

---

## 7. Implementation Plan

### Phase 1: Core Infrastructure (Week 1-2)
- [ ] Database schema migration for contingency tables
- [ ] Basic contingency template CRUD operations
- [ ] Contingency calculation engine with percentage and fixed amount support
- [ ] Integration with existing document totals calculation

### Phase 2: Advanced Calculations (Week 2-3)
- [ ] Category-specific contingency rules
- [ ] Risk-based contingency calculations
- [ ] Tiered percentage calculations
- [ ] Custom formula engine for complex contingency rules

### Phase 3: User Interface (Week 3-4)
- [ ] Contingency template management interface
- [ ] Contingency calculator component for quotes/estimates
- [ ] Contingency breakdown display and reporting
- [ ] Approval workflow interface for high contingency amounts

### Phase 4: Advanced Features (Week 4-5)
- [ ] Risk assessment integration
- [ ] Project contingency tracking and management
- [ ] Contingency usage analytics and reporting
- [ ] Client-specific contingency rules and templates

### Phase 5: Integration & Testing (Week 5-6)
- [ ] Integration with PDF generation for contingency display
- [ ] Email notifications for contingency approvals
- [ ] Comprehensive testing of contingency calculations
- [ ] Performance optimization for complex contingency rules

---

## 8. Success Criteria

### Functional Requirements
- [ ] Create and manage contingency templates with multiple rule types
- [ ] Apply contingencies automatically to estimates and quotes
- [ ] Calculate accurate contingencies based on percentage, fixed amount, and risk factors
- [ ] Support category-specific and tiered contingency calculations
- [ ] Require approval for contingencies exceeding configured thresholds
- [ ] Generate detailed contingency breakdown reports

### Technical Requirements
- [ ] Contingency calculation accuracy of 99.99%
- [ ] Support for complex contingency rules with conditional logic
- [ ] Contingency calculation processing < 200ms
- [ ] Integration with existing document total calculations
- [ ] Comprehensive audit trail for all contingency applications

### Business Requirements
- [ ] Improve estimate accuracy with systematic contingency application
- [ ] Reduce manual contingency calculation errors by 95%
- [ ] Enable risk-based contingency management
- [ ] Support different contingency strategies per client or project type
- [ ] Provide detailed contingency tracking and reporting
- [ ] Ensure consistent contingency application across all estimates

---

**File Version:** 1.0 – Contingency Calculations System Specification  
**Last Updated:** October 12, 2025