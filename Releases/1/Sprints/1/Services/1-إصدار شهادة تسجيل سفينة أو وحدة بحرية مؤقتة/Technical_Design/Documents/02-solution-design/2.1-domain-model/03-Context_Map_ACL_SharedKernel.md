# REG001 – Context Map: ACL & Shared Kernel

> **Service Code:** REG001  
> **Service Name (EN):** Issuing Temporary Registration Certificate for a Ship or Marine Unit  
> **Service Name (AR):** إصدار شهادة تسجيل سفينة أو وحدة بحرية مؤقتة  
> **Document Type:** Domain Model – Context Map ACL & Shared Kernel  
> **Version:** 1.0  
> **Last Updated:** 2026-01-08

---

## 1. Purpose / الغرض

This document details the **integration patterns** between bounded contexts, focusing on:

- Anti-Corruption Layers (ACL) for external system isolation
- Shared Kernel for internal shared concepts
- Open Host Services (OHS) for published APIs
- Translation and adaptation strategies

---

## 2. Integration Pattern Summary

### 2.1 Pattern Selection Criteria

| Pattern | When to Use | Trade-offs |
|---------|-------------|------------|
| **Anti-Corruption Layer (ACL)** | External systems with unstable or incompatible models | Higher complexity, better isolation |
| **Shared Kernel** | Internal contexts with high cohesion and shared concepts | Tight coupling, easier consistency |
| **Open Host Service (OHS)** | Well-defined public API for multiple consumers | Contract maintenance overhead |
| **Published Language** | Event-driven integration with standardized events | Eventual consistency |
| **Conformist** | Internal contexts where upstream model is acceptable | Low flexibility |

### 2.2 Pattern Distribution

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                           INTEGRATION PATTERNS - REG001                                              │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                      │
│   EXTERNAL INTEGRATIONS (ACL Required)                                                              │
│   ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│   │                                                                                              │   │
│   │   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐                   │   │
│   │   │ ROP PKI │    │ Invest  │    │  Bayan  │    │  MAFWR  │    │ Payment │                   │   │
│   │   │         │    │  Easy   │    │ Customs │    │         │    │ Gateway │                   │   │
│   │   └────┬────┘    └────┬────┘    └────┬────┘    └────┬────┘    └────┬────┘                   │   │
│   │        │              │              │              │              │                         │   │
│   │        ▼              ▼              ▼              ▼              ▼                         │   │
│   │   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐                   │   │
│   │   │   ACL   │    │   ACL   │    │   ACL   │    │   ACL   │    │   ACL   │                   │   │
│   │   │Identity │    │Eligibil.│    │ Customs │    │Approval │    │ Payment │                   │   │
│   │   │ Adapter │    │ Adapter │    │ Adapter │    │ Adapter │    │ Adapter │                   │   │
│   │   └─────────┘    └─────────┘    └─────────┘    └─────────┘    └─────────┘                   │   │
│   │                                                                                              │   │
│   └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                                      │
│   INTERNAL INTEGRATIONS (Shared Kernel / Conformist)                                                │
│   ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│   │                                                                                              │   │
│   │   ┌─────────────────────────────────────────────────────────────────────────────────────┐   │   │
│   │   │                           SHARED KERNEL                                              │   │   │
│   │   │                                                                                      │   │   │
│   │   │   Common Value Objects: RequestId, VesselId, ApplicantId, Money, Email, etc.        │   │   │
│   │   │   Common Events: Domain event schemas                                                │   │   │
│   │   │   Common Enums: Status codes, categories                                             │   │   │
│   │   │                                                                                      │   │   │
│   │   └─────────────────────────────────────────────────────────────────────────────────────┘   │   │
│   │                                                                                              │   │
│   │   ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐                   │   │
│   │   │BC-VESSEL│◄──►│ BC-REG  │◄──►│BC-BILLING│◄──►│BC-POLICY│◄──►│ BC-DOCS │                   │   │
│   │   │Conformist│   │  Core   │    │Conformist│    │  OHS    │    │Conformist│                  │   │
│   │   └─────────┘    └─────────┘    └─────────┘    └─────────┘    └─────────┘                   │   │
│   │                                                                                              │   │
│   └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                                      │
│   EVENT-DRIVEN INTEGRATIONS (Published Language)                                                    │
│   ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│   │                                                                                              │   │
│   │   BC-REG ─────► BC-NOTIFY (StatusChangeEvent, CertificateIssuedEvent)                       │   │
│   │   BC-REG ─────► BC-AUDIT (AllDomainEvents)                                                  │   │
│   │   BC-BILLING ──► BC-NOTIFY (PaymentConfirmedEvent, InvoiceGeneratedEvent)                   │   │
│   │   BC-MONITOR ──► BC-NOTIFY (ReminderDueEvent, OverdueEvent)                                 │   │
│   │                                                                                              │   │
│   └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Anti-Corruption Layers (ACL)

