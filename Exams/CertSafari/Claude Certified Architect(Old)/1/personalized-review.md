# CertSafari Mock 1 — Focused Theory Study Plan

**5 missed:** Q5 (2.2), Q17 (4.4), Q1 (5.4), Q7 (3.2), Q16 (3.5)
**Frame:** Every miss is a *layer-boundary* confusion — "who owns this responsibility?" The tool, the agent, the retry loop, the subagent, or the human. Learn the boundaries, not the answers.

> **Legend:** 🔴 *observed* = the choice you actually made · 🟡 *inferred* = the gap that likely caused it

---

## Recurring meta-pattern (read first)

The report says "no strong recurring pattern," but three of five misses share one root: **you delegated a responsibility to the wrong layer.**

| Miss | You put it on… | It belongs on… |
|------|----------------|----------------|
| Q5 | the agent (return error, let it retry) | the **tool** (retry internally) |
| Q17 | the human/halt | the **retry loop** (self-correct) |
| Q1 | the scratchpad (isolate output) | the **subagent** (isolate output) |

**The discriminating question for the whole exam:** *"At which layer does this failure/task actually get resolved, and can that layer resolve it?"* If you internalize this, all three collapse into one rule. Study these first.

---

## Study order

1. **Q5 — Tool vs agent error handling** (foundational; feeds Q17)
2. **Q17 — Semantic errors & retry loops** (builds on Q5's "which layer owns recovery")
3. **Q1 — Subagent vs scratchpad** (the isolation-vs-persistence pair)
4. **Q7 — Tool `description` vs skill frontmatter** (definition-artifact distinction)
5. **Q16 — Iterative refinement with failing tests** (technique-to-symptom matching)

---

## 1. Tool-level error handling vs agent delegation — Q5 (Domain 2.2)

**Core theory.** In the three-layer architecture (agent ↔ tool/connector ↔ external service), each error type is owned by a specific layer:

- **Transient errors** (429 rate limit, timeout, 503) → **handled inside the tool** with retry + exponential backoff. They are mechanically recoverable; the agent has no better information than the tool does, so bubbling them up just wastes reasoning tokens and turns.
- **Non-recoverable errors** (business rule violation, permission denied, and *genuinely* exhausted transient retries) → **returned to the agent** as a *structured* error (`category` + `isRetryable` + readable description) so it can escalate, explain, or change strategy.

**Why the tool, not the agent.** Separation of concerns: mechanical recovery is deterministic and belongs where the failure occurs; the agent is for *judgment*. A retry loop is not judgment.

**The trap you'll see** (per note 2.2): retrying an **empty result**. Distinguish *access failure* (service unreachable → retry may help) from *valid empty result* (search returned 0 → that IS the answer, never retry).

- 🔴 **Observed:** returned the 429 to the agent for it to retry.
- 🟡 **Inferred:** you may be treating "return structured error to agent" (correct for *non-recoverable*) as the universal pattern, missing that *transient* errors are resolved a layer lower.

**Self-check**
1. A tool hits a 429. Where is it handled, and what does the agent see if backoff eventually succeeds? *(Inside the tool; agent sees only the successful result.)*
2. Name one error the tool should NOT retry, and why. *(Business/permission — not retryable; needs escalation, not repetition.)*

---

## 2. Semantic validation error handling — Q17 (Domain 4.4)

**Core theory — the two error classes:**

- **Syntax errors** (malformed JSON, wrong types) → already eliminated by **tool_use / structured output** (4.3). Do NOT build retry loops for these.
- **Semantic errors** (valid format, *wrong content* — misread GPA, mismatched total, misplaced value) → **retry loop with specific feedback**: resend the original document + the failed extraction + the exact validation error so the model self-corrects.

**The deciding question** (from note 4.4): *Is the information actually THERE to get right?*
- **Present but wrong** → retry with feedback (recoverable).
- **Absent / needs external knowledge** → return `null` or flag for human. Retrying here causes **fabrication**.

So halting is correct *only* when data is absent — not for a semantic mismatch that the source can resolve.

**Schema self-correction trick:** return `calculated_total` alongside `stated_total` (and `conflict_detected` flags) so discrepancies are visible in the output and the validator just compares fields.

- 🔴 **Observed:** classified a semantic (present-but-wrong) error as halt-worthy.
- 🟡 **Inferred:** you're conflating "halt because absent" with "correct because wrong." The absent-vs-present axis is the real discriminator, not error severity.

**Self-check**
1. Transcript lists courses but the extracted GPA is wrong — retry or halt? *(Retry with feedback; the data is present.)*
2. Transcript never states GPA anywhere — retry or halt? *(Return null / flag human; retrying fabricates.)*

---

## 3. Subagent vs scratchpad role distinction — Q1 (Domain 5.4)

**Core theory — the isolation/persistence pair:**

| | **Subagent** | **Scratchpad file** |
|---|---|---|
| Primary job | **Isolate** a verbose/exploratory task so its raw output never enters the main context | **Persist** structured discoveries (classes, paths, dependency chains) to disk |
| Solves | Context *overflow / degradation* during the task | *Forgetting* key facts; resumability after a crash |
| Direction | Keeps main context **clean now** | Carries state **across time / sessions** |

Both fight context degradation (which is an **attention-quality** problem as verbose output accumulates — *not* a token-limit problem, so a bigger window does NOT fix it). But they attack it differently: **subagent = spatial isolation**, **scratchpad = temporal persistence**.

- 🔴 **Observed:** chose scratchpad to *isolate verbose output* and inverted the pair.
- 🟡 **Inferred:** you know both tools exist but map them to the wrong verb. Anchor: **subagent → isolate**, **scratchpad → remember**.

**Self-check**
1. An agent's file-by-file dump is flooding the main context mid-task. Which tool, and why? *(Subagent — isolates the verbose output.)*
2. You want a long codebase exploration to survive a crash. Which tool? *(Scratchpad / state manifest — persists structured state.)*

---

## 4. Tool definition vs skill definition — Q7 (Domain 3.2)

**Core theory — two different artifacts for two different jobs:**

- **Tool definition** → the **`description` field** guides the model on *what the tool does and how to fill its arguments* (e.g., HTTP method + endpoint path). Rich, specific descriptions directly improve **argument accuracy**. This is the answer when the question is "how do I make Claude call this tool with the right arguments?"
- **Skill (`SKILL.md`) frontmatter** → YAML (`name`, `description`, `argument-hint`, `allowed-tools`, `context: fork`) packages an **on-demand workflow**: discoverability/autocomplete, tool-restriction (security boundary), and isolation. Skills load *on demand*; they are not how you document a tool's arguments.

**The distinction:** a *tool* is a callable capability the model invokes with arguments; a *skill* is a packaged procedure invoked by name (`/review`) or auto-triggered. Argument guidance → tool's `description`. Workflow packaging → skill frontmatter.

- 🔴 **Observed:** chose a `SKILL.md` markdown block to improve argument accuracy.
- 🟡 **Inferred:** you're treating skills as the catch-all place for "instructions to Claude." Rule: if the question is about **how a tool's arguments are populated**, the answer lives in the **tool definition's `description`**, never in skill frontmatter.

**Self-check**
1. Claude keeps passing the wrong endpoint path to a tool. What do you improve? *(The tool definition's `description` field.)*
2. What is `argument-hint` in skill frontmatter for — vs the tool `description`? *(Prompts for a skill's parameters on invocation; not for guiding tool-call argument accuracy.)*

---

## 5. Iterative refinement with failing tests — Q16 (Domain 3.5)

**Core theory — match the technique to WHY the output is off:**

| Symptom | Technique |
|---------|-----------|
| Output varies run-to-run / prose read inconsistently | **2–3 concrete input/output examples** |
| Complex transformation, 50 unit tests failing | **Test-driven iteration: share the test FAILURES** (expected vs actual) |
| Unfamiliar domain, can't articulate needs | **Interview pattern** (have Claude ask first) |

For failing tests: the correct feedback is the **failure output itself** (expected vs actual) — unambiguous, targeted signal. Two anti-patterns:

- **Rewriting the test assertions** = throwing away your best feedback signal.
- **Modifying tests to match a buggy implementation** = violates TDD. **Tests encode desired behavior; you fix code to pass tests, never bend tests to mask bugs.**

Also (batch vs sequential): deliver **interacting** fixes in *one message*; deliver **independent** fixes *sequentially*.

- 🔴 **Observed:** chose to rewrite/modify test assertions.
- 🟡 **Inferred:** likely treating the goal as "make tests green" rather than "make code correct." The test suite is ground truth, not a variable.

**Self-check**
1. 12 of 50 parser tests fail. What do you give Claude? *(The failing tests' expected-vs-actual output — not rewritten assertions.)*
2. Two fixes depend on each other — batch or sequential? *(Batch, one message, so Claude reconciles them.)*

---

## One-page recall grid

| Concept | Wrong layer / choice | Correct owner |
|---------|---------------------|---------------|
| Transient error (429) | agent retries | **tool** retries (backoff) |
| Business/permission error | tool retries | **agent** escalates (structured) |
| Empty result | retry | it's the answer — stop |
| Semantic error, data present | halt | **retry loop** + feedback |
| Data absent | retry (fabricates) | `null` / **human** |
| Isolate verbose output | scratchpad | **subagent** |
| Persist facts across sessions | subagent | **scratchpad / manifest** |
| Guide tool argument accuracy | skill frontmatter | tool **`description`** |
| Package on-demand workflow | tool description | **skill frontmatter** |
| Failing tests | rewrite assertions | share **failure output** |
| Context drift to generics | bigger window | **attention** fix (scratchpad/subagent) |

**Related notes:** `2.2-Structured Error Responses`, `4.4-Validation & Retry Loops`, `5.4-Context in Large Codebase Exploration`, `3.2-Custom Slash Commands & Skills`, `3.5-Iterative Refinement Techniques`.
