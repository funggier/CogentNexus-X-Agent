# 02 — State, Persistence, and Recovery Implementation Plan

> **สถานะเอกสาร:** Design Baseline / Living Specification  
> **ภาษาหลัก:** ไทย โดยใช้คำศัพท์เชิงสถาปัตยกรรมภาษาอังกฤษเป็น canonical term  
> **หลักการอ่าน:** คำว่า MUST / MUST NOT / SHOULD / MAY ใช้ในความหมายเชิง normative  
> **ขอบเขต:** CogentNexus เป็นโปรแกรมอิสระที่เป็น durable semantic control plane และเชื่อมระบบภายนอกผ่าน replaceable Adapters


## 1. Objective

สร้าง durable truth ก่อนเชื่อม Hermes เพื่อให้ runtime เป็น replaceable consumer ไม่ใช่ฐานความจำของระบบ

---

## 2. Phase sequence

### Phase 2.1 — IDs and value objects
Implement stable IDs: Ticket, Task, Event, Artifact, Job, Quantum, RuntimeAttempt, Effect, Delivery, Principal, Session

### Phase 2.2 — Ticket/Task domain
- recursive parent/child
- Phase/Status/Outcome
- blockers/obligations
- transition guard API

### Phase 2.3 — Event transaction primitive
API ประมาณ:

```text
with semantic_transaction(aggregate, expected_revision):
    mutate_state()
    append_event()
```

optimistic revision conflict fail deterministically

### Phase 2.4 — SQLite schema
สร้าง table families ตาม knowledge/06 โดย migration files แบบ immutable/checksummed

### Phase 2.5 — Artifact store
crash-safe temp→hash→atomic rename→DB ref

### Phase 2.6 — Knowledge/Evidence
minimal typed records + refs + authority metadata

### Phase 2.7 — Snapshots/read models
derived only

### Phase 2.8 — Startup recovery scanner
classify stale/incomplete state โดยไม่เรียก model

---

## 3. Minimal schema priorities

P0 tables:

```text
schema_migrations
system_meta
tickets
tasks
task_dependencies
obligations
events
artifacts
artifact_links
scheduler_jobs
runtime_attempts
model_runs
```

P1:

```text
knowledge_records
evidence_records
context_manifests
adapters/capabilities/grants
effects/deliveries
ipc tables
```

---

## 4. Transaction tests

ใช้ fault injection หลังทุก step:

- before transaction
- after state mutation before event insert
- after event insert before commit
- after commit before outward ACK

Expected: rollback ทั้ง state+event หรือ commit ทั้งคู่

---

## 5. Migration strategy

- migration number + semantic ID + checksum
- test upgrade จากทุก supported prior revision
- app newer/older compatibility checks
- no ad-hoc ALTER TABLE at normal service startup modules
- backup/integrity preflight ก่อน destructive migration

---

## 6. Recovery classifier

Startup ไม่ “retry everything” แต่ classify:

```text
DEFINITELY_NOT_STARTED
STARTED_OUTCOME_UNKNOWN
OUTPUT_DURABLE_NOT_COMMITTED
COMMITTED
STALE_OWNER
WAITING_EXTERNAL
RECONCILIATION_REQUIRED
```

แต่ละ class มี deterministic transition

---

## 7. Data retention

กำหนด TTL/retention สำหรับ:

- IPC dedupe
- raw model payload
- telemetry
- snapshots
- temp artifacts

semantic events/effects/delivery evidence retention ยาวกว่า

---

## 8. Exit gate

- power-loss simulation passes
- snapshot deletion/rebuild passes
- duplicate admission idempotent
- revisions/events 1:1 invariant
- backup/restore smoke
- future schema reject
- recovery scan produces no model calls