### 3.1 ROP PKI Identity Adapter

**Purpose:** Isolate the internal identity model from ROP PKI specifics.

```
┌─────────────────────┐                ┌─────────────────────┐
│                     │                │                     │
│      ROP PKI        │   ◄─────────►  │   Identity Adapter  │
│   (External Auth)   │                │       (ACL)         │
│                     │                │                     │
└─────────────────────┘                └──────────┬──────────┘
                                                  │
                                                  ▼
                                       ┌─────────────────────┐
                                       │                     │
                                       │    BC-IDENTITY      │
                                       │  (Internal Model)   │
                                       │                     │
                                       └─────────────────────┘
```

#### Adapter Contract

| External Concept | Internal Concept | Translation |
|------------------|------------------|-------------|
| PKI Certificate | IdentitySession | Extract user claims, create session |
| PKI Validation Response | AuthenticationResult | Map to success/failure with reason |
| PKI Token | SessionToken | Wrap and validate internally |
| PKI User ID | ApplicantId | Map to internal identifier |

#### Adapter Interface

```
IdentityAdapter:
  + authenticate(pkiToken: String): AuthenticationResult
  + validateSession(sessionId: SessionId): SessionValidation
  + refreshSession(sessionId: SessionId): SessionToken
  + terminateSession(sessionId: SessionId): void

AuthenticationResult:
  - isAuthenticated: Boolean
  - applicantId: ApplicantId (nullable)
  - sessionToken: SessionToken (nullable)
  - errorCode: String (nullable)
  - errorMessage: String (nullable)
```

#### Error Mapping

| PKI Error | Internal Error | Action |
|-----------|----------------|--------|
| CERT_EXPIRED | SESSION_EXPIRED | Redirect to re-authentication |
| CERT_REVOKED | SESSION_INVALID | Terminate and notify |
| NETWORK_ERROR | EXTERNAL_UNAVAILABLE | Retry with backoff |
| UNKNOWN_ERROR | AUTHENTICATION_FAILED | Log and reject |

---

### 3.2 Invest Easy Eligibility Adapter

**Purpose:** Isolate CR validation and activity eligibility from Invest Easy API.

```
┌─────────────────────┐                ┌─────────────────────┐
│                     │                │                     │
│    Invest Easy      │   ◄─────────►  │  Eligibility Adapter │
│   (CR/Activity)     │                │       (ACL)         │
│                     │                │                     │
└─────────────────────┘                └──────────┬──────────┘
                                                  │
                                                  ▼
                                       ┌─────────────────────┐
                                       │                     │
                                       │   BC-ELIGIBLE       │
                                       │  (Internal Model)   │
                                       │                     │
                                       └─────────────────────┘
```

#### Adapter Contract

| External Concept | Internal Concept | Translation |
|------------------|------------------|-------------|
| CR Number | CompanyRegistration | Validate format, create reference |
| Activity Code | ActivityType | Map to internal classification |
| Company Status | EligibilityStatus | ACTIVE, SUSPENDED, INACTIVE |
| CR Response | CRValidationResult | Extract relevant fields |

#### Adapter Interface

```
EligibilityAdapter:
  + validateCR(crNumber: String): CRValidationResult
  + checkActivityEligibility(crNumber: String, activityType: ActivityType): ActivityEligibilityResult
  + getCompanyProfile(crNumber: String): CompanyProfileResult

CRValidationResult:
  - isValid: Boolean
  - companyName: String
  - companyNameAr: String
  - status: CompanyStatus
  - expiryDate: Date
  - errorCode: String (nullable)
  - errorMessage: String (nullable)

ActivityEligibilityResult:
  - isEligible: Boolean
  - activityCode: String
  - activityDescription: String
  - reason: String (if not eligible)
```

