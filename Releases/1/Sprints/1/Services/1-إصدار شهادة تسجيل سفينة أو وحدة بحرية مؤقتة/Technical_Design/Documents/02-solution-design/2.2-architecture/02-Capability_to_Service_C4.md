# REG001 – Capability to Service C4 Mapping

> **Service Code:** REG001  
> **Service Name (EN):** Issuing Temporary Registration Certificate for a Ship or Marine Unit  
> **Service Name (AR):** إصدار شهادة تسجيل سفينة أو وحدة بحرية مؤقتة  
> **Document Type:** Service Decomposition  
> **Version:** 1.0  
> **Last Updated:** 2026-01-11

---

## 1. Purpose / الغرض

This document provides a **detailed mapping** from business capabilities to microservices (containers), showing:
- Service responsibilities and boundaries
- Capability ownership
- Inter-service dependencies
- API contract indicators

---

## 2. Service Catalog

### 2.1 Core Services

| Service ID | Service Name | Bounded Context | Category |
|------------|--------------|-----------------|----------|
| LS-REG-ORCH | TemporaryRegistrationProcessService | BC-REG | Process/Orchestration |
| LS-VESSEL | VesselProfileService | BC-VESSEL | Domain |
| LS-APPLICANT | ApplicantProfileService | BC-APPLICANT | Domain |
| LS-DOCS | DocumentManagementService | BC-DOCS | Domain |
| LS-BILLING | BillingService | BC-BILLING | Domain |
| LS-CERT | TemporaryCertificateService | BC-CERT | Domain |

### 2.2 Policy & Data Services

| Service ID | Service Name | Bounded Context | Category |
|------------|--------------|-----------------|----------|
| LS-POLICY | RegistrationPolicyService | BC-REGISTRATION-POLICY | Decision |
| LS-MASTERDATA | MasterDataService | BC-MASTERDATA | Reference Data |

### 2.3 Supporting Services

| Service ID | Service Name | Bounded Context | Category |
|------------|--------------|-----------------|----------|
| LS-NOTIFY | NotificationService | BC-NOTIFY | Platform |
| LS-AUDIT | AuditTrailService | BC-AUDIT | Platform |
| LS-MONITOR | PostIssuanceMonitoringService | BC-MONITOR | Domain |

### 2.4 Integration Adapters

| Service ID | Service Name | External System | Category |
|------------|--------------|-----------------|----------|
| LS-IAM | IdentityAdapter | ROP PKI | ACL |
| LS-CR-ADAPTER | EligibilityAdapter | Invest Easy | ACL |
| LS-APPROVALS | ExternalApprovalAdapter | MAFWR | ACL |
| LS-INSPECTION | InspectionAdapter | Classification Societies | ACL |
| LS-CUSTOMS | CustomsAdapter | Bayan | ACL |
| LS-PAYMENTS | PaymentAdapter | ePayment Gateway | ACL |

---

## 3. Service Responsibility Matrix

### 3.1 TemporaryRegistrationProcessService (LS-REG-ORCH)

**Category:** Process/Orchestration  
**Bounded Context:** BC-REG  
**Primary Aggregate:** RegistrationRequest

#### Responsibilities

| # | Responsibility | Description |
|---|----------------|-------------|
| 1 | Workflow Orchestration | Manage end-to-end registration process via Flowable |
| 2 | Request Lifecycle | Create, update, and track registration requests |
| 3 | State Management | Maintain workflow state transitions |
| 4 | Event Coordination | Publish domain events for cross-service communication |
| 5 | Approval Coordination | Orchestrate MAFWR, Inspection, Customs approvals |

#### Owned Capabilities

| Cap ID | Capability | Description |
|--------|------------|-------------|
| CAP-01 | Service discovery & start request | Allow applicant to initiate registration |

#### Consumed Services

| Service | Purpose | Sync/Async |
|---------|---------|------------|
| LS-POLICY | Decision evaluation (age, docs, fees, etc.) | Sync |
| LS-VESSEL | Vessel data lookup, name reservation | Sync |
| LS-APPLICANT | Applicant profile lookup | Sync |
| LS-DOCS | Document completeness check | Sync |
| LS-BILLING | Invoice and payment status | Sync |
| LS-CERT | Certificate issuance | Sync |
| LS-NOTIFY | Notifications | Async |
| LS-AUDIT | Audit logging | Async |

#### State Transitions

```
DRAFT → SUBMITTED → VALIDATING → AWAITING_APPROVAL → 
AWAITING_INSPECTION → AWAITING_CUSTOMS → AWAITING_PAYMENT → 
PAYMENT_CONFIRMED → ISSUED → (COMPLETED | EXPIRED)
```

