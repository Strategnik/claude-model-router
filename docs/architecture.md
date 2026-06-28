# Architecture

`overkill` is intentionally configuration-first. It does not proxy Claude Code, inspect API traffic, or run a background router. It uses the routing mechanisms Claude Code already exposes: model-pinned subagents plus persistent user/project config.

## Constraint

Claude Code runs one main model per session. Hooks cannot switch that model. A config file can set the starting model, and subagents can be pinned to specific models, but there is no native deterministic "route this prompt to this model" hook.

That constraint shapes the design:

- the main session is the router
- the router delegates only when the task justifies spawn overhead
- hard work escalates to a stronger pinned subagent
- high-volume mechanical work can move down to a cheaper pinned subagent

## Tiers

| Tier | Role | Why it exists |
| --- | --- | --- |
| Default | Reads the prompt, does normal work, decides whether to delegate | No extra classifier call or service |
| `quick` | Cheap mechanical execution | Useful only when volume beats spawn overhead |
| `deep-work` | Expensive sealed hard reasoning | Worth it when a subtle wrong answer costs more than the model delta |

## Why Reactive Escalation

Upfront classification looks cleaner, but hard tasks often reveal themselves only after the model starts working. Reactive escalation lets the default model begin, then hand off when it finds ambiguity, deep optimization, subtle bugs, or architectural risk.

## Config Router vs External SDK Router

| Approach | Strength | Cost |
| --- | --- | --- |
| Native Claude Code config | No service, no proxy, easy to inspect | Model choice is model/human mediated |
| External SDK router | Deterministic per-request routing | More code, auth, logs, failure modes |
| Manual `/model` switching | Maximum control | User has to remember every time |

`overkill` stays at the config layer because that is the smallest mechanism that changes default behavior for everyday work.
