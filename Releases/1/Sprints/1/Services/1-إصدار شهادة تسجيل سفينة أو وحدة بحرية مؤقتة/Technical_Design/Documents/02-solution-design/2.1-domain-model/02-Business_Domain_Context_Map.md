# REG001 – Business Domain Context Map

> **Service Code:** REG001  
> **Service Name (EN):** Issuing Temporary Registration Certificate for a Ship or Marine Unit  
> **Service Name (AR):** إصدار شهادة تسجيل سفينة أو وحدة بحرية مؤقتة  
> **Document Type:** Domain Model – Business Domain Context Map  
> **Version:** 1.0  
> **Last Updated:** 2026-01-08

---

## 1. Purpose / الغرض

This document defines the **bounded contexts** for REG001 and maps their relationships using DDD strategic patterns. It provides:

- Identification of core, supporting, and generic domains
- Context boundaries and responsibilities
- Upstream/downstream relationships
- Integration patterns between contexts

---

## 2. Domain Classification

### 2.1 Core Domain

The **differentiating** business capability that provides competitive advantage.

| Domain | Description | Rationale |
|--------|-------------|-----------|
| **D01 – Registration (Temporary)** | Manages the temporary registration request lifecycle | Core business process; unique to MTCIT |

### 2.2 Supporting Domains

Domains that **support** the core domain but are not differentiating.

| Domain | Description | Rationale |
|--------|-------------|-----------|
| **D04 – Vessel / Marine Unit** | Master vessel data and attributes | Essential data for registration |
| **D05 – Policy & Decision** | DMN rule evaluation, decision APIs, policy configurations (CONF-xx) | Centralized decision logic + config |
| **D06 – Documents & Records** | Document requirements and management | Supports compliance requirements |
| **D07 – External Approvals** | MAFWR and other authority approvals | Conditional process gates |
| **D08 – Inspection** | Vessel inspection orchestration | Quality and safety assurance |
| **D10 – Billing & Payments** | Invoicing, payment processing (fee CALCULATION is in Policy) | Revenue collection |
| **D12 – Monitoring & Enforcement** | Post-issuance reminders and penalties | Compliance enforcement |
| **D15 – Master Data** | Static lookup tables (SET-xx), reference codes, categories | Reference data for all services |

### 2.3 Generic Domains

Domains that are **commoditized** and can be replaced with off-the-shelf solutions.

| Domain | Description | Rationale |
|--------|-------------|-----------|
| **D02 – Party & Identity** | Authentication and applicant profiles | Common IAM patterns |
| **D03 – Business/Company Eligibility** | CR validation, activity eligibility | Integration with external systems |
| **D09 – Customs & Trade** | Bayan customs integration | External system integration |
| **D11 – Notifications & Communication** | SMS, Email, in-app notifications | Platform capability |
| **D13 – Audit & Governance** | Audit trail and compliance logging | Cross-cutting infrastructure |

---

## 3. Bounded Contexts

