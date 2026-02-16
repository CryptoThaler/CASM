# CASM
Cybernetic Agentic Swarm Management

Technical Map: CASM (Cybernetic HITL Swarm Management)
System goal

Operate three business lines (Templates, Sentinel Service, AgentForge SaaS) on a shared swarm runtime with:

Hourly heartbeat (control loop)

Daily diurnal optimization

HITL gating on risk

Full telemetry into economic KPIs

Architecture Diagram (High Level)
flowchart LR
  subgraph Users
    Dev[Developers/Clients]
    Admin[Operator/HITL]
  end

  subgraph Edge
    CF[Cloudflare: Pages/Workers/WAF/DNS]
  end

  subgraph ControlPlane[CASM Control Plane]
    API[API Gateway]
    ORCH[Swarm Orchestrator]
    ROUTER[Model Router]
    POLICY[Policy Engine<br/>(risk + compliance)]
    HITL[HITL Gate<br/>(approve/override)]
  end

  subgraph DataPlane[Execution Plane]
    GH[GitHub Repo/PRs]
    AGENTS[Agent Swarm<br/>(Reviewer/Fixer/Bench/Doc/Redteam)]
    TOOLS[Tooling Runtime<br/>(tests, linters, scanners)]
  end

  subgraph Models
    OAI[OpenAI]
    CLAUDE[Anthropic/Claude]
    KIMI[Kimi]
    HF[Hugging Face<br/>(models/repos)]
  end

  subgraph Telemetry[Telemetry + Economics]
    LOGS[Event Logs]
    METRICS[Metrics Store]
    COST[Cost Engine<br/>(token + tool + infra)]
    KPI[KPI Dashboard<br/>(MRR, margin, churn risk)]
  end

  Dev --> CF --> API --> ORCH
  ORCH --> GH
  ORCH --> AGENTS --> TOOLS
  ROUTER --> OAI
  ROUTER --> CLAUDE
  ROUTER --> KIMI
  AGENTS --> HF

  ORCH --> POLICY --> HITL --> ORCH
  ORCH --> LOGS --> METRICS --> KPI
  ORCH --> COST --> KPI
  Admin --> HITL

Architecture Diagram (1-Hour Heartbeat Control Loop)
sequenceDiagram
  autonumber
  participant Timer as Heartbeat Timer (hourly)
  participant Or as Orchestrator
  participant Pol as Policy Engine
  participant Hitl as HITL Gate
  participant Gh as GitHub
  participant Rt as Model Router
  participant Ms as Models
  participant Tel as Telemetry/Cost

  Timer->>Or: Start cycle (repo set / client set)
  Or->>Gh: Pull latest commits/PR deltas
  Or->>Pol: Compute risk score + required gates
  Pol->>Hitl: If high risk → queue approvals
  alt HITL required
    Hitl-->>Or: Approve/deny/override
  end
  Or->>Rt: Route tasks (review/fix/bench/docs)
  Rt->>Ms: Execute multi-model swarm
  Or->>Gh: Post PR comments / open PR / attach reports
  Or->>Tel: Log tokens, tool-time, infra-time, outcomes
  Tel-->>Or: Return cost + effectiveness metrics

Cost Analysis per Token (and per Outcome)

Pricing changes; the durable way is to compute unit economics from measured usage and plug in current model rates.

1) Core token cost model

Let:

Tin = input tokens

Tout = output tokens

Pin = $/1M input tokens

Pout = $/1M output tokens

**LLM_cost = (Tin/1e6)Pin + (Tout/1e6)Pout

Now add agent overhead:

R = retry factor (e.g., 1.10 for 10% retries)

S = safety margin (e.g., 1.05)

Tool_cost = $ cost of non-LLM tools (CI minutes, scanners, etc.)

Infra_cost = Cloudflare/hosting/runtime amortized per run

Run_cost = (LLM_cost * R * S) + Tool_cost + Infra_cost

2) Cost per PR / per repo audit (what you actually sell)

Let:

Ncalls = number of model calls in the swarm run

Run_cost_i per call

HITL_minutes and Human_rate_per_min

Outcome_cost = Σ Run_cost_i + (HITL_minutes * Human_rate_per_min)

Then margin:

Gross_margin = 1 − (Outcome_cost / Price_charged)

3) Practical telemetry fields to log every call

Store per model call:

model_id, Tin, Tout

latency_s

retries

task_type (review/fix/bench/doc/redteam)

route_reason (policy decision)

outcome score (pass/fail + quality score)

This makes token economics real-time and optimizable.

Repo Structure (production-grade, swarm-first)
casm/
  README.md
  LICENSE
  SECURITY.md
  CODE_OF_CONDUCT.md

  docs/
    architecture/
      overview.md
      diagrams.mmd
      threat-model.md
      data-retention.md
    runbooks/
      heartbeat.md
      incident-response.md
      release.md
    economics/
      unit-economics.md
      pricing-model.md

  packages/
    orchestrator/                 # core swarm scheduler + workflow engine
      src/
        orchestrator.ts
        workflow.ts
        queue.ts
        state.ts
      tests/

    router/                       # model routing + fallbacks
      src/
        router.ts
        policies.ts
        cost-aware-routing.ts

    policy-engine/                # risk scoring, compliance gates, HITL triggers
      src/
        risk.ts
        gates.ts
        rules/
          default.yaml

    agents/                       # swarm agents (task-specific)
      reviewer/
      fixer/
      benchmarker/
      docgen/
      redteam/
      cost-auditor/

    connectors/                   # GitHub, HF, Cloudflare, model providers
      github/
      huggingface/
      cloudflare/
      openai/
      anthropic/
      kimi/

    telemetry/                    # events, metrics, traces, cost engine
      src/
        events.ts
        metrics.ts
        cost-engine.ts
        kpi.ts

    hitl-console/                 # lightweight operator UI / approval queue
      src/
      public/

  templates/                      # monetizable swarm templates
    repo-auditor/
    perf-optimizer/
    compliance-pack/
    security-sentinel/

  services/
    sentinel/                     # subscription “continuous code sentinel”
    agentforge/                   # SaaS control center backend

  infra/
    cloudflare/
      workers/
      pages/
      wrangler.toml
    github/
      workflows/
        ci.yml
        sentinel.yml
        release.yml

  examples/
    demo-repo/
    sample-configs/

  scripts/
    bootstrap.sh
    seed-templates.ts
    cost-report.ts

Week 7 (Q1 2026) Actionable Build Plan (Engineering-first)
Deliverables by end of week

Swarm Orchestrator MVP

