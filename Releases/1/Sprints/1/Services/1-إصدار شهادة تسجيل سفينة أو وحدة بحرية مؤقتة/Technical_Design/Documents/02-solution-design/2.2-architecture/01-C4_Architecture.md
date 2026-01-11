# REG001 – C4 Architecture

> **Service Code:** REG001  
> **Service Name (EN):** Issuing Temporary Registration Certificate for a Ship or Marine Unit  
> **Service Name (AR):** إصدار شهادة تسجيل سفينة أو وحدة بحرية مؤقتة  
> **Document Type:** C4 Architecture Model  
> **Version:** 1.0  
> **Last Updated:** 2026-01-11

---

## 1. Purpose / الغرض

This document provides the **C4 architecture model** for REG001, following the C4 model methodology:
- **Level 1: System Context** – Shows the system in relation to external actors and systems
- **Level 2: Container** – Shows high-level technology choices and container interactions
- **Level 3: Component** – Shows internal components within key containers

---

## 2. Architecture Principles

### 2.1 Core Principles

| # | Principle | Description |
|---|-----------|-------------|
| 1 | **Process Engine Authority** | Flowable BPMN/DMN is the authoritative process and decision engine |
| 2 | **Two-Tier Policy Architecture** | Policy+Config merged into single context; Master Data separate |
| 3 | **ACL for External Systems** | Anti-Corruption Layers isolate external system specifics |
| 4 | **Event-Driven Integration** | Cross-context communication via domain events |
| 5 | **Separation of Concerns** | Domain services do NOT make policy decisions |
| 6 | **API-First Design** | All services expose well-defined REST APIs |

### 2.2 Technology Stack Assumptions

| Layer | Technology |
|-------|------------|
| Process Engine | Flowable (BPMN + DMN) |
| API Gateway | Kong / API Manager |
| Backend Services | Java (Spring Boot) / .NET Core |
| Frontend | Angular SPA |
| Integration | REST + Event-Driven (Kafka/RabbitMQ) |
| Database | PostgreSQL (per-service) |

---

## 3. Level 1: System Context Diagram

### 3.1 Context Overview

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                      SYSTEM CONTEXT - REG001                                             │
├─────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                          │
│                                           ┌─────────────────┐                                            │
│                                           │   Ship Owner /  │                                            │
│                                           │    Applicant    │                                            │
│                                           │   (المستفيد)    │                                            │
│                                           └────────┬────────┘                                            │
│                                                    │                                                     │
│                                                    │ Uses web portal                                     │
│                                                    ▼                                                     │
│   ┌──────────────┐                    ┌────────────────────────────┐                    ┌──────────────┐ │
│   │  ROP PKI     │◄───────────────────│                            │───────────────────►│  Invest Easy │ │
│   │  (Identity)  │   Authentication   │                            │   CR/Activity      │  (CR/Activity)│ │
│   └──────────────┘                    │                            │   Validation       └──────────────┘ │
│                                       │      MTCIT Maritime        │                                     │
│   ┌──────────────┐                    │    Registration System     │                    ┌──────────────┐ │
│   │    MAFWR     │◄───────────────────│                            │───────────────────►│    Bayan     │ │
│   │  (Fishing)   │   Fishing Approval │                            │   Customs          │  (Customs)   │ │
│   └──────────────┘                    │                            │   Clearance        └──────────────┘ │
│                                       │                            │                                     │
│   ┌──────────────┐                    │                            │                    ┌──────────────┐ │
│   │  Inspection  │◄───────────────────│                            │───────────────────►│   Payment    │ │
│   │   Bodies     │   Vessel Inspection│                            │   Fee Payment      │   Gateway    │ │
│   └──────────────┘                    └────────────┬───────────────┘                    └──────────────┘ │
│                                                    │                                                     │
│                                                    │ Reviews & Approves                                  │
│                                                    ▼                                                     │
│                                           ┌─────────────────┐                                            │
│                                           │    MTCIT Staff  │                                            │
│                                           │   (Authority)   │                                            │
│                                           │   (الجهة المختصة)│                                           │
│                                           └─────────────────┘                                            │
│                                                                                                          │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 External Systems & Actors

