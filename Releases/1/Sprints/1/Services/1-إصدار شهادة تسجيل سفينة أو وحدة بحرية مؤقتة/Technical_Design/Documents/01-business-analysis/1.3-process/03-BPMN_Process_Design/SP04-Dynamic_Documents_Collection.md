# SP04 – Dynamic Documents Collection

## Purpose
Determine required documents dynamically and collect/upload them from the applicant.

## Trigger
Vessel data validated (SP03 completed).

## Inputs
- Vessel category/type
- Acquisition type (new build/purchase/flag change)
- Intended activity
- Config/DMN document rules

## Outputs
- Complete document set attached to the request
- Rework loop for missing/invalid documents

## Swimlanes
- Applicant (Owner/Agent)
- MTCIT System (REG001 Orchestrator)

## Steps (Sequenced)
| # | Step | Swimlane | Event Type (Optional) |
|---|------|----------|----------------------|
| 1 | Determine required documents based on rules (DMN/config) | MTCIT System |  |
| 2 | Display required documents list (incl. mandatory/optional) | MTCIT System |  |
| 3 | Upload documents | Applicant | Timer boundary (optional) |
| 4 | Validate document completeness and basic file constraints | MTCIT System |  |
| 5 | G08 – Documents complete? | MTCIT System |  |
| 6 | If not complete: notify missing items and loop back to upload | MTCIT System + Applicant | Message (send) |
| 7 | If complete: continue to routing (SP05/SP06/SP07 as applicable) | MTCIT System |  |

## Gateways
- **G08 – Documents complete?**
  - Yes → proceed
  - No → loop until complete (or terminate per policy/timeouts)

## Notes
- The source document explicitly states document requirements are dynamic and may change via configuration.
