# SP07 – Bayan Customs Clearance (Conditional)

## Functional Responsibility
This subprocess is responsible for:
- Determining whether Bayan/customs clearance is required for the request
- Sending customs clearance requests and receiving documents/decisions
- Enforcing a hard-gate decision (cleared/rejected) and notifying the applicant

## Purpose
Request and receive customs/import documentation and clearance decision from Bayan when required.

## Trigger
Routing rules determine that customs clearance is required (e.g., imported vessel path).

## Inputs
- Request ID
- Applicant identity/CR references (as applicable)
- Vessel/unit details needed for customs

## Outputs
- Customs decision: Cleared / Rejected
- Import document (PDF) when provided
- Notifications in case of rejection

## Swimlanes
- Applicant (Owner/Agent)
- MTCIT System (REG001 Orchestrator)
- Bayan Customs

## Steps (Sequenced)
| # | Step | Swimlane | Event Type (Optional) |
|---|------|----------|----------------------|
| 1 | Determine if customs/Bayan is required | MTCIT System |  |
| 2 | G13 – Customs/Bayan required? | MTCIT System |  |
| 3 | If required: send customs request to Bayan | MTCIT System → Bayan | Message (send) |
| 4 | Receive Bayan response (documents + clearance status) | Bayan → MTCIT System | Message (receive) (+Timer boundary) |
| 5 | G14 – Customs cleared? | MTCIT System |  |
| 6 | If rejected: notify applicant with reason; terminate request | MTCIT System | Message (send) + Error |
| 7 | If cleared: continue to payment and issuance | MTCIT System |  |

## Gateways

### G13 – Customs/Bayan required?
- Yes → run SP07
- No → skip

### Decision Rule (Business)
- Inputs: vessel/unit path indicators (e.g., imported vs local), acquisition context, vessel attributes
- Rule Source (DMN / Config): `reg001-customs-required.dmn`
- Outcomes:
  - Yes → send Bayan request
  - No → skip SP07

### G14 – Customs cleared?
- Cleared → proceed
- Rejected → reject + notify

### Decision Rule (Business)
- Inputs: Bayan clearance status, import document (if any), rejection reason (if any)
- Rule Source (DMN / Config): Bayan Customs response
- Outcomes:
  - Cleared → proceed to payment/issuance
  - Rejected → reject + notify with reason; terminate

## Notes
- Model Bayan response as an intermediate message catch event; include timeouts per policy.
