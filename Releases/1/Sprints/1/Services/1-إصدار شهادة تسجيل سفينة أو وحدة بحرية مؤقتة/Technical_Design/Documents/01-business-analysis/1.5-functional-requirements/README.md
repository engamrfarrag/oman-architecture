# REG001 – Functional Requirements (FR)

## Purpose
This folder defines the **Functional Requirements (FR)** package for:
- **Service Code:** REG001 (MTCIT-REG-001)
- **Service Name (EN):** Issuing Temporary Registration Certificate for a Ship or Marine Unit
- **Service Name (AR):** إصدار شهادة تسجيل سفينة أو وحدة بحرية مؤقتة

It is optimized using the workspace guidance in `FR_instructure.md`:
- User Stories = business intent
- Use Cases = system interaction flows
- Functional Requirements = atomic, testable system behavior

## Primary Sources
- **BPMN subprocess specifications (SP02…SP10):**
  - `01-business-analysis/1.3-process/03-BPMN_Process_Design/`
- **Capabilities (CAP-xx) mapping:**
  - `01-business-analysis/1.4-capabilities/01-Capability_to_Logical_Service_Mapping/`
- **Decision rules (DMN):**
  - `Technical_Design/Resources/dmn/reg001/`
- **Converted service document (DOCX → Markdown):** kept in working context during generation.

## FR ID Standard (Government-grade)
FR IDs remain stable and are referenced across User Stories → Use Cases → FR → Acceptance Criteria.

In this package, **each subprocess (SPxx) is maintained as a single file** (e.g., `03-functional-requirements/FR-SP02.md`) that contains multiple FR IDs (e.g., `FR-SP02-001`, `FR-SP02-002`, ...).

Each FR file uses:

- **`FR-CCxx-###`** for cross-cutting controls (e.g., security, notifications)
- **`FR-SPxx-###`** for business subprocess functional requirements, aligned to BPMN SP documents

Each FR contains: Title, Description, Trigger, Inputs, Processing/Rules, Outputs, Success Criteria, Failure Conditions, Related Gateways, Related Capabilities.

## Folder Structure
```text
1.5-functional-requirements/
├── README.md
├── 00-overview/
│   ├── FR-00-Overview.md
│   ├── Actors-and-Roles.md
│   └── FR-Coverage-Matrix.md
├── 01-user-stories/
├── 02-use-cases/
├── 03-functional-requirements/
├── 04-cross-cutting/
├── 05-integrations/
└── 06-acceptance-criteria/
    └── Acceptance-Criteria.md
```

## Reading Guide
- **Start here:** `00-overview/FR-00-Overview.md`
- **Business intent:** `01-user-stories/`
- **System behavior:** `02-use-cases/`
- **Buildable requirements (one file per SP):** `03-functional-requirements/FR-SPxx.md`
- **Shared platform controls:** `04-cross-cutting/`
- **External dependencies:** `05-integrations/`
- **Test coverage anchor:** `06-acceptance-criteria/Acceptance-Criteria.md`

## Data Dictionary
- A reusable template is provided in: `00-overview/Data-Dictionary-Template.md`
- Each `FR-SPxx.md` includes a SP-specific Data Dictionary section aligned to this template.
