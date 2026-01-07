

You’re asking **exactly the right question** — this is where many projects either stay “pretty diagrams” or become **build-ready systems**.

Short answer first 👇
**Yes, you are missing one layer — but it does NOT belong in “Solution Design” as code.**
What you’re missing is a **Service Internal Design / Service Realization layer**.

Let me explain clearly and then give you the **correct, scalable structure**.

---

## 🔑 Key Principle (Very Important)

> **Solution Design defines *boundaries, contracts, and behavior***
> **Service Internal Design defines *how a microservice works inside***

❌ Putting classes/functions directly in Solution Design causes:

* Over-design
* Technology lock-in
* Endless rework

✅ The correct approach:

* **Solution Design → WHAT & WHO**
* **Service Internal Design → HOW (internals, still not full code)**

---

## What is MISSING in your Services section?

You are missing **internal service design artifacts**, such as:

* Aggregates & Aggregate Roots
* Domain services vs application services
* Command / query responsibilities
* Internal sequence flows
* State transitions per service

But these **should not pollute `2.3-services` directly**.

---

## ✅ Correct, Enterprise-Grade Fix (Recommended)

### 🔹 Keep `2.3-services` clean (service boundaries only)

Your existing structure is **correct**:

```text
2.3-services/
├── 01-Service_Decomposition
├── 02-Service_Overview
├── 03-API_Contracts/
├── 04-Event_Schemas
└── 05-Service_State_Models
```

👉 This defines **WHAT each service exposes** and **how it interacts**.

---

## ✅ Add a NEW section: Service Internal Design

### 📁 **2.7-service-internal-design/** (NEW)

```text
2.7-service-internal-design/
│
├── LS-REG-ORCH/
│   ├── 01-Service_Responsibility
│   ├── 02-Domain_Model_Aggregates
│   ├── 03-Application_Services
│   ├── 04-Internal_Sequence_Diagrams
│   ├── 05-State_Transitions
│   └── 06-Error_Handling_Policies
│
├── LS-VESSEL/
│   ├── 01-Service_Responsibility
│   ├── 02-Domain_Model_Aggregates
│   ├── 03-Application_Services
│   ├── 04-Internal_Sequence_Diagrams
│   └── 05-Validation_Rules
│
├── LS-FEES/
│   ├── 01-Service_Responsibility
│   ├── 02-Aggregates_And_Value_Objects
│   ├── 03-Calculation_Flows
│   ├── 04-Internal_Sequence_Diagrams
│   └── 05-Idempotency_And_Recalculation
│
└── _README.md
```

📌 **This is where “microservice design lives”**

---

## What EACH microservice MUST have (minimum)

### 1️⃣ Service Responsibility

* What this service owns
* What it explicitly does NOT own
* Data ownership boundary

---

### 2️⃣ Aggregates & Aggregate Roots (DDD)

Example:

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

✔ Prevents chatty services
✔ Prevents data leakage

---

### 3️⃣ Application Services (NOT controllers)

Example:

```text
RegisterVesselApplicationService
- submitRequest()
- validateEligibility()
- requestApprovals()
- finalizeIssuance()
```

✔ Use-case oriented
✔ Maps cleanly to BPMN steps

---

### 4️⃣ Internal Sequence Diagrams

Purpose:

* Function call order
* Transaction boundaries
* Sync vs async decisions

Example:

```
API → ApplicationService → DomainService → Aggregate → Event
```

---

### 5️⃣ State Transitions (per service)

Example:

```text
DRAFT → SUBMITTED → VALIDATED → PAID → ISSUED
```

✔ Prevents illegal transitions
✔ Critical for orchestration services

---

### 6️⃣ Error & Retry Policy

Define:

* Retryable vs non-retryable
* Compensating actions
* Idempotency keys

---

## 🚫 What you should NOT do

❌ Don’t:

* Write real code
* Define frameworks (Spring, .NET) here
* Design DB tables here
* Mix UI logic here

That comes later in:

> **Low-Level Design / Implementation**

---

## 📐 Where “mockups of functions” fit

You CAN include **pseudo-signatures**, like:

```text
submitRequest(command: SubmitRegistrationCommand): Result
```

