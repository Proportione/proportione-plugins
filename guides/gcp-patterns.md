# GCP Patterns We Use Daily

Patterns from running microservices on GCP with a small team and Claude Code.

## Architecture

```
Cloud Run (europe-west1)
  ├── proportione-core-api      # Firestore, RBAC, Vertex AI, Cloud Tasks
  ├── proportione-ticketing     # soporte.proportione.com (React SPA)
  ├── proportione-crm-dashboard # crm.proportione.com
  ├── proportione-bi            # BI reports + SOC dashboard
  ├── proportione-bi-dashboard  # Dashboard frontend
  └── proportione-gmail-ingester # Gmail → Firestore pipeline

Firestore        # Primary database (tickets, contacts, deals, audit logs)
BigQuery         # Billing data, analytics
Cloud Tasks      # Async email, webhooks (outbox pattern)
Vertex AI        # Embeddings, classification
Cloud Storage    # Attachments, exports
```

## Cost optimization: from 63 to 30 EUR/month

We cut our GCP bill by 50% in one session:

1. **VM → Cloud Run**: Hispanidad app was on a VM. Moved to Cloud Run + Cloud SQL. Stopped the VM.
2. **Deleted duplicate services**: Found Cloud Run services with "-esw1" suffix that were copies. Deleted.
3. **Removed unused load balancer**: CRM was behind a classic HTTPS LB. Changed DNS to point directly to Cloud Run, deleted the LB.
4. **Right-sized Cloud SQL**: Matched tier to actual usage.
5. **OpenClaw**: Switched briefing model from Claude to Gemini 2.0 Flash (much cheaper for routine summarization).

**Validation**: Ran 48 E2E tests after every change. All passed.

## Cloud Run patterns

### Min instances = 0
For non-critical services, set `--min-instances=0`. You get cold starts (~2s) but pay nothing when idle. For the core API, we use `--min-instances=1` to avoid cold starts on the critical path.

### Concurrency
Set `--concurrency=80` (default) for Node.js services. Firestore client handles connection pooling internally.

### Service-to-service auth
Use IAM with `roles/run.invoker` on the calling service account. The called service validates the OIDC token automatically.

## BigQuery billing

We export billing data from two GCP organizations into BigQuery:

```sql
-- Proportione
SELECT * FROM `automation-brain.proportione_billing_data.gcp_billing_export_v1_01D81C_66B379_BD61F3`

-- IITD
SELECT * FROM `iitd-apps-script.iitd_billing_data.gcp_billing_export_v1_013537_F7C96D_C2277D`
```

Both are visualized in Looker Studio with scorecards, donut charts, and cost-by-service tables.

## Firestore tips

### Date format
Store dates as Firestore Timestamps, not strings. When reading via API, they come as ISO 8601 strings — parse with `new Date()`.

### Querying
Firestore doesn't support joins. Denormalize data and use subcollections. For our CRM, a contact's timeline is a subcollection under the contact document.

### Backup
Use Firestore scheduled exports to Cloud Storage. We run daily exports to a dedicated bucket.

## Monitoring

We built a SOC dashboard in Google Sheets that tracks:
- GCP costs (daily/monthly)
- Service health (Cloud Run latency, error rates)
- WordPress uptime
- SSL certificate expiration
- Backup status

The BI MCP server (`/bi_soc_health`) lets Claude check this dashboard from the terminal.
