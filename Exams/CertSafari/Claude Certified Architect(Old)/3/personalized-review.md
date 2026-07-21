# CertSafari Mock 3 — Focused Theory Study Plan

**Score:** 15/20 correct (75%) · **5 missed:** Q6 (1.2), Q9 (5.5), Q10 (1.7), Q16 (3.5), Q20 (2.5)
**Weakest domain:** Domain 1 — Agentic Architecture & Orchestration (2/4, *Review*). All others 3–4/4.

**Governing frame.** Every wrong answer this round picked a mechanism that **flattens or ignores the real structure of the problem**; every correct answer **matched the mechanism to that structure**:

| Miss | The structure you had to respect | You flattened it to… | Correct = match the structure |
|------|----------------------------------|----------------------|-------------------------------|
| Q6 | dependency **graph** (2 independent + 1 conditional agent) | one linear chain | DAG: parallelize independent, gate conditional |
| Q16 | one **causal chain** (JOIN dup → wrong AVG) | separate manual steps | one holistic root-cause prompt |
| Q20 | JSON **schema** + failure atomicity | flat lines of text | `jq` parse → temp file → atomic `mv` |
| Q9 | population **subgroups** (doc type × field) | one homogeneous pool | proportional stratified sampling |
| Q10 | **state validity** (world changed under you) | continuity via `/compact` | fresh session + structured summary |

> **Legend:** 🔴 *observed* = the choice you actually made · 🟡 *inferred* = the gap that likely caused it

**Note on Q6 vs Q16 (the trap):** they look contradictory — Q6 says *don't collapse into one step*, Q16 says *don't split into steps*. Both follow the same rule: **decompose along real seams, never arbitrary ones.** Q6's agents are genuinely independent → split (and gate). Q16's bug is one coupled cause-and-effect → keep together. Learn this pair as a unit.

---

## Study order

1. **Q6 — Sequential chain vs DAG orchestration** — foundational, Domain 1, drives everything below.
2. **Q16 — Holistic root-cause prompt vs step-by-step** — the decomposition contrast to Q6; study back-to-back.
3. **Q20 — Structural (`jq`) vs text (`grep`) editing** — same "respect the structure" rule, applied to data.
4. **Q9 — Stratified vs random/decile sampling** — respect the population's subgroups.
5. **Q10 — Fresh session vs resume + compaction** — respect state validity; distinct "freshness" axis.

Do 1–3 first (structure-awareness spine), then 4, then 5.

---

## 1. Sequential chaining vs DAG orchestration — Q6 (Domain 1.2)

**Core theory.** In a coordinator–subagent system, orchestration is defined by the **task dependency graph**, not by convenience:

- **Sequential chaining** — a linear pipeline A→B→C. Correct *only* when every step consumes the previous step's output. Its cost: it forces **independent** tasks to wait on each other (no parallelism) and, if you pass the raw user prompt down the chain, each agent must re-extract its own instructions (error-prone).
- **DAG execution** — model tasks as nodes with real edges. **Independent nodes run in parallel**; a node with an incoming edge runs only after its dependency completes. **Conditional invocation** means a downstream agent fires *only if* an aggregated result meets a condition.

**In Q6:** PortfolioAgent and MarketAgent are **independent** → run in parallel. DraftAgent depends on the **aggregated comparison** and should fire **only if underperforming** → conditional gate. The observed bug (DraftAgent hallucinates because it runs before the comparison) is the textbook symptom of a *missing dependency edge* — the coordinator broadcast to all three simultaneously.

**Deciding test:** *Draw the dependency graph. Are any tasks independent? Is any task conditional on an aggregate?* If yes to either, sequential chaining is wrong — it can neither parallelize the independent ones nor gate the conditional one.

- 🔴 **Observed:** chose **A** — chain all three sequentially and pass the raw user prompt down the chain.
- 🟡 **Inferred:** you treat "the DraftAgent ran too early" as an *ordering* problem (just serialize everything) rather than a *dependency + condition* problem. Serializing fixes the crash but discards parallelism and never adds the "only if underperforming" gate.

