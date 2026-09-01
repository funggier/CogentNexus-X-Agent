# 09 — Master Roadmap and Definition of Done

> **สถานะเอกสาร:** Design Baseline / Living Specification  
> **ภาษาหลัก:** ไทย โดยใช้คำศัพท์เชิงสถาปัตยกรรมภาษาอังกฤษเป็น canonical term  
> **หลักการอ่าน:** คำว่า MUST / MUST NOT / SHOULD / MAY ใช้ในความหมายเชิง normative  
> **ขอบเขต:** CogentNexus เป็นโปรแกรมอิสระที่เป็น durable semantic control plane และเชื่อมระบบภายนอกผ่าน replaceable Adapters


## 1. Roadmap overview

```text
M0 Architecture Baseline
 ↓
M1 Python Core Skeleton
 ↓
M2 Durable State + Artifact Store
 ↓
M3 Task/Transition/Knowledge Engine
 ↓
M4 Scheduler + Context Compiler
 ↓
M5 Adapter Transport + Fake Runtime
 ↓
M6 Effect/Delivery Safety
 ↓
M7 Hermes Discovery + Read-only Qualification
 ↓
M8 Hermes Reversible Managed Execution
 ↓
M9 End-to-End Ticket → Delivery
 ↓
M10 Crash/Chaos/Upgrade Acceptance
 ↓
M11 GUI Monitor
 ↓
M12 GUI Safe Control
 ↓
V1 Candidate Freeze
```

---

## 2. Milestone details

### M0 — Architecture Baseline
DoD: glossary/state/transport/effect/persistence specs approved and no major synonym conflict

### M1 — Core Skeleton
DoD: headless service, config/logging, API shell, dependency boundaries

### M2 — Persistence
DoD: migrations, tickets/tasks/events/artifacts/recovery foundation + power-loss tests

### M3 — Semantic Engine
DoD: guards/transitions/knowledge/evidence/obligations/completion contracts

### M4 — Intelligence OS Layer
DoD: one-slot scheduler, budgets, context envelope/audit, fake model

### M5 — Adapter Protocol
DoD: durable replay/fencing/conformance with fake adapters

### M6 — External Safety
DoD: grants/effects/reconciliation/delivery + negative tests

### M7 — Hermes Read-only
DoD: public seam discovery, manifest, context/result capture, no mutation

### M8 — Reversible Mutation
DoD: scoped workspace edit/test/commit through Effect Intent

### M9 — Managed E2E
DoD: real user intent durable admission through confirmed delivery

### M10 — Chaos
DoD: kill every critical boundary; no lost intent/duplicate effect

### M11/M12 — GUI
DoD: separate process, monitor then safe commands, no DB bypass

---

## 3. Dependency rules

- GUI control waits for Core command API stable
- high-risk Hermes effects wait for negative qualification
- delivery retry waits for delivery identity/reconcile semantics
- advanced semantic retrieval waits for deterministic Context Compiler baseline
- multi-model waits until one-model acceptance passes

---

## 4. Project-level Definition of Done

Feature is not Done until:

1. semantic responsibility named and documented
2. state/transition ownership clear
3. durable persistence/recovery path defined
4. authorization/effect implications defined
5. tests include failure/duplicate/stale cases where relevant
6. observability exposes identity/evidence
7. no private Hermes dependency unless explicitly diagnostic optional
8. migration/backward compatibility considered
9. GUI/CLI cannot bypass invariant
10. documentation updated with canonical vocabulary

---

## 5. V1 acceptance statement

V1 is complete only when a single modest model plus one Hermes runtime can operate under CogentNexus such that:

- user intent is durably admitted before managed execution
- long work is decomposed/scheduled without global-context growth
- every side effect is fenced and evidence-backed
- crashes/restarts recover from durable state
- ambiguous external outcomes reconcile instead of blind retry
- Hermes can be restarted/upgraded without losing logical work
- final response is distinct from delivery confirmation
- operator can monitor/control through CLI/GUI without becoming authority

---

## 6. What V1 proves conceptually

ถ้าผ่าน V1 ระบบได้พิสูจน์ property สำคัญที่สุด:

> Intelligence can be disposable while intent, evidence, authority, and progress remain durable.

จากนั้น scale ไป larger model, multiple adapters, parallel capacity หรือหลายเครื่องเป็นการเพิ่ม capacity/policy ไม่ใช่ rewrite correctness architecture