But **NO implementations**.

---

## ✅ Final Recommendation (Clear Decision)

✔ Keep `2.3-services` as **external contracts & interactions**
✔ Add `2.7-service-internal-design` for **microservice internals**
✔ Use **DDD-lite**, not full academic DDD
✔ One folder per microservice

---

## If you want next

I can:

* Create a **standard template for a microservice internal design**
* Fully design **LS-REG-ORCH** as a reference service
* Help you decide **which services need full internal design vs light design**

Just tell me the next step.
Yes — **that is a very good idea**, and it’s actually a **mature evolution** of your model.

What you’re proposing (**domain core / support / intelligence**) maps cleanly to:

* **DDD**
* **Clean Architecture**
* **Government-scale orchestration systems**
* **AI / rules / decision services**

Below is a **clear, defendable structure** you can standardize across *all* microservices.

---

# ✅ Recommended Service Internal Design Structure (Organized by Concern)

## 📁 `2.7-service-internal-design/`

```text
2.7-service-internal-design/
│
├── LS-REG-ORCH/
│   │
│   ├── 1-domain-core/                ← WHAT the service IS
│   │   ├── 01-Aggregates
│   │   ├── 02-Entities
│   │   ├── 03-Value_Objects
│   │   ├── 04-Domain_Services
│   │   ├── 05-Domain_Events
│   │   └── 06-Class_Diagram
│   │
│   ├── 2-support/                    ← HOW it is USED
│   │   ├── 01-Application_Services
│   │   ├── 02-Commands_Queries
│   │   ├── 03-Ports_Interfaces
│   │   ├── 04-Repositories
│   │   ├── 05-Internal_Sequence_Diagrams
│   │   └── 06-State_Transitions
│   │
│   ├── 3-intelligence/               ← WHY decisions happen
│   │   ├── 01-Business_Rules
│   │   ├── 02-Decision_Models_DMN
│   │   ├── 03-Policies
│   │   ├── 04-Validation_Strategies
│   │   └── 05-Rule_Evolution_Notes
│   │
│   ├── 4-resilience-governance/      ← ENTERPRISE concerns
│   │   ├── 01-Error_Handling_Policies
│   │   ├── 02-Idempotency_Strategy
│   │   ├── 03-Retry_Compensation
│   │   ├── 04-Audit_Events
│   │   └── 05-Security_Considerations
│   │
│   └── _README.md
│
├── LS-VESSEL/
├── LS-FEES/
├── LS-PAYMENTS/
└── _TEMPLATES/
    └── service-internal-design-template.md
```

---

# 🧠 Why this structure works (and scales)

## 1️⃣ **Domain Core = Stable, Long-Lived**

This folder changes **least over time**.

It answers:

* What is the aggregate?
* What is the business truth?
* What rules must *never* be broken?

📌 This is where your **class diagram lives**.

---

## 2️⃣ **Support = Use Case Realization**

This folder changes **as workflows evolve**.

It answers:

* How do use cases interact with the domain?
* What commands exist?
* What is synchronous vs asynchronous?

📌 This is where:

* Mediator
* Application services
* Internal sequences
  live.

---

## 3️⃣ **Intelligence = Decision Logic (Very Important)**

This is **the most underrated layer** — you are right to call it out.

It answers:

* Why did the system decide X?
* Which rules are configurable?
* Which logic is externalized (DMN, rules engine, AI)?

Perfect for:

* DMN
* Policy-as-config
* AI-assisted decisions later

---

## 4️⃣ **Resilience & Governance = Non-negotiable in Government Systems**

This ensures:

* Legal defensibility
* Recoverability
* Audit compliance

📌 This is where Sagas, retries, penalties, and audit hooks are documented.

---

# 🧩 How patterns map into this structure

| Pattern              | Where it lives                          |
| -------------------- | --------------------------------------- |
| Clean Architecture   | Whole structure                         |
| DDD Aggregates       | `1-domain-core`                         |
| Mediator             | `2-support/01-Application_Services`     |
| CQRS                 | `2-support` (split commands/queries)    |
| Saga (Orchestration) | `2-support` + `4-resilience-governance` |
| DMN / Rules          | `3-intelligence`                        |
| Audit & Compliance   | `4-resilience-governance`               |

