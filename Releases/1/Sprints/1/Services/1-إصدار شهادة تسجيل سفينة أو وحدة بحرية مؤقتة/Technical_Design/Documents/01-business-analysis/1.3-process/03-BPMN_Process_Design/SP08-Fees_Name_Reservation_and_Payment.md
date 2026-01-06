# SP08 – Fees, Name Reservation, and Payment

## Purpose
Calculate fees/penalties, reserve the vessel name (temporary hold), generate invoice, and complete payment.

## Trigger
All required validations and conditional approvals (SP05/SP06/SP07) are satisfied.

## Inputs
- Vessel/unit details affecting fees (type, length, tonnage, activity)
- Penalty line items (if any)
- Name selection by applicant

## Outputs
- Invoice issued
- Payment result (Success/Failed/Cancelled/Expired)
- Receipt reference (on success)
- Name reservation released on failure/expiry

## Swimlanes
- Applicant (Owner/Agent)
- MTCIT System (REG001 Orchestrator)
- Payment Gateway
- (Optional) Name Reservation capability (modeled under System unless confirmed external)

## Steps (Sequenced)
| # | Step | Swimlane | Event Type (Optional) |
|---|------|----------|----------------------|
| 1 | Calculate base fees + penalties (config/DMN) | MTCIT System |  |
| 2 | Present fee summary and allow name selection | Applicant |  |
| 3 | Reserve vessel name for a limited time window | MTCIT System | Timer boundary |
| 4 | Generate invoice and send invoice notification | MTCIT System → Payment Gateway | Message (send) |
| 5 | Applicant confirms and proceeds to payment | Applicant |  |
| 6 | Execute payment via payment gateway | Applicant + Payment Gateway | Timer boundary |
| 7 | Receive payment result/callback | Payment Gateway → MTCIT System | Message (receive) |
| 8 | G15 – Payment successful? | MTCIT System |  |
| 9 | If failed/cancelled/expired: release name reservation; notify applicant; terminate or allow retry per policy | MTCIT System | Message (send) + Error |
| 10 | If successful: store receipt and continue to certificate issuance (SP09) | MTCIT System |  |

## Gateways
- **G15 – Payment successful?**
  - Yes → proceed to issuance
  - No → stop/retry per policy; release name hold

## Notes
- The workspace includes a reusable “common payment subprocess”; model SP08 as a call activity to that subprocess.
