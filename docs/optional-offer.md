# Making Claude offer Sharpen on its own

Sharpen cannot suggest itself. A skill loads only once something has decided to load it, and
nothing decides that when you describe a rough idea in a fresh session. Tested twice on
20/09/2026: given a vague plan for a small web app, the session built and published the
whole thing without ever mentioning the skill.

The instruction has to live somewhere that loads every session regardless. That means your
own `CLAUDE.md`, not this repo.

Paste this into `~/.claude/CLAUDE.md`:

```markdown
- A rough idea for a substantial piece of work gets one line offering `/sharpen` before
  anything is built. Name the cost: 6 to 11 subagents. Offer once, then drop it if I ignore
  it. Substantial means a site, an app, a long document, anything I would be annoyed to see
  rebuilt.
```

Verify it by starting a new session and describing something vague you would like built. You
should get one line offering the skill, and nothing else should happen until you answer.