---

### 3.2 VesselProfileService (LS-VESSEL)

**Category:** Domain  
**Bounded Context:** BC-VESSEL  
**Primary Aggregate:** VesselProfile

#### Responsibilities

| # | Responsibility | Description |
|---|----------------|-------------|
| 1 | Vessel Data Management | CRUD operations for vessel master data |
| 2 | Name Reservation | Reserve, confirm, and release vessel names |
| 3 | Vessel History | Track ownership and flag changes |
| 4 | Data Validation | Validate vessel dimensions and attributes |

#### Owned Capabilities

| Cap ID | Capability | Description |
|--------|------------|-------------|
| CAP-06 | Vessel/unit data capture | Capture vessel master data |
| CAP-09 | Construction/purchase path handling | Track acquisition path |
| CAP-19 | Name reservation | Reserve vessel name during payment |

#### API Contract Summary

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/vessels` | POST | Create vessel profile |
| `/vessels/{id}` | GET | Retrieve vessel profile |
| `/vessels/{id}` | PUT | Update vessel profile |
| `/vessels/{id}/validate` | POST | Validate vessel data |
| `/vessels/names/reserve` | POST | Reserve vessel name |
| `/vessels/names/{reservationId}/confirm` | POST | Confirm name reservation |
| `/vessels/names/{reservationId}/release` | POST | Release name reservation |

---

### 3.3 ApplicantProfileService (LS-APPLICANT)

**Category:** Domain  
**Bounded Context:** BC-APPLICANT  
**Primary Aggregate:** ApplicantProfile

#### Responsibilities

| # | Responsibility | Description |
|---|----------------|-------------|
| 1 | Applicant Management | CRUD for individual/company profiles |
| 2 | Agent Management | Manage authorized representatives |
| 3 | Profile Verification | Track verification status from ROP/Invest Easy |

#### Owned Capabilities

| Cap ID | Capability | Description |
|--------|------------|-------------|
| CAP-03 | Applicant profiling | Capture beneficiary type and identifiers |

#### API Contract Summary

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/applicants` | POST | Create applicant profile |
| `/applicants/{id}` | GET | Retrieve applicant profile |
| `/applicants/{id}` | PUT | Update applicant profile |
| `/applicants/{id}/agents` | GET | List authorized agents |
| `/applicants/{id}/agents` | POST | Add authorized agent |
| `/applicants/{id}/verify` | POST | Trigger verification |

---

### 3.4 RegistrationPolicyService (LS-POLICY)

**Category:** Decision  
**Bounded Context:** BC-REGISTRATION-POLICY  
**Primary Aggregate:** None (Stateless Decision Service)

#### Responsibilities

| # | Responsibility | Description |
|---|----------------|-------------|
| 1 | DMN Evaluation | Execute DMN decision tables |
| 2 | Configuration Management | Manage CONF-xx settings |
| 3 | Rule Versioning | Manage decision table versions |
| 4 | Policy API | Expose decision evaluation APIs |

#### Owned Capabilities

| Cap ID | Capability | Description |
|--------|------------|-------------|
| CAP-07 | Vessel age policy validation | Evaluate vessel age compliance |
| CAP-08 | Prohibited/banned vessel check | Check prohibited vessels list |
| CAP-10 | Penalty trigger evaluation | Calculate timing-based penalties |
| CAP-11 | Determine required documents | Dynamic document requirements |
| CAP-14 | Inspection requirement decision | Determine if inspection needed |
| CAP-16 | Customs/Bayan requirement decision | Determine if customs needed |
| CAP-18 | Fee calculation | Calculate registration fees |
| CAP-25 | Overdue penalties | Calculate post-issuance penalties |

#### DMN Decision Tables

| DMN File | Decision Key | Purpose |
|----------|--------------|---------|
| `reg001-vessel-age-validation.dmn` | VESSEL_AGE_DECISION | Vessel age policy |
| `reg001-documents-required.dmn` | DOCUMENTS_REQUIRED_DECISION | Required documents |
| `reg001-fee-calculation.dmn` | FEE_CALCULATION_DECISION | Fee calculation |
| `reg001-penalty-calculation.dmn` | PENALTY_CALCULATION_DECISION | Penalty calculation |
| `reg001-inspection-required.dmn` | INSPECTION_REQUIRED_DECISION | Inspection requirement |
| `reg001-mafwr-required.dmn` | MAFWR_REQUIRED_DECISION | MAFWR approval requirement |
| `reg001-customs-required.dmn` | CUSTOMS_REQUIRED_DECISION | Customs requirement |

