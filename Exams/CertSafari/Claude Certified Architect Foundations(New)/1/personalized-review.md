# CertSafari CCAR-F Mock 1 — Focused Theory Study Plan

**Score:** 17/20 correct (85%) · **3 missed:** Q17 (1.3), Q8 (3.5), Q4 (4.2)
**Frame:** The report says "no strong recurring pattern," but all three misses share one root: you collapsed a **two-part, relationship-aware answer** into a **single blunt move** — merging what should stay distinct, ordering by surface simplicity instead of causal relationship, or deleting what should be paired. The discriminator is never simplicity or bluntness; it's the actual relationship between the two things in play (source vs. destination, shared root cause vs. independence, true positive vs. false positive).

> **Legend:** 🔴 *observed* = the choice you actually made · 🟡 *inferred* = the gap that likely caused it

---

## Recurring meta-pattern (read first)

| Miss | You chose (blunt / single-channel) | Correct answer (relationship-aware / paired) |
|------|-------------------------------------|-----------------------------------------------|
| Q17 | per-call prompt folds into the **coordinator's** own system prompt; subagent only sees `AgentDefinition.prompt` | **both** prompts reach the **subagent** — persistent system prompt + task-specific detail, together |
| Q8 | order messages by which failure is *simplest*, treating grouping as secondary | group strictly by **causal interaction** — shared root-cause failures bundled, independent ones separate |
| Q4 | remove the rule entirely (destroys true positives along with false positives) | **pair** a problematic example with an acceptable look-alike to teach the boundary |

**The discriminating question for the whole exam:** *"What is the actual relationship between the two things here — do they combine, and on what basis?"* If you internalize this, all three collapse into one rule. Study these first.

---

## Study order

1. **Q17 — Subagent prompt composition** (foundational orchestration concept; establishes "which destination does information actually reach")
2. **Q8 — Grouping failures by causal interaction** (builds on Q17's routing logic — group by relationship, not surface order)
3. **Q4 — Paired few-shot examples for nuanced distinctions** (a different flavor of the same "don't collapse to one option" trap)

---

## 1. Subagent prompt composition — Q17 (Domain 1.3)

**Core theory.** A subagent invocation has two independent prompt sources, and **both** reach the subagent itself:

- **`AgentDefinition.prompt`** — the subagent's **persistent system prompt**: standing role, expertise, and constraints (e.g., SQL best practices, rollback strategy, data integrity checks) that apply on *every* invocation, regardless of the specific call.
- **Per-call / invocation prompt** — supplied by the coordinator **each time** it invokes the subagent, carrying the **task-specific** details for that run (e.g., which table, which migration).

Neither replaces the other, and neither is redirected elsewhere. The coordinator *authors* the per-call prompt, but it is *delivered to the subagent*, arriving alongside the persistent definition prompt — not folded into the coordinator's own system prompt.

**The trap you'll see:** confusing "the coordinator supplies the per-call prompt" (true) with "the per-call prompt stays with the coordinator" (false — it's addressed to the subagent). Distractors typically exploit two failure modes: (a) the per-call prompt gets merged into the *wrong* system prompt (the coordinator's, not the subagent's), or (b) the persistent definition prompt gets treated as *overridden* once a per-call prompt exists.

- 🔴 **Observed:** chose the option where the per-call prompt merges into the coordinator's own system prompt, so the subagent only ever sees `AgentDefinition.prompt`.
- 🟡 **Inferred:** you conflated "who crafts the per-call prompt" (coordinator) with "who receives it" (subagent) — the authorship and the destination are different things.

**Self-check**
1. Where does the per-call prompt supplied at invocation actually land? *(In the subagent's own context, alongside its persistent `AgentDefinition.prompt` — not the coordinator's system prompt.)*
2. Does supplying a per-call prompt turn off `AgentDefinition.prompt` for that run? *(No — it remains the persistent system prompt/role; the per-call prompt only adds task-specific detail on top.)*

---

## 2. Grouping failures by causal interaction — Q8 (Domain 3.5)

**Core theory.** When structuring iterative-refinement feedback across multiple test failures, the *only* valid grouping axis is **causal relationship**, never presentation order or perceived simplicity:

- Failures that **trace to the same underlying logic/state** → bundle into **one message**, so Claude can design a single coherent fix that addresses all of them at once.
- Failures that are **causally independent** → report **separately**, so an unrelated fix doesn't get entangled with — or delayed behind — a fix for something else.

