# ☁️ Cloud Deployment Guide

Anagtia is designed to be completely Cloud-ready. Because it is built on n8n, the workflow is host-agnostic: it can run on your local machine for testing, or on a Cloud infrastructure for 24/7 production monitoring.

Here are the most common deployment strategies:

## 1. VPS with Docker (Recommended for full control)
Deploying n8n on a Virtual Private Server (VPS) via Docker is the most cost-effective and flexible approach. 
*   **Providers:** DigitalOcean, Hetzner, AWS EC2, Google Compute Engine.
*   **How it works:** You spin up a Linux server, install Docker, and run the official n8n Docker image. The workflow then lives on the server and executes autonomously via the internal Cron scheduler.
*   **Network Access:** To allow Anagtia to monitor internal/private servers via SSH, the VPS must be connected to the company network via a VPN (e.g., WireGuard or Tailscale).

## 2. Serverless Containers / PaaS (For scalability)
For a modern serverless approach, you can deploy the n8n container on managed platforms.
*   **Providers:** Google Cloud Run, AWS Fargate, Azure Container Apps.
*   **How it works:** You push the Docker image to the platform, and it handles the scaling automatically. 
*   **⚠️ Critical Note on Persistence:** In serverless environments, containers are ephemeral (they can scale to zero). You **must** connect an external database (e.g., PostgreSQL on Supabase or Cloud SQL) to persist workflow data and executions across container restarts.

## 3. Kubernetes (Enterprise-grade)
If the company already uses Kubernetes, n8n can be deployed via the official Helm chart. This ensures high availability and horizontal scaling, though it is overkill for a small number of monitors.

## 4. n8n Cloud (SaaS - Fully Managed)
If the company does not want to manage any infrastructure, they can use **n8n Cloud**, the official managed service provided by the n8n team. 
*   **Pros:** Zero maintenance, automatic updates, and built-in backups. 
*   **Cons:** It is a paid subscription, and the data resides on n8n's servers rather than the company's private network.

## 🔒 Security Considerations
*   **Environment Variables:** Store SMTP passwords, SSH keys, and API tokens in n8n's encrypted credential store, never in plain text.
*   **Network:** Ensure the Cloud instance has a static IP (or a VPN) so it can be whitelisted by the internal firewalls of the monitored servers.