#### Configuration Codes

| Code | Description | Data Type |
|------|-------------|-----------|
| CONF-01 | Document requirements rules | JSON |
| CONF-02 | Penalty timing (30 days threshold) | Integer |
| CONF-03 | Certificate validity period | Integer (months) |
| CONF-04 | Prohibited vessels list | List<String> |

#### API Contract Summary

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/decisions/vessel-age` | POST | Evaluate vessel age |
| `/decisions/documents-required` | POST | Get required documents |
| `/decisions/fees` | POST | Calculate fees |
| `/decisions/penalties` | POST | Calculate penalties |
| `/decisions/inspection-required` | POST | Check inspection requirement |
| `/decisions/mafwr-required` | POST | Check MAFWR requirement |
| `/decisions/customs-required` | POST | Check customs requirement |
| `/config/{key}` | GET | Get configuration value |
| `/config/{key}` | PUT | Update configuration |

---

### 3.5 MasterDataService (LS-MASTERDATA)

**Category:** Reference Data  
**Bounded Context:** BC-MASTERDATA  
**Primary Aggregate:** None (Lookup Service)

#### Responsibilities

| # | Responsibility | Description |
|---|----------------|-------------|
| 1 | Lookup Tables | Manage SET-xx reference codes |
| 2 | Code Validation | Validate reference code values |
| 3 | Multilingual Support | Provide AR/EN labels |

#### Reference Codes (SET-xx)

| Code | Description | Example Values |
|------|-------------|----------------|
| SET-01 | Vessel data schema | Field definitions |
| SET-02 | Owner data schema | Field definitions |
| SET-03 | Omani ports | Muscat, Sohar, Salalah |
| SET-04 | Vessel types | Cargo, Passenger, Fishing, Tug |
| SET-05 | Activity types | Commercial, Fishing, Pleasure |
| SET-06 | Service fees schedule | Fee matrix |

#### API Contract Summary

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/lookups/{category}` | GET | List lookup values |
| `/lookups/{category}/{code}` | GET | Get specific lookup |
| `/ports` | GET | List Omani ports |
| `/vessel-types` | GET | List vessel types |
| `/activity-types` | GET | List activity types |
| `/fees-schedule` | GET | Get fees schedule |

---

### 3.6 DocumentManagementService (LS-DOCS)

**Category:** Domain  
**Bounded Context:** BC-DOCS  
**Primary Aggregate:** DocumentBundle

#### Responsibilities

| # | Responsibility | Description |
|---|----------------|-------------|
| 1 | Document Upload | Handle file uploads |
| 2 | Completeness Check | Validate document completeness |
| 3 | Metadata Management | Store document metadata |
| 4 | Storage Integration | Integrate with object storage |

#### Owned Capabilities

| Cap ID | Capability | Description |
|--------|------------|-------------|
| CAP-12 | Document upload & completeness | Collect and validate documents |

#### API Contract Summary

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/documents` | POST | Upload document |
| `/documents/{id}` | GET | Retrieve document |
| `/documents/{id}` | DELETE | Delete document |
| `/requests/{requestId}/documents` | GET | List request documents |
| `/requests/{requestId}/documents/completeness` | GET | Check completeness |

---

### 3.7 BillingService (LS-BILLING)

**Category:** Domain  
**Bounded Context:** BC-BILLING  
**Primary Aggregate:** Invoice

#### Responsibilities

| # | Responsibility | Description |
|---|----------------|-------------|
| 1 | Invoice Generation | Create invoices from fees |
| 2 | Payment Tracking | Track payment status |
| 3 | Receipt Generation | Generate payment receipts |
| 4 | Penalty Application | Apply penalty line items |

#### Owned Capabilities

| Cap ID | Capability | Description |
|--------|------------|-------------|
| CAP-20 | Invoice generation | Generate invoice with fees and penalties |
| CAP-21 | Payment processing | Process and confirm payments |

#### API Contract Summary

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/invoices` | POST | Create invoice |
| `/invoices/{id}` | GET | Retrieve invoice |
| `/invoices/{id}/pay` | POST | Initiate payment |
| `/invoices/{id}/confirm` | POST | Confirm payment |
| `/invoices/{id}/cancel` | POST | Cancel invoice |
| `/requests/{requestId}/invoices` | GET | List request invoices |

---

### 3.8 TemporaryCertificateService (LS-CERT)

