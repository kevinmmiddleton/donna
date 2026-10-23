# ADR-0002: API changes ship with a test

- **Status:** Accepted
- **Owners:** (your handle or team)

> Starter rule from the Donna repo. Change the paths to match your project.

## Context

Coding agents are good at changing a handler and bad at noticing that nothing tests the new behavior. The old tests still pass because they test the old behavior. The gap shows up as a bug report, not a red build.

## Decision

**A change to an API handler comes with a test change in the same commit.** Adding, updating, or extending an existing test all count.

## Consequences

Small fixes need a small test. If a change genuinely needs no test (a comment, a log line), accept the finding in the scorecard and say why in the PR.

## Enforcement

```donna
{
  "id": "ADR-0002",
  "status": "accepted",
  "appliesTo": ["src/api/**", "src/routes/**", "src/server/**"],
  "constraints": [
    {
      "id": "api-change-needs-test-change",
      "kind": "change-requires-test",
      "severity": "concern",
      "rationale": "Existing tests cover the old behavior; a handler change needs a test that covers the new one.",
      "params": {
        "when": ["src/api/**", "src/routes/**", "src/server/**"],
        "requireChangedTest": ["**/*.test.{ts,tsx,js,jsx}", "**/*.spec.{ts,tsx,js,jsx}", "tests/**", "**/test_*.py"]
      }
    }
  ]
}
```
