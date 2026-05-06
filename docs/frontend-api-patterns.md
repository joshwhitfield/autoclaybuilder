# Clay Frontend API Patterns

These notes come from live Clay table rebuild and scratch-table benchmark work. Treat them as implementation guidance, not official Clay API documentation.

## Core Read Endpoints

```text
GET /v3/tables/:tableId?extraDataViewId=:viewId&includeExtraData=true
GET /v3/tables/:tableId/count
GET /v3/sources?tableId=:tableId
GET /v3/workspaces/:workspaceId/tables/:tableId/fields/runstatus
GET /v3/workspaces/:workspaceId/app-accounts
```

Use the table readback as the authority. Some writes return HTTP 200 while Clay normalizes or rejects parts of the persisted config.

## Scratch Creation

Observed useful paths:

```text
POST /v3/workbooks
POST /v3/tables
POST /v3/tables/:sourceTableId/duplicate/
POST /v3/sources/create-cpj-table
```

For exact API-visible view parity, `POST /v3/tables` with `template: "no_views"` can create a table with no preconfigured system views. Ordinary table creation may add API-visible preconfigured views that cannot be deleted.

A public-safe scratch creation shape:

```json
{
  "name": "Scratch rebuild - descriptive name",
  "workspaceId": "workspace_id",
  "workbookId": "workbook_id",
  "template": "no_views",
  "type": "company"
}
```

Turn auto-run off before adding action-heavy fields:

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

## Native CPJ Source Creation

For native "find list" sources, the verified frontend path in this environment was:

```text
POST /v3/sources/create-cpj-table
```

The same path without `/v3` returned 404, so keep the versioned prefix in payload builders.

Public-safe shape for a company source:

```json
{
  "workspaceId": "workspace_id",
  "workbookId": "workbook_id",
  "workbookName": "Scratch workbook",
  "workbookAndTableNameOverride": "Source table name",
  "sourceNameOverride": "Find SaaS companies",
  "cpjConfig": {
    "type": "companies",
    "clientSettings": {
      "tableType": "company"
    },
    "destinationTableSetupMode": "add_basic_fields",
    "basicFields": ["company_name", "domain", "linkedin_company_url"],
    "typeSettings": {
      "name": "Find companies",
      "actionKey": "find-lists-of-companies-with-mixrank-source",
      "actionPackageId": "e251a70e-46d7-4f3a-b3ef-a211ad3d8bd2",
      "previewActionKey": "find-lists-of-companies-with-mixrank-source-preview",
      "recordsPath": "companies",
      "idPath": "linkedin_company_id",
      "scheduleConfig": {
        "runSettings": "once"
      },
      "disableTriggerOnCreate": false,
      "disableTriggerOnUpdate": false,
      "hasEvaluatedInputs": true,
      "inputs": {
        "number_of_records": 25,
        "keywords": ["saas"],
        "start_from_method": "query",
        "startFromCompanyType": "semantic_description"
      }
    }
  }
}
```

Observed input gotchas:

- `start_from_method` accepted values such as `query`, `CsvOfCompanies`, `url`, and `CompanyAudienceSegment`; `csv_of_companies` was rejected.
- `startFromCompanyType` accepted `company_identifier` and `semantic_description`; `lookalikes` was rejected.
- Boolean-looking source filters such as `has_resolved_domain` and `resolved_domain_is_live` expected string values like `Yes` or `No` when present.
- Successful source creation can return `initialRecordIds` and `initialRecords` in table readback extra data. Treat those as live data and do not commit them.

## Field Creation

Use:

```text
POST /v3/tables/:tableId/fields
PATCH /v3/tables/:tableId/fields/:fieldId
DELETE /v3/tables/:tableId/fields/:fieldId
```

Rules observed in benchmark work:

- Create action columns as actions with the full action payload. Do not create a text field and later promote it into an action.
- `PATCH` is useful for same-action updates such as names, run guards, mappings, and settings.
- Clay rejects switching a saved action to a different `actionKey`.
- New field creation ignores supplied source field IDs, so generated field IDs must be mapped.
- Always remap field IDs inside formulas, `inputsBinding`, `inputFieldIds`, extracted fields, conditional-run formulas, view settings, and table settings.
- Do not copy live source field `typeSettings.sourceIds` into a scratch target. It can bind or materialize many source records unexpectedly. For controlled scratch rebuilds, clear source IDs and add only the selected seed rows.

## Action Account Binding

Action fields can be enabled but still show `settingsError: MISSING_AUTH` if the field has no selected connection. Use:

