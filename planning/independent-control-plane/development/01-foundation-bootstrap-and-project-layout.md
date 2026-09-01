# 01 — Foundation Bootstrap and Project Layout

> **สถานะเอกสาร:** Design Baseline / Living Specification  
> **ภาษาหลัก:** ไทย โดยใช้คำศัพท์เชิงสถาปัตยกรรมภาษาอังกฤษเป็น canonical term  
> **หลักการอ่าน:** คำว่า MUST / MUST NOT / SHOULD / MAY ใช้ในความหมายเชิง normative  
> **ขอบเขต:** CogentNexus เป็นโปรแกรมอิสระที่เป็น durable semantic control plane และเชื่อมระบบภายนอกผ่าน replaceable Adapters


## 1. เป้าหมาย

สร้าง skeleton ที่บังคับ architecture boundary ตั้งแต่ commit แรก เพื่อไม่ให้ feature code ลัดเข้า SQLite/Adapter/GUI โดยตรง

---

## 2. Repository layout

```text
pyproject.toml
src/cogentnexus/
  domain/
  application/
  persistence/
  transport/
  adapters/
  api/
  cli/
  gui/
tests/
  unit/
  integration/
  chaos/
  conformance/
docs/
  architecture/
  specifications/
  decisions/
runtime/   # gitignored
```

### `domain/`
entities/value objects/transition contracts ไม่มี network/UI dependency

### `application/`
use cases: admit, schedule, transition, authorize, reconcile, deliver

### `persistence/`
repositories/transactions/migrations/artifact store

### `transport/`
Adapter protocol/session/outbox/inbox

### `adapters/`
Hermes implementation; Core interfaces อยู่ด้านใน

### `api/cli/gui`
clients/surfaces

---

## 3. Dependency direction

```text
GUI/CLI/API
    ↓
Application Services
    ↓
Domain

Persistence/Transport/Adapters implement ports
```

Domain ห้าม import PySide6/Hermes/network packages

---

## 4. Python baseline

- current supported Python 3.x line ที่ packaging/test ecosystem เสถียรในเวลาพัฒนา
- `pyproject.toml`
- strict type checking ใน critical domain/protocol modules
- Ruff/formatter
- pytest
- Pydantic สำหรับ boundary schemas

หลีกเลี่ยง dependency จำนวนมากใน Core path

---

## 5. Configuration

แยก:

```text
semantic policy config
runtime/adapter config
secrets handles
operator preferences
GUI preferences
```

Secrets ไม่อยู่ config export ทั่วไป

Config schema version แยกจาก software release

---

## 6. Logging/telemetry contract

structured logs fields:

```text
trace_id
correlation_id
ticket_id
task_id
runtime_attempt_id
effect_id
delivery_id
adapter_id
event_type
```

ห้าม log secrets/raw user content โดย default

Instrumentation ต้อง behavior-neutral

---

## 7. Development safety rails

- persistence writes ผ่าน Unit of Work/transaction service เท่านั้น
- Adapter ไม่มี DB connection
- GUI ไม่มี repository write object
- effect execution interface ต้อง require Effect Intent
- delivery transport interface ต้อง require Delivery identity
- direct model call from random module ถูก lint/review ห้าม

---

## 8. Initial test scaffolding

สร้าง fake components ก่อน Hermes:

- Fake Adapter
- Fake Model Runtime
- Fake External Effect Target
- Fake Delivery Channel
- Crash injector
- deterministic clock/ID generator hooks สำหรับ tests

ทำให้ semantics พิสูจน์ได้โดยไม่พึ่ง external runtime

---

## 9. Early Windows considerations

- path normalization
- file locking/WAL behavior
- named pipe/localhost auth
- service vs desktop process
- process termination tests
- UTF-8/PowerShell integration
- PySide6 packaging smoke test ช่วงต้นเพื่อรู้ native dependency risk

---

## 10. Exit criteria

- `python -m cogentnexus` headless starts/stops clean
- migration framework creates DB
- domain package importable without adapter/gui deps
- fake adapter handshake works
- structured logging contains correlation IDs
- unit tests enforce dependency rules
