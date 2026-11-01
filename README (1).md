# MATACS-TH-2026-02: AI Proposal Triage Design

- **Name:** M. Arsalan
- **Reference:** MATACS-TH-2026-02
- **Time actually spent:** 3 hours 30 minutes

## Files
- [WRITEUP.md](WRITEUP.md): answers to the five questions
- [diagram/process-flow.png](diagram/process-flow.png): process flow from upload to ranked output
- [PROMPTS.md](PROMPTS.md): the extraction prompt, exactly as sent
- [ranking.json](ranking.json): expected ranking of proposals A-E

## How I used AI tools
I used Claude to help me read the brief and draft the process flow, extraction prompt, scoring rules and write-up, and I rechecked the scores for A-E by hand. I corrected the AI on Proposal C: it counted all of C's installs even though some start in 2020, which may be outside the 5-year window in the criteria. I changed this so that installs without a clear date inside the last 5 years are marked "uncertain" for the reviewer instead of being counted automatically.
