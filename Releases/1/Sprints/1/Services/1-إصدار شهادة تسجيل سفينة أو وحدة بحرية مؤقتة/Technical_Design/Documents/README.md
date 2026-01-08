# REG001 Technical Design – Temporary Ship Registration Certificate Service

> **Service:** Issuing Temporary Registration Certificate for a Ship or Marine Unit  
> **Service Code:** REG001  
> **Arabic Name:** إصدار شهادة تسجيل سفينة أو وحدة بحرية مؤقتة  
> **Release:** 1  
> **Sprint:** 1  

---

## 1. Purpose of This Document
This document defines the **complete technical design documentation** for the Temporary Ship Registration Certificate Service (REG001). It provides implementation-ready specifications for designing, building, deploying, and operating this service.

The documentation is:
- Government submission ready
- Vendor handover ready
- Audit and compliance friendly
- Suitable for Agile delivery and microservices implementation

---

## 2. Folder Structure

```
Technical_Design/Documents/
│
├── README.md (this file)
│
├── 00-proposal/
│   ├── offer.md                          # Business proposal and technical offer
│   └── exports/                          # Exported proposal documents
│
├── 01-business-analysis/
│   ├── 1.1-scope/
│   │   ├── 01-Executive_Summary          # High-level service overview
│   │   └── 02-Service_Scope_Objectives   # Detailed scope and objectives
│   │
│   ├── 1.2-stakeholders/
│   │   └── 01-Stakeholders_Governance    # Stakeholder identification and roles
│   │
│   ├── 1.3-process/
│   │   ├── 01-Service_Blueprint          # Customer journey and touchpoints
│   │   ├── 02-Business_Process_Overview  # High-level process description
│   │   └── 03-BPMN_Process_Design        # Detailed BPMN workflows
│   │
│   ├── 1.4-capabilities/
│   │   ├── 01-Capability_to_Logical_Service_Mapping
│   │   └── 02-Capability_to_Domain_Mapping
│   │
│   └── 1.5-functional-requirements/
│       ├── README.md
│       ├── 00-overview/
│       ├── 01-user-stories/
│       ├── 02-use-cases/
│       ├── 03-functional-requirements/
│       ├── 04-cross-cutting/
│       ├── 05-integrations/
│       └── 06-acceptance-criteria/
│
├── 02-solution-design/
│   ├── 2.1-domain-model/
│   │   ├── 01-UML_Domain_Structure       # UML class diagrams
│   │   ├── 02-Business_Domain_Context_Map # Bounded contexts
│   │   └── 03-Context_Map_ACL_SharedKernel
│   │
│   ├── 2.2-architecture/
│   │   ├── 01-C4_Architecture            # C4 model (Context, Container, Component)
│   │   ├── 02-Capability_to_Service_C4   # Service decomposition
│   │   ├── 03-Sequence_Diagrams          # Service interaction flows
│   │   └── 04-Integration_Architecture   # External system integrations
│   │
│   ├── 2.3-services/
│   │   ├── 01-Service_Decomposition      # Microservices breakdown
│   │   ├── 02-Service_Overview           # Service descriptions
│   │   ├── 03-API_Contracts/             # API specifications (OpenAPI/Swagger)
│   │   │   └── README
│   │   └── 04-Event_Schemas              # Event-driven messaging schemas
│   │
│   ├── 2.4-cross-cutting/
│   │   ├── 01-Security_Trust_Architecture # Security design
│   │   ├── 02-Non_Functional_Requirements # NFRs (performance, scalability)
│   │   └── 03-Notifications_Communication # Notification strategy
│   │
│   └── 2.5-data-architecture/
│       ├── 01-Logical_Data_Model         # Entity relationship diagrams
│       ├── 02-Physical_Data_Considerations # Database design
│       ├── 03-Data_Ownership_Lifecycle   # Data governance
│       └── 04-Reporting_Analytics        # Reporting requirements
│
├── 03-delivery/
│   ├── 3.1-governance/
│   │   ├── 01-RACI_Matrix                # Responsibility assignment
│   │   └── 02-Traceability_Matrix        # Requirements traceability
│   │
│   ├── 3.2-risks/
│   │   └── 01-Risks_Mitigation           # Risk register and mitigation
│   │
│   ├── 3.3-roadmap/
│   │   └── 01-Implementation_Roadmap     # Delivery timeline
│   │
│   ├── 3.4-deliverables/
│   │   ├── 01-Deliverables               # Project deliverables list
│   │   └── 02-Conclusion                 # Project summary
│   │
│   └── 3.5-quality-assurance/
│       ├── 01-Testing_Strategy           # QA approach
│       ├── 02-Test_Types_Responsibilities # Test types and owners
│       └── 03-UAT_Signoff                # User acceptance testing
│
├── 04-devops-deployment/
│   ├── 01-Environments_Strategy          # Dev, Test, Prod environments
│   ├── 02-CI_CD_Pipeline                 # Build and deployment automation
│   ├── 03-Release_Management             # Release process
│   └── 04-Operational_Model              # Support and operations
│
├── diagrams/
│   ├── bpmn/                             # Business process diagrams
│   ├── c4/                               # C4 architecture diagrams
│   ├── context/                          # Context mapping diagrams
│   ├── sequence/                         # Sequence diagrams
│   └── uml/                              # UML diagrams
│
├── templates/
│   ├── architecture-doc-template         # Reusable doc templates
│   ├── service-spec-template
│   └── testing-template
│
├── docs/                                 # Additional documentation
└── revisions/                            # Version history
```

