# SP09 – Certificate Issuance & Notifications (Functional Requirements)

This file consolidates all SP09 functional requirements in a single document, while preserving the original FR IDs for traceability.

---

## FR-SP09-001
Title: Generate temporary registration certificate (PDF + QR)

Description:
The system shall generate the temporary registration certificate artifact after successful payment, including a PDF output and a QR code that references the certificate.

Trigger:
Successful payment confirmation received (SP08 completed).

Inputs:
- Paid invoice reference
- Payment receipt reference
- Applicant and vessel/unit data (from request)
- Certificate validity period (config: REG-001-CONF-03)

Processing / Rules:
- Confirm payment status = Success.
- Confirm required subprocesses are completed (guards/state policy).
- Generate certificate PDF populated with the approved request data.
- Generate QR code referencing certificate identifier and/or verification endpoint.

Outputs:
- Temporary Registration Certificate (PDF)
- QR code embedded in certificate
- Certificate identifier/reference

Success Criteria:
- A certificate artifact is generated and can be retrieved for the request.

Failure Conditions:
- Missing prerequisites for issuance (unexpected state)
- Certificate generation service unavailable

Related Gateways:
- (N/A)

Related Capabilities:
- CAP-22, CAP-26

---

## FR-SP09-002
Title: Persist issuance and notify applicant with certificate access

Description:
The system shall persist certificate issuance metadata, link the certificate to the request, and notify the applicant that the certificate has been issued, including how to access/download it.

Trigger:
Certificate artifact generated (FR-SP09-001 completed).

Inputs:
- Certificate identifier/reference
- Certificate validity dates
- Applicant contact details

Processing / Rules:
- Persist certificate issuance record (certificate id, issuance date/time, validity range, request reference).
- Update request status to Issued (Temporary).
- Send a “certificate issued” notification including: certificate reference, validity period, and next-step guidance to complete permanent registration.
- Provide certificate access (download link/view action) to the applicant.

Outputs:
- Issuance record linked to request
- Issuance notification (SMS/Email/Inbox)
- Certificate access available to applicant

Success Criteria:
- Applicant can retrieve the certificate and receives an issuance notification.

Failure Conditions:
- Notification provider unavailable
- Persistence failure

Related Gateways:
- (N/A)

Related Capabilities:
- CAP-22, CAP-23, CAP-26

---

## Data Dictionary (SP09)
Aligned with the REG001 Data Dictionary template in `00-overview/Data-Dictionary-Template.md`.

| Data Element (Canonical) | Description | Source / Owner | Data Type | Format / Allowed Values | Validation / Rule Source | Required (Y/N) | Used By (FR IDs) | Example / Notes | Classification |
|---|---|---|---|---|---|---|---|---|---|
| certificateId | Certificate identifier/reference | LS-CERT | string |  | Certificate service | Y | FR-SP09-001, FR-SP09-002 | Stored and shared | Internal |
| certificatePdfRef | Stored PDF reference | LS-CERT/Storage | string | URI/id | Storage policy | Y | FR-SP09-001, FR-SP09-002 | Download link uses this | Internal |
| certificateQrPayload | QR code payload | LS-CERT | string | URL or token | Verification policy | Y | FR-SP09-001 | Should not leak PII | Internal |
| issuanceDateTime | Issuance timestamp | LS-CERT | datetime | ISO-8601 | System clock | Y | FR-SP09-002 |  | Internal |
| validityStartDate | Start of validity | LS-CERT | date | ISO-8601 | Config-driven | Y | FR-SP09-002 |  | Internal |
| validityEndDate | End of validity | LS-CERT | date | ISO-8601 | REG-001-CONF-03 | Y | FR-SP09-002 |  | Internal |
| requestStatus | Overall request status | LS-REG-ORCH | enum | ISSUED_TEMPORARY, ... | Orchestrator policy | Y | FR-SP09-002 |  | Internal |
