# CertSafari CCAR-F Mock 2 — Focused Theory Study Plan

**Score:** 18/20 correct (90%) · **2 missed:** Q8 (1.3), Q13 (3.6)
**Domain breakdown:** D1 3/4 · D2 4/4 · D3 3/4 · D4 4/4 · D5 4/4 (only Domains 1 and 3 lost a point each)

**Governing frame (hypothesis, not confirmed).** With only two misses the report is right that there is **no confirmed recurring pattern yet** — two data points can't establish one. But both wrong answers share a shape worth *watching*: you **bolted a patch onto a flawed mechanism instead of replacing the mechanism at its source.**

| Miss | You chose (patch the symptom, downstream) | Correct (fix the mechanism, at the source) |
|------|--------------------------------------------|---------------------------------------------|
| Q8 | add a verification *step* onto the rigid procedure | throw out the procedure — prompt the **goal + quality bar** |
| Q13 | *filter* the unwanted config out of the output | stop it loading at all — `--bare` prevents **auto-discovery** |

In both, the thing you picked still leaves the broken mechanism running (rigid steps still execute; hooks/MCP still load and act) and merely dresses up the result. The correct answer changes the mechanism so the problem can't arise. Treat this as a **watch-item**, not a diagnosis.

> **Legend:** 🔴 *observed* = the choice you actually made · 🟡 *inferred* = the gap that likely caused it

---

## Study order

1. **Q8 — Goal-oriented vs procedural subagent prompts** (Domain 1.3) — foundational orchestration principle; how you *frame* a subagent's instructions governs everything it does.
2. **Q13 — CI/CD environment isolation with `--bare`** (Domain 3.6) — a config/isolation concept; study second, but note it rhymes with Q8's "fix the source, not the output."

---

## 1. Goal-oriented vs procedural subagent prompts — Q8 (Domain 1.3)

**Core theory.** A coordinator's prompt to a subagent should specify **the goal and the quality bar** ("find credible, relevant sources; report only what is verifiable"), not a **rigid step sequence** ("Step 1 search 5 articles, Step 2 open each, Step 3 extract the publish date, Step 4 return a table"). A fixed procedure hard-codes assumptions that don't always hold — that every source *has* a publish date, that *exactly five* is the right count. When reality deviates, a step-following subagent executes literally anyway and returns wrong or forced output (a table with blank/fabricated dates; five sources when six were better). A goal-framed subagent is free to **adapt**: gather more or fewer sources, omit a field that legitimately doesn't exist, and report what it can actually verify. The principle: capable LLM subagents should be steered by *intent and success criteria*; over-specifying steps suppresses their reasoning and makes them brittle to edge cases the coordinator never foresaw.

**When it applies on the exam.** Any item where a subagent/coordinator prompt is failing on **variable inputs, missing fields, or a hard-coded quantity**, and the answers contrast "make the procedure more precise" against "reframe around the goal." The reframe wins whenever the failures come from edge cases a fixed script can't anticipate.

**The trap you'll see (Q8's options).** Three of four answers keep the rigid procedure and try to *repair* it; only one abandons it:
- **A (chosen):** add a 5th step to double-check the publish date — still rigid, and useless when a source has *no* date to check.
- **C:** raise the count 5→6 — still a fixed number, wrong whenever the ideal count varies.
- **D:** expand to a granular 10-step script "to remove ambiguity" — doubles down on the exact rigidity causing the failure.
- **B (correct):** rewrite around the research goal + quality bar, letting the subagent adapt.

- 🔴 **Observed:** chose **A** — added a verification step to the fixed four-step procedure.
- 🟡 **Inferred:** you read the procedure as "almost right, needs one more guard" rather than "the procedural framing *is* the defect." A guard still presumes the field exists to verify; it can't handle legitimately-absent data or a variable source count. This is the "patch the symptom" reflex from the governing frame.

**Contrast to hold:** *procedural prompt = fixed steps, brittle to the unforeseen; goal-oriented prompt = intent + quality bar, adaptive.* More steps or a bigger number is never the fix for a procedure that breaks on edge cases — remove the rigidity, don't reinforce it.

