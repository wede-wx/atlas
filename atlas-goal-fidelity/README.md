# Atlas Goal Fidelity

Reduce Goal Dilution in AI Agents.

Atlas Goal Fidelity is a lightweight skill for keeping the user's understanding and the agent's understanding aligned during execution.

It does not try to discover a user's hidden life goal. It extracts the parts of a request that the agent must not move on its own authority:

- what must be done
- what must not be done
- what existing behavior, layout, scope, or API must be preserved
- what assumptions the agent is relying on

## Core Principle

Agent can fail. Agent can challenge the goal. Agent cannot silently rewrite it.

## Minimal Contract

```json
{
  "goal": "",
  "must_do": [],
  "must_not_do": [],
  "preserve": [],
  "assumptions": []
}
```

Each contract item carries:

- `text`
- `hard`
- `source_quote`
- `verify_by`

Each assumption carries:

- `text`
- `risk`
- `verify_by`

## What It Guards Against

- Goal Drift: "implement this feature" becomes "hide the feature"
- Requirement Downgrade: "complete implementation" becomes "frontend-only mock"
- Reference Drift: "match this UI" becomes "same vibe, different layout"
- Preserve Breakage: "refactor this" breaks existing behavior

## First Version Scope

v0.1 focuses on:

- constraint extraction
- compact Goal Contract generation
- default anti-drift rules
- mandatory deviation disclosure
- final contract audit
- alignment snapshots and phase checks

It intentionally does not include:

- psychological goal discovery
- long-term user memory
- cross-model judging
- CI hooks
- benchmark automation

## Files

- `SKILL.md`: the agent-facing workflow
- `goal-contract.schema.json`: the compact contract schema
- `examples/`: three concrete drift cases and how Atlas should handle them
