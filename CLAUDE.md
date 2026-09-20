# Sharpen, repo rules

This folder is the **public git repo** for the Sharpen skill. It is pushed to
github.com/Caleb-Boddington/sharpen.

## Never hand-edit these

`SKILL.md`, `panel-briefs.md` and `agents/sharpen-panelist.md` are **generated**. The
canonical copies live at `skills/personal/sharpen/` in the hub, because `~/.claude/skills/`
symlinks there and Google Drive does not sync symlinks (hub rule 6).

Edit the canonical file, then run:

```bash
python3 scripts/sync-sharpen.py
```

`--check` reports drift without copying. Editing the generated copy works right up until the
next sync, which throws the edit away silently. This is the same arrangement as Quorum, and
it exists because those two copies drifted once already.

## Never copy these in

`prompts/` and `runs.md` hold real prompts and a real run log. They are personal and the sync
script refuses them by name. The repo carries empty starters so a fresh clone works.

## Everything else is canonical here

`README.md`, `CHANGELOG.md`, `SECURITY.md`, `LICENSE.md` and `docs/` are written by hand in
this folder and have no copy anywhere else.

## House style

The skill names nobody. It reads as second person, and any pronoun for the user is they or
them. If a change reintroduces a name, a hub path or a machine-specific path, it is a bug.

Four files must never name anyone: `SKILL.md`, `panel-briefs.md`, `README.md` and anything
under `agents/` or `docs/`. `LICENSE.md` carries the copyright line and `MEMORY.md` is an
author's note, so both are exempt. Check before every push:

```bash
cd ~/ClaudeHub/projects/sharpen && grep -rniE "caleb|claudehub|/users/|G:\\\\" SKILL.md panel-briefs.md README.md agents docs | grep -v Caleb-Boddington; echo "exit: clean if nothing above"
```

The `cd` is not optional. Run it from the wrong folder and grep prints "No such file or
directory" for every target, which is not the same thing as finding nothing, and the trailing
echo is there so a clean pass looks different from a broken one.

British English. No em or en dashes. Dates DD/MM/YYYY.

## Before pushing

1. Run the sync, then read the diff.
2. Run the grep above. It should return nothing but the GitHub URLs.
3. Update `CHANGELOG.md`. Every release says what changed and why.
4. Do not claim the skill is proven. Test 8 has not run.
