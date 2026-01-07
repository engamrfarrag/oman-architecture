# Solution Architect – Service Internal Design Instructions

> **Role:** Solution Architect  
> **Scope:** Service Internal Design, DDD, Clean Architecture  
> **Applies To:** `02-solution-design/2.7-service-internal-design/` folder  
> **Parent:** [SA-instructions.md](SA-instructions.md)

---

## 1. Purpose

This document defines the **Service Internal Design** layer that describes **how a microservice works inside**.

> **Solution Design defines *boundaries, contracts, and behavior***  
> **Service Internal Design defines *how a microservice works inside***

---

## 2. Key Principle (Critical)

❌ Putting classes/functions directly in Solution Design causes:
- Over-design
- Technology lock-in
- Endless rework

✅ Correct approach:
- **Solution Design (2.3-services)** → WHAT & WHO (external view)
- **Service Internal Design (2.7)** → HOW (internals, still not code)

---

## 3. Folder Structure Per Service

For EACH service, create a folder with this structure:

```text
2.7-service-internal-design/
│
├── <ServiceName>/
│   ├── 1-domain-core/
│   │   ├── 01-Aggregates.md
│   │   ├── 02-Entities.md
│   │   ├── 03-Value_Objects.md
│   │   ├── 04-Domain_Services.md
│   │   ├── 05-Domain_Events.md
│   │   └── 06-Class_Diagram.md
│   │
│   ├── 2-support/
│   │   ├── 01-Application_Services.md
│   │   ├── 02-Commands_Queries.md
│   │   ├── 03-Flowable_Process_Adapter.md
│   │   ├── 04-Ports_Interfaces.md
│   │   ├── 05-Repositories.md
│   │   ├── 06-Internal_Sequence_Diagrams.md
│   │   └── 07-State_Transitions.md
│   │
│   ├── 3-intelligence/
│   │   ├── 01-Business_Rules.md
│   │   ├── 02-Decision_Models_DMN.md
│   │   ├── 03-Policies.md
│   │   ├── 04-Validation_Strategies.md
│   │   └── 05-Rule_Evolution_Notes.md
│   │
│   ├── 4-resilience-governance/
│   │   ├── 01-Error_Handling_Policies.md
│   │   ├── 02-Idempotency_Strategy.md
│   │   ├── 03-Retry_Compensation.md
│   │   ├── 04-Audit_Events.md
│   │   └── 05-Security_Considerations.md
│   │
│   └── _README.md
│
└── _TEMPLATES/
    └── service-internal-design-template.md
```

---

## 4. Layer Definitions

### 4.1 `1-domain-core/` — WHAT the Service IS

This folder changes **least over time**.

**Answers:**
- What is the aggregate?
- What is the business truth?
- What rules must *never* be broken?

**Contents:**

| Document | Description |
|----------|-------------|
| 01-Aggregates | Aggregate roots with boundaries |
| 02-Entities | Entities within aggregates |
| 03-Value_Objects | Immutable value objects |
| 04-Domain_Services | Stateless domain logic |
| 05-Domain_Events | Events raised by aggregates |
| 06-Class_Diagram | Conceptual UML (no frameworks) |

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

**DO NOT include:**
- Controllers
- Frameworks
- Persistence details
- Workflow logic

---

### 4.2 `2-support/` — HOW it is USED

This folder changes **as workflows evolve**.

**Answers:**
- How do use cases interact with the domain?
- What commands exist?
- What is synchronous vs asynchronous?

**Contents:**

| Document | Description |
|----------|-------------|
| 01-Application_Services | Use-case oriented services |
| 02-Commands_Queries | CQRS command/query definitions |
| 03-Flowable_Process_Adapter | Flowable interaction points |
| 04-Ports_Interfaces | Inbound/outbound port definitions |
| 05-Repositories | Repository interfaces (not impl) |
| 06-Internal_Sequence_Diagrams | Call flows within service |
| 07-State_Transitions | State machine definitions |

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

**Clean Architecture MUST be assumed.**

---

### 4.3 `3-intelligence/` — WHY Decisions Happen

This is the **most underrated layer**.

**Answers:**
- Why did the system decide X?
- Which rules are configurable?
- Which logic is externalized (DMN, rules engine, AI)?

**Contents:**

| Document | Description |
|----------|-------------|
| 01-Business_Rules | Core business rules |
| 02-Decision_Models_DMN | DMN table references |
| 03-Policies | Configurable policies |
| 04-Validation_Strategies | Validation approach |
| 05-Rule_Evolution_Notes | How rules change over time |

