# Roadmap

This is the current build plan for turning AutoClayBuilder from a knowledge base into a practical builder.

## Phase 1: Make Artifacts Testable

- Add a fake Clay manifest fixture set with fields, views, field groups, records, action fields, formulas, and extracted fields.
- Add unit tests for recursive redaction.
- Add unit tests for source-to-target field ID remapping.
- Extend parity scoring to report nested diffs instead of only field-level mismatch.
- Add view parity scoring: view names, order, field order, hidden state, filters, sorts, and grouping.

## Phase 2: Normalize Manifests

- Create a small Python package under `autoclaybuilder/`.
- Move redaction and parity code into importable modules.
- Add a normalized manifest schema with:
  - fields
  - views
  - field groups
  - table settings
  - seed records
  - endpoint notes
- Add CLI commands for `redact`, `score`, and `summarize`.

## Phase 2.5: Claygent Mastery Track

Build a public-safe Claygent knowledge base that makes Codex strong at choosing, prompting, testing, and operating Claygents.

Research and document:

- Claygent builder workflow: workspace-level agents, test cases, version history, publishing, and reuse across tables.
- Claygent versus Use AI versus Sculptor versus Functions: when each surface is the right tool.
- Model selection framework: Claygent-specific models, provider models, cost/credit tradeoffs, context needs, web research needs, reasoning depth, and speed.
- Advanced prompting patterns: role, goal, inputs, source strategy, search constraints, evidence requirements, output schema, confidence, refusal/unknown handling, and retry guidance.
- Output contracts: JSON-style fields, enums, citation/evidence fields, confidence scores, `needs_review`, and downstream-safe values.
- Test workflow: free test cases where available, sample rows, A/B prompt versions, pinned versus latest agent behavior, and regression checks before workspace-wide updates.
- Data and permission rules: connection access, documents/data sources, BYOK behavior, row-level data sent to models, and public-safe documentation practices.
- Cost controls: model choice, max output length, conditional runs, qualification gates before Claygent, and when to use cheaper deterministic providers first.
- Failure modes: hallucinated evidence, paywalled/password-protected sources, stale public pages, ambiguous companies, blank inputs, overbroad prompts, and non-parseable outputs.
- Implementation targets: `docs/claygent-playbook.md`, prompt templates, model selection matrix, and fake test cases for common GTM agents.

Seed sources to verify before writing the playbook:

- https://university.clay.com/docs/claygent-builder
- https://university.clay.com/docs/ai-in-clay
- https://university.clay.com/docs/ai-tokens
- https://university.clay.com/docs/writing-ai-prompts-in-clay
- https://university.clay.com/lessons/claygent-ai-web-scraper-limitless-research
- https://university.clay.com/lessons/enriching-with-claygent
- https://university.clay.com/docs/access-settings-for-connections

## Phase 3: Payload Builders

Add payload builders for the common Clay column types:

- text/manual fields
- formulas
- extracted fields
- `http-api-v2`
- `use-ai`
- `chat-gpt-schema-mapper`
- Smartlead add/lookup/forward patterns
- Slack notification actions
- Google Sheets append/update actions
- HubSpot lookup/create/update actions
- lookup/write to other Clay tables

Every builder should include a save-without-run path and a readback verification check.

## Phase 4: Browser Verification Recipes

- Document browser-use recipes as deterministic steps.
- Add optional harness scripts for:
  - opening a table with a selected Chrome profile
  - checking auth state
  - capturing safe drawer screenshots
  - validating formula editor save behavior
- Keep browser automation optional and clearly separated from API-only logic.

## Phase 5: End-To-End Scratch Clone

Build a guarded CLI that can:

1. read a redacted or live source manifest
2. create a scratch table
3. recreate safe fields/views/rows
4. save without running action columns
5. fetch target readback
6. score parity
7. write a public-safe run summary

Live mutation support should require explicit flags and should default to scratch-only behavior.
