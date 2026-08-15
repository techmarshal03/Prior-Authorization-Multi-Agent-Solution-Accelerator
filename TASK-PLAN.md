# TASK-PLAN — Prior-Auth Multi-Agent — Phase 1: Durable State & Recovery

> The next phase decomposed into atomic tasks grouped into dependency-aware waves.
> Atomic = small, independently verifiable, traceable to a requirement. "Plan twice,
> prompt once." Wave 0 is the Phase-0 prerequisite so this is executable from a cold
> start. Covers R1, R2, R8, N5 (+ N2 carried, since the store must stay keyless).

## Waves

Tasks in the same wave have no dependency on each other and can run in parallel.
Each wave completes (and verifies) before the next begins.

### Wave 0 — Prerequisite baseline (Phase 0)
| Task | Description | Requirement | Success criterion | Owner / agent |
|------|-------------|-------------|-------------------|---------------|
| T0.1 | Run the accelerator locally via `docker compose up`; complete one sample review + one Accept and one Override. | (baseline) | All 4 agents return; a review and both decision types render in the UI. | human |
| T0.2 | Assemble a labeled synthetic case fixture (≥15 cases spanning approve/pend, good/poor documentation, valid/invalid codes) as JSON. | R7 seed / N5 | `fixtures/cases.json` exists; each case has inputs + expected recommendation; loadable by a script. | human + sub-agent |
| T0.3 | Capture the "before" numbers: per-case recommendation + wall-clock latency from T0.2 against the running stack. | N4 baseline | `baseline_report.md` with recommendation and latency per case. | script |

### Wave 1 — Foundations (parallel; depend on Wave 0)
| Task | Description | Requirement | Success criterion | Owner / agent |
|------|-------------|-------------|-------------------|---------------|
| T1.1 | Author a Bicep module for Azure Database for PostgreSQL Flexible Server + a Storage account/container for artifacts; wire Managed-Identity access (no passwords). | R1, N2 | `azd up` provisions both; backend MI has data-plane role; no connection string/secret in repo or env. | human + sub-agent |
| T1.2 | Define a `ReviewRepository` interface + Pydantic/SQL schema for `reviews`, `decisions`, `audit_artifacts` (status enum: `complete`/`partial`/`error`). | R1, R2 | Interface file + migration compile; schema review; `status` column present and non-null. | sub-agent |
| T1.3 | Define the PHI-safe (de)serialization contract for a stored review (what persists vs what is redacted in derived telemetry). | R1, N1 | Documented field map; a `to_row()`/`from_row()` round-trips a sample review losslessly. | sub-agent |

### Wave 2 — Implementation (depends on Wave 1)
| Task | Description | Requirement | Success criterion | Owner / agent |
|------|-------------|-------------|-------------------|---------------|
| T2.1 | Implement `PostgresReviewRepository` against T1.2; artifact bytes (audit PDF, letters) go to Blob, DB stores blob URIs. | R1, R8 | Unit tests: store→get round-trips a review; PDF retrievable by URI. | sub-agent |
| T2.2 | Replace `_review_store` + `store_review/get_review/store_decision/list_reviews` in `orchestrator.py` with the repository (dependency-injected; in-memory impl kept for tests). | R1 | `orchestrator.py` holds no module-level dict; existing router calls unchanged in signature; tests green. | human + sub-agent |
| T2.3 | Point `routers/decision.py` and `routers/review.py` at the repository; audit PDF/letter persisted, not returned inline base64. | R8 | `GET` decision returns a blob URI/stream; a decision persists to DB. | sub-agent |

### Wave 3 — Recovery semantics (depends on Wave 2)
| Task | Description | Requirement | Success criterion | Owner / agent |
|------|-------------|-------------|-------------------|---------------|
| T3.1 | Persist review status transitions so a run interrupted mid-pipeline is stored `partial`, never surfaced as `complete`. | R2 | A case killed mid-phase shows `status=partial` on retrieval; a finished case shows `complete`. | sub-agent |
| T3.2 | Add health/readiness gating + post-deploy verification (extend `scripts/check_agents.py`) so traffic only flows when store + agents are reachable. | N5 | Readiness fails when DB is down; `check_agents`-style script passes when all healthy. | sub-agent |

### Wave 4 — Verify (depends on Wave 3)
| Task | Description | Requirement | Success criterion | Owner / agent |
|------|-------------|-------------|-------------------|---------------|
| T4.1 | Restart-survival test: submit → restart backend → retrieve identical completed review. | R1, R2 | Automated test passes; retrieved review == pre-restart. | script |
| T4.2 | Artifact-survival test: retrieve audit PDF + notification letter for a pre-restart case. | R8 | Both download intact after restart. | script |
| T4.3 | Keyless/secret scan: confirm no connection strings or keys entered the repo/image/env. | N2 | Secret scan clean; all store access via `DefaultAzureCredential`. | script |

## Deterministic vs. judgment

- **Script/deterministic:** all of Wave 4, T0.3, the repository CRUD, schema
  migration, secret scan, readiness checks. These get real automated tests.
- **Judgment (light, human-reviewed):** T1.2/T1.3 schema and redaction design (the
  data-model choices), and T2.2's injection refactor. No LLM-agent output is on
  Phase 1's critical path — that arrives with policy/eval in later phases.

## Definition of done for this phase

Submit a review, kill and restart the backend, and retrieve the byte-identical
completed review **and** its audit PDF/letter by `request_id`; a run interrupted
mid-pipeline is retrievable as `partial`, not `complete`; readiness gates on store
health; and no secret or PHI-in-telemetry regression was introduced (N2 clean, N1
contract defined). That satisfies R1, R2, R8, N5 and unblocks Phases 2 and 4.
