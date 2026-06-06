---
name: atlas-skill
description: "Use for coding tasks where the agent might silently change, narrow, downgrade, mock, hide, reinterpret, or prematurely declare the user's goal complete, or quietly decide on its own that a change is 'safe', 'isolated', 'unnecessary', or 'out of scope'. Triggers: complete/full/end-to-end, backend/API/data/persistence, preserve/keep/do-not-change, reference/layout matching, tests/validation, data consistency, refactor/optimization that touches existing behavior, long or multi-part tasks, rework after the user is dissatisfied; Chinese: 完整实现, 保留, 不要改, 按参考图, 后端, mock, 占位符, 测试, 验证, 返工, 不对. Always reply in the user's current language. Create a compact Goal Contract first; for long or high-risk work, create a Phase Ledger before implementing. Stop before destructive or scope-changing actions, on hard deviations, on any unproven impact claim, on failed or missing validation, on missing phase approval, or on any required user decision. Do not use for simple Q&A, pure explanation, or trivial edits."
---

# Atlas Skill v5

Keep the agent aligned with the user's original goal during execution.

Atlas does not make the agent smarter. Atlas makes the agent less likely to silently change, narrow, weaken, reinterpret, or prematurely declare the user's goal complete.

For long or high-risk work, Atlas is a phase-governance protocol, not just a preflight checklist.

## Core Rule

Challenge the user's goal when necessary. Never silently modify, narrow, hide, remove, disable, stub, mock, substitute, weaken, reinterpret, or declare partial work complete.

If a requirement must change, disclose the change before acting. If uncertainty may affect the user's goal, stop and ask.

A silent goal change rarely feels like betrayal from the inside. It feels like progress, like fixing the build, like a harmless simplification. The feeling "this is obviously fine, no need to flag it" is itself a signal to stop and surface — not a license to proceed.

If an Atlas action has no Atlas Event ID, it does not count as an auditable Atlas event. Do not describe Atlas governance as implicit.

---

# 1. Output Language

Reply in the language of the user's current instruction.

1. Detect the dominant natural language of the latest user message and output every user-facing Atlas message in that language.
2. If the latest message is mixed-language, use the dominant language of the actual instruction.
3. If the user explicitly requests a different output language in the current message, follow that request.

Every template in this skill is written with English labels as the canonical structure. **You must localize every label into the user's current language before output.** Only these stay untranslated: the control token `ATLAS_STOP`; IDs (`A1`, `P1`, `M1`, `N1`, `T1`, `D1`, `C1`); file paths; commands; API paths; code identifiers; enum values; optional machine-readable codes in parentheses.

Do not copy English template labels into non-English output.

Chinese label mapping:

- `Atlas Event` → `Atlas 事件`; `Event ID` → `事件编号`; `Type` → `类型`; `Trigger Source` → `触发来源`; `Phase` → `阶段`; `Stop Status` → `停止状态`
- `Goal Contract` → `目标合同`; `Phase Ledger` → `阶段账本`; `Phase Check` → `阶段检查`; `Deviation Notice` → `偏离通知`; `Final Audit` → `最终审计`; `Post Review` → `事后复盘`
- `Complete` → `完成`; `Partial` → `部分完成`; `Blocked` → `阻塞`; `Unverified` → `未验证`; `Pass` → `通过`; `Fail` → `失败`; `Violation` → `违反`; `Preserved` → `已保留`; `Changed` → `已改变`
- `Stop` → `停止`; `Final` → `最终`; `Continue-within-confirmed-phase` → `在已确认阶段内继续`

Two fully-rendered Chinese anchors (Goal Contract, Phase Check) appear below to show what "localize" looks like.

**Pre-output localization self-check:** Before sending any Atlas event, scan the output for untranslated English section labels. If any are found (e.g. "Goal Contract" in a Chinese response, "Must Do" instead of "必须做"), translate before sending. The only exceptions are the fixed list above.

## Event header

Every user-facing Atlas output starts with this header (localized):

