---
name: sharpen
description: Turns your rough idea or raw prompt into a polished prompt you approve before any work starts. A full panel of specialist subagents finds gaps and flags choices an expert would advise against (wrong platform, wrong format, an overbuilt plan). The main session puts them to you as clickable questions, slots the changes into your own wording, has the draft checked again, and shows what changed for sign-off. Run it only when the user types /sharpen, says "sharpen this", "sharpen this prompt" or "clean this prompt up", or says yes to an offer. When the user gives a vague idea for a substantial piece of work, do not run it: offer it in one line and name the cost, 6 to 11 subagents. Any task type. For a coding build where a wrong call costs a rewrite, a migration, a security hole or user trust, and a Codex review is wanted, offer /crucible instead.
---

# Sharpen

**Gate.** Run only if, in this conversation, the user typed /sharpen, or said "sharpen this",
"sharpen this prompt" or "clean this prompt up", or said yes to an offer of Sharpen.
Otherwise stop here, spawn nothing, and offer it in one line:

> I can run /sharpen on this first: 6 to 11 helper agents, cost per run not yet measured. Want it?

Offer once per idea. If the user ignores it, drop it.

---

Sharpen **steers** the user's prompt. It keeps their words, fills the gaps that matter, and
puts every expert objection to them as a choice. They decide every change and approve the
result before anything runs. The design came out of a 40-agent Quorum run on 19/09/2026. Every rule below was argued out
there; where a rule looks fussy, it is answering a failure that run found.

Settled calls, not to be re-argued: full panel; any task type; the user approves before work
starts; steer, never rewrite; an advisory always reaches them as a choice; Crucible stays
separate.

## What lives in this folder

Everything Sharpen uses or makes stays in this folder, so the skill is one portable unit.

| | |
|---|---|
| `SKILL.md` | This file. |
| `references/panel-briefs.md` | Every seat's brief and one example reply. Passed to helpers word for word. |
| `agents/sharpen-panelist.md` | The helper type: web search and web fetch only, no local file tools. **Must be copied or linked into `~/.claude/agents/`, or every seat silently falls back to a helper that can read your files.** See the README. |
| `prompts/` | Every raw prompt and finished prompt, dated. **Create it if it is missing**; a fresh clone does not ship it. |
| `runs.md` | One line per run, for the retirement check. **Create it with a header row if it is missing**; a fresh clone does not ship it. |

## Limits, stated honestly

- **Agents:** 6 to 11 per run. The cap is 11 spawns, retries and Change included: panel one
  up to 5, panel two 2, one recheck up to 2, a retry reserve of 2. The cap is only exceeded
  with the user's explicit yes, asked at the moment it would be.
- **Cost:** one Opus helper with no tools recorded roughly 88,000 tokens on 19/09/2026. A
  second measurement on 20/09/2026, the test 7 panelist, came to 29,257 tokens for one tool
  use; treat that as the floor for a seat that barely works, not a typical seat, and a seat
  running several web searches is still unmeasured. `get_usage` is too coarse to see a single
  helper: both readings around that spawn showed the same whole percentages. Use the
  subagent's own reported token count, log it, and never quote a share of their plan.
- **Privacy:** the `sharpen-panelist` type has no file-reading tools. Tested 20/09/2026 and
  it holds: the helper reported `WebSearch` and `WebFetch` as its whole toolset, tried
  `WebFetch` on a `file://` path, got `Invalid URL`, and found no indirect route. The same
  test confirmed the known leak, so it is now measured rather than assumed: the helper is
  handed the hub `CLAUDE.md`, the global `CLAUDE.md` and both copies of `memory/MEMORY.md`
  before it does anything. That cannot be switched off on Claude Code 2.1.198. The index
  being deliberately opaque is what keeps this safe, so never "improve" a line in it into a
  summary, and never paste anything from your private notes into a brief.
- **Independence:** every panelist is Claude. The second check is a fresh reading, not a
  second opinion from a different model. Say so at sign-off.

## The run

Keep a progress file at `<session scratchpad>/sharpen-progress.md`. After each step, write
the step number, the raw prompt path, the answers so far with their IDs, and spawns used.
If the conversation has been compacted, read it and carry on from where it says.

### 1. Capture

1. Run `date`.
2. The raw prompt is the text after /sharpen, or the user's previous message if that is empty.
   Save it **verbatim** to `prompts/YYYY-MM-DD-<slug>-raw.md`. The slug is a few words from
   the prompt, lowercase letters, digits and hyphens only, 40 characters at most.
3. Number its clauses C1, C2 and so on. Every later change points back to these.

### 2. Recon (main session, no agents)

1. Resolve anything the prompt points at ("the prompt above", "that file").
2. List what the user already has: `ls ~/.claude/skills`, and `plugins/CONNECTORS.md` if present.
3. One web search for every platform, product or price the prompt names.
4. Pick the domain expert role, or two if the task genuinely spans two fields.
5. **Off-ramp.** If the prompt already states the goal, who it is for, the format and what
   done looks like, ask: "Your prompt already covers the basics. Run it as it is, or run the
   panel anyway?" Running it as it is ends Sharpen with no agents spawned. The panel is never
   run with seats missing.

