# REG001 – UML Domain Structure

> **Service Code:** REG001  
> **Service Name (EN):** Issuing Temporary Registration Certificate for a Ship or Marine Unit  
> **Service Name (AR):** إصدار شهادة تسجيل سفينة أو وحدة بحرية مؤقتة  
> **Document Type:** Domain Model – UML Structure  
> **Version:** 1.0  
> **Last Updated:** 2026-01-08

---

## 1. Purpose / الغرض

This document defines the **domain model structure** for REG001, including:
- Aggregates and Aggregate Roots
- Entities within each aggregate
- Value Objects
- Domain Events
- Conceptual class diagrams

The model is **technology-agnostic** and focuses on business concepts derived from the BPMN processes, capabilities, and business rules.

---

## 2. Domain Model Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     REG001 DOMAIN MODEL OVERVIEW                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                 REGISTRATION REQUEST (Core Aggregate)                │   │
│   │   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                 │   │
│   │   │  Applicant  │  │   Vessel    │  │  Documents  │                 │   │
│   │   │  Snapshot   │  │  Snapshot   │  │   Bundle    │                 │   │
│   │   └─────────────┘  └─────────────┘  └─────────────┘                 │   │
│   │   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                 │   │
│   │   │  Approval   │  │   Payment   │  │ Certificate │                 │   │
│   │   │   Status    │  │   Record    │  │   Issuance  │                 │   │
│   │   └─────────────┘  └─────────────┘  └─────────────┘                 │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│   │    Vessel    │  │  Applicant   │  │   Invoice    │  │ Certificate  │    │
│   │   Profile    │  │   Profile    │  │   Aggregate  │  │  Aggregate   │    │
│   │  (Separate)  │  │  (Separate)  │  │              │  │              │    │
│   └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Aggregates

### 3.1 Registration Request Aggregate (Core)

**Aggregate Root:** `RegistrationRequest`

This is the **central aggregate** that orchestrates the temporary registration process. It maintains the lifecycle state of a registration request from initiation through certificate issuance.

| Property | Type | Description |
|----------|------|-------------|
| `requestId` | RequestId (VO) | Unique identifier for the registration request |
| `requestNumber` | String | Human-readable request number (e.g., REG001-2026-00001) |
| `status` | RequestStatus (Enum) | Current workflow state |
| `applicantSnapshot` | ApplicantSnapshot (Entity) | Captured applicant data at submission |
| `vesselSnapshot` | VesselSnapshot (Entity) | Captured vessel data at submission |
| `documentsBundle` | DocumentsBundle (Entity) | Required and uploaded documents |
| `approvalStatus` | ApprovalStatus (Entity) | External approval states (MAFWR, Inspection, Customs) |
| `paymentRecord` | PaymentRecord (Entity) | Fee and payment information |
| `certificateIssuance` | CertificateIssuance (Entity) | Issued certificate details |
| `nameReservation` | NameReservation (VO) | Reserved vessel name |
| `penalties` | List<PenaltyLineItem> (VO) | Applied penalties |
| `createdAt` | DateTime | Request creation timestamp |
| `updatedAt` | DateTime | Last update timestamp |
| `submittedAt` | DateTime | Submission timestamp |
| `issuedAt` | DateTime | Certificate issuance timestamp |

**Invariants:**
- A request cannot proceed to payment without complete documents
- A request cannot be issued without successful payment
- Fishing vessels require MAFWR approval before payment
- Vessel name must be reserved before payment initiation

---

### 3.2 Vessel Profile Aggregate (Reference Data)

**Aggregate Root:** `VesselProfile`

Master vessel data that exists beyond the registration request lifecycle.

| Property | Type | Description |
|----------|------|-------------|
| `vesselId` | VesselId (VO) | Unique vessel identifier |
| `imoNumber` | String | IMO number (if applicable) |
| `vesselName` | VesselName (VO) | Current vessel name |
| `vesselCategory` | VesselCategory (Enum) | Vessel type classification |
| `buildMaterial` | BuildMaterial (Enum) | Hull construction material |
| `buildYear` | Year | Year of construction |
| `constructionDates` | ConstructionPeriod (VO) | Start/end construction dates |
| `dimensions` | VesselDimensions (VO) | Length, beam, draft, GT, NT |
| `acquisitionPath` | AcquisitionPath (Enum) | NEW_BUILD, PURCHASE, FLAG_CHANGE |
| `ownershipHistory` | List<OwnershipRecord> | Ownership chain |
| `flagHistory` | List<FlagRecord> | Flag state history |
| `activityType` | ActivityType (Enum) | Intended activity classification |
| `registrationPort` | PortReference (VO) | Associated registration port (REG-001-SET-03) |
| `homePort` | PortReference (VO) | Vessel home port |
| `isProhibited` | Boolean | Whether vessel is on prohibited list |

**Invariants:**
- Vessel age must comply with policy limits based on material
- IMO number format must be valid (if provided)
- Dimensions must be positive values

---

### 3.3 Applicant Profile Aggregate (Reference Data)

**Aggregate Root:** `ApplicantProfile`

Master applicant data (individual or company) referenced by registration requests.

| Property | Type | Description |
|----------|------|-------------|
| `applicantId` | ApplicantId (VO) | Unique applicant identifier |
| `applicantType` | ApplicantType (Enum) | INDIVIDUAL or COMPANY |
| `individualProfile` | IndividualProfile (Entity) | Personal details (if individual) |
| `companyProfile` | CompanyProfile (Entity) | Company details (if company) |
| `contactInfo` | ContactInfo (VO) | Email, phone, address |
| `agents` | List<AgentRepresentation> (Entity) | Authorized agents (موكل من مالك السفينة) |
| `nationality` | String | Applicant nationality (Omani citizen / resident) |
| `isResident` | Boolean | Whether applicant is Omani resident |

**Invariants:**
- Applicant must be Omani citizen or resident (per DOCX conditions)
- Company must have valid Commercial Registration (CR)
- Company activity must be eligible for vessel type

#### 3.3.1 IndividualProfile Entity

| Property | Type | Description |
|----------|------|-------------|
| `nationalId` | String | Civil ID number (رقم البطاقة الشخصية) |
| `fullNameAr` | String | Full name in Arabic |
| `fullNameEn` | String | Full name in English |
| `dateOfBirth` | Date | Date of birth |
| `nationalityType` | NationalityType (Enum) | OMANI_CITIZEN, OMANI_RESIDENT, GCC_CITIZEN |
| `passportNumber` | String | Passport number (for residents) |
| `residencyExpiryDate` | Date | Residency permit expiry (for residents) |
| `ropVerificationStatus` | VerificationStatus (Enum) | Verified via ROP PKI |
| `verifiedAt` | DateTime | When identity was verified |

#### 3.3.2 CompanyProfile Entity

| Property | Type | Description |
|----------|------|-------------|
| `crNumber` | String | Commercial Registration number (سجل تجاري) |
| `companyNameAr` | String | Company name in Arabic |
| `companyNameEn` | String | Company name in English |
| `crExpiryDate` | Date | CR expiry date |
| `crStatus` | CRStatus (Enum) | ACTIVE, EXPIRED, SUSPENDED |
| `activityCodes` | List<String> | Activity codes from Invest Easy |
| `eligibleActivities` | List<String> | Activities eligible for vessel registration |
| `investEasyVerificationStatus` | VerificationStatus (Enum) | Verified via Invest Easy |
| `verifiedAt` | DateTime | When CR was verified |

