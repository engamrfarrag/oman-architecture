# SP09 – Certificate Issuance & Notifications

## Functional Responsibility
This subprocess is responsible for:
- Verifying the request is eligible for issuance after successful payment
- Generating the temporary registration certificate (PDF + QR)
- Persisting certificate metadata and linking it to the request
- Notifying the applicant and providing certificate access

## Purpose
Issue the temporary registration certificate after successful payment and notify the applicant.

## Trigger
Payment successful (SP08 completed).

## Inputs
- Paid invoice/receipt reference
- Verified request data (applicant + vessel/unit)

## Outputs
- Temporary Registration Certificate (PDF + QR)
- Status update and issuance notification

## Swimlanes
- MTCIT System (REG001 Orchestrator)
- Applicant (Owner/Agent)

## Steps (Sequenced)
| # | Step | Swimlane | Event Type (Optional) |
|---|------|----------|----------------------|
| 1 | Confirm payment is successful and request is eligible for issuance | MTCIT System | Error (if eligibility fails) |
| 2 | Generate temporary certificate (PDF + QR) | MTCIT System |  |
| 3 | Persist certificate metadata and link to request | MTCIT System |  |
| 4 | Send “certificate issued” notification to applicant | MTCIT System | Message (send) |
| 5 | Provide certificate download/access | Applicant |  |

## Gateways
- (Optional) **Pre-issuance eligibility check**
  - If any mandatory dependency is missing (should not happen in normal flow), stop issuance and notify operations.

### Decision Rule (Business)
- Inputs: payment success state, completion state of required subprocesses, certificate generation prerequisites
- Rule Source (DMN / Config): Orchestrator state/guards (system policy)
- Outcomes:
  - Eligible → issue certificate
  - Not eligible → stop issuance; notify operations/applicant as applicable

## Notes
- Notifications should include: certificate reference, validity period (config), and next-step guidance to complete permanent registration.
