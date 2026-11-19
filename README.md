# 🐍 Project Anagtia - Service Monitor (n8n PoC)

This is a Proof of Concept (PoC) project demonstrating automated, dual-protocol service monitoring and incident routing built on **n8n**. The workflow checks the health of both **HTTP APIs** and **SSH server services**, filters transient glitches using a collective retry pass, and delivers team-routed HTML digest alerts.

> **Why "Anagtia"?**
> Named after the ancient Samnite goddess of healing and protection (whose sacred animal was the snake), *Anagtia* acts as the silent guardian of your IT infrastructure—monitoring services, filtering noise, and notifying the right teams before minor issues turn into prolonged outages.

## 💡 Why this project?
While standalone tools like Uptime Kuma or UptimeRobot are great for basic uptime checks, they operate as closed silos. **Anagtia** models monitoring as a **modular orchestration workflow** inside n8n, keeping health checks in the same platform used for ticketing, chatops, and automated remediation.

Key architectural highlights:
* **Dual-Protocol Routing (HTTP + SSH):** Uses a Switch node (`RouteCheckType`) to check both web endpoints (HTTP status codes) and server-level services (SSH commands, mocked in this PoC).
* **Collective Anti-Flapping Retry:** Instead of pausing 60 seconds inside a sequential loop for every single failed host, all targets are evaluated first, and a **single collective 60-second wait** (`Wait60Sec`) runs only for failed items before re-checking them in `RetryLoop`.
* **Team-Based Digest Alerting (Anti-Alert Storm):** Confirmed failures are grouped by `owner_team` (or explicit `recipient`) inside `BuildAlertDigest`, sending **one unified HTML summary table per team** instead of flooding inboxes during a broad outage.
* **Resilient Result Normalization:** `CheckHTTP` enables `fullResponse: true`, `neverError: true`, and a `10000ms` timeout so HTTP 4xx/5xx responses and slow endpoints are captured cleanly without skipping retry logic, while `RecordCheckResult` preserves original metadata across error branches.
* **Self-Hosted & Portable:** Built with standard n8n nodes so you can download the [JSON workflow](workflows/anagtia_v1_workflow.json), import it, and test both HTTP and SSH flows immediately.

## 🎯 Target Audience
* **Teams adopting n8n:** Organizations consolidating internal IT automations and lightweight monitoring into a single self-hosted tool.
* **SMBs & Mixed Infrastructures:** Teams needing a visual, low-code monitor covering both HTTP APIs and internal server services without complex custom scripts.

## 🛠️ How it works
1. **HourlyCheck (Schedule Trigger):** Runs every hour 24/7 (`0 0 * * * *`).
2. **ListProgram (Code):** Provides the mock service inventory with `program_name`, `type`, `target`, `owner_team`, and `severity`.
3. **Initial Check Pass (`ProgramLoop` → `RouteCheckType` → `RecordCheckResult`):** Evaluates all HTTP and SSH (mock) targets quickly with no in-loop sleep, normalizing every outcome into a unified schema (`is_up`, `status_code`, `error_detail`).
4. **FilterFailed & Collective Wait:** If all services are healthy, the run finishes cleanly (`AllOk`). If any failed, `Wait60Sec` pauses once for 60 seconds.
5. **Retry Pass (`RetryLoop` → `RouteRetryType` → `RecordRetryResult`):** Re-checks only the failed HTTP/SSH services.
6. **FilterStillDown → BuildAlertDigest → SendUnifiedAlert:** Filters out recovered services (`RecoveredAfterRetry`), groups confirmed outages by team routing matrix, and dispatches one HTML digest email per team.

## ⚙️ Configuration Example (`ListProgram` Node)

In this PoC, monitored services are defined in `ListProgram` so the workflow works out-of-the-box upon import. For production, swap this node with Google Sheets, PostgreSQL, or an external JSON inventory.

*Note: While `SendUnifiedAlert` uses Email in this PoC, `BuildAlertDigest` can easily be wired to Slack, Microsoft Teams, or Telegram nodes in parallel.*

```javascript
return [
  {
    json: {
      program_name: "Company Website",
      type: "http",
      target: "https://www.google.com",
      owner_team: "web-ops",
      severity: "warning"
    }
  },
  {
    json: {
      program_name: "Management API",
      type: "http",
      target: "https://httpbin.org/status/500",
      owner_team: "backend",
      severity: "critical"
    }
  },
  {
    json: {
      program_name: "Local Database Service",
      type: "ssh",
      target: "systemctl is-active postgresql",
      owner_team: "dba",
      severity: "critical",
      recipient: "oncall-dba@example.it" // Optional: overrides default team routing
    }
  }
];
```

## 📌 Transition to Production
Currently, SSH checks and SMTP credentials are mocked for safe local testing. To adapt this PoC to a real environment:
- **SSH Checks:**
  - *Single server:* Replace `CheckSSH(Mock)` and `CheckSSH(RetryMock)` with n8n's native SSH node (`Execute Command`) and check `code === 0` inside `RecordCheckResult` / `RecordRetryResult`.
  - *Multi-host fleet:* Because n8n's native SSH node binds to one static credential per node, use an `Execute Command` bastion/Ansible runner with dedicated non-root monitoring keys, or expose authenticated HTTP healthcheck agents over your internal VPN.
- **Inventory:** Replace `ListProgram` with an external source (PostgreSQL, Google Sheets, or CMDB API) for non-developer maintenance.
- **Notifications:** Configure `SendUnifiedAlert` with corporate STARTTLS/TLS SMTP credentials (and valid SPF/DKIM records), or add Slack/Teams nodes.

## ⚠️ Known Limitations (PoC Scope)
To keep this repository transparent and engineering-accurate, please note the following PoC boundaries:
* **Mocked SSH Execution:** `CheckSSH(Mock)` simulates a healthy response (`"success"`). To test the SSH retry and alert path locally, change `"success"` to `"failed"` in `CheckSSH(Mock)` and `CheckSSH(RetryMock)`.
* **Stateless Monitoring (No Deduplication):** Without an external state store (Redis/PostgreSQL), a service that stays down across multiple consecutive runs will trigger an alert on each run, and automatic "Recovered" notifications are not sent once it comes back online.
* **Code-Based PoC Inventory:** `ListProgram` uses a JavaScript Code node for zero-dependency portability rather than an external database.

## ☁️ Cloud Deployment
Anagtia can run on Docker, PaaS (Cloud Run, AWS Fargate), Kubernetes, or n8n Cloud. For persistence setup (`PostgreSQL` + `N8N_ENCRYPTION_KEY`) and network hardening (VPN, least-privilege access, firewall whitelisting), see the [Cloud Deployment Guide](docs/CLOUD_DEPLOYMENT.md).

## 🔮 Future Improvements
- **Stateful Deduplication & Recovery Alerts:** Persist service state (`UP`/`DOWN` + `last_alerted_at`) in Redis or PostgreSQL to suppress duplicate hourly alerts and notify teams upon recovery.
- **Context-Aware AI Diagnostics (Optional V2):** Integrate a local LLM (e.g., via Ollama) **only after** capturing real diagnostic context (sanitized error payloads or tailed service logs), running AI enrichment asynchronously under a strict timeout so critical alerts are never delayed.

## 📸 Workflow Screenshot
![Workflow Screenshot](service.png)

**Note on the Screenshot:**
Greyed-out nodes in n8n execution views represent conditional branches that were correctly skipped during that test run (for example, `RecoveredAfterRetry` when services remained down, or `AllOk` when at least one target failed).

---
*Disclaimer: All company names, URLs, and email addresses in this repository are entirely fictional and for demonstration purposes only.*