#### 3.3.3 AgentRepresentation Entity

| Property | Type | Description |
|----------|------|-------------|
| `agentId` | AgentId (VO) | Unique agent identifier |
| `agentType` | AgentType (Enum) | POWER_OF_ATTORNEY, AUTHORIZED_REPRESENTATIVE |
| `agentNationalId` | String | Agent's civil ID |
| `agentName` | String | Agent's full name |
| `authorizationDocument` | DocumentReference (VO) | Power of attorney document |
| `authorizationValidFrom` | Date | Authorization start date |
| `authorizationValidTo` | Date | Authorization end date |
| `scope` | List<AuthorizationScope> | What actions agent can perform |

---

### 3.4 Invoice Aggregate

**Aggregate Root:** `Invoice`

Financial artifact for fee collection.

| Property | Type | Description |
|----------|------|-------------|
| `invoiceId` | InvoiceId (VO) | Unique invoice identifier |
| `invoiceNumber` | String | Human-readable invoice number |
| `requestReference` | RequestId (VO) | Associated registration request |
| `lineItems` | List<InvoiceLineItem> (Entity) | Fee and penalty line items |
| `totalAmount` | Money (VO) | Total amount due |
| `status` | InvoiceStatus (Enum) | DRAFT, ISSUED, PAID, CANCELLED |
| `issuedAt` | DateTime | Invoice issuance timestamp |
| `paidAt` | DateTime | Payment confirmation timestamp |
| `paymentReference` | String | Payment gateway reference |

**Invariants:**
- Invoice cannot be modified after payment
- Total must equal sum of line items

---

### 3.5 Certificate Aggregate

**Aggregate Root:** `TemporaryCertificate`

The issued temporary registration certificate.

| Property | Type | Description |
|----------|------|-------------|
| `certificateId` | CertificateId (VO) | Unique certificate identifier |
| `certificateNumber` | String | Official certificate number |
| `requestReference` | RequestId (VO) | Associated registration request |
| `vesselReference` | VesselId (VO) | Registered vessel |
| `ownerReference` | ApplicantId (VO) | Certificate holder |
| `issuedDate` | Date | Issuance date |
| `expiryDate` | Date | Expiry date |
| `validityPeriod` | Period (VO) | Validity duration (config-driven) |
| `status` | CertificateStatus (Enum) | ACTIVE, EXPIRED, SUPERSEDED, CANCELLED |
| `qrCode` | String | QR code data for verification |
| `documentUrl` | String | PDF document URL |

**Invariants:**
- Expiry date must be after issuance date
- Certificate status transitions follow defined rules

---

### 3.6 Post-Issuance Monitoring Aggregate (From FR-SP10)

**Aggregate Root:** `PostIssuanceMonitor`

Tracks post-issuance reminders and penalties for permanent registration completion.

| Property | Type | Description |
|----------|------|-------------|
| `monitorId` | MonitorId (VO) | Unique monitor identifier |
| `certificateReference` | CertificateId (VO) | Associated temporary certificate |
| `requestReference` | RequestId (VO) | Original registration request |
| `monitoringState` | MonitoringState (Enum) | ACTIVE, PAUSED, CLOSED |
| `startedAt` | DateTime | When monitoring started |
| `closedAt` | DateTime | When monitoring closed (nullable) |
| `closureReason` | String | Reason for closure |
| `reminderSchedule` | ReminderSchedule (Entity) | Reminder configuration and state |
| `appliedPenalties` | List<PenaltyRecord> (Entity) | Penalties applied for overdue |
| `permanentRegistrationStatus` | CompletionStatus (Enum) | NOT_STARTED, IN_PROGRESS, COMPLETED |

**Invariants:**
- Monitoring must close when permanent registration is completed
- Penalties can only be applied when overdue thresholds are exceeded

#### 3.6.1 ReminderSchedule Entity

| Property | Type | Description |
|----------|------|-------------|
| `scheduleId` | ScheduleId (VO) | Unique schedule identifier |
| `cadenceDays` | Integer | Reminder interval in days (REG-001-CONF-02) |
| `lastReminderSentAt` | DateTime | Timestamp of last reminder |
| `nextReminderDueAt` | DateTime | Next scheduled reminder |
| `reminderCount` | Integer | Number of reminders sent |
| `isActive` | Boolean | Whether reminders are active |

#### 3.6.2 PenaltyRecord Entity

| Property | Type | Description |
|----------|------|-------------|
| `penaltyId` | PenaltyId (VO) | Unique penalty identifier |
| `penaltyType` | PenaltyType (Enum) | OVERDUE, LATE_COMPLETION |
| `amount` | Money (VO) | Penalty amount |
| `appliedAt` | DateTime | When penalty was applied |
| `daysOverdue` | Integer | Days past threshold |
| `ruleReference` | String | DMN rule reference |
| `invoiceReference` | InvoiceId (VO) | Associated invoice (nullable) |
| `status` | PenaltyStatus (Enum) | PENDING, INVOICED, PAID, WAIVED |

---

### 3.7 Inspection Aggregate (From FR-SP06)

**Aggregate Root:** `InspectionRequest`

Manages vessel inspection requests and outcomes.

| Property | Type | Description |
|----------|------|-------------|
| `inspectionId` | InspectionId (VO) | Unique inspection identifier |
| `registrationRequestRef` | RequestId (VO) | Associated registration request |
| `vesselReference` | VesselId (VO) | Vessel to inspect |
| `inspectionType` | InspectionType (Enum) | INITIAL, SAFETY, CLASSIFICATION |
| `requestedAt` | DateTime | When inspection was requested |
| `scheduledAt` | DateTime | Scheduled inspection date/time |
| `status` | InspectionStatus (Enum) | REQUESTED, SCHEDULED, IN_PROGRESS, COMPLETED, CANCELLED |
| `outcome` | InspectionOutcome (Enum) | PASSED, FAILED, CANCELLED, TIMEOUT |
| `inspectorReference` | String | External inspector reference |
| `report` | InspectionReport (Entity) | Inspection report (nullable) |
| `completedAt` | DateTime | When inspection completed |

**Invariants:**
- Outcome can only be set when status is COMPLETED
- Report is required when outcome is PASSED or FAILED

#### 3.7.1 InspectionReport Entity

| Property | Type | Description |
|----------|------|-------------|
| `reportId` | ReportId (VO) | Unique report identifier |
| `reportNumber` | String | Official report number |
| `issuedAt` | DateTime | Report issuance date |
| `findings` | String | Inspection findings |
| `recommendations` | String | Recommendations (if any) |
| `documentReference` | DocumentReference (VO) | Uploaded report document |
| `expiryDate` | Date | Report validity expiry |

---

## 4. Entities (Within Aggregates)

### 4.1 ApplicantSnapshot (within RegistrationRequest)

Immutable copy of applicant data at request submission time.

| Property | Type | Description |
|----------|------|-------------|
| `applicantReference` | ApplicantId (VO) | Reference to master applicant |
| `applicantType` | ApplicantType (Enum) | INDIVIDUAL or COMPANY |
| `displayName` | String | Full name or company name |
| `nationalId` | String | National ID (individual) or CR (company) |
| `contactEmail` | Email (VO) | Contact email |
| `contactPhone` | PhoneNumber (VO) | Contact phone |
| `capturedAt` | DateTime | Snapshot timestamp |

---

### 4.2 VesselSnapshot (within RegistrationRequest)