**Contrast to hold:** *sequential = forced linear order for genuinely dependent steps; DAG = parallel where independent, gated where conditional.* A premature-execution bug is a missing **edge/gate**, not a call to serialize the whole thing. Also: **never pass the raw user prompt down a chain** — the coordinator should route each agent its own scoped instruction.

**Self-check**
1. Two subagents need no data from each other; a third must run only when their combined result crosses a threshold. Which orchestration, and what are the two graph features? *(DAG — the two run in parallel; the third is a conditionally-invoked node gated on the aggregate.)*
2. Why doesn't "chain them sequentially" properly fix an agent that executes before its inputs exist? *(It imposes an order but adds no conditional gate and needlessly serializes independent work — it treats a dependency/condition problem as a mere ordering problem.)*

---

## 2. Holistic root-cause prompt vs step-by-step — Q16 (Domain 3.5)

**Core theory.** Iterative refinement ≠ "issue instructions one micro-step at a time." When a defect is a **single logical error with a clear cause→effect**, the highest-leverage prompt **states the root cause and the impact and asks for a comprehensive fix**, letting Claude choose the repair (dedup, `DISTINCT`, subquery, restructure). Giving the full logical context in one pass lets the model reason about the *relationship* between cause and fix; slicing it into prescribed steps hides that relationship and constrains the solution.

**In Q16:** the JOIN produces duplicate rows → `AVG(val)` is mathematically wrong. That is **one** coupled cause-and-effect. Correct answer (A): one prompt explaining *the duplicates cause the average to be wrong — fix comprehensively.* The wrong answer (C) prescribes "first add DISTINCT/GROUP BY, then separately adjust AVG" — fragmenting one causal unit into two steps you dictate.

**Deciding test:** *Is this one logical error, or several independent ones?* One coupled cause→effect → describe it holistically, one prompt. Several unrelated issues → then decompose. **Also avoid** the "give it the target number and iterate until output matches" trap (D): fitting to an expected value teaches nothing about the logic and overfits.

- 🔴 **Observed:** chose **C** — sequential step-by-step (DISTINCT first, then adjust AVG separately).
- 🟡 **Inferred:** you equate "iterative refinement" with "always break into smaller prescribed steps." Decomposition helps across *independent* problems, not *within* a single causal chain — there it starves the model of the context that produces the best fix.

**Contrast to hold (pair with Q6):** decompose along **real seams**. Q6: independent agents → split. Q16: one coupled bug → keep whole. Splitting a single cause-and-effect and merging independent tasks are the *same* mistake in mirror image — a structure mismatch.