#### Error Mapping

| Invest Easy Error | Internal Error | Action |
|-------------------|----------------|--------|
| CR_NOT_FOUND | COMPANY_NOT_REGISTERED | Reject request |
| CR_EXPIRED | COMPANY_REGISTRATION_EXPIRED | Reject with renewal guidance |
| ACTIVITY_MISMATCH | ACTIVITY_NOT_ELIGIBLE | Reject with reason |
| SERVICE_UNAVAILABLE | EXTERNAL_UNAVAILABLE | Retry with backoff |

---

### 3.3 Bayan Customs Adapter

**Purpose:** Isolate customs clearance flow from Bayan API specifics.

```
┌─────────────────────┐                ┌─────────────────────┐
│                     │                │                     │
│   Bayan Customs     │   ◄─────────►  │   Customs Adapter   │
│   (External API)    │                │       (ACL)         │
│                     │                │                     │
└─────────────────────┘                └──────────┬──────────┘
                                                  │
                                                  ▼
                                       ┌─────────────────────┐
                                       │                     │
                                       │    BC-CUSTOMS       │
                                       │  (Internal Model)   │
                                       │                     │
                                       └─────────────────────┘
```

#### Adapter Contract

| External Concept | Internal Concept | Translation |
|------------------|------------------|-------------|
| Declaration Number | ClearanceReference | Unique identifier |
| Clearance Status | ClearanceDecision | CLEARED, PENDING, REJECTED |
| Import Document | ImportCertificate | Stored document reference |
| Bayan Response | CustomsClearanceResult | Extract decision and documents |

#### Adapter Interface

```
CustomsAdapter:
  + requestClearance(request: ClearanceRequest): ClearanceInitiationResult
  + checkClearanceStatus(reference: ClearanceReference): ClearanceStatusResult
  + getImportDocument(reference: ClearanceReference): ImportDocumentResult

ClearanceRequest:
  - applicantId: ApplicantId
  - vesselDetails: VesselSummary
  - importDetails: ImportDetails

ClearanceStatusResult:
  - reference: ClearanceReference
  - status: ClearanceStatus
  - decision: ClearanceDecision (nullable)
  - documentUrl: String (nullable)
  - rejectionReason: String (nullable)
```

---

### 3.4 MAFWR Approval Adapter

**Purpose:** Isolate fishing vessel approval flow from MAFWR API.

```
┌─────────────────────┐                ┌─────────────────────┐
│                     │                │                     │
│       MAFWR         │   ◄─────────►  │   Approval Adapter  │
│   (Fishing Auth)    │                │       (ACL)         │
│                     │                │                     │
└─────────────────────┘                └──────────┬──────────┘
                                                  │
                                                  ▼
                                       ┌─────────────────────┐
                                       │                     │
                                       │   BC-APPROVAL       │
                                       │  (Internal Model)   │
                                       │                     │
                                       └─────────────────────┘
```

#### Adapter Contract

| External Concept | Internal Concept | Translation |
|------------------|------------------|-------------|
| MAFWR Request ID | ApprovalReference | Tracking identifier |
| Approval Status | ApprovalDecision | APPROVED, REJECTED, PENDING |
| MAFWR Response | ApprovalResult | Extract decision and reason |
| Fishing License | FishingAuthorization | License details if approved |

#### Adapter Interface

```
ApprovalAdapter:
  + requestApproval(request: ApprovalRequest): ApprovalInitiationResult
  + checkApprovalStatus(reference: ApprovalReference): ApprovalStatusResult
  + cancelApprovalRequest(reference: ApprovalReference): CancellationResult

ApprovalRequest:
  - requestId: RequestId
  - applicantId: ApplicantId
  - vesselDetails: VesselSummary
  - activityDetails: FishingActivityDetails

ApprovalStatusResult:
  - reference: ApprovalReference
  - status: ApprovalStatus
  - decision: ApprovalDecision (nullable)
  - reason: String (nullable)
  - decidedAt: DateTime (nullable)
```

---

### 3.5 Payment Gateway Adapter

**Purpose:** Isolate payment processing from payment gateway specifics.