| Actor/System | Type | Description | Integration |
|--------------|------|-------------|-------------|
| **Ship Owner / Applicant** | Actor | Individual or company requesting vessel registration | Web Portal (Angular) |
| **MTCIT Staff** | Actor | Authority personnel reviewing and approving registrations | Backoffice Portal |
| **ROP PKI** | External System | Royal Oman Police – Digital identity authentication | SOAP/REST API |
| **Invest Easy** | External System | Commercial Registration and Activity validation | REST API |
| **MAFWR** | External System | Ministry of Agriculture, Fisheries & Water Resources – Fishing vessel approval | REST API (async) |
| **Bayan** | External System | Customs clearance system | REST API (async) |
| **Inspection Bodies** | External System | Classification societies / inspection entities | REST API (async) |
| **Payment Gateway** | External System | Electronic payment processing | REST API |
| **SMS/Email Provider** | External System | Notification delivery | REST API |

### 3.3 System Responsibilities

The **MTCIT Maritime Registration System** provides:

1. **Temporary Registration Workflow** – End-to-end process from application to certificate issuance
2. **Policy Enforcement** – Age validation, prohibited vessel checks, document requirements
3. **External Approvals** – MAFWR, Inspection, Customs coordination
4. **Payment Processing** – Fee calculation, invoice generation, payment tracking
5. **Certificate Issuance** – PDF + QR code generation, registry update
6. **Post-Issuance Monitoring** – Reminders and penalty enforcement

---

## 4. Level 2: Container Diagram

### 4.1 Container Overview

