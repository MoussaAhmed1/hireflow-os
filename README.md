# HireFlow OS

HireFlow OS is a professional, multi-tenant applicant tracking and interview operations platform. It is also a production-style learning project focused on modern React, Next.js, MUI, and Supabase architecture.

> Status: repository foundation. The first milestone establishes the monorepo, quality gates, design system, Supabase local development, and tenant-safe authentication.

## Product scope

The platform gives each organization an isolated workspace for managing:

- jobs, hiring stages, and reusable pipeline templates;
- candidates, applications, notes, tags, and attachments;
- interview scheduling, structured scorecards, and feedback;
- team membership, roles, invitations, and audit history;
- dashboards and recruiting metrics;
- public career pages and candidate applications.

The architecture must preserve a strict distinction between a **candidate** (a person) and an **application** (that person applying to one job).

## Technology

| Area | Choice |
| --- | --- |
| Runtime and package manager | Node.js 22+, pnpm |
| Monorepo | pnpm workspaces, Turborepo |
| Web application | Next.js App Router, React, TypeScript strict |
| UI | MUI and Emotion; no Tailwind or shadcn/ui |
| Backend platform | Supabase Postgres, Auth, Storage, Realtime |
| Validation | Zod at every untrusted boundary |
| Testing | Vitest, Testing Library, Playwright |
| CI | GitHub Actions |

Prefer stable releases. Upgrade dependencies deliberately and commit the lockfile.

## Repository layout

```text
hireflow-os/
├── apps/
│   └── web/                 # Next.js application
├── packages/
│   ├── ui/                  # Shared MUI theme and reusable components
│   ├── config/              # Shared TypeScript/lint configuration
│   └── types/               # Shared domain types when truly cross-package
├── supabase/
│   ├── migrations/          # Reviewed, immutable database migrations
│   ├── seed.sql             # Deterministic local demo data
│   └── config.toml
├── docs/
│   ├── adr/                 # Architecture decision records
│   └── CODEX_QUICKSTART.md  # Compact product context for AI collaboration
├── .github/
│   └── pull_request_template.md
├── AGENTS.md                # Durable instructions for Codex
├── package.json
├── pnpm-workspace.yaml
└── turbo.json
```

The actual workspace may start smaller. Add packages only when a real sharing boundary appears.

## Prerequisites

- Node.js 22 or newer
- pnpm through Corepack
- Docker Engine or Docker Desktop
- Supabase CLI installed as a project dependency
- Git and a GitHub account

```bash
corepack enable
node --version
pnpm --version
docker --version
```

## Local setup

```bash
git clone <repository-url>
cd hireflow-os
pnpm install --frozen-lockfile
pnpm db:start
cp apps/web/.env.example apps/web/.env.local
pnpm db:reset
pnpm dev
```

Run `pnpm db:status` and copy the local Supabase URL and publishable/anonymous key into `apps/web/.env.local`. Never expose a Supabase secret or service-role key through a `NEXT_PUBLIC_*` variable.

Example client-safe variables:

```dotenv
NEXT_PUBLIC_SUPABASE_URL=http://127.0.0.1:54321
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=<local-publishable-key>
```

The exact script names are defined in the root `package.json`. If the repository does not have the aliases below yet, create them before relying on this section.

## Expected commands

| Command | Purpose |
| --- | --- |
| `pnpm dev` | Run development workspaces |
| `pnpm build` | Build all affected workspaces |
| `pnpm lint` | Run static analysis |
| `pnpm typecheck` | Run TypeScript without emitting files |
| `pnpm test` | Run unit and integration tests |
| `pnpm test:e2e` | Run Playwright journeys |
| `pnpm format:check` | Check repository formatting |
| `pnpm db:start` | Start local Supabase services |
| `pnpm db:stop` | Stop local Supabase services |
| `pnpm db:reset` | Recreate the local database from migrations and seed |
| `pnpm db:status` | Show local endpoints and keys |
| `pnpm db:lint` | Run database linting/advisors configured by the project |

## Database workflow

All schema changes are migration-first and reproducible:

1. Start Supabase locally.
2. Create a named migration with the project CLI.
3. Add schema, indexes, grants, RLS policies, and tests together.
4. Reset locally from zero and verify the seed.
5. Review the generated diff; never edit a migration already applied to a shared environment.
6. Push only after the local checks pass.

Every exposed tenant table requires both explicit privileges and Row Level Security. A valid tenant membership predicate is required; `authenticated` by itself is not authorization.

## Multi-tenant security invariants

- Tenant-owned rows include an `organization_id` or inherit it through a relationship that policies can verify.
- Authorization uses trusted database membership and role data, not editable user metadata.
- Cross-tenant reads and writes are denied and covered by tests.
- Browser code never receives service-role or secret credentials.
- Storage object paths include the tenant boundary and have matching policies.
- Privileged database functions are rare, reviewed, and never used as a shortcut around RLS.

## Contribution workflow

The delivery board is:

`Backlog → Todo → In Progress → Review → Done`

For each card:

1. Confirm the user story, acceptance criteria, dependencies, and tenant/security impact.
2. Move it to **In Progress** and create a focused branch such as `feat/HF-012-job-pipeline`.
3. Implement the smallest vertical slice, including loading, empty, error, and permission states.
4. Run relevant checks and perform a self-review.
5. Open a pull request, move the card to **Review**, and include evidence.
6. Merge only after acceptance criteria and quality gates pass; then move it to **Done**.

Use Conventional Commit-style subjects where practical, for example `feat(jobs): add pipeline editor`.

## Definition of done

A task is done only when:

- its acceptance criteria are demonstrably satisfied;
- tenant isolation and authorization were considered and tested when relevant;
- TypeScript, lint, and relevant tests pass;
- responsive, accessible loading/empty/error/success states exist;
- database changes are reproducible from migrations;
- documentation and architecture decisions are updated when behavior changed;
- the pull request explains both **what** changed and **why**.

## Documentation

- [`AGENTS.md`](./AGENTS.md): durable engineering and collaboration rules for Codex.
- [`docs/CODEX_QUICKSTART.md`](./docs/CODEX_QUICKSTART.md): concise product and architecture context.
- [`docs/adr/000-template.md`](./docs/adr/000-template.md): template for important technical decisions.
- Project briefs: copy the English and Arabic briefs supplied with the project into `docs/product/` if desired.

## License

No public license has been selected yet. Add one before distributing or accepting external contributions.
