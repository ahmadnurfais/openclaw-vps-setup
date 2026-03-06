# OpenClaw VPS Setup

This repository contains the official configuration files to deploy OpenClaw on a Virtual Private Server (VPS) using Docker Compose.

## Prerequisites

- A Linux VPS (Ubuntu 22.04 or 24.04 recommended)
- Root or sudo privileges
- [Docker](https://docs.docker.com/engine/install/) and [Docker Compose](https://docs.docker.com/compose/install/) installed

## Deployment Instructions

1. **Clone this repository to your VPS:**
   ```bash
   git clone <your-repository-url> openclaw-setup
   cd openclaw-setup
   ```

2. **Create necessary directories:**
   The official OpenClaw Docker composition requires separate directories for configuration and workspace context. Create them inside your cloned repository:
   ```bash
   mkdir openclaw-config openclaw-workspace
   ```

3. **Configure Environment Variables:**
   Copy the example environment file:
   ```bash
   cp .env.example .env
   ```
   Edit the `.env` file using a text editor (like `nano` or `vim`). You must provide a secure `OPENCLAW_GATEWAY_TOKEN` and the necessary API keys for your preferred AI providers. Make sure the relative paths for `OPENCLAW_CONFIG_DIR` and `OPENCLAW_WORKSPACE_DIR` match the folders you just created.
   ```bash
   nano .env
   ```

4. **Launch OpenClaw:**
   Start the services (gateway and cli) in detached mode:
   ```bash
   docker compose up -d
   ```

5. **Access the Admin Interface:**
   Once running, you can access the OpenClaw Gateway at `http://<your-vps-ip>:18789`. You will need your `OPENCLAW_GATEWAY_TOKEN` to authenticate.
   
   *Security Note: It is highly recommended to secure the web interface further by setting up a reverse proxy (like Nginx or Caddy) with HTTPS, or by accessing it locally via an SSH tunnel.*

## Managing the Service

- **View Logs:** `docker compose logs -f`
- **Stop Service:** `docker compose down`
- **Update OpenClaw:** 
  ```bash
  docker compose pull
  docker compose up -d
  ```

## Data Persistence

All OpenClaw container data, including standard configurations and active workspaces, are bound to the `OPENCLAW_CONFIG_DIR` and `OPENCLAW_WORKSPACE_DIR`. This ensures data is maintained across container restarts and lifecycle updates.
