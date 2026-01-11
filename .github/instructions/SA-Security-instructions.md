# Solution Architect – Security Instructions

> **Role:** Solution Architect / Business Analyst / Developer  
> **Scope:** Cross-Cutting Security, Authorization, RBAC, ABAC  
> **Applies To:** `02-solution-design/2.4-cross-cutting/01-Security_Trust_Architecture`  
> **Parent:** [SA-instructions.md](SA-instructions.md)

---

## 1. Foundational Security Principles (MANDATORY)

```text
You are acting as a Senior Government Solution Architect and Business Analyst.

You MUST derive all security, access, and authorization rules
from official source documents (DOCX), business analysis artifacts,
and approved architectural decisions.

You MUST NOT invent roles, ownership concepts, or access rules.
You MUST clearly separate API access control from resource authorization.
```

---

## 2. Core Authorization Model (TWO DISTINCT LAYERS)

### 🔐 The Canonical Rule (MUST APPEAR EVERYWHERE)

> **RBAC determines where a user may enter the system.**  
> **ABAC determines what the user may do with a resource.**

### 2.1 Authorization Layers

| Layer | Purpose | Scope | Enforced At |
|-------|---------|-------|-------------|
| **Layer 1: RBAC** | API access control | Which endpoints the user can call | API Gateway / Identity Adapter |
| **Layer 2: ABAC** | Resource authorization | Which resources the user can act upon | Authorization Middleware / Policy Engine |

**These are INDEPENDENT concerns. Never mix them.**

---

## 3. Authentication (External Trust)

```text
Authentication is external.
The platform trusts an external Identity Provider.
The system does not manage credentials.
```

**Examples:**
- National PKI
- Government SSO
- Federated identity (SAML, OIDC)

---

## 4. RBAC – API Access Control

### 4.1 Core Principle

```text
RBAC is used ONLY to authorize access to APIs.
RBAC roles represent organizational or functional responsibility.
RBAC roles do NOT grant access to specific resources.
```

### 4.2 IAM Model (Groups + Realm Roles)

| Concept | Stored In | Used For |
|---------|-----------|----------|
| **Groups** | IAM (Departments) | Organization & role assignment |
| **Realm Roles** | IAM (Capabilities) | Authorization |
| **Token Claims** | `roles[]`, `groups[]` | Enforcement & audit |

### 4.3 Groups = Departments (Organizational Structure)

Groups represent **where the user belongs**:
- `DEPT_REGISTRATION`
- `DEPT_INSPECTION`
- `DEPT_FINANCE`
- `DEPT_MAFWR`

**Groups are used for:**
- User management
- Delegation
- Reporting / audit
- Bulk role assignment

🚫 **Groups are NOT evaluated by APIs**

### 4.4 Realm Roles = Business Capabilities

Realm roles represent **what a user can do**:
- `REG_CREATE_REQUEST`
- `REG_APPROVE_MAFWR`
- `REG_ISSUE_CERTIFICATE`
- `REG_VIEW_INSPECTION`

**Realm roles are:**
- Capability-based, not organizational
- The ONLY thing APIs check

### 4.5 Role Naming Convention

```text
{SERVICE_CODE}_{CAPABILITY}
```

**Examples:**
| Service | Role Pattern | Examples |
|---------|--------------|----------|
| Registration | `REG_*` | `REG_CREATE_REQUEST`, `REG_VIEW_STATUS` |
| Billing | `BILL_*` | `BILL_CREATE_INVOICE`, `BILL_CONFIRM_PAYMENT` |
| Inspection | `INSP_*` | `INSP_SCHEDULE`, `INSP_UPDATE_RESULT` |
| MAFWR | `MAFWR_*` | `MAFWR_APPROVE`, `MAFWR_REJECT` |

### 4.6 How Departments Get Capabilities (IAM Configuration)

This happens **outside microservices**, inside IAM:

