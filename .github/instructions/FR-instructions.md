# Functional Requirements (FR) Instructions

> **Role:** Business Analyst / Technical Writer  
> **Scope:** User Stories, Use Cases, Functional Requirements  
> **Applies To:** `01-business-analysis/1.5-functional-requirements/` folder

---

## 1. Purpose

This instruction file defines the rules for generating **Functional Requirements (FR)** for services.

Functional Requirements act as the **bridge between**:
- Business Scope & Objectives
- User Stories (business intent)
- Use Cases (system behavior)
- BPMN / Service Blueprint
- Capabilities & Integrations

---

## 2. Key Concepts

### 2.1 User Stories ≠ Use Cases ≠ Functional Requirements

| Artifact | Answers | Audience | Level |
|----------|---------|----------|-------|
| **User Story** | *Why does the user need this?* | Business, Product | High-level |
| **Use Case** | *How does the system behave?* | BA, QA, Dev | Functional |
| **Functional Requirement** | *What must the system do?* | Dev, QA, Architecture | Precise & testable |

---

## 3. Artifact Flow (Correct Order)

```text
Business Scope
   ↓
User Stories (business value)
   ↓
Use Cases (system interaction)
   ↓
Functional Requirements (implementable behavior)
   ↓
Design / Build / Test
```

✔ User Stories give context  
✔ Use Cases describe behavior  
✔ Functional Requirements make it buildable

---

## 4. Folder Structure

```text
1.5-functional-requirements/
│
├── README.md
│
├── 00-overview/
│   ├── FR-00-Overview.md
│   └── Actors-and-Roles.md
│
├── 01-user-stories/
│   ├── US-01-{Story-Name}.md
│   ├── US-02-{Story-Name}.md
│   └── ...
│
├── 02-use-cases/
│   ├── UC-01-{Use-Case-Name}.md
│   ├── UC-02-{Use-Case-Name}.md
│   └── ...
│
├── 03-functional-requirements/
│   ├── FR-01-{Requirement-Name}.md
│   ├── FR-02-{Requirement-Name}.md
│   └── ...
│
├── 04-cross-cutting/
│   ├── FR-CC01-Security.md
│   ├── FR-CC02-Notifications.md
│   └── ...
│
├── 05-integrations/
│   ├── FR-INT-{Integration-Name}.md
│   └── ...
│
└── 06-acceptance-criteria/
    └── Acceptance-Criteria-Matrix.md
```

---

## 5. User Story Template

```md
# US-{XX} – {Story Title}

As an {Actor}
I want to {goal/action}
So that {business value/reason}

## Acceptance Criteria
- [ ] {Criterion 1}
- [ ] {Criterion 2}
- [ ] {Criterion 3}

## Related
- Use Case: UC-{XX}
- BPMN: SP{XX}
- Capability: CAP-{XX}
```

**When to Create a User Story:**
- A user goal exists
- Business value must be justified
- The action is visible to the user

---

## 6. Use Case Template

```md
# UC-{XX} – {Use Case Title}

## Primary Actor
{Actor name}

## Preconditions
- {Precondition 1}
- {Precondition 2}

## Main Flow
1. {Step 1}
2. {Step 2}
3. {Step 3}

## Alternate Flows
**A1 – {Alternate Flow Name}**
- {Condition}: {Steps}

**A2 – {Alternate Flow Name}**
- {Condition}: {Steps}

## Exception Flows
**E1 – {Exception Name}**
- {Condition}: {Handling}

## Postconditions
- {Success condition} OR
- {Failure condition}

## Traceability
- User Story: US-{XX}
- BPMN: SP{XX}
- Gateway: G{XX}
- Capability: CAP-{XX}
```

---

## 7. Functional Requirement Template

### 7.1 Standard FR Format