```
┌────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                           CONTAINER DIAGRAM - REG001                                                │
├────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                                     │
│   ┌──────────────────────┐        ┌──────────────────────┐                                                         │
│   │   Applicant Portal   │        │   Backoffice Portal  │                                                         │
│   │      (Angular)       │        │      (Angular)       │                                                         │
│   └──────────┬───────────┘        └──────────┬───────────┘                                                         │
│              │                               │                                                                      │
│              └───────────────┬───────────────┘                                                                      │
│                              │ HTTPS / REST                                                                         │
│                              ▼                                                                                      │
│              ┌───────────────────────────────┐                                                                      │
│              │        API Gateway            │                                                                      │
│              │    (Kong / API Manager)       │                                                                      │
│              │   - Rate Limiting             │                                                                      │
│              │   - RBAC Enforcement          │                                                                      │
│              │   - Request Routing           │                                                                      │
│              └───────────────┬───────────────┘                                                                      │
│                              │                                                                                      │
│     ┌────────────────────────┼────────────────────────┐                                                            │
│     │                        │                        │                                                            │
│     ▼                        ▼                        ▼                                                            │
│ ┌────────────────┐   ┌────────────────┐   ┌────────────────┐                                                       │
│ │ Registration   │   │ Vessel Profile │   │   Applicant    │                                                       │
│ │ Process Service│   │    Service     │   │    Service     │                                                       │
│ │ (LS-REG-ORCH)  │   │  (LS-VESSEL)   │   │ (LS-APPLICANT) │                                                       │
│ │                │   │                │   │                │                                                       │
│ │ - Workflow     │   │ - Vessel CRUD  │   │ - Profile CRUD │                                                       │
│ │ - State Mgmt   │   │ - Name Reserve │   │ - Agent Mgmt   │                                                       │
│ │                │   │                │   │                │                                                       │
│ └───────┬────────┘   └───────┬────────┘   └───────┬────────┘                                                       │
│         │                    │                    │                                                                │
│         │                    ▼                    ▼                                                                │
│         │            ┌────────────────┐   ┌────────────────┐                                                       │
│         │            │  Documents     │   │   Billing      │                                                       │
│         │            │   Service      │   │   Service      │                                                       │
│         │            │  (LS-DOCS)     │   │  (LS-BILLING)  │                                                       │
│         │            │                │   │                │                                                       │
│         │            │ - Doc Upload   │   │ - Invoicing    │                                                       │
│         │            │ - Completeness │   │ - Payments     │                                                       │
│         │            └───────┬────────┘   └───────┬────────┘                                                       │
│         │                    │                    │                                                                │
│         ▼                    ▼                    ▼                                                                │
│ ┌────────────────────────────────────────────────────────────┐                                                     │
│ │                    Flowable Process Engine                  │                                                     │
│ │                                                             │                                                     │
│ │   ┌──────────────────────┐    ┌──────────────────────┐     │                                                     │
│ │   │    BPMN Runtime      │    │    DMN Runtime       │     │                                                     │
│ │   │  - Process State     │    │  - Decision Tables   │     │                                                     │
│ │   │  - Task Assignment   │    │  - Rule Evaluation   │     │                                                     │
│ │   │  - Event Handling    │    │                      │     │                                                     │
│ │   └──────────────────────┘    └──────────────────────┘     │                                                     │
│ │                                                             │                                                     │
│ └──────────────────────────────┬──────────────────────────────┘                                                     │
│                                │                                                                                    │
│         ┌──────────────────────┼──────────────────────┐                                                            │
│         │                      │                      │                                                            │
│         ▼                      ▼                      ▼                                                            │
│ ┌────────────────┐    ┌────────────────┐    ┌────────────────┐                                                     │
│ │   Policy &     │    │  Master Data   │    │  Certificate   │                                                     │
│ │ Configuration  │    │    Service     │    │    Service     │                                                     │
│ │   Service      │    │ (LS-MASTERDATA)│    │   (LS-CERT)    │                                                     │
│ │  (LS-POLICY)   │    │                │    │                │                                                     │
│ │                │    │ - Lookups      │    │ - PDF + QR Gen │                                                     │
│ │ - DMN Rules    │    │ - Ref Codes    │    │ - Registry     │                                                     │
│ │ - Config CONF-x│    │                │    │                │                                                     │
│ └───────┬────────┘    └───────┬────────┘    └───────┬────────┘                                                     │
│         │                     │                     │                                                              │
│         └─────────────────────┼─────────────────────┘                                                              │
│                               │                                                                                    │
│                               ▼                                                                                    │
│         ┌─────────────────────────────────────────────────────────┐                                                │
│         │              INTEGRATION LAYER (ACL Adapters)           │                                                │
│         │                                                         │                                                │
│         │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │                                                │
│         │  │ Identity │ │ CR       │ │ Customs  │ │ Approval │  │                                                │
│         │  │ Adapter  │ │ Adapter  │ │ Adapter  │ │ Adapter  │  │                                                │
│         │  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘  │                                                │
│         │       │            │            │            │         │                                                │
│         │  ┌────┴─────┐ ┌────┴─────┐ ┌────┴─────┐               │                                                │
│         │  │ Inspect  │ │ Payment  │ │ Notify   │               │                                                │
│         │  │ Adapter  │ │ Adapter  │ │ Adapter  │               │                                                │
│         │  └────┬─────┘ └────┬─────┘ └────┬─────┘               │                                                │
│         │       │            │            │                      │                                                │
│         └───────┼────────────┼────────────┼──────────────────────┘                                                │
│                 │            │            │                                                                        │
│                 ▼            ▼            ▼                                                                        │
│         ┌──────────────────────────────────────────────────────────┐                                               │
│         │                    EXTERNAL SYSTEMS                       │                                               │
│         │  ROP PKI | Invest Easy | MAFWR | Bayan | Inspection |    │                                               │
│         │  Payment Gateway | SMS/Email                              │                                               │
│         └──────────────────────────────────────────────────────────┘                                               │
│                                                                                                                     │
│         ┌──────────────────────────────────────────────────────────┐                                               │
│         │                    CROSS-CUTTING SERVICES                 │                                               │
│         │                                                           │                                               │
│         │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │                                               │
│         │  │ Notification │  │    Audit     │  │  Monitoring  │   │                                               │
│         │  │   Service    │  │   Service    │  │   Service    │   │                                               │
│         │  │ (LS-NOTIFY)  │  │  (LS-AUDIT)  │  │ (LS-MONITOR) │   │                                               │
│         │  └──────────────┘  └──────────────┘  └──────────────┘   │                                               │
│         │                                                           │                                               │
│         └──────────────────────────────────────────────────────────┘                                               │
│                                                                                                                     │
└────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 4.2 Container Catalog

| Container | Type | Logical Service ID | Technology | Responsibility |
|-----------|------|-------------------|------------|----------------|
| **Applicant Portal** | Web Application | - | Angular | Public-facing portal for applicants |
| **Backoffice Portal** | Web Application | - | Angular | Internal portal for MTCIT staff |
| **API Gateway** | Infrastructure | - | Kong | Authentication, rate limiting, routing |
| **Registration Process Service** | Microservice | LS-REG-ORCH | Java/Spring Boot | Workflow orchestration, request lifecycle |
| **Vessel Profile Service** | Microservice | LS-VESSEL | Java/Spring Boot | Vessel master data, name reservation |
| **Applicant Service** | Microservice | LS-APPLICANT | Java/Spring Boot | Applicant profiles, agent management |
| **Documents Service** | Microservice | LS-DOCS | Java/Spring Boot | Document upload, completeness validation |
| **Billing Service** | Microservice | LS-BILLING | Java/Spring Boot | Invoicing, payment tracking |
| **Policy & Configuration Service** | Microservice | LS-POLICY | Java/Spring Boot | DMN rules + CONF-xx configurations |
| **Master Data Service** | Microservice | LS-MASTERDATA | Java/Spring Boot | Reference codes (SET-xx), lookups |
| **Certificate Service** | Microservice | LS-CERT | Java/Spring Boot | PDF generation, QR codes, registry |
| **Notification Service** | Microservice | LS-NOTIFY | Java/Spring Boot | Multi-channel notifications |
| **Audit Service** | Microservice | LS-AUDIT | Java/Spring Boot | Decision logging, audit trail |
| **Monitoring Service** | Microservice | LS-MONITOR | Java/Spring Boot | Post-issuance reminders, penalties |
| **Flowable Process Engine** | Platform | - | Flowable | BPMN + DMN runtime |
| **Integration Adapters** | Adapters | Various | Java/Spring Boot | ACL for external systems |

### 4.3 Container Responsibilities

#### 4.3.1 Registration Process Service (LS-REG-ORCH)

**Primary Aggregate:** `RegistrationRequest`

**Capabilities Owned:**
- CAP-01: Service discovery & start request
- Process state management
- Workflow orchestration via Flowable

**Does NOT Own:**
- Fee calculation (delegates to LS-POLICY)
- Document requirements (delegates to LS-POLICY)
- Vessel validation rules (delegates to LS-POLICY)

#### 4.3.2 Policy & Configuration Service (LS-POLICY)

**Owns:**
- All DMN decision tables (`reg001-*.dmn`)
- Configuration codes (CONF-01 to CONF-04)
- Rule versioning and evolution

**DMN Tables:**
| DMN File | Purpose | Capabilities |
|----------|---------|--------------|
| `reg001-vessel-age-validation.dmn` | Age policy validation | CAP-07 |
| `reg001-documents-required.dmn` | Dynamic document requirements | CAP-11 |
| `reg001-fee-calculation.dmn` | Fee calculation | CAP-18 |
| `reg001-penalty-calculation.dmn` | Penalty calculation | CAP-10, CAP-25 |
| `reg001-inspection-required.dmn` | Inspection requirement decision | CAP-14 |
| `reg001-mafwr-required.dmn` | MAFWR approval requirement | CAP-13 |
| `reg001-customs-required.dmn` | Customs clearance requirement | CAP-16 |

**Configurations:**
| Code | Description |
|------|-------------|
| CONF-01 | Document requirements rules |
| CONF-02 | Penalty timing (30 days threshold) |
| CONF-03 | Certificate validity period (6 months) |
| CONF-04 | Prohibited vessels list |

#### 4.3.3 Master Data Service (LS-MASTERDATA)

**Owns:** Reference codes (SET-xx)

| Code | Description |
|------|-------------|
| SET-01 | Vessel/Marine Unit data schema |
| SET-02 | Owner data schema |
| SET-03 | Omani ports |
| SET-04 | Vessel types |
| SET-05 | Activity types |
| SET-06 | Service fees schedule |

---

## 5. Level 3: Component Diagram (Key Services)

### 5.1 Registration Process Service Components

```
┌────────────────────────────────────────────────────────────────────────────────┐
│                  REGISTRATION PROCESS SERVICE (LS-REG-ORCH)                     │
├────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │                         API LAYER                                        │  │
│   │   ┌────────────────┐  ┌────────────────┐  ┌────────────────┐            │  │
│   │   │  Registration  │  │   Status       │  │   Task         │            │  │
│   │   │   Controller   │  │  Controller    │  │  Controller    │            │  │
│   │   └───────┬────────┘  └───────┬────────┘  └───────┬────────┘            │  │
│   └───────────┼───────────────────┼───────────────────┼─────────────────────┘  │
│               │                   │                   │                        │
│               ▼                   ▼                   ▼                        │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │                      APPLICATION LAYER                                   │  │
│   │   ┌────────────────────┐  ┌────────────────────┐                        │  │
│   │   │  Registration      │  │  Task              │                        │  │
│   │   │  Application Svc   │  │  Application Svc   │                        │  │
│   │   └─────────┬──────────┘  └─────────┬──────────┘                        │  │
│   │             │                       │                                    │  │
│   │   ┌─────────┴───────────────────────┴───────────────────────────────┐   │  │
│   │   │                    FLOWABLE PROCESS ADAPTER                      │   │  │
│   │   │   - startProcess()                                               │   │  │
│   │   │   - completeServiceTask()                                        │   │  │
│   │   │   - correlateMessage()                                           │   │  │
│   │   │   - evaluateDMN()                                                │   │  │
│   │   └─────────────────────────────────────────────────────────────────┘   │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │                         DOMAIN LAYER                                     │  │
│   │   ┌────────────────────┐  ┌────────────────────┐                        │  │
│   │   │  RegistrationReq   │  │  Domain            │                        │  │
│   │   │  Aggregate         │  │  Services          │                        │  │
│   │   │                    │  │                    │                        │  │
│   │   │  - ApplicantSnap   │  │  - ValidationSvc   │                        │  │
│   │   │  - VesselSnap      │  │  - StateSvc        │                        │  │
│   │   │  - DocumentBundle  │  │                    │                        │  │
│   │   │  - ApprovalStatus  │  │                    │                        │  │
│   │   │  - PaymentRecord   │  │                    │                        │  │
│   │   └────────────────────┘  └────────────────────┘                        │  │
│   │                                                                          │  │
│   │   ┌────────────────────────────────────────────────────────────────┐    │  │
│   │   │                      DOMAIN EVENTS                              │    │  │
│   │   │  - RequestSubmittedEvent                                        │    │  │
│   │   │  - RequestApprovedEvent                                         │    │  │
│   │   │  - PaymentConfirmedEvent                                        │    │  │
│   │   │  - CertificateIssuedEvent                                       │    │  │
│   │   └────────────────────────────────────────────────────────────────┘    │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │                      INFRASTRUCTURE LAYER                                │  │
│   │   ┌────────────────┐  ┌────────────────┐  ┌────────────────┐            │  │
│   │   │  Repository    │  │  Event         │  │   Client       │            │  │
│   │   │  Impl          │  │  Publisher     │  │   Adapters     │            │  │
│   │   └────────────────┘  └────────────────┘  └────────────────┘            │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
└────────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 Policy & Configuration Service Components