**Self-check**
1. A query has one logical flaw whose cause and effect you can state in a sentence. Prescribe the fix in ordered steps, or describe the root cause and ask for a comprehensive fix? Why? *(Describe the root cause — the cause↔fix relationship is what Claude needs; step-by-step hides it and over-constrains the solution.)*
2. When *does* breaking a fix into steps help? *(When the issues are genuinely independent — separate seams — not when they're one coupled cause-and-effect.)*

---

## 3. Structural (`jq`) vs text-based (`grep`) file editing — Q20 (Domain 2.5)

**Core theory.** For **structured data** (JSON/YAML/etc.), edit through a **structure-aware parser** (`jq`), not by matching text lines. Line/offset editing assumes formatting is stable (whitespace, key order, one-object-per-line) — an assumption structured files don't guarantee — so it is fragile and can **silently produce wrong results**. Pair this with the **atomic write pattern**: write the modified output to a **temp file**, then **atomically `mv`** it over the original. If the process is interrupted or the transform fails, the original stays intact (no half-written/corrupt file).

**In Q20:** update `tax_rate` `0.05→0.06` for `state == "NY"` in a large JSON array. Correct (B): `jq` selects the right object structurally, writes to temp, atomic `mv`. Wrong (C): `grep` for the line, compute an offset, line-edit in place — breaks the moment formatting varies, with no safety guarantee. (A "load whole file, edit in context, `Write` back" is also weaker: no structural targeting, not atomic; D regex-replace shares the text-fragility flaw.)

**Deciding test:** *Is the file structured data?* → parse it as structure (`jq`), never as lines. *Could a failure mid-write corrupt it?* → temp-file-then-atomic-`mv`. Two independent guarantees: **correct targeting** + **safe replacement**.

- 🔴 **Observed:** chose **C** — `grep` line-number + line-based editor relying on current formatting.
- 🟡 **Inferred:** you optimize for "find the exact line fast" and trust the file's present layout. Robust editing optimizes for *format-independence* and *crash-safety*, which line matching provides neither of.

**Contrast to hold:** *text tools (`grep`/`sed`/line edits) for unstructured text; structure-aware tools (`jq`) for structured data.* And a durable update is always **temp + atomic move**, never in-place mutation of the live file.

**Self-check**
1. Why is `grep` + line-number replacement unsafe for a JSON field even when it "works" on the current file? *(JSON formatting — whitespace, key order — can change; line matching silently breaks or edits the wrong data, with no safety guarantee.)*
2. What does write-to-temp-then-`mv` protect against that editing in place does not? *(An interrupted or failed write leaving the original file corrupted — the atomic move keeps the original intact until the new version is complete.)*

---

## 4. Stratified vs random / confidence-decile sampling — Q9 (Domain 5.5)

**Core theory.** The sampling design must match the **monitoring goal**:

- **Detect novel/subgroup errors (Q9's goal)** → **stratify by the document characteristics where errors originate** (document type × extracted field) and sample **proportionally**. This guarantees every subgroup is observed, so a format change in one bank's letters-of-credit surfaces even if that segment is small. Correct answer: **B**.
- **Validate confidence calibration** (does a 0.95 score really mean 95% correct?) → stratify by **confidence decile**. This is a *different* goal; here it's a distractor (C), because format-driven errors are tied to document/field, not to the score bucket.
- **Simple random (D)** — treats the population as homogeneous; rare document types/fields get too few samples, so segment-specific errors are missed or detected late. Wrong even when overall confidence is high.
- Stratify by **reviewer (A)** — checks annotator drift, unrelated to detecting new format errors.

**Deciding test:** *What am I trying to catch — miscalibrated scores, or errors concentrated in a subgroup?* Calibration → confidence deciles. Subgroup/novel errors → stratify by the **feature that drives the errors** (here, document type × field). "Confidence is already high" is never a reason to skip stratification.

- 🔴 **Observed:** chose **D** — simple random 2% ("stratification unnecessary when confidence > 0.95").
- 🟡 **Inferred:** you conflate high aggregate confidence with uniform reliability, and (per your notes) confidence-decile stratification with subgroup coverage. Calibration ≠ error detection: a well-calibrated model can still be blindsided by an *unannounced format change* it has never seen — which only per-segment coverage reveals.

**Contrast to hold:** *random = uniform coverage of a homogeneous pool; stratified-by-decile = calibration validation; stratified-by-doc-type/field = subgroup/novel-error coverage.* Match strata to the **source** of the errors you fear.

**Self-check**
1. You want early warning of format changes in rare document types among high-confidence extractions. Which stratification, and why not simple random? *(Stratify by document type × field, sample proportionally — random under-samples rare segments and detects their errors late.)*
2. When *is* confidence-decile stratification the right design? *(When the goal is calibration validation — checking that scores in each decile match observed accuracy — not detecting format-driven errors.)*

---

## 5. Fresh session + structured summary vs resume + `/compact` — Q10 (Domain 1.7)

**Core theory.** Session continuity is a liability once the **underlying state has changed**. `/compact` *summarizes* conversation history to reclaim context space — it does **not** delete outdated facts; stale references survive compaction and can drive **hallucinations**. When the ground truth has shifted, the correct move is to **start a fresh session and hand Claude a structured summary** of what still matters (the crash cause, the new image tag) — deliberately excluding the now-invalid details. This mirrors the file-based / `CLAUDE.md` philosophy and the resume protocol: **externalize durable state**, initialize a clean context, don't patch a contaminated one.

**In Q10:** the engineer scaled to 0, patched the image, scaled up → **pod names and events in context are now stale**. Correct (C): new session + structured summary (old crash + new tag), because the old names/events would mislead. Wrong (A): resume + `/compact` — compaction may retain the dead pod names. (B/D lean on the model reconciling stale history — same contamination risk.)

**Deciding test:** *Has the real-world state the context describes changed since it was captured?* If yes → the value of continuity is negative; **discard and re-summarize**. If no (just running low on space) → compaction/resume is fine. Compaction manages *size*, not *staleness*.

- 🔴 **Observed:** chose **A** — resume and `/compact` to distill prior steps.
- 🟡 **Inferred:** you treat `/compact` as a general context-hygiene tool that also freshens state. It only shrinks; it can't know which retained facts are now false. Staleness is fixed by **externalizing state into a fresh session**, not by summarizing the stale transcript.

**Contrast to hold:** *`/compact` = shrink a still-valid context; fresh session + structured summary = replace an invalidated one.* Ask "is it too big, or is it wrong?" — size → compact; wrong → restart clean.

**Self-check**
1. After you change infrastructure so prior identifiers no longer exist, why is `/compact`-then-resume risky, and what's the safe alternative? *(Compaction summarizes but keeps stale references that cause hallucinations; start a fresh session with a structured summary that omits the invalidated details.)*
2. What problem does `/compact` actually solve, and what does it *not* solve? *(It reclaims context-window space by summarizing history; it does not remove now-false facts — it can't distinguish valid from stale.)*

---

## One-page recall grid

| Concept | Wrong choice (structure-blind) | Correct choice (structure-matched) |
|---------|--------------------------------|-------------------------------------|
| Coordinator w/ independent + conditional tasks | sequential chain, raw prompt passthrough | **DAG**: parallel independent, gate conditional |
| Premature-execution bug in orchestration | serialize everything | add the missing **dependency edge / condition** |
| One coupled cause→effect defect | prescribe fix in separate steps | **one holistic root-cause prompt** |
| Several independent defects | one vague prompt | decompose by **real seams** |
| Editing a JSON/structured field | `grep` line-number / regex text edit | **`jq`** structural parse |
| Durable file update | edit in place | **write temp → atomic `mv`** |
| Detect novel/subgroup errors | simple random; "confidence high, skip it" | **stratify by doc type × field**, proportional |
| Validate calibration | stratify by doc type | stratify by **confidence decile** |
| Context describes changed state | resume + **`/compact`** | **fresh session + structured summary** |
| Context merely too large | restart from scratch | `/compact` / resume |

**Governing principle:** *model the true structure of the problem — dependency graph, causal unit, data schema, population strata, state validity — and choose the mechanism that matches it.* The tempting distractors all flatten that structure into something simpler and uniform.

**Cross-exam continuity:** the recurring tool-`description`-vs-`SKILL.md` gap (Mock 1 Q7, Mock 2 Q2) did **not** appear this round — keep it warm anyway. This round's spine is *structure-awareness* (Q6/Q16/Q20) plus the distinct *state-validity* axis (Q10) and *sampling-goal* distinction (Q9). Weakest domain to drill: **Domain 1 (Agentic Architecture & Orchestration)**.

**Related notes:** `1.2-Multi-Agent Orchestration`, `1.7-Session State, Resumption & Forking`, `2.5-Selecting Built-in Tools`, `3.5-Iterative Refinement`, `5.5-Human Review & Confidence Calibration`.
