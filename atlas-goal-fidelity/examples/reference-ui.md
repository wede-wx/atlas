# Example: Reference UI

## User Request

Build this page to match the attached mockup.

## Failure Mode

The agent keeps the color palette and card styling, but changes the layout from a three-column reference into a single centered column.

This is Reference Drift: style survived, structure was lost.

## Goal Contract

```json
{
  "goal": "Build the page to match the attached mockup.",
  "must_do": [
    {
      "text": "Implement the page from the attached mockup.",
      "hard": true,
      "source_quote": "Build this page to match the attached mockup.",
      "verify_by": "The rendered page matches the mockup's key regions, hierarchy, and interactions."
    }
  ],
  "must_not_do": [
    {
      "text": "Do not treat visual style similarity as enough when layout structure differs.",
      "hard": true,
      "source_quote": "match the attached mockup",
      "verify_by": "Layout structure is checked separately from palette, shadows, and typography."
    }
  ],
  "preserve": [
    {
      "text": "Preserve the mockup's layout structure, element order, major regions, and alignment relationships.",
      "hard": true,
      "source_quote": "match the attached mockup",
      "verify_by": "Compare the rendered page against the mockup before final styling polish."
    }
  ],
  "assumptions": []
}
```

## Required Behavior

If the draft only matches the vibe, the agent must correct structure before calling the UI complete.

```text
DEVIATION
WHAT: Preserve the mockup's layout structure, element order, major regions, and alignment relationships.
WHY: The draft uses a single-column layout while the reference uses a three-column structure.
IMPACT: The page would feel related but would not match the requested UI.
OPTIONS:
A. Continue with the original goal: rebuild the layout structure first.
B. Accept the proposed deviation: keep the simplified layout as a looser interpretation.
```
