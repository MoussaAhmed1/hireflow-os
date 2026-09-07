# HireFlow OS — Agent Instructions

These instructions apply to the whole repository. A nested `AGENTS.md` may add stricter, workspace-specific rules. The nearest applicable file wins when instructions conflict.

## Mission

Build HireFlow OS as a professional, multi-tenant hiring and interview operations platform while helping the maintainer understand senior-level engineering decisions. Optimize for a safe, reviewable vertical slice—not for the largest possible code change.

## Read before changing code

1. Read this file.
2. Read `docs/CODEX_QUICKSTART.md` for domain language, scope, and current priorities.
3. Inspect the affected package, its `package.json`, nearby tests, and established patterns.
4. Read a relevant ADR or product brief only when the task touches that decision.

Do not scan the entire repository when the task is already localized.

## Fixed architecture constraints

- Use pnpm workspaces and Turborepo. Do not introduce npm, Yarn, or Bun lockfiles.
- The primary frontend is one Next.js App Router application using React and strict TypeScript.
- Use MUI with Emotion. Do not add Tailwind CSS, shadcn/ui, or a second competing component system.
- Use Supabase for Postgres, Auth, Storage, and Realtime.
- Validate untrusted input at runtime with Zod or the repository's established schema layer.
- Keep tenant isolation enforceable in Postgres; UI checks are usability, not security.
- Prefer a modular monolith. Do not add services, packages, or abstractions without a current boundary.
- Pin dependency versions deliberately and commit `pnpm-lock.yaml`.

If a task appears to require breaking one of these constraints, stop and propose an ADR before implementing it.

## Workspace ownership

- `apps/web`: routes, layouts, server/client boundaries, and feature composition.
- `packages/ui`: owned MUI theme, semantic design tokens, and genuinely reusable UI primitives.
- `packages/config`: shared tool configuration only.
- `packages/types`: types shared by multiple workspaces; feature-local types stay with their feature.
- `supabase`: configuration, immutable migrations, seed data, database tests, and functions.
- `docs/adr`: architectural decisions whose alternatives and consequences matter later.

Follow the repository as it exists. Do not create a listed package only to match this map.

## Task protocol

For every implementation task:

1. Restate the goal, assumptions, and acceptance criteria in compact terms.
2. Inspect the smallest relevant surface and identify the existing convention.
3. Explain the proposed approach and why it fits before making a broad or irreversible change.
4. Implement one coherent vertical slice. Keep unrelated refactors out.
5. Add or update tests at the cheapest layer that gives real confidence.
6. Run focused checks first, then broader checks when the change warrants them.
7. Summarize changed files, tradeoffs, verification evidence, and remaining risks.

When requirements are ambiguous, ask only questions that materially change the implementation. Otherwise state a reasonable assumption and continue.

## React and Next.js rules

- Prefer Server Components. Add `'use client'` only at the smallest interaction boundary.
- Fetch server-owned data on the server; do not add a client data library without a demonstrated need.
- Use Server Actions or Route Handlers according to trust boundary and consumer needs. Both must authenticate, authorize, and validate input.
- Keep URL-worthy state—filters, search, sort, pagination—in search parameters.
- Treat route `params` and `searchParams` according to the installed Next.js API; inspect the current version before coding from memory.
- Stream meaningful route sections with `loading.tsx` or `Suspense` where it improves perceived performance.
- Provide `error.tsx`, not-found, empty, and permission-denied states when the route needs them.
- Do not add `useMemo`, `useCallback`, or `React.memo` by habit. Measure or justify them, especially when React Compiler is enabled.
- Avoid copying server data into client state. Derive values during render where possible.
- Make accessibility part of the component contract: labels, focus behavior, keyboard flow, and semantic HTML.

## MUI rules

- The application owns one theme with semantic tokens for color, spacing, radius, typography, elevation, and states.
- Configure MUI's supported App Router cache integration at the root layout.
- Prefer theme variants and shared components for repeated product patterns; use `sx` for local one-off styling.
- Avoid wrapper components that merely rename a MUI prop.
- Build responsive behavior mobile-first and verify dark/light themes if both are enabled.
- Do not import another icon or styling system when MUI already satisfies the requirement.

