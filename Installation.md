This expanded guide provides a granular, command-by-command breakdown of the installation, configuration, and operation of a NemoClaw/OpenClaw environment. It is designed for users who want to see exactly what is happening under the hood.

---

# 🛡️ Master Guide: Manual NemoClaw + OpenClaw Deployment

This guide focuses on a **Manual Onboarding** path. Unlike the "Express" install, this method gives you control over memory allocation and security policies, which is essential for stable operation on 6GB RAM systems.

---

## Phase 1: Pre-Flight & System Prep
Before running the installer, ensure your WSL2 instance is healthy and Docker is communicating correctly.

1.  **Check WSL Resources:**
    Verify how much memory your Ubuntu instance sees:
    ```bash
    free -h
    ```
2.  **Verify Docker Connection:**
    Ensure the Docker daemon is reachable from within WSL:
    ```bash
    docker ps
    ```
3.  **Initialize the Manual Installer:**
    Run the script with the `NO_EXPRESS` flag to enter interactive mode.
    ```bash
    curl -fsSL https://www.nvidia.com/nemoclaw.sh | NEMOCLAW_NO_EXPRESS=1 bash
    ```

---

## Phase 2: The Interactive Setup (Step-by-Step)

During the script execution, you will be prompted for specific inputs. Here is the recommended sequence for a low-resource manual setup:

### 1. Swap Space (The Safety Net)
*   **Prompt:** `Create a local swap file? (y/n)`
*   **Action:** Type **`y`**.
*   **Command executed behind the scenes:** `sudo fallocate -l 4G /swapfile && sudo mkswap /swapfile && sudo swapon /swapfile`
*   *Why?* Docker image compilation is CPU/RAM intensive. 4GB of swap prevents the build from crashing.

### 2. Inference Configuration (OpenRouter)
*   **Provider:** Select `3` (Other OpenAI-compatible).
*   **Endpoint:** `https://openrouter.ai/api/v1`
*   **API Key:** Paste your token.
*   **Model:** `google/gemini-flash-1.5-8b` (recommended for speed) or `google/gemini-2.0-flash-lite-preview-02-05`.

### 3. Feature Stripping (Resource Optimization)
To keep the footprint small, disable auxiliary heavy services:
*   **Brave Search:** `N`
*   **Messaging (WhatsApp/Discord):** Press **Enter** to skip.
*   **Resource Profile:** **Type `6`** (No Profile / OpenShell Defaults).

### 4. Security Policy Compilation
The script will ask to apply **Landlock Linux Security Module (LSM)** rules.
*   **Action:** Press **Enter** to accept the default network egress rules (1-8).
*   This creates a "jail" around your AI agent, allowing it to only access specific repositories like `npm`, `pypi`, and `huggingface`.

---

## Phase 3: Monitoring the Build
The build process involves downloading and compiling ~80 Docker layers.

1.  **Monitor Progress:**
    In a second WSL terminal, you can watch Docker pull the images:
    ```bash
    docker events
    ```
2.  **Verify Image Creation:**
    Once finished, check if the images exist:
    ```bash
    docker images | grep openclaw
    ```

---

## Phase 4: Entering and Using the Sandbox

### 1. Connecting to the Container
The `nemoclaw` command acts as a bridge. Replace `sangi` with your workspace name.
```bash
# Connect to the shell of the sandbox
nemoclaw sangi connect
```

### 2. Working Inside the Sandbox (`sandbox@...`)
Once inside, you are in a secure, isolated Linux environment.
*   **Start the AI Agent Chat:**
    ```bash
    openclaw tui
    ```
*   **Run a Python script securely:**
    ```bash
    python3 -c "print('Hello from the Sandbox')"
    ```
*   **Check restricted network access:**
    ```bash
    curl https://google.com  # This should fail if Landlock is active
    curl https://pypi.org    # This should succeed
    ```

### 3. The Web Dashboard
1.  **Get the URL & Token:**
    ```bash
    nemoclaw sangi dashboard-url
    ```
2.  **Identify the WSL IP:**
    If the URL shows `localhost`, but you are on Windows, find your WSL IP:
    ```bash
    hostname -I | awk '{print $1}'
    ```

---

## 📚 The "Everything" Command Reference

### Workspace Management
| Command | Description |
| :--- | :--- |
| `nemoclaw list` | Show all created sandboxes and their status. |
| `nemoclaw <name> status` | Detailed health check (CPU/RAM usage of sandbox). |
| `nemoclaw <name> stop` | Gracefully shut down the sandbox container. |
| `nemoclaw <name> start` | Restart a stopped sandbox. |
| `nemoclaw <name> destroy` | **Permanent:** Deletes the sandbox and all files inside. |
| `nemoclaw <name> logs` | View the background logs for the agent. |
| `nemoclaw <name> logs --follow` | Stream logs in real-time (good for debugging). |

### AI & Inference Settings
| Command | Description |
| :--- | :--- |
| `nemoclaw inference show` | View current API keys and models being used. |
| `nemoclaw inference set --model <model>` | Switch the LLM for a specific sandbox. |
| `nemoclaw inference set --api-key <key>` | Update your OpenRouter/OpenAI key. |

### Messaging & Channels (Munshi)
| Command | Description |
| :--- | :--- |
| `nemoclaw munshi channels list` | See active WhatsApp/Discord integrations. |
| `nemoclaw munshi channels add whatsapp`| Start the QR code pairing process for WhatsApp. |
| `nemoclaw munshi channels remove <id>` | Disconnect a messaging channel. |

### Advanced Troubleshooting (Docker-Level)
| Command | Description |
| :--- | :--- |
| `docker stats` | See real-time RAM/CPU usage of the OpenClaw container. |
| `docker inspect openclaw-<name>` | View the low-level JSON config (mounts, env vars). |
| `nemoclaw doctor` | Run a diagnostic script to find install errors. |

### Inside the Sandbox (OpenClaw Commands)
*These commands are run **after** you run `nemoclaw connect`.*
| Command | Description |
| :--- | :--- |
| `openclaw tui` | Launch the Terminal User Interface chat. |
| `openclaw version` | Check the current OpenShell core version. |
| `/exit` | Exit the TUI chat mode. |
| `exit` | Exit the container and return to Windows/WSL host. |

---

### Final Maintenance Tip:
If your sandbox becomes sluggish, it is often due to log accumulation or dangling Docker layers. Run this occasionally to keep the 6GB RAM profile lean:
```bash
docker system prune -f
``` 
*(Note: This only deletes unused data; it won't delete your active sandbox.)*
