---
name: open-todo
description: "Use Open Todo as a personal task list for agents. Covers simple email OTP setup, direct HTTP task operations, safe token handling, optional existing CLI usage, and MCP config."
---

# Open Todo

Open Todo carries your work across agents. It gives each agent a shared task list, so they can add tasks, close them, and leave status updates as work happens.

Your todo list moves with the work. The agent keeps it current.

Keep setup simple and safe:

1. Use direct HTTP by default.
2. Use the CLI only when `sanat-todo` is already installed.
3. Use MCP only when the client already exposes Open Todo MCP tools or the user asks for MCP setup.

Default hosted API URL:

```sh
https://sanat-todo.sanat-thukral.workers.dev
```

If `OPEN_TODO_API_URL` is already set, use it instead of the default.

## Setup

Use this flow when the user says "Set up Open Todo" or when no token is available.
This setup should feel like a short login prompt, not documentation.

Before giving instructions, decide what single next input is needed:

- If no email and no OTP are present, ask only: `What email should I use for Open Todo?`
- If the user says they already received an OTP but has not provided it, ask only for the six-digit code. If the email is also missing, ask for both email and code in one sentence.
- If an email is already present but no OTP has been sent in this conversation, start OTP and then ask only for the six-digit code from email.
- If the user pasted a magic-link token or `OPEN_TODO_TOKEN=...`, save it using the persistence rules below and then run a brief authenticated check.

Do not show setup tables, CLI availability checks, MCP config, shell exports, or multi-option setup plans during first-run setup. Do not mention the optional CLI unless the user asks for it or setup has succeeded and the CLI is already the best available persistence path.

Implementation steps:

1. Set `OPEN_TODO_API_URL` to `https://sanat-todo.sanat-thukral.workers.dev` unless the user provides another URL.
2. Ask for only the missing email or OTP input described above.
3. Start OTP when an email is available and an OTP has not already been sent:

```text
POST {OPEN_TODO_API_URL}/auth/otp/start
Content-Type: application/json

{ "email": "user@example.com" }
```

4. Ask only for the six-digit code from email.
5. Verify OTP:

```text
POST {OPEN_TODO_API_URL}/auth/otp/verify
Content-Type: application/json

{ "email": "user@example.com", "token": "123456" }
```

6. Save the returned `api_token` as `OPEN_TODO_TOKEN` in the user's local agent config or environment when persistence is available.
7. Send authenticated requests with `Authorization: Bearer OPEN_TODO_TOKEN`.
8. Run a brief authenticated check, then summarize only that Open Todo is connected.

If the user clicks an email magic link instead of providing an OTP, the callback page displays the one-time API token plus copyable setup snippets. Ask the user to paste only the token or env snippet needed for the current client.

## Persistence Rules

- Do not store tokens inside the skill source directory.
- Prefer the host agent's secret/config store when it exists.
- Use environment variables for ephemeral sessions, CI, and containers.
- If no persistent store exists, keep the token only for the current session and clearly say login will be needed again.
- Never print a saved token during normal reads.

## Task Operations

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

Use `sections.urgent`, `sections.thisweek`, and `sections.upcoming` from `brief` output to plan work. Fetch full task details only when notes or exact metadata are needed.

## Optional Existing CLI

If shell commands are available and `sanat-todo` is already installed, it is okay to use the CLI:

```sh
sanat-todo brief --status open --limit 50 --pretty --source codex
sanat-todo create "Short task title" --section thisweek --tag Agent --source codex
sanat-todo progress TASK_ID "Concrete progress and next step." --source codex --metadata '{"workspace":"local","branch":"main"}'
sanat-todo done TASK_ID --source codex
```

Do not install the CLI unless the user explicitly asks. Direct HTTP is the default path for this skill.

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
