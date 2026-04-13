# n8n Revenue Automation — Production Workflow Library
**myAutoBots.AI | Sammy Samet, Principal Technologist**

> A production-tested library of n8n workflows for B2B revenue automation. Each workflow is battle-tested, includes a Mermaid architecture diagram, full node-by-node documentation, and integration notes.

📅 [Book a free 30-min revenue diagnostic](https://calendly.com/ssam8005/30min) · 🌐 [myautobots.ai](https://myautobots.ai) · 🚀 [Neural-GTM Sprint](https://github.com/ssam8005/neural-gtm-sprint)

---

## What's In This Library

| # | Workflow | Use Case | Complexity |
|---|---|---|---|
| 01 | [Lead Enrichment Waterfall](workflows/01-lead-enrichment-waterfall.md) | Auto-enrich new leads via Clay multi-provider cascade | ⚡⚡ Medium |
| 02 | [CRM Sync & Write-Back](workflows/02-crm-sync-write-back.md) | Bi-directional HubSpot/Salesforce sync with enrichment data | ⚡⚡ Medium |
| 03 | [HITL Approval Gate](workflows/03-hitl-approval-gate.md) | Human-in-the-loop review for high-stakes automated actions | ⚡⚡⚡ Advanced |
| 04 | [Outbound Sequence Trigger](workflows/04-outbound-sequence-trigger.md) | Auto-enroll scored leads in tiered email sequences | ⚡⚡ Medium |
| 05 | [Intent Scoring Pipeline](workflows/05-intent-scoring-pipeline.md) | Score and tier leads 0–100 using enrichment + AI reasoning | ⚡⚡⚡ Advanced |

---

## Architecture Overview

All workflows operate as coordinated components within a single orchestration layer:

```mermaid
graph TD
    Webhook[Webhook / Schedule Trigger] --> W01[01: Lead Enrichment Waterfall]
    W01 --> W05[05: Intent Scoring Pipeline]
    W05 --> W04[04: Outbound Sequence Trigger]
    W05 --> W03[03: HITL Approval Gate]
    W03 --> W02[02: CRM Sync & Write-Back]
    W04 --> W02

    External1[Clay Waterfall API] --> W01
    External2[HubSpot / Salesforce] <--> W02
    External3[Email Platform
Instantly / Lemlist] <--> W04
    External4[AI Scoring Agent
Claude + Pinecone] <--> W05
    External5[Slack / Telegram] <--> W03

    style W03 fill:#1a2e0a,color:#c8f0a0,stroke:#4a7a2d
    style W05 fill:#0d2137,color:#e0e0e0,stroke:#5ba3f5
```

---

## Stack Requirements

- **n8n** v1.x+ (self-hosted or cloud)
- **Clay** account with API access
- **HubSpot** or **Salesforce** with API credentials
- **Pinecone** index (for workflow 05 AI scoring)
- **Slack** or **Telegram** bot (for HITL notifications)
- **Python** 3.10+ (for AI agent webhook receiver — see [agentic-lead-scoring](https://github.com/ssam8005/agentic-lead-scoring))

---

## Workflow JSON Exports

Production workflow JSON exports (importable directly into n8n) are planned for a future release. Each JSON will be sanitized — credentials removed, replaced with credential references.

**Roadmap:**
- [ ] `01-lead-enrichment-waterfall.json`
- [ ] `02-crm-sync-write-back.json`
- [ ] `03-hitl-approval-gate.json`
- [ ] `04-outbound-sequence-trigger.json`
- [ ] `05-intent-scoring-pipeline.json`

---

## Related Repos

- [neural-gtm-sprint](https://github.com/ssam8005/neural-gtm-sprint) — Full methodology, SOW, case studies
- [agentic-lead-scoring](https://github.com/ssam8005/agentic-lead-scoring) — Python AI scoring agent called by workflow 05

---

*Built by [Sammy Samet](https://linkedin.com/in/ssamet) — Principal Technologist, [myAutoBots.AI](https://myautobots.ai)*
