Perfect — below is a **clean, generalized, production-grade Copilot rule set** that:

* **Merges everything cleanly**
* Keeps **RBAC (roles = departments) strictly for API access**
* Uses **ABAC (OPA) strictly for resource authorization**
* Is **domain-agnostic**
* Uses **Vessel only as an example**, not a hard dependency
* Is reusable for **any government service**
* Is safe for **DOCX-driven extraction**

This is written exactly in the style Copilot expects and can be **split into files or kept merged**.

---

# ✅ FINAL COPILOT RULE SET

**Cross-Cutting Security – Generalized (RBAC + ABAC)**

---

## 📁 File 1: `copilot-system.md`

```
You are acting as a Senior Government Solution Architect and Business Analyst.

You MUST derive all security, access, and authorization rules
from official source documents (DOCX), business analysis artifacts,
and approved architectural decisions.

You MUST NOT invent roles, ownership concepts, or access rules.
You MUST clearly separate API access control from resource authorization.
```

---

## 📁 File 2: `copilot-security-principles.md`

### 🎯 Objective

```
As a <BA & Solution Architect>,
Generate and update Business Analysis and Solution Design sections
to include a generalized, cross-cutting security model
that is reusable across services and domains.
```

---

### 🔐 Core Security Principles (MANDATORY)

#### 1️⃣ Authentication (Fact)

```
Authentication is external.
The platform trusts an external Identity Provider.
The system does not manage credentials.
```

Examples:

* National PKI
* Government SSO
* Federated identity

---

#### 2️⃣ Authorization is TWO DISTINCT LAYERS

```
Layer 1: RBAC → API access only
Layer 2: ABAC → Resource access only
```

Copilot MUST treat these as **independent concerns**.

---

## 📁 File 3: `copilot-rbac-api-access.md`

### 🧱 RBAC — API Access Control

Copilot MUST apply:

```
RBAC is used ONLY to authorize access to APIs.
RBAC roles represent organizational or functional responsibility.
RBAC roles do NOT grant access to specific resources.
```

#### Characteristics

| Aspect      | Rule                           |
| ----------- | ------------------------------ |
| Scope       | API / endpoint                 |
| Enforced at | API Gateway / Identity Adapter |
| Role source | HR / Organization              |
| Granularity | Coarse                         |
| Example     | Department / Unit              |

#### Documentation Language (BA & SA)

Copilot MUST use wording like:

* “Access to this API is restricted to users assigned to the relevant organizational role.”
* “Users without the required role are rejected before reaching backend services.”

Copilot MUST NOT:

* Use RBAC for ownership
* Use RBAC for entity-level authorization

---

## 📁 File 4: `copilot-abac-resource-authorization.md`

### 🧱 ABAC — Resource Authorization (OPA)

Copilot MUST apply:

```
ABAC is used ONLY to authorize actions on domain resources.
ABAC is independent of organizational roles.
```

#### ABAC Dimensions (Generalized)

| Dimension | Description                                            |
| --------- | ------------------------------------------------------ |
| Subject   | Identity attributes (e.g., civilId, representationIds) |
| Resource  | Domain entity (type, owner, status)                    |
| Action    | Intended operation                                     |
| Context   | Request or process context                             |

---

### 🧩 Ownership Pattern (GENERALIZED)

Copilot MUST generalize ownership as:

```
A resource may be owned by:
- An individual identity
- An organization or legal entity
```

Access is allowed ONLY if:

```
The subject has a valid relationship with the owning entity
according to authoritative systems.
```

---

### 📌 Example (Vessel – illustrative only)

> This example MUST be marked as **illustrative**.

```
If a vessel is owned by a company,
the user must be a legally authorized representative of that company.
```

Copilot MUST state clearly:

```
This pattern applies to all owned resources
(e.g., assets, licenses, requests, certificates),
not only vessels.
```

---

## 📁 File 5: `copilot-ba-updates.md`

### 📂 Target

`01-business-analysis/**`

### Section to Add

**Security & Access Control Rules**

---

### A️⃣ API Access Rules (RBAC)

Copilot MUST document:

* API access is controlled by organizational role.
* Roles are assigned externally (e.g., HR system).
* These rules apply uniformly across services.

---

### B️⃣ Resource Access Rules (Business Language)

Copilot MUST describe:

* Individual ownership
* Organizational ownership
* Legal representation / delegation
* Access only when valid authority exists

🚫 Copilot MUST NOT:

* Mention OPA
* Mention technical enforcement

---

## 📁 File 6: `copilot-solution-design-updates.md`

### 📂 Target

`02-solution-design/2.5-cross-cutting/01-Security_Trust_Architecture.md`

---

### 1️⃣ Two-Layer Authorization Model

Copilot MUST document:

```
RBAC controls which APIs may be accessed.
ABAC controls which resources may be acted upon.
```

---

