# OpenClaw VPS Setup

This repository contains the necessary configuration files to deploy OpenClaw on a Virtual Private Server (VPS) using Docker Compose.

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

2. **Configure Environment Variables:**
   Copy the example environment file:
   ```bash
   cp .env.example .env
   ```
   Edit the `.env` file using a text editor (like `nano` or `vim`) and add your secure `OPENCLAW_GATEWAY_TOKEN` and the necessary API keys for your preferred AI providers.
   ```bash
   nano .env
   ```

3. **Launch OpenClaw:**
   Start the service in detached mode:
   ```bash
   docker compose up -d
   ```

4. **Access the Admin Interface:**
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

All OpenClaw container data, including the local SQLite databases and configurations, are stored in the `./data` directory relative to the `docker-compose.yml` file. This ensures data is maintained across container restarts and updates.
