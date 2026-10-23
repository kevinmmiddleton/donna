# ADR-0003: Every route goes through auth

- **Status:** Accepted
- **Owners:** (your handle or team)

> Starter rule from the Donna repo. Change the paths to match your project. If your app has no auth layer, delete this file rather than bending it.

## Context

A new endpoint that forgets the auth check looks exactly like one that doesn't: same shape, same passing tests (tests usually run as a signed-in user). It's the kind of mistake that's invisible in a diff and expensive in production.

## Decision

**Every route module reaches the auth module.** Public routes (health checks, webhooks with their own signature check, login itself) live in a separate folder that's exempt.

## Consequences

Public endpoints have to be put somewhere on purpose. That's the point.

Note: Donna checks that a route *imports* (directly or through other modules) the auth module, not that it calls it on every code path. It catches the forgotten check, not a misplaced one.

## Enforcement

```donna
{
  "id": "ADR-0003",
  "status": "accepted",
  "appliesTo": ["src/api/routes/**", "src/routes/**"],
  "constraints": [
    {
      "id": "routes-reach-auth",
      "kind": "required-reachability",
      "severity": "blocker",
      "rationale": "A route that never reaches the auth module is publicly reachable; put intentionally public routes in routes/public/.",
      "params": {
        "from": ["src/api/routes/**", "src/routes/**"],
        "mustReach": ["src/auth/**", "src/lib/auth/**", "src/middleware/auth.*"],
        "allowFrom": ["src/api/routes/public/**", "src/routes/public/**", "**/*.test.{ts,tsx,js,jsx}"]
      }
    }
  ]
}
```