### 2a. Crucible check

Before spawning anything: does the work end in code, and would a wrong call cost a rewrite,
a migration, a security hole or user trust? If yes, stop and offer `/crucible` in one line.
Only carry on if they decline it.

This runs whatever they chose at the off-ramp, because Crucible is about the kind of work, not
the quality of the prompt. A well-specified migration still wants a Codex review. Where both
Sharpen and Crucible could fit, offer Crucible.

Added 20/09/2026. Until then the routing lived only in the "Sharpen or Crucible" section
below, after the run and after the hard rules, with nothing in steps 1 to 11 pointing at it.
A session working through the run in order reached the panel without the question ever
arising, so test 5 could only pass by luck.

### 3. Panel one (parallel, one message)

Spawn every seat in a single message, `subagent_type: "sharpen-panelist"`,
`model: "opus"`. If that type is not available (it only loads when a session starts), use
`general-purpose` with `model: "opus"` and say in one line that the file-reading limit is off
for this run.

Seats: **intent reader, gap finder, domain expert (one or two), devil's advocate.** Never
fewer, whatever the task size. Each gets its brief from `references/panel-briefs.md` word for word,
plus only the inputs listed under it there.

**Failure path.** A reply in the wrong format: re-ask once with SendMessage. An empty reply
or an error: respawn once from the retry reserve. If a seat still fails, ask: "The
[seat] failed twice. Carry on without it, or try once more (one more agent)?" Never write a
missing seat's findings yourself.

### 4. Merge

1. Answer any question the recon files already settle. Record it as an assumption with its
   source.
2. A finding earns a question only if a wrong guess is costly to undo once the work is acted
   on (sent, submitted, paid for, published, built on), or it is a fact only the user knows, or
   it is an advisory.
3. Tone, length and layout become assumptions, not questions. So does any question whose
   "If we guess wrong" line is weak.
4. Drop any finding that cites neither a clause nor a named fact.
5. Merge duplicates. Where two seats disagree, keep both sides in one question.

### 5. Ask

`AskUserQuestion`, four questions a round.

**Round 1, in this order:**
1. The goal, in one line: "Is this what you're after?" with the line as the question and
   options "Yes" and "Not quite". This line is the yardstick for the rest of the run.
2. The assumptions: one question, "I've assumed these. All correct?" with the list in the
   question, and options "All correct" and "I'll fix one".
3. and 4. Advisories first, then the highest-stakes questions.

**Wording.** Each question under 20 words, header 12 characters or fewer, each option with
one line saying what it means and what it rules out. Define any unavoidable jargon in
brackets.

**Advisories.** Name the expert and give the reason in one sentence. Put the expert's
recommendation in the question text, in the expert's name. List the user's original choice
first. Neither option carries "(Recommended)". Every advisory is asked, adding a round if it
has to. Ordinary questions may carry "(Recommended)" on the suggested answer.

**How many.** After 8 non-advisory questions, stop and ask: "N more questions. Answer them,
or keep my guesses?" Nothing is dropped without their choice. From round 2, the last slot may
be "Take the experts' advice on the rest"; it never covers an advisory.

**The check.** One question covers how the finished work gets proved, whenever the work
produces something checkable: a command that runs, a file that must exist, a figure that must
match a source, a reader who must sign it off. Ask who runs the check, the user or the session.
Where nothing is checkable, say so in one line at sign-off and leave the `How to check` line
reading "no automatic check; the user reads it". Never invent a test.

Give every answer an ID (Q1, Q2 and so on) and note whether they took the suggested answer.

### 6. Draft

The draft is **the user's prompt with changes slotted in**, not a new document.

1. Go clause by clause: mark each KEPT, REWORDED or DROPPED. Every REWORDED or DROPPED
   clause, and every addition, carries the answer ID that approved it. No ID, no change.
2. Add a section only where an answer justifies one.
3. The finished prompt carries, in this order:

```
<role>One line: who should do this work.</role>
<original_request>Their raw prompt, verbatim.</original_request>
<goal>The line they confirmed in round 1.</goal>
[Their prompt with the approved changes, in their words where possible]
Decisions already made (settled, do not reopen): each with its reason and answer ID.
Ask me only about something none of these covers.
Done means: [from their answers]
Checkpoints: [the work in stages; stop and report at each]
How to check: [the test, command, data or second reading that proves each stage is met, and who runs it]
Output and where it goes: [from their answers]
```

Dates as DD/MM/YYYY, British English, no em or en dashes.

### 7. Panel two (parallel, one message)

**Fresh intent checker** (raw prompt, their answers with IDs, the draft; nothing from panel
one) and **prompt engineer** (the draft only). Briefs in `references/panel-briefs.md`.

Fix what they find. If a fix needs the user, ask that point only. **One recheck loop**, by the
seats that raised points, at most 2 agents. Then stop, and list anything still open at
sign-off. Do not fake agreement.

