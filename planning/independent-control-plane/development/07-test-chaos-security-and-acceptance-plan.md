# 07 — Test, Chaos, Security, and Acceptance Plan

> **สถานะเอกสาร:** Design Baseline / Living Specification  
> **ภาษาหลัก:** ไทย โดยใช้คำศัพท์เชิงสถาปัตยกรรมภาษาอังกฤษเป็น canonical term  
> **หลักการอ่าน:** คำว่า MUST / MUST NOT / SHOULD / MAY ใช้ในความหมายเชิง normative  
> **ขอบเขต:** CogentNexus เป็นโปรแกรมอิสระที่เป็น durable semantic control plane และเชื่อมระบบภายนอกผ่าน replaceable Adapters


## 1. Testing philosophy

Agentic correctness ต้องพิสูจน์ทั้ง happy path และ **negative/failure paths** โดยเฉพาะ stale owner, duplicate, ambiguous side effect และ forbidden capability

---

## 2. Test pyramid

### Unit
state transitions, guards, hashing, policy, scheduler scoring

### Property-based
idempotency, sequence/replay, state invariants, random Task graph

### Integration
SQLite transactions, artifact store, fake adapters

### Conformance
Adapter protocol/capability contracts

### Chaos
process termination/network loss/duplicate/reorder

### End-to-End
real Hermes + model + user channel where qualified

---

## 3. Core invariant test suite

ต้องมี reusable assertions:

- accepted Ticket never disappears
- every semantic revision has event
- stale generation cannot commit
- terminal state immutable against stale replay
- effect cannot execute without authorized intent
- delivery confirmed cannot revert
- context compaction preserves obligations/fences
- Adapter cannot write Core authority

---

## 4. Crash matrix

Inject crash:

```text
before persist
persist before send
send before ACK
Adapter durable inbox before execute
external commit before result persist
result persist before send
Core commit before ACK_COMMITTED
delivery external success before receipt
migration mid-step
artifact temp/write/rename/DB-ref boundaries
```

Expected recovery documented per boundary

---

## 5. Negative capability tests

- read grant → write attempt rejected
- repo A grant → repo B rejected
- branch scope mismatch
- stale lease
- altered parameter hash
- expired/revoked confirmation
- raw shell attempts forbidden operation
- manifest capability removed mid-task

---

## 6. Scheduler tests

- one model slot invariant
- fairness/starvation bounds
- pause/resume
- cancellation propagation
- WAITING_IO release
- budget exhaustion
- runtime OPAQUE_RUN policy
- priority inversion bounded

---

## 7. Context tests

- deterministic same revision → same envelope hash where inputs stable
- mandatory over-budget blocks
- authority conflict resolution
- no sibling leakage
- secret exclusion
- context audit completeness

---

## 8. Security review areas

- local IPC authentication
- capability least privilege
- secret handles
- prompt injection cannot mint grants
- GUI authorization
- artifact path traversal
- command normalization/shell injection
- protocol payload hash/identity reuse
- replay attacks/stale epochs

---

## 9. Hermes qualification acceptance

Capability-specific; no “Hermes supported” boolean เดียว

High-risk requires:

- positive path
- negative bypass path
- crash ambiguity/reconciliation path
- update requalification path

---

## 10. Final V1 scenario set

1. casual message fast path
2. long analytical Task with one model
3. new interactive input while long Task active
4. Hermes crash + semantic resume
5. Core crash during runtime attempt
6. effect external commit + lost RESULT
7. ambiguous message send + reconciliation
8. pause/resume after committed local effect
9. cancellation revokes future action
10. Hermes version/change disables unsafe capability
11. GUI crash/reconnect
12. backup/restore then continue Task

---

## 11. Evidence gate

Release gate ต้องเก็บ:

- exact commit SHA
- schema revision
- protocol revision
- package fingerprint
- Hermes/runtime identity
- test/chaos reports
- capability qualification manifest
- migration/backup proof

ไม่มี evidence = ยังไม่ผ่าน gate
