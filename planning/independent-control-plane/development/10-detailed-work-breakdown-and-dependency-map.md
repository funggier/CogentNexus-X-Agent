# 10 — Detailed Work Breakdown Structure and Dependency Map

> **สถานะ:** Implementation Planning Baseline  
> **เป้าหมาย:** แตก roadmap เป็นงานที่ commit/test ได้ทีละก้อน โดยคงลำดับ dependency และไม่สร้าง integration debt

---

## 1. Workstream map

```text
A Domain/Ontology ───────┐
B Persistence ───────────┼─→ D Scheduler/Context ─→ G Managed E2E
C Transport/Adapter ─────┤          │                    │
E Effects/Auth/Delivery ─┘          ├─→ F Hermes ────────┤
H Control API ──────────────────────┴─→ I GUI            │
J Chaos/Release/Operations ───────────────────────────────┘
```

---

## 2. Workstream A — Domain/Ontology

### A1 IDs and canonical enums
Deliverables: typed IDs, Task phase/status/outcome, scheduler/effect/delivery states

### A2 Ticket/Task models
Recursive relation, criteria, blockers, obligations

### A3 Transition engine
Guards, expected revision, events

### A4 Knowledge/evidence records
Epistemic type, provenance, supersession

**Gate A:** all domain tests pure/no SQLite/GUI/Hermes imports

---

## 3. Workstream B — Persistence

### B1 migration runner
Checksums, schema bounds, maintenance startup

### B2 semantic tables
Ticket/Task/events/obligations

### B3 artifact store
Crash-safe content addressed

### B4 scheduler/runtime tables
Jobs/slot/attempts/model runs

### B5 auth/effect/delivery/IPC tables

### B6 recovery scan + backup/integrity

**Gate B:** fault injection proves atomic boundaries

---

## 4. Workstream C — Adapter Transport

### C1 protocol schemas/hash
### C2 durable Core inbox/outbox
### C3 fake Adapter peer
### C4 ACK/result commit lifecycle
### C5 replay/watermark/order/gap
### C6 epoch/fence/backpressure
### C7 local peer authentication
### C8 Adapter SDK/conformance harness

**Gate C:** fake peer survives duplicate/reorder/crash matrix

---

## 5. Workstream D — Scheduler/Context

### D1 durable queue + one model slot
### D2 classes/aging/fairness
### D3 quantum/pause/cancel/wait IO
### D4 budget ledger
### D5 Context Manifest/Envelope
### D6 authority/dependency/obligation selection
### D7 Context Audit
### D8 small reference model benchmark

**Gate D:** continuous progress on one model, bounded context

---

## 6. Workstream E — Authority/Effects/Delivery

### E1 principal/session/trust
### E2 authorization grants + scope hash
### E3 capability registry/manifest
### E4 Effect Intent state machine
### E5 reconciliation interface
### E6 Delivery state machine
### E7 confirmation receipts
### E8 negative tests

**Gate E:** no mutation without effect intent; ambiguous send not resent blindly

---

## 7. Workstream F — Hermes Adapter

### F1 public seam inventory
### F2 manifest discovery
### F3 health/session binding
### F4 context selection/read-only execution
### F5 runtime result normalization
### F6 yield/budget/cancel qualification
### F7 tool gate/read-only qualification
### F8 reversible workspace mutation
### F9 reconciliation mappings
### F10 upgrade requalification

**Gate F:** no source patch; capability-specific qualification report

---

## 8. Workstream G — Managed End-to-End

### G1 managed ingress decision/implementation
### G2 admission→Task→scheduler→Hermes
### G3 output→validation→Task transition
### G4 delivery staging/confirmation
### G5 crash/restart semantic resume

**Gate G:** one user send → one Ticket → one semantic result → one confirmed delivery

---

## 9. Workstream H — Control API/CLI

### H1 read models/status
### H2 command framework + receipts
### H3 pause/resume/cancel
### H4 authorization/reconcile commands
### H5 context/event diagnostics
### H6 backup/integrity commands

CLI เป็น reference client ก่อน GUI

---

## 10. Workstream I — GUI

### I1 PySide6 shell/process connection
### I2 overview/adapters
### I3 Ticket/Task views
### I4 scheduler/effects/delivery
### I5 safe control commands
### I6 authorization UX
### I7 diagnostics/context audit
### I8 packaging/reconnect/load tests

---

## 11. Workstream J — Chaos/Operations/Release

### J1 deterministic crash injector
### J2 transport chaos
### J3 effect/delivery ambiguity
### J4 migration/backup/restore
### J5 Hermes update drill
### J6 frozen candidate evidence pack

---

## 12. Parallelism guidance for development

Can parallelize after interfaces stable:

- GUI read-only views กับ Hermes discovery
- artifact store กับ protocol schema
- negative tests กับ capability registry

Must stay sequential:

- ontology before DB schema freeze
- effect identity before mutating Hermes qualification
- Control API before GUI mutation controls
- scheduler quantum contract before claiming Hermes fairness

---

## 13. Commit sizing

แต่ละ commit ควรเปลี่ยน semantic responsibility เดียวและมี tests; หลีกเลี่ยง “implement entire scheduler + DB + Hermes” commit เพราะ review/recovery ยาก

---

## 14. Progress reporting

Milestone report ควรมี:

- objective
- decisions/invariants touched
- files/commits
- tests/evidence
- exact schema/protocol revisions
- risks/unknowns
- PASS/FAIL/BLOCKED
- next valid work
