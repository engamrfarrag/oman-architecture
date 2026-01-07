# Business Analyst (BA) Instructions

> **Role:** Business Analyst  
> **Scope:** Business Analysis, Process Design, Capabilities, Pre-FR Optimization  
> **Applies To:** `01-business-analysis/` folder

---

## 1. Role Overview

As a Business Analyst, you are responsible for:
- Scope definition and stakeholder governance
- Service blueprint and business process overview
- BPMN process design and subprocess documentation
- Capability mapping (to domains and logical services)
- Pre-Functional Requirements optimization

---

## 2. Document Ownership

| Document Section | BA Responsibility |
|------------------|-------------------|
| `1.1-scope/` | Owner |
| `1.2-stakeholders/` | Owner |
| `1.3-process/` | Owner |
| `1.4-capabilities/` | Owner |
| `1.5-functional-requirements/` | Co-owner with SA |

---

## 3. Pre-FR Optimization Rules

### 3.1 Add "Functional Requirement Anchor" to Each Subprocess

At the top of **each SPxx document**, add:

```md
## Functional Responsibility
This subprocess is responsible for:
- <Responsibility 1>
- <Responsibility 2>
- <Responsibility 3>
```

**Example (SP03):**

```md
## Functional Responsibility
This subprocess is responsible for:
- Capturing vessel/unit master data
- Validating vessel age against configurable policy
- Detecting penalty triggers based on timing rules
- Blocking prohibited or banned vessels
```

**Why This Matters:**
- Each bullet becomes **1–2 Functional Requirements**
- Zero ambiguity during FR writing
- Perfect alignment with CAP mapping

---

### 3.2 Convert Gateways into Explicit Decision Rules

Under each Gateway section (e.g., G05, G11), add:

```md
### Decision Rule (Business)
- Inputs:
- Rule Source (DMN / Config):
- Outcomes:
```

**Example (G05):**

```md
### Decision Rule (Business)
- Inputs: vessel type, build year, material
- Rule Source: reg001-vessel-age-validation.dmn
- Outcomes:
  - Pass → continue
  - Fail → block issuance; allow re-entry if policy permits
```

**Why This Matters:**
- FRs become **unambiguous**
- QA can derive test cases
- DMN ownership is explicit

---

## 4. Subprocess Documentation Standards

### 4.1 SPxx Document Template

```md
# SP{XX} – {Subprocess Name}

## Functional Responsibility
This subprocess is responsible for:
- <Responsibility 1>
- <Responsibility 2>

## Swimlane
- **Actor:** {Applicant | System | Authority | External}

## Steps
1. {Step description}
2. {Step description}

## Gateways
### G{XX} – {Gateway Name}

#### Decision Rule (Business)
- Inputs: {input fields}
- Rule Source: {DMN file or Config}
- Outcomes:
  - {Outcome 1} → {Action}
  - {Outcome 2} → {Action}

## Related Capabilities
- CAP-{XX}: {Capability Name}

## Related DMN Rules
- {service_code}-{rule_name}.dmn

## Traceability
- BPMN: {main process reference}
- Depends On: SP{XX}
- Triggers: SP{XX}
```

---

## 5. Capability Mapping Rules

### 5.1 Capability to Domain Mapping

When mapping capabilities:
1. Identify the business domain each capability belongs to
2. Group capabilities by bounded context
3. Document shared capabilities across domains

**Output Location:** `1.4-capabilities/02-Capability_to_Domain_Mapping`

### 5.2 Capability to Logical Service Mapping

For each capability:
1. Identify which logical service(s) implement it
2. Mark primary vs supporting services
3. Include capability ID for traceability

**Output Location:** `1.4-capabilities/01-Capability_to_Logical_Service_Mapping`

---

## 6. BPMN Process Design Standards

### 6.1 Process Hierarchy

```text
Main Process (MP)
├── Subprocess 01 (SP01)
├── Subprocess 02 (SP02)
├── ...
├── Cross-Cutting (CC01, CC02)
└── End Events
```

### 6.2 Swimlane Assignment

| Swimlane | Description |
|----------|-------------|
| Applicant | External user initiating or responding |
| System | Automated platform actions |
| Authority | Internal staff/reviewers |
| External Entity | Third-party systems (Bayan, Banks, etc.) |

---

## 7. Documentation Flow (BA Perspective)

```text
Proposal
   ↓
Scope Definition
   ↓
Stakeholders & Governance
   ↓
Service Blueprint
   ↓
Business Process Overview
   ↓
BPMN Subprocess Design (with Functional Anchors)
   ↓
Capability Mapping
   ↓
[Handoff to FR Generation]
```

---

## 8. Quality Checklist for BA Documents

Before handoff to FR/SA:

- [ ] All subprocesses have Functional Responsibility sections
- [ ] All gateways have Decision Rules with DMN references
- [ ] Capabilities are mapped to domains and services
- [ ] BPMN diagrams match subprocess documentation
- [ ] Traceability is complete (SP → CAP → G → DMN)
- [ ] Bilingual content is consistent (if applicable)

---

## 9. Related Instruction Files

- [FR-instructions.md](FR-instructions.md) – Functional Requirements generation
- [Process-instructions.md](Process-instructions.md) – Detailed process design
- [SA-instructions.md](SA-instructions.md) – Solution Architecture handoff

---

## 10. Prompt Patterns for BA Role

**Recognized Prompts:**
- "As a BA, generate the scope for {service_name}"
- "As a Business Analyst, create service blueprint for {service_code}"
- "As a BA, document subprocess SP{XX} for {service_name}"
- "As a BA, map capabilities for {service_code}"

---

**Last Updated:** 2026-01-07  
**Applies To:** All Business Analysis tasks in the BA workspace