### 3.1 Context Catalog

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                  BOUNDED CONTEXTS - REG001                                           │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                      │
│   ┌─────────────────────────────────────────────────────────────────────────────────────────────┐   │
│   │                            CORE CONTEXT: TEMPORARY REGISTRATION                              │   │
│   │                                                                                              │   │
│   │   Owns: Request lifecycle, workflow state, certificate issuance                             │   │
│   │   Language: Request, Submission, Approval, Issuance, Certificate                            │   │
│   │                                                                                              │   │
│   └─────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                                      │
│   ┌────────────────────┐  ┌────────────────────┐  ┌────────────────────┐  ┌────────────────────┐    │
│   │   VESSEL CONTEXT   │  │  APPLICANT CONTEXT │  │  DOCUMENTS CONTEXT │  │   POLICY CONTEXT   │    │
│   │                    │  │                    │  │                    │  │                    │    │
│   │ Owns: Vessel data, │  │ Owns: Applicant    │  │ Owns: Document     │  │ Owns: Business     │    │
│   │ name, dimensions,  │  │ profiles, agent    │  │ requirements,      │  │ rules, thresholds, │    │
│   │ history            │  │ representation     │  │ upload, metadata   │  │ DMN decisions      │    │
│   └────────────────────┘  └────────────────────┘  └────────────────────┘  └────────────────────┘    │
│                                                                                                      │
│   ┌────────────────────┐  ┌────────────────────┐  ┌────────────────────┐  ┌────────────────────┐    │
│   │ MAFWR-APPROVAL CTX │  │ INSPECTION CONTEXT │  │  BILLING CONTEXT   │  │ MONITORING CONTEXT │    │
│   │                    │  │                    │  │                    │  │                    │    │
│   │ Owns: MAFWR flow,  │  │ Owns: Inspection   │  │ Owns: Fees,        │  │ Owns: Reminders,   │    │
│   │ fishing approval   │  │ requests, results  │  │ invoices, payments │  │ deadlines, overdue │    │
│   └────────────────────┘  └────────────────────┘  └────────────────────┘  └────────────────────┘    │
│                                                                                                      │
│   ┌────────────────────┐  ┌────────────────────┐  ┌────────────────────┐  ┌────────────────────┐    │
│   │  IDENTITY CONTEXT  │  │ ELIGIBILITY CONTEXT│  │  CUSTOMS CONTEXT   │  │ NOTIFICATION CTX   │    │
│   │                    │  │                    │  │                    │  │                    │    │
│   │ Owns: Auth, PKI,   │  │ Owns: CR validate, │  │ Owns: Bayan        │  │ Owns: Channels,    │    │
│   │ sessions           │  │ activity check     │  │ integration        │  │ templates, delivery│    │
│   └────────────────────┘  └────────────────────┘  └────────────────────┘  └────────────────────┘    │
│                                                                                                      │
│   ┌──────────────────────────────────────────────────────────────────────────────────────────────┐  │
│   │                           AUDIT CONTEXT (Cross-Cutting)                                       │  │
│   │                                                                                               │  │
│   │   Owns: Decision logging, request trail, compliance reporting                                │  │
│   │   Consumes events from all contexts                                                          │  │
│   │                                                                                               │  │
│   └──────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Context Definitions

| Context ID | Context Name | Domain(s) | Responsibility | Ubiquitous Language |
|------------|--------------|-----------|----------------|---------------------|
| **BC-REG** | Temporary Registration | D01 | Request lifecycle, state management, certificate issuance | Request, Submission, Approval, Issuance, Certificate |
| **BC-VESSEL** | Vessel Management | D04 | Vessel master data, attributes, name reservation | Vessel, Unit, Name, Category, Dimensions |
| **BC-APPLICANT** | Applicant Management | D02 | Applicant profiles, individual/company, agents | Applicant, Owner, Agent, Profile |
| **BC-REGISTRATION-POLICY** | Registration Policy | D05, D14 | DMN rule evaluation, decision APIs, policy configurations (CONF-xx), policy versioning | Policy, Rule, Decision, Config, Threshold |
| **BC-MASTERDATA** | Master Data | D15 | Reference codes (SET-xx), lookup tables, static categories | Code, Lookup, Category, Reference |
| **BC-DOCS** | Document Management | D06 | Document requirements, upload, completeness | Document, Requirement, Upload, Attachment |
| **BC-MAFWR-APPROVAL** | MAFWR External Approvals | D07 | MAFWR approval workflow for fishing vessels | Approval, Request, Response, Decision |
| **BC-INSPECT** | Inspection | D08 | Inspection requests, scheduling, results | Inspection, Report, Classification, Result |
| **BC-BILLING** | Billing & Payments | D10 | Fee invoicing, payment processing (NOT fee calculation) | Invoice, Payment, Receipt |
| **BC-MONITOR** | Monitoring & Enforcement | D12 | Reminders, deadlines, overdue detection | Reminder, Deadline, Overdue, Enforcement |
| **BC-IDENTITY** | Identity & Access | D02 | Authentication, PKI, authorization | Identity, Session, Authentication, Authorization |
| **BC-ELIGIBLE** | Eligibility | D03 | CR validation, activity eligibility | CR, Activity, Eligibility, Validation |
| **BC-CUSTOMS** | Customs Integration | D09 | Bayan customs clearance | Clearance, Import, Declaration |
| **BC-NOTIFY** | Notifications | D11 | Multi-channel notifications | Notification, Channel, Template, Delivery |
| **BC-AUDIT** | Audit & Governance | D13 | Audit trail, decision logging | Audit, Trail, Log, Compliance |

### 3.3 Policy & Configuration Architecture (Two-Tier)

Based on analysis of the source DOCX (إصدار شهادة تسجيل سفينة أو وحدة بحرية مؤقتة.docx), the architecture uses a **two-tier** model instead of three-tier:

