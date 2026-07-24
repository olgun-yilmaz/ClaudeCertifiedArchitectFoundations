# Claude Certified Architect – Foundations (CCAR-F)

A personal study kit for the **[Claude Certified Architect – Foundations](https://anthropic.skilljar.com/claude-certified-architect-foundations-access-request)** exam by Anthropic. It brings together everything used to prepare in one place: condensed domain notes, a fast-recall cheat sheet, recorded mock-exam attempts with per-question reviews, and a self-contained browser-based exam simulator with a matching practice-exam generator.

The certification measures the ability to design production systems with Claude — agentic loops, tool/MCP design, Claude Code workflows, prompt engineering, and context/reliability management.

## Exam blueprint

The exam is scenario-based: 4 scenarios are drawn at random from a fixed pool of 6 production contexts, with scored items distributed across five content domains. Scoring is criterion-referenced on a **100–1000** scale with a cut score of **720** (72%).

| # | Domain | Weight |
|---|--------|--------|
| 1 | Agentic Architecture & Orchestration | 27% |
| 2 | Tool Design & MCP Integration | 18% |
| 3 | Claude Code Configuration & Workflows | 20% |
| 4 | Prompt Engineering & Structured Output | 20% |
| 5 | Context Management & Reliability | 15% |

The full blueprint — domain weights, the six exam scenarios, and how the exam is scored — lives in [`Data/exam-details.md`](Data/exam-details.md).

## Repository layout

```
.
├── Notes/                    Study material
│   ├── Anthropic Certifications/   30 lesson notes across the 5 domains (+ annotated screenshots)
│   └── Cheat Sheet/                Last-minute recall: decision rules, scenario map, wrong-answer patterns
├── Exams/                    Recorded mock-exam attempts (questions, screenshots, results, reviews)
│   ├── CertSafari/                 Hard mocks (old + new question banks)
│   └── Claude Certification Guide/ Full + Quick mocks
├── ExamGenerator/            Browser-based exam simulator + generated practice exams
│   ├── ExamSimulator/              Zero-build ES-module app (index.html + styles/ + js/)
│   └── GeneratedExams/             Original CCAR-F practice exams (exam.md + answer-key.md)
├── Data/                     Exam blueprint + official exam guide PDF
├── scripts/sync-manifest.js  Regenerates the exam manifest from folders on disk
├── .githooks/pre-commit      Blocks commits when the manifest is out of sync
└── sources.md                Links to the course, mock exams, and the real exam
```

## What's inside

### `Notes/` — study material

- **Anthropic Certifications** — 30 lesson notes organized by the five domains (`1.1` … `5.6`), each pairing distilled prose with annotated screenshots and explicit **Exam Trap** callouts that name the common distractors.
- **Cheat Sheet** — fast-recall material for the final stretch: the [core decision rule](<Notes/Cheat Sheet/1-The Exam’s Core Decision Rule.md>), a scenario map, "if you see this, choose that" mappings, top wrong-answer patterns, out-of-scope distractors, and a last-minute memorization list — plus one condensed sheet per domain.

### `Exams/` — recorded mock attempts

Real practice runs kept as a study record, each with the question set (`exam.md`), result screenshots, and a `personalized-review.md` that traces every miss back to its root cause and the lesson to revisit. Sources include **CertSafari** (hard) and the **Claude Certification Guide** (full + quick) mocks.

### `ExamGenerator/` — simulator + practice exams

A self-contained, no-build exam simulator (ES modules loaded straight from `index.html`) plus a set of original CCAR-F practice exams. It presents one scenario card at a time with a side navigator, then produces a domain-weighted score out of 1000 with a PASS/FAIL verdict and a per-question review with per-option rationale and source lessons. See [`ExamGenerator/README.md`](ExamGenerator/README.md) for the full architecture, file formats, and scoring model.

Run it over a local static server (browsers block `fetch()` over `file://`):

```bash
npx serve ExamGenerator
# or
python -m http.server --directory ExamGenerator 5500
```

Then open the printed localhost URL and load `ExamSimulator/index.html`.

## Adding a practice exam

Just ask Claude to generate one — e.g. *"generate a new practice exam"*. That triggers the **`cert-exam-generator`** skill (in `.claude/skills/`), which asks at most two things:

1. **How many questions?**
2. **Which domains — a specific focus, or a weighted spread across all five?**

From there it does everything: authors `exam.md` + `answer-key.md` grounded in the repo's notes, applies the CCAR-F question-quality rubric, registers the id, and regenerates the manifest. The new exam is then selectable in the simulator.

A tracked pre-commit hook keeps `manifest.json` in sync and blocks any commit where it doesn't match the folders on disk. Enable hooks once per clone (git doesn't honor `.githooks/` until pointed there):

```bash
git config core.hooksPath .githooks
```

File formats and the manual authoring path are documented in [`ExamGenerator/README.md`](ExamGenerator/README.md).

## Sources

The course, mock-exam providers, and the official exam link are collected in [`sources.md`](sources.md). Recommended starting point: the [Anthropic Certifications course](https://www.anthropiccertifications.com/courses/claude-certified-architect-foundations).

---

*Personal study material. Not affiliated with or endorsed by Anthropic; “Claude” and related marks belong to Anthropic. Recorded mock questions belong to their respective providers and are kept here only as a personal study record.*