---

## 3. Service Overview

### 3.1 Service Description
The Temporary Ship Registration Certificate Service allows ship owners and operators to obtain temporary registration certificates for ships or marine units. This service is essential for vessels that need immediate registration for operational purposes before completing full registration requirements.

### 3.2 Key Integrations
- **Customs (Bayan)**: Verify customs clearance status
- **Ministry of Agriculture, Fisheries & Water Resources (MAFWR)**: Obtain approval for fishing vessels
- **Commercial Registration (CR)**: Validate business registration
- **Inspection Services**: Schedule and manage vessel inspections
- **Payment Gateway**: Process registration fees and penalties
- **Name Reservation System**: Reserve vessel names

### 3.3 Key Business Rules
- Vessel age validation
- Document requirements based on vessel type
- Customs clearance requirements
- Inspection requirements
- Fee and penalty calculations

---

## 4. Documentation Flow & Dependencies

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           PLANNING FLOW                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌──────────────────┐                                                       │
│   │  01-BUSINESS     │   WHAT problem are we solving?                        │
│   │  ANALYSIS        │   WHO are the stakeholders?                           │
│   │                  │   WHAT processes need automation?                     │
│   └────────┬─────────┘                                                       │
│            │                                                                 │
│            ▼                                                                 │
│   ┌──────────────────┐                                                       │
│   │  02-SOLUTION     │   HOW do we build it?                                 │
│   │  DESIGN          │   WHAT services & components?                         │
│   │                  │   HOW do they integrate?                              │
│   └────────┬─────────┘                                                       │
│            │                                                                 │
│            ▼                                                                 │
│   ┌──────────────────┐                                                       │
│   │  03-DELIVERY     │   WHEN do we deliver?                                 │
│   │                  │   WHO is responsible?                                 │
│   │                  │   WHAT are the risks?                                 │
│   └──────────────────┘                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Reading Guide by Audience

| Audience | Start Here | Key Documents |
|----------|------------|---------------|
| **Executive** | `00-proposal/offer` | Executive Summary, Conclusion, Roadmap |
| **Business Analyst** | `01-business-analysis/` | Scope, Process, BPMN, Capabilities |
| **Solution Architect** | `02-solution-design/` | C4, Domain Model, Integration |
| **Developer** | `02-solution-design/2.3-services/` | Service Decomposition, Sequence Diagrams, API Contracts |
| **Project Manager** | `03-delivery/` | RACI, Roadmap, Risks, Deliverables |
| **QA Engineer** | `03-delivery/3.5-quality-assurance/` | Testing Strategy, Test Cases |
| **DevOps Engineer** | `04-devops-deployment/` | CI/CD, Environments, Operations |

