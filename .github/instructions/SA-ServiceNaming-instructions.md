# Solution Architect – Service Naming Instructions

> **Role:** Solution Architect  
> **Scope:** Service Naming Standards, Categories, Traceability  
> **Applies To:** `02-solution-design/2.3-services/` folder  
> **Parent:** [SA-instructions.md](SA-instructions.md)

---

## 1. Purpose

This standard defines **how services are named** to ensure:
- Business clarity
- Consistency across teams
- Traceability from BA → Solution Design → Implementation
- Readability for technical and non-technical stakeholders

---

## 2. Core Naming Rule (Mandatory)

> **Service names MUST describe a business capability, not a technical role.**

### Canonical Pattern

```text
[Scope] + [Business Capability] + Service
```

---

## 3. Naming Components

### 3.1 Scope (Optional but Recommended)

Defines **context or lifecycle boundary**.

| Scope | When to Use |
|-------|-------------|
| `Temporary` | Temporary / provisional processes |
| `Permanent` | Long-term lifecycle |
| `PostIssuance` | After issuance / monitoring |
| `External` | Facade or integration boundary |
| `Internal` | Platform-internal only |

📌 **Scope is NOT a project code.**

---

### 3.2 Business Capability (Mandatory)

- Must be a **noun or noun phrase**
- Must align with **capability model**
- Must reflect **data or process ownership**

**Examples:**
- `Registration`
- `VesselProfile`
- `FeeCalculation`
- `CertificateIssuance`
- `InspectionManagement`

---

### 3.3 Suffix (Mandatory)

| Suffix | Meaning |
|--------|---------|
| `Service` | Default, preferred |
| `ProcessService` | Orchestration / saga owner |
| `CaseService` | Case management focus |

📌 **Avoid creative suffixes.**

---

## 4. Service Categories

### 4.1 Process / Orchestration Services

**Purpose:** Coordinate long-running workflows (Saga)

**Naming Pattern:**
```text
[Scope][Capability]ProcessService
```

**Examples:**
- `TemporaryRegistrationProcessService`
- `PostIssuanceMonitoringProcessService`

**Characteristics:**
- ✔ Owns state transitions
- ✔ Does NOT own master data

---

### 4.2 Domain (Data-Owning) Services

**Purpose:** Own aggregates and business rules

**Naming Pattern:**
```text
[Capability]Service
```

**Examples:**
- `VesselProfileService`
- `ApplicantProfileService`
- `FeeCalculationService`

**Characteristics:**
- ✔ Owns database
- ✔ Enforces invariants

---

### 4.3 External Integration Services

**Purpose:** Isolate third-party systems

**Naming Pattern:**
```text
[ExternalSystem][Capability]Service
```

**Examples:**
- `RopPkiAuthenticationService`
- `BayanCustomsIntegrationService`
- `PaymentGatewayIntegrationService`

**Characteristics:**
- ✔ Anti-corruption layer
- ✔ No domain logic

---

### 4.4 Intelligence / Decision Services

**Purpose:** Centralize rules, policies, DMN, AI

**Naming Pattern:**
```text
[Capability]DecisionService
```

**Examples:**
- `RegistrationPolicyDecisionService`
- `InspectionRequirementDecisionService`

**Characteristics:**
- ✔ Stateless
- ✔ Configuration-driven

---

### 4.5 Supporting / Platform Services

**Purpose:** Cross-cutting capabilities

**Naming Pattern:**
```text
[Capability]Service
```

**Examples:**
- `NotificationService`
- `AuditTrailService`
- `DocumentManagementService`

---

## 5. Naming Rules (Strict)

### ✅ MUST

- Be **business-readable**
- Use **PascalCase**
- End with `Service`
- Be singular
- Be stable over time

### ❌ MUST NOT

- Use project codes (`REG001`)
- Use technical terms (`Orchestrator`, `Workflow`, `Manager`)
- Encode implementation (`Microservice`, `API`)
- Be overly generic (`CoreService`, `CommonService`)

---

## 6. Logical ID to Name Mapping (Required)

For traceability, every service MUST declare:

```text
Logical Service ID: LS-REG-ORCH
Service Name: TemporaryRegistrationProcessService
```

📌 **Logical IDs remain stable even if names evolve.**

---

## 7. REG001 – Applied Example

| Logical ID | Approved Service Name |
|------------|----------------------|
| LS-REG-ORCH | TemporaryRegistrationProcessService |
| LS-VESSEL | VesselProfileService |
| LS-APPLICANT | ApplicantProfileService |
| LS-FEES | FeeCalculationService |
| LS-NAME | VesselNameReservationService |
| LS-PAYMENTS | PaymentProcessingService |
| LS-CERT | TemporaryCertificateIssuanceService |
| LS-MONITOR | PostIssuanceMonitoringProcessService |
| LS-NOTIFY | NotificationService |

---

## 8. Folder & Package Alignment

### Folder Names MUST Match Service Names:

```text
2.7-service-internal-design/
└── TemporaryRegistrationProcessService/
```

### Code Packages MAY Mirror This Later:

```text
com.gov.registration.temporary.process
```

---

## 9. Service Naming Checklist

Before approving a service name:

- [ ] Name describes business capability
- [ ] Name uses PascalCase
- [ ] Name ends with `Service` or `ProcessService`
- [ ] No project codes in name
- [ ] No technical terms in name
- [ ] Logical ID is assigned
- [ ] Name is business-readable

---

## 10. Governance & Enforcement

- New services **must comply** with this standard
- Architecture review rejects non-compliant names
- Exceptions require **Architecture Board approval**

---

## 11. One-Line Rule

> **If a business stakeholder cannot understand the service name, it is wrong.**

---

## 12. Related Instruction Files

- [SA-instructions.md](SA-instructions.md) – Main SA instructions
- [SA-ServiceInternalDesign-instructions.md](SA-ServiceInternalDesign-instructions.md) – Internal design layers
- [BA-instructions.md](BA-instructions.md) – Capability mapping

---

**Last Updated:** 2026-01-07  
**Applies To:** All service naming in the BA workspace
