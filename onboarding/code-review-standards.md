# Code Review Standards

## Principles
- Safety first, clarity over cleverness, small PRs, tests accompany behavior changes.

## Must Haves
- Builds green; tests pass
- Adequate test coverage for new/changed logic
- No secrets committed; config via env/secret manager
- Logs and metrics for critical paths
- Backward compatible or migration plan

## Reviewer Checklist
- Correctness and defensive coding
- Test adequacy and determinism
- Security (input validation, authz/authn, secrets)
- Performance and allocations in hot paths
- Observability (logs/metrics/traces)
- Failure modes and rollback plan
- Docs updated where needed

## PR Hygiene
- Clear title and description; linked ticket
- Diff is focused; unnecessary changes avoided
- Commits organized; squash on merge

## Approvals
- Minimum: 1 buddy + 1 TL (or senior) for first PRs
- Thereafter: follow repo CODEOWNERS

## Owner
- Senior Engineers; TL maintains 