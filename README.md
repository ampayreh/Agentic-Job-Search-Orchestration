# Agentic Job-Search Orchestration

A production-grade example of using **Claude (chat + Cowork agentic sessions)** to orchestrate a multi-step knowledge workflow end to end: an AI job-search system built as a Claude skill, with capability routing, evidence-grounded generation, a market-signal feedback loop, and eval-driven quality control.

This repo is a **sanitized showcase**. The architecture, skill structure, style ruleset, and evals are the real system; all candidate data has been replaced with fictional examples.

## What the system does

One umbrella skill routes any career-related request to one of three capabilities, each with its own reference playbook:

| Capability                | Trigger examples                      | Output                                                                       |
| ------------------------- | ------------------------------------- | ---------------------------------------------------------------------------- |
| **Job discovery**         | "find SAP architect roles in Seattle" | Ranked, URL-verified, tiered discovery report                                |
| **Application tailoring** | "tailor my resume for this posting"   | Resume + cover letter + prep doc (docx/pdf), built from an approved baseline |
| **Interview prep**        | "prep me for the panel on Thursday"   | Company brief + role-fit table + STAR answer cue cards                       |

Every generated claim must trace to a **verified candidate dossier** (single source of truth). If a claim can't be traced, the system flags it for human confirmation instead of generating it.

## Why it's interesting as agentic orchestration

**1. Source-of-truth hierarchy with conflict resolution.** Facts live in one dossier; process lives in capability playbooks; style lives in one style guide. A precedence rule ("the newest dated decision in the dossier wins") lets the system resolve contradictions between files that drift at different speeds, which is the failure mode that actually degrades long-running AI workflows.

**2. A closed feedback loop between discovery and generation.** Every discovery run ends by updating a living `market-positioning.md` file (demand lanes, hard gates, tactical rules, distilled from that run's verified evidence). Every tailoring run starts by reading it. The result: applications are positioned against where the market is _moving_, not just against the text of one job description. The file is rewritten rather than appended (bounded at ~60 lines so it stays readable in one pass), every claim cites a dated report, and it is wired into both ends (the producer must update it, the consumer must read it) so it cannot silently go stale.

**3. Hard-gate screening before generation.** The pipeline refuses to spend effort on unwinnable applications: titled-years minimums the dossier can't meet, PERM-pattern postings (enumerated hyper-specific minimums + a ~$2K-wide salary band), compliance-gated roles, people-manager scope without the record. Refusals are surfaced with rationale and alternates, not silently dropped.

**4. Voice and register control at draft time.** Default model register is a real deployment problem: generic vocabulary, hedged claims, symmetrical rhythm, and restating the source back at the reader. In a document that has to represent one specific person, that register is a correctness failure, not a style preference. A single enforced ruleset constrains it at generation time rather than by post-editing (banned vocabulary, no capability assertion without a traceable artifact, no source-mirroring), with a 7-point self-check before any file is written. The ruleset lives in exactly one file in the live system and is never restated elsewhere; that constraint is the point, so this repo describes it rather than shipping a second copy to drift against.

**5. Eval-driven regression control.** The skill ships with evals ([`evals.example.json`](evals.example.json)) including negative tests: the system must _refuse_ correctly (avoid-listed employer, unmeetable minimums) as reliably as it generates correctly.

**6. Human-in-the-loop by design.** Stretch claims are flagged for keep/remove decisions; unverifiable facts are marked for confirmation; where an employer has an AI-usage policy, the system limits itself to analysis inputs so the human writes the first draft.

## Architecture

```mermaid
flowchart TD
    U[User request] --> S[SKILL.md - umbrella router]
    S -->|find roles| D[discover-jobs playbook]
    S -->|tailor application| T[tailor-application playbook]
    S -->|prep interview| P[prep-interview playbook]

    DOS[(Candidate dossier - verified facts)] --> D
    DOS --> T
    DOS --> P
    MP[(market-positioning.md - living market read)] --> D
    MP --> T
    STY[(Style guide + register ruleset)] --> T
    STY --> P

    D -->|tiered report| R1[Discovery report]
    R1 -->|update protocol| MP
    T -->|scratch md -> build| R2[Resume + letter + prep docx/pdf]
    P --> R3[Interview brief]

    E[Evals incl. negative tests] -.regression-check.-> S
```

Full design notes: [`ARCHITECTURE.md`](ARCHITECTURE.md).

## How it was built

Built iteratively with **Claude Cowork** (desktop agentic sessions over the local file system) and **Claude chat**:

- The skill and its reference files are plain Markdown. The "code" of the system is structured natural language, versioned like code.
- Cowork sessions did the heavy operations: sweeping ~390 live postings on an ATS board with per-JD verification fetches, sampling generated packages to diagnose quality drift, consolidating six drifting rule files into a strict hierarchy, and rewriting the skill in place.
- A later optimization pass (also run as an agentic session) found the systemic failure mode (rules duplicated across files obeying different vintages) and fixed it structurally rather than patching outputs. The diagnosis method: sample early / mid / recent outputs and diff them against the current rule set.

## Measured behavior (from the live system)

- 49 tailored application packages generated over ~7 weeks, each with alignment analysis and JD snapshot.
- Output drift diagnosis: early packages violated 6+ of the current hard rules; the newest violated ~2 soft rules. Improvement tracked to rule consolidation, not model change.
- Discovery runs enforce URL verification (live/unverified/dead labels) after SERP-sourced dead links were caught in an early eval.

## Repo contents

| File                                       | What it is                                                                                              |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------- |
| [`ARCHITECTURE.md`](ARCHITECTURE.md)       | Design notes: source-of-truth hierarchy, feedback loop, hard gates, evidence discipline, build pipeline |
| [`evals.example.json`](evals.example.json) | Eval suite structure, including negative tests                                                          |
| [`LICENSE`](LICENSE)                       | MIT                                                                                                     |

> This repo describes the system; it does not vendor copies of its rule files. Earlier versions shipped sanitized duplicates of the skill file, the style ruleset, and the positioning file. Those duplicates were removed in the 2026-07-25 consolidation for the reason the project exists to demonstrate: a second copy of a rule is a copy that drifts. `ARCHITECTURE.md` carries the design; the live files stay single-copy in the private system.

## Adapting it

The pattern generalizes to any recurring document-generation workflow with a truth source and a quality bar (sales proposals, grant applications, compliance responses): put facts in one dossier, process in per-capability playbooks, style in one enforced ruleset, wire outputs back into a living context file, and hold it together with evals that test the refusals as much as the generations.

## License

MIT. See [LICENSE](LICENSE).