```
┌─────────────────────┐                ┌─────────────────────┐
│                     │                │                     │
│  Payment Gateway    │   ◄─────────►  │   Payment Adapter   │
│  (e-Payment)        │                │       (ACL)         │
│                     │                │                     │
└─────────────────────┘                └──────────┬──────────┘
                                                  │
                                                  ▼
                                       ┌─────────────────────┐
                                       │                     │
                                       │   BC-BILLING        │
                                       │  (Internal Model)   │
                                       │                     │
                                       └─────────────────────┘
```

#### Adapter Contract

| External Concept | Internal Concept | Translation |
|------------------|------------------|-------------|
| Transaction ID | PaymentReference | Gateway transaction reference |
| Payment Status | PaymentStatus | SUCCESS, FAILED, PENDING, CANCELLED |
| Gateway Response | PaymentResult | Extract status and details |
| Payment URL | PaymentRedirect | Redirect URL for applicant |

#### Adapter Interface

```
PaymentAdapter:
  + initiatePayment(request: PaymentRequest): PaymentInitiationResult
  + checkPaymentStatus(reference: PaymentReference): PaymentStatusResult
  + processCallback(callback: PaymentCallback): PaymentConfirmation
  + cancelPayment(reference: PaymentReference): CancellationResult

PaymentRequest:
  - invoiceId: InvoiceId
  - amount: Money
  - description: String
  - returnUrl: String
  - notifyUrl: String

PaymentConfirmation:
  - reference: PaymentReference
  - invoiceId: InvoiceId
  - status: PaymentStatus
  - amount: Money
  - paidAt: DateTime
  - receiptNumber: String
```

---

### 3.6 Inspection Entity Adapter

**Purpose:** Isolate inspection coordination from classification societies/consultancies.

```
┌─────────────────────┐                ┌─────────────────────┐
│                     │                │                     │
│ Inspection Entity   │   ◄─────────►  │  Inspection Adapter │
│ (Classification)    │                │       (ACL)         │
│                     │                │                     │
└─────────────────────┘                └──────────┬──────────┘
                                                  │
                                                  ▼
                                       ┌─────────────────────┐
                                       │                     │
                                       │   BC-INSPECT        │
                                       │  (Internal Model)   │
                                       │                     │
                                       └─────────────────────┘
```

#### Adapter Interface

```
InspectionAdapter:
  + requestInspection(request: InspectionRequest): InspectionInitiationResult
  + checkInspectionStatus(reference: InspectionReference): InspectionStatusResult
  + getInspectionReport(reference: InspectionReference): InspectionReportResult
  + cancelInspection(reference: InspectionReference): CancellationResult

InspectionRequest:
  - requestId: RequestId
  - vesselDetails: VesselSummary
  - inspectionType: InspectionType
  - preferredDate: Date (optional)
  - location: InspectionLocation

InspectionReportResult:
  - reference: InspectionReference
  - outcome: InspectionOutcome
  - reportUrl: String
  - findings: List<InspectionFinding>
  - completedAt: DateTime
```

---

## 4. Shared Kernel

### 4.1 Shared Kernel Overview

The Shared Kernel contains common concepts used across multiple bounded contexts. Changes to the Shared Kernel require coordination between all dependent teams.

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                         SHARED KERNEL                                                │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                      │
│   ┌─────────────────────────────────┐   ┌─────────────────────────────────┐                         │
│   │       IDENTITY VALUE OBJECTS    │   │      DOMAIN VALUE OBJECTS       │                         │
│   │                                 │   │                                 │                         │
│   │   RequestId                     │   │   Money                         │                         │
│   │   VesselId                      │   │   Email                         │                         │
│   │   ApplicantId                   │   │   PhoneNumber                   │                         │
│   │   InvoiceId                     │   │   Address                       │                         │
│   │   CertificateId                 │   │   DateRange                     │                         │
│   │   DocumentId                    │   │   ContactInfo                   │                         │
│   │                                 │   │                                 │                         │
│   └─────────────────────────────────┘   └─────────────────────────────────┘                         │
│                                                                                                      │
│   ┌─────────────────────────────────┐   ┌─────────────────────────────────┐                         │
│   │       COMMON ENUMERATIONS       │   │       DOMAIN EVENTS             │                         │
│   │                                 │   │                                 │                         │
│   │   VesselCategory                │   │   DomainEvent (base)            │                         │
│   │   BuildMaterial                 │   │   - eventId                     │                         │
│   │   AcquisitionPath               │   │   - eventType                   │                         │
│   │   ActivityType                  │   │   - timestamp                   │                         │
│   │   ApplicantType                 │   │   - aggregateId                 │                         │
│   │   Currency (OMR default)        │   │   - correlationId               │                         │
│   │                                 │   │   - payload                     │                         │
│   └─────────────────────────────────┘   └─────────────────────────────────┘                         │
│                                                                                                      │
│   ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│   │                                    COMMON CONTRACTS                                          │   │
│   │                                                                                              │   │
│   │   Result<T, E>       - Success/Failure pattern for operations                               │   │
│   │   Page<T>            - Paginated response wrapper                                            │   │
│   │   AuditInfo          - Created/Updated timestamps and users                                  │   │
│   │   LocalizedText      - Text with EN/AR translations                                          │   │
│   │                                                                                              │   │
│   └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 4.2 Shared Value Objects

