---
name: e2e-architect
description: "Use when creating, auditing, or improving E2E tests that validate real business flows instead of just UI elements. Triggers on: 'generate e2e tests', 'audit my e2e tests', 'write business-driven tests', 'e2e coverage for this feature', 'what flows are untested', 'improve test quality', 'tests from requirements', or any request to make E2E tests smarter and business-aware."
argument-hint: "[discover | generate | audit] — omit for full pipeline"
---

# E2E Architect — Business-Aware Test Generation

Generate E2E tests that validate **business outcomes**, not DOM elements.
Instead of "button is visible", test "new user can publish their first project."

## Core Principle

Understand the business FIRST, then write tests. Every test must trace back
to a business flow that a real persona performs. If you can't explain which
persona benefits and what business outcome is validated, the test is shallow.

## Modes

| Mode | Argument | Output |
|---|---|---|
| **Discover** | `discover` | Business flow map at `docs/e2e-architect/business-flow-map.md` |
| **Generate** | `generate` | Playwright `.spec.ts` files + coverage matrix |
| **Audit** | `audit` | Audit report classifying existing tests |
| **Full pipeline** | _(no arg)_ | discover → approval gate → generate → coverage report |

### Mode Dependencies

`generate` and `audit` require a flow map. If `docs/e2e-architect/business-flow-map.md`
does not exist, run `discover` first automatically.

## Full Pipeline (default — no argument)

When invoked without an argument, run the complete pipeline:

1. **Discover** — scan repo, build flow map (see `references/discovery-engine.md`)
2. **Infer flows** — structure personas, journeys, rules (see `references/flow-inference.md`)
3. **GATE: User approval** — present the flow map, ask for corrections. Do NOT proceed until approved.
4. **Generate** — create Playwright tests from approved flows (see `references/test-templates.md`)
5. **Report** — produce coverage matrix (see `references/coverage-format.md`)

## Discover Mode

Read `references/discovery-engine.md` for the complete scanning strategy.

**Summary:** Scan the repo for business context across 8 source types (requirements docs,
README, routes, types, API endpoints, database schema, existing tests, knowledge graph).
If insufficient context is found (< 3 personas or < 5 journeys), enter fallback interview
mode — ask the user targeted questions about their product.

**Optional browser exploration:** If browser automation tools are available (Playwright MCP,
agent-browser, chrome-devtools-mcp) and the app is running locally, navigate key routes to
discover actual UI state. This is additive — the skill works without it.

After discovery, use `references/flow-inference.md` to structure findings into a business
flow map. Write it to `docs/e2e-architect/business-flow-map.md` using the format in
`references/coverage-format.md`.

## Generate Mode

Read `references/test-philosophy.md` before writing any test code.
Use `references/test-templates.md` for Playwright patterns.

**Prerequisites:**
- A flow map must exist at `docs/e2e-architect/business-flow-map.md`
- The user must have approved the flow map (if running in full pipeline)

**Process:**
1. Read the flow map — identify all journeys grouped by priority tier (P0–P3)
2. Discover existing test infrastructure:
   - Scan for `playwright.config.*` or `cypress.config.*` — determines framework
   - Scan for `poms/`, `page-objects/` — existing page object models
   - Scan for `fixtures/`, `utils/`, `helpers/` — existing test utilities
   - Scan for `*.spec.ts`, `*.test.ts`, `*.e2e.ts` — existing test files
3. For each journey (P0 first, then P1, P2, P3):
   - Generate a `.spec.ts` file using patterns from `references/test-templates.md`
   - Reference existing POMs/utils where they exist
   - Use business-aware test names (user stories, not DOM descriptions)
   - Assert business outcomes, not element visibility
4. Write coverage matrix to `docs/e2e-architect/coverage-matrix.md`

**File placement:** Place generated test files in the same directory as existing tests.
If no existing tests, create `tests/journeys/`. If uncertain, ask the user.

**Never overwrite** existing test files. Generate new files alongside them. If a file
name would collide, suffix with `-journey` (e.g., `project-crud.spec.ts` already exists →
generate `project-crud-journey.spec.ts`).

## Audit Mode

Read `references/audit-rubric.md` for classification criteria.

**Prerequisites:**
- A flow map must exist (auto-discovers if missing)
- Existing test files must exist (error if zero test files found)

**Process:**
1. Load the flow map — build a set of all known business flows
2. Scan all existing test files — extract test names, assertions, and page interactions
3. For each test, classify using the rubric in `references/audit-rubric.md`:
   - **Business-aligned** — test name maps to a known journey, assertions check outcomes
   - **Shallow** — tests DOM state without business context (element visible, text present)
   - **Zombie** — test doesn't map to any known business flow
   - **Redundant** — multiple tests cover the same journey differently
4. For each business flow, check if any test covers it. Unmatched flows → **Missing**
5. Write audit report to `docs/e2e-architect/audit-report.md`

**Never auto-delete or auto-rewrite.** The audit produces a report only. The user decides
what to act on, then can invoke `generate` mode for specific gaps.

## Red Flags

See `references/test-philosophy.md` for the full anti-pattern catalogue with code examples.

## What This Skill Is NOT

- **Not a DOM scraper** — it doesn't look at the page and emit tests for what it sees
- **Not a test runner** — it generates test files, it doesn't execute them
- **Not a replacement for unit tests** — it generates E2E journey tests only
- **Not a PRD writer** — it discovers existing docs, it doesn't create product specs