Immutable copy of vessel data at request submission time.

| Property | Type | Description |
|----------|------|-------------|
| `vesselReference` | VesselId (VO) | Reference to master vessel (if exists) |
| `proposedName` | String | Proposed vessel name |
| `vesselCategory` | VesselCategory (Enum) | Vessel type |
| `buildYear` | Year | Construction year |
| `buildMaterial` | BuildMaterial (Enum) | Hull material |
| `grossTonnage` | Decimal | Gross tonnage (GT) |
| `netTonnage` | Decimal | Net tonnage (NT) |
| `length` | Decimal | Length overall (meters) |
| `acquisitionPath` | AcquisitionPath (Enum) | How vessel was acquired |
| `activityType` | ActivityType (Enum) | Intended activity |
| `capturedAt` | DateTime | Snapshot timestamp |

---

### 4.3 DocumentsBundle (within RegistrationRequest)

Collection of required and submitted documents.

| Property | Type | Description |
|----------|------|-------------|
| `requiredDocuments` | List<RequiredDocument> | Documents required by policy |
| `uploadedDocuments` | List<UploadedDocument> | Documents uploaded by applicant |
| `completenessStatus` | CompletenessStatus (Enum) | INCOMPLETE, COMPLETE, VALIDATED |
| `lastValidatedAt` | DateTime | Last validation timestamp |

---

### 4.4 ApprovalStatus (within RegistrationRequest)

Tracks external approval states.

| Property | Type | Description |
|----------|------|-------------|
| `mafwrApproval` | ExternalApproval (VO) | MAFWR fishing approval status |
| `inspectionApproval` | ExternalApproval (VO) | Inspection outcome status |
| `customsApproval` | ExternalApproval (VO) | Customs clearance status |

---

### 4.5 PaymentRecord (within RegistrationRequest)

Payment tracking within the request.

| Property | Type | Description |
|----------|------|-------------|
| `invoiceReference` | InvoiceId (VO) | Associated invoice |
| `paymentStatus` | PaymentStatus (Enum) | PENDING, PROCESSING, COMPLETED, FAILED, EXPIRED |
| `paymentMethod` | String | Payment method used |
| `transactionReference` | String | Gateway transaction reference |
| `paidAmount` | Money (VO) | Amount paid |
| `paidAt` | DateTime | Payment timestamp |

---

### 4.6 CertificateIssuance (within RegistrationRequest)

Certificate issuance tracking within the request.

| Property | Type | Description |
|----------|------|-------------|
| `certificateReference` | CertificateId (VO) | Issued certificate reference |
| `issuedAt` | DateTime | Issuance timestamp |
| `issuedBy` | String | Issuing authority/system |
| `deliveryChannel` | DeliveryChannel (Enum) | How certificate was delivered |

---

### 4.7 InvoiceLineItem (within Invoice)

Individual fee or penalty line in an invoice.

| Property | Type | Description |
|----------|------|-------------|
| `lineItemId` | UUID | Unique line item identifier |
| `itemType` | LineItemType (Enum) | FEE, PENALTY, TAX, DISCOUNT |
| `description` | String | Line item description |
| `descriptionAr` | String | Arabic description |
| `amount` | Money (VO) | Line item amount |
| `ruleReference` | String | DMN rule or config reference |

---

## 5. Value Objects

### 5.1 Identity Value Objects

| Value Object | Properties | Description |
|--------------|------------|-------------|
| `RequestId` | value: UUID | Registration request identifier |
| `VesselId` | value: UUID | Vessel identifier |
| `ApplicantId` | value: UUID | Applicant identifier |
| `InvoiceId` | value: UUID | Invoice identifier |
| `CertificateId` | value: UUID | Certificate identifier |

### 5.2 Domain Value Objects

| Value Object | Properties | Description |
|--------------|------------|-------------|
| `VesselName` | name: String, reservedUntil: DateTime | Vessel name with reservation |
| `VesselDimensions` | length: Decimal, beam: Decimal, draft: Decimal, grossTonnage: Decimal, netTonnage: Decimal | Vessel measurements |
| `ConstructionPeriod` | startDate: Date, endDate: Date | Construction timeline |
| `Money` | amount: Decimal, currency: Currency | Monetary value |
| `Period` | startDate: Date, endDate: Date | Date range |
| `Email` | value: String | Validated email address |
| `PhoneNumber` | countryCode: String, number: String | Phone number |
| `ContactInfo` | email: Email, phone: PhoneNumber, address: Address | Contact details |
| `Address` | street: String, city: String, region: String, country: String, postalCode: String | Physical address |
| `NameReservation` | proposedName: String, reservedAt: DateTime, expiresAt: DateTime, status: ReservationStatus | Name reservation details |
| `PenaltyLineItem` | reason: String, ruleReference: String, amount: Money, appliedAt: DateTime | Applied penalty |
| `ExternalApproval` | status: ApprovalOutcome, requestedAt: DateTime, respondedAt: DateTime, reference: String, reason: String | External approval tracking |

---

## 6. Enumerations

### 6.1 Request Status

```
RequestStatus:
  - DRAFT                  # Initial state, data entry in progress
  - SUBMITTED              # Request submitted, validation pending
  - DOCUMENTS_PENDING      # Waiting for document upload
  - DOCUMENTS_COMPLETE     # Documents validated
  - APPROVAL_PENDING       # Waiting for external approvals
  - APPROVED               # All approvals received
  - PAYMENT_PENDING        # Waiting for payment
  - PAYMENT_PROCESSING     # Payment in progress
  - PAID                   # Payment confirmed
  - ISSUING                # Certificate generation in progress
  - ISSUED                 # Certificate issued
  - REJECTED               # Request rejected
  - CANCELLED              # Request cancelled by applicant
  - EXPIRED                # Request expired due to timeout
```

### 6.2 Vessel Category

```
VesselCategory:
  - CARGO                  # سفن الشحن
  - TANKER                 # ناقلات
  - CONTAINER              # سفن الحاويات
  - BULK_CARRIER           # ناقلات البضائع السائبة
  - PASSENGER              # سفن الركاب
  - FISHING                # سفن الصيد
  - NAVAL                  # السفن البحرية
  - RESEARCH               # سفن البحث العلمي
  - TUGBOAT                # قاطرات
  - OFFSHORE               # منصات بحرية
  - YACHT                  # يخوت
  - FERRY                  # عبّارات
  - BOAT                   # قوارب
  - OTHER                  # أخرى
```

### 6.3 Build Material

```
BuildMaterial:
  - STEEL                  # فولاذ
  - ALUMINUM               # ألومنيوم
  - FIBERGLASS             # ألياف زجاجية
  - WOOD                   # خشب
  - COMPOSITE              # مركب
  - OTHER                  # أخرى
```

### 6.4 Acquisition Path

```
AcquisitionPath:
  - NEW_BUILD              # بناء جديد
  - PURCHASE               # شراء
  - FLAG_CHANGE            # تغيير العلم
```

### 6.5 Activity Type

```
ActivityType:
  - COMMERCIAL             # تجاري
  - FISHING                # صيد
  - TOURISM                # سياحة
  - SERVICE                # خدمات
  - PRIVATE                # خاص
  - OTHER                  # أخرى
```

### 6.6 Applicant Type

```
ApplicantType:
  - INDIVIDUAL             # فرد
  - COMPANY                # شركة
```

### 6.7 Nationality Type

