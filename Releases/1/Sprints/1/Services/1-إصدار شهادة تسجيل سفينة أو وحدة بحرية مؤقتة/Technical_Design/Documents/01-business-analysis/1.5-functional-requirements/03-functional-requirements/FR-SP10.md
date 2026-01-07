# SP10 – Post-Issuance Reminders & Overdue Penalties (Functional Requirements)

This file consolidates all SP10 functional requirements in a single document, while preserving the original FR IDs for traceability.

---

## FR-SP10-001
Title: Schedule post-issuance reminders for permanent registration completion

Description:
After issuing a temporary certificate, the system shall schedule and send recurring reminders to the applicant to complete permanent registration, based on configured cadence.

Trigger:
Temporary certificate issued (SP09 completed).

Inputs:
- Certificate issuance date/time
- Reminder cadence (config; e.g., every 7 days)
- Applicant contact details
- Request/certificate reference

Processing / Rules:
- Start monitoring timers after issuance.
- At each interval, send a reminder notification to the applicant.
- Reminders must include certificate reference and next-step guidance.

Outputs:
- Reminder notifications (recurring)
- Monitoring schedule/state persisted

Success Criteria:
- Reminders are sent at the configured cadence until monitoring is stopped.

Failure Conditions:
- Scheduler unavailable
- Notification provider unavailable

Related Gateways:
- G16

Related Capabilities:
- CAP-24, CAP-23, CAP-26

---

## FR-SP10-002
Title: Stop reminders when permanent registration is completed

Description:
The system shall stop post-issuance reminders and close monitoring once permanent registration has been completed for the issued temporary certificate.

Trigger:
A permanent registration completion signal is received (event/API), or the monitoring cycle checks completion status.

Inputs:
- Request/certificate reference
- Permanent registration completion status (Yes/No)

Processing / Rules:
- Evaluate completion status at gateway G16.
- If completed, stop future reminders, close monitoring state, and record closure reason.

Outputs:
- Monitoring closed
- No further reminders sent

Success Criteria:
- Reminders stop immediately after completion is confirmed.

Failure Conditions:
- Permanent registration status cannot be determined (timeout/unavailable)

Related Gateways:
- G16

Related Capabilities:
- CAP-24, CAP-26

---

## FR-SP10-003
Title: Evaluate overdue thresholds and apply post-issuance penalties

Description:
If permanent registration is not completed within configured thresholds after temporary certificate issuance, the system shall evaluate and apply overdue penalties and notify the applicant.

Trigger:
Monitoring cycle runs and permanent registration is not completed (G16 = No).

Inputs:
- Certificate issuance date/time
- Permanent registration completion status
- Overdue threshold(s) and timing config (REG-001-CONF-02)
- Penalty rules (DMN: reg001-penalty-calculation.dmn) and/or config

Processing / Rules:
- Calculate elapsed time since issuance (or relevant configured reference date).
- If elapsed time exceeds configured overdue threshold, compute penalty per DMN/config.
- Create penalty record/line item linked to the request/certificate.
- Notify applicant of the penalty and required next steps.

Outputs:
- Penalty application record
- Applicant notification (penalty applied)

Success Criteria:
- Penalties are applied only when overdue conditions are met and are traceable to the rule source.

Failure Conditions:
- DMN/rules engine unavailable
- Penalty persistence failure

Related Gateways:
- G16

Related Capabilities:
- CAP-25, CAP-23, CAP-26

---

## Data Dictionary (SP10)
Aligned with the REG001 Data Dictionary template in `00-overview/Data-Dictionary-Template.md`.

| Data Element (Canonical) | Description | Source / Owner | Data Type | Format / Allowed Values | Validation / Rule Source | Required (Y/N) | Used By (FR IDs) | Example / Notes | Classification |
|---|---|---|---|---|---|---|---|---|---|
| reminderCadenceDays | Reminder cadence in days | LS-CONFIG/LS-MONITOR | integer | >0 | REG-001-CONF-02 | Y | FR-SP10-001 | e.g., 7 | Internal |
| monitoringState | Monitoring lifecycle state | LS-MONITOR | enum | ACTIVE, CLOSED | Monitoring policy | Y | FR-SP10-001, FR-SP10-002 |  | Internal |
| permanentRegistrationStatus | Whether permanent registration is completed | Downstream service / LS-MONITOR | enum/boolean | COMPLETED/NOT_COMPLETED | Gateway G16 | Y | FR-SP10-002, FR-SP10-003 | Source may be event/API | Internal |
| overdueThresholdDays | Threshold after which penalties apply | LS-CONFIG | integer | >=0 | REG-001-CONF-02 | Y | FR-SP10-003 |  | Internal |
| overduePenaltyAmount | Overdue penalty amount | LS-PENALTIES | number | >=0 | `reg001-penalty-calculation.dmn` | N | FR-SP10-003 | Stored as line item | Internal |
| lastReminderSentAt | Timestamp of last reminder | LS-MONITOR | datetime | ISO-8601 | Scheduler | N | FR-SP10-001 | Used for audit | Internal |
