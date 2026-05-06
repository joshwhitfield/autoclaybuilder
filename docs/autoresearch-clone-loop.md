# Clay Autoresearch Clone Loop

This loop is for reverse-engineering a Clay table and recreating it in scratch tables until config parity is perfect or a real blocker is proven.

## Safety Rules

- Do not mutate the source table.
- Use scratch workbooks and scratch tables for every attempt.
- Copy only the first 10 source rows unless the operator explicitly raises the limit.
- Save columns without running first where possible.
- Turn auto-run off before constructing action-heavy scratch tables.
- Run external or integration columns only on the scratch sample rows when validation requires it.
- Redact cookies, bearer tokens, API keys, passwords, webhook URLs, auth headers, and credential-like values before writing artifacts.
- Do not copy live source `sourceIds` bindings into a scratch target source field; clear them and insert controlled seed rows instead.

## Source Manifest

Extract the source through the logged-in Clay frontend API:

```text
GET /v3/tables/:tableId?extraDataViewId=:viewId&includeExtraData=true
```

The redacted manifest should preserve:

- workspace/table/view identifiers only when fake or intentionally redacted
- ordered fields
- field names, types, formulas, extracted-field parents, and action metadata
- `typeSettings.inputsBinding`, `inputFieldIds`, conditional run formulas, batch settings, and rate limits
- view order, field order, hidden/visible state, filters, sorts, and grouping
- first 10 seed values needed to recreate input rows
- endpoint notes with method and path template
- unknowns that require UI inspection

## Attempt Paths

Run attempts as separate folders with clear status:

- `api-first`: create and configure as much as possible through frontend APIs; use browser automation for missing endpoints and verification.
- `browser-first`: recreate through Clay UI workflows, then verify by fetching the API manifest.
- `duplicate-then-patch`: use Clay's duplicate path when allowed, then patch and compare mismatches.
- `from-scratch-api-browser`: use an API-created scratch base, then browser automation for columns or settings the API cannot reproduce.

## Rebuild Order

1. Fetch source readback.
2. Create scratch table, preferably with no default views when exact view parity matters.
3. Patch table settings so auto-run is off.
4. Create fields and build the source-to-target field map.
5. Patch fields after the complete map exists so nested references can be remapped.
6. Recreate field groups with target field IDs.
7. Recreate views with remapped field keys and embedded references.
8. Insert only the selected seed rows.
9. Fetch target readback and score config parity.
10. Run selected records and selected fields only after config looks correct.

## Parity Scoring

Score against fresh source and target readbacks from the same endpoint. Compare:

- field count, order, names, types, formulas, extracted-field config, action metadata, bindings, guards, batch settings, and reusable auth/account references
- view count, names, order, field order, hidden-column state, filters, sorts, and grouping
- field group names, ordering, and live field membership
- first 10 seed rows
- UI evidence for edit drawers, formulas, column menus, run controls, and representative cell inspection

Runtime output equality is separate evidence. A config-perfect scratch table can still need runtime logging if external APIs are async, permissioned, or rate limited.

## Sample Run Protocol

Use a selected-field, selected-record run when possible:

```text
PATCH /v3/tables/:tableId/run
```

Then poll:

```text
GET /v3/workspaces/:workspaceId/tables/:tableId/fields/runstatus
```

Interpretation guidance:

- `SUCCESS`: the row/action completed.
- `SUCCESS_NO_DATA`: the provider completed but found no matching data; this can be acceptable.
- `QUEUED`: still waiting to start.
- `AWAITING_CALLBACK`: async provider callback pending; wait or inspect provider behavior before declaring failure.
- `ERROR_BLANK_TOKEN`: a required input token was blank for that row; inspect required inputs and run conditions before rerun.

## Run Log Format

Use one append-only row per attempt:

```tsv
timestamp	run_tag	sprint	attempt	path	target_table_url	parity_score	status	notes
2026-05-03T18:00:00Z	clay-clone-benchmark	1	attempt-001	api-first	https://app.clay.com/...	0.82	partial	missing action payload fields
```

Recommended statuses:

- `perfect_config_parity`
- `partial`
- `blocked`
- `failed`
- `abandoned_by_better_path`

## Stop Conditions

Stop when one of these is true:

- perfect config parity is proven by source/target manifests, score report, row seed check, and browser evidence
- sample runtime validation proves the workflow is ready to scale, when runtime validation is part of the task
- a blocker is proven with an exact failing API/UI step, fresh readback, redacted request/response or screenshot, and evidence that the remaining known paths cannot bypass it
