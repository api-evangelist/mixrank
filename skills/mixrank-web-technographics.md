---
name: Look up a website's technology stack
description: Retrieve the technologies detected on a website, and reverse the query to find every site using a given technology tag.
api: openapi/mixrank-openapi.yml
operations: [getWebSite, getWebSiteTags, getWebTag, getWebTagSites]
---

# Look up a website's technology stack

Use MixRank's web technographics to see what a site runs and who else runs a given technology.

## Auth
API key as a path segment: `https://api.mixrank.com/v2/json/{api_key}/...`.

## Steps
1. **Load the site.** Call `getWebSite` (`GET /web/sites/{domain}`) for the overview. Use `listWebSites` to search the directory with offset pagination.
2. **Get its technologies.** Call `getWebSiteTags` (`GET /web/sites/{domain}/tags`) for the detected technology tags.
3. **Inspect a technology.** Call `getWebTag` (`GET /web/tags/{slug}`) for tag details; `listWebTags` browses the full tag directory.
4. **Reverse the lens.** Call `getWebTagSites` (`GET /web/tags/{slug}/sites`) to list every website detected using that technology — a technographic prospecting list.

## Notes
- Responses are JSON; empty fields are `null`.
- Errors return in the `errors` array with an HTTP status. See `errors/mixrank-problem-types.yml`.
