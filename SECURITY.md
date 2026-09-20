# Security Policy

## Supported versions

| Version | Supported |
|---|---|
| main | Yes |
| Anything else | No |

## What this skill can reach

The `sharpen-panelist` agent type declares `WebSearch` and `WebFetch` and nothing else.
Tested 20/09/2026 on Claude Code 2.1.198: a panelist asked to read a local file reported
those two tools as its whole set, tried `WebFetch` on a `file://` path, received
`Invalid URL`, and reported no indirect route to disk.

## What reaches it anyway

Claude Code hands every subagent the `CLAUDE.md` files in scope, and any memory index the
harness loads, before the agent acts. The skill cannot switch that off. In the same test a
panelist confirmed receiving four instruction files unprompted.

**Treat your `CLAUDE.md` as visible to every panel seat.** Move anything sensitive out of it
before running Sharpen. Never paste private notes into a panel brief; hard rule 7 in
`SKILL.md` says so and it is the only protection at that point.

## If you skip the agent install

Without `agents/sharpen-panelist.md` in `~/.claude/agents/`, the skill falls back to a
general-purpose agent with full file tools, and says so in one line. If you miss that line,
your panel can read anything on disk. Check the install step in the README.

## Reporting

Open an issue at https://github.com/Caleb-Boddington/sharpen/issues. This is a personal
project with no support commitment and no response-time promise.
