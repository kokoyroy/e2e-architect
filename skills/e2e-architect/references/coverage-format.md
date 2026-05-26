# Coverage Format — Output Specifications

Format specs for all artifacts produced by e2e-architect.
Use these templates exactly — the consistent structure makes artifacts
machine-parseable and diffable across runs.

## 1. Business Flow Map

**File:** `docs/e2e-architect/business-flow-map.md`
**Produced by:** discover mode
**Consumed by:** generate mode, audit mode

```markdown
# Business Flow Map

Generated: YYYY-MM-DD | Source: [repo name or org/repo]

## Product Summary

[One paragraph describing what the product does, who it's for, and its core
value proposition. Write this as if explaining the product to a new team member.]

## Personas

| Persona | Role | Auth State | Primary Goal | Entry Point |
|---|---|---|---|---|
| [Name] | [system role] | [auth method] | [what they want] | [starting URL] |

## Critical Journeys (P0)

### 1. [Journey title — verb phrase]

**Persona:** [Name] ([state])
**Trigger:** [What starts this journey]
**Steps:**
1. [Action] → [Expected result]
2. [Action] → [Expected result]
...
**Success criteria:** [Observable, testable business outcome]
**Business rules:** [Constraints that apply]
**Failure modes:** [What can go wrong]

### 2. ...

## Important Journeys (P1)

[Same format as P0]

## Secondary Journeys (P2)

[Same format as P0]

## Edge Cases (P3)

[Same format as P0, but steps may be shorter — focus on the constraint being tested]

## Business Rules

| # | Rule | Scope | Enforcement | Testable As |
|---|---|---|---|---|
| BR-1 | [Plain English rule] | [Entity] | [Server/Client/Both] | [How to verify in E2E] |

## Discovery Sources

[List what was scanned and what yielded useful context — helps future re-runs
know what changed since last discovery]

- Requirements docs: [files read]
- Routes: [number discovered]
- Types: [key entities found]
- API endpoints: [number found]
- Database tables: [number found]
- Existing tests: [number found]
- Knowledge graph: [used/not found]
- Live app: [explored/not available]
```

## 2. Coverage Matrix

**File:** `docs/e2e-architect/coverage-matrix.md`
**Produced by:** generate mode (after test generation)
**Consumed by:** humans, CI dashboards

```markdown
# E2E Coverage Matrix

Generated: YYYY-MM-DD | Flow map version: YYYY-MM-DD

## Summary

| Priority | Total Flows | Covered | Missing | Coverage |
|---|---|---|---|---|
| P0 | N | X | Y | X/N% |
| P1 | N | X | Y | X/N% |
| P2 | N | X | Y | X/N% |
| P3 | N | X | Y | X/N% |
| **Total** | **N** | **X** | **Y** | **X/N%** |

## Flow-to-Test Mapping

| # | Business Flow | Priority | Test File | Test Name | Status |
|---|---|---|---|---|---|
| 1 | [Journey title] | P0 | `path/to/spec.ts` | [test name] | ✅ Covered |
| 2 | [Journey title] | P0 | — | — | ❌ Missing |
| 3 | [Journey title] | P1 | `path/to/spec.ts` | [test name] | ✅ Generated |
...

## Business Rules Coverage

| # | Rule | Test File | Test Name | Status |
|---|---|---|---|---|
| BR-1 | [Rule] | `path/to/spec.ts` | [test name] | ✅ Covered |
| BR-2 | [Rule] | — | — | ❌ Missing |
...

## Uncovered Gaps (Action Items)

### Missing P0 Coverage
[List each missing P0 flow with a suggested test approach]

### Missing P1 Coverage
[List each missing P1 flow]
```

## 3. Audit Report

**File:** `docs/e2e-architect/audit-report.md`
**Produced by:** audit mode
**Consumed by:** humans

```markdown
# E2E Test Audit Report

Generated: YYYY-MM-DD | Flow map version: YYYY-MM-DD

## Summary

| Category | Count | Percentage |
|---|---|---|
| Business-aligned ✅ | N | X% |
| Shallow ⚠️ | N | X% |
| Zombie 💀 | N | X% |
| Redundant 🔁 | N | X% |
| **Total tests** | **N** | 100% |

| Gap | Count |
|---|---|
| Missing P0 flows | N |
| Missing P1 flows | N |
| Missing P2 flows | N |
| Missing P3 flows | N |

## Business-Aligned Tests ✅

| File | Test Name | Journey |
|---|---|---|
| `path/to/spec.ts:L` | [test name] | [mapped journey] |

## Shallow Tests ⚠️ (Rewrite Candidates)

| File | Current Test | Problem | Should Validate |
|---|---|---|---|
| `path/to/spec.ts:L` | [test name] | [why it's shallow] | [what it should test instead] |

## Zombie Tests 💀 (Removal Candidates)

| File | Test Name | Reason |
|---|---|---|
| `path/to/spec.ts:L` | [test name] | [why no business flow maps to this] |

## Redundant Tests 🔁 (Consolidation Candidates)

| Group | Tests | Keep | Remove |
|---|---|---|---|
| [Journey name] | `file1:L`, `file2:L` | [which to keep] | [which to remove] |

## Missing Coverage ❌

| Business Flow | Priority | Suggested Spec | Suggested Test Name |
|---|---|---|---|
| [Journey] | P0 | `suggested-file.spec.ts` | [suggested test name as user story] |

## Recommendations

1. **Immediate (P0 gaps):** [Specific actions]
2. **Next sprint (shallow rewrites):** [Specific actions]
3. **Housekeeping (zombie removal):** [Specific actions]
```

## Date Format

All dates use `YYYY-MM-DD` format. Use the current date at generation time.
Include the flow map's generation date in coverage matrix and audit report
headers so readers know if the flow map has drifted.
