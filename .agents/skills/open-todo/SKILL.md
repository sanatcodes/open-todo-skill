---
name: open-todo
description: "Use Open Todo as a personal task list for agents. Covers one-command setup, CLI install, email OTP login, credential persistence, HTTP fallback, MCP config, reading tasks, creating tasks, progress updates, and completion."
---

# Open Todo

Open Todo is an agent-friendly todo system. Use the fastest route available in the current client:

1. CLI for local shell-capable agents such as Codex, Claude Code, Cursor, Gemini CLI, and OpenCode.
2. Direct HTTP for hosted chat agents that can make network requests but cannot run a local binary.
3. MCP only when the client already exposes Open Todo MCP tools or the user asks for MCP setup.

Default hosted API URL:

```sh
https://sanat-todo.sanat-thukral.workers.dev
```

If `OPEN_TODO_API_URL` is already set, use it instead of the default.

## Setup

First check configuration:

```sh
command -v sanat-todo
sanat-todo config
```

If `sanat-todo` is missing and shell commands are available, install it:

```sh
curl -fsSL "${OPEN_TODO_API_URL:-https://sanat-todo.sanat-thukral.workers.dev}/install.sh" | sh
```

If the installer says the binary was placed in `~/.local/bin` but the command is still unavailable, run it by full path or tell the user to add `~/.local/bin` to `PATH`.

Then log in:

```sh
sanat-todo --url "${OPEN_TODO_API_URL:-https://sanat-todo.sanat-thukral.workers.dev}" login
```

The login command asks for email, asks for the six-digit code, verifies OTP, and saves the Open Todo API token locally. After login, use `sanat-todo config` to confirm `token_saved: true` without printing the token.

## Persistence Rules

- Do not store tokens inside the skill source directory.
- Prefer CLI persistence for local agents. The CLI stores credentials in `~/.config/open-todo/config.json`, or `$XDG_CONFIG_HOME/open-todo/config.json` when set.
- Prefer the host agent's secret/config store when it exists.
- Use environment variables for ephemeral sessions, CI, and containers.
- If no persistent store exists, keep the token only for the current session and clearly say login will be needed again.
- Never print a saved token during normal reads.

## CLI Usage

Use a stable `--source` for the current client, such as `codex`, `claude`, `cursor`, `chatgpt`, `manual-api`, or `cron`.

Read open tasks:

```sh
sanat-todo brief --status open --limit 50 --pretty --source codex
```

Create a task:

```sh
sanat-todo create "Short task title" --section thisweek --tag Agent --source codex
```

Record progress:

```sh
sanat-todo progress TASK_ID "Concrete progress and next step." --source codex --metadata '{"workspace":"local","branch":"main"}'
```

Complete a task:

```sh
sanat-todo done TASK_ID --source codex
```

Use `sections.urgent`, `sections.thisweek`, and `sections.upcoming` from `brief` output to plan work. Fetch full task details only when notes or exact metadata are needed.

## HTTP Fallback

Use direct HTTP when the CLI is unavailable or the current client must manage credentials itself.

Set the base URL:

```text
OPEN_TODO_API_URL=https://sanat-todo.sanat-thukral.workers.dev
```

If no token is available:

1. Ask the user for their email address.
2. Start OTP:

```text
POST {OPEN_TODO_API_URL}/auth/otp/start
Content-Type: application/json

{ "email": "user@example.com" }
```

3. Ask for the six-digit code from email.
4. Verify OTP:

```text
POST {OPEN_TODO_API_URL}/auth/otp/verify
Content-Type: application/json

{ "email": "user@example.com", "token": "123456" }
```

5. Save the returned `api_token` as `OPEN_TODO_TOKEN` in the user's local agent config or environment when persistence is available.
6. Send authenticated requests with `Authorization: Bearer OPEN_TODO_TOKEN`.

If the user clicks an email magic link instead of providing an OTP, the callback page displays the one-time API token plus copyable `OPEN_TODO_*`, CLI, and MCP setup snippets. Ask the user to paste only the token or env snippet needed for the current client.

Fast read:

```text
GET {OPEN_TODO_API_URL}/tasks/brief?status=open&limit=50
Authorization: Bearer OPEN_TODO_TOKEN
```

Create task:

```text
POST {OPEN_TODO_API_URL}/tasks
Authorization: Bearer OPEN_TODO_TOKEN
Content-Type: application/json

{
  "title": "Short task title",
  "notes": "Optional context",
  "section": "upcoming",
  "tag": "Optional",
  "source": "codex"
}
```

Record progress:

```text
POST {OPEN_TODO_API_URL}/tasks/TASK_ID/progress
Authorization: Bearer OPEN_TODO_TOKEN
X-Todo-Source: codex
Content-Type: application/json

{
  "summary": "Concrete progress and next step.",
  "metadata": {
    "workspace": "local workspace name",
    "branch": "current branch"
  }
}
```

Complete:

```text
POST {OPEN_TODO_API_URL}/tasks/TASK_ID/complete
Authorization: Bearer OPEN_TODO_TOKEN
X-Todo-Source: codex
```

## MCP

When the user wants MCP config, use:

```json
{
  "mcpServers": {
    "open-todo": {
      "url": "https://sanat-todo.sanat-thukral.workers.dev/mcp",
      "headers": {
        "Authorization": "Bearer OPEN_TODO_TOKEN"
      }
    }
  }
}
```

If `OPEN_TODO_API_URL` is set, replace the URL with `{OPEN_TODO_API_URL}/mcp`.

## Source Attribution

Always identify the client on writes with either `--source`, `X-Todo-Source`, or a JSON `source` field. Put branch names, model names, run IDs, and workspaces in `metadata`, not in `source`.
