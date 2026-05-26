# Discovery Engine — Repo Scanning Strategy

Scan the repo systematically to understand what the SaaS product does.
The goal is NOT to catalog every file — it's to extract enough context
to identify personas, journeys, business rules, and critical paths.

## Scanning Order (priority-ranked)

Scan in this order. Stop early if you have enough context (≥ 3 personas
AND ≥ 5 distinct journeys AND ≥ 3 business rules). Don't over-scan —
every file you read costs context.

### 1. Requirements & Product Docs

```bash
find . -type f \( -name "*.md" -o -name "*.txt" \) | grep -iE "(prd|requirement|spec|feature|epic|story|brief)" | head -20
```

Also check these common locations:
- `docs/`, `specs/`, `requirements/`, `.github/`
- `README.md`, `PRODUCT.md`, `FEATURES.md`

**What to extract:** Product description, user types, feature lists, acceptance criteria,
business rules, success metrics.

### 2. README + CLAUDE.md

Read the root `README.md` and `CLAUDE.md` (if they exist). These often contain:
- Product elevator pitch
- Tech stack
- Getting started instructions (reveals primary user flow)
- Architecture overview

### 3. Route Definitions

Routes are the skeleton of a web app — they reveal every screen a user can visit.

**React Router / TanStack Router:**
```bash
grep -rn "path:" --include="*.tsx" --include="*.ts" | grep -E "(route|Route|path)" | head -30
```
Or find the router config file:
```bash
find . -type f -name "*.tsx" -o -name "*.ts" | xargs grep -l "createBrowserRouter\|createRouter\|Routes" | head -5
```

**Next.js / Remix:**
```bash
find . -path "*/app/*" -name "page.tsx" -o -name "route.tsx" | head -30
```

**Express / Fastify / Hono:**
```bash
grep -rn "app\.\(get\|post\|put\|delete\|patch\)" --include="*.ts" --include="*.js" | head -30
```

**What to extract:** URL patterns → feature mapping. Group routes by prefix:
- `/auth/*` → authentication flows
- `/admin/*` → admin features
- `/api/*` → backend capabilities
- `/dashboard/*` → primary user workspace

### 4. Type Definitions / Domain Models

Types reveal the business entities — the nouns of the product.

```bash
find . -type f -name "*.ts" | xargs grep -lE "^export (type|interface)" | head -20
```

Look specifically for:
- Shared types in `types/`, `shared/`, `libs/`, `models/`
- Database schema types (Drizzle `pgTable`, Prisma `model`, TypeORM `@Entity`)
- API request/response types

**What to extract:** Entity names (User, Workspace, Project, Task, Subscription), their fields,
and relationships between them. These become the vocabulary of your flow map.

### 5. API Endpoints

The backend API reveals what the product CAN DO — every mutation is a business action.

```bash
grep -rn "router\.\(get\|post\|put\|delete\|patch\)" --include="*.ts" --include="*.js" | head -30
```

Or find OpenAPI/Swagger specs:
```bash
find . -type f \( -name "openapi*" -o -name "swagger*" -o -name "api-spec*" \) | head -5
```

**What to extract:** CRUD operations per entity, auth-protected vs public endpoints,
webhook/callback endpoints (reveal integrations).

### 6. Database Schema

Schema reveals the data model — what the business actually persists and cares about.

**Drizzle:**
```bash
find . -type f -name "*.ts" | xargs grep -l "pgTable\|mysqlTable\|sqliteTable" | head -10
```

**Prisma:**
```bash
find . -name "schema.prisma" | head -3
```

**Raw SQL migrations:**
```bash
find . -path "*/migrations/*" -name "*.sql" | tail -10
```

**What to extract:** Table names (business entities), foreign keys (relationships),
constraints (business rules encoded in schema), enum columns (states/statuses).

### 7. Existing Tests

Even bad tests reveal intended behaviour. They tell you what the developer THOUGHT
was important to test.

```bash
find . -type f \( -name "*.spec.ts" -o -name "*.test.ts" -o -name "*.e2e.ts" -o -name "*.spec.js" -o -name "*.test.js" \) | head -30
```

**What to extract:** Test file names (feature mapping), test descriptions (intended
behaviors), fixture/seed data (personas), and page objects (UI surface mapping).

### 8. Knowledge Graph

If `.understand-anything/knowledge-graph.json` exists, it's a pre-built index of
every file, function, class, route, and relationship in the codebase. Query it for:
- Nodes of type `route` → all URL paths
- Nodes of type `table` or `schema` → all database entities
- Edges of type `routes` → which files serve which URLs
- Layer assignments → architectural overview

```bash
test -f .understand-anything/knowledge-graph.json && echo "Knowledge graph available" || echo "No knowledge graph"
```

## Fallback Interview Mode

If after scanning all sources you have fewer than 3 distinct personas OR fewer than
5 identifiable journeys, enter interview mode. Ask these questions one at a time:

1. "What does your product do in one sentence?"
2. "Who are the primary user types? (e.g., admin, member, viewer)"
3. "What are the 3 most important things a user can do?"
4. "Which flows directly affect revenue? (e.g., signup, payment, checkout)"
5. "What would break your business if it stopped working for an hour?"

Use the answers to supplement what you discovered from the codebase.

## Output

Pass all discovered context to the flow inference phase (`references/flow-inference.md`).
Do not write the flow map directly from discovery — the inference phase structures
and prioritizes the raw findings.