**The deciding test:** *"Do these failures share root-cause logic?"* Yes → bundle. No → separate message. There is no legitimate reason to sequence messages by which failure is easiest to explain first — "simplest first" is not a grouping principle the exam rewards, because it has nothing to do with whether fixes interact.

**The trap you'll see:** introducing an artificial staging order ("address the simple one first, then combine the related ones") when the correct answer requires no ordering at all — just correct grouping. Prioritizing ease-of-explanation over shared-cause is exactly the anti-pattern: it can lead you to isolate an unrelated failure into its own *first* message unnecessarily, or delay bundling until later.

- 🔴 **Observed:** sequenced the malformed-JSON failure first because it was simplest, only combining the two deduplication failures afterward — grouping was reordered around simplicity rather than driven purely by shared root-cause logic.
- 🟡 **Inferred:** you treat "which failure to mention first" as a meaningful design choice; on this exam, order is irrelevant — only the bundle/separate split (driven by shared logic) is graded.

**Self-check**
1. Two failing tests both stem from the same deduplication-store bug; a third is an unrelated malformed-JSON parsing bug. How do you report these? *(Bundle the two store-logic failures into one message; report the unrelated parsing failure separately — no required ordering.)*
2. Why is "report the simplest failure first" not a valid organizing principle here? *(Simplicity has no bearing on whether fixes interact; only shared root-cause logic determines bundling.)*

---

## 3. Paired few-shot examples for nuanced distinctions — Q4 (Domain 4.2)

**Core theory.** When a rule/check is *fundamentally sound* but lacks nuance — producing false positives on legitimate look-alike cases while still catching genuine issues — the fix is to **teach the boundary**, not destroy the capability:

- **Add paired examples**: one genuinely problematic instance + one acceptable look-alike instance, each labeled with the correct verdict. This directly shows the model the specific distinction it's currently missing, reducing false positives *while preserving true positives*.
- **Removing the rule entirely** eliminates the false positives, but also eliminates the true positives — it solves the symptom by deleting the tool's actual function. This is never the right move when the rule catches *real* issues some of the time.

**The deciding test:** *is this a nuance/boundary problem, or is the rule fundamentally wrong?* If the rule still catches genuine issues (as here — the broad-except check is legitimate on its own), the fix is **contrastive few-shot pairing**, not removal. Reserve removal for rules that are wrong in essence, not merely imprecise.

- 🔴 **Observed:** removed the broad-except-clause check entirely.
- 🟡 **Inferred:** "too many false positives" is being treated as "the rule is bad, delete it," rather than "the model lacks a paired contrast to learn where the line is."

**Self-check**
1. A linter rule has real false positives on intentional top-level error boundaries but still catches genuine bugs elsewhere. Remove the rule, or add examples? *(Add paired problematic/acceptable examples with correct verdicts — preserves true-positive detection while teaching the nuance.)*
2. Why does removing the check entirely fail the actual goal (fewer false positives, same detection)? *(It also removes true positives — trading the tool's actual purpose for a lower false-positive count.)*

---

## One-page recall grid

| Concept | Blunt/wrong move | Correct, relationship-aware move |
|---------|-------------------|-------------------------------------|
| Per-call prompt destination | folds into coordinator's system prompt | delivered to the **subagent**, alongside its persistent prompt |
| `AgentDefinition.prompt` scope | overridden by per-call prompt | stays the **persistent** system prompt, every invocation |
| Multiple failing tests, shared cause | order by simplicity | **bundle** — same root-cause logic, one message |
| Multiple failing tests, independent cause | combine anyway | **separate** message — no shared logic |
| Rule with real false positives (still catches real issues) | delete the rule | **pair** problematic + acceptable examples |
| Rule that's fundamentally wrong | (would justify removal) | not the case here — nuance, not deletion, was needed |

**Related notes:** `1.3-Subagent Invocation & Context Passing`, `3.5-Iterative Refinement Techniques`, `4.2-Few-Shot Prompting`.

**Cross-exam link:** unlike this run, Mock 1 (Old cert) Q7 and Mock 2 (Old cert) Q2 both missed the *tool `description` vs `SKILL.md`* distinction — a separate, previously-flagged recurring gap. Not implicated in this exam's misses, but worth a periodic refresher given the repeat history.
