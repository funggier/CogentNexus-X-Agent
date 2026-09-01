# 04 — Scheduler, Context Compiler, and Small-Model Implementation Plan

> **สถานะเอกสาร:** Design Baseline / Living Specification  
> **ภาษาหลัก:** ไทย โดยใช้คำศัพท์เชิงสถาปัตยกรรมภาษาอังกฤษเป็น canonical term  
> **หลักการอ่าน:** คำว่า MUST / MUST NOT / SHOULD / MAY ใช้ในความหมายเชิง normative  
> **ขอบเขต:** CogentNexus เป็นโปรแกรมอิสระที่เป็น durable semantic control plane และเชื่อมระบบภายนอกผ่าน replaceable Adapters


## 1. Objective

ทำให้ระบบทำงานได้จริงด้วย model slot เดียวโดย host แบก global complexity และ model เห็นเฉพาะ bounded semantic work

---

## 2. Scheduler implementation

### Step 1 — durable queue
scheduler_jobs + model_slot + lease generation

### Step 2 — deterministic class policy
P0/P1/P2/P3

### Step 3 — fairness
owner deficits/aging/consecutive quantum cap

### Step 4 — pause/cancel/waiting_io
safe boundary semantics

### Step 5 — budget accounting
atomic consume/reserve where needed

### Step 6 — recovery
stale lease classification

---

## 3. Context Compiler implementation

### Minimal first pass
Input: Task revision + purpose + budget

Select:
- intent/objective
- current Task state
- required dependencies
- unresolved obligations
- recent authoritative facts/decisions
- required evidence
- authority/capability fences
- output contract

Emit Context Envelope + Audit

### Later enhancements
- artifact summaries/projections
- semantic candidate retrieval
- adaptive profiles
- conflict resolution views

---

## 4. Small-model routing

Avoid model calls for deterministic transitions

Model roles sequential, not persistent agents:

```text
Classifier/Extractor
Planner/Work Shaper
Investigator/Executor
Critic/Validator helper
Composer/Reporter
```

Role = prompt/schema/context profile; model เดิมใช้ซ้ำและ unload/resource policy ตาม provider

---

## 5. Pass fusion

Low-risk/simple task สามารถ combine classification + intent extraction

High-risk/ambiguous แยก passes

Decision gate ประเมิน cost/uncertainty/risk แทน fixed pipeline

---

## 6. Context budget tests

- irrelevant 1000 artifacts ไม่เพิ่ม active packet มาก
- mandatory set over budget → explicit block, no truncation
- child sees declared deps only
- obligation never dropped
- summary conflict with higher-authority evidence resolved correctly

---

## 7. Scheduler fairness tests

- endless Task A cannot starve B
- bounded interactive stream cannot starve old foreground forever
- P0 control gets next safe boundary
- continuation bonus improves localityแต่ bounded
- WAITING_IO releases slot

---

## 8. Runtime granularity adaptation

Direct LLM adapter may yield each call; Hermes may bounded turns/opaque run

Scheduler maps Compute Envelope + runtime yield capability → physical quantum

Do not expose runtime limitations to Task semantic state except blocker/capability when materially relevant

---

## 9. Exit gate

- one small reference model completes benchmark corpus
- no global queue in model context
- pause/resume retains checkpoint
- model outage pauses/recover without duplicate effect
- replacing stronger model with reference small model does not break lifecycle correctness