**Self-check**
1. A subagent must handle a variable number of sources and sometimes-missing fields. Add more explicit steps/guards, or reframe around the goal and quality bar — and why? *(Reframe around goal + quality bar; fixed steps can't cover edge cases the coordinator didn't anticipate, whereas a goal lets the subagent adapt — vary the count, drop absent fields, report only what's verifiable.)*
2. Why doesn't adding a verification step rescue a rigid procedure that fails on missing data? *(The guard still assumes the field exists to verify; it does nothing when the data is genuinely absent or when the hard-coded count is itself wrong.)*

---

## 2. CI/CD environment isolation with `--bare` — Q13 (Domain 3.6)

**Core theory.** A Claude Code run is **reproducible across runners** only if it does not depend on whatever local configuration happens to sit on the machine that picks up the job — a stray MCP server in a shared `.mcp.json`, a hook left in a teammate's `~/.claude`, a project `CLAUDE.md`, skills, or plugins. All of these are normally **auto-discovered** from the working directory and home folder. `--bare` **disables that auto-discovery entirely**: nothing local is loaded, and only what you pass *explicitly* (e.g., `--append-system-prompt`, `--settings`) shapes behavior. Every runner therefore behaves identically.

The decisive distinction is **where in the pipeline the fix acts.** `--bare` acts at the **source** — the unwanted config never loads and never executes. Filtering the output acts **downstream** — the hooks and MCP servers are *still* auto-discovered and *still run* (a hook still executes its command; an MCP server still contacts its service); you've only hidden their events from the final text. The side effects, and therefore the behavior, still vary from runner to runner. Reproducibility is a **config-isolation** problem, not an output-cleanliness problem.

**When it applies on the exam.** Any CI/CD or "behave identically everywhere / unattended run" item where distractors offer to *filter, limit, or replay* output versus a choice that *prevents local config from loading*. Whenever the goal is determinism regardless of the host, prevention-at-source beats post-hoc cleanup.

**The trap you'll see (Q13's options).**
- **B (chosen):** `--output-format stream-json` piped through a filter that drops hook/MCP events — changes only what's *displayed*; discovery and execution still happen, so behavior isn't actually reproducible.
- **C:** `-p` mode with `--max-turns 2` — caps interactions but never prevents discovery/execution; arbitrary and non-isolating.
- **D:** `--continue` with a pre-recorded session — resumes saved state, yet the runner's local hooks/MCP can still be discovered in the new run; no clean guarantee.
- **A (correct):** `--bare` in print mode, passing only the explicit flags needed.

- 🔴 **Observed:** chose **B** — filtered the output stream to hide unwanted config.
- 🟡 **Inferred:** you equated "suppress the *visible output* of the stray config" with "prevent the stray config from *running*." That treats reproducibility as making the output look clean, when the requirement is that nothing environment-specific executes at all — the same "patch downstream vs. fix at source" reflex as Q8.

**Contrast to hold:** *`--bare` = no auto-discovery, nothing local loads or runs (source-level isolation); output filtering / turn limits / replay = the config still loads and executes, you only alter or cap the result.* Determinism requires that the components never activate, not that their traces are hidden.

**Self-check**
1. What does `--bare` disable, and why does that give cross-runner reproducibility that output filtering cannot? *(It disables auto-discovery of hooks, MCP servers, skills, plugins, and CLAUDE.md — nothing local loads or executes, so only explicit flags matter; filtering merely hides events while those components still run, so side effects and behavior still vary by runner.)*
2. A runner has a stray hook in `~/.claude`. Under `--output-format stream-json` + filter, versus `--bare`, does the hook execute? *(Under the filter: yes — it's discovered and runs; you just don't see its events. Under `--bare`: no — it's never discovered, so it never runs.)*

---

## One-page recall grid

| Concept | Patch-the-symptom (wrong) | Fix-the-mechanism (correct) |
|---------|----------------------------|------------------------------|
| Subagent failing on missing fields / variable counts | add a verification step; raise the count; add more steps | reframe prompt around **goal + quality bar** |
| Why fixed procedures fail | edge cases the script never anticipated | intent-driven prompt lets the subagent **adapt** |
| Reproducible CI/CD run across runners | filter output; cap `--max-turns`; replay a session | `--bare` — disable **auto-discovery** of local config |
| What `--bare` suppresses | (nothing — output still cleaned post-hoc) | hooks, MCP servers, skills, plugins, **CLAUDE.md** |
| Where the real fix acts | downstream (the output) | at the **source** (loading/execution) |

**Governing principle (watch, don't over-fit):** when a mechanism is producing bad results, ask *"am I changing the mechanism, or just cleaning up after it?"* Both misses this round chose the cleanup. Confirm across future mocks before treating it as a settled pattern.

**Related notes:** `1.3-Subagent Invocation & Context Passing`, `3.6-Integrating Claude Code into CI/CD`.

**Cross-exam continuity:** Mock 1 (Foundations New) had no overlap with these two misses (its spine was *relationship-aware, two-part answers*). The long-standing **tool `description` vs `SKILL.md`** gap (Old-cert Mock 1 Q7, Mock 2 Q2) did not appear here either — keep it warm regardless. Domains to keep an eye on: **1 (Agentic Architecture & Orchestration)** and **3 (Claude Code Configuration & Workflows)**, the only two that dropped a point this round.
