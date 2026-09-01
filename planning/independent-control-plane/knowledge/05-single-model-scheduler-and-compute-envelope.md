# 05 — Single-Model Scheduler and Compute Envelope

> **สถานะเอกสาร:** Design Baseline / Living Specification  
> **ภาษาหลัก:** ไทย โดยใช้คำศัพท์เชิงสถาปัตยกรรมภาษาอังกฤษเป็น canonical term  
> **หลักการอ่าน:** คำว่า MUST / MUST NOT / SHOULD / MAY ใช้ในความหมายเชิง normative  
> **ขอบเขต:** CogentNexus เป็นโปรแกรมอิสระที่เป็น durable semantic control plane และเชื่อมระบบภายนอกผ่าน replaceable Adapters


## 1. หลักคิด

ระบบที่มี model slot เดียวไม่ใช่โหมดพิการ แต่เป็น baseline correctness

> Model reasons about current bounded work; host scheduler reasons about contention.

Global queue/fairness/resource state ไม่ควรเข้า LLM context

---

## 2. Compute Envelope

แยก semantic intelligence capability ออกจาก physical resources:

```yaml
compute_envelope:
  max_loaded_models: 1
  max_concurrent_model_calls: 1
  memory_budget_mb:
  context_budget_tokens:
  supports_parallel_runtime_attempts: false
  supports_parallel_model_runs: false
```

เปลี่ยนเครื่อง/provider แล้ว Compute Envelope เปลี่ยน แต่ Task semantics เดิม

---

## 3. Resource architecture

ถึง `MODEL_SLOTS=1` host work อื่นทำพร้อมกันได้:

- ingress admission
- SQLite transactions
- IPC
- hashing/artifact storage
- health monitoring
- delivery/reconciliation
- non-model deterministic validation

ห้าม serialize โปรแกรมทั้งตัวเพราะ inference serialize

---

## 4. Job classes

```text
P0_CONTROL      safety/recovery/revoke/reconcile
P1_INTERACTIVE  foreground human input
P2_FOREGROUND   ongoing user-authorized work
P3_BACKGROUND   maintenance/optional work
```

priority ไม่ใช่ license ให้ starve

---

## 5. Fairness

Recommended policy combines:

```text
base class
+ bounded aging
+ owner fairness
+ continuation locality bonus
+ recovery bonus
- over-budget penalty
```

ภายใน class ใช้ weighted fairness/Deficit Round Robin ตาม owner/Ticket เพื่อป้องกัน endless workflow monopolization

---

## 6. Execution Quantum

Quantum คือ bounded resource grant ไม่ใช่ semantic Task

safe boundaries:

- one direct model invocation
- one bounded Hermes turn batch
- checkpoint-producing unit
- before new tool result requires reasoning
- budget threshold
- pause/cancel request

Preemption correctness ควรเกิดระหว่าง model/runtime boundaries ไม่พึ่ง mid-generation cancellation

---

## 7. Scheduler state

```text
QUEUED
RUNNABLE
CLAIMED
RUNNING
WAITING_IO
CHECKPOINTED
PAUSE_REQUESTED
PAUSED
COMPLETED
FAILED
CANCELLED
```

นี่คนละ state space กับ Task `FRAME/EXECUTE/VERIFY/CLOSE`

---

## 8. Pause / Preemption / Cancellation

- **Pause**: intent ยัง valid, resume จาก checkpoint
- **Preemption**: หยุด grant quantum ชั่วคราวเพื่อ fairness/priority
- **Cancellation**: revoke future execution authority

Committed side effects เป็น historical facts ไม่สามารถ “uncall” ได้

---

## 9. Budget model

Budget ไม่ใช่ token อย่างเดียว:

```yaml
budget:
  model_calls:
  input_tokens:
  output_tokens:
  context_bytes:
  tool_calls:
  retry_count:
  side_effect_attempts:
  active_wall_clock_ms:
```

Budget exhaustion อาจนำไป `WAITING_BUDGET_REVIEW`, `PAUSED_RESOURCE_LIMIT`, `FAILED_POLICY_LIMIT` ไม่ใช่ failure เดียว

---

## 10. Waiting I/O

Task/Job ที่รอ Adapter/tool ต้อง release model slot

```text
RUNNING → WAITING_IO
result arrives durably
WAITING_IO → RUNNABLE
```

ห้ามใช้ model polling เพื่อรองานที่ host observe ได้

---

## 11. New input while model busy

Ingress ต้อง admit SQLite ได้ทันที แล้วค่อย queue P1_INTERACTIVE

current call เดินถึง safe boundary แล้ว scheduler re-evaluate

จึงไม่มีข้อความหายเพราะ model “กำลังคิด”

---

## 12. Scheduler vs LLM Planner

LLM MAY propose decomposition/dependency/risk/priority hint

Scheduler MUST own:

- global fairness
- hard class rules
- model slot lease
- budget
- pause/cancel
- starvation bounds
- capacity/backpressure

---

## 13. Hermes-specific caveat

Hermes Runtime Attempt อาจมีหลาย LLM/tool turns ภายใน ดังนั้น Adapter qualification ต้องบอก yield granularity จริง

ถ้า `OPAQUE_RUN`, Core ต้องตั้ง conservative runtime budget และ fairness boundary ที่ run level แทน pretend per-model-turn control

---

## 14. Acceptance

- model lease มีได้หนึ่ง owner
- 100 queued jobs survive restart
- interactive ingress admitted while busy
- equal owners ได้ bounded fair service
- paused resume โดยไม่ replay committed effects
- WAITING_IO ไม่กิน model slot
- stale lease result commit ไม่ได้
- budget exhaustion durable/explainable
- small model ทำ progress ต่อเนื่องโดยไม่เห็น global queue
