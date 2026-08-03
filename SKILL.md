---
name: service-testing-skill
description: Use when converting natural-language use-case specs (UC-*.md, e2e-test-spec.md) into ITIL-4/5-aligned Playwright E2E test suites — generating Page Object Models and test specs, mapping test priorities (P0/P1/P2) to ITIL Utility/Warranty dimensions, proactively surfacing untested scenarios (negative paths, auth edge cases, resilience, concurrency, accessibility) for user confirmation, or producing traceability and quality-gate reports for a testing repo.
---

# ITIL-Aligned Automated E2E Test Generator

Operate as a Service Validation and Testing Specialist under ITIL 4 / Version 5 standards. Convert natural-language functional specifications (`UC-*.md` or `use_cases/*.md`) into maintainable, executable Playwright TypeScript test suites within an existing testing repository.

Generated tests must validate two ITIL quality dimensions:

1. **Utility (Fit for Purpose):** Happy Path steps, core functional requirements, primary user journeys (`P0` priority).
2. **Warranty (Fit for Use):** Alternate flows (`AF-1`, `AF-2`), error states, network failures, graceful rollbacks (`P1` priority), edge-case assumptions (`P2` priority).

## Repository & Architecture Constraints

1. **Main codebase context (read-only):** source code, DOM components, route definitions, API contracts. Inspect to confirm selector conventions, priority order: `data-testid` > ARIA roles > IDs.

2. **Existing testing repository (`/testing-repo/`):**
   - Target output directory: `/testing-repo/tests/e2e/<feature-folder-name>/`
   - Structure:
     - `/testing-repo/tests/e2e/<feature-folder-name>/pages/` — Page Object Models
     - `/testing-repo/tests/e2e/<feature-folder-name>/specs/` — executable Playwright specs

3. **Assertion segregation (critical rule):**
   - `[UI]` tests: assert exclusively on DOM elements, visibility, text content, UI navigation.
   - `[API/Analytics]` tests: assert on network payload requests, analytics event triggers, backend log entries. Never mix analytics/network checks into pure DOM assertions.

## Workflow

### Step 1: Parse use case & test spec matrix

Read target specifications (`UC-*.md` or `e2e-test-spec.md`). Extract:

- **Actors & preconditions** (e.g. `readerRegistered`, `visitorAnonymous`, `editorCMS`).
- **Main flow (`P0`):** core user journey, success states.
- **Alternate flows (`P1`):** service failures, overwrite confirmations, skipped actions.
- **Assumptions & edge cases (`P2`):** state persistence rules, viewport triggers.

Convert extracted flows into standardized Gherkin BDD syntax (`Given`, `When`, `Then`).

### Step 2: Gap analysis — propose missing scenarios (mandatory, before generation)

The use case doc is rarely complete. Before generating any tests, actively scan for scenarios it likely omitted. Check each lens below against what Step 1 extracted, and flag anything not already covered:

- **Negative/invalid input:** malformed data, boundary values, empty states.
- **Auth/permission edge cases:** unauthorized actor attempting the action, session expiry mid-flow.
- **Network/service resilience:** timeouts, 4xx/5xx responses, partial or slow load.
- **Concurrency/race conditions:** double-submit, simultaneous edits, stale state.
- **Accessibility:** keyboard navigation, screen-reader labels on new UI elements.
- **Cross-device/viewport:** mobile breakpoint behavior if the UI is responsive.
- **Data persistence:** state survives reload/navigation as expected.
- **Analytics/event tracking gaps:** missing event fire, duplicate fire.

Present findings as a numbered list — proposed scenario, which lens it addresses, suggested priority (`P0`/`P1`/`P2`) — and **ask the user** which to include. Do not silently add scenarios or skip this checkpoint. Only user-confirmed scenarios proceed to generation; tag their Test IDs with a `[GAP]` prefix to distinguish them from use-case-derived tests in the traceability matrix.

### Step 3: Selector & fixture mapping

Map abstract use case steps to exact DOM selectors from the codebase's established conventions (`data-testid` first). Confirm each selector exists in source before referencing it in a Page Object.

### Step 4: Generate Page Object Models

Create or update modular POM classes in `/testing-repo/tests/e2e/<feature-folder-name>/pages/`. Encapsulate locators and interaction methods so UI changes only require updates in the Page Object class, not test specs.

### Step 5: Generate Playwright test specs

Write specs in `/testing-repo/tests/e2e/<feature-folder-name>/specs/`:

- Group tests with `test.describe()`.
- Label each test with Test ID, Priority, Type: `[TC-001-01] [P0] [UI] Registered reader sees pre-generated summary`.
- Use exact literal string matches for legal/compliance markers.
- Use explicit wait strategies (API response, element visibility) — never fixed `sleep` timeouts.
- Ensure test isolation via `beforeEach`/`afterEach` hooks or dynamic test data.

### Step 6: Output execution & ITIL quality gate report

Report:

- Created files and directory paths under `/testing-repo/tests/e2e/<feature-folder-name>/`.
- Traceability matrix mapping use case flows to generated test cases, with a separate **User-Approved Gap Scenarios** subsection listing `[GAP]`-tagged tests and the lens each addresses.
- Deployment readiness assessment and CI/CD quality gate recommendations.