```
┌────────────────────────────────────────────────────────────────────────────────┐
│                  POLICY & CONFIGURATION SERVICE (LS-POLICY)                     │
├────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │                         API LAYER                                        │  │
│   │   ┌────────────────────┐  ┌────────────────────┐                        │  │
│   │   │  Decision          │  │  Configuration     │                        │  │
│   │   │  Controller        │  │  Controller        │                        │  │
│   │   │                    │  │                    │                        │  │
│   │   │  POST /evaluate    │  │  GET /config/{key} │                        │  │
│   │   └────────────────────┘  └────────────────────┘                        │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │                      APPLICATION LAYER                                   │  │
│   │   ┌────────────────────────────────────────────────────────────────┐    │  │
│   │   │                 DECISION SERVICE                                │    │  │
│   │   │                                                                 │    │  │
│   │   │  + evaluateVesselAge(vesselType, buildYear, material): Result  │    │  │
│   │   │  + evaluateDocumentRequirements(type, activity, length): Docs  │    │  │
│   │   │  + calculateFees(type, tonnage, length, activity): Money       │    │  │
│   │   │  + calculatePenalty(buildEndDate, submissionDate): Money       │    │  │
│   │   │  + isInspectionRequired(phase, length, type): Boolean          │    │  │
│   │   │  + isMAFWRRequired(vesselType): Boolean                        │    │  │
│   │   │  + isCustomsRequired(acquisitionPath): Boolean                 │    │  │
│   │   │                                                                 │    │  │
│   │   └────────────────────────────────────────────────────────────────┘    │  │
│   │                                                                          │  │
│   │   ┌────────────────────────────────────────────────────────────────┐    │  │
│   │   │                 CONFIGURATION SERVICE                           │    │  │
│   │   │                                                                 │    │  │
│   │   │  + getDocumentRules(): CONF-01                                 │    │  │
│   │   │  + getPenaltyTiming(): CONF-02                                 │    │  │
│   │   │  + getCertificateValidity(): CONF-03                           │    │  │
│   │   │  + getProhibitedVessels(): CONF-04                             │    │  │
│   │   │                                                                 │    │  │
│   │   └────────────────────────────────────────────────────────────────┘    │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │                      DMN ENGINE LAYER                                    │  │
│   │   ┌────────────────────────────────────────────────────────────────┐    │  │
│   │   │                 FLOWABLE DMN RUNTIME                            │    │  │
│   │   │                                                                 │    │  │
│   │   │  ┌─────────────────────────────────────────────────────────┐   │    │  │
│   │   │  │  reg001-vessel-age-validation.dmn                        │   │    │  │
│   │   │  │  reg001-documents-required.dmn                           │   │    │  │
│   │   │  │  reg001-fee-calculation.dmn                              │   │    │  │
│   │   │  │  reg001-penalty-calculation.dmn                          │   │    │  │
│   │   │  │  reg001-inspection-required.dmn                          │   │    │  │
│   │   │  │  reg001-mafwr-required.dmn                               │   │    │  │
│   │   │  │  reg001-customs-required.dmn                             │   │    │  │
│   │   │  └─────────────────────────────────────────────────────────┘   │    │  │
│   │   └────────────────────────────────────────────────────────────────┘    │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │                      INFRASTRUCTURE LAYER                                │  │
│   │   ┌────────────────────┐  ┌────────────────────┐                        │  │
│   │   │  DMN Repository    │  │  Config Repository │                        │  │
│   │   │  (Externalized)    │  │  (Database)        │                        │  │
│   │   └────────────────────┘  └────────────────────┘                        │  │
│   └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
└────────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Data Flow & Communication

### 6.1 Synchronous Communication (REST)

| From | To | Purpose | API Pattern |
|------|----|---------|-------------|
| API Gateway | All Services | Request routing | REST |
| Registration Service | Policy Service | Decision evaluation | REST |
| Registration Service | Vessel Service | Vessel data lookup | REST |
| Registration Service | Applicant Service | Applicant lookup | REST |
| Registration Service | Documents Service | Document status | REST |
| Billing Service | Policy Service | Fee calculation | REST |
| Any Service | Master Data Service | Lookup codes | REST |

### 6.2 Asynchronous Communication (Events)

| Publisher | Event | Subscribers | Channel |
|-----------|-------|-------------|---------|
| Registration Service | `RequestSubmittedEvent` | Audit, Notification | Kafka/RabbitMQ |
| Registration Service | `RequestApprovedEvent` | Billing, Notification | Kafka/RabbitMQ |
| Billing Service | `PaymentConfirmedEvent` | Registration, Certificate, Notification | Kafka/RabbitMQ |
| Certificate Service | `CertificateIssuedEvent` | Monitoring, Notification, Audit | Kafka/RabbitMQ |
| Monitoring Service | `ReminderDueEvent` | Notification | Kafka/RabbitMQ |
| Monitoring Service | `PenaltyAppliedEvent` | Billing, Notification, Audit | Kafka/RabbitMQ |

### 6.3 External Integration Patterns

| External System | Pattern | Adapter | Sync/Async |
|-----------------|---------|---------|------------|
| ROP PKI | ACL | Identity Adapter | Sync |
| Invest Easy | ACL | Eligibility Adapter | Sync |
| MAFWR | ACL | Approval Adapter | Async (Message) |
| Bayan | ACL | Customs Adapter | Async (Message) |
| Inspection Bodies | ACL | Inspection Adapter | Async (Message) |
| Payment Gateway | ACL | Payment Adapter | Sync + Callback |
| SMS/Email | ACL | Notification Adapter | Async |

---

## 7. Deployment View

### 7.1 Deployment Architecture

```
┌────────────────────────────────────────────────────────────────────────────────┐
│                           DEPLOYMENT ARCHITECTURE                               │
├────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│   ┌─────────────────────────────────────────────────────────────────────────┐  │
│   │                         PRESENTATION TIER                                │  │
│   │   ┌─────────────────┐         ┌─────────────────┐                       │  │
│   │   │ Applicant Portal│         │ Backoffice Portal│                      │  │
│   │   │   (CDN/S3)      │         │   (CDN/S3)       │                      │  │
│   │   └────────┬────────┘         └────────┬─────────┘                      │  │
│   │            └───────────────────────────┘                                │  │
│   │                           │                                              │  │
│   └───────────────────────────┼──────────────────────────────────────────────┘  │
│                               │                                                 │
│   ┌───────────────────────────┼──────────────────────────────────────────────┐  │
│   │                    API TIER│                                              │  │
│   │   ┌───────────────────────▼───────────────────────┐                      │  │
│   │   │              Load Balancer                     │                      │  │
│   │   └───────────────────────┬───────────────────────┘                      │  │
│   │                           │                                              │  │
│   │   ┌───────────────────────▼───────────────────────┐                      │  │
│   │   │           API Gateway Cluster                  │                      │  │
│   │   │         (Kong - 2+ instances)                  │                      │  │
│   │   └───────────────────────┬───────────────────────┘                      │  │
│   └───────────────────────────┼──────────────────────────────────────────────┘  │
│                               │                                                 │
│   ┌───────────────────────────┼──────────────────────────────────────────────┐  │
│   │                   SERVICE TIER (Kubernetes)                               │  │
│   │                           │                                              │  │
│   │   ┌───────────────┬───────┴───────┬───────────────┬───────────────┐     │  │
│   │   │ Registration  │    Vessel     │   Applicant   │   Documents   │     │  │
│   │   │   Service     │   Service     │    Service    │    Service    │     │  │
│   │   │  (2+ pods)    │  (2+ pods)    │   (2+ pods)   │   (2+ pods)   │     │  │
│   │   └───────────────┴───────────────┴───────────────┴───────────────┘     │  │
│   │                                                                          │  │
│   │   ┌───────────────┬───────────────┬───────────────┬───────────────┐     │  │
│   │   │   Billing     │    Policy     │  Master Data  │  Certificate  │     │  │
│   │   │   Service     │   Service     │    Service    │    Service    │     │  │
│   │   │  (2+ pods)    │  (2+ pods)    │   (2+ pods)   │   (2+ pods)   │     │  │
│   │   └───────────────┴───────────────┴───────────────┴───────────────┘     │  │
│   │                                                                          │  │
│   │   ┌───────────────┬───────────────┬───────────────┐                     │  │
│   │   │ Notification  │    Audit      │  Monitoring   │                     │  │
│   │   │   Service     │   Service     │    Service    │                     │  │
│   │   │  (2+ pods)    │  (2+ pods)    │   (2+ pods)   │                     │  │
│   │   └───────────────┴───────────────┴───────────────┘                     │  │
│   │                                                                          │  │
│   │   ┌─────────────────────────────────────────────────────────────────┐   │  │
│   │   │              Flowable Process Engine Cluster                     │   │  │
│   │   │                     (3+ instances)                               │   │  │
│   │   └─────────────────────────────────────────────────────────────────┘   │  │
│   │                                                                          │  │
│   └──────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│   ┌──────────────────────────────────────────────────────────────────────────┐  │
│   │                           DATA TIER                                       │  │
│   │   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                   │  │
│   │   │  PostgreSQL  │  │  PostgreSQL  │  │  PostgreSQL  │  (Per-service)   │  │
│   │   │  (Primary)   │  │  (Replica)   │  │  (Flowable)  │                   │  │
│   │   └──────────────┘  └──────────────┘  └──────────────┘                   │  │
│   │                                                                           │  │
│   │   ┌──────────────┐  ┌──────────────┐                                     │  │
│   │   │    Redis     │  │   Kafka /    │                                     │  │
│   │   │   (Cache)    │  │  RabbitMQ    │                                     │  │
│   │   └──────────────┘  └──────────────┘                                     │  │
│   │                                                                           │  │
│   └──────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
└────────────────────────────────────────────────────────────────────────────────┘
```

---

## 8. Traceability

### 8.1 Capability to Container Mapping

| Capability | Primary Container | Supporting Containers |
|------------|-------------------|----------------------|
| CAP-01 | Registration Process Service | - |
| CAP-02 | Identity Adapter | Registration Process Service |
| CAP-03 | Applicant Service | Registration Process Service |
| CAP-04 | Eligibility Adapter | Applicant Service |
| CAP-05 | Eligibility Adapter | Policy Service |
| CAP-06 | Vessel Profile Service | Registration Process Service |
| CAP-07 | Policy Service | Vessel Profile Service |
| CAP-08 | Policy Service | Vessel Profile Service |
| CAP-09 | Vessel Profile Service | - |
| CAP-10 | Policy Service | Billing Service |
| CAP-11 | Policy Service | Documents Service |
| CAP-12 | Documents Service | - |
| CAP-13 | Approval Adapter | Registration Process Service |
| CAP-14 | Policy Service | - |
| CAP-15 | Inspection Adapter | Registration Process Service |
| CAP-16 | Policy Service | - |
| CAP-17 | Customs Adapter | Registration Process Service |
| CAP-18 | Policy Service | Billing Service |
| CAP-19 | Vessel Profile Service | Registration Process Service |
| CAP-20 | Billing Service | Policy Service |
| CAP-21 | Payment Adapter | Billing Service |
| CAP-22 | Certificate Service | Registration Process Service |
| CAP-23 | Notification Service | All Services |
| CAP-24 | Monitoring Service | Notification Service |
| CAP-25 | Policy Service | Monitoring Service, Billing Service |
| CAP-26 | Audit Service | All Services |

---

## 9. Related Documents

- [02-Capability_to_Service_C4.md](02-Capability_to_Service_C4.md) – Detailed service decomposition
- [03-Sequence_Diagrams.md](03-Sequence_Diagrams.md) – Service interaction flows
- [04-Integration_Architecture.md](04-Integration_Architecture.md) – External system integrations
- [02-Business_Domain_Context_Map.md](../2.1-domain-model/02-Business_Domain_Context_Map.md) – Bounded contexts

---

**Last Updated:** 2026-01-11  
**Version:** 1.0  
**Author:** Solution Architecture Team
