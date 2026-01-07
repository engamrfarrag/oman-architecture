# SP07 – Bayan Customs Clearance (Functional Requirements)

This file consolidates all SP07 functional requirements in a single document, while preserving the original FR IDs for traceability.

---

## FR-SP07-001
Title: Determine whether Bayan customs clearance is required using DMN/config

Description:
The system shall determine whether Bayan customs clearance is required for the request based on acquisition context and import indicators.

Trigger:
Customs routing decision is evaluated.

Inputs:
- Acquisition type (NEW_BUILD/PURCHASE/FLAG_CHANGE)
- Import indicators (isImported, buildCountry, isLocalBuild)
- Previous flag state (if applicable)

Processing / Rules:
- Evaluate `reg001-customs-required.dmn`.

Outputs:
- Customs required (Y/N)
- Clearance type and required documents notes

Success Criteria:
- Requests requiring Bayan are routed to customs clearance integration.

Failure Conditions:
- Rules evaluation failure

Related Gateways:
- G13

Related Capabilities:
- CAP-16

Rule Source:
- DMN: `Technical_Design/Resources/dmn/reg001/reg001-customs-required.dmn`

---

## FR-SP07-002
Title: Send Bayan customs request and store returned documents

Description:
When Bayan is required, the system shall send a customs clearance request to Bayan and store returned import documents (e.g., PDF) and clearance status.

Trigger:
Customs required = Yes.

Inputs:
- Request ID
- Applicant identity/CR references
- Vessel details required for customs

Processing / Rules:
- Send request to Bayan.
- Receive response including clearance status and documents.
- Store documents and link to request.

Outputs:
- Stored customs documents
- Clearance status

Success Criteria:
- Bayan response is linked to the correct request.

Failure Conditions:
- Bayan timeout/unavailable

Related Gateways:
- G14

Related Capabilities:
- CAP-17

---

## FR-SP07-003
Title: Reject request when Bayan clearance is rejected

Description:
The system shall reject the request when Bayan returns clearance status = Rejected.

Trigger:
Bayan response received.

Inputs:
- Clearance status
- Rejection reason (if provided)

Processing / Rules:
- Mark request as Rejected.
- Notify applicant with reason (if provided).

Outputs:
- Rejected request
- Rejection notification

Success Criteria:
- Rejected requests cannot proceed to payment.

Failure Conditions:
- None (business rejection)

Related Gateways:
- G14

Related Capabilities:
- CAP-17, CAP-23

---

## Data Dictionary (SP07)
Aligned with the REG001 Data Dictionary template in `00-overview/Data-Dictionary-Template.md`.

| Data Element (Canonical) | Description | Source / Owner | Data Type | Format / Allowed Values | Validation / Rule Source | Required (Y/N) | Used By (FR IDs) | Example / Notes | Classification |
|---|---|---|---|---|---|---|---|---|---|
| customsRequired | Whether Bayan is required | LS-POLICY | boolean | true/false | `reg001-customs-required.dmn` | Y | FR-SP07-001 |  | Internal |
| bayanRequestId | Outbound request reference | LS-CUSTOMS | string |  | Bayan contract | Conditional | FR-SP07-002 | Used for correlation | Internal |
| clearanceStatus | Customs clearance outcome | Bayan | enum | CLEARED, REJECTED, PENDING | Bayan response | Conditional | FR-SP07-002, FR-SP07-003 | Reject blocks payment | Internal |
| importDocumentPdfRef | Stored PDF document reference | LS-DOCS/Storage | string | URI/id | Storage | N | FR-SP07-002 | Bayan import doc | Internal |
| finalClearanceRef | Final clearance reference | Bayan | string |  | Bayan response | N | FR-SP07-002 |  | Internal |
| customsRejectionReason | Rejection reason (optional) | Bayan | string | free text/code | Bayan response | N | FR-SP07-003 | Included in notification | Internal |