```md
# FR-{XX} – {Requirement Title}

## Description
The system shall {action/behavior description}.

## Functional Rules
FR-{XX}.1 The system shall {specific rule 1}.
FR-{XX}.2 The system shall {specific rule 2}.
FR-{XX}.3 The system shall {specific rule 3}.

## Error Handling
- {Error scenario 1}
- {Error scenario 2}

## Traceability
- User Story: US-{XX}
- Use Case: UC-{XX}
- BPMN: SP{XX}
- Capability: CAP-{XX}
```

### 7.2 Government-Grade FR Format (Detailed)

```md
FR-SP{XX}-{###}
Title: {Requirement Title}
Description: {What the system shall do}
Trigger: {What initiates this requirement}
Inputs: {Required input data/fields}
Processing / Rules: {DMN file or business logic}
Outputs: {Expected outputs}
Success Criteria: {Measurable success conditions}
Failure Conditions: {Error scenarios}
Related Gateways: G{XX}
Related Capabilities: CAP-{XX}
```

**Example:**

```md
FR-SP03-001
Title: Validate vessel age against policy
Description: The system shall validate the vessel age using configured policy rules.
Trigger: Applicant submits vessel data.
Inputs: Vessel type, build year, material.
Processing / Rules: reg001-vessel-age-validation.dmn
Outputs: Pass or Fail decision.
Success Criteria: Vessel within allowed age range proceeds.
Failure Conditions: Age exceeds configured threshold.
Related Gateways: G05
Related Capabilities: CAP-07
```

---

## 8. FR Coverage Matrix

Add a **single coverage table** to track FR completeness:

| Capability ID | Subprocess | FR IDs | Covered (Y/N) |
|---------------|------------|--------|---------------|
| CAP-07 | SP03 | FR-SP03-001 | Y |
| CAP-18 | SP08 | FR-SP08-002 | Y |
| CAP-21 | SP09 | FR-SP09-001, FR-SP09-002 | Y |

**Location:** `1.5-functional-requirements/00-overview/FR-Coverage-Matrix.md`

---

## 9. Relationship Mapping

```text
US-{XX}  →  UC-{XX}  →  FR-{XX}.x
```

✔ One User Story → One or more Use Cases  
✔ One Use Case → Multiple Functional Requirements

---

## 10. Common Mistakes to Avoid

❌ Calling user stories "use cases"  
❌ Putting technical steps in user stories  
❌ Writing UI design in functional requirements  
❌ One giant FR file for the whole service  
❌ Missing traceability to BPMN/Capabilities  

---

## 11. Definition of Done

### User Story
- [ ] Clear actor
- [ ] Clear business value
- [ ] Acceptance criteria defined

### Use Case
- [ ] Main + alternate flows documented
- [ ] Preconditions & postconditions stated
- [ ] BPMN traceability included

### Functional Requirement
- [ ] Atomic and testable
- [ ] Rule-based with clear conditions
- [ ] Traceable to UC, BPMN, Capability
- [ ] Error handling defined

---

## 12. Dependencies

Before generating FRs, ensure these exist:
1. ✅ Subprocess documents (SPxx) with Functional Responsibility sections
2. ✅ Capability mapping (CAP to Domain and Logical Service)
3. ✅ BPMN process design with gateway decision rules
4. ✅ DMN rules in `Resources/dmn/{service_code}/`

---

## 13. Prompt Patterns

**Recognized Prompts:**
- "As a BA, generate functional requirements for {service_code}"
- "Generate user stories for {subprocess_name}"
- "Create use case for SP{XX}"
- "Generate FR-Coverage-Matrix for {service_name}"

---

## 14. Related Instruction Files

- [BA-instructions.md](BA-instructions.md) – Pre-FR optimization rules
- [Process-instructions.md](Process-instructions.md) – BPMN and subprocess design
- [SA-instructions.md](SA-instructions.md) – How FRs feed into Solution Design

---

**Last Updated:** 2026-01-07  
**Applies To:** All Functional Requirements generation in the BA workspace
