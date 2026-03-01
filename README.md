# Homelab

A private Python-based data science lab built on the [MaxLab](https://hub.docker.com/r/navigatorbbs/maxlab) container image exposed securely over [Tailscale](https://tailscale.com).

---

## Prerequisites

### On the Ubuntu Linux host

1. **Docker Engine** — Install Docker from the [official repository](https://docs.docker.com/engine/install/ubuntu/).
2. **OpenSSH Server** — Install and enable:
   ```bash
   sudo apt update
   sudo apt install openssh-server
   sudo systemctl start ssh
   sudo systemctl enable ssh
   ```
3. **Tailscale** — Install from <https://tailscale.com/download/linux/ubuntu> and log in:
   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   sudo tailscale up
   ```
   The host's Tailscale IP (e.g. `100.x.x.x`) or MagicDNS hostname is used as `SSH_HOST`.
4. **SSH key-based authentication** — Ensure your GitHub Actions public key is in `~/.ssh/authorized_keys`.

--

## First-time setup

Run the **Configure Homelab Host** workflow once before the first deploy:

1. Navigate to **Actions → Configure Homelab Host → Run workflow**.
2. Fill in:
   - **Notebook path** — Linux path where notebooks are stored persistently (default: `/mnt/storage/maxlab`).
   - **JupyterLab token** — Leave blank to auto-generate a secure token, or provide your own.
3. Click **Run workflow**.

This creates the required directories and writes `.env` on the host at `/home/$SSH_USER/homelab/.env`.

---

## Accessing JupyterLab

Once deployed, open a browser on any device in your tailnet:

```
http://<tailscale-ip-or-hostname>:8888
```

Enter the `JUPYTER_TOKEN` from `/home/$SSH_USER/homelab/.env` when prompted.

---

## Persistent storage

Notebook files are stored at the path specified during configuration (default `/mnt/storage/maxlab`) and mounted into the container at `/home/jovyan/work`. Data survives container restarts and image updates.

---

## Local development

Copy `.env.example` to `.env` in the same directory as `docker-compose.yml` and edit the values, then:

```bash
docker compose up -d
```

---

## Quick Start Scripts

This repository includes helper scripts to simplify common tasks. All scripts use bash and require Docker and Tailscale to be installed on your system.

### `start.sh` — Start the container

**Purpose:** Start the MaxLab container in the background.

```bash
./start.sh
```

**What it does:**
- Runs `docker compose up -d` to start the container
- Pulls the latest image if needed
- Container runs in detached mode

**When to use:** After initial setup or whenever you need to restart the MaxLab service.

---

### `stop.sh` — Stop the container

**Purpose:** Stop the running MaxLab container gracefully.

```bash
./stop.sh
```

**What it does:**
- Runs `docker compose down` to stop and remove the container
- Preserves mounted volumes (persistent storage)

**When to use:** When you're done using MaxLab or need to perform maintenance.

---

### `tailserve.sh` — Enable HTTPS access via Tailscale

**Purpose:** Expose JupyterLab securely over HTTPS using Tailscale Serve.

```bash
./tailserve.sh
```

**What it does:**
- Runs `tailscale serve --service=svc:maxlab --https=443 127.0.0.1:8888`
- Creates a service that proxies HTTPS traffic on port 443 to the local JupyterLab on port 8888
- Provides a secure, encrypted tunnel for remote access

**When to use:** To access JupyterLab securely from outside your local network without exposing ports directly. After running this, you can access JupyterLab via `https://<tailscale-hostname>`.

---

### `tailrm.sh` — Disable Tailscale Serve

**Purpose:** Remove the Tailscale Serve configuration.

```bash
./tailrm.sh
```

**What it does:**
- Runs `tailscale serve --remove=svc:maxlab` to remove the service proxy
- Stops exposing JupyterLab via Tailscale Serve

**When to use:** When you no longer need remote HTTPS access via Tailscale.
