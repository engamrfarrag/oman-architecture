# Solution Architect – UI Interaction Instructions

> **Role:** Solution Architect  
> **Scope:** Angular UI, Backend-for-Frontend (BFF), Flowable Isolation  
> **Applies To:** `02-solution-design/2.4-ui-interaction-design/` folder  
> **Parent:** [SA-instructions.md](SA-instructions.md)

---

## 1. Core Technology Assumptions

| Area | Decision |
|------|----------|
| **Frontend** | Angular (SPA) |
| **UI Integration** | Backend-for-Frontend (BFF) |
| **Process Engine** | Flowable (Backend Only) |

---

## 2. UI & Flowable Interaction Rule (CRITICAL)

> **Angular UI MUST NEVER interact with Flowable directly.**

Flowable is an **internal backend orchestration engine**, accessed only via backend services.

### Rules:
- Angular UI communicates ONLY with backend APIs
- Flowable is accessed ONLY by backend application services
- No BPMN task IDs, process instance IDs, or DMN details are exposed to the frontend
- The frontend is process-aware ONLY via backend-provided status abstractions

**Any design that exposes Flowable directly to the UI is INVALID.**

---

## 3. Correct Interaction Model

```text
Angular UI
   ↓ (REST / GraphQL)
Backend API (BFF / Application Layer)
   ↓
Flowable BPMN / DMN Engine
   ↓
Domain Services / Integrations
```

### Flowable Visibility:
- ❌ Has NO public API exposure
- ❌ Has NO UI coupling
- ❌ Is INVISIBLE to the frontend

---

## 4. Backend-for-Frontend (BFF) Pattern

### 4.1 BFF Responsibilities

The BFF is responsible for:
- Translating UI actions into Flowable interactions
- Aggregating task state into UI-friendly statuses
- Hiding BPMN/DMN complexity from the frontend
- Enforcing authorization and validation

### 4.2 Angular UI MUST NOT:
- Poll Flowable directly
- Correlate Flowable messages
- Understand BPMN or DMN concepts
- Reference BPMN task IDs

---

## 5. BPMN to UI Abstraction Rule

BPMN tasks and events MUST be abstracted into **business-level states** for the UI.

### Example Mapping:

| BPMN Task/State | UI State (Display) |
|-----------------|-------------------|
| SP01_INITIATE_REQUEST | "Application Started" |
| SP03_AWAIT_VESSEL_DATA | "Enter Vessel Information" |
| SP05_WAIT_MAFWR_APPROVAL | "Waiting for Fishing Approval" |
| SP08_AWAIT_PAYMENT | "Pending Payment" |
| SP09_CERTIFICATE_ISSUED | "Certificate Issued" |

> **The UI MUST NOT be aware of BPMN internals.**

---

## 6. UI Interaction Design Section (2.4)

When generating `02-solution-design/2.4-ui-interaction-design/`:

### 6.1 Include:

| Content | Description |
|---------|-------------|
| Screen Flow | User journey through screens |
| Business States | Status labels for UI display |
| API-Driven Status | How backend provides state |
| Async Waiting States | Approval, inspection, payment waits |
| Error States | Error and timeout handling |
| Empty States | No data scenarios |
| Process-to-Screen Mapping | BPMN abstraction to screens |
| Backoffice UI | Admin/authority screens |

### 6.2 Exclude:

| Content | Reason |
|---------|--------|
| BPMN User Tasks | Internal to Flowable |
| Flowable Task IDs | Implementation detail |
| Direct Workflow Control | Violates BFF pattern |
| Visual Design | Not architecture concern |
| Colors, Typography | Design system responsibility |

---

## 7. Screen Flow Standards

### 7.1 Applicant Portal Flow

```text
Login
  ↓
Dashboard (My Applications)
  ↓
New Application
  ↓
Step 1: Applicant Profile
  ↓
Step 2: Vessel Information
  ↓
Step 3: Document Upload
  ↓
Step 4: Review & Submit
  ↓
Waiting States (Payment, Approvals)
  ↓
Certificate View / Download
```

### 7.2 Authority Backoffice Flow

```text
Login
  ↓
Dashboard (Pending Tasks)
  ↓
Application Review
  ↓
Approve / Reject / Request Info
  ↓
Certificate Issuance Confirmation
```

