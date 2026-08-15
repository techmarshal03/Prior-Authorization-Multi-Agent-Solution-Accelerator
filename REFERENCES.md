# REFERENCES — Prior-Auth Multi-Agent: Production Hardening

> Real, current sources the build should lean on. Time-sensitive items (platform
> maturity, regulatory deadlines) were web-checked in August 2026 — re-verify the
> dated ones before relying on them, since regs and Azure previews move.

## Prior art

- **The accelerator itself** — `microsoft/Prior-Authorization-Multi-Agent-Solution-Accelerator`. Its own `docs/production-migration.md`, `docs/extending.md`, and `docs/technical-notes.md` are the primary build references; the migration doc already sketches the PostgreSQL schema and Blob layout Phase 1 implements. Learn from: the stateless-dispatcher pattern, `_safe_run` retry/degradation, skills-based rules. Avoid inheriting: the in-memory store, inline base64 PDFs, Medicare-only scope.
- **Microsoft AutoAuth** (`microsoft/autoauth`) — Azure AI prior-auth accelerator; useful contrast for intake and policy modeling.
- **Anthropic `prior-auth-review-skill`** — the methodology this accelerator adapts (LENIENT policy, MET/NOT_MET/INSUFFICIENT, gate rubric, audit justification). Read it to change the rubric safely.
- **Multi-Agent Custom Automation Engine** (`microsoft/Multi-Agent-Custom-Automation-Engine-Solution-Accelerator`) — orchestration patterns for multi-agent coordination.

## Standards & specs

- **HIPAA / PHI on Azure** — a signed **BAA with Microsoft** is required before real PHI; encryption at rest, RBAC, network isolation are the baseline controls (gates PROJECT success #7, N2/N3/N6).
- **CMS-0057-F (Interoperability & Prior Authorization Final Rule)** — impacted payers must meet PA decision timeframes (**72h expedited / 7 calendar days standard**) and stand up FHIR APIs including a **Prior Authorization API**, with core provisions phasing in through **2026–2027** (PA API compliance dated to Jan 1, 2027). Shapes R5 (FHIR intake) and N4 (timeliness).
- **HL7 Da Vinci PAS (Prior Authorization Support)** — the FHIR implementation guide toward which the R5 adapter should evolve; ADR-3 defers full conformance.
- **Model Context Protocol (MCP)** — spec for the five healthcare data servers the agents call.
- **Microsoft Agent Framework** — the agent runtime; **v1.0 + Hosted Agents reached GA in 2026** (the repo README still says "refreshed preview" — reconcile at build time).

## Key sources

| Source | Why it matters | Link |
|--------|----------------|------|
| CMS-0057-F official rule page | Authoritative deadlines & API scope | https://www.cms.gov/newsroom/fact-sheets/cms-interoperability-and-prior-authorization-final-rule-cms-0057-f |
| CMS-0057-F FHIR API breakdown (Firely) | Which APIs are mandatory vs optional for 2026–2027 | https://fire.ly/blog/cms-0057-f-decoded-must-have-apis-vs-nice-to-have-igs-for-2026-2027/ |
| Microsoft Agent Framework docs | Runtime, SkillsProvider, hosting | https://learn.microsoft.com/en-us/agent-framework/ |
| Agent Framework Hosted Agents GA (InfoQ) | Confirms GA maturity vs repo's "preview" label | https://www.infoq.com/news/2026/08/agent-framework-harness-ga/ |
| Azure HIPAA/HITRUST offering + BAA | PHI compliance baseline | https://learn.microsoft.com/azure/compliance/offerings/offering-hipaa-us |
| Azure Database for PostgreSQL (Flexible Server) | ADR-1 datastore; private endpoint guidance | https://learn.microsoft.com/azure/postgresql/flexible-server/ |
| Anthropic healthcare MCP marketplace | The 5 MCP data servers | https://github.com/anthropics/healthcare |
| CAQH Index (PA volume) | Business-case sizing (~300M PAs/yr) | https://www.caqh.org/insights/caqh-index-report |
| Da Vinci PAS IG | Target for FHIR intake conformance | https://hl7.org/fhir/us/davinci-pas/ |

## Domain notes

- **LENIENT vs STRICT:** default LENIENT emits only APPROVE/PEND — never DENY. A denial has appeal-rights and regulatory implications; keep STRICT off until a policy owner and an appeals workflow exist (Non-goal this milestone). *Source: repo README + Anthropic skill.*
- **Medicare-only coverage today:** the Coverage agent uses CMS LCDs/NCDs; commercial/MA plans differ, which is exactly R6's gap. *Source: repo disclaimers.*
- **PA timeliness is the business clock:** the ~2-minute pipeline claim matters because manual review is 15–20 min and regulated turnaround is tightening under CMS-0057-F. *Sources: AMA 2024 survey, CMS-0057-F.*
- **MCP gateway quirk:** the Cloudflare gateway blocks the default `Python-urllib` User-Agent; the repo injects `User-Agent: claude-code/1.0`. Preserve this or MCP calls fail. *Source: repo README/technical-notes.*
