# Prompt To Table Architecture

Updated: 2026-05-06

This guide is the translation layer between a user's plain-English GTM goal and a fast Clay build plan.

## Default Intake

When a user gives a goal, infer as much as possible and only ask questions that change the architecture.

Useful inputs:

- ICP: industry, geography, size, revenue, tech, trigger, disqualifiers.
- Row type: companies, people, jobs, locations, signals, accounts plus contacts.
- Volume: sample, hundreds, thousands, ongoing monitor.
- Output: review table, CRM update, sequencer export, Slack alert, Google Sheet, webhook.
- Credit posture: cheap sample, balanced, highest coverage.
- Freshness: one-time build, scheduled source, signal monitor.
- Compliance posture: email only, phone allowed, no personal email, region restrictions.

If these are missing, use safe defaults: scratch table, auto-run off, 10-row sample, account qualification before contact enrichment, proof fields, and export blocked until verified.

## Universal Column Groups

Most production-ready tables should have these groups.

Source:

- `source_name`
- `source_url`
- `source_type`
- `source_record_id`
- `source_date`

Normalization:

- `company_name_clean`
- `company_domain`
- `person_name_clean`
- `linkedin_url`
- `location_clean`

Qualification:

- `icp_match`
- `qualification_score`
- `qualification_reason`
- `disqualification_reason`
- `needs_review`

Evidence:

- `evidence_summary`
- `evidence_url_1`
- `evidence_url_2`
- `evidence_source`
- `confidence`

Enrichment:

- provider-specific raw columns
- canonical output fields
- `successful_provider`
- `enrichment_status`
- `enrichment_error`

Action:

- `recommended_action`
- `message_angle`
- `owner_or_route`
- `export_ready`
- `export_blocker`

Audit:

- `last_run_at`
- `last_export_status`
- `external_record_id`
- `synced_at`

## Standard Views

Create these unless the table has a more specific workflow.

- `All rows`: everything.
- `Ready to enrich`: qualified rows with missing enrichment.
- `Needs review`: thin evidence, conflicting values, blank critical fields.
- `Errored rows`: action failures, blank tokens, provider errors.
- `Fully enriched`: required fields complete.
- `Export ready`: safe rows for CRM, sequencer, sheet, or webhook.
- `High priority`: top-score rows for human inspection.

## Table Archetypes

### TAM Finder

Use when the goal is to build a target account universe.

Architecture:

1. Source companies from Find Companies, CSV, CRM, warehouse, Google Maps, Store Leads, or web search.
2. Normalize domain and company name.
3. Enrich firmographics and category.
4. Add signal sources if the prompt mentions timing.
5. Score ICP fit.
6. Create views for high fit, needs review, and disqualified.

Common integrations:

- Find Companies, People Data Labs, HG Insights, SimilarWeb, Semrush, Store Leads, CB Insights, PitchBook, Dealroom.co, Owler, PredictLeads, Google News, Claygent.

Outputs:

- company list, fit score, evidence, next action.

### Buyer Finder

Use when the goal is people at target accounts.

Architecture:

1. Start from qualified company rows.
2. Find people by title, department, seniority, location.
3. Score persona fit.
4. Run work email waterfall on qualified contacts only.
5. Verify contact data.
6. Export to sequencer or CRM.

Common integrations:

- Find People, Mixrank, ZoomInfo, People Data Labs, Nimbler, The Org, LeadIQ, Surfe, Work Email waterfall, Prospeo, Leadmagic, Hunter, DropContact, Wiza, Neverbounce, Zerobounce.

Outputs:

- buyer rows with persona match, email status, provider, export readiness.

### SaaS Founder Outreach

Use when the goal is a compact outbound table such as "find SaaS companies, find the founder or CEO, verify emails, and draft a personalized email."

Architecture:

1. Source companies with Find Companies using a specific segment, geography, size, funding, tech, or category description.
2. Keep the company source small until the run path is proven, for example 25 rows.
3. Normalize company name, domain, LinkedIn URL, and basic firmographics.
4. Find one founder, co-founder, or CEO per qualified account.
5. Extract canonical person fields from the people finder result.
6. Run work email providers behind company-domain and person-name guards.
7. Canonicalize to `work_email`, `email_status`, and `email_provider`.
8. Draft outreach with Use AI or Claygent from a product brief, account evidence, persona title, and verified email status.
9. Put unverified or no-contact rows into review, not export.

Common integrations:

- Find Companies, Find People, Mixrank, Prospeo, Leadmagic, Work Email waterfall, Use AI, Claygent.

Outputs:

- company, founder or CEO, verified email when available, provider status, message angle, email draft, export readiness.

### Signal Monitor

Use when timing matters.

Architecture:

1. Choose native signal, RSS, webhook, jobs, news, social, or product data source.
2. Match signal to account.
3. Classify signal type and strength.
4. Score urgency and buyer relevance.
5. Route to owner, Slack/Teams, CRM task, or sequencer list.