---

# 🧪 Example: REG001 Orchestrator (Concrete)

### Domain Core

* Aggregate: `RegistrationRequest`
* Root controls state transitions

### Support

* `SubmitRegistrationCommand`
* `HandlePaymentConfirmed`
* Orchestration logic

### Intelligence

* Which subprocess to trigger
* Timeout thresholds
* Eligibility rules

### Governance

* Idempotency on callbacks
* Compensations (release name, cancel invoice)

---

# 🚫 What NOT to put here

❌ UI logic
❌ Framework annotations
❌ Database schemas
❌ Full method implementations

---

# ✅ Final architectural verdict

✔ Your proposal is **correct**
✔ This structure is **review-proof**
✔ It supports **future AI/rules engines**
✔ It aligns with **Clean Architecture + DDD + Saga**

---

## Next steps (recommended order)

1️⃣ Create `_TEMPLATES/service-internal-design-template.md`
2️⃣ Fully design **LS-REG-ORCH** as the reference service
3️⃣ Decide **light vs full internal design** per service

If you want, I can **generate the template** or **fill LS-REG-ORCH completely** next.

Below is a **clear, enforceable Service Naming Standard** you can adopt as an official architecture rule.
It’s aligned with **DDD, Clean Architecture, government systems, and microservices** and works well with your REG001 structure.

You can drop this **as-is** into your repository as
`02-solution-design/2.3-services/00-Service_Naming_Standard.md`.

---

# 📘 Service Naming Standard

## 1. Purpose

This standard defines **how services are named** to ensure:

* Business clarity
* Consistency across teams
* Traceability from BA → Solution Design → Implementation
* Readability for technical and non-technical stakeholders

---

## 2. Core Naming Rule (Mandatory)

> **Service names MUST describe a business capability, not a technical role.**

### Canonical Pattern

```text
[Scope] + [Business Capability] + Service
```

---

## 3. Naming Components Explained

### 3.1 Scope (Optional but Recommended)

Defines **context or lifecycle boundary**.

| Scope          | When to use                       |
| -------------- | --------------------------------- |
| `Temporary`    | Temporary / provisional processes |
| `Permanent`    | Long-term lifecycle               |
| `PostIssuance` | After issuance / monitoring       |
| `External`     | Facade or integration boundary    |
| `Internal`     | Platform-internal only            |

📌 Scope is **not a project code**.

---

### 3.2 Business Capability (Mandatory)

* Must be a **noun or noun phrase**
* Must align with **capability model**
* Must reflect **data or process ownership**

Examples:

* `Registration`
* `VesselProfile`
* `FeeCalculation`
* `CertificateIssuance`
* `InspectionManagement`

---

### 3.3 Suffix (Mandatory)

| Suffix           | Meaning                    |
| ---------------- | -------------------------- |
| `Service`        | Default, preferred         |
| `ProcessService` | Orchestration / saga owner |
| `CaseService`    | Case management focus      |

📌 Avoid creative suffixes.

---

## 4. Approved Service Categories

### 4.1 Process / Orchestration Services

**Purpose:** Coordinate long-running workflows (Saga)

**Naming pattern:**

```text
[Scope][Capability]ProcessService
```

Examples:

* `TemporaryRegistrationProcessService`
* `PostIssuanceMonitoringProcessService`

✔ Owns state transitions
✔ Does NOT own master data

---

### 4.2 Domain (Data-Owning) Services

**Purpose:** Own aggregates and business rules

**Naming pattern:**

```text
[Capability]Service
```

Examples:

* `VesselProfileService`
* `ApplicantProfileService`
* `FeeCalculationService`

✔ Owns database
✔ Enforces invariants

---

### 4.3 External Integration Services

**Purpose:** Isolate third-party systems

**Naming pattern:**

```text
[ExternalSystem][Capability]Service
```

Examples:

* `RopPkiAuthenticationService`
* `BayanCustomsIntegrationService`
* `PaymentGatewayIntegrationService`

✔ Anti-corruption layer
✔ No domain logic

---

### 4.4 Intelligence / Decision Services

