# 09 — Python Core Service and GUI Control/Monitoring Architecture

> **สถานะเอกสาร:** Design Baseline / Living Specification  
> **ภาษาหลัก:** ไทย โดยใช้คำศัพท์เชิงสถาปัตยกรรมภาษาอังกฤษเป็น canonical term  
> **หลักการอ่าน:** คำว่า MUST / MUST NOT / SHOULD / MAY ใช้ในความหมายเชิง normative  
> **ขอบเขต:** CogentNexus เป็นโปรแกรมอิสระที่เป็น durable semantic control plane และเชื่อมระบบภายนอกผ่าน replaceable Adapters


## 1. Decision

CogentNexus Core SHOULD ใช้ Python เป็น control-plane implementation language สำหรับ V1/V2 เพราะ workload หลักคือ state machines, SQLite, IPC, scheduling, policy, hashing, process/network integration และ GUI orchestration มากกว่า numerical compute

Python ไม่จำเป็นต้องทำ LLM inference หรือ low-level native operations เอง; runtime/adapter ภายนอกทำสิ่งเหล่านั้น

---

## 2. Why Python fits

- `asyncio` เหมาะกับ I/O-heavy orchestration
- stdlib `sqlite3`, `hashlib`, `pathlib`, `logging`, `subprocess`
- Pydantic/typed dataclasses ช่วย protocol/schema validation
- test ecosystem แข็ง
- Windows packaging ทำได้
- PySide6/Qt เหมาะกับ desktop monitor/control GUI
- developer velocity สูง เหมาะกับ architecture ที่ยังต้องพิสูจน์หลาย failure semantics

Performance bottleneck หลักคาดว่าจะเป็น model/tool/network ไม่ใช่ Python instruction throughput

---

## 3. Process architecture

แนะนำแยก process แม้ product ดูเป็นโปรแกรมเดียว:

```text
CogentNexus Launcher
   ├─ cnx-core     Python headless service
   └─ cnx-gui      PySide6 control/monitor client

cnx-core
   ├─ SQLite + Artifact Store
   ├─ Scheduler
   ├─ Context Compiler
   ├─ Adapter Transport
   ├─ Policy/Effects/Delivery
   ├─ Recovery/Health
   └─ Local Control API
```

GUI crash ไม่ควรหยุด Core

---

## 4. Core concurrency model

หนึ่ง `asyncio` event loop สำหรับ orchestration services:

```text
Ingress service
Scheduler loop
Adapter sessions
Delivery worker
Recovery/reconciliation workers
Health monitor
Control API
Event broadcaster
```

DB write path serialized หนึ่ง logical writer; read-only UI queries ใช้ reader connections

CPU-heavy hashing/large transforms ใช้ bounded worker thread/process โดยไม่ block event loop

---

## 5. Internal package layout

```text
src/cogentnexus/
  core/
    tickets/
    tasks/
    transitions/
    context/
    scheduler/
    policy/
    effects/
    delivery/
    recovery/
  persistence/
    sqlite/
    migrations/
    artifacts/
  transport/
  adapters/
    hermes/
  api/
  cli/
  gui/
```

อาจแยก packages ภายหลังเมื่อ boundary stable; V1 ไม่ต้อง premature multi-repo

---

## 6. Data modeling

Pydantic เหมาะสำหรับ boundary models/protocol payload และ schema generation

Core domain logic ไม่ควรผูก Pydantic ไปทุกชั้น; ใช้ typed immutable/value objects ตามเหมาะสม

Explicit SQL/repository layer เหมาะกว่า heavy ORM สำหรับ transaction semantics ที่ต้อง inspect ได้

---

## 7. GUI philosophy

GUI มีสองบทบาท:

### Monitor
- health/runtime/adapters
- Ticket/Task graph
- scheduler queue/model slot
- Context Audit
- evidence/artifacts
- capability/authorization
- effect/delivery ledger
- recovery/reconciliation
- budgets/metrics

