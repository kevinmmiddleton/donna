# Setting up Donna

This guide is written for two readers: a person setting Donna up by hand, and a coding agent (Claude Code, Cursor, Codex) asked to do it. The steps are the same. Where an agent has to behave differently, it says so.

**If you are an agent:** you were probably asked something like *"read SETUP.md and set up Donna for this repo."* Follow the steps in order. Every change you make is a **proposal**. Show it, wait for the person to accept it, then write it. Never mark a decision `accepted` on your own, never invent an owner, and never edit an existing ADR's prose.

---

## Step 0: Pick your track

Look for existing decision records. Common places: `docs/adr/`, `docs/decisions/`, `adr/`, `architecture/`, `doc/architecture/decisions/`. Files are usually named `NNNN-title.md` or `ADR-NNNN-title.md`.

- **Found ADRs?** Use **Track A** (teams).
- **Found none?** Use **Track B** (start from scratch).
- **Found design docs but no ADRs** (an `ARCHITECTURE.md`, a `CLAUDE.md`, `AGENTS.md`, a `CONTRIBUTING.md` with rules in it)? Use **Track B**, and seed the rules from those docs.

Also look for `.github/CODEOWNERS`, `CODEOWNERS`, or `docs/CODEOWNERS`. Note where it is. Donna reads owners from it and never guesses them.

---

## Step 1: Create `donna.json` (both tracks)

Run **Donna: Create Config File** in VS Code, or write `donna.json` at the repo root. Everything is optional. The minimum useful config:

```json
{
  "contextPack": { "name": "<your project> context", "synthetic": false },
  "adrDir": "docs/adr",
  "ownership": { "codeowners": ".github/CODEOWNERS", "records": [] },
  "services": []
}
```

Then fill in what fits:

| Field | What to put | Agent: how to find it |
| --- | --- | --- |
| `adrDir` | Folder holding your ADRs. Default `docs/adr`. | The folder from Step 0. |
| `ownership.codeowners` | Path to CODEOWNERS, or `null`. | From Step 0. If none exists, set `null` and say so. Do not create one. |
| `services` | Named areas of the code: `name`, `match` globs, `owners`, `dependsOn`. | Top-level folders under `src/`, `packages/`, `apps/`, or `services/` are a good first draft. Owners only from CODEOWNERS; leave `owners: []` otherwise. Mark the whole list as a draft for the person to correct. |
| `tests.match` | Test file globs. Defaults cover `*.test.ts`, `*.spec.ts`, `test_*.py`, `tests/**`. | Only change it if the repo's tests live somewhere unusual. |
| `designDocs` | Prose docs the model pass may cite, like `ARCHITECTURE.md`. | Any architecture or design doc you found. |
| `compat` | Public contracts (API schemas, exported types) and where their versions are recorded. | Leave empty unless the repo has an obvious versioned API. Ask. |

Leave `accepted` empty. That list is for findings a person has explicitly accepted, and only a person adds to it.

---

## Track A: You already have ADRs

An ADR stays prose. Donna only needs the part a machine can check, in a fenced `donna` JSON block, usually under an `## Enforcement` heading at the bottom.

### A1. Inventory

List every ADR with its status (Accepted, Superseded, Deprecated, Proposed). Donna only enforces ADRs whose status is `accepted`, `approved`, or `adopted`. Superseded and deprecated ADRs are skipped, so don't add blocks to them. A `proposed` ADR can carry a block; it starts being enforced the day its status changes.

**Agent:** show the inventory as a table before doing anything else:

| ADR | Status | Checkable? | Proposed kind | Why / why not |
| --- | --- | --- | --- | --- |

### A2. Decide what's checkable

Donna has six constraint kinds. A decision is checkable when it's about **which code may depend on which**, **which change needs which other change**, or **what a declaration must carry**. Match the decision's wording to a kind:

| If the ADR says... | Kind | Required `params` |
| --- | --- | --- |
| "X must not import / depend on Y" | `forbidden-dependency` | `from`, `to` (globs). Optional: `transitive` (bool), `allowFrom` |
| "Every X must go through Y" | `required-reachability` | `from`, `mustReach` (globs). Optional: `allowFrom` |
| "Changing contract X needs a version bump" | `contract-change-requires-version` | `surface`, `versionRegistry` (globs). Optional: `symbolPattern` (regex) |
| "Other services may only use X's public entry" | `service-public-entry` | `service` (name from `donna.json`), `entry` (globs). Optional: `allowFrom` |
| "Changing X needs a test change in the same commit" | `change-requires-test` | `when`, `requireChangedTest` (globs) |
| "Every X declaration must be marked with Y" | `required-decorator` | `files` (globs), `declPattern` (regex), `marker` (string). Optional: `withinLines` (default 12) |

