# Audit Rubric — Test Classification Criteria

Use this rubric to classify every existing test file when running audit mode.
Read each test's name, assertions, and interactions, then assign exactly one
category per test.

## Classification Categories

### Business-Aligned ✅

**Criteria — ALL must be true:**
1. Test name describes a user goal (not a DOM element)
2. At least one assertion checks a business outcome (data persisted, state changed,
   cross-persona visible, business rule enforced)
3. Test follows a recognizable journey from the flow map

**Action:** Keep. Optionally enhance with stronger assertions.

**Example of a business-aligned test:**
```
test('admin can create a new workspace and it appears in the dashboard')
→ Fills form, submits, navigates to dashboard, verifies workspace listed
→ Maps to journey: "Admin creates workspace"
→ Verdict: Business-aligned ✅
```

### Shallow ⚠️

**Criteria — ANY is true:**
1. Test only asserts element visibility (`toBeVisible`, `toBeEnabled`, `toHaveText`)
   without verifying a business outcome
2. Test name describes a UI element ("should render X", "displays Y")
3. Test navigates to a page and only checks that the page loaded
4. Test checks form field existence without testing form submission flow

**Action:** Rewrite to test the business outcome the element enables.

**Mapping technique:** For each shallow test, ask "what business action does
this element support?" Then rewrite to test that action end-to-end.

| Shallow test | Business action | Rewrite as |
|---|---|---|
| "submit button is visible" | "admin can save project changes" | Test: fill form → submit → verify data persisted |
| "project items render" | "viewer can browse available items" | Test: navigate to public project → verify items with details |
| "error message appears" | "admin sees actionable feedback on invalid input" | Test: submit invalid data → verify specific error → fix → succeed |

### Zombie 💀

**Criteria — ANY is true:**
1. Test doesn't map to any journey in the flow map
2. Test checks purely cosmetic/structural details (logo position, footer text,
   heading hierarchy, color values)
3. Test verifies framework behavior (React renders, router navigates, state updates)
4. Test duplicates a unit test at the E2E level (testing a utility function via UI)

**Action:** Flag for removal. Some zombies may reveal undocumented features — ask the
user before deleting.

**Warning:** A test that looks like a zombie might actually test an undocumented
business flow. Before flagging, check:
- Does the tested behavior appear in the route definitions?
- Is there a corresponding API endpoint?
- Does any user-facing UI depend on this behavior?

If yes to any of these, it might be a flow you missed during discovery. Add it to
the flow map as a candidate journey, don't zombie-flag it.

### Missing ❌

**Criteria:**
A business flow from the flow map has ZERO test coverage.

**How to detect:**
For each journey in the flow map, search existing test files for:
- Test names matching the journey's action verbs
- Page navigations matching the journey's routes
- Assertions matching the journey's success criteria

If no existing test touches the journey, it's missing.

**Action:** Queue for generation. Missing P0 journeys are highest priority.

### Redundant 🔁

**Criteria:**
Multiple tests verify the same business flow from the same persona perspective,
with no meaningful difference in the scenario.

**How to detect:**
- Same journey steps in different test files
- Same assertions with different setup code
- Test files with overlapping names (e.g., `project-create.spec.ts` and `project-crud.spec.ts`
  both testing project creation)

**Action:** Consolidate into the better-structured test. Keep the one with
stronger assertions and better journey coverage.

**Exception:** Two tests covering the same journey with DIFFERENT preconditions
(e.g., "admin creates project" vs "free-plan user hits project limit") are NOT
redundant — they test different scenarios of the same flow.

## Scoring Summary

After classifying all tests, compute:

```
Total tests: N
Business-aligned: X (X/N %)
Shallow:          Y (Y/N %)
Zombie:           Z (Z/N %)
Redundant:        R (R/N %)

Missing P0 flows: A
Missing P1 flows: B
Missing P2 flows: C
Missing P3 flows: D
```

## Report Structure

Write the audit report to `docs/e2e-architect/audit-report.md` using the
format defined in `references/coverage-format.md`.
