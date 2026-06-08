---
name: atlas-ledger
description: "Companion to atlas-contract. Use after a coding task or phase governed by atlas-contract completes with a caught problem — a Final Audit marking Violation/Partial/Unverified that should have been Complete, a Post Review, a hard Deviation Notice, a recurrence caught in a Phase Check, or the user saying the result drifted/was wrong/incomplete/mocked (Chinese: 不对, 返工, 漏了, 这个记下来下次别再犯). It distills the caught drift into a reusable, project-local contract clause and, after user confirmation, records it in Atlas.md so the same drift is structurally prevented next time. Do not use on clean completions, optimization requests, ordinary code review, style preferences, or general takeaways; it records only caught drift, and only as enforceable clauses."
---

# Atlas Ledger v2

Give the Atlas series a memory.

`atlas-contract` defends the goal **within one conversation**, but it starts from zero every time — it does not know where this project drifted before. `atlas-ledger` closes that gap: when a drift is caught, it distills the lesson into a permanent, project-local **contract clause** and (after the user confirms) writes it to `Atlas.md`. Next time `atlas-contract` builds a Goal Contract, it loads the relevant clauses, so the defense line thickens with each catch. That is the compounding effect.

It is a **low-frequency, lightweight** companion. It runs only after a drift is caught, and it stays small on purpose. Do not turn it into a second heavy governance skill — its only hard job is distillation quality.

## Core idea

The job is **not** to keep a diary. A record of "what went wrong" is a memory; it changes nothing. The job is a translation:

> turn *this caught drift* → into *a clause that can enter a future contract and trigger a stop*.

A diary says "I hid the feature." A ledger clause says "WHEN a backend requirement is blocked, DON'T hide the feature, INSTEAD stop and disclose." Only the second one catches it next time. The entire value of this skill is the quality of that translation — and since it is run by the same model that drifted, the mechanisms below exist to keep it honest rather than trusting it to be careful.

---

# 1. Output Language

Write `Atlas.md` and all user-facing output in the language of the user's current instruction.

**Machine keys stay in English; clause content is localized.** Never translate the keys `WHEN` / `DON'T` / `INSTEAD`, the IDs (`L1`, `O1`), `seen`, `severity`, `Source`, `RETIRED`, or section headers `Confirmed Clauses` / `Provisional Observations` — atlas-contract parses these, and translating them makes the read-back unstable. The text after each key is written in the user's language. (E.g. `WHEN: 硬性 Must-Do 的后端部分受阻` — key English, content Chinese. Do **not** write `当: ...`.)

---

# 2. When To Run

Run distillation only when a drift has been **caught**:

1. an `atlas-contract` **Final Audit** marks something Violation / Partial / Unverified that should have been Complete;
2. an `atlas-contract` **Post Review** (the user said the result was wrong / incomplete / downgraded / mocked);
3. a hard **Deviation Notice** was raised;
4. a **Phase Check** catches the same class of error recurring;
5. the user explicitly says "record this so it doesn't happen again."

Do **not** run on: clean completions; optimization requests; ordinary code review; style preferences; general takeaways. There is nothing to enforce in those.

**Honesty boundary:** it can only learn from drift that was *detected*. Drift that slipped through unnoticed leaves no entry. Do not pretend the ledger is complete.

---

# 3. Distillation (the core)

Run in order. Output at most one clause per caught drift.

## Step 1 — State the drift as observable facts, not motive

Write what was objectively true, from the contract plus the delivered artifact — not why you think you did it.

- Good (fact): "[M2] required backend persistence (hard). Delivered code shipped the frontend with hardcoded data; no API or DB write exists."
- Bad (motive): "I thought the backend wasn't really necessary." Self-reported reasons are unreliable; a clause built on one prevents the wrong thing. Base the clause on the observable situation → action.

## Step 2 — Draft the clause: WHEN / DON'T / INSTEAD

```text
WHEN    <the situation that was true, generalized away from the specific subject>
DON'T   <the concrete wrong action taken>
INSTEAD <the concrete correct action>
```

Governing principle: **abstract the situation, keep the behavior concrete, base WHEN on facts not motive.** Drop the subject (feature name, file); keep the condition. The condition makes it match a future case; the subject makes it useless.

## Step 3 — Four acceptance gates (record only if it passes ALL four)

Run cheapest first.

1. **Actionability** — can the clause answer, concretely: what condition triggers it, what it forbids, and what to do instead? If any of the three is vague ("be more careful", "don't be lazy", "implement fully"), it is not a clause — discard. This gate exists to kill un-triggerable garbage before spending effort on the rest.
2. **Replay** — had this clause been in the contract this time, would it have caught this drift? If no → it does not describe what happened; rewrite.
3. **Generalization** — would it catch a *different* instance of the same situation (different feature, same shape)? If no → WHEN is still stuck to the subject; abstract further.
4. **Over-reach** — would it wrongly block a *legitimate* action elsewhere (e.g. the user explicitly approved frontend-first)? If yes → too broad; narrow it, usually by tightening WHEN.

If a candidate cannot pass all four, the lesson is not ready. **Record nothing rather than record noise.**

## Step 4 — Provisional vs confirmed

A single occurrence may be a fluke; do not over-fit.

- **First time** a situation is seen → record as a provisional **Observation** `[O#]`.
- A later caught drift whose WHEN **matches an existing Observation** → promote to a confirmed **Clause** `[L#]`, increment seen-count, remove the Observation.
- Only **confirmed clauses** are auto-loaded into future contracts; Observations are watched, not enforced.

