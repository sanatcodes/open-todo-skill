# Auto Todo Skill

An agent skill for setting up and operating Auto Todo, an HTTP-first todo list for agents and scripts.

## Install

```sh
npx skills@latest add sanatcodes/auto-todo-skill --skill auto-todo --global
```

Then ask your agent:

```text
Set up Auto Todo
```

The skill installs into supported coding agents through Vercel's `skills` CLI. It walks through email OTP onboarding, stores the returned bearer token when the agent environment supports persistent config, and uses direct HTTP for task operations.

It does not ask the agent to install extra binaries by default. If the `auto-todo` CLI is already installed, the skill may use it as an optional convenience path.

For a specific agent:

```sh
npx skills@latest add sanatcodes/auto-todo-skill --skill auto-todo --global --agent codex --yes
```

Replace `codex` with another supported agent name such as `claude-code` or `cursor`.

## What This Repo Contains

This public repo contains only the installable skill:

- `.agents/skills/auto-todo/SKILL.md`
- `.agents/skills/auto-todo/agents/openai.yaml`

It does not contain the private Auto Todo service implementation.

## API Configuration

The skill expects:

- `AUTO_TODO_API_URL`: the deployed Auto Todo Worker URL. If it is missing, the skill uses the hosted default.
- `AUTO_TODO_TOKEN`: the bearer token returned by OTP onboarding.

If either value is missing, the skill starts onboarding through:

- `POST /auth/otp/start`
- `POST /auth/otp/verify`

Normal task operations use the JSON HTTP API.