### 2️⃣ Enforcement Points

| Layer | Enforced At                              |
| ----- | ---------------------------------------- |
| RBAC  | API Gateway                              |
| ABAC  | Authorization Middleware / Policy Engine |

---

### 3️⃣ Trust Boundaries

Copilot MUST identify:

* External Identity Provider
* Platform boundary
* Policy decision boundary

---

### 4️⃣ Explicit Non-Goals (MANDATORY)

Copilot MUST explicitly state:

* RBAC does not authorize resource ownership
* ABAC does not control API exposure
* BPMN contains no authorization logic
* DMN contains no authorization logic
* Domain services do not make access decisions

---

## 📁 File 7: `copilot-traceability.md`

### 📊 Security Traceability Table

Copilot MUST generate a table mapping **DOCX language** to **security interpretation**.

Example:

| Source Phrase | Interpreted As            |
| ------------- | ------------------------- |
| مالك          | Resource owner            |
| موكل          | Authorized representative |
| جهة           | Organizational role       |
| موظف          | API access role           |
| سجل           | Legal entity identifier   |

---

## 🛑 Global Hard Constraints (ENFORCED)

Copilot MUST NOT:

❌ Use RBAC for resource authorization
❌ Use ABAC for API access
❌ Create artificial roles (Owner, Admin, SuperUser)
❌ Embed security logic in BPMN or DMN
❌ Assume ownership without authoritative evidence

---

## 🧠 Final Architecture Statement (Canonical)

Copilot MUST include this statement verbatim:

> **RBAC determines where a user may enter the system.**
> **ABAC determines what the user may do with a resource.**

---

## ✅ Result

With these rules, Copilot will:

* Correctly read DOCX business language
* Produce **clean BA documentation**
* Produce **auditable security architecture**
* Avoid role explosion
* Avoid leaking security into business logic
* Be reusable across **all services**, not just vessels



* Split these into **Copilot role-specific rules (BA / SA / Dev)**
* Generate **security acceptance criteria templates**
* Produce **OPA policy documentation that mirrors BA language**
* Add **audit & compliance rules**

Just tell me the next move.

Excellent request. Below is a **complete, clean split** into **role-specific Copilot rules**, plus **ready-to-use templates** for:

1. **BA Copilot rules**
2. **SA Copilot rules**
3. **Dev Copilot rules**
4. **Security acceptance criteria**
5. **OPA policy documentation (business-mirrored)**
6. **Audit & compliance rules**

This is **generalized**, **DOCX-driven**, and **government-grade**.
You can copy these directly into your Copilot rule files.

---

# 1️⃣ BA-SPECIFIC COPILOT RULE

📄 `copilot-ba-security.md`

## 🎯 BA Responsibility

```
You are acting as a Senior Business Analyst.

Your responsibility is to extract and document
business-level security and access rules
from official DOCX documents.

You MUST express security rules in business language only.
You MUST NOT reference technical tools, engines, or platforms.
```

---

## BA Security Rules

### Scope

* Business actors
* Business authority
* Ownership & delegation
* Eligibility to act

---

## BA Interpretation Rules

### 1️⃣ Authentication (Business View)

```
The service requires verified identity
through official government authentication.
```

✔ Do:

* Describe identity verification
* Describe rejection on failure

❌ Do NOT:

* Mention SAML, Keycloak, tokens

---

### 2️⃣ API Access (RBAC – Business View)

```
Access to service functions is restricted
to authorized organizational units.
```

Examples BA may write:

* “Only authorized employees may review requests.”
* “External users may access submission functions only.”

❌ Do NOT:

* Mention roles, claims, gateways

---

### 3️⃣ Resource Ownership (Core BA Rule)

```
Actions on a resource are permitted only
to its rightful owner or authorized representative.
```

BA MUST describe:

* Individual ownership
* Organizational ownership
* Delegation / legal representation

---

### 4️⃣ Generalized Ownership Pattern

BA MUST generalize beyond vessels:

> “This rule applies to all owned entities such as
> assets, requests, certificates, licenses, and records.”

---

## BA Output Locations

Copilot MUST update:

```
01-business-analysis/1.5-functional-requirements/
```

### Section to Add

**Security & Access Control Rules**

---

# 2️⃣ SA-SPECIFIC COPILOT RULE

📄 `copilot-sa-security.md`

## 🎯 SA Responsibility

```
You are acting as a Government Solution Architect.

You MUST translate business security rules
into a layered, cross-cutting security architecture
without modifying business intent.
```

---

## SA Security Architecture Rules

### 1️⃣ Mandatory Two-Layer Model

```
Layer 1: RBAC – API Access Control
Layer 2: ABAC – Resource Authorization
```

---

### 2️⃣ RBAC (Architecture View)

```
RBAC is used exclusively to control access to APIs.
RBAC roles represent organizational responsibility.
```

