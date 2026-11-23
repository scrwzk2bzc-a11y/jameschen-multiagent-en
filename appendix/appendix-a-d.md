Signed: wh-v2 | ts: 2026-11-22T15:05:00+08:00 | phase-two appendix draft (Appendices A-G + README skeleton, plus the 11 body chapters). The answers to the 3 prerequisite items are in the "Gap Closure" section at the end.

> Note: paths are reproduced as-is; Chinese directory names are literal.

> Note: proposal titles in the F.4/F.5 tables are reproduced verbatim from the original source files (Chinese), for traceability; the adjacent "theme" column gives the English gist. Full originals are in `attachments/`.

# Appendix Draft · archive-v2

## Appendix A Seven-Category Skeleton Index (23 items)
Reuses the v1 Appendix A skeleton; paths rewritten to the published-repo layout: body text goes into `docs/`, and the index lists the seven categories' 23 items (01-架构 3 / 02-机制 7 / 03-踩坑 2 / 04-速查表 5 / 05-调查记录 2 / 06-分支记录 2 / 07-实测结果 2), all in place (checked on disk at 12:47). The raw-material directory keeps v1's one-line disclosure: the lore/古集 materials, the full-conversation originals (286MB), and the discussion work-area files do not ship with the repository.

## Appendix B Script Classification Table (see G6 for the baseline revision)
Listed in full across five categories by the on-disk count of **54 items** (item by item in section F of materials-mapping.md): watchdog/monitoring 11 / confidential placeholders 6 / ops tools 5 / proposal docs 29 / other 3. The 56-item baseline of v1 Appendix B is void; the two differences are registered in G6 (different classification boundaries + mirror files not double-listed). The six confidential scripts (overlap between the five feishu-* files + the apply/cleanup pair, per the mapping's six-item baseline) are listed by name only, content withheld.

## Appendix C Outcome Table
The 18-item terminal-state inventory of v1 has moved into internal records (not listed in the public draft); v2 adds four rows (verdicts in the three-category baseline): kb-retrieval (pass; belongs to kb, a separate project; acceptance pending Owner; listed here for the record only), discussion-summary first round (closed, unconditional pass), supplement round (closed, unconditional pass), dogwatch chain (conditional pass; the level downgrade to L2 signed by the Owner).

## Appendix D Environment Setup Quick Reference

### D.1 The Five Publication-Format Items
Backfilled with the five actually-existing items per §1's publication format: README.md, LICENSE (MIT, ruled to ship with the release), .gitignore, the docs/ directory skeleton. The compose item was not found to exist (a full-disk find returned zero hits); its row was removed rather than invented — keeping v1 Appendix D's "only list what exists" discipline.

### D.2 Host and Runtime Baseline (follow this and you can reproduce something similar)

Values from main's same-day live check (PowerShell / nvidia-smi / wsl --version / docker --version). Nominal values throughout, measured values in parentheses.

| Item | This machine's value | Why it is written / what it means for the reader |
|---|---|---|
| CPU | AMD Ryzen 7 3800X (8 cores / 16 threads) | concurrent multi-agent sessions ride on core count; 8C16T is the minimum comfortable tier for running several sessions at once |
| Memory | 128 GB (nominal) / 127.95 GiB (measured) | the headroom for multiple agent sessions + sandbox containers + a local model at the same time |
| GPU | NVIDIA TITAN RTX | the inference hardware prerequisite for the local model |
| VRAM | 24 GB | decides that only quantized-tier models can run locally (see D.3); also the hardware premise of the "local can't run big pipelines" conclusion |
| GPU driver | 617.42 | for readers to compare against their own driver version when troubleshooting |
| GPU power limit | long-term capped at 80%-85% (to extend lifespan) | the second reason local model inference is slow (the first is in D.3); the Owner's deliberate trade-off; no specific wattage was given, so none is derived |
| Disk | SSD, 2 TB (nominal) / 1907.7 GiB (measured) | the material disk and the shared bus both live on the local SSD; capacity decides how much raw material the archive can keep |
| Host OS | Windows 11 IoT Enterprise LTSC, Build 26100 | the ops loop runs on the host; the GBK/encoding pits (§9) all originate on this side |
| WSL | 3.0.5.0 (WSLg 1.0.79), kernel 6.18.58.1-microsoft-standard-WSL2 | the business loop (pm/wh/plan-jc) runs inside WSL2; one software root cause of the two-loop separation (§3) |
| Distro | Ubuntu (default, v2) + docker-desktop (v2) | the sandbox container base |
| Python | 3.14.4 | runtime for the dog scripts / kb scripts |
| Node | v24.20.0 | the agent runtime |
| Docker | 29.9.2 (consistent between host and WSL) | agents run inside containers, **without root** (git cannot be installed) — this is why the PM is limited to read + file writes, and heavy work must be dispatched to the execution role |
| CUDA | nvidia-smi available | the litmus test for local-model availability |
| GitHub reachability | HTTP 200 (reachable directly) | reachability of the publishing route |
| Network | WSL default route via Windows-side NAT (172-range private address), not a public IP | readers troubleshooting network issues should first identify the NAT range; no specific host location is written |

> Unit note: the OS and nvidia-smi report **binary units (GiB/TiB)**; vendors advertise **decimal (GB/TB)**. Same stick of memory: nominal 128 GB = measured 127.95 GiB; same disk: nominal 2 TB = measured 1907.7 GiB. VRAM 24,576 MiB = exactly 24 GiB. The body of this piece always uses the **nominal value**, with the measured value in parentheses.

> How to measure: Win32_VideoController.AdapterRAM reporting 4GB is a **32-bit field overflow** (inevitable on large-VRAM cards); the authoritative value comes from `nvidia-smi --query-gpu=name,driver_version,memory.total --format=csv`; memory/disk via PowerShell give GiB values — do not mistake them for GB.

> Redaction exemption note: the pattern-2 four-segment number regex hits the kernel version `6.18.58.1` (not an IP; human-adjudicated exemption); all other credential/host/IP classes score zero hits.

### D.3 Model Configuration and Concurrency Strategy

| Role | Loop | Model | Inference location | Why configured this way |
|---|---|---|---|---|
| discussion / main / om-jc | ops loop (host) | API model | cloud API | the ops lane is low-volume and needs online lookups |
| pm | business loop (WSL) | API model | cloud API | needs long-context reasoning + outbound messaging |
| wh / plan-jc | business loop (WSL) | qwen3.8-flash-next-iq3_s (local) | local TITAN RTX 24GB | **cloud concurrency was too rough** — the cloud concurrency quota cannot sustain running execution + review sessions at the same time |

Three things spelled out:
1. **Why the mix**: cloud API concurrency is capped; execution + review roles running together queue up, so those two were moved local, freeing quota for pm and the ops lane.
2. **The two compounding reasons local is slow**: ① 24GB VRAM only fits the IQ3_S quantization tier; ② the GPU is long-term power-capped to 80%-85% (trading lifespan; see the power-limit row in D.2). **Local being slow is not model weakness; it is a trade-off.**
3. **The cost**: local is slow but stable and concurrent; API is fast but queues — in essence, **trading speed for concurrency**.

## Appendix E Glossary
All v1 terms retained (MFT has a literal use-case in the source docs but its definition was never verified; this entry is marked unverified: the definition was removed, only the use-case source remains: docs/02-机制/执行链条.md) + v2 additions: dog/bark-ledger/self-baseline (§6), 鹅牙周期 (marked: deprecated baseline — goose-tooth cycle), golden/measuring-stick/trigram floor (terms of kb, a separate project; not covered in this archive, listed for the record only), sign-off exemption channel / fixture (§7/§10), md5-prefix byte-level self-proof (§8), activity criterion (§6 bug ②).

# README Skeleton (entry point of the published repo)
In One Sentence (00 in full) → reading routes (skimmers: pits only → docs/10; want to copy it → docs/11; the machinery → docs/05+06) → table of contents (11 chapters + appendix) → recomputation guide (the "stat every number along its path" promise + three samples) → participation norms (English issues / reproducible / path+date attached, per §1) → license and attribution (pending G1).

# Gap Closure (answers to the 3 prerequisite items)
- **G6**: done — Appendix B switched to the 54-item on-disk baseline, two differences registered; mapping section F note and the logic-chain fault-section citation chain closed.
- **G1 (repo name / MIT / attribution)**: Owner ruled at 14:56 — repo name as placeholder pending final draft, license MIT (LICENSE ships with the release), attribution follows the author's own stated practice without naming the agent (consistent with verification rule ①). Chapter 01, Appendix D, and the README were rewritten per the ruling.
- **G3 (ship the confidential scripts or not)**: Owner ruled = not shipped with the repository; the README notes "they exist but are not public"; Appendix B lists names only. Rewritten per the ruling.
- Incidental G2 (entity-name generalization) keeps the v1 baseline: the pjw/dog mechanism names stay, IM names generalized; G4 (discussion work-area files) — all body citations now point to /shared-side evidence, no dangling references; G5 pre-history stays one sentence, unchanged.

---

## Appendix F · Codenames and Terms (readers: start here)

This piece is an internal record; the body keeps the field codenames. The table below translates the codenames into plain language — come back to it whenever something is unreadable.

### F.1 Role Codenames (two loops, six roles)

| Codename | Full name / origin | Which loop | What it does | Must never do |
|---|---|---|---|---|
| Owner | the author | outside the loops (bystander, upstream of the decision chain) | sets direction, final adjudication, signs levels | never writes body text, never scores |
| discussion | discussion role (created 2026-11-21) | ops loop (host) | discusses and dispatches, same as pm | cannot exec |
| main | master-control role | ops loop (host) | execution + the ops/debug/fix lane, same as wh | never ghost-signs |
| om-jc | ops-lane reviewer (created 2026-11-17; a jc copy + web access) | ops loop (host) | reviews plans and code as one | never writes code, never edits code |
| pm | Project Manager role | business loop (WSL) | clarifies, grades, dispatches, format-verifies, closes out | never writes code, never does technical review |
| wh / wh-v2 | Workhorse role (the executor) | business loop (WSL) | writes and edits code strictly on instruction | no architecture decisions, no final review |
| plan-jc | business-lane reviewer | business loop (WSL) | reviews plans and code as one | never writes code, never edits code |
| code-jc | retired (historical) | — | review once split into a plan seat and a code seat; merged into plan-jc/om-jc after the 2026-11-22 baseline unification | — |
| jc | umbrella term for review roles | both loops | from the author's own name initials in the original text of 2026-09-04, later becoming the review role's name | — |

### F.2 Mechanism Names and Objects

| Codename | What it is |
|---|---|
| the trio | status.json / context.md / SPEC.md — the record files of every task |
| file bus | the /shared directory, the single source of truth; paths travel, not content |
| dog / 狗 | the watchdog (dog_watch.py): the final judge of heartbeats, silence, and off-disk actions |
| bark# | the alert line in dog-watch.log; the gate relies on it to decide whether intervention is allowed |
| pjw-gate.py | the gate script: no run without review, no intervention without a dog bark |
| pjw | the ops surface being gated (a mechanism name, not an entity name) |
| kb | the private knowledge-base retrieval script (kb.py, FTS5 + SQLite) |
| golden | the standard answer set for kb evaluation (kb_golden.tsv, 27 entries) |
| the three hashes | the sha256 comparison of baseline and pre/post-rollback values |
| MFT | master file table (definition unverified) |
| goose-tooth cycle (鹅牙周期, deprecated) | deprecated baseline (3 dog barks = 3 minutes) |

### F.3 Numbering Systems

| Codename form | Meaning | Example |
|---|---|---|
| L1 / L2 / L3 | task level (L1 direct write, L2 five-dimension pass line, L3 a higher line + Owner signature; specific thresholds are internal institution) | L2 |
| S1…S5 | the five small steps of retrieval optimization | S4v2 |
| R1 / R2 / R3 | the "true blocker" item numbers of a review | R1 |
| P3 / P4 / P5 | the improvement-item numbers of code review | P5 |
| G1…G6 | the gap-list numbers of archive v2 (pending Owner ruling) | G1 repo name/MIT/attribution |
| O1…O6 | om-jc review improvement items (second channel) | O1 the three-hash wording |
| B1 / B2 | the blocker items of the phase-two first review | B1 the dual chapter-numbering system |
| DS-08 / KB-27 / AR-02-04 | handoff line numbers in each task's context.md | KB-27 |
| AV2-JC-01 / AV2-OMJC-01 | review trace numbers (JC lane / OMJC lane) | AV2-JC-01 |
| ops3…ops20 | ops proposal numbers (see Appendix F.4) | ops19 |
| sup | short name of the discussion-summary supplement file | sup §1.5 |

### F.4 Proposal List (ops series)

The titles below are taken from the first line of each proposal file's body (each read from disk, not inferred from file names). Three files' body titles are more specific than the common short names and are listed at body baseline (differences noted in context).

| No. | Proposal title (body first line, read from disk) | Theme | Cited in |
|---|---|---|---|
| ops3 | O&M consolidation + project-based /shared layout | ops directory and file placement | docs/05 |
| ops6 | D1 bug consolidation & fix plan | first bug fixes | docs/09 |
| ops7 | first-failure root-cause codification | codifying rules | docs/07 |
| ops8 | p28 SPEC v2 resubmission & scoped execution | resuming interrupted tasks | docs/07 |
| ops9 | Master Proposal: consolidated remediation package (merges ops7/8) | consolidated remediation | docs/07 |
| ops10 | tool-call hygiene (why my own commands failed today) | tool-call hygiene | docs/09 |
| ops11 | 嗅探-看门狗最小可行实现（MVP） | watchdog minimal viable version | docs/05 |
| ops12 | 夹具隔离计数口径 + 建档权限规范补条 | fixture isolation and count authority | docs/07 |
| ops13 | 全 bug 分级修复方案（进 L3 前停机检查整改） | fix plan and permissions | docs/10 |
| ops14 | 狗生态同步 + heal 修复 + 业务/运维线配合 | watchdog cross-node sync | docs/06 |
| ops15 | 看门狗结构性判断体系（节点级时间预算 + 项目整体时间控制） | node-level watching | docs/05 |
| ops16 | 看门狗闭环修复（检测→告警→升级→响应→修复→验证→关闭） | the watching loop | docs/05 |
| ops17 | 三级升级看门狗（嗅探→业务内部→全面探查→介入） | heartbeat/detection/action tiers | docs/05 |
| ops18 | 项目制看门狗生命周期（放狗→寻狗→牵狗回家） | watching within the project cycle | docs/06 |
| ops19 | 看门狗可靠性 + 飞书通道（回溯补审） | watchdog reliability hardening | docs/06 |
| ops20 | 狗完整性执法（SOP 防绕过） | integrity enforcement and deployment | docs/06 |

Note: ops1/2/4/5 are not found on the material disk (only 3 and 6-20 actually exist, 16 in total); this table lists only files that exist — nothing invented.

### F.5 Timeline Index (sorted by content timestamps)

Each row's time is taken from the file's own declaration (ts line / context line / in-text time point); file-name dates are not treated as time sources. "Approximate" is the source file's own annotation. All review verdicts in this table use the three-category baseline (fail / conditional pass / unconditional pass), with no specific scores (Owner's 2026-11-23 baseline).