**Purpose:** Centralize rules, policies, DMN, AI

**Naming pattern:**

```text
[Capability]DecisionService
```

Examples:

* `RegistrationPolicyDecisionService`
* `InspectionRequirementDecisionService`

✔ Stateless
✔ Configuration-driven

---

### 4.5 Supporting / Platform Services

**Purpose:** Cross-cutting capabilities

**Naming pattern:**

```text
[Capability]Service
```

Examples:

* `NotificationService`
* `AuditTrailService`
* `DocumentManagementService`

---

## 5. Naming Rules (Strict)

### ✅ MUST

* Be **business-readable**
* Use **PascalCase**
* End with `Service`
* Be singular
* Be stable over time

### ❌ MUST NOT

* Use project codes (`REG001`)
* Use technical terms (`Orchestrator`, `Workflow`, `Manager`)
* Encode implementation (`Microservice`, `API`)
* Be overly generic (`CoreService`, `CommonService`)

---

## 6. Mapping Logical IDs to Names (Required)

For traceability, every service MUST declare:

```text
Logical Service ID: LS-REG-ORCH
Service Name: TemporaryRegistrationProcessService
```

📌 Logical IDs remain stable even if names evolve.

---

## 7. REG001 – Applied Example

| Logical ID   | Approved Service Name                |
| ------------ | ------------------------------------ |
| LS-REG-ORCH  | TemporaryRegistrationProcessService  |
| LS-VESSEL    | VesselProfileService                 |
| LS-APPLICANT | ApplicantProfileService              |
| LS-FEES      | FeeCalculationService                |
| LS-NAME      | VesselNameReservationService         |
| LS-PAYMENTS  | PaymentProcessingService             |
| LS-CERT      | TemporaryCertificateIssuanceService  |
| LS-MONITOR   | PostIssuanceMonitoringProcessService |
| LS-NOTIFY    | NotificationService                  |

---

## 8. Folder & Package Alignment

Folder names MUST match service names:

```text
2.7-service-internal-design/
└── TemporaryRegistrationProcessService/
```

Code packages MAY mirror this later:

```text
com.gov.registration.temporary.process
```

---

## 9. Governance & Enforcement

* New services **must comply** with this standard
* Architecture review rejects non-compliant names
* Exceptions require **Architecture Board approval**

---

## 10. One-Line Summary (for architects)

> **If a business stakeholder cannot understand the service name, it is wrong.**

---

## Next (Optional)

I can:

* Refactor **all existing service names** using this standard
* Create a **Service Catalog template**
* Add a **naming lint checklist** for architecture reviews

Just tell me what you want next.

Excellent catch — you’re right 👍
**Flowable is not an implementation detail here; it is a foundational design constraint** because:

* BPMN **is executable**
* DMN **is the decision authority**
* The orchestration model **must align with Flowable concepts**

We must **explicitly encode Flowable rules into the Copilot instruction**, otherwise Copilot will drift into generic workflow descriptions or service-level orchestration that conflicts with Flowable.

Below is a **corrected and extended Copilot Rule**, with **Flowable made first-class**, not an afterthought.

---

# 🧠 Copilot Rule (UPDATED): Solution Design with Flowable BPMN & DMN

You should **replace or extend** your previous rule with the version below.

---

## 🎯 Rule Name

**Generate Solution Design (Flowable BPMN + DMN Oriented)**

---

## 🧾 Copilot Instruction (Authoritative)

