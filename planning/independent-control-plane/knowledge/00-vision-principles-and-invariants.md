# 00 — Vision, Principles, and Core Invariants

> **สถานะเอกสาร:** Design Baseline / Living Specification  
> **ภาษาหลัก:** ไทย โดยใช้คำศัพท์เชิงสถาปัตยกรรมภาษาอังกฤษเป็น canonical term  
> **หลักการอ่าน:** คำว่า MUST / MUST NOT / SHOULD / MAY ใช้ในความหมายเชิง normative  
> **ขอบเขต:** CogentNexus เป็นโปรแกรมอิสระที่เป็น durable semantic control plane และเชื่อมระบบภายนอกผ่าน replaceable Adapters


## 1. วิสัยทัศน์

CogentNexus มีโจทย์ใหญ่กว่าการเป็น agent framework: ทำอย่างไรให้ **intent ของมนุษย์หนึ่งคนไหลผ่าน intelligence และระบบปฏิบัติการจำนวนมากแล้วกลายเป็นการกระทำที่ยังคงทิศทางเดิม** แม้จะเกิด interruption, context loss, runtime replacement, retries, side effects และการส่งต่องานระหว่างระบบที่ไม่เห็น conversation เดียวกัน

คำว่า **Cogent** หมายถึง coherence, meaning, direction และความสามารถในการอธิบายเหตุผลของ state change; ส่วน **Nexus** หมายถึง connection, propagation, coordination และการเชื่อมสิ่งที่แตกต่างผ่าน semantic contract ร่วม

ดังนั้นชื่อระบบควรสะท้อนเป็น invariant ไม่ใช่ branding เท่านั้น:

> ทุกการเชื่อมต่อเพิ่มความสามารถได้ แต่ต้องไม่ทำให้ความหมายของ intent drift

---

## 2. ปัญหาที่ระบบต้องแก้

Agentic systems มักล้มเหลวไม่ใช่เพราะ model “ไม่ฉลาด” แต่เพราะระบบไม่มี durable structure ที่บอกว่าอะไรเป็นจริงและใครมี authority เช่น:

- model/session หายแล้วงานหาย
- agent ใหม่รับ assumption เป็น fact
- result ถูก regenerate ทั้งที่เดิม commit แล้ว
- side effect ถูกทำซ้ำเพราะ timeout ถูกแปลเป็น failure
- user message ถูกตอบก่อน durable admission
- transcript ถูกใช้เป็น database
- long-running task กิน model resource จน interactive input starvation
- GUI หรือ Adapter กลายเป็น authority โดยไม่ตั้งใจ
- runtime upgrade ทำให้ integration พังทั้ง Core

CogentNexus จึงย้าย continuity จาก conversation ไปสู่:

```text
Persistent Task State
+ Structured Knowledge
+ Evidence
+ Artifacts
+ Transition History
+ Authority / Effect / Delivery Ledgers
```

---

## 3. วัตถุประสงค์เชิงสถาปัตยกรรม

### 3.1 Intent Preservation
ระบบต้องเก็บ raw intent และ derived intent แยกกัน; inference ห้ามเขียนทับข้อความต้นฉบับ

### 3.2 Durable Progression
ทุก transition สำคัญต้อง commit พร้อม event และสามารถ reconstruct ได้หลัง crash

### 3.3 Small-Model-First
ระบบต้องลด intelligence requirement ด้วย structure: bounded packet, decision menu, deterministic routing, validation และ tool normalization

### 3.4 Replaceable Runtime
Hermes/OpenClaw/direct provider เป็น runtime infrastructure ไม่ใช่ semantic authority

### 3.5 Explicit Side-Effect Safety
reasoning และ mutation ต้องมี deterministic enforcement boundary คั่น

### 3.6 Operator Control and Visibility
ผู้ใช้ต้องสามารถดูว่า “ระบบกำลังทำอะไร อยู่ตรงไหน ติดอะไร มี authority อะไร” และควบคุม pause/cancel/resume ได้โดยไม่แก้ DB เอง

---

## 4. Invariants ระดับ Core

### I-01 Accepted Work Cannot Vanish
Ticket ที่ admit แล้วต้องจบด้วย outcome ที่ durable: success/partial/blocked/failed/cancelled/superseded และ delivery policy ที่อธิบายได้

### I-02 State Is Not Conversation
อีก intelligence หนึ่งต้องทำต่อได้จาก state/artifact/evidence โดยไม่อ่านบทสนทนาทั้งหมด

