---
name: open-todo
description: "Use Open Todo as an HTTP-first personal task list for agents. Covers first-time email OTP setup, token configuration, reading tasks, creating tasks, updating status, progress checkpoints, events, and MCP setup."
---

# Open Todo

Open Todo is an HTTP-first todo API for agents. Prefer direct HTTP calls. Use MCP only when the current agent already has the Open Todo MCP server configured.

## Configuration

Look for:

- `OPEN_TODO_API_URL`: base Worker URL, for example `https://open-todo.example.workers.dev`.
- `OPEN_TODO_TOKEN`: bearer token returned by onboarding.

If either value is missing, run onboarding before reading or writing tasks.

## Onboarding

1. Ask the user for their email address.
2. Call:

```text
POST {OPEN_TODO_API_URL}/auth/otp/start
Content-Type: application/json

{ "email": "user@example.com" }
```

3. Ask for the six-digit OTP from email.
4. Call:

```text
POST {OPEN_TODO_API_URL}/auth/otp/verify
Content-Type: application/json

{ "email": "user@example.com", "token": "123456" }
```

5. Save the returned `api_token` as `OPEN_TODO_TOKEN` in the user's local agent config or environment when the environment supports persistent config. If persistence is unavailable, keep it for the current session and tell the user what was not persisted.
6. Use the token as `Authorization: Bearer OPEN_TODO_TOKEN`.

## Fast Read

Use the brief endpoint first:

```text
GET {OPEN_TODO_API_URL}/tasks/brief?status=open&limit=50
Authorization: Bearer OPEN_TODO_TOKEN
```

Use `sections.urgent`, `sections.thisweek`, and `sections.upcoming` to plan work. Fetch full tasks only when notes or exact metadata are needed.

## Writes

Create a task:

```text
POST {OPEN_TODO_API_URL}/tasks
Authorization: Bearer OPEN_TODO_TOKEN
Content-Type: application/json

{
  "title": "Short task title",
  "notes": "Optional context",
  "section": "upcoming",
  "tag": "Optional"
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

Complete a task:

```text
POST {OPEN_TODO_API_URL}/tasks/TASK_ID/complete
Authorization: Bearer OPEN_TODO_TOKEN
X-Todo-Source: codex
```

Or record final progress and complete in one call:

```json
{
  "summary": "Finished implementation and tests.",
  "status": "done",
  "source": "codex"
}
```

## Source Attribution

Always identify the client on writes with either `X-Todo-Source` or a JSON `source` field. Use stable values such as `codex`, `claude`, `cursor`, `raycast`, `manual-api`, or `cron`. Put branch names, model names, run IDs, and workspaces in `metadata`.

## MCP

If the user wants MCP config, use:

```json
{
  "mcpServers": {
    "open-todo": {
      "url": "https://OPEN_TODO_API_URL/mcp",
      "headers": {
        "Authorization": "Bearer OPEN_TODO_TOKEN"
      }
    }
  }
}
```
