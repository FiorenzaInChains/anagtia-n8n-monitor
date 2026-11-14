# n8n Service Monitor - Proof of Concept

This is a personal Proof of Concept (PoC) project created to demonstrate automated service monitoring using n8n. The workflow checks the health of various programs (HTTP APIs and SSH servers) and alerts the IT and Administration departments in case of failures.

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

## 🛠️ How it works
1. **Schedule Trigger:** Runs every hour from 9 AM to 6 PM.
2. **ListProgram (Code):** Generates a mock list of services to monitor.
3. **ProgramLoop:** Iterates through the list.
4. **RouteHTTP/SSH:** Routes the check based on the service type.
5. **Alerting:** Sends an email notification if the service is still down after the retry.

## 📌 Transition to Production
Currently, the SSH nodes and SMTP credentials are mocked. To use it in a real environment:
- Replace the `CheckSSH(Mock)` nodes with the native n8n SSH node (using real server IP, port, and private key/password).
- Replace the mock SMTP credentials with the real corporate email server.
- Update the URLs in the `ListProgram` node with the real API endpoints.

## 🔮 Future Improvements
- **Local AI Integration:** The modular architecture of this workflow is designed to easily integrate a local AI model (e.g., via Ollama). The next step will be to analyze the error logs automatically and use the AI to generate a detailed "root cause" explanation inside the alert email, helping the IT team troubleshoot faster.
- **Dual-Version Strategy:** In a future release, I plan to offer two versions of this workflow: a lightweight "Classic" version (current) for standard setups, and an "AI-Enhanced" version for environments with enough resources to run local LLMs.

## 📸 Workflow Screenshot
![Workflow Screenshot](service.png)

**Note on the Screenshot:** 
You might notice that some nodes are grey in the image. This is expected behavior. Because the workflow uses conditional routing (Switch and IF nodes), n8n only executes the specific path required at that moment. Grey nodes simply mean that specific branch was correctly skipped during the test run (e.g., the service was working, so the alert path was not triggered).

---
*Disclaimer: All company names, URLs, and data in this repository are entirely fictional and for demonstration purposes only.*
