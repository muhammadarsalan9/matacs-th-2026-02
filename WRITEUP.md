# Write-up: AI Proposal Triage (MATACS-TH-2026-02)

**Assumptions.** "Since 2021" counts as inside the 5-year window. A proposal with no committed schedule cannot be checked against the 16-week limit, so it is flagged. Scores below are on a 0-100 scale.

## 1. AI vs. rules
**AI is used for one job only: reading messy proposal text and turning it into structured facts** (step 5 in the diagram), each with an exact quote. Everything else is plain code: loading criteria, text extraction, the injection scan, validation, the pass/fail gate, scoring formulas, ranking and reason text.
I drew the line there because reading free text is where AI is genuinely better than rules, but arithmetic, thresholds and ordering must be exact and repeatable, which code does better. Reasons are built from the score breakdown and quotes, so they cannot invent facts.

Scoring rules used for the sample (all editable by the admin):
- **Experience (30):** qualifying installs of 100 kW+ in 5 years: 12 or more = 100%, 8-11 = 75%, 5-7 = 50%, 1-4 = 25%.
- **Price (25):** 25 x lowest eligible price / bid price.
- **Timeline (15):** 15 x shortest stated weeks / bid weeks. No committed schedule = 0 and flagged.
- **Warranty (15):** half panel years (out of 25), half workmanship years (out of the longest bid).
- **Team (15):** 3.75 each for named lead, defined crew, safety plan, monitoring plan.

Sample result: A 98.0, E 90.2. C would score about 80 and D about 69 (about 84 at best). B fails the licence gate.

## 2. Consistency and explainability
- Extraction runs at temperature 0 with a fixed JSON schema, and the result is cached by file hash + prompt version + model version. Ranking the same files twice reuses it, so **the result is identical**. Scoring is code, so the same facts always give the same score.
- If a proposal is re-extracted (new model or prompt), key fields are compared with the old run. Any difference is flagged for a human instead of silently changing the rank.
- The reviewer sees, per proposal and per criterion: the extracted value, the exact quote with page number, the formula, the points and the final reason. Nothing in the rank is a black box.

## 3. Human in the loop
- **Reviewer checkpoint (step 11):** resolves every flagged proposal, confirms or reinstates each ineligible one (with a reason), can correct a wrongly extracted value, and can override a rank by writing a reason.
- **Final award** is always a human decision. The tool only recommends an order.
- **Logged for audit:** who uploaded what and when, file hashes, criteria version, prompt and model versions, raw and validated extraction, scores, flags and the injection scan result, every human edit or override with name, time and reason, and the final decision.

## 4. Changing criteria
Criteria are versioned. A run stores the version it used. If an admin edits criteria or weights, a **new version** is created and old results stay as they were, labelled "scored under v3, current is v4". The reviewer can re-score with one click. Because extraction is cached, re-scoring is fast and cheap. Only a brand-new criterion that needs a new fact triggers re-extraction of that one field. Weights must total 100 before saving. Once an award is recorded, its results are locked. Changes after that are kept as a new run.

## 5. Risks and guards
| Risk | Safeguard |
|---|---|
| **Wrong extraction** (e.g. misread number, bad OCR) | Every value needs a quote. Code checks the quote exists in the source. Failures retry once, then flag. The reviewer sees quotes next to values. |
| **Made-up facts** | Prompt says "not stated = null, never infer". Missing data is flagged, never guessed (see D's missing schedule). Reasons are template-built from data. |
| **Manipulation / prompt injection** (C contains hidden text asking for rank 1 and full marks) | Proposal text is treated as data, not instructions. A pattern scan and the AI both report such text. The text has no effect on the score. The proposal is flagged for a human. |
| **Loopholes in mandatory rules** (B's subcontractor "to be confirmed") | Prompt separates "held by bidder" from "subcontractor only". Only the first passes. |
| **Bias** (company size, name, region, writing style) | Scores use only the admin's listed facts. Company names are not used in scoring. Reviewers see the full breakdown, so bias is visible. Humans can override. |
| **Price formula unfairness** | Price is relative to the lowest eligible bid, so scores shift if the set changes. This is shown on screen and logged. The admin can switch to a fixed budget-based formula. |
| **Cost** | One extraction call per proposal, cached. Everything else is code. Page and file size limits apply. |
| **Privacy** | Only proposal text is sent to the model. No personal data is needed. Keys are kept in environment variables, never in the repo. |
