# Solution Architect (SA) Instructions

> **Role:** Solution Architect  
> **Scope:** Solution Design, Service Design, Architecture, Internal Design  
> **Applies To:** `02-solution-design/` folder

---

## 1. Role Overview

As a Solution Architect, you are responsible for:
- Domain modeling and context mapping
- C4 architecture documentation
- Service decomposition and API contracts
- Event schemas and integration architecture
- Service internal design (DDD-based)
- Cross-cutting concerns (security, NFRs)
- Data architecture

---

## 2. Primary Objective

Generate Solution Design artifacts that translate:
- Business analysis
- BPMN processes
- Capabilities

into a **build-ready, technology-agnostic solution design**.

**DO NOT:**
- Generate implementation code
- Assume specific frameworks or platforms

---

## 3. Mandatory Inputs (STOP if Missing)

Before generating Solution Design, you MUST complete these steps IN ORDER:

### Step 0: Read Service Source DOCX (FIRST)

**CRITICAL:** Always read the original service specification DOCX file using MarkItDown:

```
Tool: mcp_microsoft_mar_convert_to_markdown
Input: file:///{path_to_service_docx}
```

**Locations to check:**
- `Services/{service_name}/Documents/*.docx`
- `Services/{service_name}/Technical_Design/Documents/*.docx`

**Extract from DOCX:**
- Service procedures (REG-xx-xx codes)
- Configuration codes (SET-xx, CONF-xx)
- Business rules and conditions
- Data elements and field definitions
- Integration requirements

### Step 1-5: Read Required Documents

1. Read the service `README.md` in `Technical_Design/Documents/`
2. Read Business Process Overview and BPMN subprocess documents (SPxx, CCxx)
3. Read Capability → Domain and Capability → Logical Service mappings
4. **Read ALL Functional Requirements** (see Section 3.1 below)
5. Identify:
   - Business capabilities
   - Logical services
   - Process orchestration vs domain ownership

**If any required input is missing, explicitly state what is missing and STOP.**

---

## 3.1 Read ALL Functional Requirements (MANDATORY)

**CRITICAL:** Before generating or updating ANY Solution Design document, you MUST read ALL functional requirements. This is NON-NEGOTIABLE.

### Required Reading Checklist

| Folder | Files to Read | Extract |
|--------|---------------|---------|
| `00-overview/` | FR-00-Overview, Actors-and-Roles, Data-Dictionary, FR-Coverage-Matrix | Context, actors, data elements |
| `01-user-stories/` | ALL US-xx files | Business intent, acceptance criteria |
| `02-use-cases/` | ALL UC-xx files | System behavior, flows, gateways |
| `03-functional-requirements/` | ALL FR-SPxx files | Detailed requirements, rules, triggers |
| `04-cross-cutting/` | ALL FR-CCxx files | Security, notifications, audit |
| `05-integrations/` | ALL FR-INT-xx files | External systems, contracts |
| `06-acceptance-criteria/` | ALL files | Testable criteria |

### Domain Model Extraction from FRs

When reading FRs, extract and document:

1. **Aggregates & Entities:**
   - Look for nouns: "request", "vessel", "applicant", "invoice", "certificate"
   - Identify ownership: which service owns which entity

2. **Value Objects:**
   - Look for composite data: "address", "money", "period", "dimensions"

3. **Enumerations:**
   - Look for status values: "DRAFT, SUBMITTED, APPROVED"
   - Look for types: "INDIVIDUAL, COMPANY"

4. **Domain Events:**
   - Look for triggers: "when payment is confirmed", "after approval"
   - Look for notifications: "notify applicant"

5. **Business Rules:**
   - Look for validations: "must be", "shall not exceed"
   - Look for DMN references: "reg001-xxx.dmn"

6. **Configuration:**
   - Look for SET-xx codes (master data settings)
   - Look for CONF-xx codes (process configurations)

### Stop Condition

**STOP and inform the user if:**
- Functional requirements folder is empty
- FR-Coverage-Matrix shows gaps
- Key subprocesses lack FR documentation

---

## 4. Solution Design Structure

Generate content ONLY under:

```text
02-solution-design/
├── 2.1-domain-model/
├── 2.2-architecture/
├── 2.3-services/
├── 2.4-ui-interaction-design/
├── 2.5-cross-cutting/
├── 2.6-data-architecture/
└── 2.7-service-internal-design/
```

**DO NOT invent additional top-level folders.**

---

## 5. Service Naming Standard

All services MUST follow this naming pattern:

```text
[Scope][BusinessCapability]Service
OR
[Scope][BusinessCapability]ProcessService
```

