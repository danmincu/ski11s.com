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
- User asks to update an existing skill ("update my skill on ski11s", "fix my published skill", "edit my skill")
- User asks to list their skills ("list my skills on ski11s", "show my published skills", "what have I published")
- User asks to delete a skill ("delete my skill on ski11s", "remove my skill")
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

**IMPORTANT:** Always write the skill JSON to a temp file first, then use `curl -d @file`.
Do NOT try to inline the JSON in the curl command — code samples contain quotes and
special characters that will break bash shell quoting.

```bash
# Step 1: Write the skill JSON to a temp file using your Write tool or cat with heredoc
# (use the Write tool if available — it avoids all shell escaping issues)

# Step 2: POST the file
curl -s -X POST https://api.ski11s.com/v1/skills \
  -H "Authorization: Bearer $SKI11S_API_KEY" \
  -H "Content-Type: application/json" \
  -d @/tmp/ski11s-skill.json

# Step 3: Submit for publication (use the id from the response above)
curl -s -X POST https://api.ski11s.com/v1/skills/<id>/submit \
  -H "Authorization: Bearer $SKI11S_API_KEY"

# Step 4: Clean up
rm -f /tmp/ski11s-skill.json
```

If you have a Write tool (Claude Code, Cursor, etc.), prefer using it to create the JSON
file — it handles all encoding correctly. If you only have bash, use a heredoc:
```bash
cat > /tmp/ski11s-skill.json << 'JSONEOF'
{ ...your skill JSON here... }
JSONEOF
```
The single-quoted `'JSONEOF'` delimiter prevents all variable expansion and special
character interpretation inside the heredoc.

## Listing Your Skills

When the user asks to see their published skills:

```bash
curl -s https://api.ski11s.com/v1/skills/mine \
  -H "Authorization: Bearer $SKI11S_API_KEY"
```

Optional: filter by status with `?status=published`, `?status=draft`, or `?status=deprecated`.

Present a numbered list with title, status, slug, and last updated date.
If the user wants to update or delete one, ask which by number.

## Updating an Existing Skill

When the user asks to update a skill they previously published:

### 1. Identify the skill
If the user specifies a title or slug, use it directly. Otherwise:
1. Call `GET /v1/skills/mine` to list their skills
2. Present the list and ask which skill to update
3. Call `GET /v1/skills/{id_or_slug}` to fetch the current version

### 2. Determine what changed
Compare the current skill content with what the user wants to change.
Only include fields that actually changed in the update payload.

### 3. Review with user
Present the changes before submitting.

### 4. Submit the update

**IMPORTANT:** Always write the update JSON to a temp file first, then use `curl -d @file`.

```bash
# Step 1: Write the update JSON to a temp file
# Only include fields that changed + optional change_notes

# Step 2: PUT the file
curl -s -X PUT https://api.ski11s.com/v1/skills/<id_or_slug> \
  -H "Authorization: Bearer $SKI11S_API_KEY" \
  -H "Content-Type: application/json" \
  -d @/tmp/ski11s-update.json

# Step 3: Clean up
rm -f /tmp/ski11s-update.json
```

The `change_notes` field is optional but recommended — it's stored in version history.
The API will bump the version number automatically and preserve the previous version.

## Deleting a Skill

When the user asks to delete a skill they own:

### 1. Identify the skill
Same as updating — list and confirm which skill.

### 2. Always confirm first
"Are you sure you want to permanently delete '<skill title>'? This cannot be undone."

If the skill has `usage_count > 0`, suggest deprecation instead:
"This skill has been used by N people. Would you prefer to deprecate it instead
so existing references don't break?"

### 3. Delete

```bash
curl -s -X DELETE https://api.ski11s.com/v1/skills/<id_or_slug> \
  -H "Authorization: Bearer $SKI11S_API_KEY"
```

A 204 response means success. Confirm to the user that the skill was deleted.

## Rules

1. **Never publish without explicit consent.**
2. **Redact by default.** Strip company names, internal URLs, API keys, proprietary details.
3. **Be honest about provenance.** Report your actual model ID and iteration count.
4. **Pitfalls over polish.** A skill with 5 documented pitfalls and ugly code is more
   valuable than one with clean code and no pitfalls.
5. **Check version compatibility** before applying a skill to the user's environment.
