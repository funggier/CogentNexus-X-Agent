# CogentNexus Independent Control Plane — Architecture & Development Design Pack

> **สถานะเอกสาร:** Design Baseline / Living Specification  
> **ภาษาหลัก:** ไทย โดยใช้คำศัพท์เชิงสถาปัตยกรรมภาษาอังกฤษเป็น canonical term  
> **หลักการอ่าน:** คำว่า MUST / MUST NOT / SHOULD / MAY ใช้ในความหมายเชิง normative  
> **ขอบเขต:** CogentNexus เป็นโปรแกรมอิสระที่เป็น durable semantic control plane และเชื่อมระบบภายนอกผ่าน replaceable Adapters


## 1. จุดประสงค์ของชุดเอกสาร

ชุดเอกสารนี้รวบรวมและหลอมแนวคิดจากงานออกแบบก่อนหน้าให้เป็นฐานเดียวสำหรับการสร้าง **CogentNexus เป็นโปรแกรมอิสระหนึ่งตัว** ไม่ใช่ส่วนขยายที่ฝังอยู่ใน Hermes หรือ OpenClaw โดย CogentNexus ทำหน้าที่เป็นศูนย์กลางรักษาความตั้งใจ สถานะ หลักฐาน สิทธิ์ การกู้คืน และการประสานงาน ส่วนระบบภายนอกทำหน้าที่เป็น “มือ” ที่สามารถเปลี่ยน ถอด เพิ่ม หรืออัปเกรดได้

เป้าหมายไม่ใช่เพียงให้ agent ทำงานได้ แต่ให้ระบบตอบได้ตลอดเวลาว่า:

- ผู้ใช้ต้องการอะไร
- ระบบยอมรับงานอะไรแล้ว
- ขณะนี้งานอยู่ในสถานะใด
- อะไรเป็น fact / observation / hypothesis / unknown
- อะไรทำไปแล้วและมีหลักฐานอะไร
- side effect ใดปลอดภัยที่จะ retry
- ใครถือ execution authority อยู่
- ผลลัพธ์ใด commit แล้ว
- ผลลัพธ์ใดส่งถึงผู้ใช้จริงแล้ว
- หาก model / Hermes / GUI / process หาย ระบบจะเดินต่อจากตรงไหน

แกนสำคัญคือ:

> **CogentNexus owns logical work. External runtimes perform bounded execution attempts.**

และ:

> **Knowledge may grow; active cognitive context must remain bounded and reconstructable.**

---

## 2. โครงสร้างเอกสาร

### `knowledge/` — องค์ความรู้และสถาปัตยกรรม

| ไฟล์ | เนื้อหา |
|---|---|
| `00-vision-principles-and-invariants.md` | วิสัยทัศน์ ปัญหาที่แก้ หลักแกน และ invariants |
| `01-canonical-ontology-and-state-model.md` | คำศัพท์ canonical, Ticket/Task/Attempt, state/transition model |
| `02-independent-control-plane-and-adapter-architecture.md` | Core/Adapter boundary และโปรแกรมอิสระ |
| `03-context-knowledge-evidence-and-recovery.md` | Context Compiler, Knowledge/Evidence, recovery semantics |
| `04-adapter-transport-and-runtime-contract.md` | ACK/replay/order/dedupe/fencing/runtime attempt contract |
| `05-single-model-scheduler-and-compute-envelope.md` | scheduler สำหรับโมเดลเดียว fairness/budget/quantum |
| `06-persistence-event-log-and-artifact-store.md` | SQLite, event log, artifact store, transaction/migration |
| `07-ingress-auth-authorization-delivery.md` | ingress/auth/authorization/ownership/delivery |
| `08-side-effect-enforcement-capability-and-hermes-mapping.md` | Effect Intent, capability, replay/reconcile, Hermes mapping |
| `09-python-core-service-and-gui-control-monitoring.md` | Python Core, asyncio, PySide6 GUI, control/monitor architecture |
| `10-reference-contracts-and-data-shapes.md` | canonical reference contracts, identities, data shapes และ compatibility rules |
| `11-threat-model-reliability-and-safety-considerations.md` | threat model, trust boundaries, fail-open/closed และ safety/reliability controls |

### `development/` — แผนการพัฒนาและการพิสูจน์ระบบ