**DMN References:**
```text
- reg001-fee-calculation.dmn
- reg001-vessel-age-validation.dmn
- reg001-inspection-required.dmn
```

**DO NOT hardcode rules.**

---

### 4.4 `4-resilience-governance/` — ENTERPRISE Concerns

**Non-negotiable in government systems.**

**Ensures:**
- Legal defensibility
- Recoverability
- Audit compliance

**Contents:**

| Document | Description |
|----------|-------------|
| 01-Error_Handling_Policies | Error classification and handling |
| 02-Idempotency_Strategy | Idempotency keys and dedup |
| 03-Retry_Compensation | Retry policies and compensation |
| 04-Audit_Events | Audit trail integration |
| 05-Security_Considerations | Service-level security |

**This is where Sagas, retries, penalties, and audit hooks are documented.**

---

## 5. Pattern Mapping

| Pattern | Where It Lives |
|---------|----------------|
| Clean Architecture | Whole structure |
| DDD Aggregates | `1-domain-core` |
| Mediator | `2-support/01-Application_Services` |
| CQRS | `2-support` (split commands/queries) |
| Saga (Orchestration) | `2-support` + `4-resilience-governance` |
| DMN / Rules | `3-intelligence` |
| Audit & Compliance | `4-resilience-governance` |

---

## 6. What Each Microservice MUST Have (Minimum)

### 6.1 Service Responsibility
- What this service owns
- What it explicitly does NOT own
- Data ownership boundary

### 6.2 Aggregates & Aggregate Roots
- Prevent chatty services
- Prevent data leakage

### 6.3 Application Services
- Use-case oriented
- Maps cleanly to BPMN steps

### 6.4 Internal Sequence Diagrams
- Function call order
- Transaction boundaries
- Sync vs async decisions

```text
API → ApplicationService → DomainService → Aggregate → Event
```

### 6.5 State Transitions
- Prevents illegal transitions
- Critical for orchestration services

### 6.6 Error & Retry Policy
- Retryable vs non-retryable
- Compensating actions
- Idempotency keys

---

## 7. What NOT to Include

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

## 8. Service Internal Design Template

### `_README.md` Template Per Service

```md
# {ServiceName} – Internal Design

## Service Identity
- **Logical ID:** LS-{XXX}
- **Service Name:** {ServiceName}
- **Category:** {Process | Domain | Integration | Decision | Platform}

## Responsibility
- {What this service owns}
- {What it does NOT own}

## Data Ownership
- {Owned aggregates}
- {Shared data references}

## Flowable Integration
- {Task types: Service Task, User Task, etc.}
- {Event types: Message, Timer, etc.}

## Contents
- [1-domain-core/](1-domain-core/) – Aggregates, Entities, Events
- [2-support/](2-support/) – Application Services, Commands, Sequences
- [3-intelligence/](3-intelligence/) – Rules, DMN, Policies
- [4-resilience-governance/](4-resilience-governance/) – Errors, Retries, Audit

## Traceability
- Capabilities: CAP-{XX}, CAP-{XX}
- BPMN: SP{XX}, SP{XX}
- DMN: {rule_files}
```

---

## 9. Example: REG001 Orchestrator

### Domain Core
- Aggregate: `RegistrationRequest`
- Root controls state transitions

### Support
- `SubmitRegistrationCommand`
- `HandlePaymentConfirmed`
- Orchestration logic

### Intelligence
- Which subprocess to trigger
- Timeout thresholds
- Eligibility rules

### Governance
- Idempotency on callbacks
- Compensations (release name, cancel invoice)

---

## 10. Quality Checklist

Before completing service internal design:

- [ ] All four layers are documented
- [ ] Aggregates are clearly defined
- [ ] Application services map to BPMN tasks
- [ ] DMN rules are referenced (not hardcoded)
- [ ] State transitions are explicit
- [ ] Error handling is documented
- [ ] Flowable adapter is defined

---

## 11. Related Instruction Files

- [SA-instructions.md](SA-instructions.md) – Main SA instructions
- [SA-Flowable-instructions.md](SA-Flowable-instructions.md) – Flowable integration
- [SA-ServiceNaming-instructions.md](SA-ServiceNaming-instructions.md) – Naming standards
- [FR-instructions.md](FR-instructions.md) – Functional Requirements input

---

**Last Updated:** 2026-01-07  
**Applies To:** All Service Internal Design tasks in the BA workspace
