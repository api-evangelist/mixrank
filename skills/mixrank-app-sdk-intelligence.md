---
name: Analyze a mobile app's SDK stack
description: Find a mobile app in MixRank and enumerate the third-party SDKs it embeds, then reverse the lens to see every app that installs a given SDK.
api: openapi/mixrank-openapi.yml
operations: [listAppstoreApps, getAppstoreApp, getAppstoreAppSdks, getAppstoreSdkInstalls]
---

# Analyze a mobile app's SDK stack

Use MixRank's mobile intelligence to inspect an app's SDK footprint (iOS shown; the `/playstore/...` operations mirror this for Android).

## Auth
API key as a path segment: `https://api.mixrank.com/v2/json/{api_key}/...`.

## Steps
1. **Find the app.** Call `listAppstoreApps` (`GET /appstore/apps`) with search + offset pagination to get an `app_id`.
2. **Load the summary.** Call `getAppstoreApp` (`GET /appstore/apps/{app_id}`).
3. **List its SDKs.** Call `getAppstoreAppSdks` (`GET /appstore/apps/{app_id}/sdks`) to see the embedded third-party SDKs.
4. **Reverse the lens.** For any SDK of interest, call `getAppstoreSdkInstalls` (`GET /appstore/sdks/{sdk_slug}/installs`) to list apps that install it; `getAppstoreSdkInstallTrend` shows adoption over time.

## Notes
- Android equivalents: `getPlaystoreAppSdks`, `getPlaystoreSdkInstalls`.
- Pair with `getAppstoreAppDownloads` / `getAppstoreAppRevenue` for market sizing.
- Errors surface in the `errors` array with an HTTP status; watch 429 for quota. See `conventions/mixrank-conventions.yml`.