Decisions about naming, style, team process, or anything that needs judgment ("prefer composition", "keep it simple") are **not checkable**. Leave those without a block. They are still cited, their prose still goes to the model pass, and the scorecard says they weren't machine-checked. An honest "unchecked" beats a forced rule that misfires.

Use `allowFrom` for tests. A rule that forbids unit-testing an internal module gets switched off.

### A3. Draft the block

Add it at the end of the ADR, under `## Enforcement`:

````markdown
## Enforcement

```donna
{
  "id": "ADR-0006",
  "status": "accepted",
  "appliesTo": ["src/metrics/**"],
  "constraints": [
    {
      "id": "metrics-has-no-upward-deps",
      "kind": "forbidden-dependency",
      "severity": "blocker",
      "rationale": "One sentence a reviewer will read in the scorecard.",
      "params": {
        "from": ["src/metrics/**"],
        "to": ["src/api/**", "src/engine/**"],
        "allowFrom": ["**/*.test.ts"]
      }
    }
  ]
}
```
````

Rules for the block:

- `id` matches the ADR's own number.
- `status` copies the ADR's status. **Agent:** copy it, never upgrade it.
- `appliesTo` is the code the decision governs. If a changed file doesn't match, the ADR is skipped as out of scope, so too narrow means unchecked (the scorecard counts skipped ADRs). Leaving `appliesTo` out makes the ADR repo-wide.
- `severity` is `blocker` (a conflict sends the change to design review) or `concern` (it shows up, but doesn't block). When unsure, start with `concern`.
- `rationale` is what a developer sees next to the finding. Write it for them, not for the ADR's author.
- Every glob must match at least one real file today. A constraint whose globs match nothing reports `unknown`, which is correct but useless.

**Agent:** propose blocks one ADR at a time, as a diff, with the matching files you checked each glob against. Wait for an accept before writing. If a decision maps poorly to every kind, say so and skip it.

### A4. Verify

Make a throwaway change that should violate one of the new rules (for example, add the forbidden import). Run **Donna: Assess Current Change**. You should see a conflict citing that ADR. Revert the change. Then check the scorecard's **"What this did not check"** section: every ADR listed there as "no donna block" should be one you chose to leave unchecked.

---

## Track B: Start from scratch

You don't need a decision log to get value. You need two or three rules you already follow.

### B1. Copy the starter rules

Copy [`docs/starter-adrs/`](docs/starter-adrs/) into your project's `docs/adr/`:

| File | Rule | Kind |
| --- | --- | --- |
| `ADR-0001-ui-does-not-import-data-layer.md` | UI code doesn't import the database or server layer | `forbidden-dependency` |
| `ADR-0002-api-changes-ship-with-a-test.md` | A change to an API handler comes with a test change | `change-requires-test` |
| `ADR-0003-routes-go-through-auth.md` | Every route reaches the auth check | `required-reachability` |

### B2. Fit the paths

Each starter uses common paths (`src/components/**`, `src/api/**`, `src/auth/**`). Change them to your layout. **Agent:** look at the repo, propose the path changes as a diff, and for each glob list two files it matches. Drop a starter that doesn't fit the project (for example, no auth layer), rather than bending it.

### B3. Add rules from what you already wrote down

If the repo has a `CLAUDE.md`, `AGENTS.md`, `ARCHITECTURE.md`, or `CONTRIBUTING.md`, read it for statements shaped like the table in A2: "never import", "always goes through", "must have a test". Each one is a candidate ADR. **Agent:** propose at most three, as new files `docs/adr/ADR-000N-<slug>.md` with a short Context and Decision in plain prose plus the block. Mark them `"status": "proposed"`. Donna doesn't enforce proposed ADRs, so nothing changes until the person flips the status to `accepted`.

### B4. Verify

Same as A4: one throwaway violation, one assessment, one revert.

---

## When you're done

You should have:

- `donna.json` at the repo root (or a context-pack folder set in the `donna.contextPack` setting)
- ADRs with `donna` blocks for every decision that's checkable, and an honest "unchecked" for the rest
- One verified conflict, so you know the wiring works

**Agent:** finish with a short summary: rules added, ADRs left unchecked and why, any `owners` left empty because there was no CODEOWNERS, and anything you skipped. Do not open a PR or commit unless you were asked to.

## What Donna will never do during setup

The same limits as everywhere else:

- She does not create, edit, or supersede ADRs herself. An agent following this guide proposes; a person accepts.
- She does not assign ownership. Owners come from CODEOWNERS or `ownership.records`, which a person maintains.
- She does not edit her own configuration to make a report green.