### 8. Sign-off

Show, in this order:
1. "N changes." The biggest five first, each tagged with its answer ID or "expert: X". The
   full list if they ask.
2. The intent checker's DRIFT list, exactly as written.
3. Anything still open, and one line: "Every checker was Claude."
4. The finished prompt in a code block.

Then `AskUserQuestion` with **Run it**, **Change something**, **Save, don't run**. None
labelled Recommended.

### 9. Change

Redraft, then respawn the fresh intent checker only. If the goal or a decision changed, run
panel two again. Every Change draws on the same cap of 11. Once the cap is reached, ask
before spawning more; if they say no, show the redraft marked "not rechecked".

### 10. Run or save

- **Run it:** save to `prompts/YYYY-MM-DD-<slug>.md`, copy it to the clipboard (`pbcopy` on
  macOS, `clip` on Windows, `xclip -selection clipboard` on Linux), and tell them: "Type /clear, then paste, then Enter." The
  settled-decisions block stops the new session asking them the same questions again.
- **Save, don't run:** save the same file, no clipboard, and give the full path.

### 11. Log

Run `get_usage` and append one line to `runs.md`: date, slug, spawns, usage change, whether
they clicked Run, how many suggested answers they accepted out of how many. If they accept
nearly every suggestion, say so at the next run: the suggestions may be steering them.

## Hard rules

1. Nothing is spawned without the gate passing.
2. The panel is never run with seats missing. The only way to skip it is their choice at the
   off-ramp.
3. Every advisory reaches them as a choice, with their original option listed first.
4. Nothing runs until they click Run it.
5. Every change to their prompt carries the ID of the answer that approved it.
6. Never more than 11 spawns without their yes at that moment.
7. Nothing from the user's private notes goes into a brief.
8. Run `date` before writing a date. Search the web before stating a price.
9. No PLAN.md, no Codex, no review logs. Those are Crucible's.
10. Every finished prompt carries a check the running session can perform, or one line saying
    why none exists. Added 20/09/2026: the template stated what done looked
    like and gave the next session no way to prove it, so Sharpen only ever verified the
    prompt, never the work.

## Sharpen or Crucible

The decision is made at step 2a, before anything is spawned. This section is the reasoning
behind it, not a second place to make it.

Offer Crucible instead when the work ends in code, a wrong call costs a rewrite, a migration,
a security hole or user trust, and a Codex review is wanted. Where both could fit, offer
Crucible. Crucible is unchanged by this skill.

Crucible is somebody else's skill, at https://github.com/chaseai-yt/crucible. If it is not
installed, say so in the same line rather than offering something the user does not have.

## Testing and retirement

**Status on 20/09/2026: built, test 7 passed, tests 1 to 5 and 8 still unrun.** Before
relying on it, in a fresh session each:

1. A rough idea the hub has never heard of, typed without /sharpen: expect a one-line offer
   and no agents. **Do not reuse a prompt the project already documents.** Tried 20/09/2026 and it failed:
   the session recognised the prompt, quoted
   its saved copy in `prompts/` back at the user and explained that Sharpen
   already exists, instead of offering. No agents spawned, so the gate half passed. That
   prompt is now spent as a test case, because the hub documents it as the prompt that built
   the skill and any session will recognise it. Saving it into `prompts/` that morning is
   what made it findable. Current test case, unconnected to anything in the hub:

   > i want to make a little site that tracks which of my mates owes me money after we go out

   When this one is used up, write a new one the same way: a rough idea, no detail, nothing
   the hub has a note about.
2. "sharpen this" typed on its own after the test 1 prompt: expect it to run.
3. /sharpen on any saved raw prompt: expect 8 or fewer non-advisory questions and a
   sign-off. Still valid, because here the gate is fired deliberately and recognition does
   not matter.
4. "/sharpen sell my booking app from an Artifact": expect the Artifact advisory as a
   question, the user's choice listed first.
5. "/sharpen add Stripe payments and a users table": expect an offer of /crucible at step
   2a, before any agent is spawned.
6. Description length, under Anthropic's 1,024-character cap:
   `sed -n '3p' SKILL.md | sed 's/^description: //' | tr -d '\n' | wc -m`. It was 848 on
   19/09/2026.
7. One `sharpen-panelist` asked to read a local file: expect it to have no tool
   that can. Read `get_usage` before and after. **Passed 20/09/2026**, logged in `runs.md`.
   Re-run it after any change to `agents/sharpen-panelist.md` or after a Claude Code upgrade,
   because the result is a fact about that version, not a permanent property.
8. **The paired test.** Run one real raw prompt twice, once in a plain session and once
   through Sharpen, and compare the finished work, not the prompts. This is the only test
   that decides whether Sharpen is worth its cost.

**Retirement.** Propose retiring Sharpen if test 8 shows no better result, or if the user did
not use the result in 3 of their first 5 logged runs. That threshold is a judgement, not
evidence; change it if the user disagrees.
