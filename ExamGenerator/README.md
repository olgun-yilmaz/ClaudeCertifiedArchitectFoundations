# ExamGenerator

Architecture for hand-authored Claude Certified Architect – Foundations (CCAR-F) practice exams, taken through `ExamSimulator/index.html`. Question generation itself happens by hand, in a separate chat session, sourced from `Notes/` and `Data/Claude-Certified-Architect–Foundations-Certification-Exam-Guide.pdf` — this folder only defines the format and the viewer.

## Layout

```
ExamGenerator/
├── ExamSimulator/           Browser-based exam simulator (ES modules, no build step)
│   ├── index.html           Entry point — markup + linked styles/ and js/
│   ├── styles/              CSS split by concern: tokens, base, layout, card, results, modal
│   └── js/                  ES modules: constants, state, utils, parse, scoring, result-md,
│                              theme, timer, storage, catalog, exam, render, main
└── GeneratedExams/
    ├── manifest.json        Flat JSON array of every exam id shown in the simulator's picker
    ├── <id>/
    │   ├── exam.md          Questions
    │   ├── answer-key.md    Correct answers + per-option rationale + sources
    │   └── result.md        Saved result report (written from the simulator after a run)
    └── _sample/             Format reference — registered in manifest.json like any other id,
                              so it's also browsable/selectable in the simulator
```

## The simulator (`ExamSimulator/`)

Self-contained, no build step (ES modules loaded from `index.html`). Once an exam is picked it presents **one scenario card at a time** — never a long scroll — with a **side navigator** (numbered chips) to jump to or skip any question. Each card's badge shows the precise section: `Q<n> · D<domain> · <subtopic> <name> · q-<d>-<s>-<nnn> · <scenario>`.

On **Finish**, a results screen shows a domain-weighted score out of **1000** with a **PASS/FAIL** verdict (pass mark 72% → 720/1000) and a per-domain breakdown, then lets you review every missed question with per-option explanations and its source lessons. See "Scoring" and "Saving results" below.

### Module layout

`index.html` carries only the static markup; the look and behavior are split into small files:

- **`styles/`** — the CSS, grouped by concern: `tokens.css` (theme variables), `base.css` (reset), `layout.css` (header + navigator + main), `card.css` (question card + choices + buttons), `results.css` (results + saved-result review), `modal.css` (mode modal + responsive).
- **`js/`** — ES modules with a single responsibility each: `constants.js` (domain taxonomy, weights, `BASE` fetch path), `state.js` (shared mutable state + DOM refs), `utils.js` (escaping, `fetchText`), `parse.js` (`exam.md` / `answer-key.md` parsers), `scoring.js` (1000-point weighted scoring), `result-md.js` (the `result.md` build/parse pair), `theme.js`, `timer.js`, `storage.js` (File System Access autosave), `catalog.js` (manifest + picker), `exam.js` (lifecycle + mode modal + save actions), `render.js` (all views), and `main.js` (entry point that boots everything).

## Adding a new exam

Recommended: ask the `cert-exam-generator` skill (`.agents/skills/cert-exam-generator/`) to author the exam — it writes both files in the correct format, registers the id, and applies the CCAR-F question-quality rubric. Manual steps, if you'd rather write by hand:

1. Create `GeneratedExams/<id>/exam.md` and `GeneratedExams/<id>/answer-key.md` following the formats below (copy `_sample/` as a starting point).
2. Append `<id>` to `GeneratedExams/manifest.json`.

## Domain taxonomy

Fixed, matches the rest of the repo (`Notes/`, `Exams/`) and the real CCAR-F weightings:

| # | Domain | Weight |
|---|--------|--------|
| 1 | Agentic Architecture & Orchestration | 27% |
| 2 | Tool Design & MCP Integration | 18% |
| 3 | Claude Code Configuration & Workflows | 20% |
| 4 | Prompt Engineering & Structured Output | 20% |
| 5 | Context Management & Reliability | 15% |

## `exam.md` format

All content in English. The simulator (`ExamSimulator/js/parse.js`) parses this line-by-line with fixed rules — keep the shape exact. `Subtopic:` and `Scenario:` are optional metadata lines that, when present, must come immediately after `Domain:` (in that order). Files without them still parse; the card just falls back to the domain-level label.

```markdown
# Exam <id>

Title: <short descriptive title>
Total: <question count>

## Q1
Domain: 1. Agentic Architecture & Orchestration
Subtopic: 1.4 Workflow Enforcement & Handoff
Scenario: Customer Support Resolution Agent

<Question stem — scenario + question, one or more lines/paragraphs>

A) <choice A>
B) <choice B>
C) <choice C>
D) <choice D>

## Q2
Domain: 5. Context Management & Reliability
Subtopic: 5.1 Managing Conversation Context
Scenario: Multi-Agent Research System

...
```

- Question blocks start with `## Q<n>` (sequential, starting at 1).
- The `Domain:` line must start with the domain number followed by `.` and match the taxonomy above.
- `Subtopic:` (optional) is `<d.s> <name>`, e.g. `1.4 Workflow Enforcement & Handoff` — the `<d.s>` id drives the section number shown on every card.
- `Scenario:` (optional) is a short scenario-family title shown in the card badge.
- Exactly four choices, `A)` through `D)`.

## `answer-key.md` format

