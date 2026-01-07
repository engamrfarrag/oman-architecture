```markdown
# FR-INT-05
Title: Integrate with inspection/classification services for technical inspection outcomes

Description:
When inspection is required, the system shall integrate with the inspection/classification service(s) to submit an inspection request and receive the inspection outcome.

Trigger:
Inspection required decision = Yes (SP06).

Outbound Data (to Inspection Service):
- Request reference
- Vessel/unit details
- Required inspection type/context

Inbound Data (from Inspection Service):
- Inspection outcome (Passed/Failed)
- Report reference and/or attached report

Processing / Rules:
- Create inspection request.
- Correlate the response to the originating request.
- Allow progression only for Passed outcomes; otherwise reject and notify.

Outputs:
- Inspection outcome consumed by SP06.

Failure Handling:
- Timeout/unavailable: place request in pending state or fail per policy; notify applicant where applicable.

Related Capabilities:
- CAP-15, CAP-23, CAP-26

Related FRs:
- FR-SP06-002, FR-SP06-003
```
