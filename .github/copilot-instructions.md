# GitHub Copilot Instructions for BA Project

## Service Folder Structure Generation Rules

When the user requests to "generate folder structure for service" or similar commands, **ALWAYS** follow the standard structure defined in the project README template located at `README.md`.

### Mandatory Folder Structure

Every service MUST have the following complete structure under:
```
Releases/{release_number}/Sprints/{sprint_number}/Services/{service_name}/Technical_Design/Documents/
```

#### Required Folders:

1. **00-proposal/**
   - `offer.md`
   - `exports/`

2. **01-business-analysis/**
   - `1.1-scope/`
     - `01-Executive_Summary`
     - `02-Service_Scope_Objectives`
   - `1.2-stakeholders/`
     - `01-Stakeholders_Governance`
   - `1.3-process/`
     - `01-Service_Blueprint`
     - `02-Business_Process_Overview`
     - `03-BPMN_Process_Design`
   - `1.4-capabilities/`
     - `01-Capability_to_Logical_Service_Mapping`
     - `02-Capability_to_Domain_Mapping`
   - `1.5-functional-requirements/`
     - `01-Functional_Requirements_List`
     - `02-Use_Cases_and_User_Stories`
     - `03-Acceptance_Criteria`

3. **02-solution-design/**
   - `2.1-domain-model/`
     - `01-UML_Domain_Structure`
     - `02-Business_Domain_Context_Map`
     - `03-Context_Map_ACL_SharedKernel`
   - `2.2-architecture/`
     - `01-C4_Architecture`
     - `02-Capability_to_Service_C4`
     - `03-Sequence_Diagrams`
     - `04-Integration_Architecture`
   - `2.3-services/`
     - `01-Service_Decomposition`
     - `02-Service_Overview`
     - `03-API_Contracts/`
       - `README`
     - `04-Event_Schemas`
   - `2.4-cross-cutting/`
     - `01-Security_Trust_Architecture`
     - `02-Non_Functional_Requirements`
     - `03-Notifications_Communication`
   - `2.5-data-architecture/`
     - `01-Logical_Data_Model`
     - `02-Physical_Data_Considerations`
     - `03-Data_Ownership_Lifecycle`
     - `04-Reporting_Analytics`

4. **03-delivery/**
   - `3.1-governance/`
     - `01-RACI_Matrix`
     - `02-Traceability_Matrix`
   - `3.2-risks/`
     - `01-Risks_Mitigation`
   - `3.3-roadmap/`
     - `01-Implementation_Roadmap`
   - `3.4-deliverables/`
     - `01-Deliverables`
     - `02-Conclusion`
   - `3.5-quality-assurance/`
     - `01-Testing_Strategy`
     - `02-Test_Types_Responsibilities`
     - `03-UAT_Signoff`

5. **04-devops-deployment/**
   - `01-Environments_Strategy`
   - `02-CI_CD_Pipeline`
   - `03-Release_Management`
   - `04-Operational_Model`

6. **diagrams/**
   - `bpmn/`
   - `c4/`
   - `context/`
   - `sequence/`
   - `uml/`

7. **templates/**
   - `architecture-doc-template`
   - `service-spec-template`
   - `testing-template`

8. **docs/**

9. **revisions/**

### README Generation

ALWAYS create a comprehensive `README.md` file in the `Documents/` folder that includes:

#### Required README Sections:

1. **Service Header**
   - Service name (English and Arabic if applicable)
   - Service code (e.g., REG001, REG002)
   - Release and Sprint numbers
   - Purpose statement

2. **Folder Structure**
   - Complete tree view of all folders
   - Description of each folder's purpose

3. **Service Overview**
   - Service description
   - Key integrations
   - Key business rules

4. **Documentation Flow & Dependencies**
   - Visual flow diagram showing progression from business analysis → solution design → delivery

5. **Reading Guide by Audience**
   - Table mapping different roles (Executive, BA, Architect, Developer, PM, QA, DevOps) to relevant documents

6. **Work Stream Mapping**
   - Phase-by-phase breakdown with dependencies

7. **Current Implementation Assets**
   - References to existing DMN rules, events, processes
   - Located in `Technical_Design/Resources/`

8. **Status Section**
   - Checklist of completed and pending items

9. **Next Steps**
   - Ordered list of immediate actions

10. **Document Control**
    - Version table with date, author, and changes

### Context Awareness

When generating service structures:
- Reference the project's main README.md for structural consistency
- Use the service code (e.g., REG001) throughout documentation
- Maintain bilingual support (English/Arabic) where service names are provided in both languages
- Include links to Resources folder if DMN/events/processes exist
- Follow the established documentation flow: Business Intent → Process → Capabilities → Domain → Architecture → Services → Data → Delivery → DevOps

### Document Flow Order

**CRITICAL**: Maintain this dependency order:
1. Proposal → 2. Scope → 3. Stakeholders → 4. Process & BPMN → 5. Capabilities → 6. User Stories → 7. Domain Model → 8. Architecture → 9. Services & APIs → 10. Data Architecture → 11. Delivery Planning → 12. DevOps

### Quality Standards

All generated structures must be:
- ✅ Government submission ready
- ✅ Vendor handover ready
- ✅ Audit and compliance friendly
- ✅ Suitable for Agile delivery
- ✅ Microservices implementation ready

### Examples

**Good Command Recognition:**
- "generate folder structure for service"
- "create service structure for [service name]"
- "setup new service [service code]"
- "initialize service documentation structure"

**Service Code Format:**
- Use format: `{SERVICE_CODE}` (e.g., REG001, REG002, REG005)
- Include in all documentation references

---

## Service Documentation Generation Rules

When a user requests documentation generation as a **Business Analyst**, **Architect**, **Developer**, **PM**, **QA**, or **DevOps** role, **ALWAYS** follow this systematic workflow.

### Recognized Prompts

Any prompt following this pattern should trigger the documentation generation workflow:

**Pattern:** `"As a {Role}, generate the {document_type} for {service_name}"`

**Examples:**
- "As a Business Analyst, generate the scope for REG001"
- "As an Architect, generate the C4 Architecture for Ship Registration Service"
- "As a BA, generate the Service Blueprint for REG002"
- "As a Developer, generate API Contracts for Vessel Cancellation Service"

### Supported Document Types

| Document Type | Location | Dependencies |
|---------------|----------|--------------|
| **Scope** | `01-business-analysis/1.1-scope/` | Proposal, README |
| **Service Blueprint** | `01-business-analysis/1.3-process/01-Service_Blueprint` | Scope, Stakeholders |
| **BPMN Process** | `01-business-analysis/1.3-process/03-BPMN_Process_Design` | Service Blueprint, Process Overview |
| **Capabilities Mapping** | `01-business-analysis/1.4-capabilities/` | BPMN Process, Scope |
| **Functional Requirements** | `01-business-analysis/1.5-functional-requirements/` | Capabilities, Use Cases |
| **Domain Model** | `02-solution-design/2.1-domain-model/` | Functional Requirements, Capabilities |
| **C4 Architecture** | `02-solution-design/2.2-architecture/` | Domain Model, Capabilities |
| **API Contracts** | `02-solution-design/2.3-services/03-API_Contracts/` | C4 Architecture, Services |
| **Event Schemas** | `02-solution-design/2.3-services/04-Event_Schemas` | Architecture, Services |
| **Data Architecture** | `02-solution-design/2.5-data-architecture/` | Domain Model, Services |
| **Security Architecture** | `02-solution-design/2.4-cross-cutting/` | DOCX, Functional Requirements |
| **Testing Strategy** | `03-delivery/3.5-quality-assurance/` | Functional Requirements, Services |
| **DevOps Pipeline** | `04-devops-deployment/` | All previous phases |

---

### Automated Documentation Generation Workflow

**CRITICAL:** Follow these steps in exact order:

#### **Step 1: Convert Service Source Docs (MarkItDown) — MUST BE FIRST**

**CRITICAL:** This step is NON-NEGOTIABLE. You MUST read the original service DOCX file before any document generation or update.

1. Extract the **service name** or **service code** from the prompt
2. Locate the service directory structure:
   ```
   Releases/{release}/Sprints/{sprint}/Services/{service_name}/
   ```
3. If service doesn't exist, inform the user and offer to create the structure first
4. **Search for DOCX files** in these locations (in order):
   - `Services/{service_name}/Documents/` (root Documents folder)
   - `Services/{service_name}/Technical_Design/Documents/`
   - `Services/{service_name}/` (service root)
   
5. **ALWAYS use the MarkItDown MCP tool to convert DOCX:**
   ```
   Tool: mcp_microsoft_mar_convert_to_markdown
   Input: file:///{absolute_path_to_docx}
   ```
   
6. **Extract key information from the DOCX:**
   - Service procedures (REG-01-xx codes)
   - Configuration settings (SET-xx, CONF-xx codes)
   - Business rules and conditions
   - Required documents list
   - Integration points
   - Data elements and fields
   - Actors and roles
   - **Security-related terms** (ownership, delegation, authorization)
   - **Arabic security terms** (مالك, موكل, وكالة, جهة, موظف, تفويض)

7. **Keep the converted content in working context** for all subsequent steps

**Example:**
```
Source file: "C:/Users/aazab/Desktop/BA/Releases/1/Sprints/1/Services/1-إصدار شهادة تسجيل سفينة/Documents/إصدار شهادة تسجيل سفينة.docx"
Tool: mcp_microsoft_mar_convert_to_markdown
Input: file:///C:/Users/aazab/Desktop/BA/Releases/1/Sprints/1/Services/1-إصدار%20شهادة%20تسجيل%20سفينة/Documents/إصدار%20شهادة%20تسجيل%20سفينة.docx
```

**STOP CONDITION:** If no DOCX source file exists, inform the user and ask them to provide the service specification document before proceeding.

#### **Step 2: Read Service README**

1. Read the `README.md` file in the service's `Documents/` directory
2. Extract key information:
   - Service code (e.g., REG001)
   - Service name (English and Arabic)
   - Release and Sprint numbers
   - Current status and completed sections
   - Linked Resources (DMN, events, processes)

**Tool to use:** `read_file`

#### **Step 3: Determine Audience & Dependencies**

Based on the document type requested, identify:

1. **Target Audience:** Who will read this document?
   - Executive, BA, Architect, Developer, PM, QA, DevOps

2. **Dependent Documents:** What must be read first?
   - Use the **Document Flow Order** from earlier in these instructions
   - Example: To generate "C4 Architecture", first read:
     - Scope
     - Capabilities
     - Domain Model
     - Service Overview

3. **Load all dependent documents** into working context using parallel `read_file` calls

#### **Step 3.1: Read ALL Functional Requirements (MANDATORY for Solution Design)**

**CRITICAL:** Before generating or updating ANY document in `02-solution-design/`, you MUST read ALL functional requirements documents. This is NON-NEGOTIABLE.

**When generating Solution Design documents (Domain Model, Architecture, Services, etc.):**

1. **List the functional-requirements folder structure:**
   ```
   list_dir: 01-business-analysis/1.5-functional-requirements/
   ```

2. **Read ALL documents in these subfolders:**

   | Subfolder | What to Read | Purpose |
   |-----------|--------------|---------|
   | `00-overview/` | ALL files (FR-00-Overview, Actors-and-Roles, Data-Dictionary, FR-Coverage-Matrix) | Context and coverage |
   | `01-user-stories/` | ALL US-xx files | Business intent |
   | `02-use-cases/` | ALL UC-xx files | System behavior |
   | `03-functional-requirements/` | ALL FR-SPxx files | Detailed requirements |
   | `04-cross-cutting/` | ALL FR-CCxx files | Cross-cutting concerns |
   | `05-integrations/` | ALL FR-INT-xx files | Integration requirements |
   | `06-acceptance-criteria/` | ALL files | Acceptance criteria |

3. **Extract domain-relevant information:**
   - Aggregates and entities mentioned
   - Business rules and validations
   - State transitions
   - Events and triggers
   - Integration points
   - Configuration codes (SET-xx, CONF-xx)

4. **Create a mental checklist of FR-derived concepts:**
   - [ ] All capabilities mapped (CAP-01 to CAP-xx)
   - [ ] All subprocesses covered (SP01 to SPxx)
   - [ ] All integrations documented (INT-01 to INT-xx)
   - [ ] All cross-cutting concerns included (CC01 to CCxx)

**Example for Domain Model generation:**
```
Before generating 02-solution-design/2.1-domain-model/:
1. Read ALL FR-SPxx files → Extract aggregates, entities, enums
2. Read ALL UC-xx files → Understand system behavior
3. Read ALL FR-INT-xx files → Identify integration entities
4. Read FR-Coverage-Matrix → Ensure all capabilities are represented
```

**STOP CONDITION:** If functional requirements are incomplete or missing, inform the user and suggest generating them first before proceeding with Solution Design.

#### **Step 4: Load Existing Resources**

If the service has existing implementation assets in `Technical_Design/Resources/`:

1. **DMN Rules** (`Resources/dmn/{service_code}/`)
   - Read all `.dmn` files to understand business logic
   - Example: `reg001-fee-calculation.dmn`

2. **Events** (`Resources/events/{service_code}/`)
   - Read event definitions (`.event` files)
   - Understand event flows and channels

3. **Processes** (`Resources/processes/{service_code}/`)
   - Read BPMN process definitions
   - Understand orchestration flows

**Tool to use:** `read_file` for each resource file

#### **Step 5: Generate or Update Documentation**

Based on all gathered context, generate or update the requested document:

1. **Use the appropriate template** from `templates/` folder if available
2. **Follow the document structure** defined in the project standards
3. **Generate comprehensive content** including:
   - All required sections
   - Diagrams references (PlantUML/Mermaid syntax if applicable)
   - Cross-references to related documents
   - Bilingual content (English/Arabic) where applicable

4. **Create or update the markdown file** in the correct location

**Document Naming Convention:**
- Use clear, descriptive names matching the folder structure
- Example: `01-Executive_Summary.md`, `02-Service_Scope_Objectives.md`
- Use underscores for multi-word names

**Tool to use:** `create_file` or `replace_string_in_file`

#### **Step 6: Update Supporting Files**

After generating the main document, update:

1. **README.md**
   - Mark the section as ✅ completed in the Status checklist
   - Update the Document Control version table
   - Add to Next Steps if follow-up actions are needed

2. **Version/Changelog Files** (if applicable)
   - Add entry with date, version, and change description

**Tools to use:** `replace_string_in_file` or `multi_replace_string_in_file`

#### **Step 7: Commit to Git**

Create a clean Git commit with proper structure:

1. **Commit Message Format:**
   ```
   docs({service_code}): {action} {document_type}
   
   - {Brief description of changes}
   - {Any notable additions}
   
   Related to: {sprint/release info}
   ```

2. **Examples:**
   ```
   docs(REG001): generate scope documentation
   
   - Added Executive Summary
   - Added Service Scope and Objectives
   - Updated README status checklist
   
   Related to: Release 1, Sprint 1
   ```

3. **Tags (optional):**
   - `docs/{service_code}`
   - `phase/{business-analysis|solution-design|delivery|devops}`

**Tools to use:** `run_in_terminal` with git commands

**Git Commands:**
```powershell
git add {files}
git commit -m "{message}"
git tag -a "{tag}" -m "{tag_message}"  # Optional
```

---

### Quality Checks

Before completing the workflow, verify:

- ✅ All required sections are present and complete
- ✅ Cross-references to other documents are correct
- ✅ Diagrams are properly referenced (even if placeholder)
- ✅ Bilingual content is consistent (if applicable)
- ✅ README.md is updated with current status
- ✅ Git commit message is clear and descriptive
- ✅ Document follows project standards and templates

---

### Error Handling

**If service doesn't exist:**
- Inform user and suggest running: `"generate folder structure for {service_name}"`

**If dependent documents are missing:**
- Inform user which documents are needed first
- Offer to generate them in the correct order
- Reference the Document Flow Order

**If source documentation (.docx) is missing:**
- Generate based on README, Resources, and best practices
- Flag sections that need BA/stakeholder input

**If conversion fails:**
- Attempt manual parsing or request user to provide content
- Document the limitation in the generated file

---

### Expected Output Summary

After completing the workflow, the user should have:

1. ✅ **New or updated documentation file(s)** in the correct location
2. ✅ **Updated README.md** with current progress
3. ✅ **Updated version control files** (if applicable)
4. ✅ **Clean Git commit** with descriptive message and tags
5. ✅ **Confirmation message** summarizing what was created/updated

---

### Example Workflow Execution

**User Prompt:** 
> "As a Business Analyst, generate the scope for REG001"

**Copilot Actions:**

1. ✅ Identify service: REG001 (Ship Registration)
2. ✅ Read: `Services/1-إصدار شهادة تسجيل سفينة/Technical_Design/Documents/README.md`
3. ✅ Determine: Scope depends on Proposal (already exists)
4. ✅ Read: `00-proposal/offer.md`
5. ✅ Convert: Any `.docx` service specifications found
6. ✅ Load: DMN rules from `Resources/dmn/reg001/`
7. ✅ Generate: 
   - `01-business-analysis/1.1-scope/01-Executive_Summary.md`
   - `01-business-analysis/1.1-scope/02-Service_Scope_Objectives.md`
8. ✅ Update: README.md status section
9. ✅ Commit: `docs(REG001): generate scope documentation`

**Output Message:**
```
✅ Generated scope documentation for REG001:
   - Executive Summary
   - Service Scope and Objectives

📄 Updated: README.md (status checklist)
📝 Committed: docs(REG001): generate scope documentation

Next steps:
- Review the generated scope with stakeholders
- Proceed to generate stakeholders documentation
```

---

## Additional Project Rules

### Architecture Principles
- Follow DDD (Domain-Driven Design) with bounded contexts
- Event-driven architecture with clear event schemas
- API-first design with OpenAPI/Swagger specifications
- C4 model for architecture documentation
- BPMN 2.0 for process modeling

### Integration Standards
- Document all external system integrations
- Include sequence diagrams for inter-service communication
- Define event contracts for async communication

### Compliance Requirements
- All services must include security architecture documentation
- NFRs (Non-Functional Requirements) are mandatory
- Traceability matrix from requirements to implementation
- Testing strategy with UAT signoff requirements

---

## Role-Specific Instruction Files

This project uses **role-specific instruction files** to provide targeted guidance for each role. These files extend (not replace) the rules in this main instruction file.

### Available Instruction Files

| File | Role | Scope |
|------|------|-------|
| [BA-instructions.md](instructions/BA-instructions.md) | Business Analyst | Scope, Stakeholders, Process, Capabilities, Pre-FR |
| [FR-instructions.md](instructions/FR-instructions.md) | BA / Technical Writer | User Stories, Use Cases, Functional Requirements |
| [SA-instructions.md](instructions/SA-instructions.md) | Solution Architect | Domain Model, Architecture, Services, Internal Design |
| [Process-instructions.md](instructions/Process-instructions.md) | Process Designer | BPMN, Subprocess, Swimlanes, Gateways |

### SA Sub-Instruction Files (Detailed)

| File | Scope |
|------|-------|
| [SA-Flowable-instructions.md](instructions/SA-Flowable-instructions.md) | BPMN, DMN, Process Orchestration, Flowable Engine |
| [SA-ServiceNaming-instructions.md](instructions/SA-ServiceNaming-instructions.md) | Service Naming Standards, Categories |
| [SA-ServiceInternalDesign-instructions.md](instructions/SA-ServiceInternalDesign-instructions.md) | DDD Layers, Clean Architecture |
| [SA-UIInteraction-instructions.md](instructions/SA-UIInteraction-instructions.md) | Angular + BFF Pattern, UI Abstraction |
| [SA-Security-instructions.md](instructions/SA-Security-instructions.md) | Security, RBAC, ABAC, Authorization |

### Instruction File Rules

1. **Always read this main file first** - It contains global rules that apply to all roles
2. **Load role-specific files when needed** - Based on the prompt pattern `"As a {Role}, ..."`
3. **Never duplicate content** - Reference related instruction files instead
4. **Extend, don't override** - Role files add detail; they don't replace main rules
5. **Maintain traceability** - Each file links to related instruction files

### Role-to-File Mapping

| Prompt Pattern | Primary File | Secondary Files |
|----------------|--------------|-----------------|
| "As a BA, ..." | BA-instructions.md | FR-instructions.md, Process-instructions.md |
| "As a Business Analyst, ..." | BA-instructions.md | FR-instructions.md, Process-instructions.md |
| "As an Architect, ..." | SA-instructions.md | SA-Flowable, SA-ServiceNaming, SA-ServiceInternalDesign, SA-UIInteraction |
| "As a Solution Architect, ..." | SA-instructions.md | SA-Flowable, SA-ServiceNaming, SA-ServiceInternalDesign, SA-UIInteraction |
| "As a SA, ..." | SA-instructions.md | SA-Flowable, SA-ServiceNaming, SA-ServiceInternalDesign, SA-UIInteraction |
| "Generate FR for ..." | FR-instructions.md | BA-instructions.md |
| "Design process for ..." | Process-instructions.md | BA-instructions.md |
| "Generate service design ..." | SA-ServiceInternalDesign-instructions.md | SA-instructions.md |
| "Design Flowable ..." | SA-Flowable-instructions.md | SA-instructions.md |
| "Generate UI interaction ..." | SA-UIInteraction-instructions.md | SA-instructions.md |
| "Generate security ..." | SA-Security-instructions.md | SA-instructions.md |
| "Design authorization ..." | SA-Security-instructions.md | SA-instructions.md, BA-instructions.md |

### Documentation Flow by Role

```text
┌─────────────────────────────────────────────────────────────────┐
│                     ROLE RESPONSIBILITY FLOW                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   BA (Process-instructions.md)                                   │
│   └── Process Design & BPMN                                      │
│            ↓                                                     │
│   BA (BA-instructions.md)                                        │
│   └── Capabilities & Pre-FR Optimization                         │
│            ↓                                                     │
│   BA (FR-instructions.md)                                        │
│   └── User Stories → Use Cases → Functional Requirements         │
│            ↓                                                     │
│   SA (SA-instructions.md)                                        │
│   └── Domain Model → Architecture → Services → Internal Design   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

**Last Updated:** 2026-01-09
**Applies To:** All services in the BA workspace
