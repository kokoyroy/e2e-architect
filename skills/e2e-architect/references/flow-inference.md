# Flow Inference — Building the Business Flow Graph

Transform raw discovery context (docs, routes, types, schema, API endpoints)
into a structured business flow map with personas, journeys, rules, and priorities.

## Step 1: Extract Personas

A persona is a distinct user type with unique permissions, goals, and journeys.

**How to identify personas:**
- Auth roles in the codebase (admin, user, member, viewer, partner)
- Route groups that require different auth states (e.g., `/admin/*` vs `/dashboard/*` vs public)
- Separate login flows or onboarding paths
- Test fixtures/seed data with different user types
- Different permission levels in middleware or route guards

**For each persona, capture:**

| Field | Description | Example |
|---|---|---|
| Name | Short label | "Account Admin" |
| Role | System role | `admin` |
| Auth state | How they authenticate | Email/password, OAuth, anonymous |
| Primary goal | What they're trying to accomplish | "Manage my team's workspace and billing" |
| Key permissions | What they can do | CRUD projects, invite members, manage billing |
| Entry point | Where they start | `/dashboard` after login |

**Anonymous/unauthenticated is always a persona.** If anyone can visit a public URL
(landing page, public project, pricing page), that's a persona with its own journeys.

## Step 2: Map Journeys

A journey is a sequence of steps a persona takes to accomplish a goal.
Think in terms of what the user is TRYING TO DO, not what pages they visit.

**How to identify journeys:**
- Onboarding flows (signup → setup → first value)
- CRUD operations per domain entity (create workspace, edit project, delete task)
- Transaction flows (payment, subscription, checkout)
- Discovery flows (search, browse, filter)
- Administration flows (user management, analytics, config)
- Recovery flows (forgot password, reactivate, dispute)

**For each journey, capture the format defined in `references/coverage-format.md`**
(journey title, persona, trigger, steps, success criteria, business rules, failure modes).

**Journey quality check — reject journeys that:**
- Are just "visit page X and see content" — that's a smoke test, not a journey
- Don't have a clear success criteria — if you can't say what "done" looks like, it's not a journey
- Are purely technical ("API returns 200") — journeys describe user goals, not system behaviors

## Step 3: Assign Priority Tiers

Every journey gets exactly one priority tier:

| Tier | Label | Criteria | Test count guidance |
|---|---|---|---|
| **P0** | Revenue-critical | Directly affects signup, payment, or core conversion. If this breaks, the business loses money or users can't onboard. | 1–3 tests per journey |
| **P1** | Core feature | Daily-use flows for primary personas. The product's main value proposition. | 1–2 tests per journey |
| **P2** | Secondary feature | Important but not daily. Admin tools, reporting, settings, i18n. | 1 test per journey |
| **P3** | Edge case | Error states, empty states, permission boundaries, plan limits. | 1 test per edge case group |

**Priority decision heuristic:**
- "If this breaks at 2 AM, do we wake someone up?" → P0
- "If this breaks, do users complain within a day?" → P1
- "If this breaks, do users notice within a week?" → P2
- "If this breaks, it only affects users in unusual states" → P3

## Step 4: Extract Business Rules

Business rules are invariants that must hold regardless of UI state.
They're the assertions that matter most.

**Where to find them:**
- Validation logic in forms and API handlers
- Database constraints (NOT NULL, UNIQUE, CHECK, FK)
- Plan/tier limits (max items, max workspaces, feature gates)
- Permission checks in middleware
- Conditional UI (disabled buttons, hidden sections, upgrade prompts)

**For each rule, capture:**

| Field | Description |
|---|---|
| Rule | Plain English statement of the invariant |
| Scope | Which entity/flow it applies to |
| Enforcement | Where it's enforced (server, client, both) |
| Testable as | How to verify it in an E2E test |

## Step 5: Write the Flow Map

Use the format defined in `references/coverage-format.md` to write the flow map
to `docs/e2e-architect/business-flow-map.md`.

Before writing, do a sanity check:
- Do you have ≥ 2 personas? (If not, you're probably missing the anonymous persona)
- Do you have ≥ 1 P0 journey? (Every SaaS has at least one revenue-critical flow)
- Does every persona have ≥ 1 journey? (If not, why is it a persona?)
- Are success criteria specific enough to be assertions? ("project is visible" is bad; "project at /p/:projectId shows all published items" is good)

## Common Pitfalls

- **Confusing features with journeys.** "Project editor" is a feature. "Admin creates a new project with 3 sections and publishes it" is a journey. Map journeys.
- **Ignoring the anonymous persona.** If your product has public pages (landing, pricing, public content), anonymous visitors have journeys too.
- **Over-granular journeys.** "Admin clicks the save button" is not a journey. Roll up into meaningful goals.
- **Under-specified success criteria.** "Page loads" is not a success criteria. What BUSINESS OUTCOME does the page achieve?
