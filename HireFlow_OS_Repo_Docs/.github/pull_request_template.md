## Summary

<!-- What changed? Keep this specific and user-facing where possible. -->

## Why

<!-- What problem or user story does this solve? Link the board card. -->

- Card:
- ADR/spec:

## Change type

- [ ] Feature
- [ ] Bug fix
- [ ] Refactor with no intended behavior change
- [ ] Database/migration
- [ ] Documentation/tooling

## Evidence

<!-- Screenshots/video for UI, command results for code, and before/after when useful. -->

### Checks run

- [ ] Typecheck
- [ ] Lint
- [ ] Relevant unit/integration tests
- [ ] Relevant Playwright journey
- [ ] Build
- [ ] Supabase local reset and database checks, if applicable

List exact commands and results:

```text

```

## Tenant, auth, and data review

- [ ] No tenant/auth/data behavior changed
- [ ] Anonymous access was considered
- [ ] Permitted and forbidden roles were tested
- [ ] Cross-tenant access was tested negatively
- [ ] RLS and explicit grants were reviewed
- [ ] No secret/service-role key or personal data is exposed
- [ ] Migration, seed, storage, audit, and retention impacts were addressed

Explain relevant choices or mark this section not applicable:

## UI quality

- [ ] Loading, empty, error, success, and forbidden states are handled
- [ ] Keyboard/focus behavior and accessible names were checked
- [ ] Responsive behavior was checked
- [ ] MUI theme/tokens and established patterns were reused

## Reviewer guide

<!-- Point reviewers to the risky or important files and describe how to exercise the change. -->

1.
2.

## Risks, tradeoffs, and follow-ups

- Risk:
- Rollback:
- Follow-up cards:

## Final checklist

- [ ] Acceptance criteria are met
- [ ] The change is focused; unrelated refactors are excluded
- [ ] Tests cover the riskiest behavior, not only the happy path
- [ ] Documentation and ADRs are updated where needed
- [ ] No credentials or local-only files are committed