#### 4.2.1 Identity Value Objects

```
RequestId:
  - value: UUID
  + toString(): String (REG001-YYYY-NNNNN format)
  + equals(other: RequestId): Boolean

VesselId:
  - value: UUID
  + equals(other: VesselId): Boolean

ApplicantId:
  - value: UUID
  + equals(other: ApplicantId): Boolean

InvoiceId:
  - value: UUID
  + toString(): String (INV-YYYY-NNNNN format)
  + equals(other: InvoiceId): Boolean

CertificateId:
  - value: UUID
  + toString(): String (CERT-YYYY-NNNNN format)
  + equals(other: CertificateId): Boolean
```

#### 4.2.2 Domain Value Objects

```
Money:
  - amount: Decimal
  - currency: Currency
  + add(other: Money): Money
  + subtract(other: Money): Money
  + isPositive(): Boolean
  + isZero(): Boolean
  + equals(other: Money): Boolean
  
  Invariants:
    - Same currency for arithmetic operations
    - Amount must be non-negative for most uses

Email:
  - value: String
  + isValid(): Boolean
  + getDomain(): String
  + equals(other: Email): Boolean
  
  Invariants:
    - Must match email regex pattern
    - Cannot be empty

PhoneNumber:
  - countryCode: String
  - number: String
  + format(): String
  + equals(other: PhoneNumber): Boolean
  
  Invariants:
    - CountryCode must be valid
    - Number must contain only digits

LocalizedText:
  - en: String
  - ar: String
  + get(locale: Locale): String
  + isEmpty(): Boolean
```

### 4.3 Shared Enumerations

```
VesselCategory:
  CARGO, TANKER, CONTAINER, BULK_CARRIER, PASSENGER, FISHING, 
  NAVAL, RESEARCH, TUGBOAT, OFFSHORE, YACHT, FERRY, BOAT, OTHER

BuildMaterial:
  STEEL, ALUMINUM, FIBERGLASS, WOOD, COMPOSITE, OTHER

AcquisitionPath:
  NEW_BUILD, PURCHASE, FLAG_CHANGE

ActivityType:
  COMMERCIAL, FISHING, TOURISM, SERVICE, PRIVATE, OTHER

ApplicantType:
  INDIVIDUAL, COMPANY

Currency:
  OMR (default)
```

### 4.4 Shared Event Schema

```
DomainEvent (Base):
  - eventId: UUID
  - eventType: String
  - aggregateType: String
  - aggregateId: String
  - timestamp: DateTime
  - correlationId: UUID
  - causationId: UUID
  - metadata: Map<String, String>
  - payload: Object

Event Naming Convention:
  - {Aggregate}{Action}Event
  - Examples:
    - RegistrationRequestSubmittedEvent
    - PaymentConfirmedEvent
    - CertificateIssuedEvent
```

### 4.5 Shared Kernel Governance

| Rule | Description |
|------|-------------|
| **Change Coordination** | All changes require agreement from dependent teams |
| **Version Control** | Semantic versioning with changelog |
| **Deprecation Policy** | Minimum 2 sprint notice for breaking changes |
| **Testing** | Shared test suite maintained by all teams |
| **Documentation** | API documentation auto-generated |

---

## 5. Open Host Services (OHS)

### 5.1 Policy Service (BC-POLICY)