```
NationalityType:
  - OMANI_CITIZEN          # مواطن عماني
  - OMANI_RESIDENT         # مقيم في عمان
  - GCC_CITIZEN            # مواطن خليجي
  - FOREIGN                # أجنبي
```

### 6.8 Approval Outcome

```
ApprovalOutcome:
  - NOT_REQUIRED           # غير مطلوب
  - PENDING                # قيد الانتظار
  - APPROVED               # موافق عليه
  - REJECTED               # مرفوض
  - TIMEOUT                # انتهت المهلة
```

### 6.9 Verification Status

```
VerificationStatus:
  - NOT_VERIFIED           # لم يتم التحقق
  - PENDING                # قيد التحقق
  - VERIFIED               # تم التحقق
  - FAILED                 # فشل التحقق
```

### 6.10 CR Status (Commercial Registration)

```
CRStatus:
  - ACTIVE                 # نشط
  - EXPIRED                # منتهي
  - SUSPENDED              # موقوف
  - CANCELLED              # ملغى
```

### 6.11 Agent Type

```
AgentType:
  - POWER_OF_ATTORNEY      # موكل بتوكيل
  - AUTHORIZED_REPRESENTATIVE  # ممثل معتمد
  - COMPANY_DELEGATE       # مفوض الشركة
```

### 6.12 Configuration Status

```
ConfigurationStatus:
  - DRAFT                  # مسودة
  - ACTIVE                 # نشط
  - DEPRECATED             # ملغى
  - ARCHIVED               # مؤرشف
```

### 6.13 Monitoring State (From FR-SP10)

```
MonitoringState:
  - ACTIVE                 # نشط - مراقبة جارية
  - PAUSED                 # موقف مؤقتاً
  - CLOSED                 # مغلق - انتهت المراقبة
```

### 6.14 Completion Status (From FR-SP10)

```
CompletionStatus:
  - NOT_STARTED            # لم يبدأ
  - IN_PROGRESS            # قيد التنفيذ
  - COMPLETED              # مكتمل
```

### 6.15 Penalty Type (From FR-SP10)

```
PenaltyType:
  - OVERDUE                # تأخر في إتمام التسجيل الدائم
  - LATE_COMPLETION        # إتمام متأخر
  - CONSTRUCTION_DELAY     # تأخر في البناء (>30 يوم)
```

### 6.16 Penalty Status

```
PenaltyStatus:
  - PENDING                # قيد الانتظار
  - INVOICED               # تم إصدار الفاتورة
  - PAID                   # مدفوع
  - WAIVED                 # ملغى
```

### 6.17 Inspection Type (From FR-SP06)

```
InspectionType:
  - INITIAL                # فحص أولي
  - SAFETY                 # فحص السلامة
  - CLASSIFICATION         # فحص التصنيف
  - SEAWORTHINESS          # فحص صلاحية الإبحار
```

### 6.18 Inspection Status (From FR-SP06)

```
InspectionStatus:
  - REQUESTED              # تم الطلب
  - SCHEDULED              # مجدول
  - IN_PROGRESS            # قيد التنفيذ
  - COMPLETED              # مكتمل
  - CANCELLED              # ملغى
```

### 6.19 Inspection Outcome (From FR-SP06)

```
InspectionOutcome:
  - PASSED                 # ناجح
  - FAILED                 # فاشل
  - CANCELLED              # ملغى
  - TIMEOUT                # انتهت المهلة
```

### 6.20 Certificate Status

```
CertificateStatus:
  - ACTIVE                 # نشط
  - EXPIRED                # منتهي الصلاحية
  - SUPERSEDED             # تم استبداله
  - CANCELLED              # ملغى
  - SUSPENDED              # موقوف
```

### 6.21 Invoice Status (From FR-SP08)

```
InvoiceStatus:
  - DRAFT                  # مسودة
  - ISSUED                 # صادر
  - PAID                   # مدفوع
  - CANCELLED              # ملغى
  - EXPIRED                # منتهي الصلاحية
```

### 6.22 Reservation Status (From FR-SP08)

```
ReservationStatus:
  - RESERVED               # محجوز
  - CONFIRMED              # مؤكد
  - RELEASED               # محرر
  - EXPIRED                # منتهي
```

---

## 7. Domain Events

Domain events capture significant state changes that other services/domains may react to.

### 7.1 Registration Request Events

| Event | Trigger | Payload |
|-------|---------|---------|
| `RegistrationRequestCreated` | New request initiated | requestId, applicantId, timestamp |
| `RegistrationRequestSubmitted` | Request submitted | requestId, vesselData, applicantData |
| `DocumentsCompleted` | All required documents uploaded | requestId, documentCount |
| `ApprovalRequested` | External approval requested | requestId, approvalType, targetSystem |
| `ApprovalReceived` | External approval response | requestId, approvalType, outcome, reason |
| `PaymentInitiated` | Payment process started | requestId, invoiceId, amount |
| `PaymentCompleted` | Payment confirmed | requestId, invoiceId, transactionRef |
| `PaymentFailed` | Payment failed | requestId, invoiceId, reason |
| `CertificateIssued` | Certificate generated | requestId, certificateId, expiryDate |
| `RequestRejected` | Request rejected | requestId, reason, rejectedBy |
| `RequestCancelled` | Request cancelled | requestId, reason |

### 7.2 Vessel Events

| Event | Trigger | Payload |
|-------|---------|---------|
| `VesselProfileCreated` | New vessel registered | vesselId, vesselName, category |
| `VesselNameReserved` | Name reserved | vesselId, name, expiresAt |
| `VesselNameReleased` | Name reservation released | vesselId, name, reason |
| `VesselRegistered` | Vessel officially registered | vesselId, certificateId, registrationDate |

### 7.3 Billing Events

| Event | Trigger | Payload |
|-------|---------|---------|
| `InvoiceGenerated` | Invoice created | invoiceId, requestId, totalAmount |
| `InvoicePaid` | Invoice paid | invoiceId, paymentRef, paidAt |
| `PenaltyApplied` | Penalty added | requestId, penaltyType, amount, reason |

### 7.4 Certificate Events

| Event | Trigger | Payload |
|-------|---------|---------|
| `TemporaryCertificateIssued` | Certificate issued | certificateId, vesselId, expiryDate |
| `CertificateExpiringSoon` | Expiry reminder | certificateId, expiryDate, daysRemaining |
| `CertificateExpired` | Certificate expired | certificateId, expiredAt |

### 7.5 Inspection Events (From FR-SP06)

| Event | Trigger | Payload |
|-------|---------|---------|
| `InspectionRequested` | Inspection request created | inspectionId, requestId, vesselId, inspectionType |
| `InspectionScheduled` | Inspection scheduled | inspectionId, scheduledAt, inspectorRef |
| `InspectionCompleted` | Inspection completed | inspectionId, outcome, reportRef |
| `InspectionFailed` | Inspection failed | inspectionId, reason |
| `InspectionCancelled` | Inspection cancelled | inspectionId, reason |

### 7.6 Post-Issuance Monitoring Events (From FR-SP10)

| Event | Trigger | Payload |
|-------|---------|---------|
| `MonitoringStarted` | Monitoring activated | monitorId, certificateId, startedAt |
| `ReminderSent` | Reminder notification sent | monitorId, reminderCount, sentAt |
| `OverdueThresholdReached` | Overdue threshold exceeded | monitorId, daysOverdue |
| `OverduePenaltyApplied` | Penalty for overdue applied | monitorId, penaltyId, amount |
| `PermanentRegistrationCompleted` | Permanent registration done | monitorId, certificateId, completedAt |
| `MonitoringClosed` | Monitoring closed | monitorId, closureReason, closedAt |