workflows: review → fix → test → report → PR

Policy Engine + HITL Gate

risk scoring + approval queue

Telemetry + Cost Engine

token cost per call + per outcome

2 Templates shipped

Repo Auditor

PR Sentinel (continuous)

Minimal acceptance tests

Can run on a target repo and open a PR with:

review summary

patch set

test results

cost report (token + infra estimate)

HITL blocks “high-risk” changes (security/auth/billing files) until approved.

Recommendation: what to do next (no wasted motion)
Step 1 (here, now)

You tell me the one initial vertical to bias defaults (choose one):

SaaS web apps

ML repos

fintech/compliance

devtools

I’ll then lock:

default risk rules

default agent set

default pricing units (per PR / per repo / per month)

Step 2 (Codex app)

Create the repo with the structure above, stub packages, and implement:

orchestrator + GitHub connector + telemetry cost engine

first template: templates/repo-auditor

Codex is explicitly built to work with repos and produce PRs/branches.

If you want, I can also provide the exact “Codex task prompts” to generate this repo cleanly in 2–3 PRs (bootstrap → core runtime → templates).

devtools? Create the next phase as you logically see necessary, give the build direction over the creative controller. You = 90%. Demonstrate the capabilities and decision making

Devtools it is. I’ll bias defaults toward developer productivity + repo governance + cost control (the three things teams will actually pay for repeatedly), and I’ll drive the build direction.

Below is the next phase (beyond the architecture/spec we already laid down): what to build, in what order, with explicit decision logic, acceptance tests, and what “done” looks like.

Next Phase: Devtools CASM v0.1 → v0.3
Primary Objective

Ship a production-credible swarm runtime that can:

run on any GitHub repo,

produce measurable improvements (PRs + reports),

quantify cost/benefit per run,

enforce HITL on risky changes,

convert into paid subscriptions via “Continuous Sentinel”.

Everything else is secondary.

Product Shape for Devtools
Three monetization tracks (kept, but unified)

Templates → “Swarm Modules” (repo-auditor, perf-optimizer, release-notes-bot, etc.)

Services → “Continuous Code Sentinel” subscription (weekly/monthly)

SaaS → AgentForge control plane (later; becomes the UI for the first two)

The wedge (what earns trust fastest)

Repo Auditor Template + Continuous Sentinel
Because it creates tangible artifacts quickly:

PRs with fixes

benchmark deltas

security/quality reports

token + infra cost reports

time saved estimate

That’s your “proof of competence” engine and your sales collateral generator.

Decisions I’m Making (90% Controller Mode)
Decision 1: Build CLI-first, not dashboard-first

Why: devtools buyers adopt via CLI + GitHub Actions before they’ll trust a UI.
Outcome: “works in CI” becomes distribution.

Decision 2: Use a deterministic workflow engine + typed events

Why: swarms fail when you can’t replay/inspect decisions.
Outcome: every run is reproducible, auditable, and cost-attributed.

Decision 3: Default to cost-aware routing

Why: the buyer’s first question in 2026 is “what will this cost me weekly?”
Outcome: routing chooses “cheapest model that meets policy” unless risk requires higher reasoning.

Decision 4: Risk gating is file-path + diff-type + semantic score

Why: fastest reliable HITL without heavy ML.
Outcome: immediate enterprise credibility.

Technical Build Map (What to Implement Next)
Core runtime (must exist before anything sells)
A) casm CLI (single entrypoint)

Commands:

casm run --template repo-auditor --repo owner/name

casm sentinel --repo owner/name --schedule hourly|daily

casm cost --last-run

casm explain --decision <run-id>

B) Workflow engine (deterministic DAG)

Stages (v0.1):

ingest (repo checkout, diff detection)

analyze (review + issue extraction)

plan (patch plan + confidence scores)

apply (patch generation)

validate (tests/linters)

report (PR comment + artifacts)

telemetry (cost + quality metrics)

C) Policy + HITL gate

Default rules:

Block if touching: auth, billing, crypto, infra, secrets, CI, dependency lockfiles

Block if diff includes permission escalation, shell injection patterns, unsafe deserialization patterns

Block if test coverage delta negative (if measurable)

D) Telemetry + Cost Engine

Log per call:

Tin/Tout, model, latency, retries, task
Compute:

per-call cost

per-run cost

cost per accepted PR

cost per issue fixed

cost per % benchmark improvement

This becomes your unit economics and your “why we routed X to model Y” explanation layer.

Repo Structure (Devtools-optimized)

You already have the skeleton; I’m tightening it to accelerate shipping:

casm/
  packages/
    cli/
    core/                # workflow engine + run state + DAG
    policy/              # risk scoring + HITL triggers
    router/              # model selection + fallbacks + budgets
    agents/              # reviewer, fixer, docgen, bench, redteam-lite
    connectors/          # github, models, hf, cloudflare (later)
    telemetry/           # events + cost + kpis
  templates/
    repo-auditor/
    pr-sentinel/
  infra/
    github/workflows/
  docs/
    quickstart.md
    architecture.md
    economics.md

Architecture Diagrams (Devtools Variant)
Control Plane and Data Plane (Devtools-focused)
flowchart TB
  Dev[Dev / CI Trigger] --> CLI[casm CLI or GitHub Action]
  CLI --> Core[Workflow Engine (DAG)]
  Core --> Policy[Policy + Risk Scoring]
  Policy -->|low risk| Router[Cost-aware Model Router]
  Policy -->|high risk| HITL[HITL Queue: approve/deny]

  Router --> Agents[Agents: review/fix/bench/doc]
  Agents --> GH[GitHub: PRs, comments, artifacts]
  Agents --> Tools[Linters/Tests/Bench]
  Agents --> Models[LLMs: OpenAI/Claude/Kimi]

  Core --> Telemetry[Telemetry + Cost Engine]
  Telemetry --> Report[Cost/Quality Report]
  Report --> GH

Hourly Heartbeat (CI-native)
sequenceDiagram
  autonumber
  participant GH as GitHub Action
  participant CLI as casm
  participant W as Workflow
  participant P as Policy
  participant R as Router
  participant A as Agents
  participant T as Telemetry

  GH->>CLI: scheduled run (hourly)
  CLI->>W: start run (template pr-sentinel)
  W->>P: score risk
  alt high risk
    P-->>W: require HITL
  else normal
    W->>R: select models under budget
    R->>A: execute tasks
    A->>T: emit call events + Tin/Tout
    W->>GH: PR/comment + artifacts
  end

Cost Analysis per Token (Implementable Now)
Pricing model (internal)

