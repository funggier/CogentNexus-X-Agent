# 03 — Context, Knowledge, Evidence, and Recovery

> **สถานะเอกสาร:** Design Baseline / Living Specification  
> **ภาษาหลัก:** ไทย โดยใช้คำศัพท์เชิงสถาปัตยกรรมภาษาอังกฤษเป็น canonical term  
> **หลักการอ่าน:** คำว่า MUST / MUST NOT / SHOULD / MAY ใช้ในความหมายเชิง normative  
> **ขอบเขต:** CogentNexus เป็นโปรแกรมอิสระที่เป็น durable semantic control plane และเชื่อมระบบภายนอกผ่าน replaceable Adapters


## 1. หลัก Knowledge ≠ Context

CogentNexus ควรเก็บ knowledge ได้มาก แต่ไม่ส่งทุกอย่างให้ LLM

```text
Durable Knowledge
  ├─ raw intent
  ├─ tasks
  ├─ transitions
  ├─ facts/unknowns/decisions
  ├─ evidence/artifacts
  ├─ obligations
  ├─ effects/delivery
  └─ history
          ↓ derive
Context Compiler
          ↓
Bounded Context Envelope
          ↓
LLM / Hermes
```

Context เป็น view ที่สร้างใหม่ได้ จึงไม่ใช่ memory authority

---

## 2. Context Compiler pipeline

```text
Context Request Builder
→ Source Resolution
→ Authority Resolution
→ Relevance Selection
→ Dependency Closure
→ Obligation Preservation
→ Deduplication
→ Budget Manager
→ Envelope Renderer
→ Context Audit
```

ลำดับ selection ที่แนะนำ:

1. mandatory invariants/fences
2. explicit Task dependencies
3. unresolved obligations
4. acceptance/validator evidence
5. authoritative knowledge records
6. artifact/provenance links
7. recency/domain heuristics
8. semantic retrieval เป็น candidate discovery เท่านั้น

---

## 3. Authority ordering

ตัวอย่าง semantic authority classes:

```text
SYSTEM_FENCE
SIDE_EFFECT_PROOF
CANONICAL_WORK_STATE
VALIDATED_STATE
VERIFIED_ARTIFACT
ENVIRONMENT_OBSERVATION
CONVERSATION
DERIVED_SUMMARY
```

เมื่อ conflict:

- higher authority ชนะในการสร้าง active view
- lower authority conflict ยังถูกเก็บใน audit
- ห้ามลบ evidence เพื่อทำให้เรื่องดูเรียบง่าย

---

## 4. Context Envelope

Canonical form:

```yaml
context_envelope:
  purpose: runtime_execution
  identity:
    ticket_id:
    task_id:
    runtime_attempt_id:
    generation:
  objective:
    global:
    local:
  state:
    phase:
    status:
    checkpoint_ref:
  knowledge:
    facts: []
    unknowns: []
    decisions: []
  evidence_refs: []
  obligations: []
  dependencies: []
  authority:
    allowed_capabilities: []
    forbidden_capabilities: []
    effect_intents: []
  budget:
    context_tokens:
    model_calls_remaining:
  requested_transition:
  output_contract:
  stop_conditions: []
```

ไม่จำเป็นต้อง populate ทุก field; profile และ purpose กำหนด adaptive presence

---

## 5. Progressive profiles

### INTERACTION_LIGHT
casual/simple response, context เล็กมาก

### WORK_STANDARD
มี Task state, relevant facts, constraints, output contract

### WORK_ANALYTICAL
เพิ่ม hypotheses/evidence/uncertainty/validation plan

### RECOVERY_STRICT / CRITICAL
เพิ่ม fences, provenance, authorization, effect/delivery state, independent validation

เริ่มเบาที่สุดแล้ว promote เมื่อ evidence บอกว่าจำเป็น

---

## 6. Context budget semantics

ถ้า mandatory context เกิน budget ห้าม silent truncate

```text
CONTEXT_BUDGET_BLOCKED
```

จากนั้น Work Shaping เลือก:

- split Task
- normalize/condense evidence
- retrieve artifact on demand
- create child Task
- request larger safe context if capability permits

หลักคือ **reshape work before asking the model to hold too much**

---

## 7. Result normalization

Child/runtime ไม่ควรส่ง transcript ทั้งหมดให้ composer

```yaml
cognitive_result:
  task_id:
  status:
  claims: []
  evidence_refs: []
  artifacts: []
  uncertainty: []
  unresolved: []
  proposed_next_action:
```

raw logs/output ยังเก็บเป็น artifact และ fetch ได้

---

## 8. Evidence projection

แทนส่ง log 5,000 บรรทัด:

```yaml
observation:
  test_suite: installer
  status: failed
  failures:
    - npm12_contract
  source_ref: artifact:sha256:...
```

decision-making context ควรใช้ evidence projection แต่ต้อง trace กลับ raw artifact ได้

---

## 9. Recovery semantics

Recovery ใช้:

1. latest committed Task state/revision
2. latest transition/event
3. unresolved obligations/blockers
4. established knowledge/evidence
5. committed effect/delivery state
6. durable artifacts
7. current runtime binding/capability state
8. next valid transition/action

ไม่ recover จาก “agent กำลังคิดอะไร”

### Exact Resume
ใช้ Hermes session resume ถ้า identity/workspace/model/provider/fence compatible

### Semantic Resume
สร้าง Context Envelope ใหม่และ runtime binding ใหม่จาก durable state — นี่คือ correctness mechanism

---

## 10. Context Audit

ทุก compiled envelope ควรตอบ:

- included records อะไร
- excluded อะไร
- mandatory อะไร
- authority ของแต่ละ record
- source/provenance
- token/byte cost
- compiler revision
- reasons for exclusion
- conflicts discovered

Operator/GUI ควร inspect audit ได้โดยไม่ต้องเห็น private reasoning

---

## 11. สิ่งที่ต้องระวัง

- summary authority สูงเกิน raw evidence
- stale context หลัง generation change
- sibling transcript leakage
- secrets ใน context โดยไม่จำเป็น
- model context ใช้เป็น obligation registry
- semantic search เลือก truth โดยไม่มี deterministic guard
- context compiler fail-open ใน strict Managed Mode

---

## 12. Acceptance criteria

- delete model/session context แล้วงานต่อได้
- Context Envelope เดิม reproducible จาก same durable revision/policy
- mandatory obligation ไม่หายจาก compaction
- over-budget context block อย่าง explicit
- child Tasks ไม่เห็น unrelated sibling data
- context audit อธิบาย inclusion/exclusion ได้
- strong model optimization ถอดออกแล้ว lifecycle ยังถูกต้อง