### 7.7 Customs Events (From FR-SP07)

| Event | Trigger | Payload |
|-------|---------|---------|
| `CustomsClearanceRequested` | Request sent to Bayan | requestId, bayanReference |
| `CustomsClearanceReceived` | Response from Bayan | requestId, clearanceStatus, documentRef |
| `CustomsClearanceRejected` | Bayan rejected clearance | requestId, reason |

---

## 8. Conceptual Class Diagram (UML)

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                    REGISTRATION REQUEST AGGREGATE                                    │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                      │
│  ┌──────────────────────────────────┐                                                               │
│  │    <<Aggregate Root>>            │                                                               │
│  │    RegistrationRequest           │                                                               │
│  ├──────────────────────────────────┤                                                               │
│  │ + requestId: RequestId           │                                                               │
│  │ + requestNumber: String          │                                                               │
│  │ + status: RequestStatus          │                                                               │
│  │ + nameReservation: NameReserv... │                                                               │
│  │ + penalties: List<PenaltyLine..> │                                                               │
│  │ + createdAt: DateTime            │                                                               │
│  │ + submittedAt: DateTime          │                                                               │
│  │ + issuedAt: DateTime             │                                                               │
│  ├──────────────────────────────────┤                                                               │
│  │ + submit()                       │                                                               │
│  │ + validateDocuments()            │                                                               │
│  │ + requestApproval(type)          │                                                               │
│  │ + recordApproval(type, outcome)  │                                                               │
│  │ + initiatePayment()              │                                                               │
│  │ + confirmPayment(ref)            │                                                               │
│  │ + issueCertificate()             │                                                               │
│  │ + reject(reason)                 │                                                               │
│  │ + cancel(reason)                 │                                                               │
│  └───────────┬──────────────────────┘                                                               │
│              │                                                                                       │
│   ┌──────────┼──────────┬──────────────────┬──────────────────┬────────────────┐                    │
│   │          │          │                  │                  │                │                    │
│   ▼          ▼          ▼                  ▼                  ▼                ▼                    │
│ ┌────────────────┐ ┌────────────────┐ ┌────────────────┐ ┌────────────────┐ ┌────────────────┐      │
│ │ <<Entity>>     │ │ <<Entity>>     │ │ <<Entity>>     │ │ <<Entity>>     │ │ <<Entity>>     │      │
│ │ Applicant      │ │ Vessel         │ │ Documents      │ │ Approval       │ │ Payment        │      │
│ │ Snapshot       │ │ Snapshot       │ │ Bundle         │ │ Status         │ │ Record         │      │
│ ├────────────────┤ ├────────────────┤ ├────────────────┤ ├────────────────┤ ├────────────────┤      │
│ │ applicantRef   │ │ vesselRef      │ │ required[]     │ │ mafwrApproval  │ │ invoiceRef     │      │
│ │ applicantType  │ │ proposedName   │ │ uploaded[]     │ │ inspection     │ │ paymentStatus  │      │
│ │ displayName    │ │ vesselCategory │ │ completeness   │ │ customs        │ │ transactionRef │      │
│ │ nationalId     │ │ buildYear      │ │ lastValidated  │ │                │ │ paidAmount     │      │
│ │ contactEmail   │ │ grossTonnage   │ │                │ │                │ │ paidAt         │      │
│ │ contactPhone   │ │ length         │ │                │ │                │ │                │      │
│ │ capturedAt     │ │ activityType   │ │                │ │                │ │                │      │
│ └────────────────┘ └────────────────┘ └────────────────┘ └────────────────┘ └────────────────┘      │
│                                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                    SUPPORTING AGGREGATES                                             │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                      │
│  ┌────────────────────┐      ┌────────────────────┐      ┌────────────────────┐                     │
│  │ <<Aggregate Root>> │      │ <<Aggregate Root>> │      │ <<Aggregate Root>> │                     │
│  │ VesselProfile      │      │ ApplicantProfile   │      │ Invoice            │                     │
│  ├────────────────────┤      ├────────────────────┤      ├────────────────────┤                     │
│  │ vesselId           │      │ applicantId        │      │ invoiceId          │                     │
│  │ imoNumber          │      │ applicantType      │      │ invoiceNumber      │                     │
│  │ vesselName         │      │ individualProfile  │      │ requestReference   │                     │
│  │ vesselCategory     │      │ companyProfile     │      │ lineItems[]        │                     │
│  │ buildMaterial      │      │ contactInfo        │      │ totalAmount        │                     │
│  │ buildYear          │      │ agents[]           │      │ status             │                     │
│  │ dimensions         │      │                    │      │ paymentReference   │                     │
│  │ acquisitionPath    │      │                    │      │                    │                     │
│  │ activityType       │      │                    │      │                    │                     │
│  └────────────────────┘      └────────────────────┘      └────────────────────┘                     │
│                                                                                                      │
│  ┌────────────────────┐                                                                             │
│  │ <<Aggregate Root>> │                                                                             │
│  │ TemporaryCertif... │                                                                             │
│  ├────────────────────┤                                                                             │
│  │ certificateId      │                                                                             │
│  │ certificateNumber  │                                                                             │
│  │ requestReference   │                                                                             │
│  │ vesselReference    │                                                                             │
│  │ ownerReference     │                                                                             │
│  │ issuedDate         │                                                                             │
│  │ expiryDate         │                                                                             │
│  │ validityPeriod     │                                                                             │
│  │ status             │                                                                             │
│  │ qrCode             │                                                                             │
│  └────────────────────┘                                                                             │
│                                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 9. Policy & Configuration Architecture

Based on analysis of the source DOCX (إصدار شهادة تسجيل سفينة أو وحدة بحرية مؤقتة.docx), the domain model uses a **two-tier** architecture:

1. **BC-REGISTRATION-POLICY** — Combines DMN decisions + configurable thresholds (CONF-xx are inputs to DMN)
2. **BC-MASTERDATA** — Reference data and lookup tables (SET-xx)

**Rationale for merging Policy + Config:**
- CONF-xx codes are **inputs to DMN rules**, not separate decisions
- Only 4 configurations (CONF-01 to CONF-04) — doesn't justify a separate service
- DOCX treats them as "إعدادات الدورة" (process settings), part of policy context
- Simpler architecture with fewer inter-service calls