## Supabase and multi-tenancy rules

These are security requirements, not suggestions:

- Make schema changes through committed migrations. Never make an untracked dashboard-only production change.
- Enable RLS on every table exposed through the Data API.
- Treat SQL privileges and RLS as separate layers: grant only required operations and add policies for allowed rows.
- Every tenant policy must prove active organization membership. `TO authenticated` alone is insufficient.
- Do not use `raw_user_meta_data` / editable user metadata for authorization. Use database membership and trusted claims where appropriate.
- Do not expose secret or service-role keys to browser code, logs, screenshots, fixtures, or `NEXT_PUBLIC_*` variables.
- `UPDATE` access needs both `USING` and `WITH CHECK` semantics.
- Prefer `security_invoker` views where views expose RLS-protected data.
- Treat `SECURITY DEFINER` functions as exceptional and review their owner, search path, privileges, and tenant checks.
- Storage uploads and upserts need matching object-path conventions and INSERT/SELECT/UPDATE policies.
- Add negative tests proving one organization cannot access another organization's rows or objects.
- Run local reset and database lint/advisor checks after migration changes.

Use the project's installed Supabase CLI. Discover uncertain syntax with `pnpm supabase --help`; do not guess commands or version-specific flags.

## Domain rules

- A candidate represents a person; an application joins a candidate to a job and owns the hiring-stage lifecycle.
- Tenant-owned business data belongs to an organization.
- Use stable identifiers for records; slugs are navigation aids, not authorization boundaries.
- Model roles as explicit capabilities when permissions differ by action.
- Preserve an audit trail for security-sensitive or hiring-decision events.
- Store timestamps in UTC and format them for the user's locale at the presentation boundary.

## Code quality

- TypeScript must remain strict. Avoid `any`; narrow `unknown` at boundaries.
- Prefer explicit domain names over generic `data`, `item`, or `manager` names.
- Keep modules cohesive and functions small enough to test and explain.
- Do not duplicate domain rules between UI and server layers; share a schema or put the authoritative rule at the trusted boundary.
- Never swallow errors. Return safe user feedback and retain useful server-side diagnostic context without leaking secrets or personal data.
- Add comments for non-obvious constraints and reasons, not for syntax.
- Do not introduce dependencies when a small, well-tested local implementation is clearer.
- Do not modify generated files manually.

## Verification

Use scripts present in `package.json`; never invent a command and report it as run.

Minimum evidence by change type:

| Change | Required evidence |
| --- | --- |
| Documentation only | links and formatting checked |
| UI component | typecheck, focused test, responsive/accessibility review |
| Route or workflow | typecheck, focused tests, main success and failure path |
| Database migration | local reset, lint/advisor check, positive and cross-tenant negative tests |
| Auth or authorization | unauthenticated, permitted, forbidden, and cross-tenant cases |
| Shared package | affected package tests plus dependent typecheck/build |

If a check cannot run, say exactly why. Never claim a test passed without running it.

## Git and change safety

- Preserve user changes and inspect `git status` before broad edits.
- Never rewrite history, delete data, force-push, or change shared migrations unless explicitly authorized.
- Do not commit or push unless the user asks.
- Keep branches and pull requests focused on one card or coherent vertical slice.
- Use a Conventional Commit-style subject where practical.
- Never commit `.env*` files containing credentials. Only sanitized `.env.example` files belong in Git.

## Architecture decisions

Create an ADR when a decision is costly to reverse, changes a fixed constraint, affects multiple workspaces, or changes tenancy/security behavior. Use `docs/adr/000-template.md`. Do not create an ADR for routine implementation detail.

## Communication contract

- Use English for code, identifiers, commits, and repository documentation.
- Explain decisions in clear English; an Arabic analogy or clarification is welcome when it improves understanding.
- Lead with the outcome, then explain why, tradeoffs, and the next safe step.
- Teach through the actual change. Avoid long generic tutorials unless requested.
- Surface assumptions and risks early, especially around Supabase and tenant isolation.

## Definition of done

Before calling a task complete, confirm its acceptance criteria, relevant checks, security impact, UI states, migration reproducibility, and documentation impact. A card moves to **Review** only with evidence and to **Done** only after review criteria are satisfied.
