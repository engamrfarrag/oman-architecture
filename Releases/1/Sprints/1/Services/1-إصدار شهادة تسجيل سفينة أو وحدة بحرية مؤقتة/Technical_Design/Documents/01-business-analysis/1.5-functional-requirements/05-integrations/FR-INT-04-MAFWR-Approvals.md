```markdown
# FR-INT-04
Title: Integrate with MAFWR for fishing vessel approval

Description:
When MAFWR approval is required (e.g., fishing vessel/activity), the system shall send an approval request to MAFWR and process the approval/rejection response.

Trigger:
MAFWR approval required decision = Yes (SP05).

Outbound Data (to MAFWR):
- Request reference
- Vessel/unit details
- Applicant/owner details (as allowed by contract)

Inbound Data (from MAFWR):
- Decision (Approved/Rejected)
- Rejection reason (if rejected)

Processing / Rules:
- Submit approval request and track status.
- On rejection, terminate the request and notify applicant with reason.
- Persist decision for audit.

Outputs:
- MAFWR decision consumed by SP05.

Failure Handling:
- Timeout/unavailable: place request in pending state or fail per policy; notify applicant where applicable.

Related Capabilities:
- CAP-13, CAP-23, CAP-26

Related FRs:
- FR-SP05-002, FR-SP05-003
```
