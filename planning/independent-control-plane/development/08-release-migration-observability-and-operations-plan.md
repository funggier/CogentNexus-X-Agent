# 08 — Release, Migration, Observability, and Operations Plan

> **สถานะเอกสาร:** Design Baseline / Living Specification  
> **ภาษาหลัก:** ไทย โดยใช้คำศัพท์เชิงสถาปัตยกรรมภาษาอังกฤษเป็น canonical term  
> **หลักการอ่าน:** คำว่า MUST / MUST NOT / SHOULD / MAY ใช้ในความหมายเชิง normative  
> **ขอบเขต:** CogentNexus เป็นโปรแกรมอิสระที่เป็น durable semantic control plane และเชื่อมระบบภายนอกผ่าน replaceable Adapters


## 1. Objective

ทำให้ระบบที่ durable จริงสามารถ upgrade/backup/diagnose ได้โดยไม่สูญ semantic authority

---

## 2. Version dimensions

แยก:

```text
Software Release
Protocol Compatibility Version
Schema Revision
Adapter Manifest Revision
Capability Mapping Revision
```

ห้ามใส่ version ใน conceptual object name

---

## 3. Release candidate discipline

ก่อน acceptance freeze:

- exact source commit
- build/package hash
- migration set checksums
- schema revision
- protocol revision
- Adapter versions/manifests
- reference model/provider config

ถ้า production source เปลี่ยน → candidate ใหม่, rerun gates

---

## 4. Upgrade procedure

1. pause unsafe new admissions/effects ตาม policy
2. backup/checkpoint
3. integrity check
4. apply forward migrations
5. start Core in recovery mode
6. re-establish Adapter handshakes
7. requalify changed mappings
8. recovery scan pending attempts/effects/deliveries
9. resume scheduler

---

## 5. Hermes upgrade operations

Hermes independent update allowed

CNX response:

- detect runtime identity
- invalidate affected mapping qualification
- keep Tasks/Tickets
- passthrough/native Hermes remains possible
- run qualification suite
- enable managed capabilities gradually

---

## 6. Observability

Structured operator data:

```text
Core health/schema
model slot
queue latency/fairness
Ticket/Task counts
Adapter heartbeat/epoch
runtime attempts
outcome_unknown effects
pending delivery
reconciliation backlog
DB WAL/size/integrity
artifact store size/orphans
```

Metrics ไม่ควรเป็น authority; ใช้เพื่อ monitor

---

## 7. Alerts

Critical examples:

- DB integrity failure
- unsupported schema
- duplicate identity hash mismatch
- repeated stale fence violations
- effect OUTCOME_UNKNOWN high-risk
- delivery ambiguity beyond policy window
- Adapter manifest changed high-risk mapping
- scheduler starvation bound exceeded

---

## 8. Backup/restore

- SQLite backup API/WAL-aware
- artifact store snapshot consistent enough with DB refs
- restoration test regular
- encrypted/sensitive data policy
- restore ต้อง run integrity + recovery scan ก่อน scheduler resumes

---

## 9. Operational CLI

ตัวอย่าง:

```text
cnx status
cnx ticket list/show
cnx task graph
cnx scheduler inspect
cnx adapter list/qualify
cnx effect list/reconcile
cnx delivery inspect
cnx context explain
cnx db integrity
cnx backup create/verify
cnx recovery scan
```

GUI เรียก application services เดียวกับ CLI

---

## 10. Data retention/cleanup

background maintenance only P3:

- IPC dedupe GC by committed watermark/retention
- old snapshots
- orphan temp artifacts
- bounded raw model payload
- telemetry

Never GC semantic effect/delivery evidence ที่ยัง referenced

---

## 11. Exit criteria

- upgrade/rollback-via-backup drill
- Hermes update degrade/requalify drill
- live backup restore
- operators can explain current state without raw conversation
- alerts are actionable and evidence-linked
