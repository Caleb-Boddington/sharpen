# Testing

Dates are DD/MM/YYYY. The test list itself is at the bottom of `SKILL.md`.

Two of these are failures. They are here because the failures were more useful than the
passes, and a testing page that only records successes is not worth reading.

## Summary

| Test | What it checks | Result |
|---|---|---|
| 1 | A rough idea, no `/sharpen`: expect an offer, no agents | **Failed twice**, 20/09/2026. Fixed outside the skill. |
| 2 | "sharpen this" on its own triggers a run | Not run |
| 3 | A full run stays inside 8 non-advisory questions | Not run |
| 4 | An expert advisory reaches the user as a choice | Not run |
| 5 | A coding job defers to Crucible before spawning | **Passed**, 20/09/2026, cold read |
| 6 | Description inside Anthropic's 1,024-character cap | **Passed**. 853 characters. |
| 7 | The panelist has no file-reading tools | **Passed**, 20/09/2026 |
| 8 | Paired outcome test against a plain session | **Not run. The status badge depends on this one.** |

## Test 7, the panelist cannot read your files

Passed. A panelist was asked to list its tools and then to read a local file. It reported
`WebSearch` and `WebFetch` as its entire set, called `WebFetch` on a `file://` path, received
`Invalid URL`, and reported no indirect route to disk.

The same test confirmed what the skill already warned about: four instruction files were in
its context before it acted, handed to it by the harness. The tool restriction works. It is
not what protects your instruction files, and nothing in this skill can.

Cost: 29,257 subagent tokens for one tool use. `get_usage` showed identical whole percentages
either side of the spawn, so it cannot see a single helper.

## Test 1, failed twice, and why it is not fixable inside the skill

First attempt used the prompt that originally produced this skill. The session recognised it,
quoted its own saved copy back, and explained that Sharpen already existed. No agents spawned,
so the gate held, but no offer was made.

That failure was self-inflicted: the prompt had been saved into `prompts/` that same morning,
which made it findable. The lesson generalises. **Filing something can destroy its value as a
test input.**

Second attempt used a throwaway idea the project had never heard of: a small site for tracking
who owes whom after a night out. The session built and published the whole thing without
mentioning Sharpen once.

The cause is structural. A skill loads only once something has decided to load it, and nothing
decides that when you describe a rough idea in a fresh session. The instruction to offer was
sitting in the one file that cannot be read at the moment it is needed. It has to live in your
own `CLAUDE.md` instead; see [optional-offer.md](optional-offer.md).

## Test 5, passed on a cold read

Run by handing `SKILL.md` to an agent with no knowledge of the project and asking what it
would do with `/sharpen add Stripe payments and a users table`.

It stopped at step 2a, spawned nothing, and offered Crucible. Its own words on whether the
instruction was easy to find: "Reading top to bottom, I hit it before any spawn decision. That
fix works." That matters, because before 20/09/2026 the routing lived only in a section after
the run and after the hard rules, where a session working in order would sail past it.

It also raised eight problems. The ones that survived review and are now in the issue list:

1. **No fixed wording for the Crucible offer.** The gate offer is pinned to an exact sentence.
   This one says only "offer `/crucible` in one line", so the single output this test checks is
   unspecified and two sessions will phrase it differently.
2. **A two-part test in one place, three-part in another.** Step 2a asks whether the work ends
   in code and whether a wrong call is costly. The explanatory section and the description both
   add "and a Codex review is wanted", which cannot be known before asking.
3. **Recon is spent before the offer that may end the run.** An `ls`, a file read and a web
   search all happen before step 2a asks a question that can terminate everything.
4. **An abandoned run leaves an orphan.** The raw prompt is saved at step 1; a run ending at 2a
   never reaches the logging step.
5. **No way to check whether Crucible is installed**, although a rule depends on it.
6. **Two sequential questions where one round would do**, on a prompt that trips both the
   off-ramp and the Crucible check.

The cold reader also admitted it had read the test list at the bottom of the file, so it knew
the expected answer. Its words: "I cannot claim a clean blind pass." Taken as a pass on the
behaviour and on findability, not as a blind trial.

## Test 8, and what "proven" would require

Run one real prompt twice. Once in a plain session that simply asks good questions, once
through Sharpen. Compare the finished work, not the prompts.

Tests 1 to 7 check the machinery: whether the gate fires, whether the panel spawns, whether the
agent is locked down. Passing all of them says the machine runs. It says nothing about whether
the output is better, which is the only question that justifies the token cost. Until test 8
runs, the status badge stays red.
