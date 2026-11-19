# ☁️ Cloud Deployment Guide

Anagtia is host-agnostic and runs anywhere n8n can run in Docker. Below are the recommended deployment topologies and mandatory security controls for moving beyond local testing.

## 1. Deployment Options

### VPS with Docker Compose (Recommended for SMBs)
* **Providers:** Hetzner, DigitalOcean, AWS EC2, Google Compute Engine.
* **Setup:** Run the official `n8nio/n8n` container behind a TLS reverse proxy (Caddy, Traefik, or Nginx) backed by PostgreSQL.
* **Private Network Access:** Connect the VPS to your internal servers over a private mesh VPN (**WireGuard** or **Tailscale**) rather than exposing internal SSH/HTTP ports to the public internet.

### Serverless Containers / PaaS
* **Providers:** Google Cloud Run, AWS Fargate, Azure Container Apps.
* **⚠️ Critical Persistence Requirements:**
  1. Connect an external **PostgreSQL** database so workflows and execution history survive container restarts.
  2. Set a static **`N8N_ENCRYPTION_KEY`** environment variable stored in your Cloud Secret Manager. Without a fixed encryption key, n8n generates a random key on startup, making all saved SMTP/API credentials unreadable after the next deployment.
  3. Ensure the container has a scheduled/always-on trigger strategy (serverless scale-to-zero containers will not fire internal n8n Cron triggers unless kept warm or triggered externally via Cloud Scheduler -> n8n Webhook).

### Kubernetes
* Deploy using the community/official n8n Helm chart with PostgreSQL and Redis (queue mode) if scaling to high execution volumes.

## 2. Mandatory Security & Hardening Checklist
* **Protect the n8n Control Plane:** Never expose the n8n UI publicly without HTTPS, strong authentication/2FA (or SSO), and IP allowlisting / VPN-only access. Remember that n8n's `Code` node executes arbitrary JavaScript on the host container—compromised editor access equals Remote Code Execution.
* **Least-Privilege Monitoring Access (Avoid Root SSH):** Do not grant `root` SSH access or password authentication to n8n. Prefer read-only HTTP healthcheck endpoints secured with API keys or mTLS over VPN. If SSH is strictly required, use dedicated Ed25519 keys, a non-privileged monitoring user, and `sudoers` / `ForceCommand` restricted to read-only status commands.
* **Firewall & IP Whitelisting:** Restrict target server firewalls so healthcheck ports only accept traffic from the n8n worker's static VPN/egress IP.
* **Email Security:** Configure SMTP with STARTTLS/TLS (port 587/465) and verify SPF, DKIM, and DMARC records on your sender domain so alerts are not rejected or flagged as spam.