```
┌──────────────────────────────────────────────────────────────────────────┐
│                    POLICY & CONFIGURATION ARCHITECTURE                    │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │                 DOMAIN/PROCESS SERVICES                          │   │
│   │  (RegistrationRequest, VesselProfile, Invoice, etc.)            │   │
│   │                  ↓ calls (never owns rules)                      │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                              │                                           │
│                              ▼                                           │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │            BC-REGISTRATION-POLICY (Section 9.1)                  │   │
│   │  - DMN Engine & Rule Evaluation                                  │   │
│   │  - Policy Configurations (CONF-01 to CONF-04)                    │   │
│   │  - Policy Versioning                                             │   │
│   │                  ↓ reads (lookup only)                           │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                              │                                           │
│                              ▼                                           │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │               BC-MASTERDATA (Section 9.2)                        │   │
│   │  - Reference Codes (SET-01 to SET-06)                            │   │
│   │  - Lookup Tables                                                 │   │
│   │  - Static Categories                                             │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

### 9.1 Registration Policy Aggregate (BC-REGISTRATION-POLICY)

The Registration Policy aggregate manages DMN rule evaluation, decision logic, AND configurable thresholds/timing rules. Domain services call Policy Service for all decisions—they MUST NOT implement decision logic locally.

#### 9.1.1 Registration Policy Aggregate Root

```
<<Aggregate Root>>
RegistrationPolicy
├── policyId: RegistrationPolicyId
├── serviceCode: String (e.g., "REG-001")
├── version: VersionInfo (VO)
├── decisions: List<PolicyDecision>
├── configurations: List<PolicyConfiguration>
├── status: PolicyStatus
├── effectiveFrom: Date
├── effectiveTo: Date (nullable)
├── createdAt: DateTime
├── updatedAt: DateTime
└── createdBy: String
```

#### 9.1.2 Policy Decision Entity

```
<<Entity>>
PolicyDecision
├── decisionId: PolicyDecisionId
├── decisionKey: String (e.g., "fee-calculation", "vessel-age-validation")
├── dmnReference: DmnReference (VO)
├── inputSchema: JsonSchema
├── outputSchema: JsonSchema
├── isActive: Boolean
└── description: LocalizedText (VO)
```

#### 9.1.3 Policy Configuration Entity

```
<<Entity>>
PolicyConfiguration
├── configId: PolicyConfigurationId
├── configCode: String (e.g., "REG-001-CONF-01")
├── configNameAr: String
├── configNameEn: String
├── configType: PolicyConfigType
├── value: ConfigValue (VO)
├── dmnReference: String (nullable, if used by DMN)
├── isActive: Boolean
└── validFrom: Date
```

#### 9.1.4 DMN Reference Value Object

```
<<Value Object>>
DmnReference
├── dmnFileId: String
├── dmnFileName: String (e.g., "reg001-fee-calculation.dmn")
├── decisionTableId: String
├── versionTag: String (e.g., "v1.2")
└── deploymentId: String
```

#### 9.1.5 Config Value Objects

```
<<Value Object>>
ConfigValue
├── valueType: ConfigValueType (THRESHOLD, DURATION, FLAG, TABLE)
├── numericValue: BigDecimal (nullable)
├── durationValue: Duration (nullable)
├── booleanValue: Boolean (nullable)
├── tableValue: List<Map<String, Object>> (nullable)
└── unit: String (nullable)

<<Value Object>>
Duration
├── value: Integer
├── unit: DurationUnit
└── toJavaDuration(): java.time.Duration

<<Enumeration>>
DurationUnit:
  - HOURS
  - DAYS
  - WEEKS
  - MONTHS
  - YEARS

<<Enumeration>>
PolicyConfigType:
  - THRESHOLD          # Numeric limit (e.g., max age, max tonnage)
  - DURATION           # Time period (e.g., validity, reservation)
  - DECISION_TABLE     # Multi-value table for DMN
  - BLACKLIST          # Prohibited items list
  - DOCUMENT_RULES     # Document requirement rules
```

#### 9.1.6 Policy Decision Types (REG-001)

| Decision Key | DMN File | Purpose | Input Fields | Config Dependencies |
|--------------|----------|---------|--------------|---------------------|
| `fee-calculation` | reg001-fee-calculation.dmn | Calculate registration fees | vesselCategory, grossTonnage, length, isNew | SET-06 (fee table) |
| `vessel-age-validation` | reg001-vessel-age-validation.dmn | Validate vessel age policy | vesselType, buildYear, material | CONF-01 (age limits) |
| `documents-required` | reg001-documents-required.dmn | Determine required documents | vesselCategory, acquisitionPath, activityType | CONF-01 (doc rules) |
| `inspection-required` | reg001-inspection-required.dmn | Determine if inspection needed | vesselCategory, vesselAge, isNewBuild, length | — |
| `mafwr-required` | reg001-mafwr-required.dmn | Determine MAFWR approval need | activityType, vesselCategory, grossTonnage | CONF-02 (tonnage limit) |
| `customs-required` | reg001-customs-required.dmn | Determine customs clearance need | acquisitionPath, importCountry | — |
| `penalty-calculation` | reg001-penalty-calculation.dmn | Calculate late penalties | daysOverdue, originalFee | CONF-02 (timing) |

#### 9.1.7 Policy Configurations (REG-001-CONF-xx)

From DOCX "إعدادات الدورة" section:

| Config Code | Description (AR) | Description (EN) | Type | Value | Used By |
|-------------|------------------|------------------|------|-------|---------|
| REG-001-CONF-01 | إعداد المستندات | Document requirements by vessel type/activity | DOCUMENT_RULES | See DMN | reg001-documents-required.dmn |
| REG-001-CONF-02 | إعداد توقيت فرض مخالفة | Penalty timing (30 days from build completion) | DURATION + THRESHOLD | 30 days | Penalty trigger, MAFWR check |
| REG-001-CONF-03 | إعداد صلاحية الشهادة | Certificate validity period | DURATION | 6 months | Certificate issuance |
| REG-001-CONF-04 | إعداد السفن المحظورة | Prohibited vessels list | BLACKLIST | Dynamic list | Vessel registration block |

#### 9.1.8 PolicyStatus Enumeration

```
PolicyStatus:
  - DRAFT                  # تحت التحضير - Under development
  - ACTIVE                 # فعّال - Currently in use
  - SCHEDULED              # مجدول للتفعيل - Scheduled activation
  - DEPRECATED             # قيد الإيقاف - Scheduled for retirement
  - RETIRED                # متوقف - No longer used
```

#### 9.1.9 Policy Domain Events

| Event | Description | Raised When |
|-------|-------------|-------------|
| `RegistrationPolicyCreated` | New policy version created | Admin creates policy |
| `RegistrationPolicyActivated` | Policy activated | Policy goes live |
| `PolicyDecisionAdded` | New DMN decision registered | New DMN rule added |
| `PolicyDecisionUpdated` | DMN decision modified | DMN rule updated |
| `PolicyConfigurationUpdated` | Configuration changed | Threshold/timing modified |
| `PolicyEvaluationCompleted` | Decision evaluated | DMN rule executed |
| `PolicyEvaluationFailed` | Decision evaluation failed | DMN execution error |
| `ProhibitedVesselAdded` | Vessel added to blacklist | Admin blocks vessel |
| `ProhibitedVesselRemoved` | Vessel removed from blacklist | Admin unblocks vessel |

---

### 9.2 Master Data Aggregate (BC-MASTERDATA)

The Master Data aggregate manages reference codes, lookup tables, and static categories. This is READ-ONLY reference data used by Policy and Domain services.

#### 9.2.1 Master Data Aggregate Root

```
<<Aggregate Root>>
MasterDataCategory
├── categoryId: MasterDataCategoryId
├── categoryCode: String (e.g., "REG-001-SET-01")
├── categoryNameAr: String
├── categoryNameEn: String
├── serviceCode: String (e.g., "REG-001")
├── items: List<MasterDataItem>
├── isActive: Boolean
├── effectiveFrom: Date
├── effectiveTo: Date (nullable)
├── version: Integer
├── createdAt: DateTime
├── updatedAt: DateTime
└── createdBy: String
```

#### 9.2.2 Master Data Item Entity

```
<<Entity>>
MasterDataItem
├── itemId: MasterDataItemId
├── itemCode: String (e.g., "CARGO_SHIP", "STEEL")
├── itemNameAr: String
├── itemNameEn: String
├── sortOrder: Integer
├── isActive: Boolean
├── attributes: Map<String, Object> (extensible properties)
├── parentItemId: MasterDataItemId (nullable, for hierarchical data)
└── validFrom: Date
```

#### 9.2.3 Master Data Categories (REG-001-SET-xx)

| Category Code | Description (AR) | Description (EN) | Example Items |
|---------------|------------------|------------------|---------------|
| REG-001-SET-01 | نوع السفينة / الوحدة البحرية | Vessel Types | CARGO_SHIP, FISHING_VESSEL, YACHT, MARITIME_UNIT |
| REG-001-SET-02 | مادة البناء | Build Materials | STEEL, ALUMINUM, FIBERGLASS, WOOD |
| REG-001-SET-03 | الموانئ العمانية | Omani Ports | PORT_SULTAN_QABOOS, PORT_SALALAH, PORT_SOHAR, PORT_DUQM |
| REG-001-SET-04 | نوع الاستخدام | Activity Types | COMMERCIAL_TRANSPORT, FISHING, TOURISM, PRIVATE |
| REG-001-SET-05 | الأنشطة التجارية المؤهلة | Eligible CR Activities | MARITIME_TRANSPORT, FISHING_COMPANY, TOURISM_OPERATOR |
| REG-001-SET-06 | مصادر التملك | Acquisition Paths | LOCAL_BUILD, IMPORT, TRANSFER, INHERITANCE |

#### 9.2.4 Master Data Value Objects

```
<<Value Object>>
LocalizedName
├── nameAr: String
├── nameEn: String
└── toString(locale: Locale): String

