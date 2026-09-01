# 01 — Canonical Ontology and State Model

> **สถานะเอกสาร:** Design Baseline / Living Specification  
> **ภาษาหลัก:** ไทย โดยใช้คำศัพท์เชิงสถาปัตยกรรมภาษาอังกฤษเป็น canonical term  
> **หลักการอ่าน:** คำว่า MUST / MUST NOT / SHOULD / MAY ใช้ในความหมายเชิง normative  
> **ขอบเขต:** CogentNexus เป็นโปรแกรมอิสระที่เป็น durable semantic control plane และเชื่อมระบบภายนอกผ่าน replaceable Adapters


## 1. เหตุผลที่ ontology ต้องเล็กและคงที่

ระบบใหญ่พังง่ายเมื่อคำอย่าง Task, Workflow, Step, Job, Run, Result ถูกใช้ทับความหมายกัน การเพิ่ม object เพียงเพราะงานใหญ่ขึ้นทำให้ schema และ recovery ซับซ้อนโดยไม่เพิ่ม semantic value

หลักคือ:

> **Scale by composition, not by inventing a new conceptual model.**

---

## 2. Canonical object hierarchy

```text
Ingress Message
    ↓ durable admission
Ticket
    ↓ creates logical work
Task (recursive)
├── Task
│   └── Task
└── Task

Task competes for resources through:
Scheduler Job
    ↓ bounded grant
Execution Quantum
    ↓ dispatched to runtime
Runtime Attempt
    ├── 0..N Model Runs
    ├── 0..N Effect Attempts
    └── produces Artifacts / Observations / Evidence

Ticket output becomes:
Delivery
    └── 1..N Delivery Attempts
```

### Ticket
root ของ accepted intent, admission identity, owner principal/session และ final delivery relation

### Task
หน่วย logical responsibility ที่มี objective, success criteria, constraints, state, knowledge/evidence refs และ child Tasks

### Scheduler Job
representation ของ runnable resource request; ไม่ใช่ semantic task

### Execution Quantum
bounded scheduling unit ที่คืน control ให้ scheduler ที่ safe boundary

### Runtime Attempt
การมอบ execution contract ให้ runtime หนึ่งครั้ง เช่น Hermes; อาจครอบหลาย model/tool turns

### Model Run
provider/model invocation จริงหนึ่งครั้ง

### Effect Intent / Effect Attempt
semantic mutation identity กับ execution attempt ของ mutation เดิม

### Delivery / Delivery Attempt
semantic user-facing output identity กับ native transport attempt

---

## 3. สิ่งที่ไม่ใช้เป็น canonical top-level object

หลีกเลี่ยง `Workflow`, `WorkItem`, `Step` เป็น public ontology หากไม่ได้มี lifecycle/invariant ต่างจาก Task

สามารถใช้คำว่า workflow เชิง prose ได้ แต่ storage/protocol ใช้ Task graph

---

## 4. Task State: แยก Phase, Status, Outcome

การใช้ state เดียวเช่น `VERIFYING_BLOCKED` ทำ combinatorial explosion จึงแยกสามมิติ

### Phase — งานอยู่ช่วง semantic ใด

```text
FRAME
EXECUTE
VERIFY
CLOSE
```

- **FRAME**: objective/scope/success criteria/invariants เพียงพอสำหรับ next action
- **EXECUTE**: analysis, experiment, implementation, artifact production, dependency resolution
- **VERIFY**: ตรวจ success criteria/invariants/evidence/regression
- **CLOSE**: seal outcome, report, delivery preparation, limitations

### Status — งานกำลังเป็นอย่างไร

```text
PENDING
ACTIVE
BLOCKED
COMPLETE
```

`FAILED` ไม่จำเป็นต้องเป็น runtime status หากใช้ Outcome แยก; failure ระหว่างทางอาจยัง recover ได้

### Outcome — ถ้าจบ จบอย่างไร

```text
SUCCESS
PARTIAL
INCONCLUSIVE
BLOCKED
FAILED
CANCELLED
SUPERSEDED
```

ตัวอย่าง:

```yaml
state:
  phase: close
  status: complete
outcome:
  disposition: partial
  reason: "Windows validation unavailable"
```

---

## 5. State transition semantics

Formal model:

```text
State + Event + Guard → New State
```

LLM ไม่เปลี่ยน state โดยตรง; มันเสนอ action/output ที่อาจสร้าง event/evidence

ตัวอย่าง:

