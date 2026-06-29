# Connecting Discord to NemoClaw + OpenClaw: The Complete Beginner's Guide

Welcome! This guide will walk you through the process of connecting a Discord bot to **NemoClaw** and **OpenClaw**. Think of this like setting up a secure communication line between a remote worker (the AI agent) and your office (Discord).

---

## 1. Understanding the Stack
Before we type any commands, let’s look at the "team" we are working with:
*   **NemoClaw:** The manager. It creates a safe "sandbox" (an isolated playground) on your computer where the AI lives.
*   **OpenClaw:** The worker. This is the actual AI agent that will be talking to you.
*   **OpenShell:** The security guard. It sits inside the sandbox and, by default, blocks all internet traffic to keep things safe.
*   **Discord Bot:** The telephone. This is how you and the AI send messages back and forth.

---

## 2. Setting Up the Discord Foundation
### The "Why" and "What"
Before the AI can join your server, you need to create a "seat" for it. In Discord terms, this means creating an **Application** and a **Bot User**. Think of this as creating a specialized user account that a program (OpenClaw) can log into.

### Sufficient Code Examples
To get your IDs, you need to use Discord's **Developer Mode**:
1.  Go to **User Settings** > **Advanced** > Toggle **Developer Mode ON**.
2.  **Server ID:** Right-click your server name in the list > **Copy Server ID**.
3.  **User ID:** Right-click your own name > **Copy User ID**.

### Micro-Project/Exercise
**Task:** Create your bot identity.
1.  Visit the [Discord Developer Portal](https://discord.com/developers/applications).
2.  Create a "New Application" named "OpenClaw-Bot".
3.  Under the **Bot** tab, enable **Message Content Intent** (this allows the AI to actually read what you type).
4.  Copy your **Bot Token** and save it in a Notepad file. **Warning:** Treat this like a password; never share it!

### Official Sources & Documentation
*   [Discord Developer Portal Documentation](https://discord.com/developers/docs/intro)
*   [How to Create a Discord Bot (Discord.js Guide)](https://discordjs.guide/preparations/setting-up-a-bot-application.html)

---

## 3. Installing NemoClaw & Onboarding
### The "Why" and "What"
Now that the Discord side is ready, we need to install the software on your computer. We use an "Onboarding Wizard," which is a step-by-step interview that configures the sandbox for you.

### Sufficient Code Examples
```bash
# Step 1: Download and install the NemoClaw manager
curl -fsSL https://nvidia.com/nemoclaw.sh | bash

# Step 2: Run the interactive setup wizard
# This will ask for your Bot Token and API keys
nemoclaw onboard
```
*During the wizard, select **Discord** as your channel and paste the token you saved earlier.*

### Micro-Project/Exercise
**Task:** Verify your installation.
Run `nemoclaw --version` in your terminal. If it returns a version number, your "Manager" is successfully installed and ready to work.

### Official Sources & Documentation
*   [NVIDIA Nemo Framework Documentation](https://docs.nvidia.com/deeplearning/nemo/user-guide/docs/en/main/)

---

## 4. Managing Network Policies (The Security Guard)
### The "Why" and "What"
This is the most common place where beginners get stuck. Remember **OpenShell (The Security Guard)**? By default, he blocks the AI from talking to the internet. Since Discord lives on the internet, we have to explicitly tell the guard: "It's okay for the AI to talk to Discord."

### Sufficient Code Examples
If you see errors like `EAI_AGAIN discord.com` or `websocket closed: 1006`, it means the guard is blocking the connection.

```bash
# 1. Add the "Discord" permission to your sandbox (named 'mini')
nemoclaw mini policy-add discord

# 2. Rebuild the sandbox so the security guard gets the new rules
nemoclaw mini rebuild --yes

# 3. Check if the policy is active (look for a filled circle ●)
nemoclaw mini policy-list
```

### Micro-Project/Exercise
**Task:** Watch the "Security Guard" in real-time.
Run `nemoclaw mini logs --follow`. Watch the text scroll by. You are looking for lines that say `ALLOWED discord.com:443`. This confirms the gate is open!

### Official Sources & Documentation
*   [Git-SCM: Understanding Network Protocols](https://git-scm.com/book/en/v2/Git-on-the-Server-The-Protocols) (General concept of network layers).

---

## 5. Pairing the Bot (The Handshake)
### The "Why" and "What"
Even with the gate open, the AI needs to know *which* human it is allowed to talk to. This is a security feature called **Pairing**. It prevents random strangers from hijacking your AI agent.

### Sufficient Code Examples
To pair, you must send a Direct Message (DM) to your bot in Discord with your specific IDs:

```text
I already set my Discord bot token in config. Please finish Discord setup with User ID <your_user_id> and Server ID <your_server_id>
```
*Replace the brackets with the long numbers you copied in Step 2.*

### Micro-Project/Exercise
**Task:** Perform the Handshake.
Find your bot in the Discord member list, right-click, and select **Message**. Paste the pairing phrase. If the bot replies, you have successfully bridged the gap between your computer and Discord!

---

## 6. Diagnostic Commands (The Toolkit)
### The "Why" and "What"
In version control and DevOps, we always check the "Health" of our system. If the bot stops responding, use these commands to find out why.

### Sufficient Code Examples
| Command | What it does |
| :--- | :--- |
| `nemoclaw mini status` | Shows if the sandbox is running or crashed. |
| `nemoclaw mini doctor` | Runs a "physical" on your setup to find configuration errors. |
| `nemoclaw mini recover` | The "Restart" button. Use this if the bot is acting glitchy. |
| `nemoclaw mini logs --follow` | Shows the "brain" of the AI. Great for seeing error messages. |

---

## 7. Troubleshooting Common Roadblocks

| Symptom | Cause | The Fix |
| :--- | :--- | :--- |
| Bot is "Invisible" / Offline | Network block | Run `nemoclaw mini policy-add discord` |
| Bot is Online but won't talk | Intent missing | Go to Discord Dev Portal > Bot > Enable "Message Content Intent" |
| "Provider credential not found" | Missing API Key | Set your key: `COMPATIBLE_API_KEY=your-key nemoclaw mini channels add discord` |
| Bot replies in DM but not Channel | Mention Gating | You must @mention the bot (e.g., `@OpenClaw hello`) |

### Summary Checklist for Success:
1.  **Discord:** Bot created, Token saved, Intents enabled.
2.  **NemoClaw:** Installed and `onboard` completed.
3.  **Policy:** `policy-add discord` applied and `rebuild` finished.
4.  **Pairing:** DM sent to the bot with User/Server IDs.
5.  **Logging:** Use `nemoclaw mini logs --follow` to watch for `ALLOWED` traffic.

**Congratulations!** You have just deployed a sandboxed AI agent and connected it to a global messaging platform. You are now exploring the intersection of AI and DevOps!
