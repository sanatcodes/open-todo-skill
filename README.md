# Open Todo Skill

An agent skill for setting up and operating Open Todo, an HTTP-first todo list for agents and scripts.

## Install

```sh
npx skills add sanatcodes/open-todo-skill --skill open-todo
```

Then ask your agent:

```text
Set up Open Todo
```

The skill walks through email OTP onboarding, stores the returned bearer token when the agent environment supports persistent config, and uses direct HTTP requests for task operations.

## What This Repo Contains

This public repo contains only the installable skill:

- `.agents/skills/open-todo/SKILL.md`
- `.agents/skills/open-todo/agents/openai.yaml`

It does not contain the private Open Todo service implementation.

## API Configuration

The skill expects:

- `OPEN_TODO_API_URL`: the deployed Open Todo Worker URL.
- `OPEN_TODO_TOKEN`: the bearer token returned by OTP onboarding.

If either value is missing, the skill starts onboarding through:

- `POST /auth/otp/start`
- `POST /auth/otp/verify`

Normal task operations use the JSON HTTP API.
