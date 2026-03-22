# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

This is an **Agent Skill** package for [ski11s.com](https://ski11s.com) — a registry of battle-tested solutions discovered through LLM-assisted debugging. The repo contains no application code to build/test/lint; it is a set of instruction files consumed by LLM coding tools (Claude Code, Codex CLI, Copilot, Cursor, etc.).

## Repo Structure

- `SKILL.md` — The main skill definition (YAML frontmatter + markdown). This is what LLM tools load to learn the skill's behavior.
- `references/api-reference.md` — Full API docs for `api.ski11s.com/v1`. Loaded on demand when the skill needs to make API calls.

## Key Concepts

- **Searching**: `POST /v1/skills/search` with a natural-language query. Present results, let user pick, fetch full skill via `GET /v1/skills/{id}`, then apply it.
- **Publishing**: Extract solution details from conversation, let user review/redact, `POST /v1/skills` to create draft, then `POST /v1/skills/{id}/submit` for curation.
- **Feedback loop**: After applying a skill, report selection (`/search/{search_id}/select`) and outcome (`/search/{search_id}/feedback`).

## Auth

All API calls require `Authorization: Bearer $SKI11S_API_KEY`. The key comes from the environment variable or the project's `.env` file. If missing, direct users to `ski11s.com/signup`.

## Rules When Editing This Skill

1. Never publish user solutions without explicit consent.
2. Redact company names, internal URLs, API keys, and proprietary details by default.
3. Pitfalls are the most valuable part of a skill — prioritize documenting what went wrong over polishing code.
4. Always check `framework_versions` compatibility before applying a skill.
5. Report honest provenance (actual model ID, iteration count).
