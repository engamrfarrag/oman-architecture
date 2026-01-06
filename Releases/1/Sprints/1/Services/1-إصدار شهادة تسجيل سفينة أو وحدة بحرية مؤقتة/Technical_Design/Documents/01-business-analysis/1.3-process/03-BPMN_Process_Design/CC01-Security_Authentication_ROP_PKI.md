# CC01 – Security Authentication (ROP PKI) – Cross-Cutting

## Classification
**Cross-cutting concern (Security).**

This is a required platform control that **gates access** to the REG001 business process, but it is not considered part of the “business process result”.

## Purpose
Authenticate the applicant identity using ROP PKI so the applicant can start/continue the request.

## Trigger
Applicant starts REG001 request OR attempts to continue a saved draft where authentication is required.

## Inputs
- Applicant identity number / digital identity attributes (per PKI integration)

## Outputs
- Authentication status: Authenticated / Rejected
- Notification in case of rejection

## Swimlanes
- Applicant (Owner/Agent)
- MTCIT System (Platform Security/REG001 Orchestrator)
- ROP PKI

## Steps (Sequenced)
| # | Step | Swimlane | Event Type (Optional) |
|---|------|----------|----------------------|
| 1 | Initiate authentication (PKI request) | MTCIT System → ROP PKI | Message (send) |
| 2 | Receive PKI response | ROP PKI → MTCIT System | Message (receive) (+Timer boundary) |
| 3 | If authenticated: allow entry/continuation into business subprocesses (SP02+) | MTCIT System |  |
| 4 | If rejected: stop request and notify applicant with reason (if provided) | MTCIT System | Message (send) + Error |

## Notes
- Do **not** include this in the business gateway catalog (G02..G16). Treat it as a security gate.
- In BPMN, model as a pre-condition task/message exchange before the business process (or as an event subprocess if re-authentication is required).
