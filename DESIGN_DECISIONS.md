# Architecture & Design Decisions

This document explains the rationale behind the architectural choices made for this Proof of Concept (PoC). 

## 1. Why this project has value?
In many companies, service monitoring is handled by complex scripts or separate software that do only one thing. The value of this PoC is not just the monitoring itself, but the creation of a **Modular and Universal Framework** within n8n.

- **Zero Silos:** The monitoring is not a closed box. It lives inside the same tool the company can use for dozens of other automations. If a service goes down tomorrow, the system could easily be extended to open a Jira ticket or restart the service, all within the same flow.
- **Dual Nature:** It combines Web checks (HTTP) and Server checks (SSH) in a single architecture. Standard n8n templates usually only do one of the two.
- **Anti-False Alarms:** The `Wait 60s` and "Retry" logic prevents alert fatigue for the IT department caused by temporary network hiccups.
- **Ready-to-use (Low-Code):** Anyone can download the JSON and, by just changing the list of programs, have a working monitoring system. No programming skills required.
- **Flexible & Multi-Channel Alerting (Static & Dynamic):** The alerting logic is fully decoupled from the monitoring logic. The current PoC uses static email alerts, but the architecture supports dynamic routing by simply passing a `recipient` parameter in the loop's payload. Furthermore, it is channel-agnostic: companies can easily replace or add parallel notification nodes (e.g., Email, Slack, Teams) adapting the system perfectly to their internal workflows. It can also be extended to external customer-facing communications (like Status Page updates) during critical outages.

## 2. Comparison with other solutions

### A. vs. Uptime Kuma / UptimeRobot
- **Them:** They are standalone software ("closed boxes"). They check if a site responds or if the SSH port (22) is open, and send standard notifications.
- **This Project:** It is an **orchestrator**. It checks if the site responds, but for SSH it doesn't just open a port: it executes a command (e.g., `systemctl is-active`) and reads the actual server response. Furthermore, being in n8n, it can integrate with any other corporate system in 2 clicks.

### B. vs. Python or Bash Scripts
- **Them:** They are hard to modify. If the company wants to change the schedule or add a check 6 months from now, a developer is needed.
- **This Project:** It is **visual and highly configurable**. The IT department can modify the schedule, change the retry wait time, or add new targets by simply opening n8n, changing a value in the node, and saving. Zero-code maintenance.

### C. vs. Existing n8n Templates
- **Them:** Community templates are often monolithic (only HTTP or only Telegram) and lack advanced logic.
- **This Project:** It is a tested **universal framework** that solves advanced technical issues (like data overwriting in loops) to ensure alerts always contain the exact name of the failing program.

## 3. Technical Q&A

**Q: Why n8n and not a custom script?**
A: n8n is open-source and can be self-hosted for free on a company server (on-premise), ensuring maximum security for SSH credentials. It is Low-Code, meaning anyone in the company can understand and maintain it, and it has native Error Handling and Retry logic that would require much more code to write from scratch.

**Q: How does the SSH check work if there are no real servers connected yet?**
A: Currently, a Code node is used to simulate the check and return 'success'. In production, this node will be replaced by the native n8n SSH node, inserting the real server IP and credentials. The downstream logic (the IF and the retry) will remain identical.

**Q: What was the hardest technical challenge to solve?**
A: Data handling within the Loop. When the HTTP node executes the call, it overwrites the original program data with the server's response. To avoid losing the program name for the notification email, a specific syntax (`$('ProgramLoop').item.json.program_name`) was used to force n8n to fetch the original data from the loop's memory.

**Q: How long would it take to move to production?**
A: Very little. The structure is 100% ready. It only requires: 1) Inserting the real URLs in the List node. 2) Configuring SSH credentials. 3) Inserting the corporate SMTP server for email sending. The rest of the flow remains exactly as it is.

## 4. Architectural Extensibility (Future AI)
The choice to use a modular Loop and Switch architecture was not accidental. By isolating the monitoring logic from the alerting logic, the workflow is "AI-ready". In the future, a local AI model (e.g., via Ollama) can be inserted just before the notification nodes to analyze the error logs and generate a root-cause explanation, without needing to rewrite the core monitoring logic.