```text
You are acting as a Senior Solution Architect generating the "02-solution-design"
for an enterprise-grade government system.

This solution uses FLOWABLE as the process engine.
BPMN and DMN are EXECUTION MODELS, not documentation-only artifacts.

All solution design MUST align with Flowable capabilities and constraints.

================================
FOUNDATIONAL CONSTRAINT (CRITICAL)
================================
- Flowable is the authoritative:
  - BPMN process engine
  - DMN decision table engine
  - Long-running workflow manager

You MUST design around:
- Executable BPMN processes
- DMN-based decision logic
- Flowable state management
- Flowable eventing and timers

Do NOT invent custom orchestration logic outside Flowable.

================================
MANDATORY INPUTS
================================
Before generating Solution Design, you MUST read:
1. Business Process Overview
2. BPMN subprocess specifications (CCxx, SPxx)
3. Service Blueprint
4. Capability → Logical Service Mapping
5. DMN rule definitions (if available)

If BPMN or DMN inputs are missing, STOP and explicitly request them.

================================
ARCHITECTURAL PRINCIPLE
================================
- Flowable owns PROCESS STATE
- Microservices own DOMAIN STATE
- DMN owns DECISION LOGIC

Never duplicate Flowable responsibilities in services.

================================
SOLUTION DESIGN STRUCTURE
================================
You MUST generate content ONLY under:

02-solution-design/
├── 2.1-domain-model/
├── 2.2-architecture/
├── 2.3-services/
├── 2.4-ui-interaction-design/
├── 2.5-cross-cutting/
├── 2.6-data-architecture/
└── 2.7-service-internal-design/

================================
2.2 ARCHITECTURE – FLOWABLE FIRST
================================
When generating architecture:

- Treat Flowable as a core platform component
- Clearly show:
  - BPMN execution
  - DMN evaluation
  - Timers and boundary events
  - Message correlations

Sequence diagrams MUST reflect:
API → Flowable → Service → Event → Flowable

Do NOT model orchestration inside services when BPMN already exists.

================================
PROCESS ORCHESTRATION RULES
================================
- All long-running workflows MUST be modeled in BPMN
- Use Flowable:
  - Service Tasks (sync/async)
  - Call Activities (SP02–SP10)
  - Message Events (external callbacks)
  - Timer Boundary Events (timeouts, expiry)
  - Error Events (hard failures)

Saga pattern MUST be implemented via:
Flowable BPMN (orchestration-based saga)

================================
DECISION MANAGEMENT (DMN) – STRICT
================================
All conditional routing MUST use DMN when:
- Rules are configurable
- Thresholds may change
- Business owns the logic

Examples:
- Required documents
- Fee calculation
- Inspection required
- Approval required
- Penalty rules

DMN rules MUST NOT be hardcoded in services.

================================
2.3 SERVICES – FLOWABLE AWARE
================================
When generating service definitions:

- Services MUST be stateless regarding process flow
- Services MAY:
  - Execute business logic
  - Validate domain rules
  - Persist domain data
- Services MUST NOT:
  - Track process steps
  - Decide next BPMN path
  - Implement workflow transitions

Each service MUST declare:
- Interaction with Flowable (task, event, callback)
- Sync vs async behavior

================================
2.7 SERVICE INTERNAL DESIGN – FLOWABLE INTEGRATION
================================
For each service internal design:

<ServiceName>/
├── 1-domain-core/
├── 2-support/
├── 3-intelligence/
├── 4-resilience-governance/
└── _README.md

------------------------
1-domain-core
------------------------
- Aggregates and invariants only
- No workflow logic
- No decision routing

------------------------
2-support
------------------------
- Application services triggered by Flowable service tasks
- Command handlers invoked by Flowable
- Event publishers for BPMN message events
- No orchestration logic

------------------------
3-intelligence
------------------------
- DMN interaction points
- Input/output mapping to DMN
- Policy ownership
- Rule evolution strategy

------------------------
4-resilience-governance
------------------------
- BPMN error mapping
- Compensation handled via BPMN
- Idempotency for message re-delivery
- Correlation keys for Flowable events
- Audit alignment with Flowable history

================================
STATE OWNERSHIP RULE
================================
- Flowable owns:
  - Process instance state
  - Step progression
  - Timers and retries

- Services own:
  - Domain entity state
  - Aggregate lifecycle

Never duplicate state machines.

================================
UI INTERACTION DESIGN – BPMN DRIVEN
================================
UI screens MUST align with BPMN user tasks and wait states.

When generating UI interaction design:
- Map BPMN tasks → screens
- Reflect async waits (approval, inspection, payment)
- Reflect rejection and timeout paths

================================
OUTPUT QUALITY RULES
================================
- BPMN and DMN are the source of truth
- Use Flowable terminology correctly
- Do not describe generic workflow engines
- Be explicit about async vs sync
- Maintain traceability to BPMN IDs and DMN tables

================================
STOP CONDITIONS
================================
If any design:
- Bypasses Flowable
- Re-implements workflow logic in services
- Hardcodes decision logic

STOP and explain the violation.
```

