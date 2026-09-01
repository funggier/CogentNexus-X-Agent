# 06 — GUI Control, Monitoring, and Operator UX Plan

> **สถานะเอกสาร:** Design Baseline / Living Specification  
> **ภาษาหลัก:** ไทย โดยใช้คำศัพท์เชิงสถาปัตยกรรมภาษาอังกฤษเป็น canonical term  
> **หลักการอ่าน:** คำว่า MUST / MUST NOT / SHOULD / MAY ใช้ในความหมายเชิง normative  
> **ขอบเขต:** CogentNexus เป็นโปรแกรมอิสระที่เป็น durable semantic control plane และเชื่อมระบบภายนอกผ่าน replaceable Adapters


## 1. Objective

สร้าง GUI ที่ทำให้ผู้ใช้ “เห็นระบบคิดเชิงสถานะ” และควบคุมได้โดยไม่ลด reliability โดย GUI เป็น client ของ Core เท่านั้น

---

## 2. Phased roadmap

### GUI-0 — Headless first
CLI/API ครบพอ inspect/control ก่อน

### GUI-1 — Read-only Monitor
Overview, Tickets, Tasks, Scheduler, Adapters, Effects, Deliveries

### GUI-2 — Safe Control
pause/resume/cancel, adapter enable/disable, reconcile commands

### GUI-3 — Authorization UX
scoped approval/confirmation with hashes and expiry

### GUI-4 — Deep Diagnostics
Context Audit, event timeline, artifact/evidence viewer, migration/recovery panels

### GUI-5 — Advanced visualization
Task dependency graph, live runtime attempt map, budget charts

---

## 3. Technology

PySide6 + Qt Model/View; Core process แยก

Communication ผ่าน authenticated local Control API/event stream

GUI state เป็น projection/cache; reconnect ต้อง re-query authoritative snapshot

---

## 4. Main screens

### Overview
- Core health
- one model slot owner
- running/waiting/blocked counts
- critical OUTCOME_UNKNOWN
- Adapter health/qualification

### Ticket Detail
- raw intent + interpreted intent
- current outcome
- Task tree
- obligations
- delivery state

### Task Detail
- semantic Phase/Status
- dependencies
- current checkpoint
- evidence/unknowns
- scheduler job
- runtime attempts

### Scheduler
- queue classes
- waiting age
- quantum owner
- budgets

### Adapter/Capability
- manifest revision/hash
- qualification status
- native mapping
- last health/error

### Effect Ledger
- effect class/replay policy
- grant/scope
- attempts/evidence
- reconciliation action

### Delivery
- target session/generation
- attempts/confirmation

---

## 5. Control semantics

Button ไม่เปลี่ยน state ตรง ๆ

```text
Click Pause
→ POST command
→ Core authorize/validate
→ durable transition/event
→ GUI receives event
```

ใช้ command receipts เพื่อแสดง accepted/rejected/committed ต่างกัน

---

## 6. Safety UX

สำหรับ destructive action แสดง:

- semantic capability
- target scope
- effect ID
- replay policy
- expected evidence
- irreversible warning
- confirmation scope hash

OUTCOME_UNKNOWN UI default action = `Reconcile`, ไม่ใช่ `Retry`

---

## 7. Monitoring quality

Avoid “green means good” แบบคลุมเครือ

สถานะควรมี evidence/ref เช่น:

```text
Hermes: ONLINE (last heartbeat 2.1s)
repo.push: QUALIFIED_MUTATING (mapping rev 5)
Delivery D-18: CONFIRMED (native msg id ...)
Effect E-9: OUTCOME_UNKNOWN (reconcile required)
```

---

## 8. Performance

- event-driven updates
- virtualized tables
- pagination for events/logs
- lazy artifact loading
- no huge transcript rendering by default
- background decode/hash outside UI thread

---

## 9. Operator modes

### Basic
Overview + current work + simple pause/cancel

### Advanced
Scheduler/effects/context/capability

### Diagnostic
events, revisions, raw protocol metadata, recovery scan

ลด cognitive overload โดยไม่ซ่อน critical uncertainty

---

## 10. Acceptance

- GUI process kill has zero Core effect
- stale GUI command rejected by revision/generation where required
- reconnect converges
- no secret leakage in UI logs/export default
- every mutating GUI action maps to auditable Core command
- headless Core acceptance suite runs without Qt installed
