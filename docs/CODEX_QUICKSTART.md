# Codex Quickstart — HireFlow OS

This is the shortest useful map of the project. Read it after the root `AGENTS.md`, then inspect only the files relevant to the current card.

## One-sentence product

HireFlow OS is a multi-tenant ATS that lets organizations publish jobs, manage candidates and applications through configurable pipelines, schedule interviews, collect structured feedback, and understand hiring performance.

## Why this project exists

It has two goals:

1. Deliver a portfolio-quality SaaS product that could support freelance or real client work.
2. Practice senior frontend thinking: product boundaries, tenant-safe data design, server/client architecture, reusable UI, testing, accessibility, observability, and explicit tradeoffs.

The maintainer is experienced with frontend development and React, but is learning Supabase. Explain database and security decisions without watering them down.

## Primary actors

| Actor | Main needs |
| --- | --- |
| Organization owner | configure workspace, billing-ready settings, roles, audit visibility |
| Recruiter | create jobs, manage pipelines and applications, communicate and report |
| Hiring manager | review assigned jobs/candidates and make decisions |
| Interviewer | see interview context and submit a scorecard independently |
| Candidate | browse jobs and submit an application with minimal friction |

## Core domain language

- **Organization:** the tenant and security boundary.
- **Membership:** a user's role/capabilities inside one organization.
- **Job:** a role being recruited for.
- **Stage:** a reusable or job-specific step such as Applied, Screen, Interview, Offer, Hired, Rejected.
- **Candidate:** a person, deduplicated within an organization according to an explicit policy.
- **Application:** a candidate applying to a specific job; this record moves through stages.
- **Interview:** a scheduled evaluation attached to an application.
- **Scorecard:** structured criteria and feedback submitted by an interviewer.
- **Activity/Audit event:** an append-oriented record of an important domain or security action.

Never collapse candidate and application into one record.

## Product boundaries

### First professional release

- authentication and organization onboarding;
- organization switcher, invitations, and role-aware navigation;
- jobs and configurable pipelines;
- candidate directory and application detail/activity;
- Kanban-style application movement with safe concurrency behavior;
- interview scheduling and scorecards;
- public career page and application form;
- resume/document storage;
- search, filters, pagination, loading/empty/error/forbidden states;
- useful recruiting dashboard metrics;
- tenant security tests, audit events, CI, and deployment documentation.

### Deliberately later

- payroll or full HRIS features;
- video calling infrastructure;
- native mobile applications;
- complex billing implementation;
- AI ranking that affects hiring decisions without an explicit fairness, privacy, and human-review design;
- microservices introduced only for perceived scale.

## Architecture at a glance

| Layer | Responsibility |
| --- | --- |
| Next.js Server Components | authenticated reads, initial composition, streaming boundaries |
| Client Components | focused interaction: drag/drop, dialogs, local form experience |
| Server Actions / Route Handlers | validated mutations and external/API boundaries |
| MUI package/theme | consistent tokens, primitives, accessibility, product patterns |
| Supabase Auth | identity/session, not final authorization |
| Postgres + RLS | authoritative data model and tenant authorization |
| Supabase Storage | tenant-scoped resumes and attachments |
| Realtime | narrow collaborative updates where product value justifies complexity |

Default architecture: a modular monolith in one monorepo. Add packages when two consumers need the boundary, not before.

## Tenant model

Conceptual baseline:

```text
auth.users
  └─ organization_memberships ─ organization
                                  ├─ jobs ─ job_stages
                                  ├─ candidates
                                  └─ applications ─ interviews ─ scorecards
```

Most tenant-owned rows carry `organization_id`. If it is derived through a parent, policies must still verify the tenant relationship efficiently. Role checks come from trusted membership data. The current organization selected in the UI is never sufficient proof of authorization.

Every relevant feature needs at least these cases:

- anonymous user denied;
- member with capability allowed;
- member without capability denied;
- member of organization A denied access to organization B;
- malformed or stale input rejected safely.

## Likely capability model

Start with roles for product clarity, but authorize meaningful actions:

