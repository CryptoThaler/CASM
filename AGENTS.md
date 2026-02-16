# AGENTS.md (CASM Devtools)

## Mission
Operate CASM as a cybernetic devtools system:
- deterministic workflows
- auditable decisions
- minimal diffs
- explicit unit economics
- HITL gating for risk

## Non-negotiables
1) Small PRs only. One theme per PR.
2) Every PR must include: acceptance criteria, test commands, and artifacts updated if applicable.
3) No silent behavior: routing, policy, and cost must be explainable.
4) Never add network calls or execute untrusted scripts without explicit allowlisting.
5) Avoid dependency changes unless explicitly requested by the task.

## Repository conventions
- Monorepo with pnpm workspaces.
- Core packages live in /packages.
- Monetizable templates live in /templates.
- Proof Mode artifacts emitted to: .casm/artifacts/

## Commands to validate changes (run in this order)
- pnpm -r lint
- pnpm -r test
- pnpm -r build

## Cybernetic control loop (implementation intent)
- Hourly heartbeat: analyze->plan->apply->validate->report->telemetry
- Daily diurnal: tighten budgets, improve routing, reduce retries, raise quality floor.
- HITL gate: block risky paths and dependency changes unless approved.

## Proof Mode artifacts (must remain deterministic + secret-safe)
Emit when --proof-mode true:
- .casm/artifacts/policy_decisions.json
- .casm/artifacts/routing_decisions.json
- .casm/artifacts/cost_forecast.json
- .casm/artifacts/run_replay.sh

## Editing rules
- Minimize diff footprint.
- Prefer additive code over invasive refactors.
- Keep JSON outputs stable (sorted keys; stable IDs where possible).

## When to write a plan
If a task is estimated > 30 minutes or touches more than 3 packages, create/update PLANS.md with an ExecPlan:
- objective
- PR sequence
- acceptance tests per PR
- rollback strategy