---

## 8. Status Abstraction Layer

### 8.1 Status Categories

| Category | UI Treatment |
|----------|--------------|
| **Draft** | Editable, incomplete |
| **Submitted** | Read-only, processing |
| **Awaiting Action** | Show required action |
| **Pending External** | Show waiting message |
| **Completed** | Show success state |
| **Rejected** | Show rejection reason |
| **Expired** | Show expiry message |

### 8.2 API Response Format

```json
{
  "applicationId": "REG001-2026-00123",
  "status": "AWAITING_PAYMENT",
  "statusLabel": "Pending Payment",
  "statusDescription": "Please complete payment to proceed.",
  "allowedActions": ["PAY", "CANCEL"],
  "nextSteps": ["Complete payment within 48 hours"],
  "lastUpdated": "2026-01-07T10:30:00Z"
}
```

> **No BPMN/Flowable references in API responses.**

---

## 9. Async State Handling

### 9.1 Waiting States

| Waiting For | UI Message | Refresh Strategy |
|-------------|------------|------------------|
| Payment Confirmation | "Waiting for payment confirmation" | Poll every 30s |
| Authority Approval | "Under review by authorities" | Poll every 5min |
| External System | "Awaiting external verification" | Event-driven |
| Inspection | "Inspection scheduled" | Manual refresh |

### 9.2 Timeout Handling

| Scenario | UI Behavior |
|----------|-------------|
| Payment Timeout | Show warning, allow retry |
| Approval Timeout | Show escalation message |
| Session Timeout | Redirect to login |

---

## 10. Error State Design

### 10.1 Error Categories

| Category | UI Treatment |
|----------|--------------|
| Validation Error | Inline field errors |
| Business Error | Modal with explanation |
| System Error | Generic error page |
| Network Error | Retry prompt |
| Authorization Error | Redirect to login |

### 10.2 Error Response Format

```json
{
  "error": true,
  "errorCode": "VESSEL_AGE_EXCEEDED",
  "errorMessage": "Vessel age exceeds maximum allowed limit.",
  "errorDetails": {
    "vesselAge": 32,
    "maxAge": 30
  },
  "recoverable": false
}
```

---

## 11. Document Structure

```text
02-solution-design/2.4-ui-interaction-design/
│
├── 01-Screen_Flows/
│   ├── Applicant-Portal-Flow.md
│   └── Authority-Backoffice-Flow.md
│
├── 02-Status_Abstraction/
│   ├── Status-Categories.md
│   └── API-Response-Formats.md
│
├── 03-Process_to_Screen_Mapping/
│   └── BPMN-to-UI-Mapping.md
│
├── 04-Error_States/
│   ├── Error-Categories.md
│   └── Error-Response-Formats.md
│
├── 05-Async_States/
│   ├── Waiting-States.md
│   └── Timeout-Handling.md
│
└── _README.md
```

---

## 12. Explicitly Forbidden

Copilot MUST STOP if asked to:

- ❌ Generate Angular code calling Flowable APIs
- ❌ Expose BPMN or DMN to frontend
- ❌ Use Flowable UI forms
- ❌ Use Flowable Task APIs from UI
- ❌ Treat Flowable as a "frontend workflow"
- ❌ Include BPMN task IDs in UI code
- ❌ Poll Flowable directly from Angular

---

## 13. One-Line Rule

> **Angular talks to APIs, APIs talk to Flowable, Flowable talks to services — never the other way around.**

---

## 14. Quality Checklist

Before completing UI interaction design:

- [ ] No Flowable references in UI specifications
- [ ] All BPMN states abstracted to business states
- [ ] BFF pattern clearly defined
- [ ] Status API formats documented
- [ ] Error handling covers all categories
- [ ] Async waiting states documented
- [ ] Screen flows match BPMN process

---

## 15. Related Instruction Files

- [SA-instructions.md](SA-instructions.md) – Main SA instructions
- [SA-Flowable-instructions.md](SA-Flowable-instructions.md) – Flowable integration
- [Process-instructions.md](Process-instructions.md) – BPMN process design

---

**Last Updated:** 2026-01-07  
**Applies To:** All UI Interaction Design tasks in the BA workspace
