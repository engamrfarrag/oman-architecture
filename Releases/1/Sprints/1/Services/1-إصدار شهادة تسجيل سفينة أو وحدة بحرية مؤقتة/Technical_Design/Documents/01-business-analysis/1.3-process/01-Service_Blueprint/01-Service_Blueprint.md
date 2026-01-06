# REG001 – Service Blueprint

## Service Identification
- **Service Code (Project):** REG001
- **Service Name (EN):** Issuing Temporary Registration Certificate for a Ship or Marine Unit
- **Service Name (AR):** إصدار شهادة تسجيل سفينة أو وحدة بحرية مؤقتة
- **System Service Code:** MTCIT-REG-001
- **Release / Sprint:** 1 / 1

---

## 1. Purpose / الغرض
This Service Blueprint provides a **customer-journey + operational blueprint** for REG001, aligning:
- Customer actions (Applicant)
- Frontstage interactions (MTCIT portal + notifications)
- Backstage orchestration (rules, workflow)
- Supporting processes and external integrations

تهدف هذه الوثيقة إلى توثيق **رحلة المستفيد + ما يحدث داخل المنظومة والتكاملات** لخدمة REG001 بطريقة جاهزة للنمذجة عبر BPMN.

---

## 2. Scope & Boundaries / النطاق والحدود
### Start Trigger
- Applicant starts the REG001 request on the MTCIT platform.

### End Conditions
- **Success:** Temporary certificate issued (PDF + QR) after successful validations and payment.
- **Stop/Reject:** Request terminated with explicit reason (authentication/validation/approval/inspection/customs/payment).
- **Post-issuance:** Reminders and overdue penalties run until permanent registration is completed.

---

## 3. Actors, Channels, and Touchpoints / الأطراف والقنوات
### Primary Actor
- **Applicant (Owner/Agent) / المستفيد (مالك/وكيل)**

### Internal
- **MTCIT System (REG001 Orchestrator)**
- **(Optional) MTCIT Registration Dept (Backoffice)**: escalation/exception handling when timeouts occur (policy-driven).

### External Integrations
- **ROP PKI**: identity authentication
- **Invest Easy**: CR + activity validation (company path)
- **MAFWR**: fishing vessel approval
- **Inspection / Classification entities**: inspection outcome
- **Bayan Customs**: customs/import docs + clearance
- **Payment Gateway (ePayment)**: invoice + payment status + receipt

### Customer Touchpoints
- **MTCIT Portal UI**: guidance, data entry, document upload, confirmations
- **Notifications**: SMS/Email/In-app (rejection/approval/invoice/certificate/reminders/penalties)

---

## 4. Blueprint Stages (Journey View)
The journey is organized into stages aligned to the existing subprocess decomposition:
- **CC01** Security Authentication (ROP PKI)
- **SP02** Applicant Profiling + CR/Activity Validation
- **SP03** Vessel Data Capture & Policy Validation
- **SP04** Dynamic Documents Collection
- **SP05** MAFWR Approval (Fishing) – conditional
- **SP06** Inspection – conditional
- **SP07** Bayan Customs – conditional
- **SP08** Fees, Name Reservation, Payment
- **SP09** Certificate Issuance & Notifications
- **SP10** Post-Issuance Reminders & Penalties

---

## 5. Service Blueprint (End-to-End)
> Notation:
> - **Message (send/receive)**: integration request/response OR applicant notification
> - **Timer boundary**: timeout/expiry while waiting (policy/config-driven)
> - **Error**: hard failure path that terminates the request

