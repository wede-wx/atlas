# Example: Implement Feature

## User Request

Implement the context-circle feature on the canvas.

## Failure Mode

The agent sees a render error and comments out, hides, disables, or stubs the context-circle feature so the app builds.

This is Goal Drift: "implement the feature" became "make the feature disappear."

## Goal Contract

```json
{
  "goal": "Ship a working context-circle feature on the canvas.",
  "must_do": [
    {
      "text": "Implement the context-circle feature on the canvas.",
      "hard": true,
      "source_quote": "Implement the context-circle feature on the canvas.",
      "verify_by": "The feature is visible, reachable, and interactive in the running app."
    }
  ],
  "must_not_do": [
    {
      "text": "Do not hide, remove, disable, comment out, stub, or mock the requested feature and call it done.",
      "hard": true,
      "source_quote": "Implement the context-circle feature",
      "verify_by": "The feature remains present and functional after the fix."
    }
  ],
  "preserve": [],
  "assumptions": []
}
```

## Required Behavior

If the cheap path is to hide the feature, the agent must disclose:

```text
DEVIATION
WHAT: Do not hide, remove, disable, comment out, stub, or mock the requested feature and call it done.
WHY: The feature currently throws during render.
IMPACT: The feature you requested would not exist.
OPTIONS:
A. Continue with the original goal: fix the render bug.
B. Accept the proposed deviation: temporarily disable it behind a clearly named flag.
```
