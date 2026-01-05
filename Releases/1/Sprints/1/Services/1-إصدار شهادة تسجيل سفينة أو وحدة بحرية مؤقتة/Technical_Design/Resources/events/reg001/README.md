# REG-001 Temporary Vessel Registration Events

## إصدار شهادة تسجيل سفينة أو وحدة بحرية مؤقتة

**Total: 35 Events + 2 Channels = 37 Files**

---

## 📥 INBOUND EVENTS (Responses from External Systems)

| # | Event | Topic | Source | Description |
|---|-------|-------|--------|-------------|
| **Invest Easy - CR Validation** |||||
| 1 | `cr-validation-valid` | `investeasy.cr-validation` | Invest Easy | السجل التجاري ساري |
| 2 | `cr-validation-invalid` | `investeasy.cr-validation` | Invest Easy | السجل التجاري غير ساري |
| **Invest Easy - Activity Validation** |||||
| 3 | `activity-validation-active` | `investeasy.activity-validation` | Invest Easy | النشاط البحري ساري |
| 4 | `activity-validation-inactive` | `investeasy.activity-validation` | Invest Easy | النشاط البحري غير ساري |
| **MAFWR - Fishing Approval** |||||
| 5 | `mafwr-approved` | `mafwr.fishing-approval` | وزارة الثروة الزراعية | موافقة الصيد |
| 6 | `mafwr-rejected` | `mafwr.fishing-approval` | وزارة الثروة الزراعية | رفض الصيد |
| **Maritime - Vessel Inspection** |||||
| 7 | `inspection-passed` | `maritime.vessel-inspection` | Maritime Affairs | نجح الفحص |
| 8 | `inspection-failed` | `maritime.vessel-inspection` | Maritime Affairs | فشل الفحص |
| 9 | `inspection-cancelled` | `maritime.vessel-inspection` | Maritime Affairs | إلغاء الفحص |
| **Bayan - Customs Clearance** |||||
| 10 | `bayan-customs-cleared` | `bayan.customs-clearance` | نظام بيان | التخليص الجمركي |
| 11 | `bayan-customs-rejected` | `bayan.customs-clearance` | نظام بيان | رفض جمركي |
| **Bank Muscat - Payment** |||||
| 12 | `payment-successful` | `bankmuscat.payment-response` | Bank Muscat | الدفع ناجح |
| 13 | `payment-failed` | `bankmuscat.payment-response` | Bank Muscat | فشل الدفع |
| 14 | `payment-cancelled` | `bankmuscat.payment-response` | Bank Muscat | إلغاء الدفع |
| **Applicant - Documents** |||||
| 15 | `documents-submitted` | `applicant.documents` | Portal | تم تقديم المستندات |
| **Request Lifecycle** |||||
| 16 | `request-cancelled-by-user` | `request.lifecycle` | Applicant | إلغاء من المتقدم |
| 17 | `request-cancelled-by-system` | `request.lifecycle` | System | إلغاء تلقائي (timeout) |
| 18 | `request-cancelled-by-admin` | `request.lifecycle` | Admin | إلغاء إداري |

---

## 📤 OUTBOUND EVENTS (Requests to External Systems)

