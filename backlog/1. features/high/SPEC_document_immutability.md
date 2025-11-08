# Document Immutability & Legal Compliance Specification

**Date:** October 12, 2025  
**Version:** 1.0  
**Status:** Not Implemented  

---

## 1. Overview

Document Immutability & Legal Compliance ensures that all issued billing documents (quotes, invoices, change orders) are cryptographically secured, tamper-evident, and legally compliant for audit purposes. This system provides SHA-256 hashing, timestamping, and retention policies required for business compliance.

---

## 2. Business Requirements

### 2.1 Legal Compliance
- **7+ Year Retention**: All issued documents retained for minimum 7 years
- **Immutability**: Issued documents cannot be modified, only replaced with new versions
- **Audit Trail**: Complete history of all document changes and status updates
- **Digital Integrity**: Cryptographic proof that documents haven't been tampered with
- **Timestamping**: Legally-valid timestamps for all document actions

### 2.2 Technical Requirements
- **SHA-256 Hashing**: All PDF documents hashed and verified
- **Document Versioning**: Support for document revisions with immutable history
- **Secure Storage**: Document storage with encryption at rest
- **Hash Verification**: API endpoints to verify document integrity
- **Automated Archival**: Automatic archival of old documents with compression

### 2.3 Regulatory Standards
- **SOX Compliance**: Support for Sarbanes-Oxley financial reporting requirements
- **GAAP Standards**: Generally Accepted Accounting Principles compliance
- **Industry Standards**: Support for industry-specific retention requirements
- **Data Protection**: GDPR/CCPA compliant data handling and retention
- **Export Controls**: Ability to export compliant document sets

---

## 3. Data Model

### 3.1 Database Schema Additions

```prisma
model DocumentHash {
  id              String            @id @default(cuid())
  documentType    DocumentType      // QUOTE, INVOICE, CHANGE_ORDER
  documentId      Int               // Reference to source document
  version         Int               @default(1)
  
  // Hash Data
  sha256Hash      String            // SHA-256 hash of PDF content
  pdfSize         Int               // Size in bytes
  pdfChecksum     String            // Additional MD5 checksum
  
  // Content Metadata
  contentType     String            @default("application/pdf")
  filename        String            // Original filename
  storageUrl      String?           // S3/storage URL if different from ID
  
  // Legal Metadata
  issuedAt        DateTime          // When document was officially issued
  issuedBy        Int               // User who issued the document
  issuedStatus    String            // DRAFT, ISSUED, REVISED, ARCHIVED
  
  // Verification
  verifiedAt      DateTime?         // Last verification timestamp
  verificationStatus VerificationStatus @default(PENDING)
  verificationErrors Json?          // Any verification failures
  
  // Retention
  retentionDate   DateTime          // Legal retention expiry date
  archivedAt      DateTime?         // When moved to cold storage
  archivedBy      Int?              // User who archived
  
  // Timestamps
  createdAt       DateTime          @default(now())
  updatedAt       DateTime          @updatedAt
  
  // Relations
  document        DocumentReference @relation(fields: [documentType, documentId], references: [type, id])
  issuedByUser    User             @relation("DocumentHashIssuedBy", fields: [issuedBy], references: [id])
  archivedByUser  User?            @relation("DocumentHashArchivedBy", fields: [archivedBy], references: [id])
  verifications   HashVerification[]
  auditLogs       DocumentAuditLog[]
  
  @@unique([documentType, documentId, version])
  @@index([sha256Hash])
  @@index([issuedAt])
  @@index([retentionDate])
  @@index([verificationStatus])
}

model HashVerification {
  id              Int               @id @default(autoincrement())
  documentHashId  String
  
  // Verification Details
  verifiedAt      DateTime          @default(now())
  verifiedBy      Int?              // User who requested verification (optional for automated)
  verificationMethod VerificationMethod // AUTOMATIC, MANUAL, EXTERNAL
  
  // Results
  isValid         Boolean
  originalHash    String            // Expected hash
  currentHash     String            // Computed hash
  sizeMismatch    Boolean           @default(false)
  contentModified Boolean           @default(false)
  
  // Metadata
  ipAddress       String?
  userAgent       String?
  source          String?           // API, CRON, USER_REQUEST
  
  // Relations
  documentHash    DocumentHash      @relation(fields: [documentHashId], references: [id], onDelete: Cascade)
  verifiedByUser  User?            @relation(fields: [verifiedBy], references: [id])
  
  @@index([documentHashId])
  @@index([verifiedAt])
  @@index([isValid])
}

model DocumentAuditLog {
  id              Int               @id @default(autoincrement())
  documentHashId  String
  
  // Audit Details
  action          AuditAction       // CREATED, ISSUED, VIEWED, VERIFIED, ARCHIVED, DELETED
  performedBy     Int?              // User who performed action (null for system)
  performedAt     DateTime          @default(now())
  
  // Context
  ipAddress       String?
  userAgent       String?
  sessionId       String?
  source          String?           // WEB, API, SYSTEM, CRON
  
  // Change Details
  previousState   Json?             // Previous document state
  newState        Json?             // New document state
  changeReason    String?           // Reason for change
  
  // Metadata
  metadata        Json?             // Additional context data
  
  // Relations
  documentHash    DocumentHash      @relation(fields: [documentHashId], references: [id], onDelete: Cascade)
  user            User?            @relation(fields: [performedBy], references: [id])
  
  @@index([documentHashId])
  @@index([performedAt])
  @@index([action])
  @@index([performedBy])
}

// Polymorphic reference to any document type
model DocumentReference {
  type            DocumentType      @id
  id              Int               @id
  
  // Computed fields for easy querying
  title           String
  number          String
  status          String
  clientId        Int
  createdAt       DateTime
  updatedAt       DateTime
  
  hashes          DocumentHash[]
  
  @@unique([type, id])
  @@index([type, status])
  @@index([clientId])
}

model ComplianceSettings {
  id              Int               @id @default(autoincrement())
  accountId       Int               @unique
  
  // Retention Policies
  retentionYears  Int               @default(7)
  autoArchive     Boolean           @default(true)
  archiveAfterYears Int             @default(3)
  deleteAfterYears Int?             // null = never delete
  
  // Verification Settings
  autoVerifyOnIssue Boolean         @default(true)
  verifyFrequencyDays Int           @default(30)
  verifyOnAccess   Boolean          @default(true)
  
  // Storage Settings
  storageProvider  String           @default("LOCAL") // LOCAL, S3, AZURE
  encryptionKey    String?          // Encrypted storage key
  compressionLevel Int              @default(6)
  
  // Audit Settings
  auditLevel      AuditLevel        @default(STANDARD)
  logRetentionDays Int              @default(2555) // 7 years
  alertOnFailure  Boolean           @default(true)
  
  // Compliance Standards
  complianceStandards Json          // Array of standards: SOX, GAAP, etc.
  
  createdAt       DateTime          @default(now())
  updatedAt       DateTime          @updatedAt
  
  // Relations
  account         Account           @relation(fields: [accountId], references: [id], onDelete: Cascade)
  
  @@index([accountId])
}

enum DocumentType {
  QUOTE
  INVOICE
  CHANGE_ORDER
}

enum VerificationStatus {
  PENDING
  VALID
  INVALID
  ERROR
  EXPIRED
}

enum VerificationMethod {
  AUTOMATIC
  MANUAL
  EXTERNAL
  SCHEDULED
}

enum AuditAction {
  CREATED
  ISSUED
  VIEWED
  DOWNLOADED
  VERIFIED
  ARCHIVED
  DELETED
  MODIFIED
  STATUS_CHANGED
  PERMISSIONS_CHANGED
}

enum AuditLevel {
  MINIMAL     // Only critical actions
  STANDARD    // Normal business actions
  DETAILED    // All user interactions
  FORENSIC    // Maximum detail for compliance
}
```

