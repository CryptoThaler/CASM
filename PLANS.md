# PLANS.md

## ExecPlan: CASM v0.1 Devtools Runtime
### Objective
Ship CLI + deterministic workflow + GitHub connector + policy gate + telemetry cost engine + two templates.

### PR Sequence
- PR-0: bootstrap + contracts
- PR-1: workflow engine + replay
- PR-2: GitHub connector + minimal agents + repo-auditor
- PR-3: policy + HITL gate
- PR-4: telemetry + cost engine + budget enforcement
- PR-5: templates: repo-auditor + pr-sentinel

### Acceptance tests
Each PR must pass:
- pnpm -r lint
- pnpm -r test
- pnpm -r build

### Rollback
Revert last PR; artifacts are additive; no migrations in v0.1.
