

# Functional Requirements – README

## 1. Purpose

This folder defines the **Functional Requirements (FR)** for **REG001 – Issuing Temporary Registration Certificate for a Ship or Marine Unit**.

Functional Requirements act as the **bridge between**:

* Business Scope & Objectives
* User Stories (business intent)
* Use Cases (system behavior)
* BPMN / Service Blueprint
* Capabilities & Integrations

---

## 2. Key Concepts (Important)

### 2.1 User Stories ≠ Use Cases ≠ Functional Requirements

They are **related but not the same**.

| Artifact                   | Answers                        | Audience              | Level              |
| -------------------------- | ------------------------------ | --------------------- | ------------------ |
| **User Story**             | *Why does the user need this?* | Business, Product     | High-level         |
| **Use Case**               | *How does the system behave?*  | BA, QA, Dev           | Functional         |
| **Functional Requirement** | *What must the system do?*     | Dev, QA, Architecture | Precise & testable |

---

## 3. How They Work Together (Correct Flow)

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

✔ **User Stories give context**
✔ **Use Cases describe behavior**
✔ **Functional Requirements make it buildable**

---

## 4. Where Each Artifact Lives (IMPORTANT)

### 4.1 Folder Structure (Final & Correct)

```text
04-functional-requirements/
│
├── README.md
│
├── 00-overview/
│   ├── FR-00-Overview.md
│   ├── Actors-and-Roles.md
│
├── 01-user-stories/
│   ├── US-01-Initiate-Registration.md
│   ├── US-02-Authenticate-Applicant.md
│   ├── US-03-Submit-Vessel-Data.md
│
├── 02-use-cases/
│   ├── UC-01-Initiate-Request.md
│   ├── UC-02-Authenticate-Applicant.md
│   ├── UC-03-Validate-Company.md
│   ├── UC-04-Process-Payment.md
│
├── 03-functional-requirements/
│   ├── FR-01-Initiate-Request.md
│   ├── FR-02-Applicant-Authentication.md
│   ├── FR-03-Applicant-Profiling.md
│   ├── FR-04-Vessel-Validation.md
│   ├── FR-09-Process-Payment.md
│
├── 04-cross-cutting/
│   ├── FR-CC01-Security.md
│   ├── FR-CC02-Notifications.md
│
└── 05-integrations/
    ├── FR-INT-Payment-Gateway.md
    ├── FR-INT-Bayan.md
```

---

## 5. User Stories (What & Why)

### 5.1 Purpose of User Stories

User Stories:

* Capture **user intent and business value**
* Do **NOT** describe technical behavior
* Are **not test cases**

### 5.2 User Story Template

```md
# US-09 – Pay Registration Fees

As an Applicant  
I want to pay the required registration fees  
So that I can receive the temporary registration certificate.
```

### 5.3 When to Create a User Story

Create a User Story when:

* A user goal exists
* Business value must be justified
* The action is visible to the user

---

## 6. Use Cases (System Interaction)

### 6.1 Purpose of Use Cases

Use Cases:

* Describe **system behavior step-by-step**
* Include alternate and failure paths
* Are directly traceable to BPMN (SPxx)

### 6.2 Use Case Template

```md
# UC-09 – Process Payment

## Primary Actor
Applicant

## Preconditions
- Invoice generated
- Name reserved

## Main Flow
1. Applicant selects payment method
2. System redirects to payment gateway
3. System receives payment confirmation
4. System updates payment status

## Alternate Flows
A1 – Payment failed  
A2 – Payment expired  

## Postconditions
- Payment successful OR
- Payment failed and name released

## Traceability
- BPMN: SP08
- Gateway: G15
```

---

## 7. Functional Requirements (What the System MUST Do)

### 7.1 Purpose of Functional Requirements

Functional Requirements:

* Are **atomic and testable**
* May be derived from **one or more use cases**
* Define validations, rules, decisions, and integrations

### 7.2 Functional Requirement Template

```md
# FR-09 – Process Payment

## Description
The system shall allow the applicant to pay the calculated registration fees.

## Functional Rules
FR-09.1 The system shall generate an invoice before payment.
FR-09.2 The system shall reserve the vessel name during the payment window.
FR-09.3 The system shall confirm payment status via the payment gateway.
FR-09.4 The system shall release the name if payment fails or expires.

## Error Handling
- Payment failure
- Gateway timeout

## Traceability
- User Story: US-09
- Use Case: UC-09
- BPMN: SP08
- Capability: CAP-21
```

---

## 8. Relationship Example (Very Important)

```text
US-09  →  UC-09  →  FR-09.x
```

✔ One User Story → One or more Use Cases
✔ One Use Case → Multiple Functional Requirements

---


## 10. Common Mistakes (Avoid These)

❌ Calling user stories “use cases”
❌ Putting technical steps in user stories
❌ Writing UI design in functional requirements
❌ One giant FR file for the whole service

---

## 11. Definition of Done (All Three)

### User Story

* [ ] Clear actor
* [ ] Clear business value

### Use Case

* [ ] Main + alternate flows
* [ ] Preconditions & postconditions
* [ ] BPMN traceability

### Functional Requirement

* [ ] Atomic and testable
* [ ] Rule-based
* [ ] Traceable to UC, BPMN, Capability

---

## Optimization 1- Add a Standard Functional Requirement Template

Create **one reusable FR template** .

### Recommended FR format (government-grade)

```md
FR-<SPxx>-<###>
Title:
Description:
Trigger:
Inputs:
Processing / Rules:
Outputs:
Success Criteria:
Failure Conditions:
Related Gateways:
Related Capabilities:
```

### Example (derived from your work)

```md
FR-SP03-001
Title: Validate vessel age against policy
Description: The system shall validate the vessel age using configured policy rules.
Trigger: Applicant submits vessel data.
Inputs: Vessel type, build year, material.
Processing / Rules: reg001-vessel-age-validation.dmn
Outputs: Pass or Fail decision.
Failure Conditions: Age exceeds configured threshold.
Related Gateways: G05
Related Capabilities: CAP-07
```


---

##  Optimization  2— Introduce “FR Coverage Matrix” (1-page table)

Add **one table** (single document):

| Capability ID | SP   | FR IDs      | Covered (Y/N) |
| ------------- | ---- | ----------- | ------------- |
| CAP-07        | SP03 | FR-SP03-001 | Y             |
| CAP-18        | SP08 | FR-SP08-002 | Y             |