```text
EXECUTE/ACTIVE
+ artifact.produced
+ required_execution_artifacts_exist
→ VERIFY/ACTIVE
```

และ:

```text
VERIFY/ACTIVE
+ validation.passed
+ all_required_criteria_satisfied
+ no_unresolved_critical_obligation
→ CLOSE/ACTIVE
```

Transition record ต้องมีอย่างน้อย:

```yaml
transition_id:
aggregate_id:
from:
to:
event:
guards_evaluated:
evidence_refs:
actor_ref:
causation_id:
created_at:
```

---

## 6. Parent/Child semantics

Child COMPLETE ทั้งหมดไม่ทำให้ parent COMPLETE อัตโนมัติ เพราะ parent อาจต้อง integrate/compose/validate เอง

Child attributes ควรมี:

```yaml
child_ref:
required: true|false
purpose:
dependency_role:
```

Parent completion predicate อาจเป็น:

```text
required_children_terminal
AND required_obligations_resolved
AND parent_validation_passed
AND completion_contract_satisfied
```

---

## 7. Knowledge ontology

### Fact
สิ่งที่ Core ถือว่า established ณ revision ปัจจุบัน พร้อม provenance

### Observation
สิ่งที่ตรวจพบจาก environment/tool/test/user/runtime

### Assumption
ใช้ชั่วคราวเพื่อเดินงาน แต่ยังไม่พิสูจน์

### Hypothesis
ข้ออธิบายที่สามารถทดสอบได้

### Inference
ข้อสรุปที่ได้จาก evidence

### Unknown
สิ่งที่ยังไม่ทราบและอาจ block/เพิ่ม risk

### Decision
ทางเลือกที่ถูกเลือกพร้อม rationale/provenance ที่จำเป็น

### Evidence
สิ่งที่ใช้สนับสนุนหรือหักล้าง claim

### Obligation
สิ่งที่ยังต้องทำ/พิสูจน์/ส่ง/ขอ authorization; ห้ามหายเพราะ context compaction

---

## 8. Retry and progress ontology

Attempt count ไม่ใช่ progress

```yaml
progress:
  knowledge_delta: false
  evidence_delta: true
  artifact_delta: false
  state_delta: false
  obligation_delta: false
```

กฎ:

```text
same semantic state
+ same strategy
+ same evidence
+ same dependency state
= retry forbidden
```

อนุญาต retry เมื่อมี delta ที่มีความหมาย เช่น evidence ใหม่, strategy ใหม่, authorization ใหม่, dependency state ใหม่ หรือ idempotency guarantee ที่ชัด

---

## 9. Blocker model

`BLOCKED` ไม่ใช่ dead end

ประเภทตัวอย่าง:

```text
missing_information
missing_dependency
authorization_required
capability_unavailable
resource_unavailable
external_failure
conflicting_constraints
uncertainty_too_high
unsafe_action
context_budget_blocked
```

Blocker ต้องมี resolution condition เพื่อ wake Task แบบ deterministic

---

## 10. Naming rules

- nouns for entities: `Task`, `Evidence`, `Artifact`
- verbs for commands: `run_validation`, `cancel_task`
- past/completed semantics for events: `validation.passed`, `task.cancelled`
- version อยู่ metadata ไม่อยู่ชื่อ object
- extension ห้าม redefine core semantics

ตัวอย่าง namespace:

```text
cnx.task
cnx.evidence
cnx.artifact
cnx.runtime.*
cnx.software.*
```

---

## 11. สิ่งที่ต้องระวัง

1. อย่าใช้ `state` TEXT โดยไม่มี enum/transition contract
2. อย่าให้ Task State ปนกับ Scheduler State
3. อย่าใช้ `result` เป็นคำรวมทุกอย่าง
4. อย่าให้ summary แปลง hypothesis เป็น fact
5. อย่าให้ child completion แทน parent validation
6. อย่า rename ontology ตาม implementation release

---

## 12. Acceptance criteria

Ontology พร้อมใช้งานเมื่อ:

- state ทุก aggregate มีความหมายไม่ทับกัน
- crash recovery ไม่ต้องอ่าน conversation
- Task graph รองรับ 1 action ถึงหลายพัน child โดย conceptual model เดิม
- scheduler/runtime/effect/delivery state ไม่ปน semantic Task state
- repeated attempts ไม่สร้าง duplicate semantic work
- another runtime สามารถรับ Task เดิมได้จาก durable artifacts/context envelope
