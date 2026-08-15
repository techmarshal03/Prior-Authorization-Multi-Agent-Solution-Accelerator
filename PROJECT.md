# PROJECT — Prior-Auth Multi-Agent: Production Hardening

> The one-page north star. If a later decision conflicts with this, this wins (or
> this gets deliberately updated). Keep it to a page.

## Problem

The `microsoft/Prior-Authorization-Multi-Agent-Solution-Accelerator` is an
**AI-assisted triage demo**, not a production system. It ships four Foundry Hosted
Agents (Compliance, Clinical, Coverage, Synthesis) that produce auditable
approve/pend PA recommendations in under two minutes — but it persists reviews in a
**Python in-memory dict** (`backend/app/agents/orchestrator.py:_review_store`), has
**no authentication or RBAC**, takes PA requests only via a hand-filled web form
(no EHR/FHIR intake), evaluates against **Medicare LCDs/NCDs only**, and its own
disclaimers state it is "a proof of concept only… not intended for commercial use."
For a payer that must meet CMS-0057-F prior-authorization timelines and handle real
PHI, that gap is the whole project: turn a convincing POC into a system that can
run a real book of business without losing the accelerator's audit-first design.

## Goal

A deployed prior-authorization review service that a payer can run against **real,
PHI-bearing PA requests** — durable and recoverable state, authenticated
role-scoped access, FHIR-based intake, a policy set beyond Medicare-only, and a
measured accuracy/latency baseline — while preserving the human-in-the-loop gate
and complete audit trail. Concretely: **a clinician can submit or receive a real PA
case, get a draft recommendation with its audit justification, Accept or Override
it, and have every step survive a restart and be attributable to a named user.**

## Users / stakeholders

- **Utilization Review nurses / PA coordinators** — submit and triage cases; the primary daily users.
- **Medical Directors / Clinical Reviewers** — Accept/Override AI drafts; own the clinical decision.
- **Compliance & Privacy officers** — own HIPAA posture, audit retention, and the BAA with Microsoft.
- **Platform / security engineering (you)** — build and operate; owns infra, auth, data stores.
- **Payer IT / EHR integration owners** — own the FHIR/HL7 seam into provider systems.

## Scope

- Durable, recoverable persistence for reviews, decisions, and audit artifacts (replaces the in-memory store).
- AuthN + RBAC so submit / review / override are named, role-gated actions.
- PHI handling: encryption at rest, network isolation, PHI-safe logging/telemetry, signed BAA.
- FHIR-based PA intake (in addition to the existing manual form) so real requests can flow in.
- Policy coverage beyond Medicare LCDs/NCDs — at least one commercial or MA plan ruleset via the skills layer.
- An evaluation harness that scores recommendations against a labeled case set (accuracy, PEND rate, latency).
- Production operations: observability with PHI redaction, health/readiness gating, cost + region controls.

## Non-goals

- **Not** a fully autonomous decisioner — the human Accept/Override gate stays; DENY stays out (LENIENT mode) unless a policy owner explicitly enables STRICT.
- **Not** replacing the four-agent architecture, the gate rubric, or the MCP data sources — we harden them, not rebuild them.
- **Not** building a provider-facing portal or claims/billing adjudication — intake and review only.
- **Not** multi-language — English-only, per the accelerator's stated limits.
- **Not** clearing regulatory/clinical validation for go-live — this delivers the system; clinical sign-off and formal HIPAA audit are owned by Compliance and are gated outside this plan.

## Constraints

- **Platform:** Azure — Microsoft Foundry + Agent Framework (1.0 GA), Container Apps, ACR, App Insights; keyless via `DefaultAzureCredential`. Solo builder, AI-assisted.
- **Region:** must land where **Foundry Hosted Agents** and **GPT-5.4** co-exist (accelerator names East US 2 / Sweden Central); DataZoneStandard for US data residency is East US 2 only.
- **Regulatory:** HIPAA (BAA required before real PHI); CMS-0057-F PA timeliness (72h urgent / 7d standard) and FHIR PA API direction through 2026–2027.
- **Model access:** GPT-5.4 requires a separate Azure access grant; deployment fails without it.
- **Preserve:** keyless auth, audit PDF, SSE progress, skills-based clinical rules, MET/NOT_MET/INSUFFICIENT rubric.

## Success criteria

Checkable by someone who wasn't in the room:

1. Kill and restart the backend mid-run; a completed review and its decision are still retrievable by `request_id` (no in-memory-only state).
2. An unauthenticated call to submit/review/override is rejected; each persisted decision carries the acting user's identity and role.
3. A PA request delivered as a FHIR resource produces the same review pipeline output as the equivalent manual form submission.
4. At least one non-Medicare policy set is selectable and changes the Coverage/Synthesis criteria for a matching case.
5. The evaluation harness runs a labeled case set and emits accuracy, PEND rate, and p95 end-to-end latency; p95 stays under an agreed threshold (target ≤ 2 min).
6. No PHI (patient name/DOB/clinical notes) appears in Application Insights traces or container logs.
7. A signed Microsoft BAA and encryption-at-rest are in place before any real PHI is processed.

## Mode

**general-build (solo, hands-on), with AI-agent-coding techniques as an accelerant.**
The deliverable is a codebase, so ARCHITECTURE carries repo/module layout and the
LLM-vs-script boundary and TASK-PLAN is written as machine-checkable specs — but
one person executes it, AI-assisted, so we compress ceremony (one living STATE note,
not per-role process) and recommend Claude Code sub-agent waves where they save time.