**Rules:**
- Use business language, not technical terms
- Avoid project codes (e.g., REG001) in names
- Avoid generic names (e.g., CoreService)
- Use PascalCase
- End with "Service"

**Examples:**
- `TemporaryRegistrationProcessService`
- `VesselProfileService`
- `FeeCalculationService`

**Always include:** Logical Service ID (e.g., LS-REG-ORCH) for traceability

---

## 6. Section-Specific Rules

### 6.1 `2.3-services/` – External View ONLY

**Include:**
- Service responsibility
- Owned capabilities
- Exposed APIs (high-level)
- Published/subscribed events
- Service state model

**Exclude:**
- Classes
- Functions
- Internal algorithms
- Database schemas

---

### 6.2 `2.7-service-internal-design/` – Required for Each Service

For EACH service, create a folder with this structure:

```text
<ServiceName>/
├── 1-domain-core/
├── 2-support/
├── 3-intelligence/
├── 4-resilience-governance/
└── _README.md
```

---

## 7. Service Internal Design Layers

### 7.1 `1-domain-core/` — WHAT the service IS

**Describe:**
- Aggregates and Aggregate Roots
- Entities and Value Objects
- Domain Services
- Domain Events
- Class Diagram (conceptual, technology-agnostic)

**DO NOT include:** controllers, frameworks, persistence details

**Example Aggregate:**
```text
Aggregate: RegistrationRequest
Root: RegistrationRequest
Entities:
- Applicant
- VesselSnapshot
- ApprovalStatus
Value Objects:
- RequestId
- CertificateNumber
```

---

### 7.2 `2-support/` — HOW it is USED

**Describe:**
- Application services (use cases)
- Commands and Queries (Mediator pattern)
- Ports and interfaces
- Repositories (interfaces only)
- Internal sequence diagrams
- State transitions

**Clean Architecture MUST be assumed.**

**Example Application Service:**
```text
RegisterVesselApplicationService
- submitRequest()
- validateEligibility()
- requestApprovals()
- finalizeIssuance()
```

**Example State Transitions:**
```text
DRAFT → SUBMITTED → VALIDATED → PAID → ISSUED
```

---

### 7.3 `3-intelligence/` — WHY decisions happen

**Describe:**
- Business rules
- Policies
- DMN or decision logic
- Validation strategies
- Configuration-driven behavior

**DO NOT hardcode rules.**

**Reference DMN files:**
```text
- reg001-fee-calculation.dmn
- reg001-vessel-age-validation.dmn
- reg001-inspection-required.dmn
```

---

### 7.4 `4-resilience-governance/` — ENTERPRISE concerns

**Describe:**
- Error handling policies
- Retry and compensation logic
- Idempotency strategy
- Audit events
- Security considerations

**Saga orchestration MUST be documented here when long-running workflows exist.**

---

## 8. Architectural Patterns

Apply patterns **intentionally** (justify their use):

| Pattern | When to Apply |
|---------|---------------|
| Clean Architecture | Mandatory inside each service |
| Vertical Slice | Across services for feature isolation |
| Mediator | For application services (CQRS-lite) |
| CQRS | Only when justified (state complexity) |
| Saga (orchestration) | For long-running workflows |
| Event-driven | Preferred for cross-service communication |

**Never apply patterns by default—justify their use.**

---

## 9. UI Interaction Design

When generating `2.4-ui-interaction-design/`:

**Include:**
- Screen flow
- Low-fidelity wireframes (textual/box-level)
- Process-to-screen mapping
- Error and empty states
- Backoffice UI considerations

**Exclude:**
- Visual design
- Colors, typography, branding

---

## 10. Output Quality Rules

- Be precise and structured
- Use business terminology
- Avoid speculative content
- Clearly state assumptions
- Maintain traceability to BPMN and capabilities
- Prefer tables and diagrams over prose

**If a decision is ambiguous, document the assumption explicitly.**

---

## 11. Key Principle (Critical)

> **Solution Design defines *boundaries, contracts, and behavior***  
> **Service Internal Design defines *how a microservice works inside***

❌ Putting classes/functions directly in Solution Design causes:
- Over-design
- Technology lock-in
- Endless rework

✅ Correct approach:
- **Solution Design → WHAT & WHO**
- **Service Internal Design → HOW (internals, still not code)**

---

## 12. Stop Conditions

**STOP and explain if:**
- Required inputs are missing
- Requested output violates architecture rules
- Capability mapping is incomplete
- BPMN subprocesses lack Functional Responsibility sections

---

## 13. What NOT to Include

❌ Real implementation code  
❌ Framework-specific annotations (Spring, .NET)  
❌ Database table definitions  
❌ UI component libraries  
❌ Full method implementations  