### I-03 Evidence Before Completion
คำว่า COMPLETE เป็น semantic claim จึงต้องมี validation/evidence ตาม Completion Contract

### I-04 Model Output Is Not Authority
LLM output เป็น proposal; Core/validator/policy เป็นผู้เปลี่ยน authoritative state

### I-05 Knowledge Is Typed
Fact, Observation, Assumption, Hypothesis, Inference, Unknown, Decision, Evidence ต้องไม่ถูกรวมเป็น “context” ก้อนเดียว

### I-06 Retry Must Not Become Repetition
retry semantic workเดิมต้องรักษา identity; side effect ใหม่ต้องมี effect identity ใหม่จริง

### I-07 Unknown Is a Valid State
timeout, disconnect, process death หรือ missing ACK ไม่เท่ากับ failure ถ้า external commit อาจเกิดแล้ว

### I-08 Stale Owners Cannot Commit
scheduler, runtime, effect และ delivery ใช้ monotonic generation/fence

### I-09 No External Mutation Without Effect Intent
ต้องมี identity, authority, scope, replay policy, expected evidence ก่อน mutation

### I-10 No Required Parallelism
logical graph อาจ parallel แต่ physical execution ต้องสามารถ serialize บน model slot เดียวได้

### I-11 Context Is Derived
Context Envelope สร้างจาก durable state; การลบ context ไม่ลบ knowledge

### I-12 Hermes Is Replaceable
Core correctness ห้ามขึ้นกับ private Hermes DB/schema/session behavior เพียงอย่างเดียว

### I-13 GUI Is Not Authority
GUI แสดงผลและร้องขอ command ผ่าน Core API เท่านั้น

### I-14 Semantic Names Outlive Releases
version อยู่ metadata/boundary; ontology ไม่ควรเต็มไปด้วย `v2`, `new`, `legacy`, `final`

---

## 5. สิ่งที่ต้องคำนึงถึง

### ความมั่นคง (Reliability)
- crash ทุก boundary ต้องมี deterministic recovery
- DB transaction ห้ามคร่อม network/model/tool calls
- payload/evidence ต้องมี hash/ref ตามความเสี่ยง

### ความปลอดภัย (Safety/Security)
- authentication ≠ authorization
- least privilege ต่อ capability/scope
- raw shell เป็น meta-capability และต้องถูกจำกัด
- secrets ไม่ควรเข้า event/model context โดยไม่จำเป็น

### ความสามารถในการตรวจสอบ (Auditability)
- state transition ต้องมี causation/evidence
- Context Compiler ต้องอธิบายได้ว่า include/exclude อะไรและทำไม
- capability mapping ต้อง version/hash เพื่อย้อน audit หลัง Hermes update

### ความเรียบง่าย (Simplicity)
- เริ่มจาก smallest sufficient profile
- ใช้ recursive Task แทนเพิ่ม object types เมื่อไม่จำเป็น
- ใช้ hybrid state+event ไม่รีบ full event sourcing

---

## 6. Failure modes ที่ต้องระวัง

- Over-design: บังคับ task เล็กกรอก risk/evidence schema ทั้งชุด
- Under-design: ปล่อยให้ transcript เป็น source of truth
- Authority leakage: Adapter/GUI เขียน state โดยตรง
- Hidden retries: runtime retry side effect เองโดย Core ไม่รู้
- Capability escape: ให้ `process.exec` แล้วคิดว่า deny `system.reboot` เพียงพอ
- Context drift: summary เก่ากลายเป็น fact ที่ authority สูงกว่า evidence ใหม่
- Scheduler drift: ให้ LLM เลือก fairness/priority เอง
- Version drift: เปลี่ยน semantic names ตาม release

---

## 7. Design test สั้น ๆ

ทุก feature ใหม่ควรถาม:

1. สิ่งนี้ช่วยรักษา intent หรือแค่เพิ่ม complexity?
2. ถ้า Hermes หาย งานยังอยู่ไหม?
3. ถ้า model เปลี่ยน ความหมายของ state เปลี่ยนไหม?
4. ถ้า process crash ก่อน/หลัง external call เรารู้ว่าจะทำอะไรต่อไหม?
5. ถ้า GUI ปิด Core ยังถูกต้องไหม?
6. ถ้า context ถูกล้าง intelligence ใหม่ทำต่อได้ไหม?
7. ถ้ามี duplicate/replay จะเกิด side effect ซ้ำไหม?
8. operator อธิบาย state ปัจจุบันจาก evidence ได้ไหม?

หากตอบไม่ได้ ควรถือว่ายังไม่พร้อมเป็น Core behavior