### Control
- pause/resume/cancel Task
- change allowed priority/policy knobs
- approve/deny explicit authorization requests
- retry/reconcile ผ่าน Core command
- enable/disable Adapter/capability
- start/stop managed mode

GUI **MUST NOT** execute SQL mutations ต่อ authority tables

---

## 8. Local Control API

GUI/CLI ใช้ API เดียว เช่น:

```text
GET  /health
GET  /tickets
GET  /tasks/{task_id}
GET  /scheduler
GET  /adapters
GET  /effects
GET  /deliveries
GET  /context/{task_id}/audit
POST /commands/tasks/{task_id}/pause
POST /commands/tasks/{task_id}/resume
POST /commands/tasks/{task_id}/cancel
POST /commands/effects/{effect_id}/reconcile
POST /commands/authorizations/{grant_id}/approve
```

ทุก mutation endpoint ผ่าน authorization + state transition + event append

Transport อาจเป็น local named pipe / localhost authenticated HTTP; semantic API ไม่ควรผูกกับ GUI framework

---

## 9. GUI information architecture

### Overview
Core health, model slot, active/waiting/blocked, Adapter health, critical ambiguity

### Tickets
intent, state/outcome, Tasks, delivery

### Task Graph
recursive DAG/tree + dependencies + obligations

### Scheduler
priority classes, waiting age, current quantum, budgets

### Runtime/Adapters
manifest revision, capabilities, qualification, epochs, in-flight attempts

### Effects
authorized/executing/outcome_unknown/reconciliation/evidence

### Deliveries
staged/sending/confirmed/unknown, session generation

### Context Inspector
included/excluded records, authority, token budget

### Operations
migration, backup, integrity, recovery scan, event stream

---

## 10. Event-driven GUI updates

GUI ไม่ควร poll ทุก table ตลอด

Core publish read-only event stream เช่น:

```text
ticket.changed
task.changed
scheduler.changed
adapter.health_changed
effect.changed
delivery.changed
alert.raised
```

GUI เมื่อ reconnect ใช้ snapshot/query แล้วตามด้วย event cursor เพื่อ converge

---

## 11. Operator safety UX

- destructive commands แสดง scope + effect identity + expected evidence
- confirmation bind `scope_hash`; parameter เปลี่ยนต้อง confirm ใหม่
- OUTCOME_UNKNOWN แสดงเด่นและเสนอ reconcile ไม่ใช่ “Retry” เป็น default
- stale/disabled capability แสดงเหตุผล
- GUI ต้อง distinguish “response generated” vs “delivered confirmed”

---

## 12. Packaging

ภายหลัง package Windows executable ผ่าน PyInstaller/Nuitka หรือ installer ที่ bundle runtime

อย่าให้ bundler constraints กำหนด architecture ก่อน correctness stable

Launcher ทำให้ผู้ใช้รู้สึกเป็นแอปเดียว แม้ Core/GUI เป็นคนละ process

---

## 13. สิ่งที่ต้องระวัง

- GUI thread block จาก DB/network
- GUI direct write bypass event/guards
- reconnect แล้ว render stale state
- exposing raw secrets/log payload
- background thread ownership ของ SQLite ไม่ชัด
- packaging native Qt/plugins ซับซ้อน; ควรพิสูจน์ early smoke build
- Windows service lifecycle กับ desktop session ต้องแยก responsibility

---

## 14. Acceptance

- ปิด GUI แล้ว Core ทำงานต่อ
- เปิด GUI ใหม่ reconstruct authoritative state ได้
- GUI command ทุกอันมี event/authorization trail
- ไม่มี direct DB mutation path จาก GUI
- Core headless tests ผ่านโดยไม่ import PySide6
- GUI แสดง Task semantic state กับ Scheduler state แยกกัน
- operator เห็น OUTCOME_UNKNOWN/blocked/recovery state ชัด