For each model provider define:

price_in_per_1m

price_out_per_1m

Then compute:

LLM_cost = Tin/1e6 * Pin + Tout/1e6 * Pout

Run-level:

retry_factor default 1.10

safety_margin default 1.05

Run_cost = Σ(LLM_cost * retry_factor * safety_margin) + tool_cost + infra_cost

Devtools-specific metrics (what matters)

$ / PR created

$ / PR merged

$ / issue fixed

$ / 1% performance improvement

$ / 1k LOC audited

This is how you make ROI legible.

Next Phase Delivery Plan (PR-based, Codex-friendly)
PR-0: Bootstrap + Contracts

repo structure

TypeScript tooling

event schema (typed)

template manifest format (YAML)

Done when: pnpm test passes; casm --help works.

PR-1: Core Workflow Engine

deterministic DAG runner

state persistence (local JSON + optional sqlite)

run replay (casm explain --run-id)

Done when: a “noop” template runs end-to-end.

PR-2: Connectors + Minimal Agents

GitHub connector (comment + open PR)

Agent: reviewer → outputs issues.json

Agent: fixer → patch file(s)

Validator: run linters/tests (configurable)

Done when: repo-auditor creates a PR on a public sample repo.

PR-3: Policy + HITL Gate

default risk rules (path-based + diff heuristics)

HITL queue: local “approve file” + GitHub label gate

Done when: a risky diff requires explicit approval to proceed.

PR-4: Telemetry + Cost Engine

per-call logging

cost computation + report artifact

budget enforcement (--budget $)

Done when: run produces cost_report.md and blocks over-budget.

PR-5: Two monetizable templates

templates/repo-auditor

templates/pr-sentinel (hourly heartbeat)

Done when: both ship with docs + demo.

Demonstrating Capabilities and Decision-Making

Here’s the explicit “control logic” the system will use (this is what makes it cybernetic, not just agentic):

Control variables

budget_usd_per_run

risk_threshold

latency_slo

quality_target (tests pass + lint clean + minimal diff)

Observables

Tin/Tout per task

success rate per agent

merge rate (proxy quality)

rework rate (PRs rejected)

drift signals (repeated failures in same categories)

Policy

If risk > threshold → require HITL

If cost forecast > budget → downgrade model or reduce scope

If latency > slo → parallelize or switch models

If quality < target → re-run with higher reasoning model OR escalate HITL

This is a feedback controller. Not vibes.

Answer to “Codex app repo vs ideas here?”

Optimum path:

We’ve now pinned the devtools vertical and locked the next-phase build plan.

Move into Codex to generate the repo as PRs (that’s where it shines: file creation, compilation, tests, PR diffs).

Use this chat for architecture decisions and governance changes, then push them into Codex as tasks.

If you stay here and manually integrate, you’ll spend unnecessary cycles on:

file scaffolding

wiring dependencies

fixing build errors

keeping structure consistent

Codex will collapse that overhead.

What I need from you (minimal) to proceed cleanly

Pick one default runtime target for v0.1 (I’ll adapt either way):

Node/TypeScript (fastest for Cloudflare + devtools ecosystem)

Python (faster for ML-ish tooling)

If you don’t respond, I’ll assume Node/TypeScript because it aligns best with Cloudflare Workers/Pages + GitHub Actions + CLI distribution.

Once confirmed (or assumed), I’ll give you:

the exact Codex task list to generate PR-0 through PR-2 with minimal churn

the template manifest spec

the default policy ruleset v0.1 for devtools repos

Go for Node/TypeScript

Locked: Node/TypeScript as the v0.1 runtime.

Below is the concrete “next phase” package: Codex task plan (PR-by-PR), template manifest spec, and default devtools policy ruleset v0.1. This is designed so Codex can generate a repo that compiles and runs on day 1, while preserving cybernetic transparency (every decision explainable).

Codex Build Direction (PR Sequence)
How to run this in Codex (recommended)

Create an empty repo casm on GitHub, then in Codex:

Work in small PRs

Ensure each PR:

builds (pnpm -r build)

tests (pnpm -r test)

runs a minimal CLI command (pnpm -C packages/cli start -- --help)

PR-0: Bootstrap + Contracts (repo compiles, CLI prints help)

Codex Task Prompt (copy/paste):

Create a Node/TypeScript monorepo named "casm" using pnpm workspaces.
Goals for this PR:
1) Repo structure as described below.
2) A working CLI package @casm/cli with a --help output and commands stubbed: run, sentinel, cost, explain.
3) Shared types package @casm/core defining: RunEvent schema, RunState, TemplateManifest types.
4) Tooling: tsconfig base, eslint, prettier, vitest, build scripts.
5) Add docs: README.md with Quickstart. Add docs/quickstart.md and docs/architecture.md skeleton.
Acceptance:
- `pnpm i`
- `pnpm -r build`
- `pnpm -r test`
- `pnpm -C packages/cli start -- --help` prints commands.
Repo structure:
packages/{cli,core,policy,router,telemetry,agents,connectors}/
templates/{repo-auditor,pr-sentinel}/
infra/github/workflows/ci.yml
docs/{quickstart.md,architecture.md,economics.md}
Use commander for CLI. Use zod for runtime validation of manifests and events.


What this PR proves: repo hygiene + typed contracts + repeatable builds.

PR-1: Deterministic Workflow Engine + Run Replay

Codex Task Prompt:

Implement deterministic workflow execution in @casm/core.
Requirements:
- A DAG/workflow runner that executes named stages in order with explicit inputs/outputs.
- RunState persisted to .casm/runs/<runId>/state.json with event log .casm/runs/<runId>/events.jsonl.
- Each stage emits typed RunEvents (zod-validated).
- Add `casm explain --run-id <id>` that prints: stage outcomes, policy decisions, model routing choices, and cost summary placeholder.
- Add a sample "noop" template that runs ingest->report with dummy events.
Acceptance:
- `casm run --template noop --repo ./examples/demo-repo` produces a run folder and prints run summary.
- `casm explain --run-id <id>` replays and prints deterministically from persisted events.


Design decision: determinism > agent chaos. This is your “cybernetic spine”.

PR-2: GitHub Connector + Minimal Agents (Repo Auditor MVP)

Codex Task Prompt:

Implement GitHub connector and minimal agents to support templates/repo-auditor.
Requirements:
- @casm/connectors/github: authenticate via GITHUB_TOKEN, support:
  - fetching repo metadata
  - creating a PR branch
  - committing changes
  - opening a PR
  - posting PR comments and uploading artifacts (as PR comment with links or embedded summaries)