One entry per question. Every field is a single line (no hard breaks). `Correct:` and `Explanation:` are required; `Why-A`…`Why-D` (per-option rationale) and `Source:` (one or more lesson references) are optional but recommended — the simulator renders them in the post-exam review and the saved report.

```markdown
## Q1
Correct: B
Explanation: <short summary rationale>
Why-A: <why A is wrong>
Why-B: <why B is correct>
Why-C: <why C is wrong>
Why-D: <why D is wrong>
Source: Lesson 1.4 — Workflow Enforcement & Handoff

## Q2
Correct: D
Explanation: <short summary rationale>
```

Domain/subtopic/scenario live only in `exam.md`; correctness, rationale, and sources live only in `answer-key.md` — the simulator cross-references the two by `Q<n>`.

Unanswered questions are scored as incorrect (no penalty for guessing), matching the real exam's rule.

## `result.md` format

Written automatically by the simulator after a run — not authored by the exam-generator skill, and not something you write by hand under normal use. The simulator parses a saved `result.md` back into a rich review screen whenever that exam is reopened, so this shape is a parser contract, not just a report style. See `GeneratedExams/_sample/result.md` for a full worked example.

```markdown
# QUESTIONS

## 1. Agentic Architecture & Orchestration (1/1)

> 🟢 Evaluation: 1/1 (All questions were answered correctly.)

---

## 3. Claude Code Configuration & Workflows (1/1)

> 🟢 Evaluation: 1/1 (All questions were answered correctly.)

---

## 5. Context Management & Reliability (0/1)

### 5.1 Managing Conversation Context
**Q2** · your answer: A · correct: C

<question stem>

- 🔴 Your answer (A): <raw choice A text>
- B) <raw choice B text>
- 🟢 Correct (C): <raw choice C text>
- D) <raw choice D text>

Why-A: <one-line rationale for A>
Why-B: <one-line rationale for B>
Why-C: <one-line rationale for C>
Why-D: <one-line rationale for D>

**Where this comes from:** Lesson 5.1 — Managing Conversation Context

---

# RESULT

**Score:** 758/1000 · PASSED · 2 of 3 correct (67%) · Pass mark: 720

| Domain | Correct | Points |
|--------|---------|--------|
| D1 Agentic Architecture & Orchestration | 1/1 (100%) | +435 |
| D3 Claude Code Configuration & Workflows | 1/1 (100%) | +323 |
| D5 Context Management & Reliability | 0/1 (0%) | +0 |
```

Rules:
- Two top-level sections, in order: `# QUESTIONS` (one block per domain present in the exam) then `# RESULT` (score summary + table).
- A domain answered perfectly gets a single `> 🟢 Evaluation: <correct>/<total> (All questions were answered correctly.)` blockquote and no question detail. A domain with any miss instead lists every missed question in that domain.
- Each missed question: an optional `### <d.s> <Subtopic name>` heading, then `**Q<n>** · your answer: <letter> · correct: <letter>`, a blank line, the question stem, then exactly one bullet per available choice (`A)`–`D)`) — tagged 🔴/🟢 for your answer / the correct answer — showing that option's **raw choice text**, never its explanation.
- Immediately after the bullets (separated by a blank line), one `Why-<letter>:` line per available choice with that option's one-line rationale. Keep this block separate from the bullets above it — folding the explanation into the bullet text instead of its own `Why-<letter>:` line breaks the review screen's per-choice explanation toggle (each choice always shows its raw text; the explanation is revealed on click, correct/your-answer included).
- `**Where this comes from:**` is optional, one line, joining all sources with `; `.
- `# RESULT` has one `**Score:** <score>/<total> · PASSED|FAILED · <correct> of <totalQ> correct (<pct>%) · Pass mark: <pass>` line, then a `| Domain | Correct | Points |` table with one row per domain present in the exam.
- All of this is generated verbatim by `buildResultMarkdown()` and read back by `parseResultReport()`, both in `ExamSimulator/js/result-md.js`. If you ever hand-edit or hand-author a `result.md`, match this shape exactly.

## Scoring

Scoring is domain-weighted and scaled to **1000 points**. Each domain present in the exam contributes `1000 × weight / (sum of present weights)` points, earned in proportion to the fraction correct in that domain. With all five domains present the maxima are 270 / 180 / 200 / 200 / 150. The **pass mark is 72%** (720 / 1000). Domains with no questions in an exam are excluded and the remaining weights renormalize.

## Saving results

The results screen renders a `result.md` report matching the format used across `Exams/` (per-domain `(x/y)` breakdown with a green blockquote for perfect domains, each missed question expanded with per-option explanations and its sources, then a `# RESULT` score table — see `result.md` format above for the exact shape). Three ways to save it into `GeneratedExams/<id>/result.md`:

- **Copy report** — copies the markdown to the clipboard.
- **Download result.md** — downloads the file (move it into the exam folder).
- **Save to file…** — in Chrome/Edge on localhost, uses the File System Access API to write the file directly; pick `GeneratedExams/<id>/result.md`.

## Running the simulator

Browsers block `fetch()` of local files opened via `file://` (and ES modules only load over `http(s)`), so serve this folder over a tiny local static server, then open the simulator in the browser:

```
npx serve ExamGenerator
# or
python -m http.server --directory ExamGenerator 5500
```

Then visit the printed localhost URL and open `ExamSimulator/index.html`.
