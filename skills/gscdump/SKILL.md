---
name: gscdump
description: >-
  Runs multi-step Google Search Console workflows through gscdump MCP tools.
  Use when triaging traffic drops, finding striking-distance queries and CTR
  gaps, checking Indexing Evidence and Sitemap search visibility, comparing a
  Fleet of Sites, or opening the Search Console Explorer.
---

# gscdump MCP Workflows

Use gscdump MCP tools only. Never tell the user to run local CLI commands or terminal steps.

## 1. Start and resolve the Site

1. Call `list-sites` first. Read `siteId` (`s_...`), `siteUrl`, `syncStatus`, `oldestDateSynced`, and `newestDateSynced`.
2. If two Sites share a `siteUrl` (`sameSiteUrlAs` is non-empty), pass `siteId` as `siteUrl` in later calls.
3. Count a shared `siteUrl` once in cross-Site totals. Read it from the Site whose synced dates cover the question.
4. If `holdReason` and `holdMessage` appear, report the hold and skip sync-dependent assumptions for that Site.
5. If `storageStatus` is `provisioning` or `failed`, tell the user storage is not ready.
6. If `list-sites` returns no Sites, call `list-properties` to show each GSC Property, its `hostScope`, and its `Estimate`.
7. Ask the user which GSC Property and exact host to connect. Then call `connect-site`.
8. Keep `Site` (one exact registered host in gscdump) distinct from `GSC Property` (the Google Search Console property it syncs from).

## 2. Triage a traffic drop or anomaly

1. If sync coverage looks partial, call `get-analytics-coverage` with the current and comparison dates before explaining a drop.
2. Call `get-analytics` with `startDate`, `endDate`, `comparison: "prev-period"` (or `"yoy"`), and `timeSeries: true`.
3. Add `trajectory: true` to `get-analytics` when checking multi-month decline against peak traffic.
4. Call `list-annotations` with the same `startDate` and `endDate` to check Google Search ranking updates and custom deploy markers.
5. Call `get-pages` and `get-keywords` with the same `startDate`, `endDate`, and `comparison` to isolate top losing pages and queries.
6. When `page.hasMore` is true and `page.nextOffset` is non-null, page with the same dates, filters, and `searchType`.
7. Call `get-position-distribution` to see whether queries fell out of positions 1 to 3 or impressions shrank at steady ranks.
8. Call `get-device-gap` (Desktop vs Mobile) and `get-countries` (same dates and comparison) to check for device or region shifts.
9. Call `get-page-details` (`url`) or `get-keyword-details` (`keyword`) on top movers for daily trends and query-to-page pairs.
10. For a single composed sweep across analyzers, call `run-report` with `siteId` and a `reportId` (`health`, `risks`, `opportunities`, `movers`, `growth`, `triage`, `brand`, or `pre-publish`).

## 3. Find striking-distance queries and CTR wins

1. Call `get-keywords` for the target date window.
2. Use `questions: "questions"` for question queries or `longQueries: true` for queries with 8 or more words.
3. Filter returned rows to positions 4 to 20 with high impressions and low CTR.
4. Call `get-ctr-curve` for the same window to compare query CTR against the Site's position buckets and spot underperforming outliers.
5. Call `get-keyword-details` (`keyword`) to see every page appearing for that query and check for split rankings across URLs.
6. Call `get-page-details` (`url`) before proposing title, snippet, or content edits so you see every query the page serves.

## 4. Triage Indexing Evidence and sitemaps

Read saved `Indexing Evidence` first. Saved reads spend no URL Inspection allowance or Google daily quota.

1. Call `get-indexing` (`days`: 1 to 90) for the `Coverage ladder` (`unknown_to_google`, `discovered_not_indexed`, `crawled_not_indexed`, `indexed`) and `capture` freshness.
2. Call `get-indexing-urls` (`status`: `"indexed"`, `"not_indexed"`, or `"pending"`, optional `search`) to list affected URLs.
3. Call `list-watched-urls` to inspect scheduled checkpoint history for Watched URLs.
4. Call `get-url-inspection` (`url`) to read the saved Google verdict and `lastCheckedAt` for one exact URL on the Site host.
5. Call `inspect-url` only when no recent saved verdict exists.
6. Saved verdicts under 7 days answer without calling Google. Live checks spend the Site's URL Inspection allowance and shared Google daily quota.
7. Audit sitemaps and canonicals separately:
   - `get-sitemap-visibility`: reads `Sitemap search visibility` (sitemap URLs with search impressions), invisible sitemap URLs, and orphan ranking URLs. Keep this distinct from Google indexing.
   - `list-sitemaps`: lists sitemap URL counts, errors, warnings, and `lastDownloaded`.
   - `get-canonical-mismatches`: lists inspected URLs where Google chose a different canonical than the declared canonical.
   - `get-sitemap-submission`: checks which sitemap gscdump would submit and what blocks submission.
   - `inspect-sitemap` (`searchEngine: "google"` or `"bing"`, `url`): reads exact sitemap evidence without submitting a sitemap.
8. For IndexNow or Google Indexing API delivery checks, call `get-submissions` (`view: "summary"`, `"receipts"`, or `"outcomes"`).
9. A `Submission Receipt` proves delivery only. A `Submission outcome` shows time order only, never proof of crawling or indexing.

## 5. Compare a Fleet and open visual views

1. For multi-Site questions, call `get-fleet-analytics` instead of calling `get-analytics` per Site.
2. Read `deduplicatedTotals` so shared `siteUrl` entries count once, and report any held or unsynced Sites returned.
3. To rank biggest weekly click drops, pass `period: "last-7d"`, `comparison: "prev-period"`, `sortBy: "clicksDelta"`, and `sortDir: "asc"`.
4. Set `includeIndexing: true` or `includeTrajectory: true` when health context helps.
5. Call `open-explorer` when the user wants an interactive dashboard view.
6. Omit `siteUrl` to open the Fleet overview, or pass `siteUrl`, `tab` (`"overview"`, `"pages"`, `"queries"`, `"indexing"`), `period`, and `comparison`.

## 6. Guardrails and evidence honesty

- **Search Console lag:** State Google Search Console's 2 to 3 day delay whenever recent dates affect the answer.
- **Sync boundaries:** Check `syncStatus`, `oldestDateSynced`, and `newestDateSynced` in metadata.
- **No extrapolation:** Never estimate past synced dates or treat unsynced days as zero traffic.
- **Evidence vs opinion:** Label every figure with its Site and exact date range. Present gscdump evidence first, then label your recommendations separately.
- **Confirm before writes:** Ask the user before calling any write tool:
  - `connect-site` (`sites:connect`): connects one exact host in gscdump and starts Backfill. Never writes to Google.
  - `manage-annotations` (`annotations:write`): creates, updates, or deletes custom chart annotations. Use `list-annotations` for reads.
  - `submit-indexnow` (`indexnow:write`): sends up to 100 changed URLs on the Site host to IndexNow (Bing and Yandex; never Google). Call only when the user asks, with a unique `idempotencyKey`.
- **No silent retries:** Never retry an uncertain `submit-indexnow` call or an `inspect-url` unknown outcome without asking the user.