<<Value Object>>
MasterDataReference
├── categoryCode: String
├── itemCode: String
├── displayName: LocalizedName
└── isValid(): Boolean
```

#### 9.2.5 Master Data Domain Events

| Event | Description | Raised When |
|-------|-------------|-------------|
| `MasterDataCategoryCreated` | New category created | Admin adds new category |
| `MasterDataCategoryUpdated` | Category metadata updated | Admin modifies category |
| `MasterDataItemAdded` | New item added to category | Admin adds new lookup value |
| `MasterDataItemUpdated` | Item details updated | Admin modifies lookup value |
| `MasterDataItemDeactivated` | Item marked inactive | Item no longer valid |
| `MasterDataCacheRefreshed` | Cache invalidated | Data changed, consumers notified |

---

### 9.3 Policy Separation Rules (Enforcement)

| # | Rule | Violation Example |
|---|------|-------------------|
| 1 | **No DMN duplication** | Same fee logic in two services |
| 2 | **No policy logic in domain services** | RegistrationRequest calculates fees |
| 3 | **No master data in policy service** | Policy service stores vessel categories |
| 4 | **DMN changes ≠ service redeploy** | DMN embedded in service JAR |
| 5 | **Config is part of policy context** | Separate config service for 4 settings |
| 6 | **Master data is READ-ONLY for consumers** | Domain service modifies lookup tables |

### 9.4 Service Interaction Pattern

```
┌───────────────────────────────────────────────────────────────────────────┐
│                     DECISION FLOW EXAMPLE                                  │
├───────────────────────────────────────────────────────────────────────────┤
│                                                                           │
│   RegistrationRequest (Domain)                                            │
│       │                                                                   │
│       ├──► RegistrationPolicyService.evaluateDecision("fee-calculation")  │
│       │       │                                                           │
│       │       ├──► Load CONF-xx (internal to policy service)              │
│       │       │       └──► Returns: thresholds for DMN                    │
│       │       │                                                           │
│       │       ├──► MasterDataService.lookup("REG-001-SET-01", code)       │
│       │       │       └──► Returns: vessel category details               │
│       │       │                                                           │
│       │       └──► DMN Engine: reg001-fee-calculation.dmn                 │
│       │               └──► Returns: calculated fee                        │
│       │                                                                   │
│       └──► Invoice.addLineItem(calculatedFee)                             │
│                                                                           │
└───────────────────────────────────────────────────────────────────────────┘
```

**Key Difference from Three-Tier:** Configuration (CONF-xx) is loaded internally by the Policy Service, not via separate service call.

---

## 10. Import Document Entity

The ImportDocument entity represents the imported document from Bayan Customs system for vessels acquired via import.

### 10.1 ImportDocument Entity Definition

| Attribute | Type | Description |
|-----------|------|-------------|
| `importDocumentId` | ImportDocumentId (VO) | Unique identifier |
| `bayanReference` | String | Reference number from Bayan system |
| `importDeclarationNumber` | String | بيان الاستيراد number |
| `importDate` | Date | Date of import |
| `vesselDescription` | String | Description from Bayan |
| `originCountry` | String | Country of origin |
| `customsValue` | Money (VO) | Declared customs value |
| `customsClearanceStatus` | ClearanceStatus | PENDING, CLEARED, REJECTED |
| `documentFile` | DocumentReference (VO) | PDF document reference |
| `fetchedAt` | DateTime | When retrieved from Bayan |
| `verifiedAt` | DateTime | When verified (nullable) |

### 10.2 ClearanceStatus Enumeration

```
ClearanceStatus:
  - PENDING                # قيد الانتظار
  - CLEARED                # تم التخليص
  - REJECTED               # مرفوض
  - NOT_APPLICABLE         # غير مطلوب
