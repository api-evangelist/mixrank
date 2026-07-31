---
name: Match and enrich a company
description: Resolve a company by name/URL/LinkedIn, then pull its firmographic profile, headcount metrics, and growth trends from MixRank.
api: openapi/mixrank-openapi.yml
operations: [matchCompany, getCompany, getCompanyEmployeeMetrics, getCompanyTimeseries]
---

# Match and enrich a company

Use MixRank to turn a raw company identifier into an enriched profile.

## Auth
Every request carries your API key as a path segment: `https://api.mixrank.com/v2/json/{api_key}/...`. Validate the key first with `echo` and check remaining quota with `getUsage`.

## Steps
1. **Resolve the company.** Call `matchCompany` (`GET /companies/match`) with the name, URL, or LinkedIn to get a `company_id`. For discovery, `listCompanies` supports offset pagination (`offset`, `page_size`, `sort_field`, `sort_order`).
2. **Fetch the overview.** Call `getCompany` (`GET /companies/{company_id}`) for firmographic details.
3. **Pull headcount.** Call `getCompanyEmployeeMetrics` (`GET /companies/{company_id}/employee-metrics`) for headcount by employee type; use `listJobTags` to interpret `job_tag_id` values.
4. **Trend it.** Call `getCompanyTimeseries` (`GET /companies/{company_id}/timeseries`) for employee and social trends over time.

## Conventions & errors
- Responses are JSON; dates are ISO 8601; empty fields are `null`.
- On failure, read the `errors` array with the HTTP status (401 = bad key, 404 = unknown id, 429 = quota exceeded). See `errors/mixrank-problem-types.yml`.
