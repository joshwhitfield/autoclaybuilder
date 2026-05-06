# Waterfalls And Functions Playbook

Updated: 2026-05-06

This playbook explains when to use waterfalls, when to promote logic into a Function, and how to build reusable enrichment systems without wasting credits.

## Mental Model

A Clay table is the visible workflow. A waterfall is a fallback chain inside that workflow. A Function is a reusable mini-workflow that can be called from many tables.

Use the smallest durable shape:

- One-off table logic: use normal columns.
- Repeated multi-column logic: use a column group template.
- Repeated enrichment sequence across tables: use a Function.
- Provider fallback for one output: use a waterfall.
- Multi-stage qualification plus enrichment plus scoring: use a table or a Function depending on reuse.

## Waterfall Basics

A waterfall tries multiple providers or methods to produce one target output, usually stopping when a good enough result is found.

Good waterfall targets:

- work email
- personal email
- mobile phone
- LinkedIn/profile URL
- company domain
- company firmographics
- contact identity resolution
- tech stack or buying signal confirmation
- validation and cleanup steps

Bad waterfall targets:

- vague account research briefs with many fields
- outputs that require judgment across many sources
- expensive provider calls before qualification
- fields where conflicting results are useful evidence rather than noise

## Waterfall Design Rules

1. Qualify before the waterfall.
2. Start with existing data and deterministic transforms.
3. Use cheaper or broader providers before expensive niche providers unless accuracy demands otherwise.
4. Stop on success when the downstream workflow needs one output.
5. Keep provider outputs available during debugging, even if the final view only shows the canonical result.
6. Store `successful_provider`, `waterfall_status`, `confidence`, and `needs_review`.
7. Add conditional runs so blank or disqualified rows do not spend credits.
8. Validate after finding when the output is contact data.
9. Put errors and blanks into a dedicated view.
10. Scale only after a sample proves hit rate and cost.

## Work Email Waterfall Pattern

Use when a qualified person row needs a deliverable business email.

Inputs:

- first name
- last name
- company domain
- company name as fallback
- LinkedIn/profile URL when available
- country or region when relevant

Suggested stages:

1. Existing CRM/email field.
2. Domain normalization and obvious corporate email pattern if already known.
3. Work email providers in the workspace's preferred order.
4. Secondary providers for missing rows.
5. Verification provider.
6. Canonical output and status fields.

Useful providers and surfaces from current docs include Work Email waterfall, Prospeo, Leadmagic, Findymail, Hunter, DropContact, Datagma, Nimbler, Wiza, People Data Labs, Enrow, Fullenrich, Snov.io, Lusha, LeadIQ, Listmint, Zerobounce, Neverbounce, Debounce, and email verification statuses.

Output fields:

- `work_email`
- `email_status`
- `email_confidence`
- `email_provider`
- `email_verification_provider`
- `email_waterfall_error`
- `safe_to_export`

Common run conditions:

- only run when `persona_fit` is true
- only run when `company_domain` is present
- only run when `work_email` is blank
- do not run on free-mail domains unless the workflow specifically needs them

## Phone Waterfall Pattern

Use only when the goal justifies the cost and compliance posture.

Inputs:

- person name
- company
- location or region
- LinkedIn/profile URL
- existing CRM phone if present

Candidate providers from current docs include Lusha, Swordfish, Upcell, Signalhire, SMARTe, People Data Labs, Nimbler, Trestle, SureConnect, Zeliq, and ClearoutPhone.

Output fields:

- `phone`
- `phone_type`
- `phone_region`
- `phone_provider`
- `phone_confidence`
- `phone_do_not_use_reason`

Default rule: do not phone-waterfall every found person. Phone waterfalls should usually sit after account score, persona fit, and email outcome.

## Company Domain Waterfall Pattern

Use when the source has names but not reliable domains.

Stages:

1. Existing CRM/domain field.
2. URL cleanup from website field.
3. Find domain from company name and location.
4. Company enrich provider.
5. Search or Claygent fallback for unresolved names.
6. URL validator.

Output fields:

- `company_domain`
- `domain_source`
- `domain_confidence`
- `domain_evidence_url`
- `domain_needs_review`

## Account Intelligence Waterfall Pattern

Use when the table needs a stable account profile before people/contact enrichment.

Stages:

1. Core company enrich.
2. Industry/category normalization.
3. Employee size and revenue estimate.
4. Tech stack or product signal.
5. Hiring/news/funding signal.
6. AI summary with evidence URLs.
7. Score and disqualifier.

