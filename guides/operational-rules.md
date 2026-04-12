# Operational Rules for Claude Code in Automation Pipelines

When Claude Code generates artifacts that other systems execute (Pabbly, Stackby, Cloud Tasks), you need governance rules. These are ours.

## The core constraint

Claude generates rules, scripts, and mappings. **Pabbly executes them.** Claude does NOT execute directly in production.

## Non-negotiable principles

### 1. Idempotence
Same input must always produce the same output with no side effects. Generate stable idempotency keys.

```
BAD:  Create contact every time webhook fires
GOOD: Lookup by idempotency_key first, skip if exists
```

### 2. Determinism
Conditions must be deterministic under the same data state. No randomness, no time-dependent logic unless explicitly required.

### 3. No assumptions about execution order
Pabbly retries up to 5 times with backoff. Your logic must handle being executed multiple times in any order.

### 4. Lookup before create/update
Always check if the record exists before creating or updating. Implement duplicate detection via `idempotency_key` or natural keys.

### 5. Explicit decision-making on duplicates
If duplicates are found, decide explicitly: ERROR or SKIP. Never silently create a second record.

## What Claude CANNOT do

- Execute in production without a PR
- Modify Pabbly workflows directly
- Invent database fields that don't exist
- Assume state persistence between executions
- Use global variables
- Skip validation

## What Claude CAN do

- Generate rules and mappings for Pabbly to execute
- Write scripts that will be reviewed before deployment
- Analyze logs and suggest fixes
- Create test data for validation
- Draft automation configurations for human review

## Firestore patterns (Core API)

Our Core API uses Firestore as primary database. Key patterns:

### Auth model
- External: OAuth Bearer token + RBAC (admin, crm_agent, support_agent, viewer)
- Internal (service-to-service): header `x-internal-auth` with shared secret
- Rate limiting built-in

### Outbox pattern with Cloud Tasks
For async operations (email sending, webhook delivery), we use the outbox pattern:
1. Write the task to Firestore
2. Enqueue a Cloud Task pointing to the processing endpoint
3. The Cloud Task calls the endpoint, which reads from Firestore and executes
4. On success, mark as processed; on failure, Cloud Tasks retries

### Audit logging
Every mutation writes an audit log entry to Firestore. This is non-negotiable for compliance.

## Google Ads rules (Ad Grants specific)

When Claude generates Google Ads campaigns for Ad Grants accounts:

- Max bid: $329/day (Ad Grants limit)
- Bidding: Maximize Conversions only
- Min 2 ad groups per campaign
- Quality Score >= 3 (or pause the keyword)
- CTR >= 5% (or lose the grant)
- CSV formats: UTF-8 BOM comma-separated for web upload, UTF-16LE tab-delimited for Google Ads Editor
