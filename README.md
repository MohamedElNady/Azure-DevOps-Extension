# Azure DevOps for Mendix Studio Pro

Bring your entire Azure DevOps workflow into Mendix Studio Pro — without switching windows.

This extension adds a persistent dockable panel to Studio Pro that gives your development team
instant access to work items, CI/CD pipelines, pull requests, wiki, and dashboards, all authenticated
with your existing Azure DevOps credentials.

Works with **Azure DevOps Services (cloud)** and **Azure DevOps Server (on-premises, 2019 and later)**.

---

## Why This Extension?

Mendix developers constantly context-switch between Studio Pro and Azure DevOps: checking which
work item to pick up next, monitoring a pipeline triggered by a recent commit, reviewing a PR, or
reading a wiki page before implementing a feature. This extension eliminates that context switch by
embedding the full ADO experience where you already work.

---

## Key Features

### Work Items & Kanban Board

- **List view** and **Kanban board** with drag-and-drop between state columns
- Create work items without leaving Studio Pro — Bug, User Story, Task, Epic, Feature
- Full detail panel: edit state, priority, assignee, description, repro steps, acceptance criteria
- **Rich text editing** with Bold/Italic/lists/code, `@mention` team members, `#link` work items, inline images (paste, drag-and-drop, or upload button)
- View comments, links, attachments, and revision history in one place
- Bulk actions: update state, priority, or assignee on multiple items at once
- Copy work item ID to clipboard with one click

### CI/CD Pipelines

- Browse build definitions by folder, trigger builds, and view run history
- **Live log streaming** — auto-scrolls while a build is in progress
- Download build artifacts directly from the panel
- View test results (pass/fail/skip) per build
- Full release pipeline support: create releases, monitor environments per stage
- **Retry failed deployment stage** without creating a new release
- Approve or reject both build and release gates with an optional comment
- Approval badge on the Pipelines tab with toast notifications for new approvals

### Pull Requests

- Create pull requests with reviewer multi-select (loaded from your team roster)
- Vote: Approve / Approve with suggestions / Reject — list updates instantly
- **Complete options**: choose merge strategy (Squash / Rebase / Rebase+Merge / Merge commit), delete source branch, set squash commit message
- Review threads with full rich text commenting
- PR removed from list immediately after complete or abandon

### Wiki

- Browse the full page tree for any wiki in your project
- Read pages rendered as formatted Markdown
- Edit existing pages or create new ones directly from the panel

### Dashboard

- Current sprint progress: completion percentage, stacked bar by state, burndown chart
- Pending pipeline and release approvals — approve or reject without opening ADO
- Active builds and active pull requests at a glance
- Work items assigned to you
- Recently modified work items (last 14 days)
- Recent pipeline runs across all definitions

### Multi-Project Support

- Save and switch between multiple Azure DevOps project connections
- Each connection has its own name, organization URL, project, PAT, and default repository
- Switch projects instantly from the header — all tabs reload automatically

---

## Requirements

- **Mendix Studio Pro** 10.12 or later (Windows only)
- **Azure DevOps Services** (cloud) — any region
- **Azure DevOps Server** 2019, 2020, or 2022 (on-premises)
- A Personal Access Token (PAT) with the scopes listed below

---

## Installation

1. Open Mendix Studio Pro
2. Go to **Marketplace** in the top menu
3. Search for **Azure DevOps**
4. Click **Download** on the extension listing
5. Accept the trust dialog when prompted
6. The **Azure DevOps** item appears in the **Extensions** menu

---

## Configuration

1. Go to **Extensions → Azure DevOps** in Studio Pro
2. The panel opens — click **Settings** (gear icon)
3. Click **Add Connection** and fill in:

| Field | Description |
|---|---|
| Name | A friendly label shown in the project switcher (e.g. "Production") |
| Organization URL | `https://dev.azure.com/yourorg` for cloud, or `https://your-server/DefaultCollection` for on-prem |
| Project | The exact ADO project name |
| Personal Access Token | A PAT with the required scopes (see below) |
| Default Repository | (Optional) pre-selects a repository in the Pull Requests tab |

4. Click **Test Connection** to verify, then **Save**

Your PAT is encrypted immediately with Windows DPAPI and is never stored in plain text or sent to
any third party. Only the signed-in Windows user account can decrypt it.

### General settings

The **General** section at the top of the Settings tab holds settings that apply to the whole app:

| Setting | Default | Notes |
|---|---|---|
| Server Port | `5678` | The port the extension serves its panel and API on. Change it if another application already uses 5678. |

Changing the port takes effect **after Studio Pro restarts** — the panel you're looking at is served
from the port being changed, so it can't move underneath itself. The port is rejected if another
application already holds it, and if it becomes unavailable later the extension falls back to a free
port automatically rather than failing to start.

### Required PAT Scopes

| Scope | Required for |
|---|---|
| Work Items — Read & Write | Work items, comments, attachments, inline images |
| Build — Read | Build list, live logs, artifact downloads, test results |
| Release — Read & Write | Release list, logs, retry stage, approvals |
| Code — Read | Pull requests, repositories, branches |
| Graph — Read | Team member list, @mention lookup |
| Wiki — Read & Write | Wiki page read and edit |

---

## Keyboard Shortcuts

| Key combination | Action |
|---|---|
| `/` | Focus the global search bar |
| `G` then `H` | Go to Home / Dashboard |
| `G` then `W` | Go to Work Items |
| `G` then `P` | Go to Pipelines |
| `G` then `R` | Go to Pull Requests |

---

## API Version Compatibility

The extension automatically detects your ADO version and negotiates the correct API version on first
connection. No manual configuration is needed.

