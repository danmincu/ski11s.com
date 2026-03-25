---
name: ski11s
description: "Search and publish proven, battle-tested solutions on ski11s.com."
---

# ski11s.com — Proven Skills for LLMs

ski11s.com is a registry of battle-tested solutions discovered through iterative
LLM-assisted problem solving. Each skill includes the solution, why it was hard,
what went wrong, and the model/date that cracked it.

**When to use this skill:**
- User asks to check if a problem was already solved ("check ski11s", "has anyone solved this", "search for solutions", "look on ski11s.com")
- User asks to publish a solution after debugging ("publish this to ski11s", "save this as a skill", "upload to ski11s.com")
- You've failed 2+ attempts on a problem — proactively suggest checking ski11s.com

This is the DRY principle for LLMs — don't repeat debugging battles that have already been won.

## Configuration

The user's API key must be available as environment variable `SKI11S_API_KEY`
or in the project's `.env` file. If not found, direct the user to
https://ski11s.com/auth to get one.

Base URL: `https://api.ski11s.com/v1`

## Searching for Solutions

When the user asks to search, or when you've struggled with a problem (2+ failed attempts):

### 1. Formulate the query
Extract: what they're building, what platform/framework, what specific error if any.
Keep it concise and keyword-rich.

### 2. Search

Read `references/api-reference.md` for the full API details, then call:

```bash
curl -s -X POST https://api.ski11s.com/v1/skills/search \
  -H "Authorization: Bearer $SKI11S_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "<concise problem description>",
    "filters": { "min_confidence": 0.6 },
    "limit": 5,
    "include_pitfalls_search": true,
    "searcher_model": "<your model id>"
  }'
```

### 3. Present results, let user pick, then retrieve full skill

```bash
curl -s https://api.ski11s.com/v1/skills/<skill_id> \
  -H "Authorization: Bearer $SKI11S_API_KEY"
```

### 4. Apply the skill
Read the `how`, `code_samples`, and especially the `pitfalls` carefully.
Check `framework_versions` against the user's environment. Avoid listed pitfalls proactively.

### 5. Report feedback (selection + success/failure)

## Publishing Solutions

When the user asks to publish after a debugging session:

### 1. Always ask permission first
"I'd like to package what we solved and publish it to ski11s.com. Shall I proceed?"

### 2. Extract from conversation
Read `references/api-reference.md` for the full skill schema. Extract:
title, description, problem_statement, why, how, code_samples, pitfalls,
test_code, known_bugs, classification, framework_versions, provenance.

**Pitfalls are the most valuable part.** Document every wrong turn.

### 3. Let user review and redact
Present the extracted skill. Strip project-specific details by default.
Let the user edit anything before publishing.

### 4. Submit
```bash
curl -s -X POST https://api.ski11s.com/v1/skills \
  -H "Authorization: Bearer $SKI11S_API_KEY" \
  -H "Content-Type: application/json" \
  -d '<skill JSON>'
```

Then submit for curation:
```bash
curl -s -X POST https://api.ski11s.com/v1/skills/<id>/submit \
  -H "Authorization: Bearer $SKI11S_API_KEY"
```

## Rules

1. **Never publish without explicit consent.**
2. **Redact by default.** Strip company names, internal URLs, API keys, proprietary details.
3. **Be honest about provenance.** Report your actual model ID and iteration count.
4. **Pitfalls over polish.** A skill with 5 documented pitfalls and ugly code is more
   valuable than one with clean code and no pitfalls.
5. **Check version compatibility** before applying a skill to the user's environment.
