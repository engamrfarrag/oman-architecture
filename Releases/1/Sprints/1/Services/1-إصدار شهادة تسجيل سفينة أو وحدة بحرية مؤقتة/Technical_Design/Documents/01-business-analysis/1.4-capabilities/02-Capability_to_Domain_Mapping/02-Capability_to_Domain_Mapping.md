# REG001 – Capability to Domain Mapping

## Service Identification
- **Service Code (Project):** REG001
- **Service Name (EN):** Issuing Temporary Registration Certificate for a Ship or Marine Unit
- **Service Name (AR):** إصدار شهادة تسجيل سفينة أو وحدة بحرية مؤقتة
- **System Service Code:** MTCIT-REG-001
- **Release / Sprint:** 1 / 1

---

## 1. Purpose / الغرض
This document maps REG001 business capabilities to **business domains / bounded contexts**. It supports:
- Domain modeling (`02-solution-design/2.1-domain-model/`)
- Service decomposition (`02-solution-design/2.3-services/`)
- Integration and data ownership decisions (`02-solution-design/2.5-data-architecture/`)

### Pre-FR Optimization Hooks
For FR generation alignment, use the process anchors maintained in:
- `01-business-analysis/1.3-process/03-BPMN_Process_Design/` where each SP document includes **Functional Responsibility** bullets.
- Gateway **Decision Rule (Business)** blocks (inputs + rule source + outcomes) for each Gxx decision.

---

## 2. Domain Catalog (Proposed) / قائمة المجالات (مقترحة)
Domains are expressed as bounded contexts with ownership hints.

- **D01 – Registration (Temporary)** / التسجيل (مؤقت)
  - Owns the temporary registration case lifecycle and issuance state.

- **D02 – Party & Identity** / الأطراف والهوية
  - Owns applicant identity, representation (owner/agent), and authentication context.

- **D03 – Business/Company Eligibility** / أهلية الشركات
  - Owns CR validation and activity eligibility rules.

- **D04 – Vessel / Marine Unit** / السفن والوحدات البحرية
  - Owns vessel/unit master attributes used for validation, fees, inspection triggers.

- **D05 – Policy & Compliance** / السياسات والامتثال
  - Owns configurable policies (age, prohibited list), DMN decisions, thresholds.

- **D06 – Documents & Records** / المستندات والسجلات
  - Owns document requirements, intake, metadata, and completeness.

- **D07 – External Approvals** / الموافقات الخارجية
  - Owns approval workflows (e.g., MAFWR) and their state machines.

- **D08 – Inspection** / المعاينات
  - Owns inspection requests, assignments, outcomes, and validation.

- **D09 – Customs & Trade** / الجمارك والتجارة
  - Owns Bayan/customs clearance interactions and imported-vessel artifacts.

- **D10 – Billing, Fees & Payments** / الرسوم والدفع
  - Owns fees, penalties, invoicing, payment reconciliation.

- **D11 – Notifications & Communication** / الإشعارات والتواصل
  - Owns notification templates, channels, delivery tracking.

- **D12 – Monitoring & Enforcement** / المتابعة والالتزام
  - Owns post-issuance reminders, deadlines, overdue enforcement.

- **D13 – Audit & Governance** / التدقيق والحوكمة
  - Owns audit trail, decision logging, traceability, compliance reporting.

---

## 3. Capability → Domain Mapping

