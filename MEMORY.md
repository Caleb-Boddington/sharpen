# Sharpen repo, state

## What this is

The public release of the Sharpen skill. Set up 20/09/2026. Not yet pushed: the repo folder
and its files exist locally, `git init` has not been run and no remote exists.

## Decisions already made, do not re-argue

- **One version, not two.** The skill is generic and the author is just another user. Caleb
  chose this on 20/09/2026 over keeping a personal copy and a public fork, because two copies
  drift and that is the thing he most dislikes.
- **Quorum's layout, not Assay's.** Canonical under `skills/`, generated copy here, sync
  script between them. Assay sits directly in `repos/`, which conflicts with hub rule 12
  (that folder is for repos cloned from elsewhere). Assay is arguably misfiled; it was left
  alone rather than moved.
- **Python sync script, not PowerShell.** `sync-quorum.ps1` cannot run on the Mac.
- **`prompts/` and `runs.md` are never published.** They hold real prompts and a real log.

## Outstanding

- Not pushed. `git init`, first commit, create the GitHub repo, push. Caleb's call, not
  Claude's.
- Test 8, the paired outcome test, has not run. Until it does the README's "unproven" badge
  is honest and must stay.
- Tests 1 to 5 have not passed. Test 1 failed twice on 20/09/2026 for a reason now fixed
  outside this repo, so it needs re-running.
- Windows and Linux clipboard steps are written and untested.

## Where the reasoning lives

`memory/Notes/Projects/Sharpen Skill.md` in the hub holds the full history, including why
step 2a exists and why the offer cannot live in the skill.
