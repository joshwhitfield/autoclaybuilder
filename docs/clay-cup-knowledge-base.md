# Clay Cup Knowledge Base

Updated: 2026-05-06

This is the living operating manual for fast, public-safe Clay table building. It should capture reusable knowledge, not raw customer data, live manifests, screenshots, cookies, bearer tokens, API keys, webhook URLs, or one-off table IDs.

## North Star

Given a plain-English GTM goal, quickly produce a Clay workflow that is:

- accurate enough to run on a small sample
- explainable enough for a judge, operator, or sales team to trust
- cheap enough to avoid wasting credits on unqualified rows
- structured enough to scale after sample verification
- documented enough that the next agent can continue without rediscovery

## Default Build Loop

1. Translate the goal into final row type: company, person, job, location, signal, or cross-table workflow.
2. Define proof fields before enrichment: why the row qualifies, source URLs, confidence, and disqualifiers.
3. Put cheap filters before expensive enrichments.
4. Build in a scratch table with auto-run off.
5. Recreate or add fields in logical groups: source, qualification, enrichment, scoring, outreach/export.
6. Remap every source field ID in formulas, action bindings, extracted fields, views, filters, sorts, run conditions, and table settings.
7. Create run-control views: all rows, ready to enrich, errors, fully enriched, export-ready.
8. Run a small sample, usually 10 rows, before scaling.
9. Inspect run status, blanks, errors, and evidence fields.
10. Scale only after the sample explains itself.

## Clay Capability Map

Use these buckets when converting a goal into a table design.

- Sources: Find Companies, Find People, Find Jobs, Google Maps, CSV import, web search, webhooks, scheduled sources, CRM or data warehouse imports.
- Enrichment: company and person enrichments, work email waterfalls, personal email waterfalls, phone waterfalls, provider-specific enrichments, HTTP API, AI web research, Claygents.
- Transform: formulas, conditional logic, lookup rows, write/send to other table, formatters, lead scoring, column group templates, run conditions.
- Signals: web intent, promotions, new hires, news and fundraising, job changes, custom signals, table alerts.
- Export: CRM writes, sequencer tools, Slack or Teams, databases, ads audiences, webhooks, HTTP actions, cross-table transfer.
- AI: Use AI action, Claygent builder, formula generator, API generator, Debug with AI, Sculptor analyst mode.

## Sculptor Operating Model

Use Sculptor as a strategist and debugger, not as the only builder.

Strong fits:

- first-pass table ideas from a plain-English GTM goal
- AI column setup and prompt iteration
- formula generation and formula debugging
- table analysis through analyst mode
- recommendations for enrichments or Claygent prompts

Known boundaries from current Clay docs:

- Full support: AI columns and formulas.
- Partial support: enrichments, waterfalls, credit estimation, CRM/webhook reads, signals reads.
- Limited or not yet supported: full source creation for every source type, filters and sorting, message drafting, signals tables, direct CRM writes, existing-table writes, and one-shot complete workflow creation.

Practical rule: ask Sculptor for strategy, formulas, and debugging context; use API/manual/browser control for precise table mutation and verification.

## Public-Safe API Learnings

These are frontend API patterns observed from live rebuild work. Treat them as implementation notes, not official stable API documentation.

Core readback:

```text
GET /v3/tables/:tableId?extraDataViewId=:viewId&includeExtraData=true
GET /v3/workspaces/:workspaceId/tables/:tableId/fields/runstatus
```

Scratch creation:

```text
POST /v3/tables
```

Useful creation payload pattern:

```json
{
  "name": "Scratch rebuild - descriptive name",
  "workspaceId": "workspace_id",
  "workbookId": "workbook_id",
  "template": "no_views",
  "type": "company"
}
```

Auto-run guard:

```text
PATCH /v3/tables/:tableId
```

```json
{
  "tableSettings": {
    "AUTO_RUN_ON": false,
    "AUTO_RUN_MODE": "keep_existing",
    "HAS_SCHEDULED_RUNS": false
  }
}
```

Field groups:

```text
POST /v3/tables/:tableId/fields/group
```

```json
{
  "name": "Group name",
  "fieldIds": ["target_field_id"]
}
```

Views:

```text
POST /v3/tables/:tableId/views
```

Remap all view field dictionary keys and every embedded field ID before creating a target view.

Run selected records and fields:

```text
PATCH /v3/tables/:tableId/run
```

```json
{
  "fieldIds": ["field_id_to_run"],
  "runRecords": {
    "recordIds": ["record_id_to_run"],
    "tableId": "table_id"
  },
  "forceRun": true,
  "callerName": "codex_sample_run",
  "skipActionFieldRuns": false,
  "skipTriggerSetting": false
}
```

Observed response shape:

```json
{
  "recordCount": 10,
  "runMode": "INDIVIDUAL"
}
```

Verify run progress through fresh `fields/runstatus` readbacks. Async providers can remain in `AWAITING_CALLBACK`; action errors such as `ERROR_BLANK_TOKEN` need row/input inspection before rerun.

## Clone Hazards

- Do not copy live source field `typeSettings.sourceIds` into a scratch target. It can materialize or bind many source records unexpectedly. For scratch rebuilds, clear source IDs and insert only controlled seed rows.
- Do not leave auto-run on while constructing action-heavy tables.
- Do not treat write responses as final truth. Fetch a fresh table readback and compare persisted config.
- Do not compare field IDs literally after recreation. New fields get new IDs; build a source-to-target field map.
- Do not forget embedded IDs inside formulas, inputsBinding, inputFieldIds, extracted fields, conditional-run formulas, view settings, filters, sorts, and table settings.
- Do not store scalar cell values as `{ "value": ... }`; Clay record creation expects direct scalar cell values.
- Do not run HTTP/API columns with placeholder credentials. Verify affected cell count is zero before handing off placeholders.
- Do not treat runtime output equality as the same as config parity. Runtime results can depend on async providers, rate limits, permissions, and current external data.

## Competition Patterns

Winning Clay tables should make the answer obvious. Favor these shapes:

- Evidence-first scoring: score plus reason, trigger, source URL, and recommended action.
- Multi-signal qualification: combine firmographics, hiring, news, tech stack, web intent, and persona fit before contact enrichment.
- Credit-aware routing: qualify accounts first, then find people, then waterfall contact details only for rows worth pursuing.
- Human-ready views: `ready_to_review`, `needs_fix`, `high_priority`, `export_ready`.
- Cross-table architecture when useful: company source table, people/contact table, signal table, and export/routing table.
- Explainable disqualification: include why a row should not be pursued, not only why it should.

## Future Update Protocol

When a new Clay lesson is learned:

1. Decide whether it is a capability, endpoint, failure mode, build pattern, or competition tactic.
2. Convert live details into public-safe templates.
3. Add exact method/path templates only when verified by readback or live behavior.
4. Keep raw requests, raw manifests, table IDs, workspace IDs, credentials, and screenshots out of git.
5. Add the note to this knowledge base, then link deeper details from a focused doc when needed.
6. If the learning changes clone behavior, update `docs/frontend-api-patterns.md` or `docs/autoresearch-clone-loop.md` too.

## Useful Official Docs

- Sculptor: https://university.clay.com/docs/sculptor
- AI in Clay: https://university.clay.com/docs/ai-in-clay
- Find: https://university.clay.com/docs-topics/find
- Enrich: https://university.clay.com/docs-topics/enrich
- Transform: https://university.clay.com/docs-topics/transform
- Signals: https://university.clay.com/docs-topics/signals
- Export: https://university.clay.com/docs-topics/export