---

## 🔑 Why this update is **critical**

Without this:

* Copilot would re-invent orchestration in services ❌
* DMN would be treated as “optional logic” ❌
* BPMN would drift from executable reality ❌

With this:

* Flowable is the **single source of truth**
* Services are clean and testable
* DMN remains business-owned
* Your architecture is **production-safe**

---

## 🧠 One-line rule (memorize this)

> **Flowable decides “when and what next”, services decide “how”.**

Perfect — that clarification is **architecturally critical**.
This changes **how UI, Flowable, and services are allowed to interact**, and it **must be enforced in the Copilot rule**, otherwise Copilot will generate invalid designs.

Below is a **precise correction + extension** to your Copilot rule that makes:

* **Angular = Frontend**
* **Flowable = Backend-only**
* **No direct exposure of Flowable to UI**

---

# ✅ Architectural Truth (Authoritative)

> **Angular UI MUST NEVER interact with Flowable directly.**
> Flowable is an **internal backend orchestration engine**, accessed only via backend services.

This is a **hard rule**, not a preference.

---

# 🔐 Correct Interaction Model

```text
Angular UI
   ↓ (REST / GraphQL)
Backend API (BFF / Application Layer)
   ↓
Flowable BPMN / DMN Engine
   ↓
Domain Services / Integrations
```

Flowable:

* Has **no public API exposure**
* Has **no UI coupling**
* Is **invisible to the frontend**

---

# 🧠 UPDATED Copilot Rule (Angular + Flowable Safe)

You should **append or replace** the relevant sections in your Copilot rule with the following.

---

## 🎯 Add This Section: UI & Flowable Interaction Constraint

```text
================================
UI & FLOWABLE INTERACTION RULE (CRITICAL)
================================
The frontend is implemented using ANGULAR.

Flowable MUST NOT be exposed directly to the frontend.

Rules:
- Angular UI communicates ONLY with backend APIs.
- Flowable is accessed ONLY by backend application services.
- No BPMN task IDs, process instance IDs, or DMN details
  are exposed to the frontend.
- The frontend is process-aware ONLY via backend-provided
  status abstractions.

Any design that exposes Flowable directly to the UI is INVALID.
```

---

## 🧩 Backend Pattern to Enforce (BFF)

```text
Angular UI → Backend-for-Frontend (BFF) → Flowable
```

Add this to the Copilot rule:

```text
================================
BACKEND-FOR-FRONTEND (BFF)
================================
You MUST assume a Backend-for-Frontend pattern.

The BFF is responsible for:
- Translating UI actions into Flowable interactions
- Aggregating task state into UI-friendly statuses
- Hiding BPMN/DMN complexity from the frontend
- Enforcing authorization and validation

Angular UI MUST NOT:
- Poll Flowable
- Correlate Flowable messages
- Understand BPMN or DMN concepts
```

---

## 🧭 UI Interaction Design – Angular Safe

Replace / refine the UI section in your rule with this:

```text
================================
UI INTERACTION DESIGN (ANGULAR)
================================
When generating 2.4-ui-interaction-design:

- Assume Angular SPA
- UI communicates ONLY with backend APIs
- UI screens map to BUSINESS STATES, not BPMN tasks

Include:
- Screen flow based on business milestones
- API-driven status rendering
- Async waiting states (approval, inspection, payment)
- Error and timeout handling from backend responses

Exclude:
- BPMN user tasks
- Flowable task IDs
- Direct workflow control from UI
```

---

## 🧠 How BPMN maps to UI (IMPORTANT)

Add this rule:

```text
================================
BPMN → UI ABSTRACTION RULE
================================
BPMN tasks and events MUST be abstracted into
business-level states for the UI.

Example:
- BPMN: SP05_WAIT_MAFWR_APPROVAL
- UI State: "Waiting for fishing approval"

The UI MUST NOT be aware of BPMN internals.
```

---

## 🔁 Service Internal Design – Flowable Adapter