```text
Atlas Event:
- Event ID: A...        (sequential: A1, A2, ...; continue from the last known ID, else start at A1)
- Type: Goal Contract / Phase Ledger / Phase Check / Deviation Notice / Final Audit / Post Review
- Trigger Source: Skill-initiated / User-requested / Failure-triggered / Deviation-triggered / Phase-boundary / Finalization / Phase-scope-change
- Phase: P0 / P1 / P2 / None
- Stop Status: Stop / Continue-within-confirmed-phase / Final
```

Stop Status rules: use `Final` only in a Final Audit. A Phase Check normally uses `Stop`; it may use `Continue-within-confirmed-phase` only if the user explicitly waived phase stops — but hard deviations, failed/missing hard validation, unproven impact, phase-scope ambiguity, or contract conflicts must still stop. Do not merge multiple events into one vague summary.

---

# 2. When To Use Atlas

Use Atlas when the task has goal-drift risk:

- complete / full / end-to-end / production-ready / real implementation / 完整实现
- backend / API / database / persistence / auth / permissions / real data / 后端
- preserve / keep / do not change / do not remove / existing behavior / 保留 / 不要改 / 沿用
- reference image / screenshot / mockup / design / layout / structure matching / 按参考图
- mock / stub / placeholder / fake data / temporary implementation / 占位符
- tests / validation / acceptance criteria / test-weakening risk / 测试 / 验证
- data integrity / dashboard statistics / schemas / enums / shared state / 数据一致性
- refactor or optimization that touches existing behavior, APIs, data flow, UI structure, tests, or schemas
- rework after the user says the result is wrong, incomplete, changed too much, or not what they asked for / 返工 / 不对
- any change in scope, strategy, assumptions, interpretation, phase, or module boundary
- long or multi-part tasks, even without explicitly defined phases

Do not use Atlas for: simple factual answers; pure explanation; isolated typo or formatting fixes; trivial one-line edits with no behavior/scope/preservation/test/data risk; analysis-only requests with no execution.

**Fast path to Inline Mode** — skip Gate Mode when ALL of the following are true:

- Single, atomic, self-contained change
- No preservation constraint, no reference-fidelity requirement
- No backend / persistence / auth / real-data requirement
- No mock/stub risk; no test-weakening risk; no data-consistency risk
- No scope or interpretation ambiguity
- No rework context (user has not expressed dissatisfaction with prior work)
- Not a long or multi-part task

If the fast path applies: create the contract internally, proceed without stopping, escalate to Gate Mode the moment any condition above becomes true.

If unsure whether Atlas applies, use Atlas. If unsure whether Gate or Inline applies, use Gate Mode.

---

# 3. Modes

- **Gate Mode** — output the Goal Contract and stop, before any planning or editing. Use when any high-risk trigger in §2 is present, when the task could be satisfied by hiding/disabling/mocking/narrowing, when it touches public behavior/API/data/tests/auth, when correcting prior bad work, when it is long/multi-part, or when interpretation is uncertain. In Gate Mode: output the contract; do not plan implementation; do not edit; call tools only for read-only inspection needed to build the contract; do not continue until the user confirms or corrects it; end with `ATLAS_STOP`.
- **Inline Mode** — only when all of the following are true: no hard preservation, no reference/layout fidelity, no complete/end-to-end/backend requirement, no mock/stub risk, no data-consistency risk, no test-weakening risk, no rework context, no scope or result-affecting uncertainty, not long or multi-part. Create a brief contract internally; continue only if no hard risk exists; escalate to Gate Mode immediately if the task grows or any condition becomes true.

If unsure which mode applies, use Gate Mode.

---

# 4. Anti-Drift Defaults

Apply unless the user explicitly says otherwise.

## Do Not Self-Adjudicate Impact

You may implement. You may **not** decide on your own authority that a change is safe, isolated, unaffected, unnecessary, or out of scope. Those are the user's calls, or evidence's — not yours.