| Capability | Owner/Admin | Recruiter | Hiring manager | Interviewer |
| --- | ---: | ---: | ---: | ---: |
| Manage organization and members | Yes | No | No | No |
| Manage jobs and pipelines | Yes | Yes | Assigned/limited | No |
| Manage candidates/applications | Yes | Yes | Assigned/limited | No |
| View assigned interview context | Yes | Yes | Yes | Yes |
| Submit own scorecard | Yes | Yes | Yes | Yes |
| View all completed feedback | Yes | Yes | Policy-dependent | Policy-dependent |

This is a working hypothesis, not a final contract. Record material permission changes in an ADR and test them in RLS.

## Current milestone: repository foundation

The monorepo is being prepared for its first professional push. Complete these in order:

1. Add `README.md`, `AGENTS.md`, this quickstart, ADR template, and pull-request template.
2. Verify workspace scripts, pinned package manager, engines, ignore rules, and sanitized environment examples.
3. Add CI for install, lint, typecheck, tests, and build.
4. Establish the MUI theme, providers, typography, semantic tokens, and app shell.
5. Add Supabase CLI locally, create reproducible migrations/seeds, and document reset workflow.
6. Implement authentication plus organization onboarding as the first tenant-safe vertical slice.

Update this section after each milestone so Codex does not optimize for stale priorities.

## Board workflow

`Backlog → Todo → In Progress → Review → Done`

Only start a card when it contains:

- a user story;
- business value;
- acceptance criteria;
- dependencies and non-goals;
- UI states and responsive/accessibility expectations;
- data model/API impact;
- tenant/role rules;
- test evidence required.

Recommended card format:

```markdown
## User story
As a [role], I want [capability], so that [outcome].

## Context
[Why now, affected workflow, relevant links]

## Acceptance criteria
- Given ..., when ..., then ...

## Scope / non-goals
- In: ...
- Out: ...

## Technical notes
- Existing patterns to reuse: ...
- Data and RLS impact: ...
- Loading/empty/error/forbidden states: ...

## Verification
- Automated: ...
- Manual: ...
```

## High-signal prompt for Codex

Paste the card, then use this shape:

```text
Goal:
Implement [one vertical slice].

Context:
Card: [ID/title]
Relevant area: [paths if known]
User story and acceptance criteria: [paste]

Constraints:
Follow AGENTS.md and CODEX_QUICKSTART.md.
Preserve MUI, Next.js App Router, Supabase, and tenant-security rules.
Keep the change focused. Explain any architecture choice and why.

Done when:
[specific checks, observable UI behavior, RLS cases, documentation]

Before editing:
Inspect the relevant code and propose the smallest implementation plan.
Ask only if a missing choice materially changes the result.
```

For review, ask Codex to find issues first and prioritize correctness, tenant leaks, authorization gaps, regressions, and missing tests—not to summarize the code.

## Decision rules

- Use an ADR for expensive-to-reverse or cross-cutting choices.
- Prefer a feature-local implementation until reuse is proven.
- Prefer server rendering until interaction requires a client boundary.
- Prefer database-enforced authorization to application-only checks.
- Prefer an explicit capability and negative test to a broad role assumption.
- Prefer a small completed workflow to several half-built layers.
- Realtime must solve a visible collaboration problem and include reconnection/conflict behavior.

## Open decisions to resolve through cards or ADRs

- organization URL strategy: path slug, subdomain, or custom domain later;
- invitation lifecycle and expiry;
- precise role/capability matrix;
- candidate deduplication and merge policy;
- interview feedback visibility before all interviewers submit;
- optimistic concurrency strategy for pipeline moves;
- retention/deletion policy for resumes and candidate personal data;
- first production region and data-residency constraints;
- analytics definitions such as time-to-hire and conversion rate.

Do not silently decide these while implementing an unrelated UI task.

## Professional review lens

Before handing off a feature, ask:

1. Is the domain model correct, or is the UI hiding a confused model?
2. Can one tenant or role access data it should not?
3. What happens on slow network, empty data, stale data, duplicate submission, and failure?
4. Is the server/client boundary justified?
5. Is the behavior accessible by keyboard and understandable without color alone?
6. Is the test at the correct layer and does it prove the risky part?
7. Could another developer understand the decision and safely change it later?

## Keep this file useful

Keep it compact and current. Product narrative belongs in the project brief; durable engineering rules belong in `AGENTS.md`; resolved architecture choices belong in ADRs. This file is the operational bridge between them.
