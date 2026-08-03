# Usage

## Install

Claude Code skills load from a directory it scans at startup. Two options:

**Project-level** (only active inside a specific repo):

```bash
mkdir -p /path/to/your-project/.claude/skills/service-testing-skill
cp SKILL.md /path/to/your-project/.claude/skills/service-testing-skill/
```

**User-level** (active in every Claude Code session):

```bash
mkdir -p ~/.claude/skills/service-testing-skill
cp SKILL.md ~/.claude/skills/service-testing-skill/
```

Restart Claude Code (or start a new session) so it picks up the new skill.

## Prerequisites

- A testing repo with a `/testing-repo/tests/e2e/` structure (or point the skill at your actual path — it will create `pages/` and `specs/` subfolders per feature).
- Playwright + TypeScript already set up in that repo.
- Use-case docs to feed it: `UC-*.md` files or an `e2e-test-spec.md`, describing actors, main flow, alternate flows, and known edge cases.
- Read access to the main app's source (for selector conventions: `data-testid` > ARIA roles > IDs).

## Running it

Invoke naturally in a Claude Code conversation, e.g.:

> "Generate E2E tests from `use_cases/UC-012-login.md`."

> "Make the checkout flow test suite more robust — check for missing scenarios."

The skill triggers on requests to generate tests from use cases, or to review/strengthen existing E2E coverage.

## What to expect, step by step

1. **Parse** — Claude reads your use case doc and extracts actors, P0 main flow, P1 alternate flows, P2 edge cases.
2. **Gap analysis (you'll be asked something here)** — Claude checks the parsed flows against 8 lenses (negative input, auth/permission, network resilience, concurrency, accessibility, viewport, persistence, analytics) and lists any scenarios it thinks are missing, with a suggested priority. **Confirm which ones to include** — nothing gets added silently.
3. **Selector mapping** — Claude inspects your app's source to confirm real `data-testid`/ARIA/ID selectors exist before referencing them.
4. **Page Object generation** — writes/updates POM classes under `<feature>/pages/`.
5. **Spec generation** — writes Playwright specs under `<feature>/specs/`, labeled `[TC-ID] [Priority] [Type]`.
6. **Report** — Claude summarizes created files, a traceability matrix (use-case-derived tests plus a separate section for your approved `[GAP]`-tagged additions), and a deployment-readiness note.

## Tips

- The more detail in your use-case doc (actor names, exact preconditions), the less Claude has to guess at selectors and fixtures.
- When the gap-analysis checkpoint runs, you can reject proposed scenarios — say "skip the accessibility ones" and it moves on.
- `[UI]` and `[API/Analytics]` assertions are kept in separate tests by design; don't ask it to merge them.