**Rationale for merging BC-POLICY + BC-CONFIG → BC-REGISTRATION-POLICY:**
- CONF-xx codes are **inputs to DMN rules**, not separate decisions
- Only 4 configurations (CONF-01 to CONF-04) — doesn't justify a separate service
- DOCX treats them as "إعدادات الدورة" (process settings), part of policy context
- Simpler architecture with fewer inter-service calls

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                     POLICY & CONFIGURATION ARCHITECTURE (TWO-TIER)                       │
├─────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                          │
│   ┌──────────────────────────────────────────────────────────────────────────────────┐  │
│   │                     DOMAIN / PROCESS SERVICES                                     │  │
│   │   (BC-REG, BC-BILLING, BC-VESSEL, BC-INSPECT, BC-MONITOR)                        │  │
│   │                                                                                   │  │
│   │   Responsibility: Orchestration ONLY                                             │  │
│   │   - Workflow state management                                                     │  │
│   │   - Domain aggregates                                                             │  │
│   │   - NEVER calculates fees, validates rules, or determines requirements           │  │
│   │                                                                                   │  │
│   └───────────────────────────────────────────────────────────────────────────────────┘  │
│                                       │ calls (never owns rules)                        │
│                                       ▼                                                  │
│   ┌──────────────────────────────────────────────────────────────────────────────────┐  │
│   │               BC-REGISTRATION-POLICY (Decision + Configuration)                   │  │
│   │                                                                                   │  │
│   │   Responsibility: Decision Logic + Configurable Inputs                           │  │
│   │                                                                                   │  │
│   │   DECISIONS (DMN):                          CONFIGURATIONS (CONF-xx):            │  │
│   │   - reg001-fee-calculation.dmn              - CONF-01: Document Rules            │  │
│   │   - reg001-vessel-age-validation.dmn        - CONF-02: Penalty Timing (30 days)  │  │
│   │   - reg001-documents-required.dmn           - CONF-03: Validity Period (6 mo)    │  │
│   │   - reg001-inspection-required.dmn          - CONF-04: Prohibited Vessels List   │  │
│   │   - reg001-mafwr-required.dmn                                                     │  │
│   │   - reg001-customs-required.dmn                                                   │  │
│   │   - reg001-penalty-calculation.dmn                                                │  │
│   │                                                                                   │  │
│   │   Config is loaded INTERNALLY (not via separate service call)                    │  │
│   │                                                                                   │  │
│   └───────────────────────────────────┬──────────────────────────────────────────────┘  │
│                                       │ reads (lookup only)                              │
│                                       ▼                                                  │
│   ┌──────────────────────────────────────────────────────────────────────────────────┐  │
│   │                    BC-MASTERDATA (Reference Data Service)                         │  │
│   │                                                                                   │  │
│   │   Owns: Lookup Tables (SET-xx)              Used By:                              │  │
│   │   - SET-01: Vessel Types                    - Dropdowns in UI                     │  │
│   │   - SET-02: Build Materials                 - Policy DMN inputs                   │  │
│   │   - SET-03: Omani Ports                     - Domain entity lookups               │  │
│   │   - SET-04: Activity Types                                                        │  │
│   │   - SET-05: Eligible CR Activities          DOES NOT own: Decision logic          │  │
│   │   - SET-06: Acquisition Paths                                                     │  │
│   │                                                                                   │  │
│   └──────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                          │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

### 3.4 Service Responsibility Boundaries

| Service | Owns | Does NOT Own |
|---------|------|--------------|
| **BC-REGISTRATION-POLICY** | DMN files, rule evaluation APIs, policy configurations (CONF-xx), decision versioning | Master data, process orchestration |
| **BC-MASTERDATA** | Reference codes (SET-xx), lookup tables, static categories | Decision logic, business rules, policies |
| **BC-REG, BC-BILLING, etc.** | Workflow orchestration, state transitions, domain aggregates | Fee calculation, validation rules, eligibility decisions |

### 3.5 Golden Rules (Enforce in All Services)

| # | Rule | Violation Example |
|---|------|-------------------|
| 1 | **No DMN duplication** | Same fee logic in two services |
| 2 | **No policy logic in domain services** | Registration service calculates fees |
| 3 | **No master data in policy service** | Policy service stores vessel categories |
| 4 | **DMN changes ≠ service redeploy** | DMN embedded in JAR/DLL |
| 5 | **Config is part of policy context** | Separate config service for 4 settings |