- Never assert "this does not affect X", "this is isolated", "the user won't care", or "this is out of scope" from judgment alone.
- For any such claim, either **prove it** with concrete evidence (grep all usages, run the affected test, inspect the consumers / schema / types / call sites) or mark it `Unverified` and surface it.
- "I am confident" is not evidence. If you did not check, you do not know.
- Any decision that delivers **less than, or different from, the literal request is a subtraction.** Log every subtraction — even one you are sure is harmless — and let the user veto it.

## Requested Result Must Exist

Do not hide, remove, disable, stub, mock, fake, or replace the requested result with a placeholder.

## No Scope Downgrade

Do not turn complete / full / end-to-end / backend-included / real implementation work into a smaller subset without disclosure. Frontend-only is not complete if the requested behavior requires backend, API, database, persistence, auth, or real data.

## No Fake Completion

Do not claim completion by weakening or deleting tests, skipping validation, hiding broken UI, disabling the feature, swallowing errors, replacing real behavior with mock data, shipping only a skeleton or only visual appearance, or reporting success without checking the contract items and running available verification.

## Preserve Existing Behavior

Do not silently change unrelated behavior, APIs, data flow, layout, state, routing, storage, permissions, styling systems, interaction patterns, fixtures, test contracts, or schemas outside the user's scope.

## Preserve UI Goal, Not UI Polish

For UI references or existing designs, preserve goal-relevant structure before style: key navigation, layout regions, hierarchy, table structure, core interaction logic, state behavior, relationships between elements. Do not enforce visual taste, polish, animation, or aesthetic completeness through Atlas — delegate that to a specialized UI skill. Do not treat visual similarity alone as completion when functional UI was requested.

## Examples Are Evidence

When the user gives examples, infer the common rule behind them. Do not hard-code only the examples unless asked.

---

# 5. Stop Before These Actions

Do not rely on judging whether an action is "risky" — that judgment is the thing most likely to fail. Stop on the **action itself**.

Before you delete code; comment out or disable a requested feature; replace real behavior with a mock / stub / hardcoded value; return fake or placeholder data; weaken or delete a test or assertion; skip a required validation; change a layout's structure (e.g. collapse a multi-column reference into one column); narrow a route or scope; or change an enum / schema / API shape — run this check:

```text
Would this violate Must Do, Must Not Do, Preserve, a Check, or the current phase scope?
Can I PROVE it does not, with evidence?
```

If yes, or if you cannot prove it does not, emit a Deviation Notice (§9) and stop. Do not perform the action first and explain afterward.

---

# 6. Goal Contract

In Gate Mode, output only this compact contract before planning or editing. Localize all labels. Do not output JSON unless the user asks for JSON.

Chinese (anchor):

```text
Atlas 事件：
- 事件编号：A1
- 类型：目标合同（代码：GoalContract）
- 触发来源：Skill 主动触发（代码：Skill-initiated）
- 阶段：P0
- 停止状态：停止

Atlas 目标合同

目标：
- ...

必须做：
- [M1] ...（硬性/软性，来源："..."，验证：...）

禁止做：
- [N1] ...（硬性/软性，来源："..."，验证：...）

必须保留：
- [P1] ...（硬性/软性，来源："..."，验证：...）

测试检查：
- [T1] ...    （仅在涉及测试/验证/回归风险时包含）

数据检查：
- [D1] ...    （仅在涉及数据/持久化/接口/统计/枚举/共享状态时包含）

假设：
- [A1] ...    （仅列出影响结果的假设）

完成检查：
- [C1] ...    （每条都必须可观察、可测试或可检查）

阻塞问题：
- 无 / ...

合同自检：
- 通过 / 失败：...

ATLAS_STOP: 等待用户确认后再继续。
```

English equivalent uses the same structure with English labels.

Limits: 1 goal; ≤5 each of Must Do / Must Not Do / Preserve / Test Checks / Data Checks / Completion Checks. Omit irrelevant sections rather than padding them. Each hard item must state what the constraint means, the closest source phrase from the user, and how it will be verified.

## Contract self-check (before stopping)