- @casm/agents:
  - reviewer agent: produces issues.json (lint/test failures, TODOs, obvious smells) using a placeholder LLM interface (do not hardcode providers yet)
  - fixer agent: applies simple deterministic edits (formatting, basic refactors) and creates patch files; LLM-powered patches can be stubbed but keep interface.
  - validator agent: runs npm/pnpm scripts if present (lint/test) safely with timeouts.
- templates/repo-auditor:
  - runs ingest->analyze->apply->validate->report
  - report posts a markdown summary + cost placeholder to GitHub PR if in GitHub mode; otherwise writes local report.
Acceptance:
- Works locally on examples/demo-repo
- In GitHub mode, opens a PR with a report comment.
Security:
- Do not execute arbitrary scripts unless allowlisted in template config.


Decision rationale: ship the “artifact engine” (PRs + reports) early; LLM sophistication can evolve later.

PR-3: Policy + HITL Gate (Devtools default rules)

Codex Task Prompt:

Implement @casm/policy with devtools-focused risk scoring and HITL gating.
Requirements:
- Risk score from:
  - file path sensitivity list
  - diff heuristics (secrets-like strings, shell execution, permission escalation)
  - dependency change detection (package.json, lockfiles)
- Provide default ruleset at packages/policy/rules/default.devtools.yaml
- HITL modes:
  - local: require `casm approve --run-id <id> --reason "..."`
  - GitHub: require a PR label `casm-approved` OR approval comment by an allowlisted user.
- Integrate policy gates into workflow engine.
Acceptance:
- If risky files touched, workflow halts before apply/commit until approved.
- `casm explain` clearly shows why it gated.

PR-4: Telemetry + Cost Engine + Budget Control

Codex Task Prompt:

Implement @casm/telemetry cost engine with budgets and per-call accounting.
Requirements:
- Define ModelPriceRegistry (by provider/model) in config file.
- LLM call wrapper records Tin/Tout, latency, retries, provider/model, task type.
- Cost computed per call and aggregated per run.
- Budget enforcement: template can specify max_usd_per_run; if forecast exceeds budget, router downgrades model or reduces scope; if still exceeds, stop with a clear reason.
- Produce artifacts:
  - cost_report.md (per-stage, per-call)
  - run_summary.json
Acceptance:
- Running repo-auditor generates cost_report.md even if LLM is stubbed (use simulated token counts for now).
- `casm cost --last-run` prints summary and margin placeholder.

PR-5: Two Sellable Templates (Repo Auditor + PR Sentinel)

Codex Task Prompt:

Ship templates/repo-auditor and templates/pr-sentinel as monetizable swarm modules.
Requirements:
- Each template includes:
  - manifest.yaml
  - docs.md
  - example config
  - explicit telemetry fields
- pr-sentinel:
  - runs hourly/daily (GitHub Actions example)
  - comments on new PRs with risk score, suggested fixes, and cost forecast
Acceptance:
- docs show how to run locally and via GitHub Actions.

Template Manifest Spec (v0.1)
File: templates/<name>/manifest.yaml
version: 0.1
template:
  id: repo-auditor
  name: Repo Auditor
  description: "Swarm-based audit that generates a PR with fixes, tests, and a cost report."
  tags: [devtools, audit, pr]

entrypoint:
  workflow: repo_auditor_v1

runtime:
  language: node
  minimum_node: "20"
  sandbox:
    allow_network: false
    allow_commands:
      - "pnpm -s test"
      - "pnpm -s lint"
      - "npm test"
      - "npm run lint"
    command_timeout_seconds: 900

policy:
  ruleset: default.devtools
  hitl:
    mode: github_label   # local | github_label | github_comment
    github:
      required_label: "casm-approved"
      approvers: ["ORG:my-org", "USER:sk"]  # allowlist patterns

routing:
  budget:
    max_usd_per_run: 3.50
  strategy: cost_aware_quality_floor
  quality_floor:
    min_confidence: 0.70
  fallbacks:
    - on: timeout
      action: downgrade_model
    - on: over_budget
      action: reduce_scope

agents:
  reviewer:
    enabled: true
    model_profile: "reasoning_low"
  fixer:
    enabled: true
    model_profile: "code_mid"
  validator:
    enabled: true
  docgen:
    enabled: true
    model_profile: "cheap"

outputs:
  artifacts:
    - report_md: ".casm/artifacts/report.md"
    - cost_report_md: ".casm/artifacts/cost_report.md"
    - issues_json: ".casm/artifacts/issues.json"
  github:
    create_pr: true
    pr_title: "CASM: Repo Audit Fixes"
    pr_body_artifact: ".casm/artifacts/report.md"
    comment_artifact: ".casm/artifacts/cost_report.md"

telemetry:
  emit:
    - run_events
    - cost_events
    - policy_decisions
    - routing_decisions
  redaction:
    secrets: true
    code_snippets_max_chars: 1200

Why this spec works (devtools reality)

It’s auditable (policy + routing decisions are first-class)

It’s safe by default (allowlisted commands only)

It’s budget-governed (prevents runaway spend)

It’s portable (local + GitHub mode)

Default Policy Ruleset v0.1 (Devtools)
File: packages/policy/rules/default.devtools.yaml
version: 0.1
ruleset: default.devtools

risk_thresholds:
  warn: 0.40
  gate: 0.65
  block: 0.85

path_sensitivity:
  high:
    - "**/auth/**"
    - "**/oauth/**"
    - "**/security/**"
    - "**/crypto/**"
    - "**/billing/**"
    - "**/payments/**"
    - "**/infra/**"
    - ".github/workflows/**"
    - "Dockerfile"
    - "docker/**"
    - "**/terraform/**"
    - "**/*.tf"
    - "**/k8s/**"
    - "**/helm/**"
    - "**/.env*"
    - "**/secrets/**"
  medium:
    - "package.json"
    - "pnpm-lock.yaml"
    - "package-lock.json"
    - "yarn.lock"
    - "**/scripts/**"
    - "**/*.sh"