---

## 4. Context Map

### 4.1 Visual Context Map

```
                                    ┌─────────────────────────────────────────┐
                                    │                                         │
                                    │     EXTERNAL SYSTEMS                    │
                                    │                                         │
                                    │  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
                                    │  │ ROP PKI │ │ Invest  │ │  Bayan  │   │
                                    │  │         │ │  Easy   │ │ Customs │   │
                                    │  └────┬────┘ └────┬────┘ └────┬────┘   │
                                    │       │          │           │         │
                                    │  ┌────┴────┐ ┌───┴────┐ ┌───┴─────┐   │
                                    │  │  MAFWR  │ │ Banks/ │ │Inspect  │   │
                                    │  │         │ │Payment │ │ Entity  │   │
                                    │  └────┬────┘ └───┬────┘ └────┬────┘   │
                                    │       │          │           │         │
                                    └───────┼──────────┼───────────┼─────────┘
                                            │          │           │
                    ┌───────────────────────┼──────────┼───────────┼───────────────────────┐
                    │                       │          │           │                       │
                    │      ┌────────────────┴──────────┴───────────┴────────────────┐     │
                    │      │                                                         │     │
                    │      │              INTEGRATION LAYER (ACL)                    │     │
                    │      │                                                         │     │
                    │      │   ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │     │
                    │      │   │ Identity │ │ Eligible │ │ Customs  │ │ Payment  │  │     │
                    │      │   │ Adapter  │ │ Adapter  │ │ Adapter  │ │ Adapter  │  │     │
                    │      │   └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘  │     │
                    │      │        │            │            │            │         │     │
                    │      │   ┌────┴─────┐ ┌────┴─────┐ ┌────┴─────┐               │     │
                    │      │   │ Approval │ │ Inspect  │ │  Notify  │               │     │
                    │      │   │ Adapter  │ │ Adapter  │ │ Adapter  │               │     │
                    │      │   └────┬─────┘ └────┬─────┘ └────┬─────┘               │     │
                    │      │        │            │            │                      │     │
                    │      └────────┼────────────┼────────────┼──────────────────────┘     │
                    │               │            │            │                            │
                    │               ▼            ▼            ▼                            │
                    │      ┌────────────────────────────────────────────────────────┐     │
                    │      │                                                         │     │
                    │      │                TEMPORARY REGISTRATION                   │     │
                    │      │                   (Core Context)                        │     │
                    │      │                                                         │     │
                    │      │   ┌─────────────────────────────────────────────────┐  │     │
                    │      │   │                                                  │  │     │
                    │      │   │   Request → Validate → Approve → Pay → Issue    │  │     │
                    │      │   │                                                  │  │     │
                    │      │   └─────────────────────────────────────────────────┘  │     │
                    │      │                                                         │     │
                    │      └───────────┬───────────┬───────────┬───────────┬────────┘     │
                    │                  │           │           │           │               │
                    │                  ▼           ▼           ▼           ▼               │
                    │   ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │
                    │   │  Vessel  │ │ Applicant│ │ Document │ │  Policy  │ │ Billing  │  │
                    │   │ Context  │ │ Context  │ │ Context  │ │ Context  │ │ Context  │  │
                    │   │          │ │          │ │          │ │          │ │          │  │
                    │   │ [U]      │ │ [U]      │ │ [U]      │ │ [U]      │ │ [U]      │  │
                    │   └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘  │
                    │                                                                      │
                    │   ┌──────────┐ ┌──────────┐                                         │
                    │   │ Approval │ │Inspection│ ┌──────────────────────────────────┐   │
                    │   │ Context  │ │ Context  │ │                                   │   │
                    │   │          │ │          │ │        MONITORING CONTEXT         │   │
                    │   │ [U]      │ │ [U]      │ │                                   │   │
                    │   └──────────┘ └──────────┘ │        Post-Issuance Only         │   │
                    │                             │                                   │   │
                    │                             └──────────────────────────────────┘   │
                    │                                                                      │
                    │   ┌──────────────────────────────────────────────────────────────┐  │
                    │   │                                                               │  │
                    │   │                    AUDIT CONTEXT                              │  │
                    │   │                    (Subscribes to all domain events)          │  │
                    │   │                                                               │  │
                    │   └──────────────────────────────────────────────────────────────┘  │
                    │                                                                      │
                    │                          PLATFORM BOUNDARY                          │
                    └─────────────────────────────────────────────────────────────────────┘

Legend:
[D] = Downstream (Consumer)
[U] = Upstream (Provider)
ACL = Anti-Corruption Layer
```

