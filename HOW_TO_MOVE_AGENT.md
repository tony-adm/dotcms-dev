# How to Move an OpenClaw Agent (Brain Transplant)

This guide explains how to migrate your OpenClaw agent (identity, memory, skills, and context) from one machine to another by leveraging its Git-based workspace.

## 🧠 What gets moved?
*   **Identity:** `AGENTS.md`, `SOUL.md`, `USER.md`
*   **Memory:** `MEMORY.md` and daily logs (`memory/YYYY-MM-DD.md`)
*   **Skills:** Custom skills in `skills/`
*   **Project Context:** Any project files (like `HANDOFF_HOST2.md`)

## 🚫 What does NOT get moved?
*   **Secrets:** API keys and tokens in `~/.openclaw/config.yaml`.
*   **System Dependencies:** Node.js, npm, Docker, etc. must be installed separately on the target machine.

---

## Step 1: Source Machine (Backup)

1.  **Navigate to the Workspace:**
    ```bash
    cd ~/.openclaw/workspace
    ```

2.  **Initialize Git (if not already):**
    ```bash
    git init
    git remote add origin <your-private-repo-url>
    ```

3.  **Push the Brain:**
    ```bash
    git add .
    git commit -m "backup: agent state"
    git push -u origin main
    ```

---

## Step 2: Target Machine (Restore)

1.  **Install OpenClaw:**
    Run the standard installation command first.
    ```bash
    npm install -g openclaw
    ```

2.  **Initialize Default Config:**
    Run `openclaw init` (or just `openclaw`) once to generate the default configuration file at `~/.openclaw/config.yaml`.
    *Exit immediately after it starts (Ctrl+C).*

3.  **Swap the Brain:**
    Delete the default empty workspace and clone your backup.
    ```bash
    cd ~/.openclaw
    rm -rf workspace
    git clone <your-private-repo-url> workspace
    ```

4.  **Restore Secrets (Manual):**
    Open `~/.openclaw/config.yaml` and re-enter your API keys (OpenAI, Anthropic, etc.) manually.
    *Do not commit this file to the repo.*

5.  **Wake Up:**
    Run `openclaw`.
    The agent will wake up with all memories and context intact from the source machine.
