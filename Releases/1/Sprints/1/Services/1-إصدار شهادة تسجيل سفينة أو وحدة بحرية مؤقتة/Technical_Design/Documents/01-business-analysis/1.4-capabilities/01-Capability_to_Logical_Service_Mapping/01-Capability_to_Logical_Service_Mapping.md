# REG001 – Capability to Logical Service Mapping

## Service Identification
- **Service Code (Project):** REG001
- **Service Name (EN):** Issuing Temporary Registration Certificate for a Ship or Marine Unit
- **Service Name (AR):** إصدار شهادة تسجيل سفينة أو وحدة بحرية مؤقتة
- **System Service Code:** MTCIT-REG-001
- **Release / Sprint:** 1 / 1

---

## 1. Purpose / الغرض
This document maps **business capabilities** of REG001 to **logical services** (service candidates) that will likely own/implement them in solution design.

> Notes:
> - “Logical service” here means a **conceptual service boundary** (candidate microservice / shared platform capability / integration adapter), not final deployment.
> - Capability IDs are for traceability across BA → Solution Design.

---

## 2. Sources & Traceability / المصادر والتتبّع
Primary sources used:
- BPMN decomposition and subprocesses in:
  - `01-business-analysis/1.3-process/02-Business_Process_Overview/`
  - `01-business-analysis/1.3-process/03-BPMN_Process_Design/` (CC01, SP02…SP10)
- DMN rules in: `Technical_Design/Resources/dmn/reg001/`
  - `reg001-documents-required.dmn`
  - `reg001-fee-calculation.dmn`
  - `reg001-penalty-calculation.dmn`
  - `reg001-vessel-age-validation.dmn`
  - `reg001-inspection-required.dmn`
  - `reg001-mafwr-required.dmn`
  - `reg001-customs-required.dmn`
- Converted service source document (service `.docx`) describing end-to-end steps and integrations.

### Pre-FR Optimization Hooks
To enable clean Functional Requirements (FR) generation with minimal ambiguity:
- Each subprocess spec (SP02…SP10) includes a **Functional Responsibility** section whose bullets can be translated into 1–2 FRs each.
- Each gateway (G02…G16) includes a **Decision Rule (Business)** block capturing **Inputs**, **Rule Source (DMN/Config/Integration)**, and **Outcomes**.

---

## 3. Logical Service Catalog (Candidate Services)
These are the logical services referenced in the mapping table.

- **LS-REG-ORCH** – REG001 Orchestrator (process engine / workflow orchestration)
- **LS-IAM** – Identity & Access (authentication, PKI session, authorization)
- **LS-APPLICANT** – Applicant Profile Service (individual/company profile, agent representation)
- **LS-CR-ADAPTER** – CR / Invest Easy Integration Adapter
- **LS-ACTIVITY-VALIDATION** – Company Activity Eligibility Validation
- **LS-VESSEL** – Vessel/Marine Unit Data Service (capture, store, validate core attributes)
- **LS-POLICY** – Policy & Compliance Rules (DMN decisions, prohibited list checks, thresholds)
- **LS-DOCS** – Document Intake & Verification (required docs, upload, completeness)
- **LS-APPROVALS** – External Approval Orchestration (MAFWR routing + status)
- **LS-INSPECTION** – Inspection Orchestration (request, receive result, validate)
- **LS-CUSTOMS** – Bayan/Customs Integration Adapter
- **LS-FEES** – Fees & Invoicing (fee calculation, invoice generation)
- **LS-PENALTIES** – Penalties (calculation, applying line items)
- **LS-NAME** – Name Reservation (hold/confirm/release)
- **LS-PAYMENTS** – Payment Gateway Adapter (payment initiation + confirmation)
- **LS-CERT** – Certificate Generation & Issuance (PDF + QR, registry entry)
- **LS-NOTIFY** – Notifications (SMS/Email/in-app)
- **LS-MONITOR** – Post-Issuance Monitoring (reminders cadence, overdue tracking)
- **LS-AUDIT** – Audit/Trace (decision logging, request trail)
- **LS-CONFIG** – Dynamic Configuration (document rules, thresholds, timing)

---

## 4. Capability → Logical Service Mapping