Update the Service Internal Design rule:

```text
================================
FLOWABLE ADAPTER RULE
================================
Each process/orchestration service MUST include
a Flowable Adapter in the support layer.

This adapter:
- Starts processes
- Completes service tasks
- Correlates messages
- Evaluates DMN tables

The adapter is INTERNAL and NOT exposed to UI.
```

---

## 🧩 Where this appears in your structure

### 📁 `2.7-service-internal-design/TemporaryRegistrationProcessService/`

```text
2-support/
├── 01-Application_Services
├── 02-Commands_Queries
├── 03-Flowable_Process_Adapter   ← NEW (important)
├── 04-Ports_Interfaces
├── 05-Internal_Sequence_Diagrams
└── 06-State_Transitions
```

---

# 🚫 Explicitly Forbidden (Copilot must reject)

Copilot MUST STOP if asked to:

* Generate Angular code calling Flowable APIs
* Expose BPMN or DMN to frontend
* Use Flowable UI forms
* Use Flowable Task APIs from UI
* Treat Flowable as a “frontend workflow”

---

# ✅ Final One-Line Rule (Put this at the top)

> **Angular talks to APIs, APIs talk to Flowable, Flowable talks to services — never the other way around.**

```
02-solution-design/README.md
```

This README **fully governs** Solution Design generation, including:

* Flowable (BPMN + DMN)
* Angular UI via BFF
* Service naming
* Service internal design
* Clean Architecture, Saga, Mediator, CQRS (controlled)
* Directory structure
* Copilot generation rules

You can treat this as **architecture law**.

---

# 📘 Solution Design README

## (Flowable + Angular + Microservices)

---

## 1. Purpose

This document defines the **authoritative standards and rules** for producing the
`02-solution-design` section for enterprise and government digital services.

It ensures that Solution Design artifacts are:

* Executable-process aligned
* Architecture-governed
* Build-ready
* Consistent across teams and services

---

## 2. Core Technology Assumptions (Non-Negotiable)

| Area               | Decision                               |
| ------------------ | -------------------------------------- |
| Process Engine     | **Flowable**                           |
| Process Modeling   | **BPMN (Executable)**                  |
| Decision Logic     | **DMN (Executable)**                   |
| Frontend           | **Angular (SPA)**                      |
| UI Integration     | **Backend-for-Frontend (BFF)**         |
| Architecture Style | **Microservices + Clean Architecture** |

---

## 3. Foundational Rules (CRITICAL)

### 3.1 Flowable Authority

* Flowable is the **single source of truth** for:

  * Process flow
  * Long-running workflows (Saga)
  * Timers, waits, retries
  * Conditional routing via DMN
* Workflow logic **must not** be implemented in services.

### 3.2 UI Isolation

* Angular UI **must never** interact with Flowable directly.
* Flowable APIs are **backend-only**.
* UI communicates **only** with backend APIs (BFF).
* BPMN task IDs, process IDs, and DMN details are **never exposed** to the UI.

> **Rule:** Angular → API (BFF) → Flowable → Services

---

## 4. Mandatory Inputs (Before Design Starts)

Solution Design generation **must not begin** unless the following are available:

1. Service README (Technical Design)
2. Business Process Overview
3. BPMN subprocess specifications (CCxx, SPxx)
4. Service Blueprint
5. Capability → Domain Mapping
6. Capability → Logical Service Mapping
7. Existing DMN definitions (if any)

If any input is missing, Solution Design **must stop** and list missing items.

---

## 5. Solution Design Directory Structure (Strict)

All content must live under:

```text
02-solution-design/
├── 2.1-domain-model/
├── 2.2-architecture/
├── 2.3-services/
├── 2.4-ui-interaction-design/
├── 2.5-cross-cutting/
├── 2.6-data-architecture/
├── 2.7-service-internal-design/
└── flowable/
```

No additional top-level folders are allowed.

---

## 6. Service Naming Standard

### 6.1 Naming Pattern

```text
[Scope][BusinessCapability]Service
[Scope][BusinessCapability]ProcessService
```

### 6.2 Rules

