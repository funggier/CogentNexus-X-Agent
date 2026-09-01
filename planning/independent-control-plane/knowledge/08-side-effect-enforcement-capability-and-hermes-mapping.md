# 08 — Side-Effect Enforcement, Capability Model, and Hermes Mapping

> **สถานะเอกสาร:** Design Baseline / Living Specification  
> **ภาษาหลัก:** ไทย โดยใช้คำศัพท์เชิงสถาปัตยกรรมภาษาอังกฤษเป็น canonical term  
> **หลักการอ่าน:** คำว่า MUST / MUST NOT / SHOULD / MAY ใช้ในความหมายเชิง normative  
> **ขอบเขต:** CogentNexus เป็นโปรแกรมอิสระที่เป็น durable semantic control plane และเชื่อมระบบภายนอกผ่าน replaceable Adapters


## 1. จุดอันตรายจริงของ agentic system

LLM text ไม่ใช่ mutation; ความเสี่ยงเกิดเมื่อ proposal ข้าม boundary ไปเปลี่ยน external state

ดังนั้น:

> **Effect Intent Ledger is the only legal bridge from reasoning to mutation.**

---

## 2. Effect classes

```text
READ_ONLY
LOCAL_REVERSIBLE
EXTERNAL_IDEMPOTENT
EXTERNAL_REVERSIBLE
EXTERNAL_IRREVERSIBLE
```

classification กำหนด default replay/authorization/evidence policy

---

## 3. Effect Intent

ก่อน external execution Core durable record:

```yaml
effect_id:
ticket_id:
task_id:
capability_id:
effect_class:
target_ref_or_hash:
parameters_hash:
idempotency_key:
authorization_grant:
replay_policy:
attempt_budget:
expected_evidence:
created_at:
```

Effect ID = semantic mutation หนึ่งครั้ง; retries reuse ID

---

## 4. Effect lifecycle

```text
PROPOSED
→ AUTHORIZED
→ CLAIMED
→ EXECUTING
→ SUCCEEDED
   | FAILED_RETRYABLE
   | FAILED_PERMANENT
   | OUTCOME_UNKNOWN → RECONCILING → ...
```

SUCCEEDED immutable ยกเว้นเพิ่ม evidence/annotation

---

## 5. Replay policies

```text
SAFE_REPLAY
RETRY_SAME_IDEMPOTENCY_KEY
RECONCILE_THEN_RETRY
NO_AUTOMATIC_RETRY
```

High-risk default conservative

---

## 6. Enforcement gate

Adapter ก่อน mutation MUST validate:

- protocol/adapter epoch
- effect identity/hash
- capability support + mapping revision/hash
- grant validity/scope
- lease generation/token
- attempt budget
- replay policy
- local safety preconditions

missing required item → fail closed

---

## 7. Semantic capability ontology

Implementation-neutral names:

```text
fs.read
fs.write
repo.read
repo.modify
repo.commit
repo.push
test.run
package.install
service.query
service.restart
system.reset_app_state
system.reboot
ui.observe
ui.interact
message.send
release.publish
```

ห้ามใส่ `hermes` หรือ version ใน capability ID

---

## 8. Capability manifest and qualification

Adapter publish:

```yaml
capability_id:
native_operation:
support_state:
effect_class:
supports_cancel:
supports_dry_run:
supports_native_idempotency:
supports_reconciliation:
evidence_types:
max_concurrency:
mapping_hash:
```

Qualification states:

```text
DISCOVERED
QUALIFIED_READ_ONLY
QUALIFIED_MUTATING
QUALIFIED_HIGH_RISK
DEGRADED
DISABLED
```

Hermes update → re-discover/re-qualify mappings ไม่แก้ semantic Core

---

## 9. Raw shell hazard

`process.exec` สามารถ bypass ทุก capability policy หาก unrestricted

ควรเป็น:

- read-only allowlist
- sandboxed workspace command
- high-risk structured command grant
- implementation detail หลัง specific capability wrapper

Normalize executable/args/cwd/env/timeout/network/fs scope และ hash command

---

## 10. Evidence contract

Exit code 0 ไม่พอเสมอ

- commit → exact SHA/tree/worktree state
- push → remote ref exact SHA
- install → provenance + health
- restart → new instance healthy
- reset → post-reset generation/baseline
- message send → native delivery identity

mutation path และ verification path ควรแยกเมื่อทำได้

---

## 11. Reconciliation as first-class capability

ทุก nontrivial effect ควรมี reconciliation function เช่น:

```text
repo.push.reconcile → remote exact ref
message.send.reconcile → native message identity
install.reconcile → installed fingerprint/health
```

timeout/disconnect/process death ถ้าอาจ commit แล้ว → OUTCOME_UNKNOWN, reconcile ก่อน retry

---

## 12. Hard negative authority

Context/Execution Contract ควรมี explicit forbidden capabilities

absence of capability + structural deny แข็งกว่าข้อความ natural-language “อย่าทำ”

Negative qualification tests ต้องพิสูจน์ forbidden paths fail

---

## 13. Hermes role

```text
CNX semantic capability
→ Hermes Adapter mapping/enforcement
→ Hermes native tool/skill/subprocess
→ external system
```

Core decides **what may happen and under what invariant**; Hermes decides **how to execute bounded grant** และคืน evidence

---

## 14. Hermes Adapter questions that must be qualified

- pre-tool gate ครอบ tool paths ทุกแบบไหม
- subagent/Kanban worker ข้าม gate ได้ไหม
- shell/process paths enforce scope ได้ไหม
- context selection fail-open/closed behavior
- bounded turns/runtime/yield control
- output interception ก่อน user delivery
- session resume identity safety
- cancellation semantics
- evidence fidelity

ถ้าตอบไม่ได้ capability high-risk ต้อง DEGRADED/DISABLED

---

## 15. Rollout

A. discovery only  
B. qualify read-only  
C. reversible workspace mutation  
D. narrow external mutation + reconciliation  
E. high-risk only after explicit negative/chaos tests

---

## 16. Acceptance

- model execute ungranted mutation ไม่ได้
- every mutation has effect ID before execution
- retries retain effect ID
- stale lease cannot commit
- changed params under same effect ID reject
- ambiguous high-risk → OUTCOME_UNKNOWN
- reconciliation recovers after crash-after-commit
- raw shell cannot bypass policy
- success requires capability-specific evidence
- Hermes mapping changes auditable/versioned
