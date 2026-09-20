# Sharpen

A prompt-steering skill for Claude Code. It turns a rough idea into a prompt you have approved, line by line, before any work starts.

![version](https://img.shields.io/badge/version-0.1.0-blueviolet)
![licence](https://img.shields.io/badge/licence-MIT-blue)
![agents](https://img.shields.io/badge/agents-6%E2%80%9311%20per%20run-orange)
![status](https://img.shields.io/badge/status-unproven-red)

Read the badge. Nobody has shown this beats simply asking Claude to interview you, and the test that would settle it has not been run. I'd rather say that at the top than bury it.

## What it does

You type `/sharpen` with a half-formed idea. A panel of subagents reads it: an intent reader, a gap finder, one or two domain experts, and a devil's advocate. They find the holes and, more usefully, the choices an expert in that field would advise against. Wrong platform. Wrong format. A plan three times bigger than the job.

Those come back to you as clickable multiple-choice questions, never as a rewrite. Every change to your prompt carries the ID of the answer that approved it, so you can see exactly why a word moved. A second pair of agents, who never saw the first panel, checks the draft against your original for drift. Then you get the finished prompt and decide whether to run it.

The design principle is narrow: Sharpen steers your prompt, it doesn't replace it. An expert objection reaches you as a choice with your original listed first, never as a silent correction.

## Install

Prerequisites: Claude Code, and a plan that can spawn subagents. Tested on 2.1.198 on macOS.

1. Clone the skill.

```bash
git clone https://github.com/Caleb-Boddington/sharpen.git ~/.claude/skills/sharpen
```

2. Install the helper agent. This step is not optional and there's no warning if you skip it.

```bash
mkdir -p ~/.claude/agents && cp ~/.claude/skills/sharpen/agents/sharpen-panelist.md ~/.claude/agents/
```

3. Restart Claude Code. Agent types only load at session start.

4. Verify it worked. Start a new session and run:

```bash
ls ~/.claude/agents/sharpen-panelist.md && echo "agent installed"
```

Then type `/sharpen` followed by any rough idea. If the panel spawns and the first question asks you to confirm the goal in one line, it's working.

**What happens if you skip step 2.** The skill falls back to a general-purpose agent, which has full file-reading tools. Your panel can then read anything on disk. It still runs, and it won't tell you, which is exactly why the step is called out here.

## The thing I'd want to know before installing

Every subagent receives your `CLAUDE.md` files automatically, and there's no way to switch that off. I tested this on 20/09/2026: a panelist reported back that four instruction files had been handed to it before it did anything, including the project `CLAUDE.md` and the memory index.

The helper agent itself is locked down. It has `WebSearch` and `WebFetch` and nothing else. In the same test it tried `WebFetch` on a `file://` path, got `Invalid URL`, and reported no indirect route to disk. So it can't go looking. What reaches it anyway, reaches it through the harness, not through the skill.

If your `CLAUDE.md` contains anything you wouldn't hand a contractor, move it before running this.

## Cost

One panelist, doing almost nothing, measured 29,257 tokens on 20/09/2026. A full run spawns 6 to 11 agents, so budget in hundreds of thousands of tokens per run and treat that figure as a floor rather than a typical seat. A seat running several web searches hasn't been measured.

`get_usage` is useless here, by the way. Readings either side of a single spawn showed identical whole percentages. The skill logs the subagent's own reported count instead.

## Limits

- **Every checker is Claude.** The second panel is a fresh reading, not a second opinion from a different model. The skill says so at sign-off and I'd rather it kept saying so.
- **It's unproven.** The paired test, running one real prompt through a plain session and through Sharpen and comparing the finished work, hasn't run. Until it does there's no evidence the agents buy anything a good interview wouldn't.
- **It won't offer itself.** A skill only loads once something has decided to load it, and nothing decides that for a cold idea. If you want Claude to suggest Sharpen when you describe a rough plan, that line has to go in your own `CLAUDE.md`. There's a copy-paste version in [docs/optional-offer.md](docs/optional-offer.md).
- **No Windows testing.** Written and run on macOS. The clipboard step names `clip` for Windows and `xclip` for Linux, but neither has been exercised.

## Where it came from

A 40-agent Quorum run on 19/09/2026 rebuilt the first draft and found the thing worth finding: nobody had shown a panel beats one session asking good questions. The retirement rule in `SKILL.md` exists because of that, and it's genuine. If the paired test comes back flat, this repo gets archived rather than quietly maintained.

## Related

[Crucible](https://github.com/chaseai-yt/crucible), by chaseai-yt, does adversarial plan review with a second model actually reviewing. Sharpen defers to it for coding work where a wrong call costs a rewrite or a security hole. It isn't mine and it isn't bundled.

## Licence

MIT. See [LICENSE.md](LICENSE.md).
