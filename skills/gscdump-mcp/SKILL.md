---
name: gscdump-mcp
description: >-
  Runs multi-step Google Search Console workflows through gscdump MCP tools.
  Use when triaging traffic drops, finding striking-distance queries and CTR
  gaps, checking Indexing Evidence and Sitemap search visibility, comparing a
  Fleet of Sites, or opening the Search Console Explorer.
---

# gscdump MCP Workflows

Use gscdump MCP tools only. Never tell the user to run local CLI commands or terminal steps.

## 1. Start and resolve the Site

1. Call `list-sites` first; read `siteId` (`s_...`), `siteUrl`, `syncStatus`, `oldestDateSynced`, `newestDateSynced`.
2. If two Sites share `siteUrl` (`sameSiteUrlAs` non-empty), pass `siteId` as `siteUrl` in later calls, and count that `siteUrl` once in cross-Site totals from the Site whose synced dates cover the question.
3. If `holdReason` and `holdMessage` appear, report the hold and skip sync-dependent assumptions for that Site.
4. If `storageStatus` is `provisioning` or `failed`, tell the user storage is not ready.
5. If `list-sites` returns no Sites, call `list-properties` (shows each GSC Property, `hostScope`, `Estimate`), ask the user which GSC Property and exact host to connect, then call `connect-site`.
6. Keep `Site` (one exact registered host in gscdump) distinct from `GSC Property` (the Search Console property it syncs from).

## 2. Triage a traffic drop or anomaly

1. If sync coverage looks partial, call `get-analytics-coverage` (current and comparison dates) before explaining a drop.
2. Call `get-analytics` with `startDate`, `endDate`, `comparison: "prev-period"` (or `"yoy"`), and `timeSeries: true`; add `trajectory: true` when checking multi-month decline against peak traffic.
3. With the same `startDate` and `endDate`, call `list-annotations` (Google Search ranking updates and custom deploy markers), then `get-pages` and `get-keywords` (same `comparison`) for top losing pages and queries.
4. If `page.hasMore` is true and `page.nextOffset` is non-null, page with the same dates, filters, and `searchType`.
5. Call `get-position-distribution` to separate drops out of positions 1 to 3 from shrinking impressions at steady ranks.
6. Call `get-device-gap` (Desktop vs Mobile) and `get-countries` (same dates and comparison) for device or region shifts.
7. Call `get-page-details` (`url`) or `get-keyword-details` (`keyword`) on top movers for daily trends and query-to-page pairs.
8. For one composed sweep across analyzers, call `run-report` (`siteId`, `reportId`: `health`, `risks`, `opportunities`, `movers`, `growth`, `triage`, `brand`, or `pre-publish`).

## 3. Find striking-distance queries and CTR wins

1. Call `get-keywords` for the target window; use `questions: "questions"` for question queries or `longQueries: true` for queries with 8 or more words.
2. Filter rows to positions 4 to 20 with high impressions and low CTR.
3. Call `get-ctr-curve` (same window) to compare query CTR against the Site's position buckets and spot underperforming outliers.
4. Call `get-keyword-details` (`keyword`) to list every page ranking for that query and check split rankings across URLs.
5. Before proposing title, snippet, or content edits, call `get-page-details` (`url`) to see every query the page serves.

## 4. Triage Indexing Evidence and sitemaps

Read saved `Indexing Evidence` first; saved reads spend no URL Inspection allowance or Google daily quota.

1. `get-indexing` (`days`: 1 to 90): `Coverage ladder` (`unknown_to_google`, `discovered_not_indexed`, `crawled_not_indexed`, `indexed`) and `capture` freshness.
2. `get-indexing-urls` (`status`: `"indexed"`, `"not_indexed"`, or `"pending"`; optional `search`): affected URLs.
3. `list-watched-urls`: scheduled checkpoint history for Watched URLs.
4. `get-url-inspection` (`url`): saved Google verdict and `lastCheckedAt` for one exact URL on the Site host.
5. Call `inspect-url` only when no recent saved verdict exists. Saved verdicts under 7 days answer without calling Google; live checks spend the Site's URL Inspection allowance and shared Google daily quota.
6. Audit sitemaps and canonicals separately:
   - `get-sitemap-visibility`: `Sitemap search visibility` (sitemap URLs with search impressions), invisible sitemap URLs, orphan ranking URLs; keep distinct from Google indexing.
   - `list-sitemaps`: sitemap URL counts, errors, warnings, `lastDownloaded`.
   - `get-canonical-mismatches`: inspected URLs where Google chose a canonical different from declared.
   - `get-sitemap-submission`: candidate sitemap gscdump would submit and what blocks submission.
   - `inspect-sitemap` (`searchEngine: "google"` or `"bing"`, `url`): exact sitemap evidence without submitting a sitemap.
7. Delivery checks (IndexNow or Google Indexing API): `get-submissions` (`view: "summary"`, `"receipts"`, or `"outcomes"`). A `Submission Receipt` proves delivery only; a `Submission outcome` shows time order only, never proof of crawling or indexing.

## 5. Compare a Fleet and open visual views

1. For multi-Site questions, use `get-fleet-analytics` instead of per-Site `get-analytics`. Read `deduplicatedTotals` so shared `siteUrl` entries count once; report any held or unsynced Sites returned.
2. Biggest weekly click drops: pass `period: "last-7d"`, `comparison: "prev-period"`, `sortBy: "clicksDelta"`, `sortDir: "asc"`. Add `includeIndexing: true` or `includeTrajectory: true` when health context helps.
3. Interactive dashboard: call `open-explorer`. Omit `siteUrl` for Fleet overview, or pass `siteUrl`, `tab` (`"overview"`, `"pages"`, `"queries"`, `"indexing"`), `period`, and `comparison`.

## 6. Guardrails and evidence honesty

- **Search Console lag:** State Google Search Console's 2 to 3 day delay whenever recent dates affect the answer.
- **Sync boundaries:** Check `syncStatus`, `oldestDateSynced`, and `newestDateSynced` in metadata. Never estimate past synced dates or treat unsynced days as zero traffic.
- **Evidence vs opinion:** Label every figure with its Site and exact date range. Present gscdump evidence first, then label recommendations separately.
- **Confirm before writes:** Ask the user before calling any write tool:
  - `connect-site` (`sites:connect`): connects one exact host in gscdump and starts Backfill; never writes to Google.
  - `manage-annotations` (`annotations:write`): creates, updates, or deletes custom chart annotations; use `list-annotations` for reads.
  - `submit-indexnow` (`indexnow:write`): sends up to 100 changed URLs on the Site host to IndexNow (Bing and Yandex; never Google). Call only when the user asks, with a unique `idempotencyKey`.
- **No silent retries:** Never retry an uncertain `submit-indexnow` call or an `inspect-url` unknown outcome without asking the user.
