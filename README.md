# Ultimate Jira Projects Dashboard

A single self‑contained HTML dashboard that gives product and project managers three
drill‑down levels of Jira health in one place:

1. **Portfolio Health** — multi‑project scorecards (completion donut, status split, per‑type breakdown for Epics/Stories/Tasks/Bugs).
2. **Product Backlog Health** — flow (created vs. resolved), release progress by Fix Version, and lead/cycle time.
3. **Sprint Health** — active‑sprint hero, completion, sprint health (work vs. time), velocity forecast, burndown, and a work‑items table.

It’s designed to run as a **live artifact inside [Claude Cowork](https://claude.com/)**, pulling
fresh data from your connected Atlassian/Jira account every time you open it.

> ⚠️ **Read the [Security](#-security--before-you-push-to-a-public-repo) section before you push this to a public GitHub repo.** The working copy contains identifiers for *your* Jira instance that you should replace with placeholders first.

---

## Features

- One shared header with a connection badge, left‑aligned level tabs, and a consistent controls card on every tab.
- **Click‑to‑drill / cross‑tab context:** pick a project on Portfolio and it carries into the Backlog and Sprint tabs.
- Graceful handling when a carried project isn’t a Scrum/software project (Sprint shows a note and falls back).
- **Always‑fresh data:** every tab switch re‑fetches; queries are cache‑busted; a tab’s background work is cancelled when you leave it.
- Each level is CSS‑isolated in its own Shadow DOM view, so their styles never collide.
- Charts via Chart.js (loaded from CDN).

---

## How it works

- The whole app is one file: **`index.html`** (HTML + CSS + JS, no build step).
- It calls Jira through Cowork’s connector bridge — `window.cowork.callMcpTool(name, args)` —
  **not** a hard‑coded API key. Authentication happens once, when you connect Jira inside Cowork (OAuth).
- All connector calls are serialized (one at a time) and cache‑busted so tabs are always up to date.

Because it relies on `window.cowork`, the dashboard only shows live data **when opened as a live
artifact inside Cowork**. Opening the raw `index.html` in a normal browser will show a “can’t reach
the Jira connector” banner (there’s no bridge outside Cowork).

---

## Requirements

- **Claude Cowork** (desktop) with the **Atlassian / Jira connector authorized**.
- A Jira Cloud instance you have access to.
- Internet access to the Chart.js CDN (already referenced in the file).

---

## Quick start

1. **Fork / clone** this repo.
2. Open `index.html` and set the [configuration values](#configuration) for your Jira instance.
3. In Claude Cowork, add it as a live artifact (or ask Cowork to open/recreate the artifact from this file).
4. Make sure your **Atlassian/Jira connector is connected** in Cowork.
5. Open the artifact from the Cowork sidebar → the connection badge should read **Connected**.
6. On **Portfolio Health**, select one or more projects → then click through **Product Backlog Health** and **Sprint Health**.

---

## Configuration

Open `index.html` and update these **instance‑specific values** (they appear near the top of the
inline `<script>`, and the tool list is declared in the `cowork-artifact-meta` block at the very top):

| Value | What it is | Example / placeholder |
|-------|------------|-----------------------|
| `CLOUD_ID` | Your Atlassian **Cloud ID** (the site’s UUID) | `YOUR_CLOUD_ID` |
| `SITE_HOST` | Your Jira site host (used to build issue links) | `your-company.atlassian.net` |
| `mcp__<server-id>__...` | The ID of *your* connected Atlassian connector in Cowork | `mcp__YOUR_CONNECTOR_ID__...` |

**Find your Cloud ID:** while logged into Jira, open
`https://<your-site>.atlassian.net/_edge/tenant_info` — the `cloudId` field is the value you need.

**Find your connector ID:** it’s the `mcp__<id>__...` prefix used in the `callMcpTool` calls and in
the `cowork-artifact-meta` → `mcpTools` list. When Cowork (re)creates the artifact against your own
connected Jira, it wires this up for you; if you’re editing by hand, replace the `<id>` to match your
connected Atlassian connector.

The connector tools this dashboard uses (declared in `cowork-artifact-meta`):

- `getAccessibleAtlassianResources`
- `getVisibleJiraProjects`
- `searchJiraIssuesUsingJql`
- `getJiraIssue`

---

## Where does an API token go?

**As shipped, this dashboard does not use — and must not contain — an API token.** Auth is handled
by Cowork’s Jira connector (OAuth). There is nothing secret to paste into the HTML for the normal
Cowork use case.

You only need a **Jira API token** if you fork this to call the Jira REST API **directly** (for
example, to run it as a standalone web page outside Cowork). In that case:

1. Create a token at **https://id.atlassian.com/manage-profile/security/api-tokens**.
2. Add a small config block near the top of your script and reference it in your own `fetch()` /
   Authorization header — **using placeholders, never your real token in the committed file**:

   ```js
   // --- Direct Jira REST API config (only if you adapt this outside Cowork) ---
   const JIRA_EMAIL     = "YOUR_ATLASSIAN_EMAIL";      // e.g. you@company.com
   const JIRA_API_TOKEN = "YOUR_JIRA_API_TOKEN";       // ⚠️ never commit the real value
   const SITE_HOST      = "your-company.atlassian.net";
   // Authorization: 'Basic ' + btoa(`${JIRA_EMAIL}:${JIRA_API_TOKEN}`)
   ```

   > ⚠️ Browsers block cross‑origin calls to `*.atlassian.net`, and Basic‑auth tokens embedded in a
   > public page are visible to anyone. For a real standalone app, proxy Jira through a small backend
   > and keep the token as a **server‑side environment variable** — never in front‑end code.

---

## 🔐 Security — before you push to a public repo

Treat the following as **sensitive** and scrub them before publishing:

- `JIRA_API_TOKEN` (if you added one) — **highest priority; a leaked token can act as you.**
- `CLOUD_ID` — identifies your Atlassian tenant.
- `SITE_HOST` — your Jira site.
- The `mcp__<connector-id>__...` server ID.

### Recommended workflow

1. **Keep your real, working file private.** Save your configured copy as `index.local.html`
   (or `.env`‑style values) that never gets committed.
2. **Commit a placeholder version.** In the committed `index.html`, replace real values with
   placeholders: `YOUR_CLOUD_ID`, `your-company.atlassian.net`, `YOUR_JIRA_API_TOKEN`,
   `mcp__YOUR_CONNECTOR_ID__...`.
3. **Add a `.gitignore`** so local copies and secrets never get pushed:

   ```gitignore
   # Local, configured copies — never commit
   *.local.html
   .env
   .env.*
   secrets.*
   ```

4. **Pre‑push checklist** — run these and confirm they return nothing before pushing:

   ```bash
   # Should print NOTHING if you sanitized correctly
   git grep -nE "atlassian\.net|[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}|API_TOKEN|ATATT" -- . ':!README.md'
   ```

   (`ATATT` is the prefix of Atlassian API tokens; the UUID pattern catches Cloud IDs.)

5. If you ever commit a real token by accident, **revoke it immediately** in your Atlassian API‑token
   settings and rotate it — deleting the commit is not enough once it’s public.

> Note: GitHub push protection and secret scanning may catch some of these automatically, but do
> **not** rely on it. Scrub first.

---

## Customizing

- **Change which projects/period load by default** — see the query‑building code in each view.
- **Add metrics** — the fetch → aggregate → render pattern is repeated per level and easy to extend.
- **Restyle** — design tokens (colors, spacing) live in the `:host` blocks of each `<template>` so
  you can retheme each level independently.

---

## License

Released under the [MIT License](LICENSE) — you're free to use, modify, and redistribute it,
including in your own forks and internal tools. Keep the copyright notice, and remember to scrub
your own Jira identifiers (see [Security](#-security--before-you-push-to-a-public-repo)) before
publishing a fork.

---

## Credits

Built as a Claude Cowork live artifact. Not affiliated with or endorsed by Atlassian. “Jira” is a
trademark of Atlassian.