**Severity exception — confirm on first occurrence** (skip the provisional stage) when the drift is any of:

1. mock / stub / fake data passed off as a real implementation;
2. hiding, deleting, or disabling a feature the user explicitly required;
3. weakening or deleting tests to force a pass;
4. data loss, broken persistence, or corrupted user data;
5. a security / permissions / auth mis-change;
6. a declared Preserve item broken;
7. downgrading Complete / end-to-end work to frontend-only.

Mark these `severity: high` and note `confirmed on first occurrence`.

## Step 5 — Propose, then write only after confirmation

`Atlas.md` is long-term project state — a wrong clause silently shapes every future contract. So the model does **not** write it unsupervised. Default flow:

```text
caught drift
 → draft clause (Steps 1–2)
 → pass four gates (Step 3)
 → output the candidate clause as a proposal
 → ATLAS_STOP, await user confirmation
 → on confirmation, write to Atlas.md (Step 6)
```

Only skip the stop if the user has explicitly said something like "auto-update Atlas.md". The confirmation is not red tape: it puts a human on the one artifact that is permanent, and lets the user fix a mis-distilled clause before it pollutes future work.

## Step 6 — Write to Atlas.md, merging first

Before adding, scan `Atlas.md` for an existing clause/observation with an overlapping WHEN.

- If one exists → **merge** into a single, more general clause, then re-run the four gates on the merged result. No near-duplicates.
- If confirmed clauses already number 15, merge the two closest before adding.

**Never only append.** A ledger that only grows hits the same long-context decay atlas-contract fights. Merging two concrete instances is often what produces the correctly-general rule.

---

# 4. Atlas.md format

One file at the workspace root. Stable structure (atlas-contract reads it). Keys English, content localized, `Source` anchored to the phase / event ID that caught it (not a guessed date — the model does not reliably know the date).

```text
# Atlas Ledger
<!-- Maintained by atlas-ledger. Confirmed clauses are loaded into new Goal Contracts by atlas-contract.
     Keys (WHEN/DON'T/INSTEAD, IDs, severity, Source) are fixed English; content is localized.
     Keep confirmed clauses general and <= 15. -->

## Confirmed Clauses
- [L1] (seen 2x, severity: high)
  WHEN:    硬性 Must-Do 的后端 / API / 持久化部分受阻或比预期更难
  DON'T:   用前端 mock、隐藏入口、静态数据或假成功来冒充完成
  INSTEAD: 停下来披露阻塞点，让用户决定继续原目标、批准偏离或改方案
  Source:  P3 Final Audit; P2 Post Review

## Provisional Observations
- [O1] (seen 1x)
  WHEN:    某个要求的测试失败且修复不明显
  DON'T:   削弱或跳过断言来让它通过
  INSTEAD: 报告失败，提出真实修复或发起偏离通知
  Source:  P2 Deviation Notice
```

---

# 5. Clause maintenance (keep the ledger alive, not ossified)

A clause distilled early can become wrong as the project evolves. The ledger must be able to shrink and retire, not only grow.

- The user may **retire** any clause at any time; mark it `RETIRED` (or remove it) and stop loading it.
- If a confirmed clause is **overridden by the user twice** (carried into a contract and waved off both times), flag it `review: stale` and surface it for retirement — it likely no longer matches the project.
- Retiring and merging are the two ways the ledger stays small; only-append is forbidden (Step 6).

---

# 6. Integration with atlas-contract (the read-back half)

This skill owns the **write** half. The **read** half is a single hook in atlas-contract's §6. Add this to atlas-contract:

```text
## Project Ledger Hook (read-back)

Before building the Goal Contract, check for Atlas.md at the workspace root. If it exists:
1. Read only the Confirmed Clauses (ignore Provisional Observations unless one is directly
   relevant and clearly marked advisory).
2. Match clauses whose WHEN is relevant to the current task.
3. Carry in at most 5 of the most relevant clauses — not all of them.
4. Convert each: DON'T -> a Must Not Do; INSTEAD -> its required response / stop rule.
5. Show them in the contract under "Carried-in Ledger Clauses" so the user sees the ledger working.

Precedence: ledger clauses are project DEFAULTS, not law. The user's current explicit instruction
always overrides a carried-in clause. If a carried-in clause conflicts with what the user is asking
for this time, do not silently enforce it — surface the conflict and let the user decide.

If Atlas.md is missing, malformed, stale, oversized, or ambiguous, say so in one line and continue
without pretending it was fully applied. Never fabricate clauses.
```

Without that hook the clauses are written but never enforced, and the ledger degrades into a diary. With it, every caught drift becomes a standing guardrail that routes through the mechanism that already works (the contract + the stop).

---

# 7. Final Principle

atlas-ledger turns a one-time, caught mistake into a permanent project constraint — that is the compounding. Its worth is entirely in distillation quality: too specific and it never fires, too broad and it fires constantly, built on a guessed motive and it guards the wrong thing. The four gates, confirm-before-write, merge-first, and retirement rules exist to hold that quality and keep the ledger small.

**Self-enforcement ceiling:** like atlas-contract, this skill is run by the same model it governs, so it can mis-distill or miss a drift worth recording. It raises the project's floor over time; it is not a guarantee, and the user confirming each clause is part of the design, not a formality. One more layer in the Atlas series — not a closed loop on its own.
