# 11 — Threat Model, Reliability, and Safety Considerations

> **สถานะ:** Cross-cutting Architecture Baseline  
> **เป้าหมาย:** รวมสิ่งที่ต้องระวังด้าน security, safety, reliability และ semantic integrity ซึ่งไม่ควรถูกฝากไว้กับ subsystem ใด subsystem หนึ่ง

---

## 1. Threat philosophy

CogentNexus ต้องรับมือทั้ง malicious และ non-malicious failure ภายใต้ assumption ว่า:

- LLM อาจผิด/ถูก prompt injection
- Adapter อาจมี bug
- process อาจ crash ตรงจุดแย่ที่สุด
- external API อาจตอบกำกวม
- local IPC ไม่ได้ปลอดภัยเพียงเพราะอยู่เครื่องเดียว
- operator อาจกดคำสั่งซ้ำ
- stale worker อาจกลับมาหลัง ownership transfer
- software update อาจเปลี่ยน behavior โดยไม่เปลี่ยน API name

ดังนั้น security และ reliability ใช้ identity/fencing/evidence ร่วมกัน

---

## 2. Trust boundaries

```text
[User/External Channel]
       │ untrusted/native auth evidence
       ▼
[Ingress Adapter]
       │ normalized claims
       ▼
[CogentNexus Core]  ← primary trust/semantic boundary
       │ narrow grants/contracts
       ▼
[Runtime/Action Adapter]
       │ native commands
       ▼
[Hermes / OS / Git / Network / UI]
```

GUI เป็นอีก boundary:

```text
[GUI process] → authenticated local Control API → Core
```

GUI user access ไม่ควร imply OS/admin permission โดยอัตโนมัติ

---

## 3. Asset classes

### Semantic assets
intent, Ticket/Task state, decisions, obligations

### Authority assets
grants, lease tokens, capability mappings, confirmations

### Evidence assets
commit SHA, hashes, receipts, validation reports

### Sensitive assets
credentials, private content, environment secrets

### Availability assets
DB, artifact store, Adapter connectivity, model slot

แต่ละ class มี retention/access/backup policy ต่างกัน

---

## 4. Major threat scenarios

### T1 — Prompt injection tries to mint authority
Mitigation: model cannot create grant; grants originate deterministic Core policy/operator authorization

### T2 — Raw shell bypasses capability restrictions
Mitigation: sandbox/allowlist/structured commands; negative capability tests

### T3 — Duplicate/replay executes mutation twice
Mitigation: effect identity/idempotency/reconciliation/fencing

### T4 — Stale runtime commits after takeover
Mitigation: monotonic generation checked in Core transaction and Adapter gate

### T5 — Adapter claims success without real effect
Mitigation: capability-specific evidence + independent read-back

### T6 — Delivery function returns but user never sees output
Mitigation: Delivery state/receipt policy separate from runtime result

### T7 — Context compiler leaks secret/unrelated task
Mitigation: sensitivity metadata + explicit selection/audit + owner isolation

### T8 — GUI bypasses safeguards
Mitigation: no DB write; command API runs same authorization/transitions

### T9 — DB/artifact inconsistency after crash
Mitigation: durable artifact-before-reference, atomic state+event, startup integrity scan

### T10 — Hermes update changes tool semantics
Mitigation: mapping hash/revision + requalification + degraded mode

---

## 5. Reliability hazards

### Split-brain authority
Two processes believe they own same work; generation fence mandatory

### Lost wakeup
Durable state must make runnable work discoverable by scan; wake signal is optimization, not sole truth

### Poison message
Protocol frame repeatedly crashes receiver; durable quarantine/protocol error state and operator inspect

### Unbounded replay
Use committed watermarks/retention and terminal receipts

### Priority starvation
Aging/fairness/starvation tests

### Clock dependence
Prefer monotonic elapsed time for lease internals where possible; durable timestamps primarily audit. Expiry semantics must tolerate clock skew if multi-machine later

---

## 6. Safety classification

Risk should consider:

- probability
- impact
- reversibility
- observability of outcome
- blast radius
- authorization sensitivity
- recovery cost

A low-probability irreversible action remains high importance

---

## 7. Fail-open vs fail-closed policy

Fail closed for:

- authorization uncertainty
- effect parameter/hash mismatch
- stale generation
- required context fence missing in Managed Mode
- unsupported required schema/protocol
- high-risk capability mapping unqualified

May fail open/degrade for:

- optional telemetry
- optional summary/index
- non-authoritative UI visualization
- optional semantic retrieval

Explicitly document each boundary; “best effort” without semantics is forbidden for authority

---

## 8. Human authorization safety

Confirmation UX ต้อง bind exact scope/parameters using hash

Avoid vague confirmation:

```text
Allow dangerous action? [Yes]
```

Prefer:

```text
Capability: system.reset_app_state
Target: CogentNexus-Hermes local profile
Effect: eff_123
Irreversible data categories: session/runtime state
Expected postcondition: generation increment + clean baseline
```

Material change invalidates confirmation

---

## 9. Data privacy

- raw messages/artifacts may contain secrets/personal data
- Context Compiler should honor sensitivity and least exposure
- audit should explain exclusion without echoing secret value
- event payloads favor refs/hashes/redacted metadata
- GUI exports opt-in for sensitive payloads

---

## 10. Security testing checklist

- forged Adapter identity
- replay old authorization token
- stale epoch reconnect
- modified payload under same message ID
- path traversal artifact refs
- shell metacharacter/argument injection
- unauthorized GUI command
- session generation mismatch
- cross-Ticket context leakage
- capability manifest downgrade/rollback
- corrupted event/artifact hash

---

## 11. Incident/recovery principle

When evidence is insufficient, preserve uncertainty and stop irreversible progression. Recovery quality is measured by ability to continue safely, not by forcing every Task into SUCCESS/FAILED quickly

---

## 12. Acceptance

Threat model is adequately implemented when every high-risk boundary has:

1. explicit authority owner
2. identity/fence
3. precondition validation
4. evidence/postcondition
5. ambiguous-outcome policy
6. negative test
7. observability without secret leakage
8. recovery transition
