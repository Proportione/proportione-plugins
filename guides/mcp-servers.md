# MCP Servers — Lessons Learned

We built three MCP servers that connect Claude Code to our internal systems: ticketing, CRM, and BI. Here's what we learned.

## Architecture

All three MCPs follow the same pattern:

```
mcp-server.mjs          # MCP tool definitions (McpServer + zod schemas)
  └── *-client.mjs      # HTTP client for our API
       └── core-api-client.mjs  # Shared auth + fetch (Cloud Run IAM)
```

The MCP servers talk to a single **Core API** (Node.js on Cloud Run, Firestore backend) that handles auth, RBAC, and business logic. The MCPs are thin wrappers that expose API endpoints as Claude Code tools.

## Auth: the hard lesson

**Problem:** Our first implementation relied on the user's active `gcloud` session for auth. When you switch GCP projects or your tokens expire, all three MCPs break silently with "Proxy exited with code 1".

**Root cause:** The `core-api-client.mjs` tries two auth strategies:
1. `google-auth-library` (needs Application Default Credentials)
2. `gcloud run services proxy` (needs correct active account + project + fresh tokens)

Both fail when gcloud state changes.

**Fix:** Use a service account key via `GOOGLE_APPLICATION_CREDENTIALS`:

```bash
# Create SA key
gcloud iam service-accounts keys create ~/.config/gcloud/sa-keys/core-api.json \
  --iam-account=my-sa@my-project.iam.gserviceaccount.com

# Grant Cloud Run Invoker
gcloud run services add-iam-policy-binding my-service \
  --member="serviceAccount:my-sa@my-project.iam.gserviceaccount.com" \
  --role="roles/run.invoker" --region=europe-west1

# Register MCP with env var
claude mcp add ticketing -s user \
  -e GOOGLE_APPLICATION_CREDENTIALS="$HOME/.config/gcloud/sa-keys/core-api.json" \
  -- node /path/to/mcp-server.mjs
```

Now `google-auth-library` succeeds directly — no proxy subprocess, no dependency on user's gcloud state.

**Rule:** Never depend on the user's interactive gcloud session for MCP auth. Use service account keys.

## Design tips

### Keep MCPs thin
The MCP server should only translate between MCP protocol and your API. No business logic in the MCP.

### Use a shared client
If multiple MCPs talk to the same backend, extract the auth logic into a shared module. We have `core-api-client.mjs` used by all three.

### Register at user level, not project level
```bash
claude mcp add ticketing -s user ...  # Works from any directory
```

### Include internal auth headers
Our Core API uses both IAM auth (Cloud Run) and an internal shared secret for service-to-service calls:
```javascript
headers = {
  ...iamHeaders,
  'x-internal-auth': INTERNAL_SECRET,
  'x-internal-user-email': USER_EMAIL,
  'x-internal-user-name': USER_NAME,
};
```

This lets the API know which user is acting, even when the request comes from a service account.

## What we expose

| MCP | Tools | Use case |
|-----|-------|----------|
| ticketing | list, get, reply, update, merge, stats, tags, canned | Reviewing and managing support tickets from Claude Code |
| crm | contacts, companies, deals, stats, 360-view, follow-ups | CRM operations without leaving the terminal |
| bi | GA4, Google Ads, Search Console, billing, SOC health | Quick data lookups during work |