**You CAN include pseudo-signatures:**
```text
submitRequest(command: SubmitRegistrationCommand): Result
```

---

## 14. Policy Service & Master Data Separation Pattern

**CRITICAL:** When designing services in `02-solution-design/`, apply these architectural patterns for decision logic and reference data separation.

### 14.1 Core Principle

> **Separate decision logic (Policy) from reference data (Master Data) from process orchestration (Domain Services).**

### 14.2 Three-Tier Decision Architecture

```text
┌──────────────────────────┐
│   Domain/Process Service │  ← Orchestration ONLY
│   (e.g., Registration)   │
└─────────┬────────────────┘
          │ calls (never owns rules)
          ▼
┌──────────────────────────┐
│   Policy Microservice    │  ← Decision Logic
│   - DMN Engine           │
│   - Rule Evaluation APIs │
│   - Configurable Policies│
└─────────┬────────────────┘
          │ reads (lookup only)
          ▼
┌──────────────────────────┐
│ Master Data Microservice │  ← Reference Data
│   - Lookups & Codes      │
│   - Static Reference     │
└──────────────────────────┘
```

### 14.3 Service Responsibility Boundaries

| Service Type | Owns | Does NOT Own |
|--------------|------|--------------|
| **Policy Service** | DMN files, rule evaluation, configurable thresholds, policy versioning | Master data, process orchestration, external integrations |
| **Master Data Service** | Reference codes, lookup tables, static categories | Decision logic, business rules, policies |
| **Domain/Process Service** | Workflow orchestration, state transitions, domain aggregates | Fee calculation, validation rules, eligibility decisions |

### 14.4 DMN Ownership Rules

#### Single Responsibility per DMN

- Each DMN file = One decision type
- **DO NOT** combine multiple concerns in one DMN

| Decision Type | Separate DMN File |
|---------------|-------------------|
| Fee calculation | `{service_code}-fee-calculation.dmn` |
| Document requirements | `{service_code}-documents-required.dmn` |
| Eligibility/validation | `{service_code}-{entity}-validation.dmn` |
| Inspection required | `{service_code}-inspection-required.dmn` |
| Penalty calculation | `{service_code}-penalty-calculation.dmn` |
| External approval required | `{service_code}-{external}-required.dmn` |

#### Explicit Inputs Rule

❌ Never pass complex objects:
```text
input: vessel
```

✅ Always use explicit primitive fields:
```text
vesselCategory, grossTonnage, length, buildYear, activityType
```

### 14.5 Configuration vs Decision Logic

| Content Type | Where It Lives | Change Frequency |
|--------------|----------------|------------------|
| Decision formulas | DMN | Rarely |
| Thresholds & limits | Config | Periodically |
| Timing rules | Config | Periodically |
| Feature flags | Config | Frequently |

**Rule:** DMN reads configuration at runtime. Config changes do NOT require DMN redeployment.

### 14.6 DMN Versioning Principles

1. **DMN files are externalized** – Not embedded in service code
2. **Version independently** – `{decision}_v{major}.{minor}.dmn`
3. **No recompile for rule changes** – Load from database, storage, or volume
4. **Runtime selection** – Use `decisionKey` + `versionTag`

| Change Type | Required Action |
|-------------|-----------------|
| Business rule logic | Deploy new DMN version |
| Threshold/config | Update config store only |
| Service code | Redeploy service |

### 14.7 Domain Service Interaction Pattern

**Domain/Process services MUST:**
- Call Policy service for all decisions
- Treat policy responses as **authoritative**
- Never implement decision logic locally

**Domain/Process services MUST NOT:**
- Calculate fees
- Validate eligibility
- Determine required documents
- Duplicate any DMN logic

### 14.8 Golden Rules (Enforce in All Designs)

| # | Rule | Violation Example |
|---|------|-------------------|
| 1 | **No DMN duplication** | Same fee logic in two services |
| 2 | **No policy logic in domain services** | Registration service calculates fees |
| 3 | **No master data in policy service** | Policy service stores vessel categories |
| 4 | **DMN changes ≠ service redeploy** | DMN embedded in JAR/DLL |
| 5 | **Config drives behavior, DMN drives decisions** | Hardcoded thresholds in DMN |

### 14.9 When to Apply This Pattern

Apply Policy/MasterData separation when the service has:

- [ ] Fee or payment calculations
- [ ] Configurable validation rules
- [ ] Document/requirement determination
- [ ] Eligibility or approval logic
- [ ] Penalty or fine calculations
- [ ] External system routing decisions

### 14.10 Design Checklist for 02-solution-design

When generating service designs, verify:

