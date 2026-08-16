# Architecture Notes

## 1. The problem this design solves

Long-running AI workflows degrade in a specific way: instructions get duplicated across files, the copies drift, and each session obeys whichever copy it happens to read. In this system's earlier life, writing rules lived in six places; sampled outputs from three points in time each complied with the rules *of their generation date*: early outputs violated rules that later files had introduced, and vice versa. Patching outputs would have fixed nothing. The fix was structural.

## 2. Source-of-truth hierarchy

```
1. Candidate dossier:     FACTS + dated framing decisions. Newest dated decision wins, everywhere.
2. Capability playbooks:  PROCESS (discover / tailor / prep). No facts, no style rules.
3. Style guide:           STYLE + build conventions, exactly one copy. Playbooks point to it.
```

Three rules make it hold:

- **No restating.** A playbook may reference the style guide but never paraphrase it (paraphrases fork).
- **Dated decisions.** Every fact-level decision in the dossier carries its date; conflicts resolve by recency, mechanically.
- **A frequently-broken-decisions checklist** in the umbrella skill: the handful of decisions that history shows get violated most (a de-hedged metric, an authorship correction, a naming rule) are re-checked on every output.

## 3. The discovery → positioning → tailoring feedback loop

Discovery runs produce dated reports (tiered, URL-verified openings). But reports are point-in-time; their strategic content decays silently. The loop fixes that:

1. Every discovery report ends with a required **Market signals** section (lane confirmations/shifts, hard gates observed, sponsor flags, one-sentence positioning implication).
2. A mandatory final step applies an **update protocol** to `market-positioning.md`: confirm or revise a bounded "Current posture" section (~60 lines max, so it stays readable in one pass), stamp the date, append a changelog line citing the report.
3. Every tailoring run reads `market-positioning.md` after the dossier. A dedicated step asks: *which demand lane is this role in, and what is the winning evidence line for that lane?* The JD defines the floor; the positioning file defines what differentiates against the rest of the applicant pool.

Properties worth copying: the file is **rewritten, not appended** (no unbounded growth); it is **evidence-cited** (every claim traces to a dated report); and it is **wired into both ends** (producer must update it, consumer must read it) so it cannot silently go stale.

## 4. Hard gates before generation

Generation is the expensive step, so screening runs first, in order of cheapness:

1. **Avoid-list filter.** Employers excluded for compliance or strategic reasons; refusal surfaces the rationale and offers alternates.
2. **Literal-minimums check.** Titled-years requirements ("8 years as X") are treated as literal gates, not negotiable suggestions; if the dossier has no titled years, the pipeline stops and says why.
3. **PERM-pattern detector.** A posting with hyper-specific enumerated multi-year minimums plus a salary band only ~$1–3K wide is flagged as likely drafted around an incumbent; auto-skip.
4. **Scope check.** People-management scope with no people-management record: stop.
5. **Dedupe + sequencing.** Check for existing packages at the same employer; stagger multiple applications into the same org rather than blasting simultaneously.

Negative evals assert these refusals fire. A pipeline that only measures its generations will quietly learn to generate its way past its own guardrails.

## 5. Evidence discipline in generation

- Resume bullets are never invented, only reordered, condensed, expanded, or rephrased from an approved baseline.
- Every cover-letter claim carries an artifact, number, or outcome; bare capability assertions ("I'm familiar with…") are banned.
- Untraceable claims are emitted as flags ("Unverified: needs confirmation"), not prose.
- Stretch matches are acknowledged or omitted, never implied.
- Where the employer publishes an AI-usage policy, the system produces analysis inputs (alignment tables, evidence maps) and the human writes the draft.

## 6. Build pipeline

Markdown is the working format; deliverables are docx/pdf built by a deterministic converter (fonts, spacing, justification encoded once, in code). Draft `.md` lives in a scratch directory and never ships. The deliverable folder contains only built artifacts, so there is exactly one rendered truth per package.

## 7. Evals

The suite covers: one happy path per capability, format-compliance assertions (page counts, banned characters, banned content classes), the feedback loop (positioning file read on tailor, updated on discovery), and two negative tests (avoid-list refusal; unmeetable-minimums refusal). Evals are re-run after any rule consolidation. They are the regression net that makes refactoring the "codebase of prose" safe.
