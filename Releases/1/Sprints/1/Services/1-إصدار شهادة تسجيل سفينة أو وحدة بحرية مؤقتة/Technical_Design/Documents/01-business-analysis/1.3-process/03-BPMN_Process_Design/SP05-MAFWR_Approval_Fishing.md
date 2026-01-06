# SP05 – MAFWR Approval (Fishing Vessels)

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
- **G09 – Fishing vessel?**
  - Yes → run SP05
  - No → skip

- **G10 – MAFWR approved?**
  - Approved → proceed
  - Rejected → reject + notify

## Notes
- Approval is conditional and acts as a hard gate for fishing vessels.