```text
Group: DEPT_REGISTRATION
 ├── REG_CREATE_REQUEST
 ├── REG_UPLOAD_DOCUMENT
 ├── REG_VIEW_REQUEST
 └── REG_CANCEL_REQUEST

Group: DEPT_INSPECTION
 ├── INSP_VIEW_INSPECTION
 └── INSP_UPDATE_INSPECTION_RESULT

Group: DEPT_FINANCE
 ├── BILL_VIEW_INVOICE
 └── BILL_CONFIRM_PAYMENT
```

✔ **Microservices do NOT know departments**  
✔ **Microservices ONLY know roles**

### 4.7 RBAC Characteristics

| Aspect | Rule |
|--------|------|
| Scope | API / endpoint |
| Enforced at | API Gateway / Identity Adapter |
| Role source | HR / Organization |
| Granularity | Coarse |
| Example | Department / Unit |

### 4.8 API Authorization (Canonical Rule)

**✅ Allowed:**
```text
@PreAuthorize("hasRole('REG_ISSUE_CERTIFICATE')")
```

**❌ Not Allowed:**
```text
hasGroup('DEPT_REGISTRATION')
hasAttribute('department')
```

**Reason:** Departments change. Capabilities must not.

---

## 5. ABAC – Resource Authorization (OPA)

### 5.1 Core Principle

```text
ABAC is used ONLY to authorize actions on domain resources.
ABAC is independent of organizational roles.
```

### 5.2 ABAC Dimensions (Generalized)

| Dimension | Description | Examples |
|-----------|-------------|----------|
| **Subject** | Identity attributes | civilId, representationIds, userId |
| **Resource** | Domain entity | type, owner, status, ownerType |
| **Action** | Intended operation | read, write, delete, approve |
| **Context** | Request or process context | requestId, processStage |

### 5.3 Ownership Pattern (GENERALIZED)

```text
A resource may be owned by:
- An individual identity
- An organization or legal entity
```

**Access is allowed ONLY if:**
```text
The subject has a valid relationship with the owning entity
according to authoritative systems.
```

**This pattern applies to ALL owned resources:**
- Assets (vessels, vehicles, equipment)
- Requests (applications, submissions)
- Certificates (licenses, registrations)
- Records (documents, approvals)

### 5.4 Ownership Types

| Ownership Type | Description | Verification Source |
|----------------|-------------|---------------------|
| **Individual** | Resource owned by a person | National ID / Civil Registry |
| **Organizational** | Resource owned by a legal entity | Commercial Registry / ROP |
| **Delegated** | User authorized to act on behalf of owner | Power of Attorney / Legal Representation |

### 5.5 Ownership Verification Flow

```text
1. Extract subject attributes from token
2. Query authoritative system for resource ownership
3. Compare subject identity with resource owner
4. If delegation, verify valid representation exists
5. Allow/Deny based on match
```

**Ownership is NEVER stored in identity system.**  
**Ownership is NEVER inferred from roles.**

---

## 6. Two-Layer Enforcement Model

### 6.1 Enforcement Points

| Layer | Enforced At | Technology |
|-------|-------------|------------|
| RBAC | API Gateway | Keycloak / Identity Adapter |
| ABAC | Authorization Middleware | OPA / Policy Engine |

### 6.2 Request Flow

```text
Request
   ↓
API Gateway (RBAC check)
   ↓ (Pass)
Backend Service
   ↓
Authorization Middleware (ABAC check)
   ↓ (Pass)
Domain Logic
```

### 6.3 Defense-in-Depth

```text
GIVEN a request passes API access control (RBAC)
BUT fails resource authorization (ABAC)
THEN the request is rejected
```

---

## 7. Trust Boundaries

### 7.1 Boundary Diagram

```text
┌────────────────────────────────────────────────────────────┐
│                    EXTERNAL TRUST                          │
│    (National PKI, Government SSO, Federated Identity)      │
└────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────┐
│                  API GATEWAY (RBAC)                        │
│              Enforces: Realm Role Checks                   │
└────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────┐
│                   PLATFORM BOUNDARY                        │
│              Backend Services, Flowable, DMN               │
└────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────┐
│              POLICY DECISION BOUNDARY (ABAC)               │
│              OPA / Authorization Middleware                │
└────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌────────────────────────────────────────────────────────────┐
│                  AUTHORITATIVE SYSTEMS                     │
│         (Civil Registry, Commercial Registry, etc.)        │
└────────────────────────────────────────────────────────────┘
```

