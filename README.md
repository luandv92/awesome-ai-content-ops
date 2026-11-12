# awesome-ai-content-ops

A curated map of the tools and patterns behind running an AI content pipeline that
generates, checks, and ships content on a schedule without a human in the loop for
every step — written from actually operating one, not from research.

This isn't a list of AI content *generation* tools (there are plenty of those lists
already). It's the layer almost nobody writes about: what has to exist **around**
generation so an autonomous pipeline doesn't quietly ship garbage, waste budget, or
lie to you about whether it's working.

## The architecture

```
generate → QC gate (fail-closed) → dedup check → schedule/distribute → outcome check → learn
                ↑______________________________________________________________|
                                    (loop feeds back into what to generate next)
```

Every arrow in that diagram is a real failure mode if you skip it:
- No QC gate → you ship broken output (silent audio, black video, half-rendered image)
  because "the generation call succeeded" is not the same claim as "the output is good."
- No dedup check → you re-post the same thing wearing a different filename, which reads
  as spam to both platforms and audiences.
- No outcome check → a scheduled job can exit 0 and do nothing for days before anyone
  notices, because exit code and actual outcome are two different claims.
- No feedback loop → you keep producing by taste instead of by what the data says
  actually retains attention, which is a slow way to run out of budget.

## Tools in this ecosystem

Each of these started as an internal script, painful enough to build that open-sourcing
the generalized version seemed more useful than keeping it private:

| Tool | What it solves |
|---|---|
| [llm-cascade](https://github.com/luandv92/llm-cascade) | Falls back through cheaper LLM tiers automatically instead of hard-failing when a paid API call fails. |
| [cc-worktree-ops](https://github.com/luandv92/cc-worktree-ops) | Git worktree helpers for running multiple coding-agent sessions in parallel without them stepping on each other. |
| [cc-cost-kit](https://github.com/luandv92/cc-cost-kit) | Config that stops a coding subagent from silently inheriting the most expensive model when none is declared. |
| [outcome-watchdog](https://github.com/luandv92/outcome-watchdog) | Catches scheduled jobs that exit 0 but produced no actual change — the "your cron is lying to you" problem. |
| [media-gate](https://github.com/luandv92/media-gate) | Fail-closed QC: blocks black video, silent audio, and near-duplicate images before they ship. |
| [honest-timer](https://github.com/luandv92/honest-timer) | A scarcity countdown that doesn't reset on reload and actually raises the price when it hits zero. |
| [stripe-hardening-kit](https://github.com/luandv92/stripe-hardening-kit) | Pre-deploy gate that blocks a banned/deprecated payment integration from leaking back into shipped code, plus a fail-closed webhook reference. |
| [content-ops-automation-templates](https://github.com/luandv92/content-ops-automation-templates) | Lead capture → email drip → UTM attribution, as small reusable Python templates. |
| [video-prompt-formulas](https://github.com/luandv92/video-prompt-formulas) | Structural prompt formulas for AI video generation (hook timing, loopable shots, character-consistency anchors). |

## Runnable agent workflow examples

Start with the [agent workflow guide](docs/agent-workflows.md): a bounded
planner/worker/reviewer example in `llm-cascade`, then an accepted-result
monitor in `outcome-watchdog`. Both have offline fixture demos and tests.
The guide explains the contracts between routing, review and outcome
monitoring, and identifies the missing pieces before production deployment.
It is an integration path through existing tools, not a new framework or
a claim that a distributed agent network is already deployed.

## Patterns worth stealing, even if you don't use any of the tools above

- **Fail-closed, not fail-open.** A QC/deploy gate that can't verify something should
  block, not assume it's fine. The failure mode of a gate that fails open is silent —
  you find out from a customer, not from the gate.
- **Graduated verdicts, never binary.** "Broken" vs. "working" hides the most useful
  state: *couldn't verify*. A tool being unreachable is not proof the thing it monitors
  is actually broken — conflating the two produces false alarms that train people to
  ignore the alarm.
- **The source of truth for a limit is runtime state, not a comment in a doc.** A cap,
  a threshold, a cadence — write it into code/config that's read live. Docs drift from
  reality; a value read at runtime can't.
- **Every scheduled/published unit of work should be traceable to why it happened.**
  If you can't answer "why did this run today, with this content, at this time" by
  reading a config file, the pipeline is running on vibes, not on a system.

## Contributing

If you're solving one of these problems and think a tool belongs in the list, open an
issue or a PR — genuinely interested in what else is out there.

## License

MIT — see [LICENSE](LICENSE).