**Purpose:** Provide DMN decisions and policy rules to consuming contexts.

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                    POLICY SERVICE (OHS)                                              │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                      │
│   Consumers: BC-REG, BC-VESSEL, BC-DOCS, BC-BILLING, BC-APPROVAL, BC-INSPECT, BC-CUSTOMS            │
│                                                                                                      │
│   ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│   │                                   PUBLIC API                                                 │   │
│   ├─────────────────────────────────────────────────────────────────────────────────────────────┤   │
│   │                                                                                              │   │
│   │   /api/v1/policy/vessel-age-validation                                                      │   │
│   │     POST - Validate vessel age against policy                                               │   │
│   │     Input: { vesselCategory, buildYear, buildMaterial }                                     │   │
│   │     Output: { isValid, maxAllowedAge, message }                                             │   │
│   │                                                                                              │   │
│   │   /api/v1/policy/documents-required                                                         │   │
│   │     POST - Get required documents for request                                               │   │
│   │     Input: { vesselCategory, acquisitionPath, activityType }                                │   │
│   │     Output: { documents: [{ code, name, mandatory }] }                                      │   │
│   │                                                                                              │   │
│   │   /api/v1/policy/fee-calculation                                                            │   │
│   │     POST - Calculate fees for registration                                                  │   │
│   │     Input: { vesselCategory, grossTonnage, length, activityType }                           │   │
│   │     Output: { baseFee, tonnageFee, lengthFee, totalFee }                                    │   │
│   │                                                                                              │   │
│   │   /api/v1/policy/penalty-calculation                                                        │   │
│   │     POST - Calculate applicable penalties                                                   │   │
│   │     Input: { constructionEndDate, submissionDate }                                          │   │
│   │     Output: { penaltyApplies, amount, reason }                                              │   │
│   │                                                                                              │   │
│   │   /api/v1/policy/inspection-required                                                        │   │
│   │     POST - Determine if inspection is required                                              │   │
│   │     Input: { vesselCategory, length, constructionPhase }                                    │   │
│   │     Output: { isRequired, inspectionType, reason }                                          │   │
│   │                                                                                              │   │
│   │   /api/v1/policy/mafwr-required                                                             │   │
│   │     POST - Determine if MAFWR approval is required                                          │   │
│   │     Input: { vesselCategory, activityType }                                                 │   │
│   │     Output: { isRequired, reason }                                                          │   │
│   │                                                                                              │   │
│   │   /api/v1/policy/customs-required                                                           │   │
│   │     POST - Determine if customs clearance is required                                       │   │
│   │     Input: { acquisitionPath, vesselOrigin }                                                │   │
│   │     Output: { isRequired, reason }                                                          │   │
│   │                                                                                              │   │
│   └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                                      │
│   Implementation: Flowable DMN Engine                                                               │
│   DMN Tables: reg001-*.dmn                                                                          │
│                                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 Notification Service (BC-NOTIFY)

**Purpose:** Provide multi-channel notification delivery to all contexts.

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                  NOTIFICATION SERVICE (OHS)                                          │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                      │
│   Consumers: BC-REG, BC-BILLING, BC-MONITOR, BC-APPROVAL, BC-INSPECT                                │
│                                                                                                      │
│   ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│   │                                   PUBLIC API                                                 │   │
│   ├─────────────────────────────────────────────────────────────────────────────────────────────┤   │
│   │                                                                                              │   │
│   │   /api/v1/notifications                                                                     │   │
│   │     POST - Send notification                                                                │   │
│   │     Input: { recipientId, channel, templateCode, parameters }                               │   │
│   │     Output: { notificationId, status }                                                      │   │
│   │                                                                                              │   │
│   │   /api/v1/notifications/{id}/status                                                         │   │
│   │     GET - Check notification delivery status                                                │   │
│   │     Output: { notificationId, channel, status, deliveredAt }                                │   │
│   │                                                                                              │   │
│   │   Channels: SMS, EMAIL, PORTAL, PUSH                                                        │   │
│   │                                                                                              │   │
│   └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                                      │
│   Event Subscription: Consumes domain events for automatic notifications                            │
│                                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Published Language (Events)

### 6.1 Event Catalog

