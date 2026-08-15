# SKILLS & TOOLING — Prior-Auth Multi-Agent: Production Hardening

> The recommended toolkit, mapped to the phase where it earns its keep — so you know
> not just *what* to use but *when*. Solo + AI-assisted, so several entries are Claude
> Code sub-agent patterns rather than people.

## By phase

| Phase | Skills / tools / MCPs | What it does here |
|-------|-----------------------|-------------------|
| Initialize (done) | this **gsd-method** skill; WebSearch | scoped the effort; verified CMS/MAF facts |
| Discuss | **engineering:architecture** (ADRs), **engineering:system-design** | lock ADR-1..4 (datastore, identity, FHIR, policy) before coding |
| Plan | **gsd-method**; **TaskCreate/TaskUpdate** | decompose each phase into waves; track state |
| Execute — infra | **azd** CLI, **Bicep**, **Kubernetes/Container Apps** knowledge; Claude Code sub-agents in waves | provision Postgres/Blob/VNet; wire Managed Identity (T1.1) |
| Execute — backend | **engineering:testing-strategy**, **engineering:code-review**; Claude Code sub-agent per Wave-2/3 task | repository refactor, recovery semantics, DI (T2.x/T3.x) |
| Execute — data/eval | **xlsx** or **dataviz** for the eval report; Python harness | build labeled fixtures + accuracy/latency report (T0.2/T0.3, R7) |
| Execute — FHIR/policy | Da Vinci PAS IG; agent **SKILL.md** authoring | intake adapter (R5); commercial policy skill (R6) |
| Verify | **engineering:code-review**, **engineering:debug**; pytest; **engineering:deploy-checklist** | acceptance tests (Wave 4); pre-deploy gate |
| Complete | git (atomic commits per task); **engineering:incident-response** (runbook) | archive STATE, tag milestone, write the ops runbook |

## Frameworks / MCP connectors

- **Microsoft Agent Framework 1.0 (GA)** + Foundry Hosted Agents — the agent runtime; already in the repo. Reconcile the repo's "preview" label with current GA SKUs.
- **azd (≥ the repo's pinned `azure.yaml` version)** with the `azure.ai.agents` extension — builds/registers the four agents; don't fork this flow.
- **The 5 Anthropic healthcare MCP servers** (NPI, ICD-10, CMS Coverage, ClinicalTrials, PubMed) — external dependencies; keep the User-Agent workaround; the eval harness (R7) is your early-warning for MCP quality drift.
- **AI-agent-coding accelerant:** since you're solo, run Execute waves as Claude Code **sub-agent waves** — one fresh sub-agent per independent Wave task (e.g. T1.1/T1.2/T1.3 in parallel), each with only its task's context, atomic commit on completion. Keep a single living `STATE.md` as the handoff between phases.

## Gaps

- **No labeled ground-truth dataset ships** with the accelerator — R7's credibility depends on building one (T0.2). If real adjudicated cases aren't available, "accuracy" is agreement-with-intended-label; state that limitation in the eval report.
- **No FHIR tooling installed** — you'll add a FHIR parsing/validation library (e.g. `fhir.resources`) for the R5 adapter; not a skill, a dependency to pull in.
- **HIPAA sign-off is not a tool** — the BAA + formal compliance review (PROJECT success #7) is a human/legal gate outside any skill; schedule it early because Phase 4 blocks on it.
- **Load-testing tool** for N4 (e.g. Locust/k6) isn't in the repo — add one when you reach Phase 5.
