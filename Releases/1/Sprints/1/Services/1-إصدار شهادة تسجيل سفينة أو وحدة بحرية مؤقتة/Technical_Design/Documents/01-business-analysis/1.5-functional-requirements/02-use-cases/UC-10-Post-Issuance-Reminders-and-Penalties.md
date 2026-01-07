# UC-10 – Post-Issuance Reminders and Overdue Penalties

## Primary Actor
MTCIT System (Scheduler/Orchestrator)

## Preconditions
- Temporary certificate issued.

## Main Flow
1. System starts monitoring timers after issuance.
2. System sends reminder notification at configured cadence (e.g., every 7 days).
3. System checks if permanent registration is completed.
4. If completed, system stops reminders.
5. If not completed, system evaluates overdue thresholds and applies penalties per policy.

## Alternate / Failure Flows
- **A1 – Downstream status unavailable**: System retries per policy and continues monitoring.

## Postconditions
- Reminders and penalties applied until permanent registration completion.

## Traceability
- User Story: US-10
- BPMN: SP10 (G16)
- Capabilities: CAP-24, CAP-25
- Functional Requirements: [FR-SP10.md](../03-functional-requirements/FR-SP10.md) (FR-SP10-001…FR-SP10-003)