```

### 10.3 Integration with RegistrationRequest

The ImportDocument is linked via the DocumentsBundle entity when acquisition path is PURCHASE (imported):

```
DocumentsBundle
├── required: List<RequiredDocument>
├── uploaded: List<UploadedDocument>
├── importDocument: ImportDocument (nullable) ← Linked for imported vessels
├── completeness: Percentage
└── lastValidatedAt: DateTime
```

---

## 11. Aggregate Boundaries & Rules

### 11.1 Data Ownership

| Aggregate | Bounded Context | Owns | Does NOT Own |
|-----------|-----------------|------|--------------|
| RegistrationRequest | BC-REGISTRATION | Request lifecycle, snapshots, approvals, payment tracking | Master vessel data, master applicant data, fee calculation |
| VesselProfile | BC-VESSEL | Vessel master data, name, dimensions | Registration requests, age validation rules |
| ApplicantProfile | BC-APPLICANT | Applicant master data, contact info | Registration requests, eligibility rules |
| Invoice | BC-BILLING | Fee line items, payment status | Request details, fee calculation logic |
| TemporaryCertificate | BC-CERTIFICATE | Certificate metadata, validity | Registration process state |
| RegistrationPolicy | BC-REGISTRATION-POLICY | DMN rules, decision evaluation, policy configurations (CONF-xx), policy versioning | Master data, process orchestration |
| MasterDataCategory | BC-MASTERDATA | Reference codes (SET-xx), lookup tables, static categories | Decision logic, business rules, policies |
| PostIssuanceMonitor | BC-MONITORING | Monitoring lifecycle, reminders, overdue penalties | Certificate data, permanent registration |
| InspectionRequest | BC-INSPECTION | Inspection lifecycle, reports, outcomes | Vessel data, registration request details |

### 11.2 Service Responsibility Boundaries

| Service Type | Owns | Does NOT Own |
|--------------|------|--------------|
| **Registration Policy Service** | DMN files, rule evaluation, configurable policies (CONF-xx), policy versioning | Master data, process orchestration, external integrations |
| **Master Data Service** | Reference codes (SET-xx), lookup tables, static categories | Decision logic, business rules, policies |
| **Domain/Process Service** | Workflow orchestration, state transitions, domain aggregates | Fee calculation, validation rules, eligibility decisions |

### 11.3 Invariant Enforcement

| Aggregate | Key Invariants |
|-----------|----------------|
| RegistrationRequest | Status transitions must follow defined workflow; documents must be complete before approval; payment must be confirmed before issuance |
| VesselProfile | Age must comply with material-based limits; dimensions must be positive; port must be valid Omani port |
| ApplicantProfile | Applicant must be Omani citizen/resident; Company must have valid CR; activity must be eligible |
| Invoice | Line items must sum to total; cannot modify after payment |
| TemporaryCertificate | Expiry must be after issuance; validity is 6 months (REG-001-CONF-03); status transitions are irreversible |
| RegistrationPolicy | One active version per policy; decisions must have valid DMN references; configurations must be versioned |
| MasterDataCategory | Items must be unique within category; only one active version per category code |

---

## 12. Traceability

### 12.1 Capability to Domain Mapping

| Capability ID | Domain Concept | Aggregate |
|---------------|----------------|-----------|
| CAP-01 | Service initiation | RegistrationRequest |
| CAP-02 | Applicant authentication | ApplicantProfile (ROP PKI verification) |
| CAP-03 | Applicant profiling | ApplicantProfile, ApplicantSnapshot, IndividualProfile, CompanyProfile |
| CAP-04 | CR validation | CompanyProfile (Invest Easy verification) |
| CAP-05 | Activity eligibility | CompanyProfile.eligibleActivities |
| CAP-06 | Vessel data capture | VesselProfile, VesselSnapshot |
| CAP-07 | Vessel age validation | VesselProfile (policy-driven via REG-001-CONF-01) |
| CAP-08 | Penalty trigger evaluation | PenaltyLineItem, PenaltyRecord |
| CAP-10 | Prohibited vessel check | VesselProfile.isProhibited |
| CAP-11 | Dynamic documents | DocumentsBundle, ImportDocument |
| CAP-12 | Document upload/validation | DocumentsBundle, RequiredDocument, UploadedDocument |
| CAP-13 | MAFWR approval | ApprovalStatus.mafwrApproval |
| CAP-14 | Inspection determination | InspectionRequest (DMN-driven) |
| CAP-15 | Inspection workflow | InspectionRequest, InspectionReport |
| CAP-16 | Customs determination | ApprovalStatus.customsApproval (DMN-driven) |
| CAP-17 | Customs clearance | ApprovalStatus.customsApproval, ImportDocument |
| CAP-18 | Fee calculation | Invoice, InvoiceLineItem |
| CAP-19 | Name reservation | NameReservation (VO), REG-001-CONF-04 |
| CAP-20 | Invoice generation | Invoice, InvoiceLineItem |
| CAP-21 | Payment processing | PaymentRecord, Invoice |
| CAP-22 | Certificate issuance | TemporaryCertificate, CertificateIssuance |
| CAP-23 | Notifications | (Cross-cutting - NotificationService) |
| CAP-24 | Post-issuance reminders | PostIssuanceMonitor, ReminderSchedule |
| CAP-25 | Overdue penalties | PostIssuanceMonitor, PenaltyRecord |
| CAP-26 | Audit trail | (Cross-cutting - AuditService) |

### 12.2 Configuration Traceability

| Config Code | Domain Concept | Used By |
|-------------|----------------|---------|
| REG-001-SET-01 | VesselCategory Enum | Vessel type dropdown |
| REG-001-SET-02 | BuildMaterial Enum | Build material dropdown |
| REG-001-SET-03 | PortReference VO | Port selection |
| REG-001-SET-04 | ActivityType Enum | Activity classification |
| REG-001-SET-05 | CompanyProfile.eligibleActivities | CR eligibility check |
| REG-001-SET-06 | AcquisitionPath Enum | Acquisition source |
| REG-001-CONF-01 | VesselProfile age validation | reg001-vessel-age-validation.dmn |
| REG-001-CONF-02 | MAFWR check threshold | reg001-mafwr-required.dmn |
| REG-001-CONF-03 | TemporaryCertificate.validityPeriod | 6 months validity |
| REG-001-CONF-04 | NameReservation.expiresAt | 30 days reservation |

### 12.3 DMN Rule Mapping

| DMN File | Domain Concept | Triggered By |
|----------|----------------|--------------|
| reg001-vessel-age-validation.dmn | VesselProfile | SP03 - Vessel data capture |
| reg001-documents-required.dmn | DocumentsBundle.required | SP03 - Document determination |
| reg001-fee-calculation.dmn | InvoiceLineItem | SP08 - Fee calculation |
| reg001-penalty-calculation.dmn | PenaltyLineItem | SP08 - Penalty assessment |
| reg001-mafwr-required.dmn | ApprovalStatus.mafwrApproval | SP05 - MAFWR check |
| reg001-inspection-required.dmn | ApprovalStatus.inspectionApproval | SP06 - Inspection check |
| reg001-customs-required.dmn | ApprovalStatus.customsApproval | SP07 - Customs check |

---

## 13. Related Documents

- [02-Business_Domain_Context_Map](02-Business_Domain_Context_Map.md) – Bounded context relationships
- [03-Context_Map_ACL_SharedKernel](03-Context_Map_ACL_SharedKernel.md) – Integration patterns
- [01-Capability_to_Domain_Mapping](../../01-business-analysis/1.4-capabilities/02-Capability_to_Domain_Mapping/02-Capability_to_Domain_Mapping.md) – Source mapping
- [BPMN Process Design](../../01-business-analysis/1.3-process/03-BPMN_Process_Design/00-BPMN_Index.md) – Process context
- [Functional Requirements](../../01-business-analysis/1.5-functional-requirements/README.md) – FR source

---

**Document Control**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-08 | Solution Architect | Initial domain model structure |
| 1.1 | 2026-01-08 | Solution Architect | Added Configuration aggregate, ImportDocument entity, IndividualProfile, CompanyProfile, AgentRepresentation entities; Added NationalityType, VerificationStatus, CRStatus, AgentType, ConfigurationStatus enums; Enhanced traceability with Config and DMN mappings |
| 1.2 | 2026-01-08 | Solution Architect | FR-driven updates: Added PostIssuanceMonitor aggregate (FR-SP10), InspectionRequest aggregate (FR-SP06), ReminderSchedule, PenaltyRecord, InspectionReport entities; Added 10 new enumerations (MonitoringState, InspectionStatus, InspectionOutcome, etc.); Added 15+ new domain events; Expanded capability mapping to 26 capabilities |
| 1.3 | 2026-01-08 | Solution Architect | Renamed BC-APPROVAL to BC-MAFWR-APPROVAL for clearer MAFWR context identification; Context naming alignment with domain model |
| 1.4 | 2026-01-08 | Solution Architect | Three-Tier Decision Architecture: Added BC-POLICY, BC-MASTERDATA, BC-CONFIG bounded contexts |
| 1.5 | 2026-01-08 | Solution Architect | **DOCX-driven revision:** Merged BC-POLICY + BC-CONFIG into BC-REGISTRATION-POLICY based on source document analysis. CONF-xx are inputs to DMN rules (not separate service). Kept BC-MASTERDATA separate for SET-xx reference data. Updated aggregates: RegistrationPolicy (combined policy + config), MasterDataCategory; Simplified architecture from three-tier to two-tier |
