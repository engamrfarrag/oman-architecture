```markdown
# FR-INT-06
Title: Integrate with Bayan Customs for import document and final clearance

Description:
When customs clearance is required, the system shall integrate with Bayan to submit the customs request, receive the import document (PDF) and final customs clearance status.

Trigger:
Customs required decision = Yes (SP07).

Outbound Data (to Bayan):
- Request reference
- Applicant identity/CR (as applicable)
- Vessel/unit and import details (as required)

Inbound Data (from Bayan):
- Import document (PDF)
- Final clearance status (Cleared/Not cleared)
- Optional reason/details

Processing / Rules:
- Submit customs request and track status.
- Persist received documents into the request document store.
- If clearance is not granted, reject the request and notify applicant.

Outputs:
- Bayan document(s) stored and linked
- Clearance decision consumed by SP07

Failure Handling:
- Timeout/unavailable: place request in pending state or fail per policy; notify applicant where applicable.

Related Capabilities:
- CAP-17, CAP-23, CAP-26

Related FRs:
- FR-SP07-002, FR-SP07-003
```