```text
GET /v3/workspaces/:workspaceId/app-accounts
PATCH /v3/tables/:tableId/fields/:fieldId
```

Patch the field with the chosen connection in `typeSettings.authAccountId`, then fetch the table again to verify the settings error cleared. Never commit real auth account IDs; examples should use placeholders like `aa_provider`.

## Views And Ordering

Observed paths:

```text
POST /v3/tables/:tableId/views
PATCH /v3/tables/:tableId/views/:viewId/fields/:fieldId
```

For view creation, remap the `fields` dictionary keys and every embedded field ID before posting to the target table. This includes hidden/visible state, filters, sorts, grouping, and any field references inside view settings.

Sequential moves with `beforeFieldId` or `afterFieldId` are a reliable fallback when bulk field reorder paths fail.

Rebuild visible/hidden state from the API, not just what the UI currently shows. Export views often hide many implementation fields.

## Records

```text
POST /v3/tables/:tableId/records
DELETE /v3/tables/:tableId/records
```

Clay expects direct scalar cell values:

```json
{
  "records": [
    {
      "cells": {
        "f_target": "example value"
      }
    }
  ]
}
```

Do not wrap values as `{ "value": "example" }`; that stores nested values and can cause coercion errors.

If an accidentally copied source binding materializes too many records, delete extras with `DELETE /v3/tables/:tableId/records` in batches and then patch the source field binding before continuing.

## Field Groups

Generic groups can be recreated with:

```text
POST /v3/tables/:tableId/fields/group
POST /v3/tables/:tableId/fields/group/:groupId
```

The create payload requires a `fieldIds` array:

```json
{
  "name": "Group name",
  "fieldIds": ["f_target"]
}
```

Map generated group IDs separately. Source `fieldGroupMap` can contain stale field references that are not present in the live `fields[]` array, so score only live refs.

## Run Control

Run selected fields and selected records with:

```text
PATCH /v3/tables/:tableId/run
```

Minimum observed payload shape:

```json
{
  "fieldIds": ["f_target_action_or_formula"],
  "runRecords": {
    "recordIds": ["r_target_seed_row"],
    "tableId": "target_table_id"
  },
  "forceRun": true,
  "callerName": "codex_sample_run",
  "skipActionFieldRuns": false,
  "skipTriggerSetting": false
}
```

Alternative run selectors include `viewId` and `viewIdTopRecords` inside `runRecords` when the goal is to run a view slice instead of explicit record IDs.

Observed response shape:

```json
{
  "recordCount": 10,
  "runMode": "INDIVIDUAL"
}
```

After a run, poll:

```text
GET /v3/workspaces/:workspaceId/tables/:tableId/fields/runstatus
```

Useful statuses include `SUCCESS`, `SUCCESS_NO_DATA`, `QUEUED`, `AWAITING_CALLBACK`, and action-specific errors such as `ERROR_BLANK_TOKEN`. Persistent `AWAITING_CALLBACK` means wait or inspect provider behavior. `ERROR_BLANK_TOKEN` usually means a required input token is blank for the affected rows and should be debugged before rerun.

## HTTP API Columns

Clay HTTP API columns use:

```json
{
  "type": "action",
  "typeSettings": {
    "actionKey": "http-api-v2",
    "actionVersion": 1,
    "actionPackageId": "4299091f-3cd3-4d68-b198-0143575f471d"
  }
}
```

Use `inputsBinding[].formulaMap` for headers. Keep placeholder values when the correct key is unavailable, and verify `cellCount = 0` before handoff so placeholders cannot run accidentally.

## Use AI And Claygent

`use-ai` fields require extra care:

- inspect `typeSettings.useCase`, model, prompt, extracted-field wiring, and run guards
- observed action key: `use-ai`
- observed package: `67ba01e9-1898-4e7d-afe7-7ebe24819a57`
- important inputs include `useCase`, `prompt`, `temperature`, `reasoningLevel`, `model`, `maxCostInCents`, `jsonMode`, and `systemPrompt`
- distinguish standard OpenAI prompt columns from Claygent-style generated variants
- fetch a fresh table readback after every write
- expect Clay to normalize some hidden settings or input field order on save

Benchmark blocker: new/copy-created `use-ai` fields normalized hidden `customRateLimitRules` from a legacy source value of `200/1000ms` to the current workspace default of `5/1000ms`, even after direct field patches returned HTTP 200. The Edit Column drawer exposed model, account, prompt, output, run condition, and delay controls, but no editable custom-rate control.

Treat the readback value as authoritative.
