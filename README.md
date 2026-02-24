# Homelab

A private Python-based data science lab built on the [MaxLab](https://hub.docker.com/r/navigatorbbs/maxlab) container image, deployed to a Windows Server 2025 host via GitHub Actions and exposed securely over [Tailscale](https://tailscale.com).

---

## Architecture

```
GitHub Actions
     │
     ├─ tailscale/github-action  ──►  Joins runner to your tailnet
     │
     └─ appleboy/ssh-action ──────►  Windows Server 2025 (Docker Engine)
                                          │
                                          └─ docker compose up
                                               └─ navigatorbbs/maxlab:latest
                                                    Port 8888  ◄──  Tailscale IP
                                                    Volume: C:\homelab\notebooks
```

---

## Prerequisites

### On the Windows Server 2025 host

1. **Docker Engine** — Install the Docker Engine for Windows (not Docker Desktop).
2. **OpenSSH Server** — Enable via *Settings → Apps → Optional Features → OpenSSH Server*, or with PowerShell:
   ```powershell
   Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
   Start-Service sshd
   Set-Service -Name sshd -StartupType Automatic
   ```
3. **Tailscale** — Install from <https://tailscale.com/download> and log in.  
   The host's Tailscale IP (e.g. `100.x.x.x`) or MagicDNS hostname is used as `SSH_HOST`.
4. **PowerShell** must be the default SSH shell (it is by default on Windows Server 2025).

### GitHub repository secrets

Add the following secrets under **Settings → Secrets and variables → Actions**:

| Secret | Description |
|---|---|
| `TAILSCALE_AUTHKEY` | Reusable (or ephemeral) auth key from the [Tailscale admin console](https://login.tailscale.com/admin/settings/keys). Tag it with `tag:ci`. |
| `SSH_HOST` | Tailscale IP or MagicDNS hostname of the Windows Server. |
| `SSH_USER` | Windows username with permission to run Docker commands. |
| `SSH_PRIVATE_KEY` | Private key whose public counterpart is in `%USERPROFILE%\.ssh\authorized_keys` on the host. |

---

## First-time setup

Run the **Configure Homelab Host** workflow once before the first deploy:

1. Navigate to **Actions → Configure Homelab Host → Run workflow**.
2. Fill in:
   - **Notebook path** — Windows path where notebooks are stored persistently (default: `C:/homelab/notebooks`).
   - **JupyterLab token** — Leave blank to auto-generate a secure token, or provide your own.
3. Click **Run workflow**.

This creates the required directories and writes `C:\homelab\.env` on the host.

---

## Deploying

The **Deploy MaxLab** workflow runs automatically on every push to `main` that changes `docker-compose.yml` or the deploy workflow file. You can also trigger it manually:

1. Navigate to **Actions → Deploy MaxLab → Run workflow**.
2. Optionally enable **Force pull** to pull the latest image even if the compose file hasn't changed.
3. Click **Run workflow**.

The workflow:
1. Joins the GitHub Actions runner to your Tailscale tailnet.
2. Copies `docker-compose.yml` to `C:\homelab\` on the host.
3. Pulls `navigatorbbs/maxlab:latest` and restarts the service via `docker compose up -d`.

---

## Accessing JupyterLab

Once deployed, open a browser on any device in your tailnet:

```
http://<tailscale-ip-or-hostname>:8888
```

Enter the `JUPYTER_TOKEN` from `C:\homelab\.env` when prompted.

---

## Persistent storage

Notebook files are stored at the path specified during configuration (default `C:\homelab\notebooks`) and mounted into the container at `/home/jovyan/work`. Data survives container restarts and image updates.

---

## Local development

Copy `.env.example` to `.env` in the same directory as `docker-compose.yml` and edit the values, then:

```powershell
docker compose up -d
```