| ไฟล์ | เนื้อหา |
|---|---|
| `00-development-strategy-and-v1-boundary.md` | กลยุทธ์, V1 scope, non-goals, gates |
| `01-foundation-bootstrap-and-project-layout.md` | repository/package/process layout และ coding discipline |
| `02-state-persistence-and-recovery-implementation-plan.md` | ลำดับสร้าง state engine/SQLite/recovery |
| `03-transport-adapter-and-protocol-implementation-plan.md` | transport protocol และ Adapter SDK |
| `04-scheduler-context-and-small-model-implementation-plan.md` | scheduler/context/small-model runtime |
| `05-hermes-adapter-discovery-qualification-and-rollout-plan.md` | integration แบบไม่ patch Hermes และ qualification |
| `06-gui-control-monitoring-and-operator-ux-plan.md` | GUI control/monitor roadmap |
| `07-test-chaos-security-and-acceptance-plan.md` | TDD, chaos, negative tests, security, acceptance |
| `08-release-migration-observability-and-operations-plan.md` | release, migration, backup, diagnostics, operations |
| `09-master-roadmap-and-definition-of-done.md` | roadmap รวม, dependencies, DoD, exit gates |
| `10-detailed-work-breakdown-and-dependency-map.md` | workstreams, dependencies, gates และ commit-sized work breakdown |
| `11-review-checklists-and-change-governance.md` | architecture review, capability/state/context/GUI checklists และ ADR governance |

---

## 3. Canonical vocabulary ที่ใช้ทั้งชุด

คำเหล่านี้ต้องใช้ความหมายเดียวกันทุกไฟล์:

- **Ticket** — durable admission root ของ intent จากผู้ใช้/ระบบ
- **Task** — หน่วย logical work แบบ recursive; งานใหญ่ scale โดย child Tasks
- **Task State** — semantic state ของงาน ไม่ใช่สถานะการใช้ CPU/model
- **Scheduler Job** — หน่วยที่รอแข่งขันเพื่อ resource
- **Execution Quantum** — ช่วง bounded execution ก่อนคืน control ให้ scheduler
- **Runtime Attempt** — การส่ง Task/Quantum ไปยัง runtime หนึ่งครั้ง เช่น Hermes
- **Model Run** — provider/model invocation หนึ่งครั้ง อาจมี 0..N ต่อ Runtime Attempt
- **Effect Intent** — durable authorization envelope ของ external mutation หนึ่งความหมาย
- **Effect Attempt** — การพยายาม execute Effect Intent เดิม
- **Delivery** — semantic intent ที่จะส่ง payload หนึ่งชุดไปยังผู้รับ
- **Delivery Attempt** — transport attempt ของ Delivery เดิม
- **Context Envelope** — derived bounded view ที่ intelligence ต้องเห็นในรอบนั้น
- **Artifact** — durable content-addressed output/reference
- **Observation** — สิ่งที่ตรวจพบ
- **Evidence** — material ที่ใช้สนับสนุน/หักล้าง claim
- **Claim / Inference / Decision / Unknown** — epistemic records ที่ไม่ควรปะปนกัน
- **Adapter** — boundary component ที่แปล semantics และ actuate ภายนอก
- **Capability** — semantic operation ที่ Core อนุญาต เช่น `repo.read`, `repo.push`
- **Generation / Fence** — monotonic authority token ป้องกัน stale owner

ห้ามสร้าง synonym ใหม่โดยไม่มี semantic responsibility ใหม่จริง

---

## 4. Shared architectural invariants

1. **Core is the durable semantic authority.**
2. **Adapters translate and actuate; they do not own Core truth.**
3. **State is not conversation.**
4. **Model output is a proposal until validated/committed.**
5. **Accepted work cannot silently disappear.**
6. **Completion requires evidence appropriate to the contract.**
7. **Transport may be at-least-once; semantic commitment must converge effectively once.**
8. **External ambiguity is represented as uncertainty, never guessed into success/failure.**
9. **No external mutation without a durable Effect Intent.**
10. **One small model must remain a valid operational baseline.**
11. **No required parallelism and no required multi-model consensus.**
12. **Context is a derived view, not durable memory itself.**
13. **Recovery must work without reconstructing private chain-of-thought.**
14. **Hermes is replaceable infrastructure; correctness must survive its upgrade/restart/session loss.**
15. **GUI/CLI are clients of Core authority, not alternate authority paths.**
16. **Stable semantic names are separated from protocol/schema/software revisions.**

---

## 5. Recommended reading order

สำหรับเข้าใจระบบ: `knowledge/00 → 01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09 → 10 → 11`

สำหรับเริ่มพัฒนา: อ่าน knowledge อย่างน้อย 00–02 และ 06–09 ก่อน แล้วจึง `development/00 → ... → 11`

---

## 6. วิธีใช้ชุดเอกสารนี้

- ใช้เป็น architecture source of truth ก่อนเขียน production code
- ทุก implementation decision ที่เบี่ยงจาก invariant ต้องมี ADR/rationale
- schema/protocol จริงอาจ evolve แต่ชื่อ semantic ไม่ควรเปลี่ยนตาม release
- acceptance ต้องพิสูจน์ด้วย evidence ไม่ใช่เพียง “code looks correct”
- ถ้า Hermes รุ่นใหม่เปลี่ยน behavior ให้ re-qualify Adapter; อย่าแก้ Core semantics เพื่อไล่ตาม runtime
