# Atlas

Prompt skills that keep a capable AI agent **honest about the goal** — so a strong model stays accountable to what you actually asked for, instead of quietly trading it for an easier or "more sensible" version along the way.

[中文说明 → README.zh-CN.md](./README.zh-CN.md) · [Examples → EXAMPLES.md](./EXAMPLES.md)

---

## Who this is for

Atlas is **not** a crutch for weak models, and it is **not** about code quality. A strong model with a decent `CLAUDE.md` already writes good code — and models keep getting better, so that floor only rises.

What a stronger model does *not* fix — and arguably makes worse — is this: deep into a task, a capable agent makes a stream of decisions on your behalf. It substitutes an approach, narrows a scope, picks a default, "improves" something next door. The smarter the model, the more *plausible* those decisions look, which makes them easier to wave through without noticing — and harder to catch when one quietly diverges from what you wanted. You end up betting that it chooses right every single time, on every decision you never saw.

Atlas doesn't make the agent smarter. It makes those decisions, shortcuts, and unverified claims **impossible to hide** — so you can trust a strong model on long, high-stakes work where this kind of silent drift actually happens. See **[EXAMPLES.md](./EXAMPLES.md)** for a real run where both versions wrote good code, but only one let the user stay in the loop on a decision that changed the original request.

## The problem

A silent goal change rarely feels like betrayal from the inside. It feels like progress — like fixing the build, like a harmless simplification, like a sensible substitution. So asking the model "are you being honest?" doesn't help: that question goes to the same judgment that already drifted.

You don't fix this by making the model smarter. You fix it by changing the *process* around it: freeze the goal up front, surface the decisions that change it, and force every deviation to become an auditable event you can see and approve.

## The skills

| Skill | Role | When it runs |
|---|---|---|
| **[atlas-contract](./atlas-contract)** | In-conversation governance. Freezes the request into an auditable **Goal Contract**, scales its footprint to task complexity (Light / Medium / Heavy), stops on the *action* that changes the goal (substitute, narrow, mock, delete, weaken a test) instead of asking the model to judge its own risk, and runs an adversarial **Final Audit** that marks anything unproven as `Unverified` — never "done". | Every non-trivial coding task. |
| **[atlas-ledger](./atlas-ledger)** | Cross-session memory (compounding). When a drift is **caught**, it distills the lesson into a reusable project clause (`WHEN / DON'T / INSTEAD`) and, after you confirm, records it in `Atlas.md`. | Only after a caught drift. |

**How they compound:** `atlas-contract` enforces the goal in the moment. `atlas-ledger` turns each caught drift into a permanent clause in `Atlas.md`. The next time `atlas-contract` builds a contract, it loads the relevant clauses as guardrails — so the project's defense line thickens with every catch, instead of repeating the same mistake.

```
atlas-contract  ──catches a drift──▶  atlas-ledger  ──writes clause──▶  Atlas.md
       ▲                                                                    │
       └──────────────── loads clauses into the next contract ◀────────────┘
```

## What it looks like

On a non-trivial task, the agent freezes the goal and stops for your confirmation before touching code — so any decision that changes your request surfaces *before* it's acted on, not after:

```
Atlas Goal Contract

Goal: Add a live news headline per conversion, fetched from a real source.

Must Do:
- [M1] Fetch a real headline at request time (hard)

Must Not Do:
- [N1] No hardcoded, mocked, faked, or placeholder headlines (hard)

Preserve:
- [P1] Existing conversion / rate / history / delete must keep working (hard)

Summary: I'll fetch real headlines per currency and wire them in, without
breaking any existing behavior.

ATLAS_STOP: awaiting your confirmation before starting.
```

See **[EXAMPLES.md](./EXAMPLES.md)** for the full before/after run.

## How to use

Each skill is a single `SKILL.md`. Copy its contents into:

- **Codex / GitHub Copilot agent** — system instructions, or a `.md` skill file in the workspace
- **Claude** — Project Instructions
- **Cursor** — system prompt or `.cursorrules`
- **Any instruction-following agent** — wherever it reads a system prompt

No installation, no dependencies. Start with `atlas-contract`; add `atlas-ledger` once you want the project to remember its caught mistakes.

## Repository layout

```
atlas-contract/SKILL.md    in-conversation goal governance (start here)
atlas-ledger/SKILL.md      compounding memory of caught drift → Atlas.md
EXAMPLES.md                real before/after runs
LICENSE                    MIT
```

## Honest limits

These skills are enforced by the same model they govern. They raise the floor of goal-fidelity and make silent drift structurally harder — but a sufficiently drifted model can still produce a clean-looking audit over incomplete work, because the adversarial pass is also self-run. For high-stakes or long-running work, a **code-layer mechanical gate** — one that compares tool actions against the contract before they execute, without asking the model to judge — is the external backstop a prompt cannot provide by itself. That layer is **[atlas-agent](https://github.com/wede-wx/atlas-agent)**. Treat the Atlas skills as necessary layers, not a complete solution.

## License

[MIT](./LICENSE)