### 4.2 Context Relationships Matrix

| Upstream Context | Downstream Context | Relationship Type | Integration Pattern |
|------------------|-------------------|-------------------|---------------------|
| BC-VESSEL | BC-REG | Customer-Supplier | Conformist |
| BC-APPLICANT | BC-REG | Customer-Supplier | Conformist |
| BC-POLICY | BC-REG | Customer-Supplier | Conformist |
| BC-POLICY | BC-VESSEL | Customer-Supplier | Conformist |
| BC-POLICY | BC-DOCS | Customer-Supplier | Conformist |
| BC-POLICY | BC-BILLING | Customer-Supplier | Conformist |
| BC-DOCS | BC-REG | Customer-Supplier | Conformist |
| BC-MAFWR-APPROVAL | BC-REG | Customer-Supplier | Partnership |
| BC-INSPECT | BC-REG | Customer-Supplier | Partnership |
| BC-BILLING | BC-REG | Customer-Supplier | Conformist |
| BC-IDENTITY | BC-REG | Customer-Supplier | ACL |
| BC-IDENTITY | BC-APPLICANT | Customer-Supplier | ACL |
| BC-ELIGIBLE | BC-APPLICANT | Customer-Supplier | ACL |
| BC-CUSTOMS | BC-REG | Customer-Supplier | ACL |
| BC-NOTIFY | BC-REG | Customer-Supplier | Published Language |
| BC-NOTIFY | BC-BILLING | Customer-Supplier | Published Language |
| BC-NOTIFY | BC-MONITOR | Customer-Supplier | Published Language |
| BC-REG | BC-MONITOR | Customer-Supplier | Partnership |
| BC-REG | BC-AUDIT | Published Language | Event-driven |
| BC-BILLING | BC-AUDIT | Published Language | Event-driven |
| BC-MAFWR-APPROVAL | BC-AUDIT | Published Language | Event-driven |

---

## 5. Relationship Patterns

### 5.1 Partnership

Contexts that evolve together with mutual dependencies.

```
┌────────────────────┐         ┌────────────────────┐
│                    │ ◄─────► │                    │
│   BC-REG           │         │  BC-MAFWR-APPROVAL │
│   (Registration)   │  Sync   │  (MAFWR Approval)  │
│                    │         │                    │
└────────────────────┘         └────────────────────┘

Characteristics:
- Shared release planning
- Coordinated API evolution
- Bidirectional dependency
```

**Applies to:**
- BC-REG ↔ BC-MAFWR-APPROVAL (MAFWR workflow)
- BC-REG ↔ BC-INSPECT (Inspection workflow)
- BC-REG ↔ BC-MONITOR (Post-issuance monitoring)

### 5.2 Customer-Supplier

Clear upstream/downstream with defined contracts.

```
┌────────────────────┐         ┌────────────────────┐
│                    │ ──────► │                    │
│   BC-VESSEL        │         │   BC-REG           │
│   (Upstream)       │ Provides│   (Downstream)     │
│                    │  Data   │                    │
└────────────────────┘         └────────────────────┘

Characteristics:
- Supplier owns the contract
- Customer adapts to supplier changes
- Clear data ownership
```

**Applies to:**
- BC-VESSEL → BC-REG
- BC-APPLICANT → BC-REG
- BC-POLICY → BC-REG, BC-VESSEL, BC-DOCS, BC-BILLING
- BC-DOCS → BC-REG
- BC-BILLING → BC-REG

### 5.3 Conformist

Downstream context fully adopts upstream model.

```
┌────────────────────┐         ┌────────────────────┐
│                    │ ──────► │                    │
│   BC-POLICY        │         │   BC-REG           │
│   (Upstream)       │ Adopts  │   (Downstream)     │
│                    │  Model  │                    │
└────────────────────┘         └────────────────────┘

Characteristics:
- No translation needed
- Direct model usage
- Tight coupling (acceptable for internal contexts)
```

**Applies to:**
- BC-POLICY decisions consumed directly by BC-REG
- BC-VESSEL data consumed directly by BC-REG (as snapshot)