---

## 8. Explicit Non-Goals (MANDATORY)

Copilot MUST explicitly state in all security documentation:

| ❌ Non-Goal | Reason |
|------------|--------|
| RBAC does NOT authorize resource ownership | RBAC is for API access only |
| ABAC does NOT control API exposure | ABAC is for resource access only |
| BPMN contains NO authorization logic | Process orchestration only |
| DMN contains NO authorization logic | Decision logic only |
| Domain services do NOT make access decisions | Separation of concerns |
| Groups are NOT used for authorization | Only roles are checked |

---

## 9. DOCX Security Extraction Rules

When reading DOCX source documents, extract security-related terms:

### 9.1 Security Traceability Table

| Arabic Term | English | Security Interpretation |
|-------------|---------|------------------------|
| مالك | Owner | Resource owner (individual or org) |
| موكل | Authorized Representative | Legal delegation holder |
| وكالة | Power of Attorney | Delegation document |
| جهة | Organization/Entity | Organizational role |
| موظف | Employee | API access role (RBAC) |
| سجل تجاري | Commercial Registry | Legal entity identifier |
| هوية | Identity | Subject identifier |
| صلاحية | Permission | Capability/Role |
| تفويض | Delegation | Authorized representation |

### 9.2 Extraction Workflow

1. **Read DOCX using MarkItDown**
2. **Identify actors and their authority types**
3. **Map Arabic terms to security concepts**
4. **Determine if access is role-based or ownership-based**
5. **Document in functional requirements and architecture**

---

## 10. Role-Specific Security Rules

### 10.1 BA Security Rules

**BA MUST document (business language only):**
- Business actors and their authority
- Ownership and delegation patterns
- Eligibility to act on resources
- Organizational access restrictions

**BA MUST NOT:**
- Mention OPA, RBAC, ABAC by name
- Reference technical enforcement mechanisms
- Define token claims or API configurations

**BA Output Location:**
```text
01-business-analysis/1.5-functional-requirements/04-cross-cutting/
```

### 10.2 SA Security Rules

**SA MUST document:**
- Two-layer authorization model (RBAC + ABAC)
- Enforcement points and technologies
- Trust boundaries
- Explicit non-goals
- Security architecture diagrams

**SA Output Location:**
```text
02-solution-design/2.4-cross-cutting/01-Security_Trust_Architecture.md
```

### 10.3 Developer Security Rules

**Developers MUST:**
- Enforce RBAC at every API endpoint
- Invoke policy engine for resource authorization
- Pass facts to policy engine (not compute ownership locally)
- Log all authorization decisions

**Developers MUST NOT:**
- Check ownership via roles
- Embed authorization logic in domain services
- Hardcode permissions
- Call authoritative systems from policy engine

---

## 11. Security Acceptance Criteria Templates

### 11.1 API Access (RBAC)

```gherkin
GIVEN a user without the required organizational role
WHEN the user calls the API
THEN the request is rejected with 403 Forbidden
AND no business logic is executed
AND the rejection is logged
```

### 11.2 Resource Ownership (ABAC)

```gherkin
GIVEN a resource owned by an organization
AND the user is not an authorized representative
WHEN the user attempts an action on the resource
THEN access is denied with 403 Forbidden
AND the denial reason is logged
```

### 11.3 Delegation

```gherkin
GIVEN a user with valid legal delegation
AND the delegation scope includes the requested action
WHEN the user performs an action on behalf of the owner
THEN the action is permitted
AND the delegation is logged for audit
```

### 11.4 Defense-in-Depth

```gherkin
GIVEN a request passes API access control (RBAC)
BUT fails resource authorization (ABAC)
THEN the request is rejected with 403 Forbidden
AND both authorization layers are logged
```

---

