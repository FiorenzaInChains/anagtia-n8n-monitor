# 🐍 Project Anagtia - Service Monitor (n8n PoC)

This is a personal Proof of Concept (PoC) project created to demonstrate automated service monitoring using n8n. The workflow checks the health of various programs (HTTP APIs and SSH servers) and alerts the IT and Administration departments in case of failures.

> **Why "Anagtia"?**
> Named after the ancient Samnite goddess of healing and protection (whose sacred animal was the snake), *Anagtia* acts as the silent guardian of your IT infrastructure. Just as the goddess cured and protected her people, this workflow monitors, heals (through alerts), and protects your servers, keeping them safe from unseen failures.

## 💡 Why this project?
While tools like Uptime Kuma or UptimeRobot are excellent for basic uptime monitoring, they act as closed silos. This project was built as a **Universal, Modular Framework** within n8n. 

Key advantages:
* **Dual Nature (HTTP + SSH):** Unlike standard templates, it uses a Switch to route checks. It monitors web endpoints (HTTP) and executes real commands on servers (SSH) to verify application health.
* **Modular & Universal:** Anyone can download the JSON and adapt it to their needs without writing a single line of code. Just change the data in the `ListProgram` node.
* **Anti-False Alarms:** Integrated retry logic prevents alert fatigue for the IT department.
* **Self-Hosted & Secure:** Runs entirely on your own infrastructure (Docker), keeping sensitive SSH credentials and data within your network perimeter.

## 🎯 Target Audience
This framework is designed for:
- **Companies migrating to n8n:** Organizations looking to consolidate their IT tools (including monitoring) into a single, open-source, self-hosted automation platform to reduce costs and maintenance overhead.
- **SMBs (Small and Medium Businesses):** Looking for a cost-effective, self-hosted monitoring solution without enterprise software costs.
- **Mixed Infrastructures:** Companies that need to monitor both web services (HTTP) and internal servers (SSH) from a single visual dashboard.
- **IT Teams preferring Low-Code:** Who want a visual, easily maintainable solution instead of complex custom scripts.

## 🚀 Features
- **Dynamic Routing:** Fetches a list of programs and uses a Switch node to route checks to HTTP Request or SSH execution.
- **Retry Logic to avoid False Alarms:** If a service fails, the workflow waits 60 seconds and tries again before sending an alert.
- **Loop Management:** Processes items one by one to prevent system overload.
- **Error Handling:** Uses `Continue (using error output)` to ensure the loop never breaks, even if a service is completely unreachable or the email server fails.
- **Extensible Alerting:** While Email is used in this PoC, the modular architecture allows swapping or running in parallel the notification node (e.g., sending alerts via both Email and Slack/Teams) without changing the core monitoring logic.
- **Flexible Alerting (Static & Dynamic):** The workflow currently uses static alerting (fixed email addresses in the notification nodes). However, the architecture is designed for **Dynamic Alert Routing**: by simply adding a `recipient` field (e.g., `dba@company.com`) to the `ListProgram` node, the workflow can automatically route alerts to the specific team responsible for the failing service, reducing alert fatigue for others.
- **Fully Configurable (Zero-Code Maintenance):** The schedule (Cron), retry wait times, and the list of monitored targets can be easily adjusted directly from the n8n visual interface, without touching the underlying logic or writing code.

## 🛠️ How it works
1. **Schedule Trigger:** Runs every hour from 9 AM to 6 PM.
2. **ListProgram (Code):** Generates a mock list of services to monitor.
3. **ProgramLoop:** Iterates through the list.
4. **RouteHTTP/SSH:** Routes the check based on the service type.
5. **Alerting:** Sends a notification if the service is still down after the retry.

## ⚙️ Configuration Example (ListProgram Node)

The `ListProgram` node is the "brain" of the system where you define the services to monitor. You can configure it in two ways depending on your alerting strategy. 

*Note: While this example uses Email for notifications, the `recipient` field can be adapted for any notification channel (e.g., a Slack Channel ID, a Teams Webhook, or a Telegram Chat ID).*

### Option 1: Static Alerting (Current PoC Setup)
If you want to send all alerts to the same fixed destination (e.g., a generic IT email), simply omit the `recipient` field. You will hardcode the destination directly in your notification nodes (`SendAlertHTTP` and `SendAlertSSH`).

```javascript
return [
  {
    json: {
      program_name: "Web Portal",
      type: "http",
      target: "https://api.example.com/health"
    }
  }
];
```

### Option 2: Dynamic Alert Routing (Advanced)
If you want to route alerts to specific teams based on the failing service, add the `recipient` field to the list. 

**Important:** To make this work, you must update the destination field (e.g., "To Email" in the Email node) to use the expression: `{{ $('ProgramLoop').item.json.recipient }}`.

```javascript
return [
  {
    json: {
      program_name: "Database Server",
      type: "ssh",
      target: "systemctl is-active postgresql",
      recipient: "dba-team@company.com"
    }
  }
];
```

## 📌 Transition to Production
Currently, the SSH nodes and SMTP credentials are mocked. To use it in a real environment:
- Replace the `CheckSSH(Mock)` nodes with the native n8n SSH node (using real server IP, port, and private key/password).
- Replace the mock SMTP credentials with the real corporate email server.
- Update the URLs in the `ListProgram` node with the real API endpoints.

## ☁️ Cloud Deployment
This workflow is designed to be Cloud-ready. It can be deployed on any Cloud provider (AWS, GCP, Azure) or VPS by running n8n in a Docker container. The workflow itself is agnostic to the hosting environment, allowing the orchestration of both Cloud-native APIs (HTTP) and private Cloud VMs (SSH) from a single centralized dashboard.

## 🔮 Future Improvements
- **Local AI Integration:** The modular architecture of this workflow is designed to easily integrate a local AI model (e.g., via Ollama). The next step will be to analyze the error logs automatically and use the AI to generate a detailed "root cause" explanation inside the alert email, helping the IT team troubleshoot faster.
- **Dual-Version Strategy (Classic vs AI-Enhanced):** The current V1 (Classic) is the core engine and is already fully modular. Features like Dynamic Alert Routing or sending alerts to Slack/Teams can be achieved by simply changing a field or swapping a node—no new version is needed. The planned V2 (AI-Enhanced) will be an optional version *exclusively* for environments with enough resources to run a local LLM (e.g., via Ollama), adding automated root-cause analysis to the alerts.

## 📸 Workflow Screenshot
![Workflow Screenshot](service.png)

**Note on the Screenshot:** 
You might notice that some nodes are grey in the image. This is expected behavior. Because the workflow uses conditional routing (Switch and IF nodes), n8n only executes the specific path required at that moment. Grey nodes simply mean that specific branch was correctly skipped during the test run (e.g., the service was working, so the alert path was not triggered).

---
*Disclaimer: All company names, URLs, and data in this repository are entirely fictional and for demonstration purposes only.*