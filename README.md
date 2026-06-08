# Atlas

A series of prompt skills that keep AI agents **honest about the goal** — they stop agents from silently narrowing, mocking, hiding, or faking completion, and make every deviation impossible to bury.

[中文说明 → README.zh-CN.md](./README.zh-CN.md)

---

## The problem

AI agents drift. Given a real task, an agent will quietly narrow the goal, mock a backend, weaken a test, change a file it was told to preserve — and then report success. It doesn't feel like betrayal from the inside; it feels like progress. Asking the model "are you being honest?" doesn't help: that question goes to the same judgment that already drifted.

You can't fix this by making the model smarter. You fix it by changing the *process* around it.

## The skills

| Skill | Role | When it runs |
|---|---|---|
| **[atlas-contract](./atlas-contract)** | In-conversation governance. Freezes the request into an auditable **Goal Contract**, scales its footprint to task complexity (Light / Medium / Heavy), stops before destructive or scope-changing actions, and runs an adversarial **Final Audit** before declaring anything done. | Every non-trivial coding task. |
| **[atlas-ledger](./atlas-ledger)** | Cross-session memory (compounding). When a drift is **caught**, it distills the lesson into a reusable project clause (`WHEN / DON'T / INSTEAD`) and, after you confirm, records it in `Atlas.md`. | Only after a caught drift. |

**How they compound:** `atlas-contract` enforces the goal in the moment. `atlas-ledger` turns each caught drift into a permanent clause in `Atlas.md`. The next time `atlas-contract` builds a contract, it loads the relevant clauses and pre-fills them as guardrails — so the project's defense line thickens with every catch, instead of repeating the same mistake.

```
atlas-contract  ──catches a drift──▶  atlas-ledger  ──writes clause──▶  Atlas.md
       ▲                                                                    │
       └──────────────── loads clauses into the next contract ◀────────────┘
```

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
LICENSE                    MIT
```

## Honest limits

These skills are enforced by the same model they govern. They raise the floor of goal-fidelity and make silent drift structurally harder, but a sufficiently drifted model can still produce a clean-looking audit over incomplete work. For high-stakes or long-running work, a **code-layer mechanical gate** — one that compares tool actions against the contract before they execute, without asking the model to judge — is the external backstop a prompt cannot provide by itself. That layer is **[atlas-agent](https://github.com/wede-wx/atlas-agent)**. Treat the Atlas skills as necessary layers, not a complete solution.

## License

[MIT](./LICENSE)
