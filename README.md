# Ultimate Jira Projects Dashboard

A single self‑contained HTML dashboard that gives product and project managers three
drill‑down levels of Jira health in one place:

1. **Portfolio Health** — multi‑project scorecards (completion donut, status split, per‑type breakdown for Epics/Stories/Tasks/Bugs).
2. **Product Backlog Health** — flow (created vs. resolved), release progress by Fix Version, and lead/cycle time.
3. **Sprint Health** — active‑sprint hero, completion, sprint health (work vs. time), velocity forecast, burndown, and a work‑items table.

It’s designed to run as a **live artifact inside [Claude Cowork](https://claude.com/)**, pulling
fresh data from your connected Atlassian/Jira account every time you open it.

---

## Features

- One shared header with a connection badge, left‑aligned level tabs, and a consistent controls card on every tab.
- **Click‑to‑drill / cross‑tab context:** pick a project on Portfolio and it carries into the Backlog and Sprint tabs.
- Graceful handling when a carried project isn’t a Scrum/software project (Sprint shows a note and falls back).
- **Always‑fresh data:** every tab switch re‑fetches, so what you see is current.
- Charts via Chart.js (loaded from CDN).

---

## How it works

- The whole app is one file: **`Live Artefact/index.html`** (HTML + CSS + JS, no build step).
- It calls Jira through Cowork’s connector bridge — `window.cowork.callMcpTool(name, args)` —
  **not** a hard‑coded API key. Authentication happens once, when you connect Jira inside Cowork (OAuth).

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
2. Open `Live Artefact/index.html` and set the [configuration values](#configuration) for your Jira instance.
3. In Claude Cowork, add it as a live artifact (or ask Cowork to open/recreate the artifact from this file).
4. Make sure your **Atlassian/Jira connector is connected** in Cowork.
5. Open the artifact from the Cowork sidebar → the connection badge should read **Connected**.
6. On **Portfolio Health**, select one or more projects → then click through **Product Backlog Health** and **Sprint Health**.

---

## Configuration

Open `Live Artefact/index.html` and update these **instance‑specific values** (they appear near the top of the
inline `<script>`, and the tool list is declared in the `cowork-artifact-meta` block at the very top):

| Value | What it is | Example / placeholder |
|-------|------------|-----------------------|
| `CLOUD_ID` | Your Atlassian **Cloud ID** (the site’s UUID) | `YOUR_CLOUD_ID` |
| `SITE_HOST` | Your Jira site host (used to build issue links) | `your-company.atlassian.net` |
| `mcp__<server-id>__...` | The ID of *your* connected Atlassian connector in Cowork | `mcp__YOUR_CONNECTOR_ID__...` |

> If you publish a fork, swap these identifiers back to the placeholders above so you don't expose your own tenant.

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

## Customizing

- **Change which projects/period load by default** — see the query‑building code in each view.
- **Add metrics** — the fetch → aggregate → render pattern is repeated per level and easy to extend.
- **Restyle** — design tokens (colors, spacing) live in the `:host` blocks of each `<template>` so
  you can retheme each level independently.

---

## Screenshots

A per‑tab walkthrough with copy‑ready descriptions lives in [DESCRIPTION.md](Live%20Artefact/DESCRIPTION.md).
Data shown below is from a demo Jira instance.

### 1 · Portfolio Health

Multi‑project scorecards — completion donut, status split, and per‑type panels (Epics, Stories, Tasks, Bugs) for each selected project.

![Portfolio Health tab](Screenshot/jira-portfolio-health.png)

### 2 · Product Backlog Health

Flow health for one project — completion, created vs. resolved throughput, cumulative flow trend, and release progress by Fix Version.

![Product Backlog Health tab](Screenshot/jira-backlog-health.png)

### 3 · Sprint Health

A single sprint in detail — active‑sprint hero, completion, on‑track read, velocity forecast, burndown, and the work‑items table.

![Sprint Health tab](Screenshot/jira-sprint-health.png)

---

## License

Released under the [MIT License](LICENSE) — you're free to use, modify, and redistribute it,
including in your own forks and internal tools. Keep the copyright notice.
