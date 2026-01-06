# SP03 – Vessel/Unit Data Capture & Policy Validation

## Purpose
Collect vessel/unit details and apply key business rules (age policy, penalty trigger, prohibited list).

## Trigger
Applicant profile validated (SP02 completed).

## Inputs
- Vessel/unit type
- Build year and (where applicable) build material
- Acquisition path: New build / Purchase / Flag change
- Construction dates (start/end) or relevant acquisition dates
- Vessel attributes used for downstream routing (length, tonnage, activity, category)

## Outputs
- Validated vessel/unit record (ready for documents collection)
- Penalty line item (if triggered)
- Rejection in case of prohibited vessel or hard validation failure

## Swimlanes
- Applicant (Owner/Agent)
- MTCIT System (REG001 Orchestrator)

## Steps (Sequenced)
| # | Step | Swimlane | Event Type (Optional) |
|---|------|----------|----------------------|
| 1 | Select vessel/unit type | Applicant |  |
| 2 | Enter vessel/unit core details (build year, length/tonnage, activity) | Applicant |  |
| 3 | Validate vessel age policy (config/DMN) | MTCIT System |  |
| 4 | G05 – Age policy passed? | MTCIT System |  |
| 5 | If failed: show “cannot register” + allow re-entry where policy permits | MTCIT System + Applicant |  |
| 6 | Select acquisition path (New build / Purchase / Flag change) | Applicant |  |
| 7 | Enter construction start/end dates (if applicable) | Applicant |  |
| 8 | Check penalty trigger: >30 days after construction end at submission (config) | MTCIT System |  |
| 9 | G06 – Penalty applies? | MTCIT System |  |
| 10 | If yes: add penalty line item to fees | MTCIT System |  |
| 11 | Check prohibited/banned vessel list (config) | MTCIT System |  |
| 12 | G07 – Vessel prohibited? | MTCIT System |  |
| 13 | If prohibited: reject + notify + terminate | MTCIT System | Message (send) + Error |
| 14 | If not prohibited: continue to SP04 | MTCIT System |  |

## Gateways
- **G05 – Vessel age policy passed?**
  - Pass → proceed
  - Fail → stop with user warning / allow correction (as defined by policy)

- **G06 – Penalty applies?**
  - Yes → add penalty
  - No → proceed

- **G07 – Vessel prohibited/banned?**
  - Yes → reject + notify
  - No → proceed

## Notes
- Age validation is configuration/DMN-driven; actual thresholds may vary by material/category.
- Penalty timing rule is configuration-driven; the source describes a 30-day example.
