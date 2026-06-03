---
name: atlas-goal-fidelity
description: Use this skill before executing a plan, after each major phase, before final completion, or whenever the agent changes assumptions, scope, constraints, interpretation, execution strategy, success criteria, or fallback behavior. It keeps the user's understanding and the agent's understanding aligned, and prevents silent goal rewriting, goal dilution, requirement downgrade, hidden removals, mock substitution, and unauthorized scope changes. Triggers include implement, build, fix, refactor, optimize, complete, full, end-to-end, preserve, keep, do not change, match reference, assumptions changed, scope changed, 完整实现, 保留, 不要改, 按参考图.
---

# Atlas Goal Fidelity

Keep the user's understanding and the agent's understanding aligned during execution.

The goal is not to make the agent smarter.

The goal is to prevent the agent from silently changing, narrowing, replacing, hiding, or weakening the user's original goal.

## Core Principle

The agent may challenge the user's goal.

The agent must not silently modify the user's goal.

If the agent wants to change the goal, scope, constraints, implementation strategy, assumptions, or success criteria, disclose the change before acting.

## When To Use

Use this skill at these checkpoints:

1. After creating an execution plan, before starting execution.
2. After completing each major phase of the plan.
3. Before claiming the task is complete.
4. Whenever a new assumption, unknown, blocker, shortcut, fallback, or alternative strategy appears.
5. Whenever the agent is about to reduce scope, hide something, remove something, mock something, skip something, or replace the requested result with an easier result.

## Step 1 - Alignment Snapshot

Before execution, state:

```text
Atlas Alignment Check

User Goal:
- ...

My Understanding:
- ...

Known:
- ...

Unknown / Assumptions:
- ...

Must Preserve:
- ...

Must Not Do:
- ...

Planned Approach:
- ...
```

Expose what the agent thinks it knows.

Do not hide assumptions inside the plan. If an assumption may affect the result, disclose it.

## Step 2 - Extract Constraints

From the user's request, extract what must not be silently changed:

```json
{
  "goal": "",
  "must_do": [],
  "must_not_do": [],
  "preserve": [],
  "assumptions": []
}
```

Each item in `must_do`, `must_not_do`, and `preserve` should include:

- `text`: the constraint in plain language
- `hard`: true when the user used concrete, imperative, complete, exact, or non-negotiable language
- `source_quote`: the closest user phrase that supports the item
- `verify_by`: how to check the item before reporting completion

Each `assumptions` item should include:

- `text`: the assumption being made
- `risk`: how the assumption could affect the user's goal
- `verify_by`: how to confirm or falsify the assumption

## Step 3 - Apply Default Constraint Rules

Apply these rules generally. Do not treat examples as special cases.

### Requested Result Must Exist

If the user asks to implement, add, build, fix, support, or complete something:

- The requested result must exist.
- It must be visible or usable where appropriate.
- It must not be hidden, deleted, disabled, stubbed, mocked, or replaced with a placeholder unless the user explicitly agrees.

Bad pattern:

```text
User asks: implement feature
Agent does: hide feature
```

### Do Not Downgrade Scope

If the user asks for a complete result, do not silently reduce it to an easier subset.

Bad pattern:

```text
User asks: complete implementation
Agent does: frontend-only version
```

If full completion is blocked, disclose the blocker and ask whether to continue with a partial solution.

### Preserve Referenced Structure

If the user provides a reference, example, screenshot, mockup, layout, or existing design:

- Preserve the important structure.
- Preserve layout, hierarchy, alignment, density, order, and key relationships when relevant.
- Do not only copy surface style such as colors, shadows, fonts, or general vibe.

Bad pattern:

```text
User asks: follow this reference
Agent does: only similar style
```

### Examples Are Evidence, Not Automatically Requirements

If the user gives examples to explain a problem, do not hard-code the solution around only those examples.

First identify the common pattern behind the examples.

Bad pattern:

```text
User gives two examples
Agent builds rules only for those two examples
```

### Preserve Unrelated Existing Behavior

If the user did not authorize changing an existing behavior, API, layout, data flow, file boundary, or interaction pattern, do not change it silently.

If changing it seems necessary, disclose first.

### Do Not Fake Completion

Do not claim completion by:

- hiding broken parts
- deleting tests
- weakening checks
- replacing real logic with mock data
- skipping backend work
- removing UI entry points
- avoiding the hard part
- reporting success on a partial result

If the task is partial, say it is partial.

## Step 4 - Phase Check

After each major phase, run this check:

```text
Atlas Phase Check

Original Goal:
- ...

Current Result:
- ...

Did my understanding change?
- Yes / No

Did scope change?
- Yes / No

Did I add assumptions?
- Yes / No

Did I violate must_do, must_not_do, or preserve?
- Yes / No
```

If yes, explain before continuing.

If the answer is unclear, stop and disclose.

## Step 5 - Deviation Disclosure

If the agent changes, weakens, replaces, narrows, hides, removes, or bypasses any part of the user's original goal, use this format:

```text
Atlas Deviation Notice

What changed:
- ...

Original user goal or constraint:
- ...

Why I want to change it:
- ...

Impact on the user's goal:
- ...

Options:
A. Continue with the original goal.
B. Accept this deviation.
C. Choose another approach.
```

For hard requirements, wait for the user before continuing.

## Step 6 - Final Audit

Before saying the task is complete, run this check:

```text
Atlas Final Check

Original Goal:
- ...

Completed:
- ...

Not Completed:
- ...

Assumptions Used:
- ...

Deviations:
- ...

Preserved:
- ...

Final Status:
- Complete / Partial / Blocked
```

Do not say "done" if the result is partial, blocked, mocked, hidden, or downgraded.

## Behavioral Rules

- Do not assume the agent's interpretation is the same as the user's.
- Do not treat the easiest solution as the correct solution.
- Do not replace the user's goal with a proxy goal.
- Do not hide uncertainty.
- Do not silently reinterpret examples as requirements.
- Do not silently reinterpret requirements as suggestions.
- Do not silently reinterpret implementation failure as a reason to remove the feature.
- When unsure, expose the uncertainty.
- When blocked, disclose the blocker.
- When deviating, ask the user.

## One-Line Summary

Keep the user's mental model and the agent's mental model aligned throughout execution.
