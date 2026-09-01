# 11 — Review Checklists and Change Governance

> **สถานะ:** Engineering Governance Baseline  
> **วัตถุประสงค์:** ป้องกัน architecture drift เมื่อ codebase ใหญ่ขึ้น โดยใช้ checklist เบาแต่เข้มกับ semantic boundaries

---

## 1. Why governance matters

CogentNexus มี contract หลายชั้นที่ดูคล้ายกัน การ refactor เล็ก ๆ สามารถทำลาย recovery/authority โดยไม่เห็นใน happy path เช่นเปลี่ยน ACK semantics, ย้าย DB write เข้า Adapter หรือเพิ่ม raw shell path

Governance ไม่ควรเป็น bureaucracy; ใช้เฉพาะจุดที่ correctness meaning เปลี่ยน

---

## 2. Architecture change classification

### Class A — Internal Refactor
ไม่มี semantic/protocol/schema change; normal review

### Class B — Compatible Extension
optional field/capability/read model; compatibility tests

### Class C — Semantic Change
state meaning/guard/authority/replay behavior เปลี่ยน; ต้อง ADR + spec + migration/compatibility review

### Class D — High-Risk Boundary Change
side effect/auth/delivery/fencing; ต้อง negative+chaos tests และ explicit approval in project process

---

## 3. Pull-request architecture checklist

- [ ] ใช้ canonical vocabulary หรือมีเหตุผลสำหรับ term ใหม่
- [ ] Task state ไม่ปน scheduler/runtime state
- [ ] DB state mutation มี event/revision เมื่อ required
- [ ] external call อยู่นอก DB transaction
- [ ] retry semantics ระบุ identity/replay policy
- [ ] stale generation path tested
- [ ] secrets ไม่เข้า logs/events/context โดยไม่จำเป็น
- [ ] Adapter ไม่ broaden authority
- [ ] GUI/API command ไม่ bypass application service
- [ ] compatibility/migration impact ระบุ

---

## 4. New capability checklist

ก่อนเพิ่ม capability:

1. semantic name implementation-neutral หรือไม่
2. effect class
3. scope model
4. preconditions
5. expected evidence/postconditions
6. replay policy
7. reconciliation function
8. authorization/trust requirement
9. negative tests
10. Adapter mapping qualification

หากข้อ 5/6/7 ตอบไม่ได้ อย่าเปิด mutating capability ใน Managed Mode

---

## 5. New state/event checklist

- เป็นสถานะของ work จริงหรือ cognitive activity?
- แยก Phase/Status/Outcome แล้วหรือยัง
- transition guard machine-checkable แค่ไหน
- event เป็นสิ่งเกิดแล้วหรือ command?
- recovery ต้องทำอย่างไรเมื่อ crash หลัง transition
- terminal immutability มีไหม

---

## 6. Context change checklist

- เพิ่มข้อมูลเพราะ relevant จริงหรือเพราะ “เผื่อไว้”
- authority order ชัด
- mandatory/optional
- token cost
- sensitivity
- stale/superseded behavior
- audit explainability
- small-model benchmark regression

---

## 7. Hermes update checklist

- runtime identity changed?
- public seams changed?
- manifest mappings changed?
- yield/cancel/context behavior re-tested?
- negative tool bypass re-tested?
- delivery route behavior changed?
- session resume semantics changed?

ห้าม re-enable high-risk capability เพียงเพราะ smoke test success

---

## 8. GUI change checklist

- read-only projection หรือ command?
- command ผ่าน Core API?
- stale revision/generation handled?
- destructive scope shown?
- OUTCOME_UNKNOWN UI avoids blind retry?
- secret/content exposure reviewed?
- GUI crash/reconnect tested?

---

## 9. Naming governance

สร้าง glossary canonical terms และ lint/review warning สำหรับ public names:

```text
new old legacy final nextgen v2 v3 advanced
```

ไม่ได้ห้ามใน migration/internal temporary code แต่ห้ามกลายเป็น durable ontology โดยไม่ตั้งใจ

---

## 10. ADR triggers

ต้อง ADR เมื่อ:

- เปลี่ยน authority owner
- เปลี่ยน storage correctness model
- เปลี่ยน protocol guarantee
- เพิ่ม/ลบ semantic entity
- เปลี่ยน side-effect replay policy class
- ผูกกับ Hermes private behavior
- เปลี่ยน GUI/Core process topology

ADR ควรมี context/options/decision/consequences/reversibility

---

## 11. Release review checklist

- exact commit/package hash
- schema/protocol revisions
- migrations checksums
- all required gates green
- capability qualification current
- backup/restore tested
- no unresolved critical OUTCOME_UNKNOWN
- operator diagnostics usable
- release notes separate semantic vs implementation changes

---

## 12. Long-term architecture health indicators

สัญญาณว่า architecture เริ่ม drift:

- same concept มีหลายชื่อ
- Adapter code import Core persistence internals
- GUI SQL writes
- model prompt มี scheduler queue ทั้งหมด
- `process.exec` usage เติบโตแทน semantic capabilities
- recovery ต้องอ่าน transcript มากขึ้น
- retries แก้ด้วย sleep/try-again โดยไม่มี evidence delta
- version numbers เริ่มฝังใน class/table names

พบสัญญาณเหล่านี้ควรหยุด feature growth ชั่วคราวและ normalize architecture ก่อน debt แข็งตัว