| Environment | API Version | Status |
|---|---|---|
| Azure DevOps Services (cloud) | v7.1 / v7.0 / v6.x | Full support |
| Azure DevOps Server 2022 | v7.1 / v7.0 | Full support |
| Azure DevOps Server 2020 | v6.1 / v6.0 | Full support |
| Azure DevOps Server 2019 | v5.1 / v5.0 | Full support (some features with fallback) |

For detailed per-feature compatibility information, see the project's
[COMPATIBILITY.md](https://github.com/your-repo/AzureDevOps/blob/main/COMPATIBILITY.md).

---

## Security & Privacy

- Your PAT is encrypted with **Windows DPAPI** (`DataProtectionScope.CurrentUser`) and stored in `settings.json` in your Mendix app folder — only your Windows user account can decrypt it
- Settings are **per-Mendix-project**: each app you open in Studio Pro has its own isolated credentials, so switching between apps never mixes configurations
- The React frontend never receives your PAT — all Azure DevOps calls are made server-side
- **The local API requires a session token.** The extension runs a local HTTP server (default `localhost:5678`). It generates a random token each time it starts and gives it only to the panel inside Studio Pro. Web pages you visit in a browser cannot use the API, and opening the port in a browser shows a small status page rather than anything sensitive
- **Credentials only go to your Azure DevOps host.** Requests that carry your PAT are restricted to your configured organization (plus Microsoft's own Azure DevOps domains for cloud accounts); any other destination is refused
- **Content is sanitized before display.** Work item descriptions, comments, and wiki pages are authored by anyone with write access to your project, so they are cleaned of scripts and event handlers before rendering
- No telemetry, no analytics, no external services — nothing leaves your machine except calls to your own Azure DevOps instance

---

## Troubleshooting

**The panel shows a status card instead of the app.**
That page appears when the panel is opened outside Studio Pro, or before the extension finished
starting. Close the panel and reopen it from **Extensions → Azure DevOps**. Typing the address into
a browser will always show it — the interface only works inside Studio Pro.

**My connection or port setting disappeared after restarting.**
Fixed in 2.0.0. Earlier versions stored settings in a Studio Pro cache folder that is cleared when
the app re-syncs. After upgrading, check **Settings → Build → Settings** — the path shown should be
inside your app folder. A red warning appears if it ever resolves somewhere unstable.

**I changed the port but nothing happened.**
Port changes apply when Studio Pro restarts. After restarting, the General section shows the port
actually in use.

**Something else is using port 5678.**
Set a different port in **Settings → General** and restart. If the port is unavailable at startup,
the extension picks a free one automatically rather than failing.

**Did my update actually install?**
**Settings → Build** shows the version and build time of both halves of the extension, plus when
Studio Pro last loaded it. If **DLL built** is newer than **Loaded at**, restart Studio Pro.

---

## Release Notes

### 2.0.0 — August 2026

A security-focused release. Full notes: `RELEASE_NOTES.md`.

**Security**
- The local API now requires a per-session token, so no web page you visit can drive the extension or reach your credentials
- Requests carrying your PAT are restricted to your own Azure DevOps host
- Work item, comment, and wiki content is sanitized before rendering

**Fixed**
- Connections and settings no longer reset when Studio Pro restarts
- Inline images stay visible while editing a description instead of appearing to vanish
- The panel no longer needs to be closed and reopened after Studio Pro starts

**New**
- Configurable server port with automatic fallback if the port is taken
- Build panel in Settings showing versions, load time, and settings location
- Status page and `/health` endpoint on the extension's port

**Upgrading:** restart Studio Pro after installing — reloading the extension is not enough. Your
connections carry over; if they don't appear, re-add the connection once and it will persist.

### 1.1.0 — April 2026

**New**
- Inline image support in all rich text fields (comments, description, repro steps, acceptance criteria): paste, drag-and-drop, or upload button
  - Images upload as actual ADO attachments (byte-safe binary proxy) with `AttachedFile` relation linking for reliable rendering in ADO web UI
  - Image-containing comments automatically route through `System.History` PATCH (ADO threaded comments API strips `<img>` tags)
- Retry failed release stage button (`↺ Retry`) on any Failed/Rejected/Canceled environment
- Artifact downloads from build detail panel
- Test run results (pass/fail/skip) in build detail
- Complete PR options panel: merge strategy selector, delete-branch checkbox, squash commit message
- PR vote and complete/abandon update the list instantly without closing the detail panel
- `@mention` and `#work item` links in rich text editor with live dropdown search
- Copy work item ID to clipboard (click the `#ID` badge in the detail header)
- "Recently Modified by Me" dashboard widget (last 14 days)
- "Recent Pipeline Runs" dashboard widget (last 10 builds across all definitions)
- Multi-project connections: add, edit, remove, switch from header dropdown
- Keyboard shortcuts: `/` for search, `G+H/W/P/R` for tab navigation
- Global search across work items, builds, releases, and pull requests

**Improved**
- Kanban board quick-create: `+` button per column pre-fills state
- Board drag-and-drop between state columns
- Bulk actions floating bar for multi-select updates
- Responsive layout — icon-only nav at narrow widths
- PAT encrypted with Windows DPAPI (migrates automatically from plain-text)
- API version auto-negotiation for cloud and on-premises in one install

---

## Support

- **Source code & issues:** [GitHub repository](https://github.com/your-repo/AzureDevOps)
- **Bug reports:** Open a GitHub issue with your ADO version, Studio Pro version, and steps to reproduce
- **Feature requests:** Open a GitHub discussion or issue tagged `enhancement`