- [ ] Policy decisions are NOT in domain service
- [ ] DMN files follow single-responsibility
- [ ] Master data is separate from policy service
- [ ] Domain service calls policy service for decisions
- [ ] Configuration is separate from decision logic
- [ ] DMN versioning strategy is documented

---

## 15. Security Architecture (Cross-Cutting)

**CRITICAL:** Security is a cross-cutting concern that spans all services. For detailed security rules, see [SA-Security-instructions.md](SA-Security-instructions.md).

### 15.1 Two-Layer Authorization Model

> **RBAC determines where a user may enter the system.**  
> **ABAC determines what the user may do with a resource.**

| Layer | Purpose | Enforced At |
|-------|---------|-------------|
| **RBAC** | API access control | API Gateway |
| **ABAC** | Resource authorization | Policy Engine (OPA) |

### 15.2 RBAC Role Naming Convention

```text
{SERVICE_CODE}_{CAPABILITY}
```

**Examples:** `REG_CREATE_REQUEST`, `BILL_CONFIRM_PAYMENT`, `INSP_UPDATE_RESULT`

### 15.3 ABAC Ownership Pattern

```text
A resource may be owned by:
- An individual identity
- An organization or legal entity

Access is allowed ONLY if the subject has valid authority over the owning entity.
```

### 15.4 Security Non-Goals (MANDATORY)

| ❌ Non-Goal | Reason |
|------------|--------|
| RBAC does NOT authorize resource ownership | RBAC is for API access only |
| ABAC does NOT control API exposure | ABAC is for resources only |
| BPMN contains NO authorization logic | Process orchestration only |
| DMN contains NO authorization logic | Decision logic only |
| Domain services do NOT make access decisions | Separation of concerns |

### 15.5 Output Location

```text
02-solution-design/2.4-cross-cutting/01-Security_Trust_Architecture.md
```

### 15.6 Related Instruction File

**For complete security rules:** [SA-Security-instructions.md](SA-Security-instructions.md)

---

## 16. Prompt Patterns

**Recognized Prompts:**
- "As an Architect, generate C4 Architecture for {service_name}"
- "As a SA, design service decomposition for {service_code}"
- "As a Solution Architect, create internal design for {service_name}"
- "Generate API contracts for {service_name}"
- "Design event schemas for {service_code}"
- "Generate security architecture for {service_code}"
- "Design authorization model for {service_name}"

---

## 17. Dependencies from BA Phase

Before generating Solution Design, verify:
1. ✅ Scope and Stakeholders documented
2. ✅ BPMN subprocesses with Functional Responsibility
3. ✅ Capability mapping complete
4. ✅ **ALL Functional Requirements read** (see Section 3.1):
   - ALL FR-SPxx files (subprocess requirements)
   - ALL UC-xx files (use cases)
   - ALL US-xx files (user stories)
   - ALL FR-INT-xx files (integrations)
   - ALL FR-CCxx files (cross-cutting)
   - FR-Coverage-Matrix verified
5. ✅ DMN rules documented in Resources

---

## 18. Related Instruction Files

### SA Sub-Instructions (Detailed)

| File | Scope |
|------|-------|
| [SA-Flowable-instructions.md](SA-Flowable-instructions.md) | BPMN, DMN, Process Orchestration |
| [SA-ServiceNaming-instructions.md](SA-ServiceNaming-instructions.md) | Service Naming Standards |
| [SA-ServiceInternalDesign-instructions.md](SA-ServiceInternalDesign-instructions.md) | Internal Design Layers (DDD) |
| [SA-UIInteraction-instructions.md](SA-UIInteraction-instructions.md) | Angular + BFF Pattern |
| [SA-Security-instructions.md](SA-Security-instructions.md) | Security, RBAC, ABAC, Authorization |

### BA Phase Inputs

| File | Scope |
|------|-------|
| [BA-instructions.md](BA-instructions.md) | Business Analysis inputs |
| [FR-instructions.md](FR-instructions.md) | Functional Requirements structure |
| [Process-instructions.md](Process-instructions.md) | BPMN subprocess design |

---

## 19. SA Instruction File Hierarchy

```text
SA-instructions.md (Main)
│
├── SA-Flowable-instructions.md
│   └── BPMN, DMN, Process Engine Rules
│
├── SA-ServiceNaming-instructions.md
│   └── Naming Standards, Categories
│
├── SA-ServiceInternalDesign-instructions.md
│   └── DDD Layers, Clean Architecture
│
├── SA-UIInteraction-instructions.md
│   └── Angular, BFF, UI Abstraction
│
└── SA-Security-instructions.md
    └── Security, RBAC, ABAC, Authorization
```

---

**Last Updated:** 2026-01-09  
**Applies To:** All Solution Architecture tasks in the BA workspace
