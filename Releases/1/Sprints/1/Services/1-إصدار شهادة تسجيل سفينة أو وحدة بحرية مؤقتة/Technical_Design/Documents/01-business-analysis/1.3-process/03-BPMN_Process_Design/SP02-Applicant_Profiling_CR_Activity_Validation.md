# SP02 – Applicant Profiling + CR/Activity Validation

## Functional Responsibility
This subprocess is responsible for:
- Capturing applicant type (individual/company) and required identifiers
- Validating company Commercial Registration (CR) where applicable
- Validating company activity eligibility aligned to vessel/unit type
- Enforcing hard-stop rejection and notifications on failed eligibility checks

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

### G02 – Beneficiary type?
- Individual → skip validations
- Company → CR + activity validations

### Decision Rule (Business)
- Inputs: beneficiary type selection
- Rule Source (DMN / Config): Config/UI rules
- Outcomes:
  - Individual → proceed to SP03
  - Company → perform CR + activity validations

### G03 – CR valid?
- Valid → proceed
- Invalid → reject + notify

### Decision Rule (Business)
- Inputs: CR number, company identifiers (as provided)
- Rule Source (DMN / Config): Invest Easy (Oman Business Platform) integration response
- Outcomes:
  - Valid → continue
  - Invalid → reject + notify (include reason if provided)

### G04 – Activity active/eligible?
- Active/eligible → proceed
- Inactive/not eligible → reject + notify

### Decision Rule (Business)
- Inputs: CR number, intended activity, vessel/unit type/category
- Rule Source (DMN / Config): Invest Easy activity status + eligibility mapping (config/policy)
- Outcomes:
  - Active/eligible → proceed to SP03
  - Inactive/not eligible → reject + notify

## Notes
- The company path is a hard gate: the request cannot proceed with invalid CR/activity.
