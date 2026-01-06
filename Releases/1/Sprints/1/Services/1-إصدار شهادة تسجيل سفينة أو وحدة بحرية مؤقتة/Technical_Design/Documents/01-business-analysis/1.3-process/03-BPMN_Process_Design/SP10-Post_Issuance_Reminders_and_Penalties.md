# SP10 – Post-Issuance Reminders & Overdue Penalties (Monitoring)

## Purpose
After issuing the temporary certificate, send periodic reminders to complete permanent registration and apply penalties when overdue, based on configured timings.

## Trigger
Temporary certificate issued (SP09 completed).

## Inputs
- Certificate issuance date
- Certificate validity period (months) (config)
- Reminder cadence (e.g., every 7 days) (config)
- Overdue threshold and penalty rules (config/DMN)
- Permanent registration completion status (external/related service)

## Outputs
- Reminder notifications (recurring)
- Penalty application records + notifications (when overdue)
- Monitoring stops when permanent registration is completed

## Swimlanes
- MTCIT System (Scheduler/Orchestrator)
- Applicant (Owner/Agent)
- (Optional) Downstream Permanent Registration Service (status reference)

## Steps (Sequenced)
| # | Step | Swimlane | Event Type (Optional) |
|---|------|----------|----------------------|
| 1 | Start monitoring timer(s) after certificate issuance | MTCIT System | Timer boundary |
| 2 | Send reminder notification at configured interval | MTCIT System | Message (send) + Timer boundary |
| 3 | G16 – Permanent registration completed? | MTCIT System |  |
| 4 | If completed: stop reminders and close monitoring | MTCIT System |  |
| 5 | If not completed: continue reminders | MTCIT System |  |
| 6 | Evaluate overdue threshold and penalty rules | MTCIT System |  |
| 7 | If overdue: apply penalty and notify applicant | MTCIT System | Message (send) |

## Gateways
- **G16 – Permanent registration completed?**
  - Yes → stop monitoring
  - No → continue monitoring

## Notes
- This subprocess is typically modeled as a separate timer-driven process (or event subprocess) rather than a simple linear flow.
