# Clay Integration Selector

Updated: 2026-05-06

This guide turns Clay's large integration surface into fast build decisions. It is intentionally public-safe: names and categories only, no live credentials, raw manifests, table IDs, workspace IDs, or customer data.

Treat the provider inventory as a current working index from Clay University topic pages, not a guarantee of every action each integration exposes. When exact inputs, account requirements, or action names matter, open the specific Clay doc or inspect the enrichment drawer in Clay.

## Fast Routing

Use this first when a user gives a broad goal.

| Goal in the prompt | Start with | Then add | Output shape |
| --- | --- | --- | --- |
| Build a TAM or account list | Find Companies, CSV, CRM, warehouse, web search | firmographics, traffic, tech, hiring, news, ICP scoring | company rows with proof and priority |
| Find buyers at accounts | company rows, Find People, role/seniority filters | person enrich, work email waterfall, phone only if qualified | person rows with contactability and reason |
| Catch active buying signals | Signals, jobs, news, web intent, TrustRadius, social/web sources | AI classification, score, routing | signal rows or account-stage updates |
| Enrich CRM records | Salesforce, HubSpot, Attio, Pipedrive, warehouse | lookups first, then missing-field waterfalls | clean records and writeback status |
| Build outbound from scratch | companies -> people -> email waterfall | message drafting, sequencer export | export-ready contacts and copy |
| Research accounts deeply | domain/name source | Claygent, Use AI, web scraping, news, filings, jobs | account intelligence brief |
| Local or geo prospecting | Google Maps, Yelp, Mapbox, CSV territories | owner/contact finding, reviews, website scrape | location rows and contacts |
| Ecommerce or app prospecting | Store Leads, Shopify, Dappradar, Google Shopping | tech/traffic/enrichment, owner/person find | storefront/app rows |
| Competitor or category intent | TrustRadius, Reddit, Hacker News, X.com, Google News, web search | AI classifier, persona match, contact find | accounts with trigger evidence |
| Data push or activation | CRM, sequencer, Slack/Teams, Sheets, webhook, HTTP API | conditional runs, export-ready view | synced records and audit columns |

## Source And Intake Families

Use sources to decide the row universe before spending enrichment credits.

- Native list sourcing: Find Companies, Find People, Find Jobs, Find AI, saved searches.
- File and manual intake: CSV import, table rows, Clay Chrome extension, Clip to Clay.
- Realtime intake: webhooks, Typeform, RSS feed, web intent.
- CRM intake: Salesforce, Salesforce SOQL, HubSpot, Attio, Microsoft Dynamics 365, Pipedrive, Close.
- Warehouse/database intake: BigQuery, Snowflake, Databricks, Postgres, Fivetran.
- App/workflow intake: Google Sheets, Airtable, Coda, Notion, Linear, Mixpanel, Stripe, Marketo.
- Web source intake: web search, Google Maps, Yelp, Apify, Phantombuster, Zenrows, ScrapeMagic.

Default rule: if the source is broad, add cheap qualification and proof columns before person finding or contact waterfalls.

## Company And Account Intelligence

Use these when the row is an account, business, location, or domain.

Core company data and firmographics:

- Find Companies, People Data Labs, HG Insights, ZoomInfo, CB Insights, PitchBook, Dealroom.co, Owler, Harmonic.ai, HitHorizons, Firmable, Explorium, Pubrio, SMARTe, Versium, Ocean.io, Enigma.

Revenue, size, traffic, market, and competitive context:

- HG Insights, People Data Labs, Pubrio, RocketReach, SimilarWeb, Semrush, SerpStat, Store Leads, Datagma, CB Insights, PitchBook, Owler, Google PageSpeed, Google Shopping, Glassdoor, Google News, PredictLeads.

Tech stack, hiring, and operational signals:

- HG Insights, BuyerCaddy, Sumble, TheirStack, PredictLeads, Google Jobs/Find Jobs, TrustRadius, Intellizence, Google News.

Corporate hierarchy and ownership:

- HG Insights, HitHorizons, The Org, Claygent or Use AI web research, filings/news sources, company websites.

Local and physical footprint:

- Google Maps, Yelp, Mapbox, Enigma, Openmart, RB2B, Store Leads, Claygent/website scraping.

Build bias: accounts should end with `account_score`, `score_reason`, `evidence_urls`, `disqualification_reason`, `best_next_action`, and `owner_or_route`.

## Person, Identity, And Contact Data

Use these after account qualification unless the task is explicitly people-first.

People discovery:

- Find People, Mixrank, Nimbler, Pubrio, ZoomInfo, People Data Labs, The Org, The Swarm, Modash, Upfluence, Influencer Club, Weekday, Loxo.

Profile and identity resolution:

- People Data Labs, LiveData, Reverse Contact, Minerva, Clay Enrichments, ContactLevel, Vector, Identity Matrix, LeadIQ, Surfe, Nimbler.

Work email finding and validation:

- Work Email waterfall, Prospeo, Leadmagic, Findymail, Hunter, DropContact, Datagma, Nimbler, Wiza, People Data Labs, Enrow, Fullenrich, BetterContact, Snov.io, Lusha, LeadIQ.

Email verification and cleaning:

- Listmint, Zerobounce, Neverbounce, Debounce, Trestle, ClearoutPhone where relevant, email verification statuses.

Phone/mobile coverage:

- Lusha, Swordfish, Upcell, Signalhire, SMARTe, People Data Labs, Nimbler, Trestle, SureConnect, Zeliq, ClearoutPhone.

Contactability and compliance:

- Trestle, SureConnect, Meer, email verification statuses, connection access settings, conditional runs.

Build bias: contact rows should end with `persona_fit`, `seniority_match`, `work_email`, `email_status`, `phone_if_allowed`, `successful_provider`, `contact_priority`, and `do_not_contact_reason`.

## AI, Research, And Web Scraping

Use these when structured providers are not enough or when the edge comes from synthesis.

AI-native Clay surfaces:

- Use AI, Claygent builder, Sculptor, formula generator, API generator, Debug with AI, Find AI, AI tokens, sandbox mode.

Web research and extraction:

- Claygent, Use AI web research, ScrapeMagic, Zenrows, Apify, Phantombuster, Perplexity, Twain, Google News, Google PageSpeed, website scraping patterns.

Social and community sources:

- Reddit, Hacker News, X.com, Instagram, YouTube, TrustRadius, Trigify, Modash, Upfluence, Influencer Club.

Content and language actions:

- Google Translate, ElevenLabs, Lavender, Autobound, Sendspark, HeyGen, Tavus, message drafting, email sequencer.

Build bias: AI outputs should be structured, cite or store evidence URLs, include confidence, and produce `needs_human_review` when evidence is thin.

## Signals And Triggers

Use signals when the workflow needs timing, not just enrichment.

Native signal docs currently include:

- Web intent.
- Monitor for promotions.
- Monitor for new hire.
- Monitor for news and fundraising.
- Monitor for job changes.
- Custom Signals.

Signal-like integrations and sources:

- PredictLeads, TheirStack, TrustRadius, Trigify, RB2B, Reddit, Hacker News, Google News, RSS feed, Mixpanel, webhooks, product/CRM data.

Build bias: signal tables need `signal_type`, `signal_date`, `signal_strength`, `source_url`, `account_match`, `person_match`, `recommended_action`, and `expires_at`.

## Export, Activation, And System Sync

Use these only after a row has enough evidence to act.

CRM and customer systems:

- Salesforce, Salesforce SOQL, HubSpot, Attio, Pipedrive, Close, Microsoft Dynamics 365, Marketo, Outreach, Salesloft, Gong, Rox.

Sequencing and outbound activation:

- Email sequencer, Smartlead, Instantly, Lemlist, La Growth Machine, HeyReach, Reply.io, Woodpecker, Salesforge, EmailBison, ActiveCampaign, Customer.io, Groove, Za-zu.

Data warehouses and docs:

- BigQuery, Snowflake, Databricks, Postgres, Fivetran, Google Sheets, Google Docs, Google Slides, Airtable, Coda, Notion.

Notifications and workflow:

- Slack, Microsoft Teams, Linear, Zapier, webhooks, HTTP API, HTTP API with JWT authentication.

Other activation:

- Clay Ads, Sendoso, Lob, Webflow, Stripe, Ironclad, Bitly, Octave.

Build bias: export views should include `export_ready`, `export_blocker`, `last_export_status`, `external_record_id`, and `synced_at`.

## Integration Inventory From Official Topic Pages

This is a compact current index scraped from Clay University topic pages on 2026-05-06. Keep it refreshed as Clay adds or renames docs.

Find topic:

- Guide: Finding companies and people in Clay
- ContactLevel integration
- Enigma integration
- Vector integration
- Work Email waterfall
- Webhooks in Clay
- Typeform integration
- TrustRadius integration
- Trigify integration
- Table columns
- RSS feed integration
- Pubrio integration
- RB2B integration
- Openmart integration
- Modash integration
- Mixpanel integration overview
- Manage cell data
- Infinite loops
- GitHub integration
- Find People in Clay
- Find Companies in Clay
- CSV import
- Crossbeam integration
- Clip to Clay extension
- Clay Chrome extensions
- Apify integration

Enrich topic:

- Google BigQuery integration
- CB Insights
- Attio integration
- Lusha integration
- Explorium integration
- Sendoso integration
- Using Clay in Claude
- Using Clay in ChatGPT
- Listmint integration
- EmailBison integration
- Zerobounce integration
- ZoomInfo integration
- YouTube integration
- X.com integration
- Yelp integration
- Wiza integration
- Versium integration
- Weekday integration
- Webflow integration
- Upfluence integration
- Upcell integration
- Twain integration
- Tavus integration
- Trestle integration
- The Org integration
- TheirStack integration
- The Swarm integration
- Stripe integration
- Swordfish integration
- SureConnect integration
- Surfe integration
- Summit integration
- Store Leads integration
- Snov.io integration
- SMARTe integration
- SimilarWeb integration
- Signalhire integration
- Shopify integration
- SerpStat integration
- Sendspark integration
- Semrush integration
- Rox integration
- Reverse Contact integration
- Reddit integration
- Prospeo integration overview
- Perplexity integration overview
- PredictLeads integration
- Owler integration overview
- PitchBook integration
- Phantombuster integration
- Neverbounce integration
- Mixrank integration overview
- Ocean.io integration
- Oakie.ai integration
- Nimbler integration overview
- Madkudu integration overview
- Managing integration accounts
- Minerva integration
- Marketo webhook connection
- Marketo integration
- Mapbox integration
- Loxo integration
- Loom integration
- Lob integration overview
- LiveData integration
- Lavender integration overview
- Limadata integration
- Leadmagic integration overview
- LeadIQ integration
- HitHorizons integration overview
- Ironclad integration
- Icypeas integration overview
- Intellizence integration
- Instagram integration
- Influencer Club integration overview
- Identity Matrix integration
- Hunter
- HG Insights integration
- Harmonic.ai integration overview
- Hacker News integration overview
- Google Translate integration
- Glassdoor integration overview
- Google Shopping integration overview
- Google PageSpeed integration overview
- Google News integration
- Fullenrich integration
- Firmable integration
- Forager integration
- Findymail integration
- Enrichley integration overview
- DropContact integration overview
- Exellius integration
- Debounce integration overview
- Enrow integration
- Enrichments
- ElevenLabs integration
- Dealroom.co integration
- Datagma integration
- Dappradar integration overview
- ClearoutPhone integration overview

Transform topic:

- Share workbooks and tables as templates
- URL Validator integration
- Run progress
- Lookup Rows
- Logical operators
- Lead scoring overview
- Find your Clay API key
- Formulas
- Find your ICP based on your domain
- Comparison operators
- Column group templates
- Clay formatters overview
- Bitly integration
- Auto-delete in tables

Web scraping topic:

- Meer integration
- BuyerCaddy integration
- Sumble integration
- Zenrows integration
- ScrapeMagic integration
- People Data Labs integration overview

Signals topic:

- Web intent
- Monitor for promotions
- Monitor for new hire
- Monitor for news and fundraising
- Monitor for job changes
- Custom Signals

Export topic:

- Salesforce integration FAQs
- Microsoft Teams integration
- Salesforce SOQL
- Za-zu integration
- Woodpecker integration
- Snowflake integration
- Smartlead integration
- Slack integration
- Send table data
- Reply.io integration
- Salesloft integration overview
- Salesforge integration
- Salesforce integration
- Postgres integration overview
- Pipedrive integration overview
- Outreach integration
- Octave integration
- Notion integration
- Microsoft Dynamics 365 CRM integration overview
- Linear integration overview
- Lemlist integration
- Instantly integration
- Lead-to-account matching for Salesforce
- La Growth Machine integration overview
- HeyReach integration
- HubSpot integration
- HeyGen integration
- Groove integration
- Google Slides integration
- Google Sheets integration
- Google Docs integration
- Gong integration
- Fivetran integration overview
- Email sequencer
- Databricks integration
- Customer.io integration
- Credit spend limits FAQ
- Coda integration overview
- Close integration overview
- Send Clay data to Zapier
- Airtable integration overview
- Autobound integration
- ActiveCampaign integration

Gen AI topic:

- Use AI
- Sculptor
- Sandbox mode
- Claygent builder
- AI Tokens
- Writing AI prompts in Clay

Getting started topic:

- Functions
- Table versions
- Table alerts
- Clay Ads
- Zeliq integration
- Write to Other Table
- Sources
- Scheduled sources
- Scheduled columns
- Saved searches
- Round-robin integration overview
- Referencing dynamic data from other columns
- Message drafting overview
- HTTP API with JWT authentication
- HTTP API
- Enrichment recipes
- Email verification statuses
- Data tests for work email, personal email, and mobile phone providers
- Actions and Data Credits
- Conditional statements
- Conditional runs
- Guide: Ways to save Clay credits

## Maintenance Notes

- Refresh this inventory from official Clay docs before a major competition build.
- Keep provider names here, not implementation secrets.
- Move exact endpoint behavior to `docs/frontend-api-patterns.md`.
- Move reusable workflow design to `docs/waterfalls-functions-playbook.md`.

## Sources

- https://university.clay.com/docs-topics/find
- https://university.clay.com/docs-topics/enrich
- https://university.clay.com/docs-topics/transform
- https://university.clay.com/docs-topics/web-scraping
- https://university.clay.com/docs-topics/signals
- https://university.clay.com/docs-topics/export
- https://university.clay.com/docs-topics/getting-started
- https://university.clay.com/docs-topics/gen-ai
