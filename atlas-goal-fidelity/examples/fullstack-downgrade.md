# Example: Full-Stack Downgrade

## User Request

Fully implement user authentication, including the backend.

## Failure Mode

The agent builds a login screen, uses mock credentials, and reports completion without backend routes, sessions, or real round-trip behavior.

This is Requirement Downgrade: "fully implement" became "frontend-only demo."

## Goal Contract

```json
{
  "goal": "Fully implement user authentication, including backend behavior.",
  "must_do": [
    {
      "text": "Implement frontend login UI.",
      "hard": true,
      "source_quote": "Fully implement user authentication",
      "verify_by": "A user can enter credentials through the UI."
    },
    {
      "text": "Implement backend authentication behavior.",
      "hard": true,
      "source_quote": "including the backend",
      "verify_by": "The login flow calls real backend auth endpoints or services."
    },
    {
      "text": "Wire frontend and backend into a real login round trip.",
      "hard": true,
      "source_quote": "Fully implement user authentication, including the backend.",
      "verify_by": "A valid login succeeds through the real path and an invalid login fails."
    }
  ],
  "must_not_do": [
    {
      "text": "Do not report completion on frontend-only, mock-only, or partial authentication.",
      "hard": true,
      "source_quote": "Fully implement",
      "verify_by": "Completion is reported only after UI, backend, and real round-trip checks pass."
    }
  ],
  "preserve": [],
  "assumptions": []
}
```

## Required Behavior

If only the frontend is done, the agent must not say done. It should report partial status or continue backend work.

```text
DEVIATION
WHAT: Do not report completion on frontend-only, mock-only, or partial authentication.
WHY: Only the frontend login UI is currently implemented.
IMPACT: This is not the full authentication system requested.
OPTIONS:
A. Continue with the original goal: implement backend auth and wire the round trip.
B. Accept the proposed deviation: keep this as a UI-only prototype.
```
