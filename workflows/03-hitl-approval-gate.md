# Workflow 03: HITL Approval Gate
**n8n Revenue Automation Library | myAutoBots.AI**

Human-in-the-loop approval workflow for high-stakes automated actions. Stages AI-generated content (emails, deal updates, sequence changes) for rep review before execution. Timeout escalation and full audit trail included.

---

## Flow Diagram

```mermaid
sequenceDiagram
    participant Source as Upstream Workflow
(01 / 04 / 05)
    participant n8n as n8n HITL Gate
    participant Notify as Slack / Telegram
    participant Rep as Sales Rep
    participant CRM as CRM / Email Platform

    Source->>n8n: Staged action payload
(type · content · lead_id · confidence)
    n8n->>n8n: Create approval record
(status: pending)
    n8n->>Notify: Send preview notification
(lead name · action · content preview · approve/reject links)

    Note over Rep: 4-hour approval window

    alt Rep approves
        Rep->>n8n: GET /approve?token=XYZ
        n8n->>CRM: Execute staged action
        n8n->>Source: Approval signal (learning feedback)
        n8n->>n8n: Log: approved · latency · rep_id
    else Rep edits and approves
        Rep->>n8n: POST /approve?token=XYZ with edited content
        n8n->>CRM: Execute with edited content
        n8n->>Source: Edit feedback signal
        n8n->>n8n: Log: approved-with-edit · diff stored
    else Rep rejects
        Rep->>n8n: GET /reject?token=XYZ with optional reason
        n8n->>Source: Rejection signal + reason
        n8n->>n8n: Log: rejected · reason · rep_id
    else Timeout (4 hours)
        n8n->>Notify: Escalation reminder
        n8n->>n8n: Log: timeout · no action taken
        Note over CRM: No automated action — lead stays in queue
    end
```

---

## Node Reference

| # | Node | Type | Purpose |
|---|---|---|---|
| 1 | Webhook Receiver | Webhook | Receives staged action from upstream workflows |
| 2 | Create Approval Record | Airtable / Supabase | Stores pending action with unique token |
| 3 | Build Notification | Function | Formats Slack/Telegram message with preview + action URLs |
| 4 | Send Notification | Slack / Telegram | Sends to rep channel or DM based on lead ownership |
| 5 | Approval Webhook | Webhook | `/approve` endpoint — rep clicks to approve |
| 6 | Edit Handler | Webhook | `/approve` with body — accepts edited content |
| 7 | Rejection Webhook | Webhook | `/reject` endpoint — rep clicks to reject |
| 8 | Timeout Check | Schedule | Runs every 30 min — checks for actions pending >4 hours |
| 9 | Execute Action | HTTP Request / HubSpot / Salesforce | Executes the staged action on approval |
| 10 | Feedback Signal | HTTP Request | Notifies upstream workflow of approval/rejection for model improvement |
| 11 | Audit Logger | Function | Logs all outcomes with timestamp, rep, latency, and diff |

---

## Action Types Supported

| Action Type | Trigger Source | Default Behavior |
|---|---|---|
| Outbound email draft | Workflow 05 (AI personalization) | Stage for rep approval |
| Sequence enrollment | Workflow 04 | Auto-approve if score >85 |
| Deal stage change | Workflow 02 | Stage for rep approval |
| CRM field override | Workflow 02 (conflict) | Auto-approve if enrichment confidence >90% |
| Unsubscribe / suppression | Any | Always require rep approval |

---

## Notification Format (Slack/Telegram)

```
🔔 HITL Review Required — [Lead Name] at [Company]

Action: Outbound email — Personalized opener
Score: 87/100 (HOT) | Confidence: 91%
Lead context: VP Sales, funded $4.2M Series A (2 months ago)

Preview:
"Congrats on the Series A — most VP Sales leaders I talk to at
this stage are staring at a CRM full of data and a pipeline that
doesn't move fast enough. Worth a quick chat?"

✅ Approve → [link]  ✏️ Edit → [link]  ❌ Reject → [link]
Expires in: 4 hours
```

---

## Configuration

```json
{
  "approval_window_hours": 4,
  "escalation_reminder_hours": 3,
  "auto_approve_threshold": 85,
  "notification_channel": "slack",
  "slack_channel_id": "{{ $env.SLACK_HITL_CHANNEL }}",
  "telegram_chat_id": "{{ $env.TELEGRAM_CHAT_ID }}",
  "audit_store": "airtable"
}
```

---

*Part of the [Neural-GTM Sprint](https://github.com/ssam8005/neural-gtm-sprint) methodology.*
