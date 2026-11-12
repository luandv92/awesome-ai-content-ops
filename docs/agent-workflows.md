# Agent workflows with observable outcomes

This is a map of runnable examples in the existing Python ecosystem. It does
not introduce a new framework or claim a deployed agent network.

## Start with a bounded workflow

The [llm-cascade agent example](https://github.com/luandv92/llm-cascade/blob/main/examples/agent_workflow.py)
uses planner, worker and reviewer roles to produce a software design. Each
handoff has a JSON contract. The workflow makes at most three router calls,
records the model and reported cost, rejects malformed outputs and stops
on a rejected review. It neither executes generated code nor publishes it.

Run `python examples/agent_workflow.py --demo` from that checkout after
`pip install -e .`. Fixtures are deterministic and offline. Live mode is an
explicit opt-in; provider fallback and model choices are configured by the
router. A model approving another model's output is not independent proof
of correctness. Add deterministic application checks and human review
before any consequential action.

## Monitor accepted outcomes separately

The [outcome-watchdog example](https://github.com/luandv92/outcome-watchdog/blob/master/examples/agent_outcomes.py)
checks an accepted-result ledger rather than counting model attempts. It
demonstrates `OK`, `NO_CHANGE` and `UNVERIFIED` with local fixtures. Run
`python examples/agent_outcomes.py` from its checkout after `pip install -e .`.

```text
goal -> plan -> candidate -> review -> application checks -> accepted ledger
          \________ model trace ________/                       |
                                                    outcome-watchdog
```

The examples are separate, composable demonstrations: the router example
prints a result and does not automatically write the watchdog ledger. The
application owns the acceptance policy. Write `accepted_at` only after the
artifact exists and the required checks pass. Put failures, rejected reviews
and retries in the attempt trace instead. Deduplicate accepted run IDs in
the producer so a replay cannot masquerade as another delivered result.

## Engineering decisions before scaling

| Concern | Existing tool or next concrete step |
|---|---|
| Provider failure and task routing | `llm-cascade`; configure tiers and review fallback quality |
| Silent jobs with no accepted outputs | `outcome-watchdog`; define freshness and the ledger contract |
| Tests touching live files | `pytest-sideeffects`; audit the working tree around tests |
| Required checks removed from a caller | `wiringguard`; assert the call-site invariant |
| Parallel coding sessions | `cc-worktree-ops`; isolate branches and review merge behavior |
| Execution and external actions | Add explicit tool allowlists, isolation and authorization in the application |
| Restart and concurrency | Add durable run IDs, atomic writes and claim/retry semantics before scheduling |
| Model quality | Keep a task dataset and score outputs independently; the fixture demo is not an eval benchmark |

Do not add concurrent agents, a queue service or another framework until a
measured workload needs them. A useful next project is an end-to-end API
prototype with a small published evaluation dataset, tests for rejected and
malformed outputs, and documented restart behavior. These are future
integration steps, not capabilities these examples already provide.