### 5.4 Anti-Corruption Layer (ACL)

Protection from external system models.

```
┌────────────────────┐         ┌─────────┐         ┌────────────────────┐
│                    │ ──────► │   ACL   │ ──────► │                    │
│   External System  │         │Translator│         │   Internal Context │
│   (Bayan, ROP)     │         │         │         │                    │
└────────────────────┘         └─────────┘         └────────────────────┘

Characteristics:
- Isolation from external changes
- Model translation
- Adapter pattern implementation
```

**Applies to:**
- ROP PKI → BC-IDENTITY
- Invest Easy → BC-ELIGIBLE
- Bayan Customs → BC-CUSTOMS
- Payment Gateway → BC-BILLING
- MAFWR → BC-MAFWR-APPROVAL
- Inspection Entity → BC-INSPECT

### 5.5 Published Language

Event-driven communication with well-defined events.

```
┌────────────────────┐         ┌────────────────────┐
│                    │ ─Event─►│                    │
│   BC-REG           │         │   BC-NOTIFY        │
│   (Publisher)      │         │   (Subscriber)     │
│                    │         │                    │
└────────────────────┘         └────────────────────┘

Characteristics:
- Loose coupling
- Asynchronous communication
- Standardized event schema
```

**Applies to:**
- BC-REG → BC-NOTIFY (status change notifications)
- BC-REG → BC-AUDIT (decision logging)
- BC-BILLING → BC-NOTIFY (payment notifications)
- BC-MONITOR → BC-NOTIFY (reminder notifications)

---

## 6. Context Teams (Recommended)

### 6.1 Team Topology

| Team | Owned Contexts | Responsibilities |
|------|----------------|------------------|
| **Registration Team** | BC-REG, BC-VESSEL | Core registration workflow, vessel management |
| **Applicant Team** | BC-APPLICANT, BC-IDENTITY, BC-ELIGIBLE | User management, authentication, eligibility |
| **Compliance Team** | BC-DOCS, BC-POLICY, BC-AUDIT | Document management, policy rules, audit |
| **External Integration Team** | BC-MAFWR-APPROVAL, BC-INSPECT, BC-CUSTOMS | External authority integrations |
| **Finance Team** | BC-BILLING, BC-MONITOR | Billing, payments, post-issuance monitoring |
| **Platform Team** | BC-NOTIFY | Notification infrastructure |

### 6.2 Team Dependencies

```
                    ┌──────────────────┐
                    │  Platform Team   │
                    │  (BC-NOTIFY)     │
                    └────────┬─────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Registration │    │   Finance    │    │  External    │
│    Team      │    │    Team      │    │ Integration  │
│              │◄───┤              │    │    Team      │
└──────┬───────┘    └──────────────┘    └──────┬───────┘
       │                                       │
       │            ┌──────────────┐           │
       └───────────►│  Compliance  │◄──────────┘
                    │    Team      │
                    └──────┬───────┘
                           │
                    ┌──────┴───────┐
                    │  Applicant   │
                    │    Team      │
                    └──────────────┘
```

---

## 7. Data Ownership by Context

| Context | Owned Data | Shared/Referenced Data |
|---------|------------|------------------------|
| BC-REG | RegistrationRequest, CertificateIssuance | VesselSnapshot, ApplicantSnapshot |
| BC-VESSEL | VesselProfile, NameReservation | — |
| BC-APPLICANT | ApplicantProfile, AgentRepresentation | Identity reference |
| BC-POLICY | PolicyRules, Thresholds, DMN tables | — |
| BC-DOCS | DocumentRequirements, UploadedDocuments | — |
| BC-MAFWR-APPROVAL | MAFWRApprovalRequest, ApprovalDecision | RequestReference |
| BC-INSPECT | InspectionRequest, InspectionResult | VesselReference, RequestReference |
| BC-BILLING | Invoice, Payment, Penalty | RequestReference |
| BC-MONITOR | ReminderSchedule, OverdueRecord | CertificateReference |
| BC-IDENTITY | IdentitySession, AuthenticationEvent | — |
| BC-ELIGIBLE | EligibilityCheck, CRValidation | ApplicantReference |
| BC-CUSTOMS | ClearanceRequest, ClearanceDecision | RequestReference |
| BC-NOTIFY | NotificationRecord, DeliveryStatus | — |
| BC-AUDIT | AuditEntry, DecisionLog | All references |

