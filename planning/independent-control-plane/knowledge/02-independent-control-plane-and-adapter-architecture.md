# 02 — Independent Control Plane and Adapter Architecture

> **สถานะเอกสาร:** Design Baseline / Living Specification  
> **ภาษาหลัก:** ไทย โดยใช้คำศัพท์เชิงสถาปัตยกรรมภาษาอังกฤษเป็น canonical term  
> **หลักการอ่าน:** คำว่า MUST / MUST NOT / SHOULD / MAY ใช้ในความหมายเชิง normative  
> **ขอบเขต:** CogentNexus เป็นโปรแกรมอิสระที่เป็น durable semantic control plane และเชื่อมระบบภายนอกผ่าน replaceable Adapters


## 1. Architectural shape

CogentNexus ควรเป็นโปรแกรมที่ “ยืนได้ด้วยตัวเอง” และมีมือยื่นไปเชื่อม runtime/tool/channel ภายนอก

```text
                         User / System
                              │
                              ▼
                    ┌──────────────────┐
                    │ CogentNexus Core │
                    │ Durable Authority │
                    └───────┬──────────┘
                            │ Adapter Transport
          ┌─────────────────┼──────────────────┐
          ▼                 ▼                  ▼
     Hermes Adapter    OpenClaw Adapter    Action Adapter
          │                 │                  │
        Hermes           OpenClaw        Git/OS/Browser/etc.
```

Core ไม่ควร import private Hermes internals หรือถือ Hermes session เป็น durable truth

---

## 2. ความรับผิดชอบของ Core

Core owns:

- Ticket admission และ raw intent
- recursive Tasks และ semantic transitions
- Scheduler/Compute Envelope
- Context Compiler และ Context Audit
- Knowledge/Evidence/Artifact references
- Authorization/Policy/Capability selection
- Effect Intent Ledger
- Delivery intent/confirmation
- Recovery, event history, snapshots
- Adapter registry/qualification state
- operator API/commands

Core ห้าม delegate semantic authority เหล่านี้ให้ Adapter

---

## 3. ความรับผิดชอบของ Adapter

Adapter owns:

- native connection/API/tool integration
- translation ระหว่าง semantic protocol กับ native system
- local spool เมื่อ transport ต้อง ACK เร็ว
- enforcing granted capability/scope/fence ที่ boundary
- executing explicitly authorized operations
- gathering native evidence/receipts
- reporting capability manifest/capacity
- bounded transport micro-retries ตาม Core policy

Adapter MUST NOT:

- สร้าง permission ใหม่เอง
- broaden scope
- mark Ticket complete
- silently retry ambiguous semantic side effects
- mutate Core DB tables โดยตรง

---

## 4. Adapter categories

### Cognitive Runtime Adapter
เช่น Hermes, OpenClaw, direct LLM runtime; ทำ analysis/planning/execution passes

### Action Adapter
เช่น GitHub, filesystem, OS, browser; ทำ deterministic/external side effect

### Channel Adapter
รับ ingress และส่ง delivery เช่น dashboard/Discord/CLI/web

### Model Provider Adapter
ถ้าต้องการแยก model invocation ออกจาก agent runtime

หนึ่ง Adapter อาจทำหลาย role ใน V1 แต่ Core contract ควรแยก capability ตาม semantic role

---

## 5. Runtime Bridge abstraction

Generic interface เชิง semantic:

```text
HELLO / NEGOTIATE
CAPABILITIES
HEALTH
FLOW
EXECUTE
OBSERVE
CANCEL
RESULT
RECONCILE
```

Execution Contract ควรมี:

```yaml
ticket_id:
task_id:
job_id:
quantum_id:
runtime_attempt_id:
generation:
idempotency_key:
context_envelope_ref:
capability_grants:
forbidden_capabilities:
budget:
stop_conditions:
expected_output_contract:
```

---

## 6. Capability-based compatibility

อย่าใช้เพียง `Hermes version == X`

Adapter ต้อง publish manifest:

```yaml
runtime_family: hermes
adapter_release:
protocol_range:
manifest_revision:
capabilities:
  context_selection: ...
  bounded_turns: ...
  cancellation: ...
  tool_preflight_gate: ...
  tool_observation: ...
  delivery_receipt: ...
```

Core classify integration:

```text
FULL
DEGRADED
INCOMPATIBLE
```

และ capability แต่ละตัวมี qualification state แยก

---

## 7. Passthrough vs Managed Mode

### Native/Passthrough
CNX ไม่เป็น authority ของ channel นั้น; Hermes ทำงาน native ตามปกติ

### Managed
Ingress ถูก admit เป็น Ticket ก่อน, Core สร้าง Context Envelope/Execution Contract, runtime output ถูก capture เป็น intermediate/candidate result และ delivery ถูก Core ตัดสิน

ข้อดีของการมีสอง mode คือ Hermes ยังใช้งานได้หาก CNX ปิด และการอัปเดต Hermes ไม่ควรทำให้ native workflow พัง

---

## 8. Failure containment

- GUI crash → Core ไม่ตาย
- Hermes crash → Tasks remain, runtime attempt recovery
- Adapter crash → Core outbox/replay
- Core crash → SQLite/event/artifact recovery
- protocol incompatibility → managed capability disabled, state preserved
- runtime session lost → semantic resume ผ่าน new binding

---

## 9. ความเสี่ยงที่ต้องระวัง

### Adapter becomes second Core
เกิดเมื่อ Adapter เก็บ state สำคัญที่ Core ไม่มี copy/identity

### Private API coupling
ใช้ private Hermes function/DB schema แล้ว update พัง

### Hidden authority
runtime มี raw shell/native tools ที่ bypass capability policy

### Hidden delivery
runtime ส่ง user-facing output ก่อน Core validate

### Hidden retry
runtime retry external mutation เองโดยไม่มี effect identity

---

## 10. Architectural acceptance

- ถอด Hermes แล้ว Core ยัง inspect/pause/cancel/recover Ticket ได้
- เปลี่ยน Hermes Adapter implementation โดยไม่ migrate Ticket ontology
- Adapter ไม่มี SQL write access ต่อ Core authority tables
- managed feature ถูกเปิดเฉพาะ capability ที่ qualified
- fallback/passthrough ไม่เปลี่ยน durable state โดยไม่ตั้งใจ
- runtime binding เป็น reference ไม่ใช่ source of truth