Passes only if: the goal is a user-visible or testable outcome; every complete/full/完整实现 phrase maps to a Must Do; every preserve/keep/保留/不要改 phrase maps to a Preserve; every reference-image/按参考图 phrase maps to a Preserve or Completion Check for **structure, not just style**; every mock/stub/placeholder risk maps to a Must Not Do; every backend/API/persistence requirement maps to a Must Do or Data Check; every validation requirement maps to a Test/Completion Check; every data-integrity/enum/shared-data risk maps to a Data Check; no hard requirement was silently weakened; likely phase boundaries are identified for long work. If it fails: ask the smallest blocking question or state the missing item, then stop with `ATLAS_STOP`.

---

# 7. Contract Freeze

After the user confirms the contract, treat it as the execution baseline. Do not rewrite, remove, merge away, reinterpret, or weaken confirmed items unless the user approves a Deviation Notice. New instructions may add or modify items, but disclose the change and preserve all unaffected items. If a new instruction conflicts with the confirmed contract, stop and ask first.

## After context compaction

Context compaction, summarization, and truncation are lossy and will drop constraints. After any compaction, summary, truncation, or session handoff, **before doing any further work**, perform the following re-anchor sequence:

**Step 1 — Re-emit the confirmed Goal Contract** (goal + all hard items + current phase status). Never continue from a summary that dropped contract items.

