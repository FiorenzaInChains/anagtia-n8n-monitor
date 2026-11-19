# 🐍 Anagtia - Architecture & Design Decisions

This document explains the architectural choices, trade-offs, and evolution of **Anagtia** (Proof of Concept).

## 1. Core Architectural Philosophy
* **Orchestration over Silos:** Embedding monitoring inside n8n allows health checks to integrate directly with ticketing, chatops, and automated remediation without maintaining standalone custom scripts.
* **Collective Retry over In-Loop Blocking:** Waiting 60 seconds inside a per-item loop creates an $O(N \times 60\text{s})$ bottleneck when multiple services fail simultaneously. By separating the *Check Pass* from the *Collective Wait & Retry Pass*, the retry delay remains fixed at ~60 seconds regardless of how many services go down.
* **Unified Digest Alerting (DRY):** Rather than duplicating email nodes for HTTP and SSH branches (which causes alert storms when 50+ services drop at once), all confirmed outages converge into a single `BuildAlertDigest` node that groups failures by `owner_team` and dispatches one formatted HTML summary per team.

## 2. Key Technical Challenges Solved
1. **HTTP Error Short-Circuiting & Missing Status Codes:** By default, n8n's HTTP Request node throws an error on 4xx/5xx responses and omits `statusCode` from `$json`. Configuring `fullResponse: true`, `neverError: true`, and an explicit `10000ms` timeout ensures HTTP 500s and slow endpoints are cleanly evaluated and retried rather than failing silently or hanging the execution.
2. **Loop Payload Preservation (`pairedItem` Safety):** Downstream HTTP and Wait nodes overwrite `$json` and can break `pairedItem` tracking on error branches. Using dedicated normalization nodes (`RecordCheckResult` / `RecordRetryResult`) with `$('ProgramLoop').first().json` (safe for `batchSize: 1`) guarantees service names, targets, and error details are never lost.

## 3. Comparison with Other Approaches
* **vs. Uptime Kuma / UptimeRobot:** Dedicated uptime tools are faster to set up for pure HTTP/TCP ping monitoring, whereas Anagtia trades simplicity for workflow extensibility (custom team matrices, Jira/Slack orchestration, future log enrichment).
* **vs. Custom Python/Bash Scripts:** Visual flows lower the maintenance barrier for mixed IT teams and provide built-in execution logs, retry scheduling, and encrypted credential storage.

## 4. Production Readiness Assessment
Unlike naive PoCs, Anagtia is intentionally documented with clear boundaries before production adoption:
* **Inventory Externalization:** Move `ListProgram` out of the Code node into PostgreSQL/Google Sheets/CMDB so non-developers can edit targets safely without touching JavaScript.
* **Multi-Host Server Checks:** Because n8n's native SSH node only supports one static host credential per node, production deployments should monitor internal servers via lightweight authenticated HTTP healthcheck agents over VPN, or delegate multi-host SSH checks to an Ansible/bastion runner.
* **State Persistence:** Add Redis/PostgreSQL state tracking to prevent hourly alert repetition and enable recovery notifications.

## 5. Future AI Integration (V2 Strategy)
Adding a local LLM (e.g., via Ollama) is planned as an optional V2 module, contingent on two architectural safeguards:
1. **Real Diagnostic Context:** Passing only `"HTTP 500"` to an LLM produces generic hallucinations. V2 must first fetch sanitized error bodies or tail logs (`journalctl`) before prompting the model.
2. **Non-Blocking Alerting:** Incident alerts must never wait on LLM inference latency or fail if Ollama is down. Immediate alerts should dispatch first, with AI root-cause summaries sent as a follow-up or under a strict timeout with fallback.