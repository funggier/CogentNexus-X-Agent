# 04 — Adapter Transport and Runtime Contract

> **สถานะเอกสาร:** Design Baseline / Living Specification  
> **ภาษาหลัก:** ไทย โดยใช้คำศัพท์เชิงสถาปัตยกรรมภาษาอังกฤษเป็น canonical term  
> **หลักการอ่าน:** คำว่า MUST / MUST NOT / SHOULD / MAY ใช้ในความหมายเชิง normative  
> **ขอบเขต:** CogentNexus เป็นโปรแกรมอิสระที่เป็น durable semantic control plane และเชื่อมระบบภายนอกผ่าน replaceable Adapters


## 1. เป้าหมาย

Core ↔ Adapter ไม่ควรถูกมองเป็น RPC ที่ “เรียกแล้วได้ผลกลับ” เพราะ crash/replay/external ambiguity ทำให้ request/response ธรรมดาอธิบายความจริงไม่พอ

หลัก:

> **Adapter transport is a durable replicated conversation between authorities.**

---

## 2. Guarantees

```text
at-least-once transport
+ durable dedupe
+ per-stream ordering
+ causal dependencies
+ fenced authority
+ idempotent Core transitions
+ effect-specific replay policy
```

เป้าหมายคือ effectively-once semantic commitment ไม่ใช่ exactly-once packet delivery

---

## 3. Protocol identity

ชื่อ semantic ที่แนะนำ:

```yaml
protocol:
  name: cnx.adapter.transport
  version: 1.0
schema:
  revision: 1
```

อย่าผูกชื่อกับ `IPC` เพราะ transport อาจขยายจาก local pipe ไป remote network

---

## 4. Stable envelope

```yaml
message_id: msg_...
command_id: cmd_...          # optional for non-command frames
correlation_id: ticket_...
causation_id: evt_...
stream_id: task:...
stream_seq: 17
sender:
sender_instance:
sender_epoch:
recipient:
message_type:
payload_type:
payload_schema:
payload_schema_revision:
payload_hash:
created_at:
expires_at:
trace_id:
payload_ref_or_inline:
```

Identities ห้ามปน:

- `message_id` = transport frame
- `command_id` = semantic command
- `runtime_attempt_id` = runtime dispatch
- `effect_id` = mutation identity
- `delivery_id` = semantic delivery identity

---

## 5. ACK semantics

### ACK_RECEIVED
parsed/recognized แต่ยังไม่ durable

### ACK_DURABLE
persisted enough to recover; sender หยุด transport replay ได้ตาม policy

### RESULT
semantic outcome ของ command เช่น success/retryable/permanent/unknown/stale/rejected

### ACK_COMMITTED
semantic authority commit RESULT แล้ว; Adapter จึง retire durable result outbox ได้

ห้ามตีความ ACK_DURABLE ว่า external action สำเร็จ

---

## 6. Replay and dedupe

Duplicate frame เป็น normal behavior

Rules:

| Condition | Behavior |
|---|---|
| same message_id + same hash | return prior ack/result |
| same message_id + different hash | fail closed |
| same command_id + same semantic hash | converge |
| same command_id + different hash | conflict reject |
| stale generation | reject no mutation |
| replay terminal | return terminal receipt |

---

## 7. Ordering

หลีกเลี่ยง global total ordering

- unordered telemetry
- ordered per stream สำหรับ lifecycle
- explicit causal dependency สำหรับ cross-stream requirement

Gap handling:

```text
buffer bounded window
OR NACK_GAP + replay request
```

ห้าม apply required N+1 ก่อน N แบบเงียบ ๆ

---

## 8. Fencing

ทุก authority ที่อาจ stale มี:

```yaml
owner:
generation:
lease_token:
expires_at:
```

Core compare expected generation ใน transaction ก่อน commit

ใช้กับ scheduler, runtime attempt, effect attempt, delivery attempt, recovery worker

---

## 9. Runtime Attempt contract

Runtime Attempt เป็นชั้นสำคัญระหว่าง Scheduler กับ Hermes:

```yaml
runtime_attempt:
  id:
  ticket_id:
  task_id:
  job_id:
  quantum_id:
  adapter_id:
  adapter_epoch:
  generation:
  context_envelope_ref:
  execution_contract_ref:
  native_session_ref:
  native_run_ref:
  state:
  started_at:
  ended_at:
```

หนึ่ง Runtime Attempt อาจมีหลาย Model Runs/Tool Turns

---

## 10. Yield/Quantum capability

Adapter ต้องประกาศ granularity ที่ Core ควบคุมได้จริง:

```text
SINGLE_CALL
BOUNDED_TURNS
BOUNDED_RUNTIME
COOPERATIVE_CHECKPOINT
OPAQUE_RUN
```

พร้อม properties:

```yaml
max_turns_supported:
run_budget_supported:
cancel_support:
checkpoint_support:
model_call_visibility:
```

Scheduler ห้ามสมมุติ fairness ที่ละเอียดกว่าความสามารถของ runtime

---

## 11. Backpressure

Adapter รายงาน:

```yaml
accepting:
max_inflight:
current_inflight:
retry_after_ms:
```

durable queue หลักควรอยู่ใน Core ไม่ใช่ Adapter RAM

---

## 12. Transport security

แม้ local IPC ควรมี peer authentication เช่น named-pipe ACL / socket permission / authenticated localhost transport

Envelope bind:

- peer identity
- instance epoch
- manifest hash
- authorization reference

connect ได้ ≠ ได้ permission ทุก capability

---

## 13. Failure semantics

Timeout คือ “ไม่มี conclusive observation ในเวลา” ไม่ใช่ failure

หลัง timeout classify:

- definitely not started → retry possible
- idempotent external system → retry same key
- may have committed → reconcile
- authoritative terminal evidence → terminal

---

## 14. Acceptance criteria

- duplicate 100 frames → one semantic command
- crash/reconnect/replay converge
- changed hash under same identity rejected
- stream gaps ไม่ reorder silently
- stale owner commit ไม่ได้
- Adapter crash หลัง durable inbox ไม่ทำงานหาย
- RESULT loss replay ได้จน Core ACK_COMMITTED
- protocol semantics ใช้กับ Hermes/OpenClaw/future adapters เดิม