| Event | Publisher | Subscribers | Purpose |
|-------|-----------|-------------|---------|
| `RegistrationRequestCreated` | BC-REG | BC-AUDIT | Track request initiation |
| `RegistrationRequestSubmitted` | BC-REG | BC-NOTIFY, BC-AUDIT | Trigger submission notification |
| `DocumentsCompleted` | BC-DOCS | BC-REG, BC-AUDIT | Signal document completeness |
| `ApprovalRequested` | BC-REG | BC-APPROVAL, BC-AUDIT | Request external approval |
| `ApprovalDecisionReceived` | BC-APPROVAL | BC-REG, BC-NOTIFY, BC-AUDIT | Notify approval outcome |
| `InspectionRequested` | BC-REG | BC-INSPECT, BC-AUDIT | Request inspection |
| `InspectionCompleted` | BC-INSPECT | BC-REG, BC-NOTIFY, BC-AUDIT | Notify inspection result |
| `PaymentInitiated` | BC-REG | BC-BILLING, BC-AUDIT | Start payment process |
| `PaymentConfirmed` | BC-BILLING | BC-REG, BC-NOTIFY, BC-AUDIT | Confirm payment success |
| `PaymentFailed` | BC-BILLING | BC-REG, BC-NOTIFY, BC-AUDIT | Notify payment failure |
| `CertificateIssued` | BC-REG | BC-MONITOR, BC-NOTIFY, BC-AUDIT | Trigger post-issuance monitoring |
| `ReminderDue` | BC-MONITOR | BC-NOTIFY | Send reminder notification |
| `OverduePenaltyApplied` | BC-MONITOR | BC-BILLING, BC-NOTIFY, BC-AUDIT | Apply overdue penalty |

### 6.2 Event Schema Examples

```json
// RegistrationRequestSubmitted Event
{
  "eventId": "uuid",
  "eventType": "RegistrationRequestSubmitted",
  "aggregateType": "RegistrationRequest",
  "aggregateId": "REG001-2026-00001",
  "timestamp": "2026-01-08T10:30:00Z",
  "correlationId": "uuid",
  "payload": {
    "requestId": "REG001-2026-00001",
    "applicantId": "uuid",
    "applicantName": "Company ABC",
    "vesselName": "Sea Explorer",
    "vesselCategory": "CARGO"
  }
}

// CertificateIssued Event
{
  "eventId": "uuid",
  "eventType": "CertificateIssued",
  "aggregateType": "RegistrationRequest",
  "aggregateId": "REG001-2026-00001",
  "timestamp": "2026-01-08T14:00:00Z",
  "correlationId": "uuid",
  "payload": {
    "requestId": "REG001-2026-00001",
    "certificateId": "CERT-2026-00001",
    "certificateNumber": "TRC-2026-00001",
    "vesselName": "Sea Explorer",
    "issuedDate": "2026-01-08",
    "expiryDate": "2026-07-08"
  }
}
```

---

## 7. Integration Patterns Summary

| Integration | Pattern | Adapter/Contract | Event-Driven |
|-------------|---------|------------------|--------------|
| ROP PKI → BC-IDENTITY | ACL | IdentityAdapter | No |
| Invest Easy → BC-ELIGIBLE | ACL | EligibilityAdapter | No |
| MAFWR → BC-APPROVAL | ACL | ApprovalAdapter | Yes (callbacks) |
| Bayan → BC-CUSTOMS | ACL | CustomsAdapter | Yes (callbacks) |
| Payment Gateway → BC-BILLING | ACL | PaymentAdapter | Yes (webhooks) |
| Inspection Entity → BC-INSPECT | ACL | InspectionAdapter | Yes (callbacks) |
| BC-POLICY → All consumers | OHS | REST API | No |
| BC-NOTIFY → All consumers | OHS | REST API + Events | Yes |
| BC-* → BC-AUDIT | Published Language | Event Bus | Yes |

---

## 8. Related Documents

- [01-UML_Domain_Structure](01-UML_Domain_Structure.md) – Aggregate and entity definitions
- [02-Business_Domain_Context_Map](02-Business_Domain_Context_Map.md) – Bounded context overview
- [04-Integration_Architecture](../2.2-architecture/04-Integration_Architecture.md) – Technical integration design

---

**Document Control**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-08 | Solution Architect | Initial ACL and Shared Kernel documentation |