| Cap ID | Capability (EN) | Primary Domain | Secondary Domains (if any) | Notes |
|---|---|---|---|---|
| CAP-01 | Service discovery & start request | D01 Registration (Temporary) | D13 Audit & Governance | Case initiation and tracking are registration-owned; audit captures initiation. |
| CAP-02 | Identity authentication (PKI) | D02 Party & Identity | D13 Audit & Governance | Authentication is identity domain; outcomes logged for traceability. |
| CAP-03 | Applicant profiling (individual/company) | D02 Party & Identity | D01 Registration (Temporary) | Applicant profile and representation belongs to Party; registration consumes it. |
| CAP-04 | Commercial Registration validation | D03 Business/Company Eligibility | D02 Party & Identity | CR checks relate to company eligibility; tied to applicant. |
| CAP-05 | Company activity eligibility validation | D03 Business/Company Eligibility | D05 Policy & Compliance, D04 Vessel | Activity eligibility depends on vessel category and policies. |
| CAP-06 | Vessel/unit data capture | D04 Vessel / Marine Unit | D01 Registration (Temporary) | Vessel attributes are mastered in vessel context; registration references them. |
| CAP-07 | Vessel age policy validation | D05 Policy & Compliance | D04 Vessel | Policy evaluates vessel inputs. |
| CAP-08 | Prohibited/banned vessel check | D05 Policy & Compliance | D04 Vessel | Prohibited list is a policy asset. |
| CAP-09 | Construction/purchase path handling | D04 Vessel / Marine Unit | D05 Policy & Compliance | Acquisition attributes drive policies and document requirements. |
| CAP-10 | Penalty trigger evaluation (timing) | D10 Billing, Fees & Payments | D05 Policy & Compliance | Penalty rules are finance-owned; governed by policy configuration. |
| CAP-11 | Determine required documents dynamically | D06 Documents & Records | D05 Policy & Compliance, D04 Vessel | Document requirements are document domain with policy inputs. |
| CAP-12 | Document upload & completeness validation | D06 Documents & Records | D13 Audit & Governance | Document custody and completeness checks. |
| CAP-13 | MAFWR approval routing (fishing) | D07 External Approvals | D05 Policy & Compliance, D11 Notifications | Approval required decision comes from policy; results trigger notifications. |
| CAP-14 | Inspection requirement decision | D05 Policy & Compliance | D08 Inspection, D04 Vessel | Decision is policy; execution is inspection domain. |
| CAP-15 | Inspection orchestration & result validation | D08 Inspection | D07 External Approvals, D11 Notifications | Inspection flow owns request/result lifecycle. |
| CAP-16 | Customs/Bayan requirement decision | D05 Policy & Compliance | D09 Customs & Trade | Policy decides; customs domain executes. |
| CAP-17 | Bayan/customs clearance integration | D09 Customs & Trade | D06 Documents & Records, D11 Notifications | Customs documents may be stored/managed as records. |
| CAP-18 | Fee calculation | D10 Billing, Fees & Payments | D05 Policy & Compliance, D04 Vessel | Fee model depends on vessel attributes and configured rules. |
| CAP-19 | Name reservation (hold & confirm) | D01 Registration (Temporary) | D10 Billing, Fees & Payments | Name reservation supports registration; release rules align with payment. |
| CAP-20 | Invoice generation | D10 Billing, Fees & Payments | D01 Registration (Temporary) | Invoice belongs to billing; linked to registration case. |
| CAP-21 | Payment processing | D10 Billing, Fees & Payments | D11 Notifications, D01 Registration (Temporary) | Payment is billing-owned; status updates registration and triggers notifications. |
| CAP-22 | Certificate issuance (temporary) | D01 Registration (Temporary) | D06 Documents & Records, D11 Notifications | Certificate is a registration artifact; also a record/document. |
| CAP-23 | Notifications (status + outcomes) | D11 Notifications & Communication | D01 Registration (Temporary) | Registration emits events; notification domain delivers messages. |
| CAP-24 | Post-issuance reminders | D12 Monitoring & Enforcement | D11 Notifications, D01 Registration (Temporary) | Monitoring owns cadence and stop condition; notification delivers. |
| CAP-25 | Overdue penalties (post-issuance) | D12 Monitoring & Enforcement | D10 Billing, Fees & Payments | Monitoring triggers penalty; billing applies and invoices if needed. |
| CAP-26 | Audit & traceability | D13 Audit & Governance | (All) | Cross-cutting; consumes events/logs from all domains. |

---

## 4. Domain Boundaries (Quick Rules) / قواعد الحدود (مختصر)
- **A domain owns its data**: Registration owns certificate state; Billing owns invoices/payments; Documents owns uploaded files and metadata.
- **Policies are configuration-driven**: Keep thresholds, required documents, eligibility mapping outside hard-coded flows.
- **Integration adapters are not domains**: They belong to the domain they serve (e.g., Customs adapter in D09).

---

## 5. Next Steps / الخطوات التالية
- Use this mapping to create the initial **Context Map** and **bounded context relationships** in `02-solution-design/2.1-domain-model/02-Business_Domain_Context_Map`.
- Use the Capability → Logical Service mapping to propose service decomposition in `02-solution-design/2.3-services/01-Service_Decomposition`.