Common integrations:

- Web intent, promotions, new hire, news and fundraising, job changes, Custom Signals, PredictLeads, TheirStack, TrustRadius, Trigify, RB2B, Google News, Reddit, Hacker News, webhooks.

Outputs:

- signal rows, account match, trigger evidence, recommended action, expiry date.

### Account Research Brief

Use when the goal is a high-quality account dossier.

Architecture:

1. Normalize company and domain.
2. Enrich core account data.
3. Scrape or research website, jobs, news, product pages, reviews, and tech stack.
4. Use AI/Claygent to synthesize the brief.
5. Store evidence URLs and confidence.
6. Score account and propose message angle.

Common integrations:

- Use AI, Claygent, Google News, Perplexity, ScrapeMagic, Zenrows, Apify, SimilarWeb, Semrush, HG Insights, TrustRadius, Reddit, Hacker News.

Outputs:

- summary, pain signals, evidence, confidence, message angle.

### CRM Hygiene And Enrichment

Use when the goal is to clean or augment existing CRM data.

Architecture:

1. Import or read CRM records.
2. Dedupe and normalize key fields.
3. Lookup existing Clay/table data before calling providers.
4. Fill only missing or stale fields.
5. Validate emails/domains.
6. Write back only high-confidence fields.

Common integrations:

- Salesforce, HubSpot, Attio, Pipedrive, Microsoft Dynamics 365, Close, BigQuery, Snowflake, Postgres, Lookup Rows, Write to Other Table, HTTP API.

Outputs:

- changed fields, confidence, writeback status, manual review queue.

### Local Market Prospecting

Use for geographic or location-heavy lists.

Architecture:

1. Source locations from Google Maps, Yelp, Mapbox, Openmart, CSV territories, or web search.
2. Normalize address, domain, and business category.
3. Enrich owner/company data.
4. Find decision maker or public contact channel.
5. Score based on category, reviews, location, website maturity, and trigger.

Common integrations:

- Google Maps, Yelp, Mapbox, Openmart, Enigma, RB2B, Store Leads, Claygent, Work Email waterfall.

Outputs:

- business/location rows, owner/contact route, evidence, priority.

### Ecommerce Or App Prospecting

Use for Shopify, commerce, marketplace, or app targets.

Architecture:

1. Source stores/apps from Store Leads, Shopify, Dappradar, Google Shopping, or category searches.
2. Enrich domain, traffic, category, tech, size, and reviews.
3. Detect trigger or fit.
4. Find founder, marketing, ecommerce, or partnerships persona.
5. Waterfall contact data.

Common integrations:

- Store Leads, Shopify, Dappradar, Google Shopping, SimilarWeb, Semrush, People Data Labs, Find People, Work Email waterfall.

Outputs:

- store/app rows, fit score, trigger, contact, export readiness.

### Competitive Intent Miner

Use when the prompt mentions competitors, replacement, review sites, or complaints.

Architecture:

1. Source intent from TrustRadius, Reddit, Hacker News, X.com, Google News, reviews, social, or web search.
2. Classify whether the mention indicates active buying, dissatisfaction, migration, or research.
3. Match to account and persona.
4. Score urgency and fit.
5. Find buyer/contact only for strong signals.

Common integrations:

- TrustRadius, Reddit, Hacker News, X.com, Google News, Trigify, RB2B, Claygent, Use AI.

Outputs:

- trigger evidence, account match, urgency, suggested outreach angle.

## Build Plan Template

Use this internal shape before constructing a table.

```text
Goal:
Row type:
Source strategy:
Qualification gates:
Enrichment chain:
Waterfalls:
AI or Claygent tasks:
Output destination:
Views:
Sample run scope:
Scale criteria:
Risks:
```

## Default Run Strategy

- Create scratch table.
- Turn auto-run off.
- Create source and normalization columns.
- Add cheap qualification.
- Add expensive enrichments behind run conditions.
- Add waterfalls after qualification.
- Add scoring and evidence.
- Add export views.
- Run 10 rows.
- Inspect runstatus, blanks, and proof fields.
- Patch logic.
- Scale selected view or selected rows.

## Quality Bar

A table is production-ready when:

- every high-priority row explains why it is high priority
- every export-ready row has proof and required contact/status fields
- every disqualified row explains why it was filtered out
- failures are isolated in an error view
- expensive enrichments are guarded by conditions
- final outputs are canonical and do not require the user to inspect raw provider columns
- evidence URLs or source summaries exist for AI claims

## Sources

- https://university.clay.com/docs-topics/find
- https://university.clay.com/docs-topics/enrich
- https://university.clay.com/docs-topics/transform
- https://university.clay.com/docs-topics/web-scraping
- https://university.clay.com/docs-topics/signals
- https://university.clay.com/docs-topics/export
- https://university.clay.com/docs-topics/gen-ai
- https://university.clay.com/docs/sculptor