### 3.2 TypeScript Interfaces

```typescript
export interface DocumentHash {
  id: string;
  documentType: DocumentType;
  documentId: string;
  version: number;
  
  // Hash data
  sha256Hash: string;
  pdfSize: number;
  pdfChecksum: string;
  
  // Content
  contentType: string;
  filename: string;
  storageUrl?: string;
  
  // Legal metadata
  issuedAt: string;
  issuedBy: string;
  issuedStatus: string;
  
  // Verification
  verifiedAt?: string;
  verificationStatus: VerificationStatus;
  verificationErrors?: any;
  
  // Retention
  retentionDate: string;
  archivedAt?: string;
  archivedBy?: string;
  
  createdAt: string;
  updatedAt: string;
}

export interface HashVerification {
  id: string;
  documentHashId: string;
  verifiedAt: string;
  verifiedBy?: string;
  verificationMethod: VerificationMethod;
  
  // Results
  isValid: boolean;
  originalHash: string;
  currentHash: string;
  sizeMismatch: boolean;
  contentModified: boolean;
  
  // Context
  ipAddress?: string;
  userAgent?: string;
  source?: string;
}

export interface DocumentAuditLog {
  id: string;
  documentHashId: string;
  action: AuditAction;
  performedBy?: string;
  performedAt: string;
  
  // Context
  ipAddress?: string;
  userAgent?: string;
  sessionId?: string;
  source?: string;
  
  // Changes
  previousState?: any;
  newState?: any;
  changeReason?: string;
  metadata?: any;
}

export interface ComplianceSettings {
  id: string;
  accountId: string;
  
  // Retention
  retentionYears: number;
  autoArchive: boolean;
  archiveAfterYears: number;
  deleteAfterYears?: number;
  
  // Verification
  autoVerifyOnIssue: boolean;
  verifyFrequencyDays: number;
  verifyOnAccess: boolean;
  
  // Storage
  storageProvider: string;
  encryptionKey?: string;
  compressionLevel: number;
  
  // Audit
  auditLevel: AuditLevel;
  logRetentionDays: number;
  alertOnFailure: boolean;
  
  complianceStandards: string[];
  
  createdAt: string;
  updatedAt: string;
}

export interface DocumentIntegrityReport {
  documentId: string;
  documentType: DocumentType;
  currentVersion: number;
  totalVersions: number;
  
  // Status
  integrityStatus: 'VALID' | 'COMPROMISED' | 'MISSING' | 'ERROR';
  lastVerified: string;
  nextVerificationDue: string;
  
  // Issues
  issues: DocumentIntegrityIssue[];
  
  // Compliance
  complianceStatus: 'COMPLIANT' | 'NON_COMPLIANT' | 'WARNING';
  retentionExpiry: string;
  archivalStatus: 'ACTIVE' | 'ARCHIVED' | 'SCHEDULED';
}

export interface DocumentIntegrityIssue {
  type: 'HASH_MISMATCH' | 'MISSING_FILE' | 'SIZE_MISMATCH' | 'CORRUPTED';
  severity: 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL';
  description: string;
  detectedAt: string;
  resolvedAt?: string;
}
```

