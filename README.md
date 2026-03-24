# CASM — Cybernetic Agentic Swarm Management

A CLI-first devtools runtime that audits repos, opens PRs with fixes, and quantifies cost per outcome — with human-in-the-loop gating on risky changes.

## What It Does

- **Repo Auditor**: Runs a swarm of agents (reviewer, fixer, benchmarker, docgen) against any GitHub repo and opens a PR with fixes + a cost report.
- **PR Sentinel**: Continuous monitoring (hourly/daily via GitHub Actions) that comments on new PRs with risk scores, suggested fixes, and cost forecasts.
- **Proof Mode**: Every routing, policy, and cost decision is emitted as inspectable JSON artifacts for full auditability.

## Quick Start

```bash
# Install
pnpm install

# Audit a repo
casm run --template repo-auditor --repo owner/name

# Continuous sentinel
casm sentinel --repo owner/name --schedule daily

# View cost of last run
casm cost --last-run

# Replay a run deterministically
casm explain --run-id <id>
```

## Architecture

```
CLI → Workflow Engine (DAG) → Policy + Risk Scoring
                                  ↓
                         Low risk → Cost-aware Model Router → Agents → GitHub PR
                         High risk → HITL Queue (approve/deny)
                                  ↓
                            Telemetry + Cost Engine → Report
```

**Control loop**: Hourly heartbeat runs `ingest → analyze → plan → apply → validate → report → telemetry`. Policy gates block risky changes (auth, billing, secrets, CI, lockfiles) until a human approves.

## Repo Structure

```
packages/
  cli/            # CLI entrypoint (commander + zod)
  core/           # Workflow engine, state, DAG runner
  policy/         # Risk scoring, HITL triggers, default rulesets
  router/         # Cost-aware model selection + fallbacks
  agents/         # reviewer, fixer, benchmarker, docgen, redteam
  connectors/     # GitHub, model providers, HuggingFace
  telemetry/      # Events, metrics, cost engine, KPIs
templates/
  repo-auditor/   # One-shot audit → PR with fixes + cost report
  pr-sentinel/    # Continuous monitoring via GitHub Actions
skills/
  pr-hygiene/     # PR quality checks
  proof-mode/     # Deterministic artifact generation
```

## Proof Mode Artifacts

When `--proof-mode true`, each run emits:

| Artifact | Purpose |
|----------|---------|
| `policy_decisions.json` | Why the run gated/blocked, with risk scores and evidence |
| `routing_decisions.json` | Model selection per task, with budget/quality constraints |
| `cost_forecast.json` | Pre-run forecast + post-run actual costs + unit economics |
| `run_replay.sh` | One-command deterministic re-run with pinned config |

## Key Design Decisions

1. **CLI-first** — devtools buyers adopt via CLI + CI before trusting a UI
2. **Deterministic workflows** — every run is reproducible and auditable
3. **Cost-aware routing** — cheapest model that meets the quality floor
4. **Risk gating** — file-path sensitivity + diff heuristics + HITL approval
5. **Budget enforcement** — runs stop or downgrade if they exceed `max_usd_per_run`

## Development

```bash
pnpm -r lint
pnpm -r test
pnpm -r build
```

See [PLANS.md](PLANS.md) for the PR-by-PR build roadmap and [AGENTS.md](AGENTS.md) for contributor conventions.

## License

MIT — see [LICENSE](LICENSE).
