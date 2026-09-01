# 07 — Ingress, Authentication, Authorization, Ownership, and Delivery

> **สถานะเอกสาร:** Design Baseline / Living Specification  
> **ภาษาหลัก:** ไทย โดยใช้คำศัพท์เชิงสถาปัตยกรรมภาษาอังกฤษเป็น canonical term  
> **หลักการอ่าน:** คำว่า MUST / MUST NOT / SHOULD / MAY ใช้ในความหมายเชิง normative  
> **ขอบเขต:** CogentNexus เป็นโปรแกรมอิสระที่เป็น durable semantic control plane และเชื่อมระบบภายนอกผ่าน replaceable Adapters


## 1. ทำไมต้องแยก ownership

Distributed failures จำนวนมากคือ “ใครคิดว่าตัวเองเป็นเจ้าของอะไร” ไม่ชัด เช่น Core และ Adapter retry delivery พร้อมกัน หรือ session ID ถูกใช้เป็น permission

ทุก operation ต้องตอบ:

> Who may decide, who may execute, and who may confirm?

---

## 2. Four concepts that must not collapse

### Principal
ใครกำลัง act

### Session
interaction route ปัจจุบันอยู่ไหน

### Authorization
Principal/Ticket ทำ capability/scope ใดได้

### Ownership
fenced actor ใดถือ claim ต่อ operation หนึ่งในขณะนี้

Authenticated session ไม่เท่ากับ destructive authorization

---

## 3. Ownership chain

```text
External Channel
→ Ingress Adapter (native transport/auth evidence)
→ Core Admission (durable Ticket)
→ Core Scheduler/Authorization
→ Execution Adapter
→ Core Result Commit
→ Core Delivery Intent
→ Delivery Adapter
→ Native Receipt/Evidence
→ Core Delivery Confirmation
```

Core เป็น semantic authority ตลอด chain

---

## 4. Ingress Adapter

Owns:
- native channel connection
- platform authentication mechanism
- stable external message identity
- origin metadata normalization
- duplicate callback handling
- local spool ถ้าต้อง ACK platform เร็ว

Does not own:
- semantic authorization
- Ticket lifecycle
- completion
- arbitrary execution permission

---

## 5. Durable admission

ACCEPTED มีความหมายหลัง transaction:

1. dedupe ingress
2. map principal/session claims
3. create/resolve Ticket
4. store content/intent refs
5. append admission event
6. create runnable/waiting logical state

UI click/socket write ไม่ใช่ durable admission

Early platform ACK ต้องบอกเพียง callback received และควร local spool หาก ACK ก่อน Core commit

---

## 6. Authentication vs trust vs authorization

Adapter normalize claims; Core decides trust policy

Trust examples:

```text
UNAUTHENTICATED
LOCAL_AUTHENTICATED
REMOTE_AUTHENTICATED
OPERATOR_VERIFIED
SYSTEM_INTERNAL
```

Authorization grant ต้อง narrow:

```yaml
grant_id:
principal_id:
ticket_id:
capability_id:
scope_hash:
constraints:
requires_confirmation:
expires_at:
```

---

## 7. Session generation

session reset/recreate/auth change increments generation หรือสร้าง new identity

Delivery bound generation N ห้ามข้ามไป N+1 แบบเงียบ ๆ

---

## 8. Execution ownership

temporary fenced lease includes:

```text
Task/runtime identity
capability
generation/token
grant
deadline
idempotency key
```

Lease lost → no new Core commit authority; external effect evidence ที่อาจเกิดแล้วยัง report ได้เพื่อ reconciliation

---

## 9. Delivery ownership

Core owns:
- whether to deliver
- payload identity/hash
- recipient/session generation
- retry/dedupe policy
- completion predicate

Adapter owns:
- native send
- native delivery ID/callback
- evidence
- micro-retry allowed by Core

Function return/socket write ไม่พอเป็น confirmation

---

## 10. Delivery state

```text
STAGED
→ CLAIMED
→ SENDING
→ CONFIRMED
       or FAILED_RETRYABLE
       or OUTCOME_UNKNOWN → RECONCILING → ...
```

CONFIRMED terminal; stale failure callback ห้ามย้อนกลับ

---

## 11. Retry/dedupe

Ingress preferred dedupe `(adapter_id, external_message_id)`

Delivery มี stable `delivery_id`; transport retries reuse identity

ถ้า native idempotency ไม่มี → reconcile ambiguous send ก่อน resend

Core owns semantic retry policy

---

## 12. Multi-Adapter routing

Ticket เดียวอาจ:

```text
OpenClaw Adapter = ingress/delivery
Hermes Adapter   = cognitive/action runtime
Direct Provider  = model invocation
```

ไม่มี Adapter ใดต้องเห็น hidden conversation ของอีกตัว ถ้า Core ส่ง durable state/artifacts ที่จำเป็น

---

## 13. Secrets

- OS/service secret store
- opaque handle
- no event payload copy
- no model context unless necessary
- scoped to capability

Core มักต้องรู้ว่ามี credential binding ไม่ใช่ secret

---

## 14. Strict Ticket-first with Hermes

Architecture ต้องพิสูจน์ public seam ที่ทำ:

```text
authenticated ingress
→ CNX durable admission
→ then Hermes agent execution
```

ถ้า Hermes native channel seam ไม่รองรับ strict ordering โดยไม่ patch core ให้ใช้ CNX-owned managed ingress/channel adapter แทน

นี่เป็น qualification decision ไม่ใช่เหตุให้ Core ผูกกับ Hermes internals

---

## 15. Acceptance

- duplicate ingress one Ticket
- unauthenticated no privileged grant
- session reset fences old delivery
- wrong-owner callback rejected
- confirmed delivery immutable
- Core crash ไม่ยก authority ให้ Adapter
- external send ambiguity ไม่ blind resend
- authorization revoke stops future effects
- one Ticket cross OpenClaw→Hermes→OpenClaw โดยไม่ share hidden transcript