---

## 4. API Endpoints

### 4.1 Document Hashing & Verification

```typescript
// Generate hash for document (internal)
POST /api/documents/hash
Body: {
  documentType: DocumentType,
  documentId: string,
  pdfBuffer: Buffer,
  issuedBy: string
}
Response: DocumentHash

// Verify document integrity
POST /api/documents/verify/:documentHashId
Body: { 
  verificationMethod?: VerificationMethod,
  source?: string
}
Response: HashVerification

// Get document hash details
GET /api/documents/hash/:documentHashId
Response: DocumentHash

// Verify document by upload (for external verification)
POST /api/documents/verify-upload
Body: FormData with PDF file
Response: {
  originalHash: string,
  uploadedHash: string,
  isValid: boolean,
  verification: HashVerification
}
```

### 4.2 Audit & Compliance

```typescript
// Get document audit trail
GET /api/documents/audit/:documentHashId
Query: { 
  action?: AuditAction,
  fromDate?: string,
  toDate?: string,
  performedBy?: string
}
Response: DocumentAuditLog[]

// Get compliance report for account
GET /api/compliance/report
Query: {
  documentType?: DocumentType,
  dateRange?: { from: string, to: string },
  status?: string
}
Response: {
  summary: ComplianceSummary,
  documents: DocumentIntegrityReport[]
}

// Get compliance settings
GET /api/compliance/settings
Response: ComplianceSettings

// Update compliance settings
PUT /api/compliance/settings
Body: Partial<ComplianceSettings>
Response: ComplianceSettings

// Bulk verification
POST /api/compliance/verify-bulk
Body: {
  documentType?: DocumentType,
  dateRange?: { from: string, to: string },
  maxConcurrent?: number
}
Response: {
  jobId: string,
  totalDocuments: number,
  estimatedDuration: string
}

// Get bulk verification status
GET /api/compliance/verify-bulk/:jobId
Response: {
  jobId: string,
  status: 'RUNNING' | 'COMPLETED' | 'FAILED',
  progress: {
    total: number,
    completed: number,
    failed: number
  },
  results?: HashVerification[]
}
```

### 4.3 Document Lifecycle

```typescript
// Issue document (makes it immutable)
POST /api/documents/:documentType/:documentId/issue
Body: { 
  issuedBy: string,
  generatePdf?: boolean
}
Response: {
  documentHash: DocumentHash,
  pdfUrl: string
}

// Archive document
POST /api/documents/hash/:documentHashId/archive
Body: {
  archivedBy: string,
  reason?: string
}
Response: DocumentHash

// Get document versions
GET /api/documents/:documentType/:documentId/versions
Response: DocumentHash[]

// Export compliance package
GET /api/compliance/export
Query: {
  documentType?: DocumentType,
  dateRange: { from: string, to: string },
  format: 'ZIP' | 'TAR' | 'JSON'
}
Response: Binary archive with documents and audit trails
```

---

## 5. Core Services

### 5.1 Document Hashing Service

```typescript
export class DocumentHashingService {
  async hashDocument(
    documentType: DocumentType,
    documentId: string,
    pdfBuffer: Buffer,
    issuedBy: string
  ): Promise<DocumentHash> {
    // Generate SHA-256 hash
    const sha256Hash = crypto
      .createHash('sha256')
      .update(pdfBuffer)  
      .digest('hex');
    
    // Generate MD5 checksum for additional verification
    const pdfChecksum = crypto
      .createHash('md5')
      .update(pdfBuffer)
      .digest('hex');
    
    // Get compliance settings
    const settings = await this.getComplianceSettings();
    const retentionDate = addYears(new Date(), settings.retentionYears);
    
    // Store in database
    const documentHash = await prisma.documentHash.create({
      data: {
        documentType,
        documentId: parseInt(documentId),
        sha256Hash,
        pdfSize: pdfBuffer.length,
        pdfChecksum,
        contentType: 'application/pdf',
        filename: `${documentType.toLowerCase()}-${documentId}.pdf`,
        issuedAt: new Date(),
        issuedBy: parseInt(issuedBy),
        issuedStatus: 'ISSUED',
        retentionDate,
        verificationStatus: 'PENDING'
      }
    });
    
    // Store PDF file
    await this.storePdf(documentHash.id, pdfBuffer, settings);
    
    // Schedule automatic verification
    if (settings.autoVerifyOnIssue) {
      await this.scheduleVerification(documentHash.id);
    }
    
    // Log audit event
    await this.logAuditEvent(documentHash.id, 'CREATED', issuedBy);
    
    return documentHash;
  }
  
  async verifyDocument(
    documentHashId: string,
    method: VerificationMethod = 'AUTOMATIC',
    verifiedBy?: string
  ): Promise<HashVerification> {
    const documentHash = await this.getDocumentHash(documentHashId);
    
    // Retrieve stored PDF
    const pdfBuffer = await this.retrievePdf(documentHash.id);
    
    // Compute current hash
    const currentHash = crypto
      .createHash('sha256')
      .update(pdfBuffer)
      .digest('hex');
    
    // Compare with original  
    const isValid = currentHash === documentHash.sha256Hash;
    const sizeMismatch = pdfBuffer.length !== documentHash.pdfSize;
    
    // Create verification record
    const verification = await prisma.hashVerification.create({
      data: {
        documentHashId,
        verificationMethod: method,
        verifiedBy: verifiedBy ? parseInt(verifiedBy) : null,
        isValid,
        originalHash: documentHash.sha256Hash,
        currentHash,
        sizeMismatch,
        contentModified: !isValid || sizeMismatch
      }
    });
    
    // Update document hash status
    await prisma.documentHash.update({
      where: { id: documentHashId },
      data: {
        verifiedAt: new Date(),
        verificationStatus: isValid ? 'VALID' : 'INVALID'
      }
    });
    
    // Log audit event
    await this.logAuditEvent(
      documentHashId, 
      'VERIFIED', 
      verifiedBy, 
      { verificationResult: isValid }
    );
    
    // Alert if verification failed
    if (!isValid) {
      await this.alertVerificationFailure(documentHash, verification);
    }
    
    return verification;
  }
  
  private async storePdf(
    documentHashId: string,
    pdfBuffer: Buffer,
    settings: ComplianceSettings
  ): Promise<void> {
    switch (settings.storageProvider) {
      case 'S3':
        await this.storeToS3(documentHashId, pdfBuffer, settings);
        break;
      case 'AZURE':
        await this.storeToAzure(documentHashId, pdfBuffer, settings);
        break;
      default:
        await this.storeLocally(documentHashId, pdfBuffer, settings);
    }
  }
  
  private async storeLocally(
    documentHashId: string,
    pdfBuffer: Buffer,
    settings: ComplianceSettings
  ): Promise<void> {
    const storageDir = path.join(
      process.env.DOCUMENT_STORAGE_PATH || './storage/documents',
      documentHashId.substring(0, 2), // Distribute across subdirectories
      documentHashId.substring(2, 4)
    );
    
    await fs.ensureDir(storageDir);
    
    const filePath = path.join(storageDir, `${documentHashId}.pdf`);
    
    // Encrypt if encryption key provided
    let finalBuffer = pdfBuffer;
    if (settings.encryptionKey) {
      finalBuffer = await this.encryptBuffer(pdfBuffer, settings.encryptionKey);
    }
    
    await fs.writeFile(filePath, finalBuffer);
  }
}
```

