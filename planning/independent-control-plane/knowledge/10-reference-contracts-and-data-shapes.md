# 10 — Reference Contracts and Data Shapes

> **สถานะเอกสาร:** Design Baseline / Reference Contract  
> **วัตถุประสงค์:** ให้ implementation teams มี canonical shape สำหรับคุยกันโดยไม่ผูกกับ ORM, GUI หรือ Hermes implementation  
> **หลัก:** schema ด้านล่างเป็น semantic baseline; physical representation เปลี่ยนได้เมื่อรักษาความหมายเดิม

---

## 1. ทำไมต้องมี Reference Contracts

Architecture ที่อธิบายดีแต่ไม่มี contract shape มัก drift เมื่อเริ่ม code เพราะแต่ละ subsystem ตีความคำว่า Task, Result, Context, Grant และ Effect ต่างกัน เอกสารนี้จึงกำหนด “รูปร่างขั้นต่ำ” ที่ควรใช้ในการออกแบบ Python models, JSON schemas, DB mapping และ Adapter protocol

Contract เหล่านี้ต้องมีคุณสมบัติ:

- stable semantic identity
- explicit revision metadata
- optional fields เพิ่มได้โดย backward-compatible rules
- unknown optional fields ignore/preserve safely
- hashable canonical representation สำหรับ identity-sensitive records
- แยก secret handles ออกจาก payload

---

## 2. Ticket Contract

```yaml
ticket:
  id: ticket_...
  revision: 7
  request_key: req_...
  ingress_id: ing_...
  owner_principal_id: principal_...
  origin_session:
    id: session_...
    generation: 4
  raw_intent_ref: artifact:sha256:...
  interpreted_intent_ref: artifact:sha256:...
  priority_class: P2_FOREGROUND
  state:
    status: active
  outcome: null
  root_task_id: task_...
  delivery_id: null
  created_at: ...
  updated_at: ...
```

### Invariants

- `request_key` unique สำหรับ semantic admission
- raw intent immutable
- interpretation replace/version ได้แต่ห้าม overwrite raw intent
- Ticket terminal outcome เปลี่ยนได้เฉพาะ transition ที่อนุญาต

---

## 3. Task Contract

```yaml
task:
  id: task_...
  revision: 12
  ticket_id: ticket_...
  parent_task_id: null
  objective_ref: artifact:sha256:...
  success_criteria:
    - criterion_id: c1
      statement: ...
      required: true
  constraints: []
  invariants: []
  complexity_profile: standard
  state:
    phase: execute
    status: active
  outcome: null
  blocker_refs: []
  obligation_refs: []
  current_checkpoint_ref: artifact:sha256:...
  created_at: ...
  updated_at: ...
```

### Task dependency

```yaml
dependency:
  task_id: task_B
  depends_on_task_id: task_A
  requirement: artifact_or_state
  required_state: complete
  required_artifact_type: test_result
```

Task graph MUST be acyclic สำหรับ dependency edges ที่ require completion; parent-child nesting เองไม่ควรถูกใช้แทน dependency semantics ทุกกรณี

---

## 4. Knowledge Record

```yaml
knowledge_record:
  id: know_...
  task_id: task_...
  epistemic_type: fact | observation | assumption | hypothesis | inference | unknown | decision
  statement_ref: artifact:sha256:...
  authority_class: VERIFIED_ARTIFACT
  confidence: likely
  source_refs: []
  evidence_refs: []
  supersedes: null
  valid_from: ...
  valid_until: null
  created_at: ...
```

### Rules

- confidence optional; numeric probability ไม่บังคับ
- `fact` ต้องมี authority/provenance ที่ policy ยอมรับ
- `hypothesis` ห้าม promote เป็น fact โดย serialization/summary อย่างเดียว
- superseded record ยังอยู่ใน history

---

## 5. Obligation Contract

```yaml
obligation:
  id: obl_...
  ticket_id: ticket_...
  task_id: task_...
  kind: acceptance | validation | safety | delivery | user_decision | repair
  statement_ref: artifact:sha256:...
  required: true
  state: open | satisfied | waived | superseded
  satisfaction_evidence_refs: []
  owner_task_id: task_...
  created_at: ...
```

Obligation คือ durable antidote ต่อ context loss: historical prose หายได้แต่ required obligation หายไม่ได้

---

## 6. Context Manifest / Envelope Contract

### Manifest — บอกวิธีสร้าง

```yaml
context_manifest:
  id: ctxm_...
  compiler_revision: 3
  ticket_revision: 7
  task_revision: 12
  purpose: runtime_execution
  budget_tokens: 8192
  required_record_ids: []
  selected_record_ids: []
  excluded_record_ids: []
  conflict_refs: []
  envelope_hash: sha256:...
  audit_ref: artifact:sha256:...
```

### Envelope — สิ่งที่ runtime เห็น

```yaml
context_envelope:
  identity: {...}
  objective: {...}
  current_state: {...}
  relevant_knowledge: []
  evidence_refs: []
  obligations: []
  dependencies: []
  authority:
    grants: []
    forbidden_capabilities: []
  budget: {...}
  stop_conditions: []
  output_contract: {...}
```