Legend:
- **Primary logical service** = the service that owns the capability end-to-end.
- **Supporting services** = services called/used to fulfil it.

| Cap ID | Capability (EN) | Capability (AR) | Description (Business) | Trace (BPMN/SP) | Key Rules (DMN/Config) | Primary Logical Service | Supporting Logical Services / Integrations |
|---|---|---|---|---|---|---|---|
| CAP-01 | Service discovery & start request | بدء الخدمة وقراءة التعليمات | Allow applicant to select REG001, view instructions, initiate a request/case. | P0 (entry) | Config-driven content | LS-REG-ORCH | LS-CONFIG, LS-AUDIT |
| CAP-02 | Identity authentication (PKI) | التصديق الإلكتروني (PKI) | Authenticate applicant; allow/terminate based on ROP PKI result. | CC01 | Security policy | LS-IAM | (Integration) ROP PKI, LS-REG-ORCH, LS-AUDIT |
| CAP-03 | Applicant profiling (individual/company) | تحديد نوع المستفيد | Capture beneficiary type and required identifiers. | SP02 (G02) | Config-driven validation rules | LS-APPLICANT | LS-REG-ORCH, LS-AUDIT |
| CAP-04 | Commercial Registration validation | التحقق من السجل التجاري | Validate company CR for company applicants. | SP02 (G03) | Integration contract; eligibility rules | LS-CR-ADAPTER | (Integration) Invest Easy, LS-REG-ORCH |
| CAP-05 | Company activity eligibility validation | التحقق من النشاط التجاري | Validate company activity against vessel/unit type. | SP02 (G04) | Policy mapping (config/DMN as needed) | LS-ACTIVITY-VALIDATION | LS-CR-ADAPTER, LS-POLICY, LS-REG-ORCH |
| CAP-06 | Vessel/unit data capture | إدخال بيانات الوحدة البحرية | Capture unit type, year built, dimensions, ownership/acquisition details, etc. | SP03 | Data model constraints | LS-VESSEL | LS-REG-ORCH, LS-AUDIT |
| CAP-07 | Vessel age policy validation | التحقق من عمر السفينة | Evaluate age policy; block/allow with warnings per policy. | SP03 (G05) | `reg001-vessel-age-validation.dmn` | LS-POLICY | LS-VESSEL, LS-CONFIG, LS-AUDIT |
| CAP-08 | Prohibited/banned vessel check | التحقق من عدم الحظر | Reject prohibited vessels based on configured banned list/policy. | SP03 (G07) | Config (REG-001-CONF-04) | LS-POLICY | LS-CONFIG, LS-VESSEL, LS-REG-ORCH |
| CAP-09 | Construction/purchase path handling | مسار بناء/بيع/تغيير علم | Capture acquisition type, dates, and context needed for downstream rules (penalty, docs). | SP03 | Config + validations | LS-VESSEL | LS-POLICY, LS-REG-ORCH |
| CAP-10 | Penalty trigger evaluation (timing) | التحقق من تطبيق مخالفة | If submission exceeds configured time (e.g., >30 days after construction end), add penalty line item. | SP03 (G06) / SP10 | `reg001-penalty-calculation.dmn` + config (REG-001-CONF-02) | LS-PENALTIES | LS-REG-ORCH, LS-FEES, LS-AUDIT |
| CAP-11 | Determine required documents dynamically | تحديد المستندات المطلوبة ديناميكياً | Calculate required documents based on unit type/activity/acquisition/length. | SP04 | `reg001-documents-required.dmn` + dynamic config (REG-001-CONF-01) | LS-DOCS | LS-POLICY, LS-CONFIG, LS-REG-ORCH |
| CAP-12 | Document upload & completeness validation | رفع المستندات والتحقق من الاكتمال | Collect documents, check completeness, loop until satisfied or reject. | SP04 (G08) | Rules from CAP-11 | LS-DOCS | Applicant UI, LS-AUDIT |
| CAP-13 | MAFWR approval routing (fishing) | موافقة وزارة الثروة الزراعية (سفن الصيد) | If fishing vessel, request approval and process response. | SP05 (G09–G10) | `reg001-mafwr-required.dmn` (or rule/config) | LS-APPROVALS | (Integration) MAFWR, LS-NOTIFY, LS-REG-ORCH |
| CAP-14 | Inspection requirement decision | تحديد الحاجة للمعاينة | Decide whether inspection is required based on rules (construction phase, length > 24m, tug, etc.). | SP06 (G11) | `reg001-inspection-required.dmn` | LS-POLICY | LS-VESSEL, LS-CONFIG |
| CAP-15 | Inspection orchestration & result validation | إدارة دورة المعاينة | Initiate inspection, receive result, validate pass/fail and proceed/reject. | SP06 (G12) | Config/criteria; inspection results | LS-INSPECTION | (Integration) Inspection/Classification, LS-NOTIFY, LS-REG-ORCH |
| CAP-16 | Customs/Bayan requirement decision | تحديد الحاجة لبيان/الجمارك | Decide whether Bayan customs flow is required (e.g., imported flows). | SP07 (G13) | `reg001-customs-required.dmn` | LS-POLICY | LS-VESSEL, LS-CONFIG |
| CAP-17 | Bayan/customs clearance integration | تكامل بيان/الإفراج الجمركي | Request customs clearance and receive import document + final clearance. | SP07 (G14) | Integration contract; document receipt | LS-CUSTOMS | (Integration) Bayan, LS-DOCS, LS-REG-ORCH |
| CAP-18 | Fee calculation | حساب الرسوم | Calculate fees based on unit type, tonnage, length, activity, registration type. | SP08 | `reg001-fee-calculation.dmn` + config (REG-001-SET-06) | LS-FEES | LS-POLICY, LS-CONFIG |
| CAP-19 | Name reservation (hold & confirm) | حجز اسم السفينة | Reserve a vessel name during payment window; release on failure/expiry. | SP08 | Configured hold duration | LS-NAME | LS-REG-ORCH, LS-PAYMENTS |
| CAP-20 | Invoice generation | إصدار الفاتورة | Generate invoice including fees and penalties; provide invoice reference to applicant. | SP08 | Fee + penalty aggregation | LS-FEES | LS-PENALTIES, LS-AUDIT |
| CAP-21 | Payment processing | سداد الرسوم | Initiate payment, handle confirmation/failure, drive next steps accordingly. | SP08 (G15) | Payment policy; expiry rules | LS-PAYMENTS | (Integration) ePayment, LS-NOTIFY, LS-NAME, LS-REG-ORCH |
| CAP-22 | Certificate issuance (temporary) | إصدار الشهادة المؤقتة | Produce temporary certificate (PDF + QR), register issuance, notify applicant. | SP09 | Certificate validity config (REG-001-CONF-03) | LS-CERT | LS-REG-ORCH, LS-NOTIFY, LS-AUDIT |
| CAP-23 | Notifications (status + outcomes) | الإشعارات | Notify applicant on approval/rejection/payment/certificate issued and other key statuses. | SP02–SP10 | Notification templates | LS-NOTIFY | SMS/Email providers, LS-REG-ORCH |
| CAP-24 | Post-issuance reminders | تنبيهات استكمال التسجيل الدائم | Send periodic reminders (e.g., every 7 days) until permanent registration completed. | SP10 | Configured cadence | LS-MONITOR | LS-NOTIFY, LS-CONFIG |
| CAP-25 | Overdue penalties (post-issuance) | غرامات التأخير بعد الإصدار | Apply overdue penalty after configured time if permanent registration not completed. | SP10 (G16) | `reg001-penalty-calculation.dmn` + config | LS-PENALTIES | LS-MONITOR, LS-FEES |
| CAP-26 | Audit & traceability | التتبّع والتدقيق | Maintain auditable trail of inputs, decisions, integrations, outcomes. | Cross-cutting | Logging policy | LS-AUDIT | All logical services |

---

## 5. Notes / ملاحظات
- Capability boundaries above are intentionally **business-oriented**; technical design will later refine service ownership and coupling.
- Rules for documents/fees/penalties/inspection/MAFWR/customs are best implemented through a **rules engine (DMN) + config service** to preserve business agility.
- Capabilities CAP-24/CAP-25 represent **long-running monitoring**, which may be implemented via scheduler + event-driven state changes in later phases.