### 5.2 Compliance Monitoring Service

```typescript
export class ComplianceMonitoringService {
  async generateComplianceReport(
    accountId: string,
    options: ComplianceReportOptions = {}
  ): Promise<ComplianceReport> {
    const settings = await this.getComplianceSettings(accountId);
    
    // Get all documents in scope
    const documents = await this.getDocumentsInScope(accountId, options);
    
    const reports: DocumentIntegrityReport[] = [];
    const summary = {
      totalDocuments: documents.length,
      compliantDocuments: 0,
      nonCompliantDocuments: 0,
      warningDocuments: 0,
      integrityIssues: 0,
      retentionIssues: 0
    };
    
    for (const doc of documents) {
      const report = await this.generateDocumentIntegrityReport(doc);
      reports.push(report);
      
      // Update summary
      switch (report.complianceStatus) {
        case 'COMPLIANT':
          summary.compliantDocuments++;
          break;
        case 'NON_COMPLIANT':
          summary.nonCompliantDocuments++;
          break;
        case 'WARNING':
          summary.warningDocuments++;
          break;
      }
      
      summary.integrityIssues += report.issues.filter(i => 
        ['HASH_MISMATCH', 'MISSING_FILE', 'CORRUPTED'].includes(i.type)
      ).length;
    }
    
    return {
      generatedAt: new Date().toISOString(),
      accountId,
      summary,
      documents: reports,
      settings
    };
  }
  
  async generateDocumentIntegrityReport(
    documentHash: DocumentHash
  ): Promise<DocumentIntegrityReport> {
    const issues: DocumentIntegrityIssue[] = [];
    
    // Check if document exists and is accessible
    try {
      const pdfBuffer = await this.retrievePdf(documentHash.id);
      
      // Verify hash
      const currentHash = crypto
        .createHash('sha256')
        .update(pdfBuffer)
        .digest('hex');
      
      if (currentHash !== documentHash.sha256Hash) {
        issues.push({
          type: 'HASH_MISMATCH',
          severity: 'CRITICAL',
          description: 'Document hash does not match original',
          detectedAt: new Date().toISOString()
        });
      }
      
      // Check size
      if (pdfBuffer.length !== documentHash.pdfSize) {
        issues.push({
          type: 'SIZE_MISMATCH',
          severity: 'HIGH',
          description: `Document size mismatch: expected ${documentHash.pdfSize}, got ${pdfBuffer.length}`,
          detectedAt: new Date().toISOString()
        });
      }
      
    } catch (error) {
      issues.push({
        type: 'MISSING_FILE',
        severity: 'CRITICAL',
        description: 'Document file not accessible',
        detectedAt: new Date().toISOString()
      });
    }
    
    // Check retention compliance
    const now = new Date();
    const retentionExpiry = new Date(documentHash.retentionDate);
    const isRetentionExpired = now > retentionExpiry;
    
    // Determine compliance status
    let complianceStatus: 'COMPLIANT' | 'NON_COMPLIANT' | 'WARNING' = 'COMPLIANT';
    
    const criticalIssues = issues.filter(i => i.severity === 'CRITICAL');
    const highIssues = issues.filter(i => i.severity === 'HIGH');
    
    if (criticalIssues.length > 0) {
      complianceStatus = 'NON_COMPLIANT';
    } else if (highIssues.length > 0 || isRetentionExpired) {
      complianceStatus = 'WARNING';
    }
    
    // Calculate next verification due
    const lastVerified = new Date(documentHash.verifiedAt || documentHash.createdAt);
    const nextVerificationDue = addDays(lastVerified, 30); // Default 30 days
    
    return {
      documentId: documentHash.documentId.toString(),
      documentType: documentHash.documentType,
      currentVersion: documentHash.version,
      totalVersions: await this.countDocumentVersions(documentHash),
      integrityStatus: issues.length > 0 ? 'COMPROMISED' : 'VALID',
      lastVerified: documentHash.verifiedAt || documentHash.createdAt,
      nextVerificationDue: nextVerificationDue.toISOString(),
      issues,
      complianceStatus,
      retentionExpiry: documentHash.retentionDate,
      archivalStatus: documentHash.archivedAt ? 'ARCHIVED' : 'ACTIVE'
    };
  }
  
  async scheduleRetentionCleanup(accountId: string): Promise<void> {
    const settings = await this.getComplianceSettings(accountId);
    
    if (!settings.autoArchive) return;
    
    // Find documents ready for archival
    const documentsToArchive = await prisma.documentHash.findMany({
      where: {
        issuedAt: {
          lte: subYears(new Date(), settings.archiveAfterYears)
        },
        archivedAt: null,
        // Only archive documents that are still valid
        verificationStatus: 'VALID'
      }
    });
    
    for (const doc of documentsToArchive) {
      await this.archiveDocument(doc.id, 'SYSTEM');
    }
    
    // Find documents ready for deletion (if enabled)
    if (settings.deleteAfterYears) {
      const documentsToDelete = await prisma.documentHash.findMany({
        where: {
          issuedAt: {
            lte: subYears(new Date(), settings.deleteAfterYears)
          },
          archivedAt: {
            not: null
          }
        }
      });
      
      for (const doc of documentsToDelete) {
        await this.deleteDocument(doc.id, 'SYSTEM');
      }
    }
  }
}
```