**Category:** Domain  
**Bounded Context:** BC-CERT  
**Primary Aggregate:** Certificate

#### Responsibilities

| # | Responsibility | Description |
|---|----------------|-------------|
| 1 | Certificate Generation | Generate PDF with QR code |
| 2 | Registry Update | Update vessel registry |
| 3 | Certificate Validation | Validate certificate authenticity |
| 4 | Validity Management | Track certificate validity |

#### Owned Capabilities

| Cap ID | Capability | Description |
|--------|------------|-------------|
| CAP-22 | Certificate issuance | Issue temporary certificate |

#### API Contract Summary

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/certificates` | POST | Issue certificate |
| `/certificates/{id}` | GET | Retrieve certificate |
| `/certificates/{id}/download` | GET | Download PDF |
| `/certificates/{id}/validate` | GET | Validate certificate |
| `/certificates/{id}/qr` | GET | Get QR code |

---

### 3.9 NotificationService (LS-NOTIFY)

**Category:** Platform  
**Bounded Context:** BC-NOTIFY  
**Primary Aggregate:** Notification

#### Responsibilities

| # | Responsibility | Description |
|---|----------------|-------------|
| 1 | Multi-Channel Delivery | SMS, Email, In-App |
| 2 | Template Management | Manage notification templates |
| 3 | Delivery Tracking | Track delivery status |
| 4 | Preference Management | Manage user preferences |

#### Owned Capabilities

| Cap ID | Capability | Description |
|--------|------------|-------------|
| CAP-23 | Notifications | Send status and outcome notifications |

#### API Contract Summary

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/notifications` | POST | Send notification |
| `/notifications/{id}` | GET | Get notification status |
| `/templates` | GET | List templates |
| `/templates/{id}` | GET | Get template |
| `/preferences/{userId}` | GET | Get user preferences |

---

### 3.10 PostIssuanceMonitoringService (LS-MONITOR)

**Category:** Domain  
**Bounded Context:** BC-MONITOR  
**Primary Aggregate:** MonitoringRecord

#### Responsibilities

| # | Responsibility | Description |
|---|----------------|-------------|
| 1 | Reminder Scheduling | Schedule periodic reminders |
| 2 | Deadline Tracking | Track permanent registration deadline |
| 3 | Overdue Detection | Detect overdue registrations |
| 4 | Penalty Triggering | Trigger penalty application |

#### Owned Capabilities

| Cap ID | Capability | Description |
|--------|------------|-------------|
| CAP-24 | Post-issuance reminders | Send periodic reminders |

