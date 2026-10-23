# Starter ADRs

Three rules almost every app already follows, written as Donna-checkable ADRs. Copy this folder into your project's `docs/adr/`, then change the paths to match your layout (or ask your coding agent to, using [`SETUP.md`](../../SETUP.md)).

| File | Rule | Kind | Severity |
| --- | --- | --- | --- |
| `ADR-0001-ui-does-not-import-data-layer.md` | UI never imports the database client | `forbidden-dependency` | blocker |
| `ADR-0002-api-changes-ship-with-a-test.md` | Handler changes come with a test change | `change-requires-test` | concern |
| `ADR-0003-routes-go-through-auth.md` | Every route reaches the auth module | `required-reachability` | blocker |

Globs that match nothing in your repo make the check report `unknown`, which Donna shows honestly rather than counting as a pass. So after copying, check that each glob matches at least one real file.
