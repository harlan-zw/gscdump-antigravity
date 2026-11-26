# gscdump

Google Search Console data for agents

Ask Antigravity about your Google Search Console data. gscdump connects your Sites with one Google sign-in, backfills up to 16 months of available history, and keeps your record after Google deletes it.

Ask which pages lost clicks, which queries sit on page two, or whether a page is indexed. Answers come from your rows, with the dates they cover.

## Interactive UI (MCP Apps)

In the Antigravity App, gscdump renders interactive Search Console views directly beside the agent's answer:

- **Inline in chat:** daily clicks and impressions charts (`get-analytics`), sortable page and query tables (`get-pages`, `get-keywords`), URL Inspection verdict cards (`inspect-url`, `get-url-inspection`), coverage ladders (`get-indexing`), and multi-Site Fleet summaries (`get-fleet-analytics`).
- **Side panel (`Search Console`):** open from the side panel `+` menu or via `open-explorer` to switch Sites, date windows, and tabs without cluttering chat. Click any page, query, or verdict to attach its metrics to your next message.
- **Full page (`gscdump`):** open from **Plugins** in the left sidebar for a workspace-wide Fleet overview and indexing health view.

Antigravity CLI and non-UI clients use the same tools with plain-text output.

Evidence tools only read your data. inspect-url asks Google live and uses your URL Inspection allowance. connect-site adds a Site to your gscdump account and starts its backfill. submit-indexnow sends the URLs you choose to IndexNow. submit-feedback and record-agent-outcome send reports to gscdump. Nothing writes to Google.

Open source: the engine, CLI, and SDK are MIT licensed on GitHub.

Free during beta.

## Connect

1. Install the plugin. Without the Marketplace, run `agy plugin install https://github.com/harlan-zw/gscdump-antigravity`.
2. Start the sign-in. In the Antigravity CLI, open `/plugin`, expand gscdump under Installed, and choose **Sign in**.
3. Sign in to gscdump with Google, then allow access for Google Antigravity.
4. If Antigravity asks for an authorization code, paste the code from the antigravity.google page.

## Example prompts

1. Which of my Sites lost the most clicks this week?
2. Which queries sit at positions 4 to 20 with high impressions?
3. Is my home page indexed by Google? Show the verdict and the date.
4. Compare this quarter with the same quarter last year.

## Links

- Documentation: https://gscdump.com/mcp
- Support: https://gscdump.com/support
- Privacy policy: https://gscdump.com/privacy
- Terms: https://gscdump.com/terms
- Access and scopes: https://gscdump.com/trust
