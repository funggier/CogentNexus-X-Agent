# 06 — Persistence, Event Log, and Artifact Store

> **สถานะเอกสาร:** Design Baseline / Living Specification  
> **ภาษาหลัก:** ไทย โดยใช้คำศัพท์เชิงสถาปัตยกรรมภาษาอังกฤษเป็น canonical term  
> **หลักการอ่าน:** คำว่า MUST / MUST NOT / SHOULD / MAY ใช้ในความหมายเชิง normative  
> **ขอบเขต:** CogentNexus เป็นโปรแกรมอิสระที่เป็น durable semantic control plane และเชื่อมระบบภายนอกผ่าน replaceable Adapters


## 1. เป้าหมาย

Physical persistence คือ **memory of fact** ของ CogentNexus ไม่ใช่ conversational memory

หาก model/Adapter/GUI/process/machine หาย DB+artifacts ต้องตอบว่า intent/state/authority/effect/delivery อะไรเป็นจริง

---

## 2. Storage philosophy

ใช้ hybrid:

```text
Current State Tables   — operational authority
Append-only Event Log  — causation/audit/recovery evidence
Derived Snapshots      — acceleration/compact continuation
Artifact Store         — immutable large content
```

ไม่เริ่มด้วย full event sourcing เพราะ reducer/migration/replay complexity สูงเกินประโยชน์ V1

---

## 3. SQLite operating profile

```sql
PRAGMA journal_mode=WAL;
PRAGMA foreign_keys=ON;
PRAGMA busy_timeout=5000;
PRAGMA synchronous=FULL;
```

หนึ่ง logical writer, many readers, short transactions

Adapter ห้าม mutate tables โดยตรง

---

## 4. Canonical table families

```text
schema_migrations / system_meta
adapters / adapter_capabilities
principals / sessions
ingress_messages / tickets
tasks / task_dependencies / obligations
knowledge_records / claims / evidence_records
artifacts / artifact_links
context_manifests
scheduler_jobs / model_slot
runtime_attempts / model_runs
authorization_grants
effect_intents / effect_attempts
deliveries / delivery_attempts
ipc_outbox / ipc_inbox
events / snapshots
```

ทุก table ที่เป็น semantic aggregate และมี transition ควรมี `current_revision`

---

## 5. Transaction invariant

Core-controlled semantic transition:

```text
CURRENT STATE MUTATION
+
EVENT APPEND
```

ใน transaction เดียว

ห้ามมี state changed without event หรือ event claim without committed state

---

## 6. Key transaction boundaries

### Admission
- dedupe ingress
- create/resolve Ticket
- store intent refs
- append `ticket.accepted`
- enqueue initial Task/Job
- commit ก่อน report accepted

### Scheduler claim
- verify runnable/budget
- claim job+model slot
- increment generation
- append event
- commit ก่อน model/runtime call

### Runtime/model start
- create attempt/run record
- append start event
- commit
- external call

### Finish
- persist output artifact/ref/hash
- validate generation
- transition run/Task checkpoint
- release slot
- append event

### Effect prepare
- validate grant
- create effect identity/idempotency
- claim attempt
- append event
- commit ก่อน mutation

### Delivery stage/confirm
- create Delivery + payload hash + target generation
- send after commit
- confirmation transaction validates owner/generation/evidence

---

## 7. Never hold transaction across external call

ห้ามคร่อม:

- LLM/provider
- HTTP
- Adapter IPC
- subprocess
- browser automation
- large filesystem I/O
- user confirmation

State machine ครอบ external call แทน

---

## 8. Event log

Fields:

```text
event_id
aggregate_type
aggregate_id
aggregate_revision
event_type
causation_id
correlation_id
actor_ref
payload_ref
payload_hash
created_at
```

unique aggregate revision ป้องกัน concurrent semantic commit conflict

---

## 9. Artifact store

Content-addressed:

```text
runtime/
  core.sqlite3
  artifacts/sha256/ab/abcdef...
```

Crash-safe write protocol:

1. write temp file
2. flush/fsync ตาม durability policy
3. compute/verify SHA-256
4. atomic rename ไป content-addressed path
5. DB transaction reference artifact
6. orphan temp/content file ที่ไม่มี DB ref GC ภายหลัง

DB ห้าม reference non-durable artifact

---

## 10. Knowledge/Evidence persistence

`knowledge_records` แยก epistemic type และ source/provenance

`evidence_records` อ้าง artifact/observation + authority/freshness

`obligations` ต้อง durable และ independently queryable เพื่อไม่หายตอน context compaction

`context_manifests` เก็บ compiler inputs/revision/hash/audit refs ไม่ต้องเก็บ prompt ทุกครั้งถ้าสร้างใหม่ได้

---

## 11. Snapshots

Snapshot ต้องระบุ event high-water mark/revision

Authority order เมื่อ conflict:

1. current transactional tables
2. event history
3. snapshot (discard/rebuild)

Snapshot deletion ต้องไม่ทำให้ correctness เสีย

---

## 12. Migrations

- centralized, checksummed, ordered
- startup maintenance phase ก่อน runtime services
- forward-only โดยปกติ
- application มี MIN/MAX_SCHEMA_REVISION
- DB newer than app → fail closed
- filesystem migration ใช้ journal+immutable prepare+DB commit+later GC

---

## 13. Integrity and backup

- `PRAGMA integrity_check`
- safe SQLite backup API/checkpoint policy
- migration checksum
- artifact hash validation
- optional event hash chain
- retention class แยก semantic events/IPC dedupe/raw payload/secrets

ห้าม copy live `.sqlite3` โดยไม่สน WAL

---

## 14. Startup recovery scan

ตรวจ deterministic:

- stale scheduler/model leases
- runtime/model attempts stuck running
- effects executing/outcome_unknown
- pending result ACK
- unconfirmed deliveries
- incomplete migration
- orphan artifacts
- integrity failures

แต่ละ category ต้องมี explicit recovery transition

---

## 15. Acceptance

- power loss ทุก transaction boundary ไม่สร้าง impossible state
- duplicate ingress → one Ticket
- every revision ↔ event
- stale generation mutate ไม่ได้
- transactions ไม่คร่อม external calls
- effect/delivery ambiguity survive restart
- snapshot rebuildable
- supported schema migrations deterministic
- live backup restore valid references
- Adapter ไม่มี write ownership ต่อ authority tables
