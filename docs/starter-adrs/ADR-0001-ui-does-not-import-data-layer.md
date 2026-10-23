# ADR-0001: UI code doesn't import the data layer

- **Status:** Accepted
- **Owners:** (your handle or team)

> Starter rule from the Donna repo. Change the paths to match your project.

## Context

The fastest way for a coding agent to get data onto a screen is to import the database client straight into a component. It works, the tests pass, and now your UI knows your schema. The next schema change breaks screens nobody thought to check, and queries end up scattered across components where nobody can find them.

## Decision

**Components and pages never import the database client or server-only modules.** They get data through the API layer (or hooks/loaders that call it).

## Consequences

Some small features need one more file (an endpoint or a hook) than the shortcut would. That file is where the query is easy to find, test, and change.

## Enforcement

```donna
{
  "id": "ADR-0001",
  "status": "accepted",
  "appliesTo": ["src/components/**", "src/pages/**", "src/app/**", "src/ui/**"],
  "constraints": [
    {
      "id": "ui-does-not-import-data-layer",
      "kind": "forbidden-dependency",
      "severity": "blocker",
      "rationale": "UI that imports the database client couples screens to the schema; route data through the API layer instead.",
      "params": {
        "from": ["src/components/**", "src/pages/**", "src/app/**", "src/ui/**"],
        "to": ["src/db/**", "src/server/**", "src/lib/db/**"],
        "allowFrom": ["**/*.test.{ts,tsx,js,jsx}", "**/*.spec.{ts,tsx,js,jsx}"]
      }
    }
  ]
}
```
