# REQUIREMENTS — Prior-Auth Multi-Agent: Production Hardening

> What "done" means, as statements a stranger could verify. Every requirement has an
> ID and an acceptance test. IDs are referenced by ROADMAP, TASK-PLAN, and the Verify
> phase. This is the full milestone; the current phase (Phase 1) covers R1–R4, N1–N3.

## Functional

| ID | Requirement | Acceptance test |
|----|-------------|-----------------|
| R1 | Reviews, decisions, and audit artifacts persist in a durable store (PostgreSQL or Cosmos DB), replacing the in-memory `_review_store`. | Submit a review, restart the backend container, `GET /api/review/{id}` returns the full review and any attached decision. |
| R2 | A completed review is recoverable after process loss without re-running agents. | Kill the backend mid-pipeline for a case already stored as complete; after restart the stored result is byte-identical to pre-kill; a *partial* run is marked `incomplete`, never silently "done". |
| R3 | Submit / review / override endpoints require authentication (Entra ID) and reject anonymous calls. | `curl` without a valid token to `POST /api/review`, `/api/decision` returns 401/403; with a valid token it succeeds. |
| R4 | Each decision records the acting user's identity and role (RBAC: only a Reviewer role may Accept/Override). | A user without the Reviewer role calling `POST /api/decision` is denied; an accepted decision row contains the user's Entra object id + role; the audit PDF Section 9 shows that identity. |
| R5 | PA requests can be ingested as FHIR resources, not only via the manual form. | Post a FHIR `Claim`/`ServiceRequest`(+`Coverage`,`Patient`) bundle to the intake endpoint; the pipeline runs and produces a review equivalent to the mapped manual submission. |
| R6 | At least one non-Medicare policy set (commercial or MA) is selectable and alters coverage evaluation. | Submit an identical case under "Medicare" vs the new plan; the Coverage/Synthesis `criteria_assessment` differs per the plan's rules; the SKILL.md change required no backend code change. |
| R7 | An evaluation harness scores pipeline output against a labeled case set. | Run the harness over N labeled cases; it emits per-case predicted vs expected recommendation, an accuracy/agreement number, PEND rate, and latency distribution as a report artifact. |
| R8 | Notification letters and audit PDFs are generated and retrievable per decision after restart. | After R1, `GET` the decision's audit PDF and notification letter for a case created before the restart; both download intact. |

## Non-functional

| ID | Requirement | Acceptance test |
|----|-------------|-----------------|
| N1 | No PHI in telemetry or logs. | Run a case with distinctive synthetic PHI (name/DOB/notes); grep App Insights traces + container stdout — zero occurrences of those values. |
| N2 | PHI encrypted at rest; secrets keyless. | Data store shows encryption-at-rest enabled; no API keys/connection strings in repo, image, or env (all access via `DefaultAzureCredential` / Managed Identity). |
| N3 | Network isolation for the data plane. | Backend↔datastore and backend↔Foundry traffic does not traverse the public internet (VNet / private endpoints); a public-internet connection attempt to the datastore is refused. |
| N4 | End-to-end review latency holds under load. | Under a defined concurrent load, p95 review completion ≤ 2 min (matching the accelerator's headline claim) or an explicitly agreed revised threshold. |
| N5 | Recoverable, observable operations. | Health/readiness endpoints gate traffic; `scripts/check_agents.py`-style verification passes post-deploy; a synthetic failure of one agent degrades to a `warning`/`error` status, not a crash. |
| N6 | Auditable data retention. | Audit artifacts have a defined retention policy and are retrievable for the retention window; deletion is logged. |

## Out of scope (this milestone)

Provider-facing portal; claims adjudication/billing; STRICT-mode DENY workflow (config
exists but the denial appeals process is not built); multi-language; automated
regulatory certification; load-testing beyond the N4 threshold definition.

## Open questions / assumptions

- **Datastore choice (R1):** PostgreSQL (accelerator's `production-migration.md` targets it) vs Cosmos DB — resolve in Discuss/ADR-1. *Assumption: PostgreSQL unless a Cosmos-first shop.*
- **Identity model (R3/R4):** Entra ID app roles vs an external IdP/SSO — *assumption: Entra ID app roles, matching the keyless posture.*
- **FHIR scope (R5):** which resources/profile (Da Vinci PAS vs a lighter custom map) — resolve in ADR-3. *Assumption: a minimal internal FHIR→request-dict mapper first, PAS-aligned later.*
- **Policy source (R6):** are real commercial/MA policy documents available, or synthetic stand-ins for the harness? *Assumption: one synthetic-but-realistic commercial ruleset for the milestone.*
- **Labeled data (R7):** does a labeled/adjudicated case set exist, or must it be built from synthetic cases? *This gates R7 quality — flag early.*
- **BAA status:** is a Microsoft BAA already signed for this subscription? If not, real PHI cannot be processed and Phase 4 blocks (see PROJECT success #7).