Candidate sources include Find Companies, People Data Labs, HG Insights, SimilarWeb, Semrush, CB Insights, PitchBook, Dealroom.co, Owler, PredictLeads, TheirStack, Google News, and Claygent.

Output fields:

- `account_score`
- `account_score_reason`
- `top_signal`
- `top_signal_url`
- `disqualified_reason`
- `next_action`

## Functions

Functions are reusable Clay workflows that can be called from tables. Current Clay course material frames them as a way to save reusable sequences as an action, run them in the background, return clean outputs, and roll updates out to every table using that Function.

Use a Function when:

- the same enrichment sequence will appear in multiple tables
- the workflow has multiple input columns and multiple outputs
- you want one centrally maintained version of an ICP score, email waterfall, account brief, or persona matcher
- the workflow is complex enough that copy-pasting columns will create drift
- you need team-scale reuse with consistent outputs

Do not use a Function when:

- the logic is experimental and will only run once
- the workflow depends on ad hoc table-specific fields
- the output needs manual per-table tuning
- the function would hide important debugging fields too early

## Function Design Process

1. Prototype the logic in a normal scratch table.
2. Name the intended Function by output, not by provider: `Get verified work email`, `Score healthcare fintech account`, `Detect competitor switch intent`.
3. Define a minimal input contract.
4. Define a small output contract with stable field names.
5. Add debug outputs only if future tables need them.
6. Test on a small sample.
7. Save or convert the sequence into a Function.
8. Use it from a second table to prove reuse.
9. Document inputs, outputs, expected cost, and failure modes.
10. Keep old versions only when downstream tables still depend on them.

## Recommended Function Contracts

Verified work email:

- Inputs: `first_name`, `last_name`, `company_domain`, `company_name`, optional `linkedin_url`.
- Outputs: `work_email`, `email_status`, `provider`, `confidence`, `error`.

Account ICP score:

- Inputs: `company_name`, `domain`, `industry`, `employee_count`, optional `description`, `tech_stack`, `job_posts`, `news`.
- Outputs: `score`, `tier`, `fit_reason`, `disqualifier`, `recommended_action`.

Persona matcher:

- Inputs: `full_name`, `title`, `linkedin_url`, `company_name`, `target_persona`.
- Outputs: `persona_match`, `seniority`, `department`, `match_reason`, `priority`.

Account research brief:

- Inputs: `company_name`, `domain`, optional `source_urls`.
- Outputs: `summary`, `pain_signals`, `evidence_urls`, `confidence`, `needs_review`.

Buying-window score:

- Inputs: account profile, hiring/news signals, tech/intent data, persona availability.
- Outputs: `score`, `window`, `reason`, `trigger`, `suggested_message_angle`.

## Function Governance

- Put version in the description or name when changing outputs.
- Keep outputs stable; downstream tables break when output names or types drift.
- Avoid embedding customer-specific prompts unless the Function is intentionally customer-specific.
- Use run conditions outside the Function when the caller table should decide eligibility.
- Use run conditions inside the Function when eligibility is intrinsic to the Function.
- Document expected credit behavior.
- Keep a sample test table or fake fixture pattern for each important Function.

## Function Versus Template Versus Column Group

Use a column group template when the columns are reusable but each table can own its copy.

Use a table template or Claybook when the whole workflow shape should be reused: sources, views, columns, and exports.

Use a Function when the logic should be centrally updated and reused as a callable action.

Use Sculptor when you need fast suggestions, formulas, AI prompts, or debugging help before turning the pattern into a durable asset.

## Prompt-To-Waterfall Defaults

When the user asks for contacts:

1. Source accounts.
2. Score accounts.
3. Find people by persona.
4. Score persona fit.
5. Run work email waterfall only on qualified people.
6. Verify email.
7. Export only if `safe_to_export` is true.

When the user asks for account research:

1. Normalize domain.
2. Enrich company.
3. Add signal sources.
4. Use AI or Claygent for synthesis.
5. Store evidence URLs.
6. Score and explain.

When the user asks for monitoring:

1. Choose signal source.
2. Match signal to account.
3. Classify signal with AI.
4. Route or alert.
5. Write back to CRM only when confidence is high.

## Sources

- https://university.clay.com/docs/work-email-waterfall
- https://university.clay.com/docs/functions
- https://university.clay.com/lessons/introduction-to-functions
- https://university.clay.com/lessons/build-your-first-function
- https://university.clay.com/lessons/roll-functions-out-across-your-team
- https://university.clay.com/docs/conditional-runs
- https://university.clay.com/docs/guide-ways-to-save-clay-credits
- https://university.clay.com/docs-topics/enrich
