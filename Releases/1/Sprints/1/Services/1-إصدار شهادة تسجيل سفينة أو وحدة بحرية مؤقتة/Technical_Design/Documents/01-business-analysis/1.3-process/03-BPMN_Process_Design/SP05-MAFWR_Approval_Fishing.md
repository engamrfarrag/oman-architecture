# SP05 – MAFWR Approval (Fishing Vessels)

## Functional Responsibility
This subprocess is responsible for:
- Determining whether the request requires MAFWR approval (fishing vessels)
- Sending the approval request to MAFWR and tracking the response
- Applying a hard-gate decision (approve/reject) and notifying the applicant

## Purpose
Obtain approval from MAFWR for fishing vessels before proceeding to issuance.

## Trigger
Documents are complete (SP04 completed) AND vessel category indicates fishing vessel.

## Inputs
- Request ID
- Vessel category/type
- Owner/applicant data
- Vessel details relevant to fishing approval

## Outputs
- Approval decision (Approved/Rejected)
- Notification to applicant

## Swimlanes
- Applicant (Owner/Agent)
- MTCIT System (REG001 Orchestrator)
- MAFWR

## Steps (Sequenced)
| # | Step | Swimlane | Event Type (Optional) |
|---|------|----------|----------------------|
| 1 | G09 – Fishing vessel? | MTCIT System |  |
| 2 | If yes: send approval request to MAFWR | MTCIT System → MAFWR | Message (send) |
| 3 | Receive MAFWR decision | MAFWR → MTCIT System | Message (receive) (+Timer boundary) |
| 4 | G10 – MAFWR approved? | MTCIT System |  |
| 5 | If rejected: notify applicant with reason and terminate | MTCIT System | Message (send) + Error |
| 6 | If approved: notify applicant and continue to next routing (inspection/customs/payment) | MTCIT System | Message (send) |
| 7 | If not fishing vessel: skip SP05 | MTCIT System |  |

## Gateways

### G09 – Fishing vessel?
- Yes → run SP05
- No → skip

### Decision Rule (Business)
- Inputs: vessel/unit category/type, intended activity
- Rule Source (DMN / Config): `reg001-mafwr-required.dmn` (or equivalent config/policy mapping)
- Outcomes:
  - Yes (requires MAFWR) → send approval request
  - No → continue to next routing without MAFWR

### G10 – MAFWR approved?
- Approved → proceed
- Rejected → reject + notify

### Decision Rule (Business)
- Inputs: MAFWR decision status, rejection reason (if any)
- Rule Source (DMN / Config): External decision (MAFWR response)
- Outcomes:
  - Approved → proceed to downstream routing (SP06/SP07/SP08)
  - Rejected → reject + notify with reason; terminate

## Notes
- Approval is conditional and acts as a hard gate for fishing vessels.
