# 05 — Hermes Adapter Discovery, Qualification, and Rollout Plan

> **สถานะเอกสาร:** Design Baseline / Living Specification  
> **ภาษาหลัก:** ไทย โดยใช้คำศัพท์เชิงสถาปัตยกรรมภาษาอังกฤษเป็น canonical term  
> **หลักการอ่าน:** คำว่า MUST / MUST NOT / SHOULD / MAY ใช้ในความหมายเชิง normative  
> **ขอบเขต:** CogentNexus เป็นโปรแกรมอิสระที่เป็น durable semantic control plane และเชื่อมระบบภายนอกผ่าน replaceable Adapters


## 1. Non-negotiable constraint

CogentNexus MUST NOT require patch/fork/source modification of Hermes for core correctness. ใช้ documented/public extension points เท่านั้น; Hermes update ต้องกระทบ Adapter qualification ไม่ใช่ Core ontology

---

## 2. Discovery before assumptions

Adapter startup probes:

- Hermes release/runtime identity
- plugin/hook availability
- context selection capability
- structured LLM access if used
- message/internal injection
- tool preflight/post observers
- session resume
- Kanban/task lifecycle hooks
- bounded turns/runtime/cancel
- native delivery path/receipts

Store manifest + mapping hash

---

## 3. Qualification matrix

แต่ละ capability มี tests + minimum status

### Context Control
พิสูจน์ selected context เข้า provider request จริง และ failure semantics

### Execution Yield
พิสูจน์ max turns/runtime/checkpoint/cancel boundaries

### Tool/Effect Fence
พิสูจน์ทุก mutating route ที่เปิดใน Managed Mode ผ่าน gate

### Result Capture
พิสูจน์ intermediate vs final output separation

### Session Resume
พิสูจน์ identity/workspace/model/provider safety

### Kanban
ใช้เป็น execution substrate ได้แต่ไม่เป็น Core authority

### Delivery
พิสูจน์ว่า user-facing send สามารถ route/confirm ผ่าน Core policy หรือกำหนด managed channel แยก

---

## 4. Critical ingress decision

ทดสอบว่ามี public post-auth/pre-agent seam หรือ equivalent หรือไม่

หากไม่มี strict Ticket-first safe seam:

```text
Managed mode uses CNX-owned ingress/channel adapter
```

Hermes native channels remain passthrough

ห้ามใช้ pre-auth observer เป็น authoritative admission หาก trust/authorization ยังไม่ resolved

---

## 5. Tool safety decision

Hermes อาจมี terminal/subagent/native tool paths หลายแบบ

Qualification ต้อง attempt bypass:

- raw shell
- subagent tool
- Kanban worker
- plugin-injected operation
- alternate filesystem/git commands

ถ้า capability fence ครอบไม่ครบ ห้ามเปิด high-risk semantic capability

---

## 6. Rollout stages

### A Discovery
no mutation

### B Read-only
repo/fs/status/test observations

### C Reversible local
isolated worktree/edit/test/local commit

### D External narrow
push/install/service lifecycle with reconciliation

### E High-risk
reset/reboot/message send/release publish only after negative + crash tests

---

## 7. Upgrade handling

On Hermes update:

1. Adapter sees runtime identity change
2. previous manifest mapping becomes `NEEDS_REQUALIFICATION` for affected capabilities
3. pending Tasks remain durable
4. Managed Mode may degrade capability-wise
5. run conformance/qualification
6. re-enable only proven mappings

Do not delete CNX state or force session reuse

---

## 8. Shadow mode

ก่อน authority enforcement:

- observe native Hermes runs
- compile CNX state/context in parallel
- compare what would be granted/blocked
- collect divergence reports

จากนั้น enable read-only managed, then mutation

---

## 9. Acceptance

- Hermes replaced/restarted → Task survives
- unsupported capability → deterministic blocker
- Adapter cannot broaden grant
- no private DB/API dependency required
- context/result capture reproducible
- high-risk bypass tests fail
- update changes mapping without semantic schema migration