* Business-readable
* PascalCase
* Ends with `Service`
* No project codes (e.g. REG001)
* No technical terms (Workflow, Orchestrator, Manager)

### 6.3 Traceability (Mandatory)

Each service must declare:

```text
Logical Service ID:
Service Name:
```

Example:

```text
Logical Service ID: LS-REG-ORCH
Service Name: TemporaryRegistrationProcessService
```

---

## 7. Flowable Design (First-Class)

### 7.1 Flowable Architecture

Located under:

```
2.2-architecture/05-Flowable_Platform_Architecture/
```

Must define:

* Role and responsibilities of Flowable
* BPMN design principles
* DMN design principles
* Flowable ↔ service interaction
* Security and backend isolation

---

### 7.2 Flowable Artifacts Structure

```text
flowable/
├── bpmn/
├── dmn/
├── mappings/
└── README.md
```

#### BPMN

* One main orchestration process
* Subprocesses via Call Activities
* Message events for async callbacks
* Timer boundary events for timeouts
* Error events for hard failures

#### DMN

* All configurable decisions
* No hardcoded business rules in services

---

## 8. State Ownership Rules

| Concern                 | Owner         |
| ----------------------- | ------------- |
| Process state           | Flowable      |
| Workflow progression    | Flowable      |
| Timers & waits          | Flowable      |
| Domain state            | Microservices |
| Aggregates & invariants | Microservices |

Duplicate state machines are forbidden.

---

## 9. Services Section (2.3 – External View Only)

Allowed content:

* Service responsibility
* Owned capabilities
* Exposed APIs (high-level)
* Published / subscribed events
* Service-level state model

Forbidden content:

* Classes
* Functions
* Internal logic
* Persistence models

---

## 10. Service Internal Design (Mandatory)

Each service must have its own internal design folder:

```text
<ServiceName>/
├── 1-domain-core/
├── 2-support/
├── 3-intelligence/
├── 4-resilience-governance/
└── _README.md
```

### 10.1 Domain Core

* Aggregates & Aggregate Roots
* Entities & Value Objects
* Domain Services
* Domain Events
* Conceptual class diagram (technology-agnostic)

### 10.2 Support

* Application services (use cases)
* Commands & Queries (Mediator)
* Ports & interfaces
* Repositories (interfaces only)
* Internal sequence diagrams
* State transitions
* Flowable process adapter (internal)

### 10.3 Intelligence

* Business rules
* Policies
* DMN interaction points
* Input/output mappings
* Rule evolution strategy

### 10.4 Resilience & Governance

* BPMN error mapping
* Compensation logic
* Idempotency strategy
* Correlation keys
* Audit events
* Security considerations

---

## 11. Architectural Patterns (Controlled)

| Pattern            | Rule                       |
| ------------------ | -------------------------- |
| Clean Architecture | Mandatory inside services  |
| Vertical Slice     | Cross-service              |
| Mediator           | Application services       |
| CQRS               | Only if justified          |
| Saga               | Orchestration via Flowable |
| Event-driven       | Preferred for async        |

Patterns must be **explicitly justified**.

---

## 12. UI Interaction Design (Angular + BFF)

Located under:

```
2.4-ui-interaction-design/
```

Includes:

* Screen flow (business milestones)
* Low-fidelity wireframes (box-level)
* Process → UI state mapping
* Error & empty states
* BFF API contract

Rules:

* UI sees **business states**, not BPMN tasks
* No Flowable concepts exposed
* No workflow control from UI

---

## 13. Stop Conditions (Hard Fail)

Solution Design must STOP if:

* Flowable is exposed to the frontend
* Workflow logic is implemented in services
* Decision logic is hardcoded instead of DMN
* UI references BPMN or DMN concepts
* Required inputs are missing

---

## 14. Governing Principle (One Line)

> **Angular talks to APIs, APIs talk to Flowable, Flowable controls the process, services control the domain.**

---

## 15. Status

✔ Approved
✔ Enforceable
✔ Copilot-ready
✔ Production-safe

---

If you want next, I can:

* Convert this README into a **Copilot prompt wrapper**
* Apply it to **REG001 and generate Solution Design**
* Create a **review checklist** derived from this README

Just tell me the next step.