| # | Event | Topic | Target | Description |
|---|-------|-------|--------|-------------|
| **Invest Easy Requests** |||||
| 19 | `cr-validation-request` | `investeasy.cr-request` | Invest Easy | طلب التحقق من السجل التجاري |
| 20 | `activity-validation-request` | `investeasy.activity-request` | Invest Easy | طلب التحقق من النشاط |
| **MAFWR Request** |||||
| 21 | `mafwr-approval-request` | `mafwr.fishing-request` | MAFWR | طلب موافقة الصيد |
| **Maritime Request** |||||
| 22 | `inspection-request` | `maritime.inspection-request` | Maritime | طلب فحص السفينة |
| **Bayan Request** |||||
| 23 | `bayan-customs-request` | `bayan.customs-request` | Bayan | طلب التخليص الجمركي |
| **Bank Muscat - Invoice & Payment** |||||
| 24 | `invoice-issued` | `bankmuscat.invoice` | Bank Muscat | إصدار فاتورة |
| 25 | `invoice-expired` | `bankmuscat.invoice` | Bank Muscat | انتهاء صلاحية الفاتورة |
| 26 | `invoice-cancelled` | `bankmuscat.invoice` | Bank Muscat | إلغاء الفاتورة |
| 27 | `payment-request` | `bankmuscat.payment-request` | Bank Muscat | طلب الدفع |
| **Applicant Notifications** |||||
| 28 | `notification` | `applicant.notification` | Portal | إشعار عام |
| 29 | `status-update` | `applicant.notification` | Portal | تحديث الحالة |
| 30 | `penalty-applied` | `applicant.notification` | Portal | تطبيق غرامة |
| 31 | `permanent-registration-reminder` | `applicant.notification` | Portal | تذكير التسجيل الدائم |
| **Vessel Name Reservation** |||||
| 32 | `name-reserved` | `vessel.name-reservation` | Registry | تم حجز الاسم |
| 33 | `name-reservation-released` | `vessel.name-reservation` | Registry | تم تحرير الحجز |
| 34 | `name-reservation-cancelled` | `vessel.name-reservation` | Registry | إلغاء حجز الاسم |
| **Certificate Issuance** |||||
| 35 | `temporary-certificate-issued` | `certificate.issuance` | Certificate Svc | إصدار الشهادة المؤقتة |

---

## 📡 CHANNELS

| Channel | Type | Topics |
|---------|------|--------|
| `reg001-inbound.channel` | Kafka Inbound | 8 topics |
| `reg001-outbound.channel` | Kafka Outbound | 10 topics |

### Inbound Topics
- `mtcit.reg001.investeasy.cr-validation`
- `mtcit.reg001.investeasy.activity-validation`
- `mtcit.reg001.mafwr.fishing-approval`
- `mtcit.reg001.maritime.vessel-inspection`
- `mtcit.reg001.bayan.customs-clearance`
- `mtcit.reg001.bankmuscat.payment-response`
- `mtcit.reg001.applicant.documents`
- `mtcit.reg001.request.lifecycle`

### Outbound Topics
- `mtcit.reg001.investeasy.cr-request`
- `mtcit.reg001.investeasy.activity-request`
- `mtcit.reg001.mafwr.fishing-request`
- `mtcit.reg001.maritime.inspection-request`
- `mtcit.reg001.bayan.customs-request`
- `mtcit.reg001.bankmuscat.invoice`
- `mtcit.reg001.bankmuscat.payment-request`
- `mtcit.reg001.applicant.notification`
- `mtcit.reg001.vessel.name-reservation`
- `mtcit.reg001.certificate.issuance`

---

## 🎯 DMN Tables (To Create)

| DMN | Purpose |
|-----|---------|
| `reg001-documents-required` | Required docs per vessel type |
| `reg001-vessel-age-validation` | Vessel age limits |
| `reg001-fee-calculation` | Service fees |
| `reg001-penalty-calculation` | Late registration penalty |
| `reg001-inspection-required` | Inspection criteria |
| `reg001-mafwr-required` | Fishing approval needed? |
| `reg001-customs-required` | Customs clearance needed? |

---

## Architecture Notes

### Separation of Concerns

| Layer | Responsibility | Implementation |
|-------|----------------|----------------|
| **Security** | PKI Authentication (شرطة عمان السلطانية) | Keycloak / Spring Security |
| **Business Rules** | Calculations, validations, decisions | DMN Tables |
| **Events** | External integrations, state changes | Kafka Events |

### Topic Naming Convention
```
mtcit.reg001.{source-system}.{business-action}
```

---

*Generated: January 2026*
