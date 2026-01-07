# Process Design Instructions

> **Role:** Business Analyst / Process Designer  
> **Scope:** Process Design, BPMN Modeling, Subprocess Documentation  
> **Applies To:** `01-business-analysis/1.3-process/` folder

---

## 1. Goal

Divide the business process into clear **processes**, **sub-processes**, and **swimlanes** based on service documentation.

---

## 2. Document Analysis Workflow

### Step 1: Analyze Source Documents

1. Locate the service DOCX file (e.g., `إصدار شهادة تسجيل سفينة أو وحدة بحرية مؤقتة`)
2. Convert to Markdown using MarkItDown MCP tool:
   ```
   Tool: mcp_microsoft_mar_convert_to_markdown
   Input: file:///{absolute_path_to_docx}
   ```
3. Identify:
   - Business steps
   - Actors
   - Decisions (gateways)
   - Dependencies

### Step 2: Context Completion

1. Detect additional required documents:
   - Regulations
   - Prerequisites
   - Related services
2. State assumptions if documents are missing

### Step 3: Process Design

1. Recommend end-to-end business process
2. Decompose into:
   - Main processes
   - Sub-processes
3. Assign to appropriate swimlanes

---

## 3. Swimlane Definitions

| Swimlane | Actor | Responsibilities |
|----------|-------|------------------|
| **Applicant** | External user | Initiate requests, provide data, make payments |
| **System** | Platform/Automation | Validate, calculate, orchestrate, notify |
| **Authority** | Internal staff | Review, approve, issue certificates |
| **External Entity** | Third-party | Bayan, Banks, Customs, Ministry systems |

---

## 4. Process Hierarchy

```text
Main Process (MP-{ServiceCode})
│
├── SP01 – {Subprocess Name}
├── SP02 – {Subprocess Name}
├── SP03 – {Subprocess Name}
├── ...
├── CC01 – {Cross-Cutting: Notifications}
├── CC02 – {Cross-Cutting: Audit}
└── End Events
```

---

## 5. Subprocess Document Template

```md
# SP{XX} – {Subprocess Name}

## Purpose
Brief description of what this subprocess accomplishes.

## Functional Responsibility
This subprocess is responsible for:
- {Responsibility 1}
- {Responsibility 2}
- {Responsibility 3}

## Swimlane
**Primary Actor:** {Applicant | System | Authority | External Entity}

## Trigger
- {What initiates this subprocess}

## Steps

| Step | Action | Actor | Description |
|------|--------|-------|-------------|
| 1 | {Action} | {Actor} | {Description} |
| 2 | {Action} | {Actor} | {Description} |
| 3 | {Action} | {Actor} | {Description} |

## Gateways

### G{XX} – {Gateway Name}

**Type:** {Exclusive | Parallel | Inclusive}

#### Decision Rule (Business)
- **Inputs:** {field1, field2, ...}
- **Rule Source:** {DMN file | Config | Business Logic}
- **Outcomes:**
  - {Outcome 1} → {Next step/subprocess}
  - {Outcome 2} → {Next step/subprocess}

## Outputs
- {Output 1}
- {Output 2}

## Dependencies
- **Depends On:** SP{XX}
- **Triggers:** SP{XX}

## Related Artifacts
- **Capabilities:** CAP-{XX}
- **DMN Rules:** {service_code}-{rule_name}.dmn
- **Events:** {EventName}

## Error Handling
- {Error scenario} → {Handling action}
```

---

## 6. Gateway Types and Usage

| Gateway Type | Symbol | When to Use |
|--------------|--------|-------------|
| **Exclusive (XOR)** | ◇ | One path only based on condition |
| **Parallel (AND)** | ◆ | All paths execute simultaneously |
| **Inclusive (OR)** | ⬡ | One or more paths based on conditions |
| **Event-Based** | ⬢ | Wait for external event |

---

## 7. Decision Rule Documentation

Every gateway MUST have a decision rule:

```md
### Decision Rule (Business)
- **Inputs:** {Required data fields}
- **Rule Source:** {DMN file path or "Config" or "Policy"}
- **Outcomes:**
  - {Condition 1} → {Action/Next SP}
  - {Condition 2} → {Action/Next SP}
  - {Default} → {Fallback action}
```

**Example:**

```md
### G05 – Vessel Age Policy Check

#### Decision Rule (Business)
- **Inputs:** vessel type, build year, material
- **Rule Source:** reg001-vessel-age-validation.dmn
- **Outcomes:**
  - Pass → Continue to SP04
  - Fail → Block issuance; allow re-entry if policy permits
```

---

## 8. Cross-Cutting Processes

Document shared processes that apply across multiple services:

| CC ID | Name | Description |
|-------|------|-------------|
| CC01 | Notifications | SMS, Email, Portal alerts |
| CC02 | Audit Logging | System activity tracking |
| CC03 | Error Handling | Centralized error management |
| CC04 | Payment Processing | Fee calculation and collection |

---

## 9. Output Structure

```text
01-business-analysis/1.3-process/
│
├── 01-Service_Blueprint/
│   └── Service-Blueprint.md
│
├── 02-Business_Process_Overview/
│   └── Process-Overview.md
│
└── 03-BPMN_Process_Design/
    ├── MP-{ServiceCode}-Main-Process.md
    ├── SP01-{SubprocessName}.md
    ├── SP02-{SubprocessName}.md
    ├── ...
    ├── CC01-Notifications.md
    └── CC02-Audit.md
```

---

## 10. Expected Output Quality

A well-structured business process model must be directly usable for:

✅ Swimlane diagrams  
✅ BPMN workflows  
✅ Service digitalization  
✅ Automation initiatives  
✅ Capability extraction  
✅ Functional Requirements derivation  

---

## 11. Quality Checklist

Before completing process design:

- [ ] All subprocesses have Functional Responsibility sections
- [ ] All gateways have Decision Rules documented
- [ ] Swimlane assignments are correct for each step
- [ ] Process flow is complete (no gaps or orphan steps)
- [ ] Dependencies between subprocesses are documented
- [ ] DMN rule references are included where applicable
- [ ] Cross-cutting processes are identified

---

## 12. Prompt Patterns

**Recognized Prompts:**
- "Design the business process for {service_name}"
- "Create BPMN subprocess for {subprocess_name}"
- "Document SP{XX} for {service_code}"
- "Generate service blueprint for {service_name}"
- "Map swimlanes for {process_name}"

---

## 13. Dependencies

**Required Before Process Design:**
1. ✅ Scope definition complete
2. ✅ Stakeholders identified
3. ✅ Source service documentation available

**Enables After Process Design:**
1. → Capability mapping
2. → Functional Requirements
3. → Solution Architecture

---

## 14. Related Instruction Files

- [BA-instructions.md](BA-instructions.md) – Overall BA workflow
- [FR-instructions.md](FR-instructions.md) – How process feeds into FR
- [SA-instructions.md](SA-instructions.md) – How process maps to services

---

**Last Updated:** 2026-01-07  
**Applies To:** All Process Design tasks in the BA workspace