diff_heuristics:
  secret_like_patterns:
    weight: 0.30
    patterns:
      - "AKIA[0-9A-Z]{16}"                 # AWS access key
      - "-----BEGIN (RSA|EC|OPENSSH) KEY"  # private keys
      - "xox[baprs]-"                      # Slack tokens
      - "ghp_[A-Za-z0-9]{36,}"             # GitHub tokens
  command_exec_additions:
    weight: 0.25
    patterns:
      - "child_process\\.exec"
      - "child_process\\.spawn"
      - "execSync\\("
      - "spawnSync\\("
      - "eval\\("
  deserialization_risk:
    weight: 0.20
    patterns:
      - "yaml\\.load\\("
      - "pickle\\.loads\\("
      - "JSON\\.parse\\(.*req\\.body"
  permission_escalation:
    weight: 0.25
    patterns:
      - "chmod\\s+\\+x"
      - "sudo\\s"
      - "setcap\\s"
      - "CAP_NET_ADMIN"
      - "runAsUser:\\s*0"
  dependency_risk:
    weight: 0.20
    files:
      - "package.json"
      - "pnpm-lock.yaml"
      - "package-lock.json"
    signals:
      - "adds_new_dependency: true"
      - "updates_dependency_major: true"

gating:
  require_hitl_if:
    - "risk_score >= risk_thresholds.gate"
    - "touches_any(path_sensitivity.high)"
    - "touches_dependency_files: true"
  block_if:
    - "risk_score >= risk_thresholds.block"
    - "secret_like_detected: true"

actions:
  on_gate:
    - "stop_before_apply"
    - "emit_hitl_request"
  on_block:
    - "stop_immediately"
    - "emit_security_alert"


HITL behavior: policy engine generates a “decision packet” (why gated, what changed, suggested safe scope reduction). Your operator approves with explicit reason; that reason is logged.

“Demonstrate the capabilities and decision making” (Operationally)

Here’s how the system behaves on a typical PR, with explicit control logic:

Example: PR touches .github/workflows/ci.yml and package.json

policy.score() sees high-sensitivity path + dependency changes

risk score jumps above gate

workflow halts before patch application

HITL request created with:

diff summary

risk factors triggered

minimal-safe change recommendation:

“Only comment; do not modify workflow”

“Do not change dependency versions, only add lint script”

If approved:

router selects higher-reasoning model profile for review

patch is constrained to allowlisted file set

This is cybernetic: constraints + feedback + explicit intervention.

What you should do now (exact next move)
In Codex: execute PR-0 using the prompt above.

Once PR-0 merges, execute PR-1, PR-2, etc.

If you want the lowest-friction start, do this immediately:

Codex PR-0 prompt = the one in this message.

Optional: add a “Proof Mode” in v0.1 (I recommend it)

A switch that forces transparency artifacts:

routing_decisions.json

policy_decisions.json

cost_forecast.json

run_replay.sh (re-run with same inputs)

This makes your expertise undeniable because every decision is inspectable.

If you want, I’ll write the exact schemas for those artifacts next (so Codex implements them cleanly).

Create the final schemas for the suggested artifacts. To be clear, it is the recommended 3 .json and .sh files. 

Below are the final, implementation-ready schemas for the four “Proof Mode” artifacts:

policy_decisions.json

routing_decisions.json

cost_forecast.json

run_replay.sh

They’re designed to be:

deterministic (stable field ordering encouraged)

diff-friendly

auditable

safe (no secret material; include redaction markers)

You can implement these as Zod schemas and emit the JSON exactly as specified.

1) policy_decisions.json (Schema v0.1)

Purpose: Explain why the run gated/blocked/warned, with traceable evidence and HITL requirements.

File path

.casm/artifacts/policy_decisions.json

JSON Schema (draft 2020-12)
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://casm.dev/schemas/policy_decisions.v0.1.json",
  "title": "CASM Policy Decisions",
  "type": "object",
  "required": [
    "schema_version",
    "run",
    "ruleset",
    "summary",
    "risk",
    "signals",
    "decisions",
    "hitl",
    "redaction"
  ],
  "properties": {
    "schema_version": { "type": "string", "const": "0.1" },

    "run": {
      "type": "object",
      "required": ["run_id", "template_id", "repo", "started_at", "commit"],
      "properties": {
        "run_id": { "type": "string" },
        "template_id": { "type": "string" },
        "repo": {
          "type": "object",
          "required": ["type", "id"],
          "properties": {
            "type": { "type": "string", "enum": ["local", "github"] },
            "id": { "type": "string", "description": "Local path or owner/name" },
            "url": { "type": "string" }
          },
          "additionalProperties": false
        },
        "started_at": { "type": "string", "format": "date-time" },
        "commit": {
          "type": "object",
          "required": ["base", "head"],
          "properties": {
            "base": { "type": "string" },
            "head": { "type": "string" }
          },
          "additionalProperties": false
        },
        "pull_request": {
          "type": "object",
          "required": ["present"],
          "properties": {
            "present": { "type": "boolean" },
            "number": { "type": "integer" },
            "url": { "type": "string" }
          },
          "additionalProperties": false
        }
      },
      "additionalProperties": false
    },

    "ruleset": {
      "type": "object",
      "required": ["id", "hash", "source"],
      "properties": {
        "id": { "type": "string", "description": "e.g., default.devtools" },
        "hash": { "type": "string", "description": "sha256 of ruleset file contents" },
        "source": { "type": "string", "enum": ["bundled", "template", "override"] }
      },
      "additionalProperties": false
    },

    "summary": {
      "type": "object",
      "required": ["status", "reason_codes"],
      "properties": {
        "status": { "type": "string", "enum": ["pass", "warn", "gate", "block"] },
        "reason_codes": {
          "type": "array",
          "items": { "type": "string" },
          "description": "Short stable codes, e.g. PATH_SENSITIVE_HIGH, DEP_CHANGE, SECRET_LIKE"
        },
        "human_summary": {
          "type": "string",
          "description": "Concise explanation safe for PR comment."
        }
      },
      "additionalProperties": false
    },

    "risk": {
      "type": "object",
      "required": ["score", "thresholds", "components"],
      "properties": {
        "score": { "type": "number", "minimum": 0, "maximum": 1 },
        "thresholds": {
          "type": "object",
          "required": ["warn", "gate", "block"],
          "properties": {
            "warn": { "type": "number" },
            "gate": { "type": "number" },
            "block": { "type": "number" }
          },
          "additionalProperties": false
        },
        "components": {
          "type": "array",
          "items": {
            "type": "object",
            "required": ["name", "weight", "contribution", "evidence_count"],
            "properties": {
              "name": { "type": "string" },
              "weight": { "type": "number", "minimum": 0, "maximum": 1 },
              "contribution": { "type": "number", "minimum": 0, "maximum": 1 },
              "evidence_count": { "type": "integer", "minimum": 0 }
            },
            "additionalProperties": false
          }
        }
      },
      "additionalProperties": false
    },

    "signals": {
      "type": "array",
      "description": "Evidence items that triggered risk components.",
      "items": {
        "type": "object",
        "required": ["signal_id", "type", "severity", "file", "message"],
        "properties": {
          "signal_id": { "type": "string" },
          "type": {
            "type": "string",
            "enum": [
              "path_sensitivity",
              "diff_pattern",
              "dependency_change",
              "command_execution",
              "permission_escalation",
              "secret_like",
              "test_regression",
              "lint_regression",
              "other"
            ]
          },
          "severity": { "type": "string", "enum": ["info", "low", "medium", "high"] },
          "file": { "type": "string" },
          "line_start": { "type": "integer", "minimum": 1 },
          "line_end": { "type": "integer", "minimum": 1 },
          "message": { "type": "string" },
          "evidence": {
            "type": "object",
            "required": ["kind"],
            "properties": {
              "kind": { "type": "string", "enum": ["pattern", "path", "metadata", "diff_hunk"] },
              "pattern_id": { "type": "string" },
              "diff_hunk_sha256": { "type": "string", "description": "Hash of diff snippet (not raw snippet)" }
            },
            "additionalProperties": false
          }
        },
        "additionalProperties": false
      }
    },

    "decisions": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["decision_id", "action", "stage", "status", "reason_codes"],
        "properties": {
          "decision_id": { "type": "string" },
          "stage": { "type": "string", "description": "workflow stage where decision applied" },
          "action": { "type": "string", "enum": ["allow", "warn", "gate", "block", "scope_reduce"] },
          "status": { "type": "string", "enum": ["applied", "pending_hitl", "skipped"] },
          "reason_codes": { "type": "array", "items": { "type": "string" } },
          "scope_reduction": {
            "type": "object",
            "properties": {
              "allow_files": { "type": "array", "items": { "type": "string" } },
              "deny_files": { "type": "array", "items": { "type": "string" } },
              "notes": { "type": "string" }
            },
            "additionalProperties": false
          }
        },
        "additionalProperties": false
      }
    },

    "hitl": {
      "type": "object",
      "required": ["required", "mode"],
      "properties": {
        "required": { "type": "boolean" },
        "mode": { "type": "string", "enum": ["none", "local", "github_label", "github_comment"] },
        "required_approvals": { "type": "integer", "minimum": 1 },
        "approvers": { "type": "array", "items": { "type": "string" } },
        "required_label": { "type": "string" },
        "approval_comment_regex": { "type": "string" },
        "status": {
          "type": "string",
          "enum": ["not_required", "pending", "approved", "denied", "expired"]
        },
        "approval": {
          "type": "object",
          "properties": {
            "approved_by": { "type": "string" },
            "approved_at": { "type": "string", "format": "date-time" },
            "reason": { "type": "string" }
          },
          "additionalProperties": false
        }
      },
      "additionalProperties": false
    },

    "redaction": {
      "type": "object",
      "required": ["secrets", "snippets_included", "notes"],
      "properties": {
        "secrets": { "type": "boolean" },
        "snippets_included": { "type": "boolean" },
        "notes": { "type": "string" }
      },
      "additionalProperties": false
    }
  },
  "additionalProperties": false
}

