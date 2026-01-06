# SP06 – Vessel Inspection (Conditional)

## Purpose
Perform technical inspection when rules require it (construction phase, vessel length threshold, tug boat, etc.) and confirm compliance.

## Trigger
Routing rules determine that inspection is required.

## Inputs
- Request ID
- Vessel category/type
- Vessel length (m)
- Gross tonnage (GT)
- Inspection requirement rules (DMN/config)

## Outputs
- Inspection outcome (Passed / Failed / Cancelled / Timeout)
- Inspection report reference
- Notification and/or escalation actions

## Swimlanes
- Applicant (Owner/Agent) – may provide reports where required
- MTCIT System (REG001 Orchestrator)
- Inspection Entity (Classification Society / Consultancy / MTCIT Technical Staff)
- (Optional) MTCIT Registration Dept (Backoffice) – escalation

## Steps (Sequenced)
| # | Step | Swimlane | Event Type (Optional) |
|---|------|----------|----------------------|
| 1 | Determine whether inspection is required (rules) | MTCIT System |  |
| 2 | G11 – Inspection required? | MTCIT System |  |
| 3 | If required: select/assign inspection type and entity | MTCIT System |  |
| 4 | Send inspection request to inspection entity | MTCIT System → Inspection Entity | Message (send) |
| 5 | Perform inspection and produce report/outcome | Inspection Entity |  |
| 6 | Receive inspection outcome | Inspection Entity → MTCIT System | Message (receive) (+Timer boundary) |
| 7 | G12 – Inspection passed? | MTCIT System |  |
| 8 | If failed/cancelled/timeout: notify applicant; terminate or escalate per policy | MTCIT System (+ Backoffice optional) | Message (send) + Error |
| 9 | If passed: continue to next routing (customs/payment/issuance) | MTCIT System |  |

## Gateways
- **G11 – Inspection required?**
  - Yes → run inspection
  - No → skip

- **G12 – Inspection passed?**
  - Passed → proceed
  - Failed/Cancelled/Timeout → reject or escalate per policy; notify applicant

## Notes
- The workspace includes a reusable BPMN inspection subprocess; model SP06 as a call activity in the main process.
- Timeouts should be modeled with boundary timer events.
