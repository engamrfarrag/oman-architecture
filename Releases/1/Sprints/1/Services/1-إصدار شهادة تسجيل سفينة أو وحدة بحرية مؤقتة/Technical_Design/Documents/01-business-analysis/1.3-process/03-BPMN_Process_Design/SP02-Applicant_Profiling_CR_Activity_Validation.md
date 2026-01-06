# SP02 – Applicant Profiling + CR/Activity Validation

## Purpose
Capture applicant type (individual/company) and validate company registration and activity where applicable.

## Trigger
CC01 Security Authentication (ROP PKI) succeeded.

## Inputs
- Beneficiary type selection (Individual / Company)
- Company CR number (if Company)
- Requested vessel/unit type and intended activity (used to validate activity eligibility)

## Outputs
- Beneficiary profile confirmed
- CR validation result (valid/invalid) if Company
- Activity validation result (active/inactive) if Company
- Rejection notification in failure cases

## Swimlanes
- Applicant (Owner/Agent)
- MTCIT System (REG001 Orchestrator)
- Invest Easy (Oman Business Platform)

## Steps (Sequenced)
| # | Step | Swimlane | Event Type (Optional) |
|---|------|----------|----------------------|
| 1 | Select beneficiary type (Individual / Company) | Applicant |  |
| 2 | G02 – Beneficiary type? | MTCIT System |  |
| 3 | If Individual: continue to SP03 | MTCIT System |  |
| 4 | If Company: submit company details (CR) | Applicant |  |
| 5 | Send CR validation request | MTCIT System → Invest Easy | Message (send) |
| 6 | Receive CR validation result | Invest Easy → MTCIT System | Message (receive) (+Timer boundary) |
| 7 | G03 – CR valid? | MTCIT System |  |
| 8 | If invalid: send rejection notification and terminate | MTCIT System | Message (send) + Error |
| 9 | If valid: send activity validation request (aligned to vessel type/activity) | MTCIT System → Invest Easy | Message (send) |
| 10 | Receive activity validation result | Invest Easy → MTCIT System | Message (receive) (+Timer boundary) |
| 11 | G04 – Activity active/eligible? | MTCIT System |  |
| 12 | If inactive/not eligible: reject + notify + terminate | MTCIT System | Message (send) + Error |
| 13 | If active: continue to SP03 | MTCIT System |  |

## Gateways
- **G02 – Beneficiary type?**
  - Individual → skip validations
  - Company → CR + activity validations

- **G03 – CR valid?**
  - Valid → proceed
  - Invalid → reject + notify

- **G04 – Activity active/eligible?**
  - Active/eligible → proceed
  - Inactive/not eligible → reject + notify

## Notes
- The company path is a hard gate: the request cannot proceed with invalid CR/activity.