2) routing_decisions.json (Schema v0.1)

Purpose: Explain model selection per task, with budget/latency/quality constraints and fallbacks.

File path

.casm/artifacts/routing_decisions.json

{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://casm.dev/schemas/routing_decisions.v0.1.json",
  "title": "CASM Routing Decisions",
  "type": "object",
  "required": [
    "schema_version",
    "run",
    "routing_strategy",
    "constraints",
    "tasks",
    "summary"
  ],
  "properties": {
    "schema_version": { "type": "string", "const": "0.1" },

    "run": {
      "type": "object",
      "required": ["run_id", "template_id", "started_at"],
      "properties": {
        "run_id": { "type": "string" },
        "template_id": { "type": "string" },
        "started_at": { "type": "string", "format": "date-time" }
      },
      "additionalProperties": false
    },

    "routing_strategy": {
      "type": "object",
      "required": ["name", "version"],
      "properties": {
        "name": {
          "type": "string",
          "enum": [
            "cost_aware_quality_floor",
            "quality_first",
            "budget_first",
            "latency_first"
          ]
        },
        "version": { "type": "string", "description": "router algorithm version" }
      },
      "additionalProperties": false
    },

    "constraints": {
      "type": "object",
      "required": ["budget", "latency_slo_ms", "quality_floor"],
      "properties": {
        "budget": {
          "type": "object",
          "required": ["max_usd_per_run", "max_usd_per_task_default"],
          "properties": {
            "max_usd_per_run": { "type": "number", "minimum": 0 },
            "max_usd_per_task_default": { "type": "number", "minimum": 0 }
          },
          "additionalProperties": false
        },
        "latency_slo_ms": { "type": "integer", "minimum": 0 },
        "quality_floor": {
          "type": "object",
          "required": ["min_confidence"],
          "properties": {
            "min_confidence": { "type": "number", "minimum": 0, "maximum": 1 }
          },
          "additionalProperties": false
        }
      },
      "additionalProperties": false
    },

    "tasks": {
      "type": "array",
      "items": {
        "type": "object",
        "required": [
          "task_id",
          "task_type",
          "stage",
          "selected",
          "candidates",
          "decision"
        ],
        "properties": {
          "task_id": { "type": "string" },
          "task_type": {
            "type": "string",
            "enum": [
              "review",
              "fix",
              "docgen",
              "benchmark",
              "redteam",
              "cost_audit",
              "other"
            ]
          },
          "stage": { "type": "string" },

          "selected": {
            "type": "object",
            "required": ["provider", "model", "profile"],
            "properties": {
              "provider": { "type": "string", "enum": ["openai", "anthropic", "kimi", "hf", "other"] },
              "model": { "type": "string" },
              "profile": { "type": "string", "description": "e.g. cheap, code_mid, reasoning_low" },
              "estimated_input_tokens": { "type": "integer", "minimum": 0 },
              "estimated_output_tokens": { "type": "integer", "minimum": 0 },
              "estimated_cost_usd": { "type": "number", "minimum": 0 },
              "estimated_latency_ms": { "type": "integer", "minimum": 0 }
            },
            "additionalProperties": false
          },

          "candidates": {
            "type": "array",
            "description": "Ranked list of considered options",
            "items": {
              "type": "object",
              "required": ["provider", "model", "profile", "score"],
              "properties": {
                "provider": { "type": "string" },
                "model": { "type": "string" },
                "profile": { "type": "string" },
                "score": { "type": "number" },
                "reject_reasons": { "type": "array", "items": { "type": "string" } },
                "estimated_cost_usd": { "type": "number", "minimum": 0 },
                "estimated_latency_ms": { "type": "integer", "minimum": 0 },
                "estimated_quality": { "type": "number", "minimum": 0, "maximum": 1 }
              },
              "additionalProperties": false
            }
          },

          "decision": {
            "type": "object",
            "required": ["reason_codes", "constraints_applied"],
            "properties": {
              "reason_codes": { "type": "array", "items": { "type": "string" } },
              "constraints_applied": {
                "type": "object",
                "required": ["budget", "latency", "quality"],
                "properties": {
                  "budget": { "type": "boolean" },
                  "latency": { "type": "boolean" },
                  "quality": { "type": "boolean" }
                },
                "additionalProperties": false
              },
              "fallbacks": {
                "type": "array",
                "items": {
                  "type": "object",
                  "required": ["on", "action"],
                  "properties": {
                    "on": { "type": "string", "enum": ["timeout", "over_budget", "low_quality", "provider_error"] },
                    "action": { "type": "string", "enum": ["downgrade_model", "switch_provider", "reduce_scope", "stop"] }
                  },
                  "additionalProperties": false
                }
              }
            },
            "additionalProperties": false
          }
        },
        "additionalProperties": false
      }
    },

    "summary": {
      "type": "object",
      "required": ["estimated_total_cost_usd", "estimated_total_latency_ms", "notes"],
      "properties": {
        "estimated_total_cost_usd": { "type": "number", "minimum": 0 },
        "estimated_total_latency_ms": { "type": "integer", "minimum": 0 },
        "notes": { "type": "string" }
      },
      "additionalProperties": false
    }
  },
  "additionalProperties": false
}

