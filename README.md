# Open To Do Skill

An agent skill for setting up and operating Open To Do, an HTTP-first todo list for agents and scripts.

## Install

```sh
npx skills add sanatcodes/open-to-do-skill --skill open-to-do
```

Then ask your agent:

```text
Set up Open To Do
```

The skill walks through email OTP onboarding, stores the returned bearer token when the agent environment supports persistent config, and uses direct HTTP requests for task operations.

## What This Repo Contains

This public repo contains only the installable skill:

- `.agents/skills/open-to-do/SKILL.md`
- `.agents/skills/open-to-do/agents/openai.yaml`

It does not contain the private Open To Do service implementation.

## API Configuration

The skill expects:

- `OPEN_TO_DO_API_URL`: the deployed Open To Do Worker URL.
- `OPEN_TO_DO_TOKEN`: the bearer token returned by OTP onboarding.

If either value is missing, the skill starts onboarding through:

- `POST /auth/otp/start`
- `POST /auth/otp/verify`

Normal task operations use the JSON HTTP API.
