# 00 — Development Strategy and V1 Boundary

> **สถานะเอกสาร:** Design Baseline / Living Specification  
> **ภาษาหลัก:** ไทย โดยใช้คำศัพท์เชิงสถาปัตยกรรมภาษาอังกฤษเป็น canonical term  
> **หลักการอ่าน:** คำว่า MUST / MUST NOT / SHOULD / MAY ใช้ในความหมายเชิง normative  
> **ขอบเขต:** CogentNexus เป็นโปรแกรมอิสระที่เป็น durable semantic control plane และเชื่อมระบบภายนอกผ่าน replaceable Adapters


## 1. Development philosophy

พัฒนา CogentNexus แบบ **correctness-first, small-model-first, adapter-independent** โดยสร้าง durable control plane ก่อน GUI/advanced intelligence

หลัก:

- TDD สำหรับ state/protocol/recovery semantics
- evidence gates แทน “ดูเหมือนทำงานได้”
- shadow/qualification ก่อน enforce external runtime
- freeze exact candidate ก่อน acceptance
- production source fix → rerun relevant validation
- ไม่ patch live candidate แล้วถือว่าเป็น candidate เดิม

---

## 2. V1 objective

พิสูจน์ end-to-end invariant ต่อไปนี้ด้วย Python Core + SQLite + Hermes Adapter หนึ่งตัว:

```text
User ingress
→ durable Ticket
→ recursive Task
→ bounded Context Envelope
→ scheduler model slot = 1
→ Hermes Runtime Attempt
→ evidence/artifact/result capture
→ state transition/verification
→ durable Delivery
→ confirmed user-visible output
```

พร้อม kill/restart recovery และ no duplicate side effect

---

## 3. V1 MUST HAVE

- headless Python Core
- SQLite WAL + migrations
- content-addressed artifact store
- Ticket/Task/Transition/Event model
- Knowledge/Evidence/Obligation minimal records
- Context Compiler minimal deterministic pipeline
- single-model scheduler + durable model lease
- Runtime Attempt abstraction
- Adapter transport with durable inbox/outbox/replay/fence
- Hermes Adapter discovery + bounded qualified capabilities
- Effect Intent Ledger + reconciliation foundation
- Delivery ledger/confirmation
- CLI/operator diagnostics
- local control API
- basic GUI monitor/control shell (can be later V1.x if core acceptance first)

---

## 4. V1 SHOULD HAVE

- PySide6 overview/Tickets/Tasks/Adapters/Effects/Delivery screens
- Context Audit view
- backup/integrity operation
- capability qualification report
- chaos harness

---

## 5. V1 NON-GOALS

- distributed multi-machine scheduler
- vector DB as mandatory dependency
- multi-model consensus
- autonomous model routing marketplace
- full browser/robotics automation
- arbitrary remote GUI control
- plugin ecosystem marketplace
- full event sourcing
- complex role hierarchy of persistent AI agents

---

## 6. Development gates

### Gate A — Domain/State
ontology/transition invariants tested without runtime

### Gate B — Persistence
crash-safe transaction/event/artifact rules

### Gate C — Scheduler
one-slot fairness/recovery/budget

### Gate D — Transport
replay/dedupe/fencing chaos tests

### Gate E — Side Effects
Effect Intent + negative capability tests

### Gate F — Hermes Read-only
manifest/discovery/context/result capture

### Gate G — Reversible Mutation
workspace edit/test/commit qualification

### Gate H — End-to-End Managed Flow
Ticket → Hermes → durable result → delivery

### Gate I — Crash/Upgrade Recovery
kill Core/Adapter/Hermes boundaries

### Gate J — GUI Control/Monitor
no authority bypass

---

## 7. Key risks to manage early

1. Hermes public seam ไม่พอ strict ticket-first/tool fence
2. runtime run granularity ใหญ่เกิน scheduler quantum
3. SQLite schema โตเร็วก่อน ontology stable
4. GUI ถูกพัฒนาเร็วเกิน Core API
5. raw shell bypass policy
6. artifact/DB crash atomicity
7. hidden native delivery จาก Hermes

ทุก risk ต้องมี qualification experiment ก่อนลงทุน implementation ใหญ่

---

## 8. Definition of V1 success

V1 ถือว่าพร้อมเมื่อสามารถสาธิตบนเครื่อง resource จำกัดว่า:

- new input admitted ขณะ model busy
- one small model ทำ long workflow สลับ interactive work โดยไม่ starvation
- kill Hermes กลางงาน → semantic resume
- kill Core หลัง effect commit ก่อน receipt → reconcile/no duplicate
- restart GUI ไม่มีผลต่อ Core
- Hermes update ที่ capability changed → managed capability degrade/disable อย่าง deterministic
- final user response ถูก delivery-confirmed ไม่ใช่แค่ model generated