#### API Contract Summary

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/monitoring/schedules` | POST | Create monitoring schedule |
| `/monitoring/schedules/{id}` | GET | Get schedule status |
| `/monitoring/overdue` | GET | List overdue registrations |
| `/monitoring/reminders/send` | POST | Trigger reminder batch |

---

## 4. Integration Adapters

### 4.1 IdentityAdapter (LS-IAM)

**External System:** ROP PKI  
**Pattern:** Anti-Corruption Layer

#### Responsibilities

| # | Responsibility | Description |
|---|----------------|-------------|
| 1 | PKI Authentication | Authenticate via ROP PKI |
| 2 | Session Management | Manage user sessions |
| 3 | Token Translation | Translate PKI tokens to internal format |

#### Owned Capabilities

| Cap ID | Capability | Description |
|--------|------------|-------------|
| CAP-02 | Identity authentication | Authenticate applicant via PKI |

#### External Contract

| Operation | External API | Internal Translation |
|-----------|--------------|---------------------|
| Authenticate | `ROP/authenticate` | `AuthenticationResult` |
| Validate | `ROP/validate-token` | `SessionValidation` |

---

### 4.2 EligibilityAdapter (LS-CR-ADAPTER)

**External System:** Invest Easy  
**Pattern:** Anti-Corruption Layer

#### Responsibilities

| # | Responsibility | Description |
|---|----------------|-------------|
| 1 | CR Validation | Validate Commercial Registration |
| 2 | Activity Validation | Validate eligible activities |
| 3 | Company Profile | Retrieve company information |

#### Owned Capabilities

| Cap ID | Capability | Description |
|--------|------------|-------------|
| CAP-04 | Commercial Registration validation | Validate company CR |
| CAP-05 | Company activity eligibility | Validate activity for vessel type |

#### External Contract

| Operation | External API | Internal Translation |
|-----------|--------------|---------------------|
| Validate CR | `InvestEasy/cr/validate` | `CRValidationResult` |
| Check Activity | `InvestEasy/activity/check` | `ActivityEligibilityResult` |

---

### 4.3 ExternalApprovalAdapter (LS-APPROVALS)

**External System:** MAFWR  
**Pattern:** Anti-Corruption Layer (Async)

#### Responsibilities

| # | Responsibility | Description |
|---|----------------|-------------|
| 1 | Approval Request | Submit approval requests |
| 2 | Response Handling | Process approval responses |
| 3 | Status Tracking | Track approval status |

#### Owned Capabilities

| Cap ID | Capability | Description |
|--------|------------|-------------|
| CAP-13 | MAFWR approval routing | Request fishing vessel approval |

#### External Contract

| Operation | External API | Internal Translation |
|-----------|--------------|---------------------|
| Request Approval | `MAFWR/approvals` | `ApprovalRequest` |
| Receive Response | Callback/Message | `ApprovalResponse` |

---

### 4.4 InspectionAdapter (LS-INSPECTION)

**External System:** Classification Societies / Inspection Entities  
**Pattern:** Anti-Corruption Layer (Async)

#### Responsibilities

| # | Responsibility | Description |
|---|----------------|-------------|
| 1 | Inspection Request | Submit inspection requests |
| 2 | Result Processing | Process inspection results |
| 3 | Report Storage | Store inspection reports |

#### Owned Capabilities

| Cap ID | Capability | Description |
|--------|------------|-------------|
| CAP-15 | Inspection orchestration | Manage inspection workflow |

---

### 4.5 CustomsAdapter (LS-CUSTOMS)

**External System:** Bayan  
**Pattern:** Anti-Corruption Layer (Async)

#### Responsibilities

| # | Responsibility | Description |
|---|----------------|-------------|
| 1 | Clearance Request | Submit customs clearance requests |
| 2 | Document Receipt | Receive import documents |
| 3 | Clearance Confirmation | Receive final clearance |

#### Owned Capabilities

| Cap ID | Capability | Description |
|--------|------------|-------------|
| CAP-17 | Bayan/customs clearance | Integrate with Bayan for customs |

---

### 4.6 PaymentAdapter (LS-PAYMENTS)

**External System:** ePayment Gateway  
**Pattern:** Anti-Corruption Layer

#### Responsibilities

| # | Responsibility | Description |
|---|----------------|-------------|
| 1 | Payment Initiation | Initiate payment transactions |
| 2 | Payment Confirmation | Handle payment callbacks |
| 3 | Receipt Retrieval | Retrieve payment receipts |

#### Owned Capabilities

| Cap ID | Capability | Description |
|--------|------------|-------------|
| CAP-21 | Payment processing (adapter) | Payment gateway integration |

---

## 5. Service Dependency Matrix

```
┌────────────────┬─────────┬─────────┬─────────┬─────────┬─────────┬─────────┬─────────┬─────────┐
│                │REG-ORCH │ VESSEL  │APPLICANT│ POLICY  │  DOCS   │ BILLING │  CERT   │ NOTIFY  │
├────────────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┤
│ REG-ORCH       │    -    │    ✓    │    ✓    │    ✓    │    ✓    │    ✓    │    ✓    │    ✓    │
│ VESSEL         │         │    -    │         │    ✓    │         │         │         │         │
│ APPLICANT      │         │         │    -    │         │         │         │         │         │
│ POLICY         │         │         │         │    -    │         │         │         │         │
│ DOCS           │         │         │         │    ✓    │    -    │         │         │         │
│ BILLING        │         │         │         │    ✓    │         │    -    │         │    ✓    │
│ CERT           │         │    ✓    │         │         │         │         │    -    │    ✓    │
│ MONITOR        │         │         │         │    ✓    │         │    ✓    │         │    ✓    │
└────────────────┴─────────┴─────────┴─────────┴─────────┴─────────┴─────────┴─────────┴─────────┘

Legend: ✓ = Depends on (sync or async)
```

---

## 6. Related Documents

- [01-C4_Architecture.md](01-C4_Architecture.md) – C4 model diagrams
- [03-Sequence_Diagrams.md](03-Sequence_Diagrams.md) – Service interaction flows
- [04-Integration_Architecture.md](04-Integration_Architecture.md) – External system integrations
- [01-Capability_to_Logical_Service_Mapping.md](../../01-business-analysis/1.4-capabilities/01-Capability_to_Logical_Service_Mapping/01-Capability_to_Logical_Service_Mapping.md) – Business capability mapping

---

**Last Updated:** 2026-01-11  
**Version:** 1.0  
**Author:** Solution Architecture Team
