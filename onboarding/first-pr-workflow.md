# First PR Workflow

## Goal
Time to first PR ≤ 3 days; Time to first deploy ≤ 10 days.

## Steps
1. Pick a scoped ticket (size ≤ 1 day). Confirm acceptance criteria with your buddy.
2. Create a branch: `feature/<ticket-id>-<short-title>`.
3. Make minimal, safe changes; prefer additive.
4. Run unit tests locally; ensure build passes.
5. Write a clear PR description using the template below.
6. Open PR, assign reviewers (buddy + TL), and add labels.
7. Address review feedback promptly; keep commits small.
8. Merge strategy: squash on green; link ticket.
9. Verify in staging; document validation steps.

## PR Template
- Summary: what/why
- Ticket link:
- Scope: components/services touched
- Screenshots (if UI):
- Test coverage: added/updated tests
- Risk: low/medium/high; rollback plan
- Validation: steps to verify locally/staging

## CI Expectations
- All checks required: build, unit tests, lint
- No flaky tests ignored without issue reference

## Review Checklist (For Reviewers)
- Correctness, tests, security, logging/observability
- Config/secrets handled per policy
- Backward compatibility and migration plan
- Docs updated (README/runbook if needed)

## Post-Merge
- Staging deploy verification steps
- Create follow-up tasks if needed

## Owners
- Default owner: Platform TL
- Maintainers: Senior Engineers 