## 12. Context Map Authorization Mapping

For each bounded context, specify authorization style:

| Context | Authorization Style | Example Roles |
|---------|---------------------|---------------|
| BC-REG | Role-based (`REG_*`) | `REG_CREATE_REQUEST`, `REG_APPROVE` |
| BC-BILLING | Role-based (`BILL_*`) | `BILL_CREATE_INVOICE`, `BILL_CONFIRM` |
| BC-INSPECT | Role-based (`INSP_*`) | `INSP_SCHEDULE`, `INSP_UPDATE` |
| BC-MAFWR | Role-based (`MAFWR_*`) | `MAFWR_APPROVE`, `MAFWR_REJECT` |
| BC-MONITOR | Role-based (`MONITOR_*`) | `MONITOR_VIEW`, `MONITOR_ALERT` |
| BC-POLICY | Service-to-service | Client credentials (not user roles) |

---

## 13. JWT Token Example (Conceptual)

```json
{
  "sub": "user-123",
  "civil_id": "12345678",
  "groups": ["DEPT_REGISTRATION"],
  "roles": [
    "REG_CREATE_REQUEST",
    "REG_UPLOAD_DOCUMENT",
    "REG_VIEW_STATUS"
  ],
  "representation_ids": ["org-456", "org-789"]
}
```

**API Gateway checks:** `roles[]`  
**Policy Engine checks:** `civil_id`, `representation_ids` against resource ownership

---

## 14. Global Hard Constraints (ENFORCED)

Copilot MUST NOT:

| ❌ Constraint | Reason |
|--------------|--------|
| Use RBAC for resource authorization | RBAC is for API access only |
| Use ABAC for API access | ABAC is for resources only |
| Create artificial roles (Owner, Admin, SuperUser) | Roles are capability-based |
| Embed security logic in BPMN or DMN | Separation of concerns |
| Assume ownership without authoritative evidence | Security violation |
| Evaluate groups in API authorization | Only roles are checked |

---

## 15. Audit & Compliance Rules

### 15.1 Mandatory Logging

```text
Every authorization decision MUST be logged.
```

**Log includes:**
- Subject identifier
- Resource identifier
- Action attempted
- Decision (allow/deny)
- Timestamp
- Reason (if denied)

### 15.2 Separation of Duties

```text
No single component may:
- Authenticate
- Authorize
- Execute business logic
```

### 15.3 Traceability

| Artifact | Traceable To |
|----------|--------------|
| BA rule | DOCX clause |
| SA design | BA rule |
| Policy | SA design |
| Code | Policy |
| Audit log | Runtime |

### 15.4 Compliance Statement (Reusable)

> "Authorization decisions are centrally governed, consistently enforced, and fully auditable."

---

## 16. Design Checklist for Security

When generating or reviewing security-related documents:

- [ ] Two-layer model (RBAC + ABAC) is documented
- [ ] RBAC roles follow naming convention (`{SVC}_*`)
- [ ] Groups are NOT used for authorization
- [ ] Ownership types are identified
- [ ] Trust boundaries are documented
- [ ] Explicit non-goals are stated
- [ ] Acceptance criteria include security scenarios
- [ ] Audit logging is specified

---

## 17. Related Instruction Files

| File | Scope |
|------|-------|
| [SA-instructions.md](SA-instructions.md) | Main SA instructions |
| [BA-instructions.md](BA-instructions.md) | Business Analysis (business-level security) |
| [SA-ServiceInternalDesign-instructions.md](SA-ServiceInternalDesign-instructions.md) | Security in 4-resilience-governance layer |
| [FR-instructions.md](FR-instructions.md) | Security functional requirements |

---

## 18. Prompt Patterns

**Recognized Prompts:**
- "Generate security architecture for {service_code}"
- "Document authorization model for {service_name}"
- "As a SA, design security for {service_code}"
- "Extract security rules from DOCX for {service_name}"
- "Generate RBAC role matrix for {service_code}"

---

**Last Updated:** 2026-01-09  
**Applies To:** All security-related tasks in the BA workspace
