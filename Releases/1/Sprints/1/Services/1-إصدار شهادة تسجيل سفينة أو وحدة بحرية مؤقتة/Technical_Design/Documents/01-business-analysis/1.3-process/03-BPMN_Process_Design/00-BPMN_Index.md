# REG001 – BPMN Process Design (Index)

## Purpose
This folder contains BPMN-ready process specifications for REG001. Each sub-process is documented in its own Markdown file, including:
- Steps (sequenced)
- Swimlane mapping
- Gateways (decision logic)
- External dependencies/events

## Files
- **CC01** Security Authentication (ROP PKI) (Cross-cutting)
- **SP02** Applicant Profiling + CR/Activity Validation
- **SP03** Vessel Data Capture & Policy Validation
- **SP04** Dynamic Documents Collection
- **SP05** MAFWR Approval (Fishing Vessels)
- **SP06** Vessel Inspection
- **SP07** Bayan Customs Clearance
- **SP08** Fees, Name Reservation, and Payment
- **SP09** Certificate Issuance & Notifications
- **SP10** Post-Issuance Reminders & Penalties

## Modeling Notes (BPMN)
- Use a **main orchestration process** that calls SP05–SP08 conditionally.
- Model external responses as **message intermediate catch events** (or event subprocesses) with **timeout boundary events**.
- Use clear **business** gateway IDs that match the subprocess documents (G02..G16).
- Security gates (PKI) are documented under **CC01** and are not part of the business gateway catalog.

## Annotation-Level BPMN Event Types (Optional)
To make event handling explicit (without overloading the diagrams), each SP document may tag steps with an **Event Type (Optional)** label.

**Event Types:**
- **Message**: message sent to / received from an external participant (including applicant notifications)
- **Message (send)**: request/notification sent (message throw)
- **Message (receive)**: external response/callback received (message catch)
- **Timer boundary**: timeout/expiry while waiting (e.g., SLA breach, session expiry, name-hold expiry)
- **Error**: hard failure path that stops/terminates the request (business rejection or technical hard-stop)

**How to apply (documentation-level only):**
- In the SP “Steps (Sequenced)” table, populate **Event Type (Optional)** for relevant rows.
- If a wait has both an external response and a potential timeout, use a combined tag (e.g., **Message (receive) (+Timer boundary)**).