---

## 6. Work Stream Mapping

| Phase | Folder | Output | Depends On |
|-------|--------|--------|------------|
| **Phase 1: Discovery** | `1.1-scope/` + `1.2-stakeholders/` | Business Requirements | — |
| **Phase 2: Process Design** | `1.3-process/` | BPMN, Service Blueprint | Phase 1 |
| **Phase 3: Capability Mapping** | `1.4-capabilities/` | Capability Model | Phase 2 |
| **Phase 4: Domain Modeling** | `2.1-domain-model/` | Context Map, UML | Phase 3 |
| **Phase 5: Architecture** | `2.2-architecture/` | C4 Diagrams, Integration | Phase 4 |
| **Phase 6: Service Design** | `2.3-services/` | Microservices Specs | Phase 5 |
| **Phase 7: Cross-Cutting** | `2.4-cross-cutting/` | Security, NFRs | Phase 5 |
| **Phase 8: Data Architecture** | `2.5-data-architecture/` | Data Models, Lifecycle | Phase 6 |
| **Phase 9: Delivery Planning** | `03-delivery/` | Roadmap, RACI, QA | Phase 7+8 |
| **Phase 10: DevOps** | `04-devops-deployment/` | CI/CD, Operations | Phase 9 |

---

## 7. Current Implementation Assets

### 7.1 Decision Management (DMN)
Located in: `Technical_Design/Resources/dmn/reg001/`
- `reg001-customs-required.dmn` - Customs clearance decision rules
- `reg001-documents-required.dmn` - Document requirements by vessel type
- `reg001-fee-calculation.dmn` - Fee calculation logic
- `reg001-inspection-required.dmn` - Inspection requirement rules
- `reg001-mafwr-required.dmn` - MAFWR approval requirements
- `reg001-penalty-calculation.dmn` - Penalty calculation logic
- `reg001-vessel-age-validation.dmn` - Vessel age validation rules

### 7.2 Event Architecture
Located in: `Technical_Design/Resources/events/reg001/`
- Event-driven architecture definitions
- Inbound/outbound channels
- Event schemas for all service interactions

### 7.3 Process Definitions
Located in: `Technical_Design/Resources/processes/reg001/`
- BPMN process implementations
- Workflow orchestration

---

## 8. Status

✔ Folder structure created  
✔ Scope completed (Executive Summary + Scope & Objectives)  
✔ Stakeholders & governance completed  
✔ Process & BPMN completed (event-type tags added)  
✔ Capabilities mapping completed  
✔ Functional requirements completed (entry: `01-business-analysis/1.5-functional-requirements/README.md`)  
✔ Domain Model completed (UML Structure + Context Map + ACL/SharedKernel)  
⏳ Architecture (C4 + Integration) pending  
⏳ Services design pending  
⏳ Delivery planning pending  
⏳ DevOps configuration pending

---

## 9. Next Steps

1. Review Domain Model documents (`02-solution-design/2.1-domain-model/`) with Solution Architects
2. Generate C4 Architecture and Integration Architecture under `02-solution-design/2.2-architecture/`
3. Generate Service Decomposition and API Contracts under `02-solution-design/2.3-services/`
4. Confirm proposal content and populate `00-proposal/offer.md` (currently missing)

---

## 10. Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-05 | System | Initial structure created |
| 1.1 | 2026-01-05 | Copilot | Added Scope documents (Executive Summary + Scope & Objectives) |
| 1.2 | 2026-01-06 | Copilot | Added Stakeholders & Governance document |
| 1.3 | 2026-01-06 | Copilot | Added Capabilities mapping documents |
| 1.4 | 2026-01-08 | Copilot | Added Domain Model (UML Structure, Context Map, ACL/SharedKernel) |

---
