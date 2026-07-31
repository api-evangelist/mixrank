---
name: Prospect and verify an email address
description: Generate a likely email for a person at a company, verify a single address, and run bulk email validation as an asynchronous job.
api: openapi/mixrank-openapi.yml
operations: [prospectEmail, validateEmail, createEmailBulkJob, getEmailBulkJob]
---

# Prospect and verify an email address

Use MixRank's email tooling for both single-address and bulk verification.

## Auth
API key as a path segment: `https://api.mixrank.com/v2/json/{api_key}/...`.

## Single-address flow
1. **Prospect.** Call `prospectEmail` (`GET /email/prospect`) to generate a likely email for a person at a company.
2. **Verify.** Call `validateEmail` (`GET /email/validate`) to confirm the address exists.

## Bulk flow (asynchronous job)
1. **Submit.** Call `createEmailBulkJob` (`POST /email/validate/bulk-job`) with the list of addresses; you receive a `job_id`.
2. **Poll.** Call `getEmailBulkJob` (`GET /email/validate/bulk-job/{job_id}`) until the job completes. `listEmailBulkJobs` shows prior jobs.
3. **Clean up.** Call `deleteEmailBulkJob` (`DELETE /email/validate/bulk-job/{job_id}`) to delete the output when done.

## Notes
- Bulk operations are job-based, not idempotent-by-key; do not resubmit a job to retry — poll it. See `conventions/mixrank-conventions.yml`.
- Errors return in the `errors` array with an HTTP status (429 = quota exceeded; check `getUsage`).
