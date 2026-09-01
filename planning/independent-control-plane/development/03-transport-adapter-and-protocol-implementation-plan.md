# 03 — Transport, Adapter SDK, and Protocol Implementation Plan

> **สถานะเอกสาร:** Design Baseline / Living Specification  
> **ภาษาหลัก:** ไทย โดยใช้คำศัพท์เชิงสถาปัตยกรรมภาษาอังกฤษเป็น canonical term  
> **หลักการอ่าน:** คำว่า MUST / MUST NOT / SHOULD / MAY ใช้ในความหมายเชิง normative  
> **ขอบเขต:** CogentNexus เป็นโปรแกรมอิสระที่เป็น durable semantic control plane และเชื่อมระบบภายนอกผ่าน replaceable Adapters


## 1. Objective

สร้าง Adapter SDK/transport ที่ทำให้ Hermes/OpenClaw/future runtime ใช้ semantic contract เดียวกันและ crash/replay ได้

---

## 2. Protocol implementation order

### A — Envelope and identity
Pydantic schemas + canonical hashing + message/command/stream identities

### B — Durable outbox/inbox
Core persistence ก่อน network transport จริง

### C — ACK stages
ACK_RECEIVED, ACK_DURABLE, RESULT, ACK_COMMITTED

### D — Replay/reconnect
HELLO, protocol/schema negotiation, watermark, missing range replay

### E — Ordering/gap
per-stream sequence, bounded buffer/NACK_GAP

### F — Fencing
adapter epoch + execution generation/token

### G — FLOW/backpressure
capacity state

### H — Security
peer auth + manifest binding

---

## 3. Adapter SDK interfaces

```text
AdapterLifecycle
CapabilityProvider
RuntimeExecutor
EffectExecutor
DeliveryTransport
IngressSource
EvidenceCollector
Reconciler
```

Adapter implementation เลือก interfaces ที่รองรับ

---

## 4. Conformance suite

ทุก Adapter ต้องผ่าน generic tests:

- duplicate message
- hash mismatch
- reconnect replay
- stale epoch/fence
- gap handling
- capacity exhausted
- RESULT replay until ACK_COMMITTED
- command conflict
- unsupported schema

---

## 5. Runtime Attempt protocol

Add commands/events:

```text
runtime.execute
runtime.cancel
runtime.observe
runtime.result
runtime.checkpoint
runtime.capacity
```

Result ต้อง structured; raw native output เป็น artifact ref

---

## 6. Yield capability

Manifest declares yield granularity + budget/cancel/checkpoint support

Tests ต้องพิสูจน์จริง ไม่ใช่เชื่อ declaration

---

## 7. Local transport choice

V1 อาจเริ่ม authenticated localhost transport หรือ Windows named pipe โดย abstraction ไม่เปลี่ยน protocol semantics

Criteria:

- reliable reconnect
- peer identity
- easy test/fault injection
- packaging simplicity

หลีกเลี่ยง premature custom binary framing; JSON/msgpack with canonical hashing เพียงพอถ้าคุม schema

---

## 8. Chaos tests

kill process boundary:

1. Core persisted command before send
2. after send before ACK
3. Adapter after durable inbox
4. after external action before RESULT
5. RESULT sent/lost
6. Core committed before ACK_COMMITTED

---

## 9. Exit gate

Fake Adapter และ second reference Adapter ต้องผ่าน conformance เดียวกันก่อน Hermes เพื่อพิสูจน์ Core ไม่ Hermes-specific