3) cost_forecast.json (Schema v0.1)

Purpose: Provide pre-run (forecast) and post-run (actual) costs, plus unit economics proxies.

File path

.casm/artifacts/cost_forecast.json

{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://casm.dev/schemas/cost_forecast.v0.1.json",
  "title": "CASM Cost Forecast",
  "type": "object",
  "required": [
    "schema_version",
    "run",
    "prices",
    "forecast",
    "actual",
    "unit_economics",
    "budget",
    "notes"
  ],
  "properties": {
    "schema_version": { "type": "string", "const": "0.1" },

    "run": {
      "type": "object",
      "required": ["run_id", "template_id", "started_at"],
      "properties": {
        "run_id": { "type": "string" },
        "template_id": { "type": "string" },
        "started_at": { "type": "string", "format": "date-time" },
        "completed_at": { "type": "string", "format": "date-time" }
      },
      "additionalProperties": false
    },

    "prices": {
      "type": "object",
      "required": ["registry_version", "currency", "models"],
      "properties": {
        "registry_version": { "type": "string" },
        "currency": { "type": "string", "const": "USD" },
        "models": {
          "type": "array",
          "items": {
            "type": "object",
            "required": ["provider", "model", "price_in_per_1m", "price_out_per_1m"],
            "properties": {
              "provider": { "type": "string" },
              "model": { "type": "string" },
              "price_in_per_1m": { "type": "number", "minimum": 0 },
              "price_out_per_1m": { "type": "number", "minimum": 0 }
            },
            "additionalProperties": false
          }
        }
      },
      "additionalProperties": false
    },

    "forecast": {
      "type": "object",
      "required": ["llm", "tools", "infra", "total_usd", "confidence"],
      "properties": {
        "llm": {
          "type": "object",
          "required": ["input_tokens", "output_tokens", "cost_usd", "by_task"],
          "properties": {
            "input_tokens": { "type": "integer", "minimum": 0 },
            "output_tokens": { "type": "integer", "minimum": 0 },
            "cost_usd": { "type": "number", "minimum": 0 },
            "by_task": {
              "type": "array",
              "items": {
                "type": "object",
                "required": ["task_id", "provider", "model", "input_tokens", "output_tokens", "cost_usd"],
                "properties": {
                  "task_id": { "type": "string" },
                  "provider": { "type": "string" },
                  "model": { "type": "string" },
                  "input_tokens": { "type": "integer", "minimum": 0 },
                  "output_tokens": { "type": "integer", "minimum": 0 },
                  "cost_usd": { "type": "number", "minimum": 0 }
                },
                "additionalProperties": false
              }
            }
          },
          "additionalProperties": false
        },
        "tools": {
          "type": "object",
          "required": ["cost_usd", "details"],
          "properties": {
            "cost_usd": { "type": "number", "minimum": 0 },
            "details": {
              "type": "array",
              "items": {
                "type": "object",
                "required": ["name", "unit", "quantity", "unit_cost_usd", "cost_usd"],
                "properties": {
                  "name": { "type": "string" },
                  "unit": { "type": "string", "enum": ["seconds", "minutes", "runs", "requests"] },
                  "quantity": { "type": "number", "minimum": 0 },
                  "unit_cost_usd": { "type": "number", "minimum": 0 },
                  "cost_usd": { "type": "number", "minimum": 0 }
                },
                "additionalProperties": false
              }
            }
          },
          "additionalProperties": false
        },
        "infra": {
          "type": "object",
          "required": ["cost_usd", "details"],
          "properties": {
            "cost_usd": { "type": "number", "minimum": 0 },
            "details": {
              "type": "array",
              "items": {
                "type": "object",
                "required": ["name", "unit", "quantity", "unit_cost_usd", "cost_usd"],
                "properties": {
                  "name": { "type": "string" },
                  "unit": { "type": "string", "enum": ["seconds", "minutes", "requests", "gb_month"] },
                  "quantity": { "type": "number", "minimum": 0 },
                  "unit_cost_usd": { "type": "number", "minimum": 0 },
                  "cost_usd": { "type": "number", "minimum": 0 }
                },
                "additionalProperties": false
              }
            }
          },
          "additionalProperties": false
        },
        "total_usd": { "type": "number", "minimum": 0 },
        "confidence": { "type": "number", "minimum": 0, "maximum": 1 }
      },
      "additionalProperties": false
    },

    "actual": {
      "type": "object",
      "required": ["present", "llm", "tools", "infra", "total_usd"],
      "properties": {
        "present": { "type": "boolean" },
        "llm": {
          "type": "object",
          "required": ["input_tokens", "output_tokens", "cost_usd", "by_call"],
          "properties": {
            "input_tokens": { "type": "integer", "minimum": 0 },
            "output_tokens": { "type": "integer", "minimum": 0 },
            "cost_usd": { "type": "number", "minimum": 0 },
            "by_call": {
              "type": "array",
              "items": {
                "type": "object",
                "required": ["call_id", "task_id", "provider", "model", "input_tokens", "output_tokens", "cost_usd", "latency_ms"],
                "properties": {
                  "call_id": { "type": "string" },
                  "task_id": { "type": "string" },
                  "provider": { "type": "string" },
                  "model": { "type": "string" },
                  "input_tokens": { "type": "integer", "minimum": 0 },
                  "output_tokens": { "type": "integer", "minimum": 0 },
                  "cost_usd": { "type": "number", "minimum": 0 },
                  "latency_ms": { "type": "integer", "minimum": 0 },
                  "retries": { "type": "integer", "minimum": 0 }
                },
                "additionalProperties": false
              }
            }
          },
          "additionalProperties": false
        },
        "tools": {
          "type": "object",
          "required": ["cost_usd", "details"],
          "properties": {
            "cost_usd": { "type": "number", "minimum": 0 },
            "details": { "type": "array", "items": { "type": "object" } }
          },
          "additionalProperties": true
        },
        "infra": {
          "type": "object",
          "required": ["cost_usd", "details"],
          "properties": {
            "cost_usd": { "type": "number", "minimum": 0 },
            "details": { "type": "array", "items": { "type": "object" } }
          },
          "additionalProperties": true
        },
        "total_usd": { "type": "number", "minimum": 0 }
      },
      "additionalProperties": false
    },

    "unit_economics": {
      "type": "object",
      "required": ["metrics"],
      "properties": {
        "metrics": {
          "type": "array",
          "items": {
            "type": "object",
            "required": ["name", "value", "unit", "basis"],
            "properties": {
              "name": {
                "type": "string",
                "enum": [
                  "cost_per_issue_found",
                  "cost_per_issue_fixed",
                  "cost_per_pr_created",
                  "cost_per_pr_merged",
                  "cost_per_1k_loc_audited"
                ]
              },
              "value": { "type": "number", "minimum": 0 },
              "unit": { "type": "string", "enum": ["USD", "USD_per_unit"] },
              "basis": {
                "type": "object",
                "required": ["count", "description"],
                "properties": {
                  "count": { "type": "number", "minimum": 0 },
                  "description": { "type": "string" }
                },
                "additionalProperties": false
              }
            },
            "additionalProperties": false
          }
        }
      },
      "additionalProperties": false
    },

    "budget": {
      "type": "object",
      "required": ["max_usd_per_run", "status"],
      "properties": {
        "max_usd_per_run": { "type": "number", "minimum": 0 },
        "status": { "type": "string", "enum": ["within_budget", "forecast_over_budget", "actual_over_budget", "blocked"] },
        "enforcement_action": { "type": "string", "enum": ["none", "downgrade_model", "reduce_scope", "stop"] }
      },
      "additionalProperties": false
    },

    "notes": { "type": "string" }
  },
  "additionalProperties": false
}

