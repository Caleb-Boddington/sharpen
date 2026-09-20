# Changelog

Dates are DD/MM/YYYY.

## 0.1.0, 20/09/2026

First public release. Unproven: the paired test has not run.

**Added**
- Step 2a, the Crucible check, before any agent is spawned. It previously lived only in a
  section after the run and after the hard rules, with nothing pointing at it, so a session
  working through the steps in order reached the panel without the question ever arising.
- `Checkpoints:` and `How to check:` in the finished-prompt template, with a question in
  step 5 that sources them and hard rule 10 enforcing them. The template used to state what
  done looked like and give the next session no way to prove it, so the skill verified the
  prompt and never the work.
- A stated install step for `agents/sharpen-panelist.md`. Skipping it silently downgrades
  every seat to an agent that can read your files.

**Changed**
- The skill no longer names one person. It reads as second person throughout.
- Cost guidance now uses the subagent's own reported token count. `get_usage` cannot see a
  single spawn: readings either side of one showed identical whole percentages.

**Layout**
- Repo root matches Assay: five files and two folders, plus `agents/` because the install
  step copies from it. Working notes and run logs stay in the author's own working copy.
- Matches the house layout used by Quorum and Assay: `references/` for supporting material,
  `docs/` for run records and testing, and README sections in the same order.
  `panel-briefs.md` moved to `references/panel-briefs.md`.

**Known limits**
- Test 8, the paired outcome test, has not run.
- The skill cannot offer itself. See `docs/optional-offer.md`.
- Windows and Linux clipboard steps are written but untested.
