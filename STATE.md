# STATE — Prior-Auth Multi-Agent: Production Hardening

> The living handoff between phases. Update this at the end of each work session so a
> fresh context (you next week, or a fresh Claude Code agent) can resume without
> re-reading the whole conversation. Keep it short and current.

## Current position
- **Milestone:** Production-Ready v1 (Phases 0–5). See ROADMAP.md.
- **Active phase:** Phase 0 → 1 (not started). TASK-PLAN.md is planned to task level for Phase 1.
- **Mode:** general-build, solo + AI-assisted.

## Decisions locked (see ARCHITECTURE.md ADRs)
- ADR-1 PostgreSQL + Blob for durable state.
- ADR-2 Entra ID app roles + Managed Identity (keyless).
- ADR-3 FHIR intake as a thin adapter (PAS deferred).
- ADR-4 Policy breadth via SKILL.md variants.

## Open blockers to resolve early
- [ ] Is a Microsoft **BAA** signed for this subscription? (gates Phase 4 / real PHI)
- [ ] Does a **labeled case set** exist, or synthetic-only? (gates R7 credibility)
- [ ] **GPT-5.4 access** granted + region (East US 2 / Sweden Central) confirmed?
- [ ] Reconcile repo's "Hosted Agents preview" label with current GA SKU.

## Next actions
1. Phase 0: `docker compose up`, run a sample case, capture baseline (T0.1–T0.3).
2. Then Phase 1 Wave 1 (T1.1–T1.3) — can run in parallel.

## Changelog
- <date> — GSD package created (PROJECT, REQUIREMENTS, ROADMAP, ARCHITECTURE, TASK-PLAN, REFERENCES, SKILLS, STATE).