| Time (in content) | Codename/phase | What it is | Source (file + line) |
|---|---|---|---|
| 2026-11-17 16:10 | ops3 | proposal body cites the date of an Owner instruction (no ts line in this file) | attachments/ops3 line 4 (incl. source header; body line 3) |
| 2026-11-17T17:30 | ops6 | D1 bug consolidation & fix plan | attachments/ops6 line 4 ts (incl. source header) |
| 2026-11-17T18:32 | ops7 | first-failure root-cause codification | attachments/ops7 line 4 ts (incl. source header) |
| 2026-11-17T18:34 | ops8 | p28 SPEC v2 resubmission | attachments/ops8 line 4 ts (incl. source header) |
| 2026-11-17T18:42 | ops9 | Master Proposal consolidated package (merges ops7/8) | attachments/ops9 line 4 ts (incl. source header) |
| 2026-11-17T19:45 | ops10 | tool-call hygiene | attachments/ops10 line 4 ts (incl. source header) |
| 2026-11-18T12:00 | ops11 | 嗅探-看门狗 MVP | attachments/ops11 line 4 ts (incl. source header) |
| 2026-11-18T13:40 | ops12 | 夹具隔离计数口径 | attachments/ops12 line 4 ts (incl. source header) |
| 2026-11-18T13:56 | ops13 | 全 bug 分级修复方案 | attachments/ops13 line 4 ts (incl. source header) |
| 2026-11-18T13:59 | ops14 | 狗生态同步 | attachments/ops14 line 4 ts (incl. source header) |
| 2026-11-18T14:09 | ops15 | 节点级时间预算 | attachments/ops15 line 4 ts (incl. source header) |
| 2026-11-18T14:52 | ops16 | 看门狗闭环修复 | attachments/ops16 line 4 ts (incl. source header) |
| 2026-11-18T15:04 | ops17 | 三级升级看门狗 | attachments/ops17 line 4 ts (incl. source header) |
| 2026-11-18T15:19 | ops18 | 项目制看门狗生命周期 | attachments/ops18 line 4 ts (incl. source header) |
| 2026-11-18 19:53 | ops19 | 看门狗可靠性+飞书通道 | attachments/ops19 line 5 ts (incl. source header) |
| 2026-11-18 20:11 | ops20 | 狗完整性执法 | attachments/ops20 line 5 ts (incl. source header) |
| 2026-11-21 (approximate; the source's own date annotation is unusable) | dog lane R1 | v2 plan review: fail (score over the line, true blocker R1) | _ops/reviews/dogwatch-phase1-v2-review line 7 time + line 14 five dimensions |
| 2026-11-21 (approximate, same annotation) | v3 conditional pass | v3 plan conditional pass | same directory v3-review line 6 time |
| 2026-11-21 (approximate, same annotation) | P3/P4/P5 | three code-review improvement items | same directory patch-review line 7 time |
| 2026-11-21 17:03 | state-probe | dog state inventory (per discussion's 16:53 instruction) | _ops/tasks/_ops/dogwatch-state-probe line 3 |
| 2026-11-21 17:16 | rollback | rollback executed (per Owner's 17:15 instruction) | dogwatch-rollback in the same directory, line 3 |
| 2026-11-22 10:02 time point | observation period | bark-ledger at 7.5MB, actively growing | docs/06 line 19 |
| 2026-11-21T11:33:13 | discussion first round: unconditional pass | discussion-summary review pass | _ops/reviews/t-20261121-discussion-summary-review line 4 ts |
| 2026-11-21T13:02~13:25 | DS-08…DS-11 stretch | pm-verify → supplement dispatch → closed, three lines | _ops/tasks/t-20261121-discussion-summary/context.md lines 11/12/15 ts |
| 2026-11-21T13:18:26 | supplement round: unconditional pass | supplement review pass | same directory supplement-review line 4 ts |
| 2026-11-22T12:25 | archive task card created | task dispatched | this task's context.md line 3 ts |
| 2026-11-22T14:52 | phase one: unconditional pass | framework-review done (AV2-JC-01) | same file line 16 |
| 2026-11-22 14:56 | Owner passes the loop | G1 three-item ruling (declared in docs/01 body) | docs/01 line 17 |
| 2026-11-22T15:55 | phase-two first review: fail | B1/B2 blockers (AV2-JC-02) | context.md line 27 |
| 2026-11-22T17:16:14 | re-review: unconditional pass | stage2-recheck pass (AV2-JC-03) | same file line 37 |
| 2026-11-22T18:08 | om-jc: conditional pass | second-channel review (AV2-OMJC-01) | same file line 42 |
| 2026-11-22T20:56 | go live | closed-published (GitHub commit) | same file line 48 |
| 2026-09-04 | JC origin | the jc name comes from the author's name initials in the original text | docs/03 line 7 baseline (Appendix F.1 synced) |
| 2026-11-17 14:55 | om-jc created | built as a jc copy + web access | docs/03 line 26 |
| 2026-11-19 | permission incident | the docs subtree re-injected as root | docs/05 line 44 (body narrates the scene that day) |
| 2026-11-21 | discussion created | local agent created and connectivity-tested | docs/03 line 9 (citing sup §1.1-3 content baseline) |
| 2026-11-22 | two-loop six-role baseline | Owner unified the baseline; code-jc retired/merged | docs/03 line 59 |
| no content timestamp | retrieval lane S1-S5 | moved out of this archive (kb belongs to the separate _ops project); not in F.5 | see removed-kb-chapter/ and the _ops separate project |

### F.6 Codename Quick Reference (sorted by "saw the word, need the meaning")

| Codename | One line on what it is | Where to look |
|---|---|---|
| AR- | context handoff line numbers of the archive-v2 lane (this task) | F.3 |
| AV2- | archive-v2 review trace numbers (JC/OMJC lanes) | F.3; F.5 archive-lane rows |
| B1/B2 | the two blockers of the phase-two first review (dual chapter numbering / dangling references) | F.3; F.5 |
| code-jc | the retired split review seat (merged into plan-jc/om-jc) | F.1 |
| discussion | discussion role (ops loop) | F.1; §3 |
| dog / 狗 | the watchdog process and alert system | F.2; §6 |
| DS- | context line numbers of the discussion-summary task | F.3 |
| G1…G6 | archive-v2 gap-list numbers | F.3 |
| golden | the kb evaluation answer set (kb, a separate project) | F.2 |
| jc | umbrella term for review roles | F.1 |
| KB- | kb task context line numbers (kb moved out; record only) | F.3 |
| kb | the private knowledge-base retrieval lane (a separate _ops project) | F.2; §preface note |
| L1-L3 | task levels | F.3; §4 |
| main | master-control role (ops loop) | F.1; §3 |
| minirerun | the minimal full instance (wordcount walks the six-step chain) | attachment A-minirerun |
| MFT | master file table (definition unverified) | F.2 |
| O1…O6 | om-jc second-channel improvement items | F.3 |
| ops3…ops20 | ops proposal numbers (16 files actually exist) | F.4; attachments/ |
| channel account identifiers | the real identifiers that were redacted (feishu app IDs etc.; disclosed in the attachments verification table) | verification table .md |
| pjw / pjw-gate | the gated ops surface / the gate script | F.2; §6 |
| plan-jc | business-lane reviewer | F.1; §3 |
| P3-P5 | code-review improvement item numbers | F.3; §7 |
| pm | Project Manager role | F.1; §3 |
| R1-R3 | plan-review true blocker numbers | F.3; §7 |
| S1-S5 | the five retrieval-optimization steps (kb moved out) | F.3 (record only) |
| sup | short name of the discussion-summary supplement | F.3 |
| wh / wh-v2 | the Workhorse execution role | F.1; §3 |

---

## Appendix G · Research and References

Both the research files cited in the body and the uncited ones are listed here: two zones, 21 rows (G.1 seven items / G.2 fourteen items). The date column comes from file names (the research files' bodies carry no content timestamps).

### G.1 Ops-Docs Zone (7 files, material disk 运维文档/)

| File name | Date (file name) | Theme | Cited by which chapter / record only |
|---|---|---|---|
| survey-industry-2026-11-17.md | 2026-11-17 | industry survey | record only (uncited) |
| ai-firewall-survey-20261118.md | 2026-11-18 | AI firewall survey | record only (uncited) |
| sniff-watchdog-research.md | no date | sniffing-watchdog research | record only (uncited) |
| watchdog-industry-research.md | 2026-11-18 | watchdog industry research | cited → ch.06 (three noise-reduction items, industry isomorphism) |
| architecture-audit-20261118.md | 2026-11-18 | architecture audit | cited → ch.03 (environment-instance-level differences) |
| L1-L3-deep-review-20261118.md | 2026-11-18 | deep review of the three-level review | cited → ch.05/07 (review channels) |
| full-process-audit-20261118.md | 2026-11-18 | full-process audit | record only (uncited) |

### G.2 Discussion-Handoff Zone (14 files, material disk discussion-handoff/materials/)

| File name | Date (file name) | Theme | Cited by which chapter / record only |
|---|---|---|---|
| survey-queue-upstream-github-20261120.md | 2026-11-20 | queue governance + upstream repo survey | cited → ch.01 (issue norms) |
| private-kb-rag-survey-20261120.md | 2026-11-20 | private knowledge-base RAG survey | record only (moved out with the retrieval chapter; kb is a separate project) |
| ai-cashflow-survey-20261120.md | 2026-11-20 | AI cash-flow survey | one conclusion line cited → ch.09 (business surveys do not enter the piece) |
| prompt-engineering-survey-20261120.md | 2026-11-20 | prompt engineering survey | cited → ch.09 root causes |
| durable-execution-design-20261120.md | 2026-11-20 | durable execution design | cited → ch.09 close-out table |
| deep-research-method-digest-20261120.md | 2026-11-20 | deep-research methodology | record only (uncited) |
| project-postmortem-compare-20261120.md | 2026-11-20 | project postmortem comparison | record only (uncited) |
| project-postmortem-guji-20261120.md | 2026-11-20 | the 古集 project postmortem | record only (uncited) |
| project-postmortem-lore-20261120.md | 2026-11-20 | the lore project postmortem | cited → ch.08 (moved out; record only) |
| survey-single-source-20261120.md | 2026-11-20 | single source of truth survey | cited → ch.09 |
| survey-personal-tools-research-skills-20261120.md | 2026-11-20 | personal-tools research survey | record only (uncited) |
| skill-ownership-map-20261120.md | 2026-11-20 | skill ownership map | record only (uncited) |
| zhihu-notes-20261118-skill-trending.md | 2026-11-18 | Zhihu skill trends | record only (uncited) |
| lore-skill-survey-20261120.md | 2026-11-20 | lore skill survey | cited → ch.08 (moved out; record only) |

All 21 research files above stay archived on the material disk and do not ship with the repository (materials are not public); this table exists for the record, so readers know which conclusions rest on research and where to dig deeper. Wherever the body marks "(research file name)", that is the citation point.
