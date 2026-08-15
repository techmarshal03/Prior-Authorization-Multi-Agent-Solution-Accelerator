# ROADMAP — Prior-Auth Multi-Agent: Production Hardening

> The use case broken into ordered phases. Each phase is a shippable increment with a
> demoable outcome — not an internal milestone no one can see. Phase 1 is planned to
> task level in TASK-PLAN.md; later phases are re-planned when reached (GSD loops).

## Phases

| # | Phase | Outcome (what's demoable) | Requirements covered | Depends on |
|---|-------|---------------------------|----------------------|------------|
| 0 | Deploy & baseline | The accelerator runs (`docker compose` locally, then `azd up`); a labeled synthetic case set exists; current accuracy/latency captured as the "before" number. | (baseline for R7/N4) | — |
| 1 | Durable state & recovery | Reviews/decisions/audit artifacts live in PostgreSQL; restart-survivable; partial runs marked incomplete. Demo: submit → restart → retrieve. | R1, R2, R8; N5 | Phase 0 |
| 2 | Identity & access | Entra auth on all mutating endpoints; Reviewer-only Accept/Override; decisions attributed to a named user in DB + audit PDF §9. Demo: anon rejected, override signed. | R3, R4 | Phase 1 |
| 3 | FHIR intake & policy breadth | FHIR bundle → same pipeline output; a selectable non-Medicare policy set changes coverage criteria. Demo: FHIR case + plan toggle. | R5, R6 | Phase 1 |
| 4 | PHI hardening & isolation | PHI-safe telemetry/logs; encryption at rest; VNet/private endpoints; BAA in place. Demo: PHI grep = 0 hits; datastore unreachable from public internet. | N1, N2, N3, N6 | Phases 1–2 |
| 5 | Evaluation & load | Harness reports accuracy/PEND/latency over the labeled set; p95 ≤ target under load. Demo: one-command eval report + load result. | R7, N4 | Phases 0, 3 |

## Sequencing notes

- **Phase 0 first, always.** You cannot harden what you cannot run, and every later
  claim (accuracy, latency, "unchanged behavior") is measured against the Phase 0
  baseline. Building the labeled case set here also de-risks R7's biggest unknown.
- **Phase 1 is the linchpin** — durable state is the foundation identity (2), PHI
  retention (4), and evaluation history (5) all lean on. Do it before anything else
  structural.
- **Phases 2 and 3 are independent** of each other (auth vs intake/policy) and can
  interleave; both only need Phase 1's persistence. If forced to choose, do Phase 2
  before Phase 4 (PHI hardening assumes attributable access).
- **Phase 4 gates real PHI.** Nothing before Phase 4 completes may process real
  patient data — Phases 0–3 run on synthetic cases only.
- **Phase 5 closes the loop** with the number that justifies the whole effort.

## Milestone definition

**Milestone "Production-Ready v1"** = Phases 0–5 complete. Exit criteria: all of
PROJECT.md's seven success criteria pass their Verify tests, **and** the Compliance
owner has signed off on the HIPAA posture (BAA + N1/N2/N3/N6) — the one gate outside
this build's control. Shipping = the service can accept a real, authenticated,
FHIR-delivered PA case, produce a human-reviewable draft with audit trail, and
retain it durably and privately.