---

## 6. Integration with Existing System

### 6.1 Quote Integration

```typescript
// Update existing QuoteService
export class QuoteService {
  async updateQuoteStatus(
    quoteId: number, 
    userId: number, 
    status: QuoteStatus,
    notes?: string
  ): Promise<Quote> {
    const quote = await super.updateQuoteStatus(quoteId, userId, status, notes);
    
    // If quote is being sent or accepted, generate immutable hash
    if (status === 'SENT' || status === 'ACCEPTED') {
      await this.makeQuoteImmutable(quote, userId);
    }
    
    return quote;
  }
  
  private async makeQuoteImmutable(quote: Quote, userId: number): Promise<void> {
    // Generate PDF
    const pdfBuffer = await pdfService.generateQuotePDF(quote.id);
    
    // Create document hash
    await documentHashingService.hashDocument(
      'QUOTE',
      quote.id.toString(),
      pdfBuffer,
      userId.toString()
    );
    
    console.log(`Quote ${quote.number} made immutable with hash verification`);
  }
}
```

### 6.2 Invoice Integration

```typescript
// Update existing InvoiceService  
export class InvoiceService {
  async updateInvoiceStatus(
    invoiceId: string,
    status: InvoiceStatus
  ): Promise<Invoice> {
    const invoice = await super.updateInvoiceStatus(invoiceId, status);
    
    // If invoice is being sent, make it immutable
    if (status === 'SENT') {
      await this.makeInvoiceImmutable(invoice);
    }
    
    return invoice;
  }
  
  private async makeInvoiceImmutable(invoice: Invoice): Promise<void> {
    // Generate PDF
    const pdfBuffer = await pdfService.generateInvoicePDF(invoice.id);
    
    // Create document hash
    await documentHashingService.hashDocument(
      'INVOICE',
      invoice.id,
      pdfBuffer,
      'SYSTEM' // System-issued for invoices
    );
    
    console.log(`Invoice ${invoice.number} made immutable with hash verification`);
  }
}
```

### 6.3 PDF Service Integration

```typescript
// Update PDFService to support hashing
export class PDFService {
  async generateQuotePDF(quoteId: number, withHash: boolean = false): Promise<Buffer> {
    const pdfBuffer = await super.generateQuotePDF(quoteId);
    
    if (withHash) {
      // Add hash metadata to PDF (optional)
      const hash = crypto.createHash('sha256').update(pdfBuffer).digest('hex');
      return this.embedHashInPdf(pdfBuffer, hash);
    }
    
    return pdfBuffer;
  }
  
  private embedHashInPdf(pdfBuffer: Buffer, hash: string): Buffer {
    // Use PDF-lib or similar to embed hash in PDF metadata
    // This makes the hash visible in PDF properties
    return pdfBuffer; // Simplified for example
  }
}
```

---

## 7. User Interface Components

### 7.1 Document Integrity Dashboard