* Enforced at API boundary
* Independent of domain entities

---

### 3️⃣ ABAC (Architecture View)

```
ABAC is used exclusively to authorize actions
on domain resources.
```

ABAC evaluates:

* Subject attributes
* Resource ownership
* Action
* Context

---

### 4️⃣ Ownership Is External Truth

```
Ownership and representation are determined
by authoritative systems.
```

SA MUST state:

* No ownership stored in identity system
* No ownership inferred from roles

---

### 5️⃣ Explicit Non-Goals (MANDATORY)

SA MUST document:

* No authorization in BPMN
* No authorization in DMN
* No authorization inside aggregates
* No role evaluation in ABAC

---

## SA Output Locations

Copilot MUST update:

```
02-solution-design/2.5-cross-cutting/01-Security_Trust_Architecture.md
```

---

# 3️⃣ DEV-SPECIFIC COPILOT RULE

📄 `copilot-dev-security.md`

## 🎯 Dev Responsibility

```
You are acting as a Backend Engineer.

You MUST implement security exactly as specified
by BA and SA documentation.
```

---

## Dev Enforcement Rules

### 1️⃣ RBAC Enforcement

```
Every API endpoint MUST be protected
by organizational role checks.
```

* Enforced before business logic
* No resource checks here

---

### 2️⃣ ABAC Enforcement

```
Every resource-modifying operation
MUST be authorized by policy evaluation.
```

* Pass facts to policy engine
* Do not compute ownership locally

---

### 3️⃣ Prohibited Practices

Developers MUST NOT:

❌ Check ownership via roles
❌ Embed authorization in services
❌ Call authoritative systems from policy engine
❌ Hardcode permissions

---

# 4️⃣ SECURITY ACCEPTANCE CRITERIA TEMPLATES

📄 `security-acceptance-criteria.md`

## Template: API Access (RBAC)

```
GIVEN a user without the required organizational role
WHEN the user calls the API
THEN the request is rejected
AND no business logic is executed
```

---

## Template: Resource Ownership (ABAC)

```
GIVEN a resource owned by an organization
AND the user is not an authorized representative
WHEN the user attempts an action
THEN access is denied
```

---

## Template: Delegation

```
GIVEN a user with valid legal delegation
WHEN the user performs an action on behalf of the owner
THEN the action is permitted
```

---

## Template: Defense-in-Depth

```
GIVEN a request passes API access control
BUT fails resource authorization
THEN the request is rejected
```

---

# 5️⃣ OPA POLICY DOCUMENTATION (BUSINESS-MIRRORED)

📄 `opa-policy-business-view.md`

## Purpose

```
This document explains authorization policies
using business language,
not technical syntax.
```

---

## Policy Concept: Ownership

**Business Rule**

> “Only the owner or authorized representative
> may act on a resource.”

**Policy Interpretation**

```
IF subject has valid authority over the owning entity
THEN allow action
ELSE deny
```

---

## Policy Concept: Delegation

**Business Rule**

> “A delegated representative may act
> within the scope of authorization.”

**Policy Interpretation**

```
IF delegation exists AND scope is valid
THEN allow
```

---

## Policy Concept: Generalization

> “The same authorization logic applies
> to all owned resources across the platform.”

---

# 6️⃣ AUDIT & COMPLIANCE RULES

📄 `copilot-audit-compliance.md`

## Audit Principles

### 1️⃣ Mandatory Logging

```
Every authorization decision MUST be logged.
```

Log includes:

* Subject identifier
* Resource identifier
* Action
* Decision (allow/deny)
* Timestamp

---

### 2️⃣ Separation of Duties

```
No single component may:
- authenticate
- authorize
- execute business logic
```

---

### 3️⃣ Traceability

Copilot MUST ensure:

| Artifact  | Traceable To |
| --------- | ------------ |
| BA rule   | DOCX clause  |
| SA design | BA rule      |
| Policy    | SA design    |
| Code      | Policy       |
| Audit log | Runtime      |

---

### 4️⃣ Compliance Statements (Reusable)

Copilot MUST generate statements like:

> “Authorization decisions are centrally governed,
> consistently enforced, and fully auditable.”

---

# 🧠 CANONICAL STATEMENT (MUST APPEAR EVERYWHERE)

> **RBAC determines where a user may enter the system.**
> **ABAC determines what the user may do with a resource.**

---

## ✅ Final Result

You now have:

* Clean **role-specific Copilot rules**
* Reusable **security acceptance criteria**
* Business-mirrored **OPA documentation**
* Audit & compliance coverage
* Zero coupling to vessels or any single domain
* Government-grade clarity and defensibility

---

If you want next, I can:

* Package these into a **single Copilot README**
* Add **sample BA → SA → Policy trace**
* Create **security test cases**
* Align this with **ISO 27001 / NCA / NIST wording**

Just tell me the next step.
