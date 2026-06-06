# Atlas Skill

A prompt skill that keeps AI agents aligned with the original goal throughout execution — including across long, multi-phase sessions that would otherwise drift.

[中文说明 → README.zh-CN.md](./README.zh-CN.md)

---

## The problem

AI agents silently drift. They narrow the task, mock the backend, weaken a test, change a file they were told to preserve, and then report success. The agent doesn't experience this as betrayal — it feels like progress, like fixing the build, like a harmless simplification.

Asking the model "are you being honest?" doesn't fix this. That question goes to the same broken judgment that drifted in the first place.

## What Atlas Skill does

Before execution, it freezes the request into a **Goal Contract**: what must happen, what must not, what to preserve, scope boundaries, and how each item is verified.

During execution, it governs progress through **Phase Checkpoints** that stop before scope changes, silent narrowing, mock substitutions, failed validation, or unproven impact claims. Every stop is logged with an **Event ID** — goal changes, deviations, and unverified items cannot be buried in a progress summary.

Before completion, it runs an **adversarial audit**: not a self-reported summary, but a structured check against every contract item with concrete evidence. The model must assume it drifted and actively look for what it under-delivered.

## Why it survives long sessions

Context compaction drops constraints. After any compaction or session handoff, Atlas re-emits the confirmed contract **and** a five-rule anchor before continuing. This is why it can sustain 70+ phased tasks in a single conversation without goal drift.

---

## How to use it

Copy the contents of [`SKILL.md`](./SKILL.md) into one of:

**Codex / GitHub Copilot agent** — paste into the agent's system instructions or use it as a `.md` skill file in your workspace.

**Claude projects** — paste into the Project Instructions field.

**Cursor agent / system prompt** — paste at the top of your custom system prompt or `.cursorrules`.

**Any instruction-following model** — the skill works wherever the model follows a system prompt.

No installation. No dependencies. Drop in and go.

---

## What you get

When Atlas activates (Gate Mode), the agent outputs a Goal Contract before touching any code:

```
Atlas Event:
- Event ID: A1
- Type: Goal Contract
- Trigger Source: Skill-initiated
- Phase: P0
- Stop Status: Stop

Atlas Goal Contract

Goal:
- Implement full user authentication with JWT, including login, register,
  token refresh, and protected route middleware.

Must Do:
- [M1] Real JWT implementation — no hardcoded tokens, no mock auth (hard,
        source: "real implementation", verify: login returns a signed JWT,
        middleware rejects invalid tokens)
- [M2] Persistent sessions — tokens survive server restart (hard,
        source: "persistence", verify: token still valid after restart)

Must Not Do:
- [N1] No mock/stub/placeholder auth at any layer (hard)
- [N2] Do not change existing user schema (hard)

Preserve:
- [P1] Existing /api/users routes and response shapes (hard)

Completion Checks:
- [C1] Login with valid credentials → returns JWT
- [C2] Request with expired token → 401
- [C3] Protected route without token → 401

ATLAS_STOP: Awaiting confirmation before continuing.
```

After confirmation, for complex work it builds a **Phase Ledger** and runs a **Phase Check** at each boundary. Before declaring done, it runs the adversarial audit — checking every contract item against actual diffs and test output, not memory.

---

## What it prevents

| Silent failure mode | Atlas response |
|---|---|
| Mock the backend and call it done | Must Not Do → hard block before the action |
| Weaken or delete a failing test | Stop Before Actions (§5) → Deviation Notice |
| Narrow "full implementation" to frontend only | No Scope Downgrade default → must disclose |
| Forget a preserve constraint after compaction | Re-emits contract + rule anchor after compaction |
| Declare phase complete without checking | Phase Check requires evidence per contract item |
| "I'm confident this is isolated" | No Self-Adjudicate Impact → prove it or mark Unverified |
| Mark Final Audit Complete when items are mocked | Adversarial checklist: must inspect runtime code path |

---

## Compatibility

Works with any model that follows a system prompt:

- **GitHub Copilot / Codex** — primary development environment
- **Claude** (Projects, system prompt)
- **Cursor** (agent mode, `.cursorrules`)
- **ChatGPT** (custom instructions, system prompt)
- **Any other instruction-following agent**

The skill auto-detects the user's language and localizes all output labels. English and Chinese are fully tested; other languages follow the same localization rules.

---

## Related

**[atlas-agent](https://github.com/wede-wx/atlas-agent)** — a local-first Rust + Tauri desktop agent that pairs this skill with a code-layer mechanical enforcement gate (ContractGate, ImpactEvidenceGate). The skill is the declaration layer; atlas-agent is the mechanical enforcement layer.

---

## License

[MIT](./LICENSE)