```typescript
interface DocumentIntegrityDashboardProps {
  accountId: string;
}

export function DocumentIntegrityDashboard({ accountId }: DocumentIntegrityDashboardProps) {
  const { data: report, isLoading } = useQuery({
    queryKey: ['compliance-report', accountId],
    queryFn: () => getComplianceReport(accountId)
  });

  if (isLoading) return <DashboardSkeleton />;

  return (
    <Container maxWidth="xl">
      <Typography variant="h4" gutterBottom>
        Document Integrity & Compliance
      </Typography>

      {/* Summary Cards */}
      <Grid container spacing={3} sx={{ mb: 4 }}>
        <Grid item xs={12} sm={6} md={3}>
          <MetricCard
            title="Total Documents"
            value={report.summary.totalDocuments}
            icon={<Description />}
            color="primary"
          />
        </Grid>
        <Grid item xs={12} sm={6} md={3}>
          <MetricCard
            title="Compliant"
            value={report.summary.compliantDocuments}
            icon={<CheckCircle />}
            color="success"
          />
        </Grid>
        <Grid item xs={12} sm={6} md={3}>
          <MetricCard
            title="Warnings"
            value={report.summary.warningDocuments}
            icon={<Warning />}
            color="warning"
          />
        </Grid>
        <Grid item xs={12} sm={6} md={3}>
          <MetricCard
            title="Non-Compliant"
            value={report.summary.nonCompliantDocuments}
            icon={<Error />}
            color="error"
          />
        </Grid>
      </Grid>

      {/* Compliance Status Chart */}
      <Card sx={{ mb: 4 }}>
        <CardContent>
          <Typography variant="h6" gutterBottom>
            Compliance Status Overview
          </Typography>
          <Box sx={{ height: 300 }}>
            <PieChart
              data={[
                { name: 'Compliant', value: report.summary.compliantDocuments, color: '#4caf50' },
                { name: 'Warning', value: report.summary.warningDocuments, color: '#ff9800' },
                { name: 'Non-Compliant', value: report.summary.nonCompliantDocuments, color: '#f44336' }
              ]}
            />
          </Box>
        </CardContent>
      </Card>

      {/* Document List with Issues */}
      <Card>
        <CardContent>
          <Box sx={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center', mb: 2 }}>
            <Typography variant="h6">
              Document Integrity Status
            </Typography>
            <Button
              variant="contained"
              startIcon={<Refresh />}
              onClick={() => runBulkVerification()}
            >
              Verify All Documents
            </Button>
          </Box>

          <DocumentIntegrityTable 
            documents={report.documents}
            onVerify={(doc) => verifyDocument(doc.id)}
            onViewAuditTrail={(doc) => setSelectedDocumentForAudit(doc)}
          />
        </CardContent>
      </Card>

      {/* Audit Trail Modal */}
      <DocumentAuditTrailModal
        open={!!selectedDocumentForAudit}
        document={selectedDocumentForAudit}
        onClose={() => setSelectedDocumentForAudit(null)}
      />
    </Container>
  );
}
```

### 7.2 Document Verification Component

```typescript
interface DocumentVerificationProps {
  documentHash: DocumentHash;
  onVerificationComplete: (result: HashVerification) => void;
}

export function DocumentVerification({ documentHash, onVerificationComplete }: DocumentVerificationProps) {
  const [isVerifying, setIsVerifying] = useState(false);
  const [verificationResult, setVerificationResult] = useState<HashVerification | null>(null);

  const handleVerify = async () => {
    setIsVerifying(true);
    try {
      const result = await verifyDocument(documentHash.id, 'MANUAL');
      setVerificationResult(result);
      onVerificationComplete(result);
    } catch (error) {
      toast.error('Verification failed: ' + error.message);
    } finally {
      setIsVerifying(false);
    }
  };

  const getStatusColor = (status: VerificationStatus) => {
    switch (status) {
      case 'VALID': return 'success';
      case 'INVALID': return 'error';
      case 'PENDING': return 'warning';
      default: return 'default';
    }
  };

  return (
    <Card>
      <CardContent>
        <Box sx={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center', mb: 3 }}>
          <Typography variant="h6">
            Document Verification
          </Typography>
          <Chip
            label={documentHash.verificationStatus}
            color={getStatusColor(documentHash.verificationStatus)}
            icon={
              documentHash.verificationStatus === 'VALID' ? <CheckCircle /> :
              documentHash.verificationStatus === 'INVALID' ? <Error /> :
              <Schedule />
            }
          />
        </Box>

        {/* Document Details */}
        <Grid container spacing={2} sx={{ mb: 3 }}>
          <Grid item xs={12} sm={6}>
            <Typography variant="body2" color="text.secondary">
              Document Type
            </Typography>
            <Typography variant="body1" sx={{ fontWeight: 'medium' }}>
              {documentHash.documentType}
            </Typography>
          </Grid>
          <Grid item xs={12} sm={6}>
            <Typography variant="body2" color="text.secondary">
              Document ID
            </Typography>
            <Typography variant="body1" sx={{ fontWeight: 'medium' }}>
              {documentHash.documentId}
            </Typography>
          </Grid>
          <Grid item xs={12} sm={6}>
            <Typography variant="body2" color="text.secondary">
              SHA-256 Hash
            </Typography>
            <Typography 
              variant="body2" 
              sx={{ 
                fontFamily: 'monospace',
                wordBreak: 'break-all',
                fontSize: '0.75rem'
              }}
            >
              {documentHash.sha256Hash}
            </Typography>
          </Grid>
          <Grid item xs={12} sm={6}>
            <Typography variant="body2" color="text.secondary">
              File Size
            </Typography>
            <Typography variant="body1">
              {formatBytes(documentHash.pdfSize)}
            </Typography>
          </Grid>
          <Grid item xs={12} sm={6}>
            <Typography variant="body2" color="text.secondary">
              Issued Date
            </Typography>
            <Typography variant="body1">
              {format(new Date(documentHash.issuedAt), 'PPP')}
            </Typography>
          </Grid>
          <Grid item xs={12} sm={6}>
            <Typography variant="body2" color="text.secondary">
              Last Verified
            </Typography>
            <Typography variant="body1">
              {documentHash.verifiedAt 
                ? format(new Date(documentHash.verifiedAt), 'PPP')
                : 'Never'
              }
            </Typography>
          </Grid>
        </Grid>

        {/* Verification Results */}
        {verificationResult && (
          <Alert 
            severity={verificationResult.isValid ? 'success' : 'error'}
            sx={{ mb: 3 }}
          >
            <Typography variant="subtitle2" gutterBottom>
              Verification Result
            </Typography>
            <Typography variant="body2">
              {verificationResult.isValid 
                ? 'Document integrity verified successfully'
                : 'Document integrity verification failed'
              }
            </Typography>
            {!verificationResult.isValid && (
              <Box sx={{ mt: 1 }}>
                <Typography variant="body2">
                  <strong>Original Hash:</strong> {verificationResult.originalHash}
                </Typography>
                <Typography variant="body2">
                  <strong>Current Hash:</strong> {verificationResult.currentHash}
                </Typography>
                {verificationResult.sizeMismatch && (
                  <Typography variant="body2" color="error">
                    ⚠️ File size mismatch detected
                  </Typography>
                )}
              </Box>
            )}
          </Alert>
        )}

        {/* Actions */}
        <Box sx={{ display: 'flex', gap: 2 }}>
          <Button
            variant="contained"
            onClick={handleVerify}
            disabled={isVerifying}
            startIcon={isVerifying ? <CircularProgress size={20} /> : <Verified />}
          >
            {isVerifying ? 'Verifying...' : 'Verify Now'}
          </Button>
          
          <Button
            variant="outlined"
            onClick={() => downloadDocument(documentHash.id)}
            startIcon={<Download />}
          >
            Download PDF
          </Button>
          
          <Button
            variant="outlined"
            onClick={() => viewAuditTrail(documentHash.id)}
            startIcon={<History />}
          >
            Audit Trail
          </Button>
        </Box>
      </CardContent>
    </Card>
  );
}
```

