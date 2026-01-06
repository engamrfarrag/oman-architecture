# REG001 – Business Process Overview

## Service Identification
- **Service Code (Project):** REG001
- **Service Name (AR):** إصدار شهادة تسجيل سفينة أو وحدة بحرية مؤقتة
- **Service Name (EN):** Issuing Temporary Registration Certificate for a Ship or Marine Unit
- **System Service Code:** MTCIT-REG-001
- **Release / Sprint:** 1 / 1

---

## 1. Purpose
Provide an end-to-end, business-friendly view of REG001, decomposed into processes and sub-processes with clear swimlanes and gateways that can be used directly for BPMN modeling and automation.

---

## 2. Business Outcome
### Successful Outcome
- **Temporary Registration Certificate** issued (PDF + QR)
- **Payment receipt** available
- Applicant receives status notifications

### Failure Outcomes (Typical)
- Request rejected due to: identity failure, invalid CR/activity, prohibited vessel, missing documents, rejected approvals, failed inspection, customs rejection, or payment failure.

---

## 3. Scope Boundaries
### Start Trigger
- Applicant (Owner/Agent) initiates request on MTCIT platform.

### End Conditions
- **Success:** certificate issued and applicant notified.
- **Stop/Reject:** request terminated with a clear rejection reason.
- **Post-issuance monitoring:** reminders and penalties run until the certificate is converted to permanent registration (handled as a long-running monitoring process).

---

## 4. Preconditions & Eligibility (Scope-Level)
- Applicant is vessel owner or authorized agent.
- Applicant is Omani citizen or resident (per source service document).
- Vessel/unit is not on the prohibited/banned list (configuration-driven).

---

## 5. Core Integrations & Dependencies
- **ROP PKI:** identity authentication (accept/reject)
- **Invest Easy:** CR validation and activity validation
- **MAFWR:** approval for fishing vessels (conditional)
- **Inspection / Classification:** inspection outcome (conditional)
- **Bayan Customs:** import document & customs clearance (conditional)
- **Payment Gateway:** invoice + payment confirmation

---

## 6. Swimlanes (BPMN-Ready)
Use these swimlanes consistently across all subprocesses:

1. **Applicant (Owner/Agent)**
   - Data entry, document upload, selection/confirmation steps.

2. **MTCIT System (REG001 Orchestrator)**
   - Validations, decision tables (DMN), orchestration, calculation, notifications.

3. **Authority / External Entities**
   - **ROP PKI**: identity
   - **Invest Easy**: CR/activity
   - **MAFWR**: fishing approval
   - **Inspection Entity**: inspection execution + result
   - **Bayan Customs**: customs clearance
   - **Payment Gateway**: payment processing

4. **MTCIT Registration Dept (Backoffice) – Optional**
   - Manual follow-up/escalations when timeouts occur (policy-driven).

---

## 7. Process Hierarchy (Process → Sub-process)

### P0. REG001 – Issue Temporary Registration Certificate (End-to-End)
**Sub-processes:**
- **SP02** Applicant Profiling + CR/Activity Validation (Company Path)
- **SP03** Vessel/Unit Data Capture & Policy Validation
- **SP04** Dynamic Documents Collection
- **SP05** MAFWR Approval (Fishing Vessels) – Conditional
- **SP06** Vessel Inspection – Conditional
- **SP07** Bayan Customs Clearance – Conditional
- **SP08** Fees, Name Reservation, and Payment
- **SP09** Certificate Issuance & Notifications
- **SP10** Post-Issuance Reminders & Overdue Penalties (Monitoring)

### Cross-Cutting (Non-Business Outcome)
These are required platform concerns that **gate access** to the business process, but are not considered part of the “business process result” itself.

- **CC01** Security Authentication (ROP PKI) – required to start/continue the request

---

## 8. Gateway Catalog (Key Decisions)
> Gateways are listed in business terms for BPMN usage. Details per gateway appear in the corresponding subprocess file.
>
> Note: PKI authentication is a **security cross-cutting concern** (CC01) and is intentionally excluded from the business process gateway catalog.

- **G02 – Beneficiary type? (Individual vs Company)**
  - Individual → skip company validations
  - Company → CR + activity validations

- **G03 – Company CR valid?**
  - Valid → continue
  - Invalid → reject + notify

- **G04 – Company activity active/eligible?**
  - Active/eligible → continue
  - Inactive/not eligible → reject + notify

- **G05 – Vessel age policy passed?** (config/DMN)
  - Passed → continue
  - Failed → stop with warning / allow re-entry where policy permits

- **G06 – Penalty applies? (>30 days after construction end at submission)**
  - Yes → add penalty line item
  - No → continue

- **G07 – Vessel prohibited/banned?**
  - Not prohibited → continue
  - Prohibited → reject + notify

- **G08 – Documents complete?**
  - Complete → continue
  - Missing → request missing docs (loop)

- **G09 – Fishing vessel?**
  - Yes → MAFWR approval required
  - No → skip

- **G10 – MAFWR approved?**
  - Approved → continue
  - Rejected → reject + notify

- **G11 – Inspection required?** (rules: construction phase, length, tug, etc.)
  - Required → run inspection subprocess
  - Not required → continue

- **G12 – Inspection passed?**
  - Passed → continue
  - Failed/cancelled/timeout → reject/escalate + notify

- **G13 – Customs/Bayan required?**
  - Required → request clearance
  - Not required → continue

- **G14 – Customs cleared?**
  - Cleared → continue
  - Rejected → reject + notify

- **G15 – Payment successful?**
  - Successful → issue certificate
  - Failed/cancelled/expired → stop/retry per policy; release name reservation

- **G16 – Permanent registration completed?** (monitoring)
  - Completed → stop reminders
  - Not completed → continue reminders; apply penalties when overdue

---

## 9. Notes & Assumptions
- Documents list and thresholds are **configuration-driven**; process assumes the system can evaluate DMN/config at runtime.
- Integrations may be synchronous or event-driven; BPMN should model external responses using **message events** plus **timeouts**.
- Name reservation is treated as a system capability unless a dedicated external service owner is confirmed.

### 9.1 BPMN Event Type Tags (Annotation-Level)
In the subprocess specifications under `03-BPMN_Process_Design/`, steps may be tagged with an **Event Type (Optional)** to explicitly mark:
- **Message** events (external responses)
- **Timer boundary** events (timeouts/expiry while waiting)
- **Error** events (hard failures/terminations)
