# Don't Build Employees, Build Constraints: A One-Person-One-Model Multi-Agent Collaboration Log (Archive v2)

One line: one person + one model, running a multi-agent collaboration system from incident-driven to institution-driven — don't build employees, build constraints.

**About this English edition.** This is a parallel reference translation of the Chinese master (`scrwzk2bzc-a11y/JamesChen`). Two things follow from that, and they matter:

1. **The Chinese master is the authoritative text.** Where this translation and the Chinese master disagree, the Chinese master wins. This edition is here so non-Chinese readers can follow the argument; it is not a second, independently maintained work.
2. **`attachments/` is shared, and is not duplicated here.** The attachments (19 files) already live in the Chinese repository. This repository ships the translated body text only — no `attachments/` directory, by design.

This is archive rewrite v2: it explains the design, and then explains how these constraints, over the five days that followed, pruned themselves — their own guard processes (the watchdogs), their own measuring stick (the kb line, a separate project), their own role table (the role reshuffle). Every number in the text carries a file path; run `stat` on that path and you can recompute it.

> **AI generation notice**: this piece was produced with AI assistance; the author reviewed it.

> **Disclaimer**: this is the author's personal practice log. It is not advice of any kind; if you copy it, you carry the risk.

> **Reposting and citation**: reposting and citation are welcome. Credit the source (repository URL) and the author. Do not strip or alter the attribution or the source.

## Reading paths
- Codes look opaque (pXX / rXX / L1-L3 / opsXX) → `appendix/appendix-a-d.md`, Appendix F "Codes and Glossary" (Appendix F is the reader's entry point; start there)
- Only want to avoid the pitfalls → `docs/09-踩坑.md`
- Want to copy the landing steps → `docs/10-落地清单.md` (ten steps, each with a verification)
- Want to know how the mechanism turns → `docs/05-协作机制.md` + `docs/06-狗件全流程.md`
- Reading straight through → go in directory order

## Contents
- `docs/` — 00 one-liner · 01 release-form design · 02 problem · 03 roles · 04 the three iron rules · 05 collaboration mechanism · 06 the watchdog end-to-end flow · 07 review close-out · 08 discussion and decision line · 09 pitfalls · 10 landing checklist · 11 conclusion
- `appendix/appendix-a-d.md` — Appendices A-F (F = codes and glossary, read it before you start)
- `glossary.md` — EN terminology table used while translating (source term → English rendering, domain, date coined)

The chapter files keep their original Chinese filenames from the master; content is English.

## How to recompute
Pick any number in the text, then `stat` / `wc -l` / `sha256sum` the path printed next to it. If a number does not check out, start at `docs/10` step 1 and verify the permission baseline first. Three worked examples: 54 ops documents (`ls` the live `materials/素材/运维文档/` path), the watchdog baseline hash (three-value comparison against the rollback file), the 25-item discussion-materials count (live `discussion-handoff/materials/` plus footnote).

## Contributing
Issues in English, with reproduction steps, file paths and dates.

## License and attribution
MIT (see `LICENSE`). Attribution: the author. Ops scripts that touch live infrastructure exist but are not published (owner's call).