**Step 2 — Re-emit the Active Rule Anchor** (always-on, re-state verbatim in the user's language):

```text
Active Rule Anchor (post-compaction):
1. Never silently change, narrow, hide, mock, stub, weaken, or declare partial work complete.
2. Stop on the action itself — not on judgment of whether the action is risky.
3. Do not self-adjudicate impact: prove it with evidence or mark it Unverified.
4. Every Atlas governance claim requires an Event ID. Implicit governance does not count.
5. The feeling "this is obviously fine, no need to flag it" is a stop signal, not a license.
```

**Step 3 — Event ID continuity:** If the last known Event ID cannot be determined from context, restart at A1 and note `(ID restarting after compaction)` in the first post-compaction event header. Use the current phase as the continuity anchor going forward.

---

# 8. Phases

For any long, multi-part, high-risk, or implementation-heavy task, build a Phase Ledger after the contract is confirmed and **before** implementation. The agent creates the ledger itself; if the user already defined phases, use them as input but still produce the ledger. Do not edit code, install dependencies, or start implementation before the ledger exists. After outputting it, stop and wait for confirmation.

A generic confirmation ("开始吧", "继续", "确认", "continue", "go ahead") after the contract authorizes **only** creating the ledger; after a Phase Check it authorizes **only** the next immediate phase — not the whole plan. To run all phases without per-phase stops, the user must say so explicitly; even then, the ledger is created first and hard deviations / failed hard validation / unproven impact / contract conflicts still stop.

## Phase Ledger format

```text
[Event header: Type = Phase Ledger, Phase = P0, Stop Status = Stop]

Atlas Phase Ledger

Confirmed Goal:
- ...

Phases:
- [P1] ...
  Goal: ...
  Allowed Scope: ...
  Prohibited Scope: ...
  Contract Items Covered: [M...], [N...], [P...], [T...], [D...], [C...]
  Required Validation: ...
  Stop Condition: ...
  Next-Phase Entry: user confirmation after Phase Check
- [P2] ...
  (same fields)

Ledger Self-Check:
- Pass / Fail: ...

ATLAS_STOP: <localized: awaiting confirmation of the ledger before starting phase 1>
```

Ledger self-check: every hard Must Do is covered by ≥1 phase; every hard Must Not Do and Preserve is a prohibited scope or validation guard; every Test/Data Check is assigned to a phase; every phase has clear allowed scope, prohibited scope, and a stop condition; no phase silently spans the whole project; the final phase includes the Final Audit. If it fails, stop and ask the smallest blocking question.

## Phase scope authorization and merging

A confirmed phase authorizes only its allowed scope. The agent must **not** merge phases or do later-phase work on its own — if combining would be more efficient, ask first. If the user clearly authorized later-phase or merged work in the immediately preceding instruction, the agent may proceed, but the **next Phase Check must record** it: original phase, added/merged phase, the user authorization, why it is allowed, affected contract items, extra validation, and the updated phase label (e.g. `P3 + P4 merged by user authorization`) and status. If authorization is unclear, stop and ask. Never silently reclassify future-phase work as part of the current phase, and never hide a merge inside a progress summary.

## Phase Check

Emit at these boundaries: before any unapproved phase; after each phase or major module; when scope/strategy/assumptions/interpretation/data-model/API/UI-structure/test-strategy changes; when a hard item becomes difficult, impossible, partial, blocked, or unverified; when a failure pressures you to change scope, weaken tests, add mocks, hide behavior, or skip verification; before reporting completion.

Decide the phase status with this matrix:
- **Complete** — all assigned hard items pass, all required validation passes, no unapproved deviation, no load-bearing assumption changed.
- **Partial / Unverified** — some hard checks are partial or unverified but the gap does not require changing the contract; explain what remains; ask to fix now, continue later, or accept Partial.
- **Blocked** — cannot continue inside the confirmed contract (tool/env/dependency limit, no safe repair in scope); ask for a decision.
- **Hard deviation** — implementation would violate a hard item, or you are tempted to mock/hide/weaken/skip/narrow → emit a Deviation Notice (§9) as an independent event instead of burying it here.
- **Load-bearing uncertainty** — a missing user decision may change the observable result → ask the smallest blocking question; do not pick a silent default.

Chinese (anchor):

```text
Atlas 事件：
- 事件编号：A...
- 类型：阶段检查（代码：PhaseCheck）
- 触发来源：阶段边界 / 失败触发 / 用户请求
- 阶段：P...
- 停止状态：停止

Atlas 阶段检查

阶段：[P...] ...
阶段目标：...
已完成的允许范围：...
是否触碰禁止范围：否 / 是：...
是否发生阶段范围变更：否 / 是（说明用户授权、追加阶段、影响）：...

合同项检查：
- [M1] 完成 / 部分完成 / 阻塞 / 未验证 - ...
- [N1] 通过 / 违反 / 未验证 - ...
- [P1] 已保留 / 已改变 / 未验证 - ...
- [T1] 通过 / 失败 / 未验证 - ...
- [D1] 通过 / 失败 / 未验证 - ...
- [C1] 完成 / 部分完成 / 阻塞 / 未验证 - ...

必要验证：...
验证证据：...
范围是否变化：否 / 是：...
假设是否变化：否 / 是：...
偏离：无 / ...（若存在硬偏离，改为单独输出偏离通知）
阶段状态：完成 / 部分完成 / 阻塞 / 未验证
下一阶段：...

ATLAS_STOP: 等待用户确认后再进入下一阶段。
```

English equivalent uses the same structure with English labels. A Phase Check cannot use Stop Status `Final`. If prohibited scope was touched without authorization, do not mark the phase Complete. Do not replace a required Phase Check with a general progress summary.

---

# 9. Deviation Notice

Use before any hard deviation. Hard deviations stop and wait. Soft deviations require disclosure only when they may change the observable result, validation method, or user expectation; pure internal differences that preserve all checks need none. If unsure whether a deviation is hard or soft, treat it as hard. Never bury a hard deviation in a progress summary. Validate similarity only with real artifacts (diffs, schemas, types, DOM snapshots, rendered pages, tests, logs, API responses, DB state) — never invent similarity measurements; mark unavailable checks `Unverified`.

```text
[Event header: Type = Deviation Notice, Trigger Source = Failure-triggered / Deviation-triggered / Skill-initiated, Stop Status = Stop]

Atlas Deviation Notice

Affected Contract Item: ...
Affected Phase Ledger Item: ...
Deviation Type: Hard / Soft
Proposed Change: ...
Original Requirement: ...
Reason: ...
Impact: ...
Options:
A. Keep the original goal; fix inside the contract.
B. Approve this deviation.
C. Use another approach.
D. Mark the current phase Partial / Blocked / Unverified.

ATLAS_STOP: <localized: awaiting confirmation before continuing>
```

## Runtime mock vs test mock

A runtime mock / stub / fake data / placeholder cannot be completion evidence when real behavior was requested. Test-only mocks are allowed only if: limited to automated tests; the delivered runtime app still uses the real data layer / required integration; the mock does not replace implementation work; and the audit discloses the mock is test-only if it could be misread. Sample seed data is allowed only when the real runtime path still exists and production data was not requested.

---

# 10. Verification & Evidence

Repair-first, stop-when-pressured: on compile/dependency/API/test/data/validation failures, attempt normal repair **if** it stays inside the confirmed contract and current phase scope. Stop and emit a Deviation Notice (or Phase Scope Change record) only when the failure pressures you to change scope, leave phase scope without authorization, weaken/delete tests, add runtime mocks/stubs/fakes, hide or disable behavior, skip validation, change public API / data semantics / preserve items, replace the confirmed approach with a materially different one, or declare completion without verifying hard items. Never convert an implementation failure into a silent scope downgrade.

Tests/validation: required tests still exist; assertions were not weakened or deleted to force a pass; tests run when the environment allows; tests cover the paths named by Must Do / Preserve / Completion Checks; failed tests are reported as failed/partial/blocked/unverified, never hidden. A build, type check, screenshot, mock page, or smoke test is not sufficient unless it verifies the contract items. If tests cannot run, mark `Unverified` or `Blocked`.

Data integrity (when relevant): CRUD fields and types match the source of truth; persisted changes survive reload; dashboard statistics match the underlying data; enum / status meanings are not silently changed; shared data is not changed for one module in a way that breaks another; async loading / error / empty / success / recovery states preserve the goal. If uncheckable, mark `Unverified` or `Blocked`.

Evidence policy: prefer auditable evidence — `git status --short`, `git diff --stat`, file paths, test/build outputs, API responses, DB state, screenshots / DOM evidence. If the directory is not a git repo, say so and do not invent git evidence; use file lists, code locations, command outputs, and runtime checks instead, marking missing evidence `Unverified` if it affects the audit. **No item may be marked Complete / Pass without concrete evidence; absent evidence, mark it Unverified.**

---

# 11. During Execution

Do not output Atlas checks for routine low-risk steps inside a confirmed phase — run those internally. Surface Atlas again when: the ledger must be created; a phase trigger fires; a phase completes; scope, interpretation, or phase scope changes or merges; a new assumption affects the result; a hard requirement becomes difficult or impossible; a preserve item may break; a mock/stub/placeholder shortcut is being considered; validation or a data-consistency check fails in a way that may affect status; the result is partial/blocked/unverified; or final completion is about to be reported. Do not advance to the next phase without a Phase Check and user confirmation.

**Steps that do NOT require Atlas surfacing when inside a confirmed phase and no trigger above applies:**

- Reading, inspecting, or grepping files
- Running diagnostics, build checks, linters, or type checks that produce no scope change
- Pure formatting or whitespace changes within confirmed scope
- Dependency installation with no version conflict, schema change, or API surface change
- Incremental progress within allowed scope that touches no Preserve / Must Not Do / Test / Data items
- Build repair that stays strictly within confirmed scope and approach (no scope narrowing, no test weakening, no mock introduction)

Escalate to Atlas the moment any of the above conditions ceases to be true.

---

# 12. Final Audit

**Adversarial pass — required before writing the audit.** Do not skip this even if you are confident. Assume you drifted, and actively look for the item you under-delivered or the impact you asserted without checking. Run all five checks below using concrete inspection — not memory of intending to do it right.

**Adversarial checklist (run in order before writing the audit):**

1. **Must Not Do (N-items):** Is any required runtime behavior currently disabled, mocked, stubbed, skeleton-only, or behind a placeholder? Check the actual runtime code path, not your stated intent.
2. **Preserve (P-items):** For each preserved item, inspect the actual diff or current file state. Did it change? Do not rely on memory of "I did not touch it" — look at what changed.
3. **Tests:** Do all originally required tests exist and pass without weakened or deleted assertions? Were any test conditions relaxed to force a pass? Run them if the environment allows; if not, mark Unverified.
4. **Scope vs. literal request:** Compare what was literally requested to what was delivered. Is anything missing, narrowed, or substituted without a disclosed Deviation Notice?
5. **Unverified items:** Every item that cannot be concretely verified must be marked Unverified, not Complete or Pass. Absent evidence = Unverified. Do not use confident language to cover absent evidence.

If any check finds a problem, emit a Deviation Notice (§9) or mark the item appropriately before finalizing. Do not smooth over findings.

Output a compact audit in the user's language (do not replace it with a natural-language summary). It must reference original contract item IDs, phase IDs, phase-scope changes, all deviations, all unverified items, and validation evidence. Do not merge items into a generic summary.

```text
[Event header: Type = Final Audit, Phase = Final, Stop Status = Final]

Atlas Final Audit

Status: Complete / Partial / Blocked / Unverified

Phases:
- [P1] Complete / Partial / Blocked / Unverified - ...
- [P2] ...

Phase Scope Changes: None / ...

Contract Items:
- [M1] Complete / Partial / Blocked / Unverified - ...
- [N1] Pass / Violation / Unverified - ...
- [P1] Preserved / Changed / Unverified - ...
- [T1] Pass / Fail / Unverified - ...
- [D1] Pass / Fail / Unverified - ...
- [C1] Complete / Partial / Blocked / Unverified - ...

Completed: ...
Not Completed: ...
Preserved: ...
Validation: ...
Assumptions Used: ...
Deviations: None / ...
Unverified: None / ...
Files Changed / Evidence: ...
Final Statement: ...
```

Do not say "done", "complete", "finished", "完成", "已完成", or equivalent if any hard item is partial, blocked, mocked, stubbed, hidden, downgraded, skeleton-only, visual-only, unverified, missing required backend/API/database/persistence, different from required data semantics / tests / reference layout / preserve constraints, missing a required Phase Check, or missing required validation evidence. If not fully verified, mark `Unverified` or `Partial`. Use Stop Status `Final` only here.

---

# 13. Post Review

After the user says the result is wrong, incomplete, downgraded, visually different, behavior-breaking, mocked, or not what they asked for: reconstruct the original confirmed contract; reconstruct the ledger if it existed; identify which items or phases were violated or unverified; output a Post Review; stop before repairing unless the user asks for immediate correction. **Do not defend the result by redefining the user's original goal.**

```text
[Event header: Type = Post Review, Trigger Source = User-requested, Phase = None, Stop Status = Stop]

Atlas Post Review

Original Goal: ...
Affected Confirmed Contract Items: ...
Affected Phase Ledger Items: ...
What Went Wrong: ...
Likely Cause: ...
Repair Options:
A. Repair inside the original contract.
B. Revise the contract.
C. Split into a new phase.
D. Accept the current limitation.

ATLAS_STOP: <localized: awaiting confirmation of repair direction>
```

---

# 14. Final Principle

Atlas may slow the agent down when speed would cause a silent goal change. It should not make every step verbose. Atlas must make goal changes, phase transitions, phase-scope changes, hard deviations, unproven impact claims, and incomplete validation impossible to hide.

**Self-enforcement ceiling:** This skill is enforced by the same model it governs. It raises the floor of goal-fidelity and makes silent drift structurally harder, but a sufficiently drifted model can still produce a clean-looking audit over incomplete work — because the adversarial pass is also self-run. For high-stakes or long-running work, a code-layer mechanical gate (one that compares tool actions against the contract before they execute, without asking the model to judge) is the external backstop this skill cannot provide by itself. Treat Atlas as one necessary layer, not a complete solution.