### 7.3 Bulk Verification Interface

```typescript
export function BulkVerificationDialog({ open, onClose, accountId }: BulkVerificationDialogProps) {
  const [verificationJob, setVerificationJob] = useState<BulkVerificationJob | null>(null);
  const [filters, setFilters] = useState({
    documentType: '',
    dateRange: { from: '', to: '' },
    status: ''
  });

  const startBulkVerification = async () => {
    try {
      const job = await startBulkVerification(accountId, filters);
      setVerificationJob(job);
      
      // Poll for progress
      const interval = setInterval(async () => {
        const updatedJob = await getBulkVerificationStatus(job.jobId);
        setVerificationJob(updatedJob);
        
        if (updatedJob.status === 'COMPLETED' || updatedJob.status === 'FAILED') {
          clearInterval(interval);
        }
      }, 2000);
      
    } catch (error) {
      toast.error('Failed to start bulk verification');
    }
  };

  return (
    <Dialog open={open} onClose={onClose} maxWidth="md" fullWidth>
      <DialogTitle>
        Bulk Document Verification
      </DialogTitle>
      
      <DialogContent>
        {!verificationJob ? (
          /* Setup Form */
          <Box>
            <Typography variant="body1" paragraph>
              Run verification on multiple documents at once. This will check the integrity 
              of all selected documents against their stored hashes.
            </Typography>

            <Grid container spacing={3}>
              <Grid item xs={12} sm={4}>
                <FormControl fullWidth>
                  <InputLabel>Document Type</InputLabel>
                  <Select
                    value={filters.documentType}
                    onChange={(e) => setFilters(prev => ({ 
                      ...prev, 
                      documentType: e.target.value 
                    }))}
                  >
                    <MenuItem value="">All Types</MenuItem>
                    <MenuItem value="QUOTE">Quotes</MenuItem>
                    <MenuItem value="INVOICE">Invoices</MenuItem>
                    <MenuItem value="CHANGE_ORDER">Change Orders</MenuItem>
                  </Select>
                </FormControl>
              </Grid>
              
              <Grid item xs={12} sm={4}>
                <TextField
                  fullWidth
                  type="date"
                  label="From Date"
                  value={filters.dateRange.from}
                  onChange={(e) => setFilters(prev => ({ 
                    ...prev, 
                    dateRange: { ...prev.dateRange, from: e.target.value }
                  }))}
                  InputLabelProps={{ shrink: true }}
                />
              </Grid>
              
              <Grid item xs={12} sm={4}>
                <TextField
                  fullWidth
                  type="date"
                  label="To Date"
                  value={filters.dateRange.to}
                  onChange={(e) => setFilters(prev => ({ 
                    ...prev, 
                    dateRange: { ...prev.dateRange, to: e.target.value }
                  }))}
                  InputLabelProps={{ shrink: true }}
                />
              </Grid>
            </Grid>

            <Alert severity="info" sx={{ mt: 3 }}>
              This operation may take several minutes depending on the number of documents.
              You can close this dialog and the verification will continue in the background.
            </Alert>
          </Box>
        ) : (
          /* Progress Display */
          <Box>
            <Typography variant="h6" gutterBottom>
              Verification Progress
            </Typography>
            
            <Box sx={{ mb: 3 }}>
              <Box sx={{ display: 'flex', justifyContent: 'space-between', mb: 1 }}>
                <Typography variant="body2">
                  Status: {verificationJob.status}
                </Typography>
                <Typography variant="body2">
                  {verificationJob.progress.completed} / {verificationJob.progress.total}
                </Typography>
              </Box>
              
              <LinearProgress
                variant="determinate"
                value={(verificationJob.progress.completed / verificationJob.progress.total) * 100}
                sx={{ height: 8, borderRadius: 4 }}
              />
            </Box>

            {verificationJob.progress.failed > 0 && (
              <Alert severity="warning">
                {verificationJob.progress.failed} documents failed verification
              </Alert>
            )}

            {verificationJob.status === 'COMPLETED' && (
              <Alert severity="success">
                Bulk verification completed successfully!
              </Alert>
            )}

            {verificationJob.status === 'FAILED' && (
              <Alert severity="error">
                Bulk verification failed. Please try again or contact support.
              </Alert>
            )}
          </Box>
        )}
      </DialogContent>
      
      <DialogActions>
        <Button onClick={onClose}>
          {verificationJob?.status === 'COMPLETED' ? 'Close' : 'Cancel'}
        </Button>
        
        {!verificationJob && (
          <Button
            variant="contained"
            onClick={startBulkVerification}
          >
            Start Verification
          </Button>
        )}
        
        {verificationJob?.results && (
          <Button
            variant="outlined"
            onClick={() => downloadVerificationReport(verificationJob.jobId)}
          >
            Download Report
          </Button>
        )}
      </DialogActions>
    </Dialog>
  );
}
```

