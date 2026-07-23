# CertSafari Mock 2 — Focused Theory Study Plan

**Score:** 9/10 correct (90%) · **2 missed:** Q2 (Domain 3.2), Q11 (Domain 1.6)
**Frame:** Both misses are *"which mechanism / which structure is authoritative for this job?"* — you reached for the flexible, general-purpose option when the exam rewards the **specific, deterministic** one. Learn the two contrasts below, not the two answers.

> **Legend:** 🔴 *observed* = the choice you actually made · 🟡 *inferred* = the gap that likely caused it

---

## Meta-pattern across both misses (read first)

| Miss | You chose (flexible/adaptive) | Correct answer (specific/deterministic) |
|------|-------------------------------|------------------------------------------|
| Q2 | `SKILL.md` markdown body (general "instructions" container) | tool definition's **`description`** field (the *purpose-built* place for argument guidance) |
| Q11 | dynamic agentic decomposition (adaptive, agent decides) | **fixed prompt chain** (deterministic, you decide the steps) |

**The discriminating question for both:** *"Is the structure of this task already known and fixed? If yes, use the purpose-built, deterministic mechanism — don't hand the decision to a more general/adaptive layer."* Adaptivity is a cost (complexity, non-determinism, tokens), justified only when the task's shape is genuinely unknown at design time.

---

## Study order

1. **Q11 — Fixed vs dynamic chaining** (foundational orchestration concept; Domain 1)
2. **Q2 — Tool `description` vs skill `SKILL.md`** (artifact/mechanism distinction; Domain 3)

Do Q11 first: it establishes the "known structure → deterministic choice" reasoning that also explains Q2.

---

## 1. Fixed prompt chain vs dynamic decomposition — Q11 (Domain 1.6)

**Core theory.** Task decomposition has two families:

- **Fixed prompt chain (a.k.a. prompt chaining / multi-stage prompting):** *you* predefine an ordered sequence of focused sub-prompts; each step's output feeds the next. Deterministic, inspectable, reliable. **Use when the workflow is well-defined and the steps are known in advance.** Improves accuracy and consistency by letting the model focus on one narrow subtask at a time.
- **Dynamic / agentic decomposition:** the *agent* inspects the input at runtime and decides how to break the work down. Adaptive, but adds complexity, non-determinism, and error surface. **Use only when the structure can't be known ahead of time** (e.g., inputs of unpredictable shape that need runtime routing).

**The deciding test:** *Are the steps known before you see the specific input?*
- **Yes → fixed chain.** (Extract terms → verify usage → flag inconsistencies is a fixed 3-step pipeline; every merger agreement runs the same steps.)
- **No → dynamic.** Reserve autonomy for genuine uncertainty; don't pay for it when the pipeline is stable.

**Sequential vs parallel (the second half of this question).** A step that *consumes the previous step's output* has a **sequential dependency** — it cannot be parallelized. Here, *verification of terms requires the extracted terms first*, so extraction → verification must be sequential. Parallel processing is only valid for **independent** subtasks.

- 🔴 **Observed:** chose option C — dynamic decomposition where the agent autonomously identifies pages, extracts, then verifies. (Also flagged: leaning toward parallel processing despite the extract→verify dependency.)
- 🟡 **Inferred:** you may equate "complex / large document (200 pages)" with "needs an adaptive agent." Document *size* doesn't imply *unknown structure* — the task shape is fixed regardless of length, so a fixed chain wins.

**Contrast to hold:** *complexity of the input ≠ unpredictability of the workflow.* Fixed chain = reliability for known steps; dynamic = flexibility for unknown steps. Default to fixed when steps are enumerable.

**Self-check**
1. A workflow always runs the same three ordered steps on every input. Fixed chain or dynamic decomposition — and why? *(Fixed chain: steps are known in advance; determinism buys reliability with no downside.)*
2. Why can't "extract defined terms" and "verify their usage" run in parallel? *(Verification consumes the extraction output — a sequential data dependency; parallelism requires independent subtasks.)*

---

## 2. Tool `description` field vs `SKILL.md` — Q2 (Domain 3.2)

**Core theory — two different artifacts for two different jobs:**

- **Tool definition `description` field:** the authoritative place to tell the model *what the tool does, when to use it, and how to format each argument* (e.g., the HTTP method and endpoint path). Anthropic explicitly recommends **extremely detailed** tool descriptions to improve tool-call performance and **argument accuracy**. When the question is "how do I stop the model from generating incomplete/incorrect arguments for a tool?" → the answer is always the tool's `description`.
- **`SKILL.md` (Claude Code skill):** YAML frontmatter (`name`, `description`, `argument-hint`, `allowed-tools`, …) plus a markdown body — packages an **on-demand workflow/procedure** invoked by name. It is **not** the mechanism for documenting a general API tool's arguments.

**The distinction:** a *tool* is a callable capability the model invokes with arguments — its guidance lives in the tool definition's `description`. A *skill* is a packaged procedure for Claude Code. Don't route tool-argument guidance through skill files.

**Why the distractors are wrong (worth knowing for the exam):**
- **B — `argument-hint` in SKILL.md frontmatter:** `argument-hint` prompts for a *skill's* invocation parameters; it does not govern a general tool's argument generation.
- **C — `required-args` in `.claude/config.json`:** client-side validation *rejects* bad input after the fact; it does not *guide the model* to produce good input, and this isn't the documented mechanism.
- **D — markdown block in the SKILL.md body:** examples can help, but the primary, documented lever for tools is the `description` field; SKILL.md is for Claude Code skills, not generic tool definitions.

- 🔴 **Observed:** chose D — a detailed markdown block in the `SKILL.md` body.
- 🟡 **Inferred:** you're treating `SKILL.md` as the catch-all container for "instructions to Claude." Anchor rule: **argument accuracy for a tool → the tool definition's `description`, never a skill file.**

**Contrast to hold:** *tool = capability the model calls (guided by `description`); skill = procedure the user/model invokes by name (packaged in SKILL.md).* Improving how arguments are populated is always a tool-definition concern.

**Self-check**
1. Claude keeps passing a malformed endpoint path to a tool and wasting tokens. What do you improve? *(The tool definition's `description` field — detail what the tool does and how each argument is formatted.)*
2. What is `SKILL.md` actually for, and why is it the wrong home for tool-argument guidance? *(It defines an on-demand Claude Code skill/workflow invoked by name — not a general API tool definition; tool argument accuracy lives in the tool's `description`.)*

---

## One-page recall grid

| Concept | Wrong choice (flexible/general) | Correct choice (specific/deterministic) |
|---------|--------------------------------|------------------------------------------|
| Known, ordered workflow | dynamic agentic decomposition | **fixed prompt chain** |
| Step needs prior step's output | parallel processing | **sequential** chain |
| Large/complex *input* | assume it needs an adaptive agent | judge by *workflow* predictability, not size |
| Guide a tool's argument accuracy | SKILL.md body / frontmatter | tool definition's **`description`** |
| Package an on-demand workflow | tool `description` | **SKILL.md** |
| Reject malformed args after the fact | (config validation ≠ guidance) | guidance still belongs in **`description`** |

**Governing principle:** known structure → purpose-built, deterministic mechanism. Reserve adaptivity (dynamic decomposition) and general containers (SKILL.md) for cases the specific mechanism genuinely can't cover.

**Related notes:** `1.6-Task Decomposition & Prompt Chaining`, `3.2-Custom Slash Commands & Skills` · Cross-reference Mock 1 §4 (same tool-`description`-vs-skill distinction — recurring across exams).
