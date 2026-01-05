# REG001 – Executive Summary (Scope)

## Service Identification
- **Service Code (Project):** REG001
- **Service Name (EN):** Issuing Temporary Registration Certificate for a Ship or Marine Unit
- **Service Name (AR):** إصدار شهادة تسجيل سفينة أو وحدة بحرية مؤقتة
- **System Service Code:** MTCIT-REG-001
- **Release / Sprint:** 1 / 1

---

## 1. Purpose / الغرض
The purpose of REG001 is to enable ship owners or authorized agents to obtain a **temporary registration certificate** in the Sultanate of Oman, allowing the vessel/marine unit to operate legally **while permanent registration requirements are being completed**.

الغرض من الخدمة هو تمكين مالك السفينة أو الوكيل من **إصدار شهادة تسجيل مؤقتة** تتيح تشغيل السفينة/الوحدة البحرية بشكل قانوني خلال الفترة المؤقتة لحين استكمال متطلبات التسجيل الدائم.

---

## 2. Business Value / القيمة التجارية
- Reduces time-to-operation for vessels requiring immediate legal documentation.
- Provides a controlled, auditable pathway from temporary registration to permanent registration.
- Improves service transparency via automated validations (identity, CR, customs) and clear decision outcomes.

---

## 3. Target Users / المستفيدون
- **Primary:** Vessel owners / ملاك السفن
- **Authorized agent:** Ship agent acting on behalf of the owner / الوكيل
- **Eligibility highlights (per source):** Omani citizen or resident; vessel must not be on the banned/blocked list.

---

## 4. High-Level Scope Summary / ملخص النطاق
### In Scope
- Initiation of temporary registration request and guided application journey.
- Identity authentication via **ROP PKI**.
- Validation of **Commercial Registration (CR)** and related business activity (for companies).
- Vessel data capture (type, year built, construction/sales details) and validations (e.g., vessel age policy).
- Dynamic document collection based on configured rules.
- Orchestration of external approvals/steps where applicable:
  - **MAFWR** approval for fishing vessels
  - Inspection workflow triggering based on criteria (e.g., construction phase, length threshold, tug boat)
  - **Bayan (Customs)** integration for imported vessels/units where applicable
- Fee/penalty charging and payment step, followed by certificate issuance.
- Notifications (approval/rejection/payment/certificate issued; reminders for permanent registration; penalties for overdue).

### Out of Scope (at this stage)
- Detailed BPMN-level process maps and exception flows (covered in later BA artifacts).
- Detailed service decomposition, API contracts, and event schema definitions (covered in Solution Design).
- Full permanent registration workflow (separate service/phase).

---

## 5. Key Stakeholders (Preview) / أصحاب المصلحة (ملخص)
- Ministry of Transport, Communications and Information Technology – Maritime Affairs / قسم التسجيل
- Royal Oman Police (ROP) – PKI + identity, and Bayan/customs interactions
- Oman Business Platform (Invest Easy) – CR and activity validation
- MAFWR – fishing vessel approval
- Inspection / Classification entities
- Payment gateway

---

## 6. Success Criteria / مؤشرات النجاح
- Requests processed with clear outcomes (approved/rejected) and traceable reasons.
- Certificate issuance is gated by successful validations and payment.
- Automated reminders and penalties support timely transition to permanent registration.

---

## 7. Assumptions & Gaps / افتراضات ونواقص
- The proposal document `00-proposal/offer.md` was not found in the service folder at the time of writing; this scope is based on:
  - The service technical design README
  - Converted source service description (from the service `.docx`)
- Document requirements and some thresholds (e.g., age limit, reminder timing) are configurable and require confirmation from business owners.
