# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A Claude Code plugin that installs as `/paper-quiz` — a skill that turns any academic paper (local PDF or URL) into a self-contained browser quiz. No server, no build step, no external dependencies.

Install: `claude plugins install chrishalkias/paper-quiz`

## Plugin structure

This follows the Claude Code plugin spec:

```
.claude-plugin/
  plugin.json           # Plugin metadata
  marketplace.json      # Marketplace listing
skills/
  paper-quiz/
    SKILL.md            # Orchestration steps for the /paper-quiz skill
    quiz-template.html  # The HTML quiz UI with QUIZ_DATA_PLACEHOLDER
agents/
  tester.md             # Claude Opus subagent: paper text → quiz JSON
```

The root `SKILL.md` and `quiz-template.html` are legacy copies — the canonical files live under `skills/paper-quiz/`.

## How the skill executes

1. `SKILL.md` orchestrates: reads input (Read for local PDF, WebFetch for URL)
2. Spawns `paper-quiz:tester` subagent (Opus, no tools) with raw paper text only
3. Tester returns a raw JSON object — no markdown fences
4. Skill globs for `**/paper-quiz/quiz-template.html`, reads it
5. Replaces the literal string `"QUIZ_DATA_PLACEHOLDER"` with the JSON (must be valid JS, not a string)
6. Writes `/tmp/paper-quiz-<timestamp>.html` and opens it

The placeholder replacement is the critical injection point. The result must be `const QUIZ_DATA = {...};` not `const QUIZ_DATA = "{...}";`.

## Quiz JSON schema

```json
{
  "paper_title": "string",
  "questions": [
    { "id": 1, "type": "mcq", "question": "...", "options": ["A","B","C","D"], "correct": 0, "explanation": "..." },
    { "id": 8, "type": "open", "question": "...", "model_answer": "...", "key_points": ["...", "..."] }
  ]
}
```

5–7 MCQ + 3 open-ended. IDs must be sequential from 1.

## Developing / testing

No build process. Edit files directly.

To test end-to-end: run `/paper-quiz <pdf-or-url>` in a Claude Code session with the plugin installed (or locally loaded).

To test the HTML template in isolation: inject sample JSON manually by replacing `"QUIZ_DATA_PLACEHOLDER"` in `skills/paper-quiz/quiz-template.html` and opening it in a browser.

To test the tester agent prompt: spawn it directly via `Agent` tool with `subagent_type: "paper-quiz:tester"` and a paper text payload — verify the response is raw JSON only.

## Deployment

After changes, commit and push to `https://github.com/chrishalkias/paper-quiz`. Users reinstall to pick up updates. No CI configured.
