---
name: open-todo
description: "Use Open Todo as a personal task list for agents. Covers simple email OTP setup, direct HTTP task operations, safe token handling, optional existing CLI usage, and MCP config."
---

# Open Todo

Open Todo is an agent-friendly todo system. Keep setup simple and safe:

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

1. Set `OPEN_TODO_API_URL` to `https://sanat-todo.sanat-thukral.workers.dev` unless the user provides another URL.
2. Ask the user for their email address.
3. Start OTP:

```text
POST {OPEN_TODO_API_URL}/auth/otp/start
Content-Type: application/json

{ "email": "user@example.com" }
```

4. Ask for the six-digit code from email.
5. Verify OTP:

```text
POST {OPEN_TODO_API_URL}/auth/otp/verify
Content-Type: application/json

{ "email": "user@example.com", "token": "123456" }
```

6. Save the returned `api_token` as `OPEN_TODO_TOKEN` in the user's local agent config or environment when persistence is available.
7. Send authenticated requests with `Authorization: Bearer OPEN_TODO_TOKEN`.

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