| Stage / Subprocess | Customer Actions (Applicant) | Frontstage (Portal + Notifications) | Backstage (Orchestrator + Rules) | Support / External Systems | Evidence (User-visible) | Event Types (Annotation-Level) | Key Rules / Config Anchors |
|---|---|---|---|---|---|---|---|
| 0. Entry & guidance | Login, select REG001, read instructions, start request | Show service instructions, start form | Initialize request draft | — | Instructions page; request created | — | Eligibility: citizen/resident; not banned (policy) |
| 1. Identity authentication (**CC01**) | Proceed to authenticate | Redirect/trigger PKI flow; show success/failure | Send PKI request; gate access | ROP PKI | Auth success screen OR rejection message | Message (send/receive) (+Timer boundary), Error | PKI mandatory gate |
| 2. Applicant type & company validation (**SP02**) | Choose Individual/Company; enter CR if company | Form fields; show validation outcomes | Send CR + activity validation requests; decide path | Invest Easy | Validation passed OR rejection notification | Message (send/receive) (+Timer boundary), Error | Company path hard gate |
| 3. Vessel data & policy validation (**SP03**) | Select unit type; enter year built; enter acquisition/construction/sale data | Data capture forms; warnings for age policy | Validate age policy; penalty trigger check; banned list check | (Rules/DMN + configuration) | Warning, or rejection notification | Message (send) (if reject), Error | Age threshold (config); penalty timing (example: >30 days); banned list (config) |
| 4. Dynamic documents intake (**SP04**) | Upload required documents | Show dynamic required list; missing-doc prompts | Determine required docs (DMN/config); validate completeness | (Config/DMN) | Required docs checklist; missing docs notification | Message (send), Timer boundary (optional) | Document rules are configuration-driven |
| 5. Fishing approval (conditional) (**SP05**) | Wait for decision (if fishing) | Status updates; approval/rejection notification | Send approval request; wait for decision; route next | MAFWR | Approval notification OR rejection reason | Message (send/receive) (+Timer boundary), Error | Fishing vessel gate |
| 6. Inspection (conditional) (**SP06**) | Attend/submit inspection needs (if required) | Show inspection status; notify result | Trigger inspection workflow; wait for outcome; route next | Inspection/Classification | Inspection passed OR failed/cancelled/timeout notification | Message (send/receive) (+Timer boundary), Error | Trigger rules: construction phase / >24m / tug boat / etc. (config/DMN) |
| 7. Customs (conditional) (**SP07**) | Wait for customs clearance (if required) | Show customs status; notify outcome | Send Bayan request; receive import doc + clearance; route next | Bayan Customs | Clearance confirmed OR rejection reason | Message (send/receive) (+Timer boundary), Error | Imported path / customs required (routing rules) |
| 8. Fees, name reservation & payment (**SP08**) | Choose name; confirm invoice; pay | Fee summary; invoice notification; payment status | Calculate fees/penalties; reserve name; generate invoice; wait payment callback; decide | Payment Gateway (+ optional name reservation capability) | Invoice; receipt; failure/expiry message | Message (send/receive), Timer boundary, Error | Name hold window (minutes, config); payment expiry; fee/penalty DMN |
| 9. Certificate issuance (**SP09**) | Download certificate | Certificate issued notification; download access | Generate certificate (PDF+QR); persist metadata; notify | — | Certificate (PDF+QR); issuance notification | Message (send), Error (if eligibility fails) | Certificate validity months (config) |
| 10. Post-issuance reminders & penalties (**SP10**) | Receive reminders; complete permanent registration | Periodic reminders; penalty notifications | Scheduler/timer-driven monitoring; check completion; apply penalties | (Downstream permanent registration status) | Reminder every configured interval; penalties when overdue | Message (send) + Timer boundary | Reminder cadence (example: 7 days); overdue thresholds & penalties (config/DMN) |

---

## 6. Failure Points & Controls (Blueprint Summary)
### Typical Hard Stops (Error)
- PKI authentication rejected
- Company CR invalid / activity not eligible
- Vessel prohibited/banned
- MAFWR rejected (fishing)
- Inspection failed/cancelled/timeout (policy)
- Bayan customs rejected
- Payment failed/cancelled/expired

### Timeout / Expiry (Timer boundary) – Policy Driven
- Waiting for external responses (PKI/Invest Easy/MAFWR/Inspection/Bayan/Payment)
- Name reservation expiry window
- Post-issuance reminder cadence + overdue thresholds

---

## 7. Traceability (Source → Blueprint)
This blueprint is derived from:
- Converted service source document (`Documents/1-إصدار شهادة تسجيل سفينة أو وحدة بحرية مؤقتة.docx`)
- REG001 scope docs
- REG001 subprocess BPMN-ready specs (SP02–SP10, CC01)

---

## 8. Open Points / نقاط بحاجة لتأكيد
1. Confirm final policy values for: vessel age threshold, inspection triggers, certificate validity (months).
2. Confirm reminder cadence + penalty schedule (configuration source of truth).
3. Confirm whether name reservation is an internal capability or a separate integrated system.
4. Confirm customs applicability rules for all imported/non-imported paths (not only the <24m scenario described in the source).