4) run_replay.sh (Deterministic Re-run Script)

Purpose: One-command reproduction of the run with identical constraints, using local artifacts as inputs where possible.

File path

.casm/artifacts/run_replay.sh

Script contract (requirements)

Must be POSIX shell compatible (/usr/bin/env sh)

Must not leak secrets

Must pin:

template id

repo id

base/head commits (if applicable)

ruleset hash

routing strategy + budget

run id (as a replay label)

Template (generate and fill with run fields)
#!/usr/bin/env sh
set -eu

# CASM Proof Mode replay script
# Safe: does not embed secrets. Requires environment variables for tokens.
#
# Required env vars (set as needed):
#   GITHUB_TOKEN (if replaying in GitHub mode)
#   OPENAI_API_KEY / ANTHROPIC_API_KEY / KIMI_API_KEY (if enabled)
#
# This script replays the run configuration deterministically.
#
# Generated at: {{generated_at}}
# Run ID:       {{run_id}}
# Template:     {{template_id}}
# Repo:         {{repo_type}}:{{repo_id}}
# Base:         {{base_commit}}
# Head:         {{head_commit}}
# Ruleset:      {{ruleset_id}} (hash={{ruleset_hash}})
# Routing:      {{routing_strategy}} (budget={{budget_usd}} USD)

ROOT_DIR="$(cd "$(dirname "$0")/../.." && pwd)"
ART_DIR="$ROOT_DIR/.casm/artifacts"
RUN_ID="{{run_id}}"

echo "== CASM replay =="
echo "root: $ROOT_DIR"
echo "run:  $RUN_ID"

cd "$ROOT_DIR"

# Optional: ensure dependencies installed
if [ ! -d "node_modules" ]; then
  echo "Installing dependencies..."
  pnpm install
fi

# Replay using pinned settings. Avoid network unless explicitly allowed by template.
pnpm -C packages/cli start -- \
  run \
  --template "{{template_id}}" \
  --repo-type "{{repo_type}}" \
  --repo "{{repo_id}}" \
  --base "{{base_commit}}" \
  --head "{{head_commit}}" \
  --ruleset "{{ruleset_id}}" \
  --ruleset-hash "{{ruleset_hash}}" \
  --routing "{{routing_strategy}}" \
  --budget-usd "{{budget_usd}}" \
  --proof-mode true \
  --replay-of "{{run_id}}" \
  --artifacts-dir "$ART_DIR"

echo "Replay complete. Artifacts in: $ART_DIR"

Notes on determinism

To make this meaningful, ensure the runtime:

uses a stable file ordering

includes the ruleset hash check

records the effective config (merged template + overrides)

optionally supports --seed for any randomized logic (default fixed seed)

Implementation Notes (so Codex builds it correctly)

Emit these four artifacts only when --proof-mode true or template.telemetry.proof_mode = true.

Keep any diff snippet content out of these JSONs; use:

line ranges

hashed hunks (sha256)

Use stable IDs:

run_id = ULID recommended (sortable)

signal_id, decision_id, task_id, call_id = ULID or UUIDv7

Codex follow-up task (single PR add-on)

If you want this cleanly integrated, give Codex this after PR-1:

Add Proof Mode artifacts:
- Emit .casm/artifacts/policy_decisions.json, routing_decisions.json, cost_forecast.json, run_replay.sh
- Validate JSON output against zod schemas mirroring the JSON Schema fields.
- Ensure no raw code snippets or secrets are emitted; use diff hunk hashes only.
- Update docs/quickstart.md with Proof Mode usage.
Acceptance:
- `casm run --template noop --repo ./examples/demo-repo --proof-mode true`
  generates all 4 artifacts.