---

## 8. Implementation Plan

### Phase 1: Core Infrastructure (Week 1-2)
- [ ] Database schema migration for DocumentHash and related tables
- [ ] Basic hashing service with SHA-256 implementation
- [ ] Document storage service (local/S3/Azure)
- [ ] Core API endpoints for hash creation and verification
- [ ] Basic audit logging infrastructure

### Phase 2: Verification System (Week 2-3)
- [ ] Manual document verification API and UI
- [ ] Automated verification scheduling
- [ ] Bulk verification system with job queuing
- [ ] Verification failure alerting and notifications
- [ ] Integration with existing PDF generation

### Phase 3: Compliance Framework (Week 3-4)
- [ ] Compliance settings management
- [ ] Document retention policy enforcement
- [ ] Compliance reporting dashboard
- [ ] Document archival system
- [ ] Export functionality for compliance packages

### Phase 4: System Integration (Week 4-5)
- [ ] Integration with Quote service (immutable on send/accept)
- [ ] Integration with Invoice service (immutable on send)
- [ ] Integration with Change Order service
- [ ] Update PDF services to support hash embedding
- [ ] Audit trail integration across all document types

### Phase 5: Advanced Features (Week 5-6)
- [ ] Advanced compliance reporting and analytics
- [ ] Document integrity monitoring dashboard
- [ ] Automated retention cleanup and archival
- [ ] External verification API for third parties
- [ ] Performance optimization for large document sets

### Phase 6: Testing & Hardening (Week 6-7)
- [ ] Security audit and penetration testing
- [ ] Performance testing with large document volumes
- [ ] Compliance verification against industry standards
- [ ] Disaster recovery testing
- [ ] Documentation and user training

---

## 9. Security Considerations

### 9.1 Cryptographic Security
- **SHA-256 Standard**: Industry-standard cryptographic hashing
- **Key Management**: Secure storage of encryption keys using AWS KMS/Azure Key Vault
- **Hash Salting**: Optional salting for additional security
- **Algorithm Agility**: Support for upgrading to newer hash algorithms (SHA-3)

### 9.2 Storage Security
- **Encryption at Rest**: All stored documents encrypted with AES-256
- **Access Controls**: Role-based access to document storage
- **Secure Transmission**: TLS 1.3 for all data in transit
- **Backup Security**: Encrypted backups with separate key management

### 9.3 Audit Security
- **Tamper-Evident Logs**: Audit logs protected against modification
- **Log Integrity**: Hash chains for audit log verification
- **Access Logging**: All access to documents and hashes logged
- **Retention Protection**: Audit logs protected for compliance periods

---

## 10. Success Criteria

### Functional Requirements
- [ ] All issued documents automatically hashed and verified
- [ ] Document integrity verified on access and scheduled intervals
- [ ] Complete audit trail for all document lifecycle events
- [ ] Compliance reports generated for regulatory requirements
- [ ] Automated retention and archival policies enforced
- [ ] External verification capability for auditors

### Technical Requirements
- [ ] Hash generation and verification < 1 second per document
- [ ] 99.99% hash verification accuracy
- [ ] Support for 10,000+ documents with good performance
- [ ] Secure storage with encryption at rest and in transit
- [ ] Comprehensive API for integration and external access
- [ ] Automated backup and disaster recovery

### Compliance Requirements
- [ ] SOX compliance for financial document retention
- [ ] GAAP compliance for accounting records
- [ ] Industry-specific retention requirements met
- [ ] Audit trail suitable for regulatory examination
- [ ] Data protection compliance (GDPR/CCPA)
- [ ] Legal admissibility of digital documents

---

**File Version:** 1.0 – Document Immutability & Legal Compliance Specification  
**Last Updated:** October 12, 2025