Manifest durable กว่า rendered prompt; prompt renderer อาจเปลี่ยนตาม model/runtime

---

## 7. Scheduler Job and Quantum

```yaml
scheduler_job:
  id: job_...
  task_id: task_...
  owner_key: principal:...
  class: P2_FOREGROUND
  state: runnable
  enqueue_seq: 1002
  waiting_since: ...
  consecutive_quanta: 1
  budget_ref: budget_...
```

```yaml
execution_quantum:
  id: qnt_...
  job_id: job_...
  lease:
    owner: core-instance-...
    generation: 8
    token: lease_...
    expires_at: ...
  boundary_policy: bounded_turns
  max_runtime_ms: 120000
  created_at: ...
```

Quantum identity ช่วย audit fairness และ resource ownership โดยไม่ทำให้ Task state ปน scheduling

---

## 8. Runtime Attempt

```yaml
runtime_attempt:
  id: ratt_...
  task_id: task_...
  quantum_id: qnt_...
  adapter_id: hermes
  adapter_epoch: 19
  lease_generation: 8
  context_manifest_id: ctxm_...
  execution_contract_hash: sha256:...
  state: submitted | running | waiting | completed | failed | outcome_unknown | cancelled
  native_session_ref: ...
  native_run_ref: ...
  started_at: ...
  ended_at: ...
```

### Runtime Result

```yaml
runtime_result:
  runtime_attempt_id: ratt_...
  status: completed
  claims: []
  observations: []
  evidence_refs: []
  artifacts: []
  proposed_next_action: ...
  unresolved: []
  native_output_ref: artifact:sha256:...
```

Core validates result contract ก่อน commit semantic transition

---

## 9. Authorization Grant

```yaml
authorization_grant:
  id: grant_...
  principal_id: principal_...
  ticket_id: ticket_...
  capability_id: repo.push
  scope_ref: artifact:sha256:...
  scope_hash: sha256:...
  constraints:
    max_attempts: 1
  requires_confirmation: true
  confirmation_id: conf_...
  confirmed_scope_hash: sha256:...
  issued_at: ...
  expires_at: ...
  revoked_at: null
```

Material parameter change → scope hash เปลี่ยน → confirmation เดิม invalid

---

## 10. Effect Intent / Attempt

```yaml
effect_intent:
  id: eff_...
  ticket_id: ticket_...
  task_id: task_...
  capability_id: repo.push
  effect_class: EXTERNAL_IDEMPOTENT
  grant_id: grant_...
  target_hash: sha256:...
  parameters_hash: sha256:...
  idempotency_key: cnx:eff_...
  replay_policy: RETRY_SAME_IDEMPOTENCY_KEY
  expected_evidence_schema: repo_push_evidence
  attempt_budget: 2
  state: authorized
```

```yaml
effect_attempt:
  id: effatt_...
  effect_id: eff_...
  adapter_id: hermes
  mapping_revision: 5
  mapping_hash: sha256:...
  lease_generation: 8
  attempt_no: 1
  state: executing
  external_operation_ref: null
  evidence_refs: []
```

---

## 11. Delivery Contract

```yaml
delivery:
  id: del_...
  ticket_id: ticket_...
  payload_ref: artifact:sha256:...
  payload_hash: sha256:...
  recipient_principal_id: principal_...
  target_session:
    id: session_...
    generation: 4
  policy: CONFIRM_REQUIRED
  state: staged
```

Retry retains `delivery_id`; native platform ID เป็น evidence ไม่ใช่ semantic identity หลัก

---

## 12. Transition/Event Contract

```yaml
event:
  id: evt_...
  aggregate_type: task
  aggregate_id: task_...
  aggregate_revision: 13
  event_type: validation.passed
  causation_id: evt_or_command_...
  correlation_id: ticket_...
  actor_ref: validator:...
  payload_ref: artifact:sha256:...
  payload_hash: sha256:...
  created_at: ...
```

Events describe occurrences; Commands describe desired actions

---

## 13. Compatibility rules

### Minor-compatible
- optional field เพิ่ม
- optional event type เพิ่มที่ old receiver ignore ได้
- extension namespace ใหม่

### Major-incompatible
- field semantic เปลี่ยน
- transition invariant เปลี่ยนแบบ old state invalid
- required field/guard ใหม่ที่ old implementation ทำไม่ได้

Core must fail clearly on unsupported required semantics

---

## 14. Review checklist

ก่อนเพิ่ม field/object:

1. semantic responsibility ใหม่จริงหรือไม่
2. เป็น durable truth หรือ derived view
3. identity ต้อง stable across retries หรือไม่
4. ต้อง event/revision หรือไม่
5. มี secret/sensitivity หรือไม่
6. compatibility behavior เมื่อ old reader ไม่รู้ field
7. recovery ใช้ record นี้อย่างไร
8. GUI มีสิทธิ์ดู/command ผ่านอะไร