---

## 8. Ubiquitous Language Glossary

### 8.1 Core Terms

| Term (EN) | Term (AR) | Context | Definition |
|-----------|-----------|---------|------------|
| Registration Request | طلب التسجيل | BC-REG | A formal request to register a vessel temporarily |
| Certificate | شهادة التسجيل | BC-REG | The official document proving temporary registration |
| Submission | تقديم الطلب | BC-REG | The act of submitting a complete request |
| Issuance | الإصدار | BC-REG | The act of generating and delivering the certificate |

### 8.2 Vessel Terms

| Term (EN) | Term (AR) | Context | Definition |
|-----------|-----------|---------|------------|
| Vessel | سفينة | BC-VESSEL | A watercraft subject to registration |
| Marine Unit | وحدة بحرية | BC-VESSEL | A non-vessel marine structure (e.g., platform) |
| Gross Tonnage | الحمولة الإجمالية | BC-VESSEL | Measure of vessel volume |
| Name Reservation | حجز الاسم | BC-VESSEL | Temporary hold on a vessel name |

### 8.3 Process Terms

| Term (EN) | Term (AR) | Context | Definition |
|-----------|-----------|---------|------------|
| Approval | موافقة | BC-MAFWR-APPROVAL | Authorization from MAFWR for fishing vessels |
| Inspection | معاينة | BC-INSPECT | Physical examination of vessel |
| Clearance | إفراج جمركي | BC-CUSTOMS | Customs release for imported vessel |
| Penalty | غرامة | BC-BILLING | Financial charge for rule violation |

---

## 9. Traceability to Capabilities

| Context | Capabilities Owned |
|---------|-------------------|
| BC-REG | CAP-01, CAP-22 |
| BC-VESSEL | CAP-06, CAP-09, CAP-19 |
| BC-APPLICANT | CAP-03 |
| BC-IDENTITY | CAP-02 |
| BC-ELIGIBLE | CAP-04, CAP-05 |
| BC-POLICY | CAP-07, CAP-08, CAP-10, CAP-14, CAP-16 (Decision APIs) |
| BC-MASTERDATA | Reference data for SET-01 to SET-06 (Lookup Tables) |
| BC-CONFIG | Process configurations CONF-01 to CONF-04 (Thresholds) |
| BC-DOCS | CAP-11, CAP-12 |
| BC-MAFWR-APPROVAL | CAP-13 |
| BC-INSPECT | CAP-15 |
| BC-CUSTOMS | CAP-17 |
| BC-BILLING | CAP-18, CAP-20, CAP-21 (Invoicing, NOT calculation) |
| BC-MONITOR | CAP-24, CAP-25 |
| BC-NOTIFY | CAP-23 |
| BC-AUDIT | CAP-26 |

---

## 10. Related Documents

- [01-UML_Domain_Structure](01-UML_Domain_Structure.md) – Aggregate and entity definitions
- [03-Context_Map_ACL_SharedKernel](03-Context_Map_ACL_SharedKernel.md) – Integration patterns detail
- [02-Capability_to_Domain_Mapping](../../01-business-analysis/1.4-capabilities/02-Capability_to_Domain_Mapping/02-Capability_to_Domain_Mapping.md) – Source capability mapping
- [Functional Requirements](../../01-business-analysis/1.5-functional-requirements/README.md) – FR source

---

**Document Control**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-08 | Solution Architect | Initial context map |
| 1.1 | 2026-01-08 | Solution Architect | Added D14-Configuration domain; Added BC-CONFIG bounded context |
| 1.2 | 2026-01-08 | Solution Architect | Renamed BC-APPROVAL to BC-MAFWR-APPROVAL for clearer MAFWR context identification |
| 1.3 | 2026-01-08 | Solution Architect | Added BC-MASTERDATA for reference data; Refined BC-POLICY for decision-only; Updated BC-BILLING (invoicing only, not calculation); Three-tier separation: Policy/MasterData/Config |
| 1.4 | 2026-01-08 | Solution Architect | **DOCX-driven revision:** Merged BC-POLICY + BC-CONFIG → BC-REGISTRATION-POLICY. CONF-xx are inputs to DMN rules (not separate service). Simplified to two-tier architecture: BC-REGISTRATION-POLICY + BC-MASTERDATA. Removed D14, D16; Updated D05 to include config |