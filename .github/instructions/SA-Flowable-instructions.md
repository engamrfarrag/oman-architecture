# Solution Architect – Flowable Instructions

> **Role:** Solution Architect  
> **Scope:** Flowable BPMN, DMN, Process Orchestration  
> **Applies To:** `02-solution-design/` folder  
> **Parent:** [SA-instructions.md](SA-instructions.md)

---

## 1. Foundational Constraint (CRITICAL)

This solution uses **FLOWABLE** as the process engine.  
BPMN and DMN are **EXECUTION MODELS**, not documentation-only artifacts.

All solution design MUST align with Flowable capabilities and constraints.

---

## 2. Flowable Authority

Flowable is the **authoritative**:
- BPMN process engine
- DMN decision engine
- Long-running workflow manager

You MUST design around:
- Executable BPMN processes
- DMN-based decision logic
- Flowable state management
- Flowable eventing and timers

**DO NOT invent custom orchestration logic outside Flowable.**

---

## 3. Architectural Principle

| Component | Owns |
|-----------|------|
| **Flowable** | Process State |
| **Microservices** | Domain State |
| **DMN** | Decision Logic |

> **Never duplicate Flowable responsibilities in services.**

---

## 4. State Ownership Rule

### Flowable Owns:
- Process instance state
- Task assignments
- Wait states
- Timers and retries

### Services Own:
- Domain entity state
- Aggregate lifecycle

**Never duplicate state machines.**

---

## 5. Process Orchestration Rules

### 5.1 BPMN Requirements

- All long-running workflows MUST be modeled in BPMN
- Use Flowable:
  - **Service Tasks** (sync/async)
  - **User Tasks** (approval queues)
  - **Message Events** (external triggers)
  - **Timer Events** (deadlines, SLAs)
  - **Error Events** (hard failures)

### 5.2 Saga Pattern

Saga pattern MUST be implemented via:

```text
Flowable BPMN (orchestration-based saga)
```

**NOT via:**
- Custom service orchestration
- Application-level state machines
- Database-driven workflow tables

---

## 6. Decision Management (DMN) – Strict

### 6.1 When to Use DMN

All conditional routing MUST use DMN when:
- Rules are configurable
- Thresholds may change
- Business owns the logic

### 6.2 DMN Use Cases

| Decision Type | DMN Required |
|---------------|--------------|
| Required documents | ✅ Yes |
| Fee calculation | ✅ Yes |
| Inspection required | ✅ Yes |
| Approval required | ✅ Yes |
| Penalty rules | ✅ Yes |
| Vessel age validation | ✅ Yes |

### 6.3 DMN Rule

> **DMN rules MUST NOT be hardcoded in services.**

---

## 7. Architecture Section (2.2) – Flowable First

When generating `02-solution-design/2.2-architecture/`:

### 7.1 Treat Flowable as Core Platform Component

Clearly show:
- BPMN execution
- DMN invocation
- Event correlation
- Message correlations

### 7.2 Sequence Diagrams MUST Reflect:

```text
API → Flowable → Service → Event → Flowable
```

**DO NOT model orchestration inside services when BPMN already exists.**

---

## 8. Services Section (2.3) – Flowable Aware

When generating service definitions:

### 8.1 Services MUST Be Stateless Regarding Process Flow

**Services MAY:**
- Execute business logic
- Emit domain events
- Persist domain data

**Services MUST NOT:**
- Track process steps
- Own workflow state
- Implement workflow transitions

### 8.2 Each Service MUST Declare:

- Interaction with Flowable (task, event, callback)
- Sync vs async behavior

---

## 9. Service Internal Design – Flowable Integration

For each service in `2.7-service-internal-design/`:

### 9.1 `1-domain-core/`

- Aggregates and invariants only
- No workflow logic
- No decision routing

### 9.2 `2-support/`

- Application services triggered by Flowable service tasks
- Command handlers invoked by Flowable
- Event publishers for BPMN message events
- **No orchestration logic**

### 9.3 `3-intelligence/`

- DMN interaction points
- Input/output mapping to DMN
- Policy ownership
- Rule evolution strategy

### 9.4 `4-resilience-governance/`

- BPMN error mapping
- Compensation handled via BPMN
- Idempotency for message re-delivery
- Correlation keys for Flowable events
- Audit alignment with Flowable history

---

## 10. Flowable Adapter Rule

Each process/orchestration service MUST include a **Flowable Adapter** in the support layer.

### 10.1 Adapter Responsibilities

- Starts processes
- Completes service tasks
- Correlates messages
- Evaluates DMN tables

### 10.2 Structure Location

```text
2.7-service-internal-design/<ServiceName>/
└── 2-support/
    └── 03-Flowable_Process_Adapter
```

**The adapter is INTERNAL and NOT exposed to UI.**

---

## 11. BPMN Task Types Reference

| Task Type | Use Case |
|-----------|----------|
| **Service Task** | Invoke domain service (sync/async) |
| **User Task** | Human approval/review |
| **Send Task** | Publish event/message |
| **Receive Task** | Wait for external event |
| **Script Task** | Lightweight transformation |
| **Business Rule Task** | Invoke DMN decision |

---

## 12. Event Types Reference

| Event Type | Use Case |
|------------|----------|
| **Start Event** | Process initiation |
| **End Event** | Process completion |
| **Message Event** | External system trigger |
| **Timer Event** | Deadline/SLA enforcement |
| **Error Event** | Exception handling |
| **Signal Event** | Broadcast notification |

---

## 13. Stop Conditions

**STOP and explain if design:**
- Bypasses Flowable
- Re-implements workflow logic in services
- Hardcodes decision logic
- Exposes Flowable to frontend

---

## 14. Output Quality Rules

- BPMN and DMN are the **source of truth**
- Use Flowable terminology correctly
- Do not describe generic workflow engines
- Be explicit about async vs sync
- Maintain traceability to BPMN IDs and DMN tables

---

## 15. One-Line Rule

> **Flowable decides "when and what next", services decide "how".**

---

## 16. Related Instruction Files

- [SA-instructions.md](SA-instructions.md) – Main SA instructions
- [SA-UIInteraction-instructions.md](SA-UIInteraction-instructions.md) – Angular + Flowable isolation
- [SA-ServiceInternalDesign-instructions.md](SA-ServiceInternalDesign-instructions.md) – Internal design layers
- [Process-instructions.md](Process-instructions.md) – BPMN subprocess design

---

**Last Updated:** 2026-01-07  
**Applies To:** All Flowable-based Solution Architecture tasks
