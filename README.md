# Sharpen

A prompt-steering skill for Claude Code. It turns a rough idea into a prompt you have approved, line by line, before any work starts.

![version](https://img.shields.io/badge/version-0.1.1-blueviolet)
![licence](https://img.shields.io/badge/licence-MIT-blue)
![agents](https://img.shields.io/badge/agents-6%E2%80%9311%20per%20run-orange)
![status](https://img.shields.io/badge/status-experimental-yellow)

I use it on my own work. What hasn't been done yet is a side-by-side: the same prompt run through a plain session and through Sharpen, and the finished work compared. Until that runs, treat it as experimental, which is where Quorum and Assay sit too.

## What it does

You type `/sharpen` with a half-formed idea. A panel of subagents reads it: an intent reader, a gap finder, one or two domain experts, and a devil's advocate. They find the holes and, more usefully, the choices an expert in that field would advise against. Wrong platform. Wrong format. A plan three times bigger than the job.

Those come back to you as clickable multiple-choice questions, never as a rewrite. Every change to your prompt carries the ID of the answer that approved it, so you can see exactly why a word moved. A second pair of agents, who never saw the first panel, checks the draft against your original for drift. Then you get the finished prompt and decide whether to run it.

The design principle is narrow: Sharpen steers your prompt, it doesn't replace it. An expert objection reaches you as a choice with your original listed first, never as a silent correction.

## What it exposes you to

Every subagent receives your `CLAUDE.md` files automatically, and there's no way to switch that off. I tested this on 20/09/2026: a panelist reported back that four instruction files had been handed to it before it did anything, including the project `CLAUDE.md` and the memory index.

The helper agent itself is locked down. It has `WebSearch` and `WebFetch` and nothing else. In the same test it tried `WebFetch` on a `file://` path, got `Invalid URL`, and reported no indirect route to disk. So it can't go looking. What reaches it anyway, reaches it through the harness, not through the skill.

If your `CLAUDE.md` contains anything you wouldn't hand a contractor, move it before running this.

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

4. Verify it worked.

```bash
ls ~/.claude/agents/sharpen-panelist.md && echo "agent installed"
```

**What happens if you skip step 2.** The skill falls back to a general-purpose agent, which has full file-reading tools. Your panel can then read anything on disk. It still runs, and it won't tell you, which is exactly why the step is called out here.

## Usage

```
/sharpen <your rough idea>
```

Or say "sharpen this" after a message you've already written, and it will take that message as the raw prompt. Nothing spawns unless one of those phrases fires; the skill will not start itself.

A run goes: it saves your prompt verbatim, numbers its clauses, does its own reconnaissance, checks whether the job actually wants [Crucible](https://github.com/chaseai-yt/crucible) instead, spawns the panel, then asks you up to four questions a round. First question is always the goal in one line, because that becomes the yardstick for everything after it.

At the end you get a change list, a drift report, and three buttons: run it, change something, or save without running. Choose run and the prompt goes to your clipboard with instructions to `/clear` and paste, so the work starts in a clean session.

If the job ends in code and a wrong call would cost a rewrite, a migration or a security hole, the skill stops before spawning anything and points you at Crucible instead.

## What a run costs

One panelist, doing almost nothing, measured 29,257 tokens on 20/09/2026. A full run spawns 6 to 11 agents, so budget in hundreds of thousands of tokens per run and treat that figure as a floor rather than a typical seat. A seat running several web searches hasn't been measured.

`get_usage` is useless here, by the way. Readings either side of a single spawn showed identical whole percentages. The skill logs the subagent's own reported count instead.

## How it works

| Stage | What happens | Agents |
|---|---|---|
| Capture | Your prompt saved verbatim, clauses numbered C1, C2 | 0 |
| Recon | Resolves references, lists your installed skills, one web search per named product | 0 |
| Crucible check | Stops here and defers if the job wants adversarial code review | 0 |
| Panel one | Intent reader, gap finder, domain expert, devil's advocate | up to 5 |
| Merge | Findings become questions only if a wrong guess is costly to undo | 0 |
| Ask | Up to four questions a round, each answer gets an ID | 0 |
| Draft | Your prompt with changes slotted in, each carrying its answer ID | 0 |
| Panel two | Fresh intent checker and prompt engineer, neither saw panel one | 2 |
| Sign-off | Change list, drift report, then you choose | up to 2 recheck |

The cap is 11 spawns including retries. Going past it needs your explicit yes, asked at the moment it would happen.

## Controls

- **No change without an ID.** Every reworded or dropped clause carries the ID of the answer that approved it. A change with no ID is flagged by the drift checker and shown to you in its own words.
- **Advisories are never silent.** When an expert disagrees with your choice, you see both, with yours listed first and neither marked as recommended.
- **The verifier line.** Every finished prompt carries a check the next session can actually run, or one line saying why none exists. Added 20/09/2026, because the template used to state what done looked like and give the next session no way to prove it.
- **Retirement rule.** If the paired test comes back flat, this gets archived rather than quietly maintained.

## Known limitations

- **Every checker is Claude.** The second panel is a fresh reading, not a second opinion from a different model. The skill says so at sign-off and I'd rather it kept saying so.
- **No side-by-side yet.** The paired test, running one real prompt through a plain session and through Sharpen and comparing the finished work, hasn't run. Until it does there's no evidence the agents buy anything a good interview wouldn't.
- **It won't offer itself.** A skill only loads once something has decided to load it, and nothing decides that for a cold idea. If you want Claude to suggest Sharpen when you describe a rough plan, that line has to go in your own `CLAUDE.md`. There's a copy-paste version in [docs/optional-offer.md](docs/optional-offer.md).
- **The Crucible offer has no fixed wording.** The gate offer is pinned to an exact sentence; this one isn't, so two sessions will phrase it differently. Found by a cold read on 20/09/2026, recorded in [docs/testing.md](docs/testing.md).
- **An abandoned run leaves an orphan.** Your prompt is saved at step 1, before the Crucible check can end the run. Nothing cleans that up.
- **No Windows testing.** Written and run on macOS. The clipboard step names `clip` for Windows and `xclip` for Linux, but neither has been exercised.

## Testing

Test results, including two failures and what they revealed, are in [docs/testing.md](docs/testing.md). The paired outcome test is outstanding and is the only one that would justify changing the status badge.

## Credits

A 40-agent [Quorum](https://github.com/Caleb-Boddington/quorum) run on 19/09/2026 rebuilt the first draft and found the thing worth finding: nobody had shown a panel beats one session asking good questions. The retirement rule exists because of that.

[Crucible](https://github.com/chaseai-yt/crucible), by chaseai-yt, does adversarial plan review with a second model actually reviewing. Sharpen defers to it rather than competing with it. It isn't mine and it isn't bundled.

## Licence

MIT. See [LICENSE.md](LICENSE.md).
