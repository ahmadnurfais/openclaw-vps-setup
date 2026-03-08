# OpenClaw VPS Setup (Secure Private Edition)

This is my personal repository used to deploy [OpenClaw](https://github.com/openclaw/openclaw) on a Virtual Private Server (VPS). 

While the official OpenClaw documentation provides a great starting point, this specific configuration is tailored for **maximum security while enabling advanced "Sandbox Isolation"**. Because giving an AI agent Docker access (via `docker.sock`) is incredibly dangerous if exposed to the internet, this setup completely blocks public web access. 

If you are a newbie like me trying to set this up safely, feel free to use or modify this! You do **not** need to open any web ports on your VPS firewall (like UFW); we will tunnel everything securely through SSH.

## Why Docker over a Native Host Install?

If you are like me and run other active services (websites, databases, etc.) on your VPS, you **should** run OpenClaw in Docker. Here is why I chose this setup over a direct host installation:

**Pros of Docker:**
- **Zero Dependency Clashes:** Keeps OpenClaw entirely isolated. It will not break any existing apps or node versions on your server.
- **Easy Cleanup:** If the AI breaks its environment, you just destroy the container and spin it back up.
- **Security Boundaries:** While we map `docker.sock` for advanced tasks, the agent itself starts with heavily restricted networking and privileges compared to a raw Linux user.

**Cons of Docker (Trade-offs):**
- **Browser Automation:** Features where the AI tries to open a real web browser (Puppeteer/Playwright) are harder to configure and troubleshoot inside a headless container.
- **File Permissions:** Files created in the `openclaw-workspace` are owned by Docker's internal user (`node`), which can cause minor read/write annoyances if you try editing them directly on the VPS host.
- **System Blindness:** The AI cannot see your VPS desktop, process list, or other running services.

## Prerequisites

- A Linux VPS (Ubuntu 22.04 or 24.04 recommended)
- SSH access to the VPS
- [Docker](https://docs.docker.com/engine/install/) and [Docker Compose](https://docs.docker.com/compose/install/) installed

## Deployment Instructions

1. **Clone this repository to your VPS:**
   ```bash
   git clone https://github.com/ahmadnurfais/openclaw-vps-setup.git openclaw-setup
   cd openclaw-setup
   ```

2. **Create Data Directories:**
   Create the folders where OpenClaw will permanently save its brain and workspace:
   ```bash
   mkdir openclaw-config openclaw-workspace
   ```

3. **Configure Environment Variables:**
   ```bash
   cp .env.example .env
   nano .env
   ```
   Add your secure `OPENCLAW_GATEWAY_TOKEN`. *(Note: You do not need to add AI API keys here; they are managed internally by OpenClaw).* 

4. **Get your Docker Group ID (Crucial for Sandboxing):**
   Because we are letting the OpenClaw container talk to your host's Docker engine to spin up sandboxes, it needs permission. Run this command on your VPS to get the ID number:
   ```bash
   stat -c '%g' /var/run/docker.sock
   ```
   Open your `.env` file again and add that number at the bottom like this (replace 999 with your number):
   `DOCKER_GID=999`

5. **Launch OpenClaw:**
   ```bash
   docker compose up -d
   ```
   *Note: If you run `ufw status` on your VPS, you do NOT need to open port 18789. The `docker-compose.yml` is hardcoded to `127.0.0.1`, making it invisible to the outside world.*

6. **Complete the First-Time Onboarding:**
   After the first launch, the `openclaw-cli` container will exit immediately and disappear from `docker ps`. This is **expected** — it means the CLI has not been configured yet.

   **If you are using a local Ollama model**, you must download the model before running onboarding (skip this block if you are using OpenAI, Anthropic, etc.):
   ```bash
   # Enter the Ollama container shell
   docker exec -it ollama bash

   # Pull your model — choose the size based on your VPS RAM:
   # 16 GB RAM or more → use 14b
   # Under 16 GB RAM  → use 7b
   ollama pull qwen2.5-coder:14b

   # Verify the model was downloaded successfully
   ollama list

   # Exit the container shell
   exit
   ```

   Now run the onboarding wizard:
   ```bash
   docker compose run --rm -it openclaw-cli onboard
   ```

   When prompted for **Model/Auth provider**, follow the path for your setup:

   - **OpenAI or Anthropic**: Simply select your provider from the list and enter your API key when prompted.
   - **Local Ollama model**: Select **Custom provider**, then fill in the prompts as follows:
     - **Base URL:** `http://ollama:11434/v1`
     - **API Key:** choose *paste API key now* and leave it blank (just press Enter)
     - **Endpoint compatibility:** `OpenAI-compatible`
     - **Model ID:** the exact model you pulled, e.g. `qwen2.5-coder:14b`
     - **Endpoint ID:** a short slug, e.g. `ollama-qwen`
     - **Alias:** e.g. `ollama`

   Follow the remaining prompts to finish the initial configuration. Once done, restart the stack:
   ```bash
   docker compose up -d
   ```
   The CLI container should now stay running.

## Accessing the Admin Interface

Because the web UI is blocked from the internet, you cannot just type your VPS IP into your browser. You must create a secure "tunnel" from your personal computer to the VPS.

### Option A: Standard SSH Tunnel (Using Default Keys or Passwords)

1. **On your personal laptop/computer**, open a terminal or command prompt.
2. Run the following SSH Tunnel command (replace `root@your-vps-ip` with your actual VPS login):
   ```bash
   ssh -L 18789:localhost:18789 root@your-vps-ip
   ```
3. Leave that terminal window open! As long as it is running, the tunnel connects your laptop to the secure OpenClaw server.
4. Open your web browser and go to: `http://localhost:18789`
5. Enter the `OPENCLAW_GATEWAY_TOKEN` you created in your `.env` file.

### Option B: SSH Tunnel (Using a `.pem` Identity File)

If your server requires a specific `.pem` file to connect via SSH (common on AWS, EC2, or custom configurations), you need to pass it into the tunnel command using the `-i` flag.

*(If you do not know how to generate or configure a `.pem` file for SSH access, please refer to my guide here: [https://www.ahmadnurfais.my.id/](https://www.ahmadnurfais.my.id/))*

1. Run the SSH Tunnel command, pointing to your `.pem` key:
   ```bash
   ssh -i /path/to/your/key.pem -L 18789:localhost:18789 root@your-vps-ip
   ```
2. Leave the terminal window open to maintain the connection.
3. Open your web browser and go to: `http://localhost:18789`
4. Log in using your `OPENCLAW_GATEWAY_TOKEN`.

## Configuring AI Models

OpenClaw stores API keys and model profiles in its own internal SQLite database within your `openclaw-config` folder, keeping your `.env` file clean. 

### Option 1: Using Official APIs (OpenAI, Anthropic, etc.)
If you want fast responses and do not mind paying for API usage:
1. Access the web interface at `http://localhost:18789` (via your SSH tunnel).
2. Navigate to the **Settings** > **AI Providers** section.
3. Paste your official API key (e.g., your `sk-...` key from OpenAI) into the respective box and save.
4. When starting a chat, simply select `gpt-4o` or `claude-3-5-sonnet` from the dropdown list.

### Option 2: Using Free Local Models (Ollama)
This repository comes pre-configured with a private **Ollama** container. This allows you to download open-source AI models and run them entirely on your VPS's CPU and RAM for free. *(Note: Without a GPU, generation speed will be noticeably slower than Option 1)*.

The full setup for Ollama is covered in **Step 6** of the Deployment Instructions above, as the model must be downloaded *before* running the onboarding wizard. The recommended model is `qwen2.5-coder:14b` (or `qwen2.5-coder:7b` for VPS with under 16 GB RAM).

*If you change your mind and no longer want to host local models, simply delete the `ollama:` block from your `docker-compose.yml` and run `docker compose up -d` to clean it up.*

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

## Troubleshooting

### Error: `EACCES: permission denied, open '/home/node/.openclaw/node.json...'`
If you are running `docker compose` as the `root` user on your VPS, any data folders you mapped (like `~/Tools/openclaw-config`) will be owned by `root`. However, OpenClaw intentionally runs as a restricted, non-root user named `node` (User ID 1000) inside the container for security.

Because `node` cannot write to a `root`-owned folder, it crashes. To fix this, you must change the ownership of your host folders to UID 1000:
   ```bash
   chown -R 1000:1000 ~/Tools/openclaw-config ~/Tools/openclaw-workspace
   ```
