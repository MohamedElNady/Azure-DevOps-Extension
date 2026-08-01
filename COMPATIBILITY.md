# Azure DevOps Extension — API Version Compatibility

## Supported Environments

| Environment | API Version | Status |
|---|---|---|
| **Azure DevOps Services (Cloud)** | v7.1, v7.0, v6.x | Full support |
| **Azure DevOps Server 2022** | v7.1, v7.0 | Full support |
| **Azure DevOps Server 2020** | v6.1, v6.0 | Full support |
| **Azure DevOps Server 2019** | v5.1, v5.0 | Full support (some features with fallback) |
| **Team Foundation Server 2018** | v4.1 | Partial — limited WIQL, no approvals |

---

## How Version Negotiation Works

```
First API call
    │
    ▼
Try v7.1 (cloud maximum)
    │
    ├─ Success → persist v7.1, use for all subsequent calls
    │
    └─ 400 Bad Request (server doesn't support v7.1)
           │
           ▼
       Extract server's max version from error message
           │
           ▼
       Retry with supported version (e.g. v5.1)
           │
           ▼
       Persist version in settings file — no re-negotiation needed
```

**Manual override** — edit `settings.json`. Since 2.0.0 it lives at
`{AppDir}\extensions\MendixAzureDevOpsExtension\settings.json`, with
`%APPDATA%\MendixAzureDevOpsExtension\settings.json` as the fallback. The exact path in use is
shown in **Settings → Build → Settings**.
```json
{
  "connections": [
    {
      "orgUrl": "https://azuredevops.yourserver.com/DefaultCollection",
      "project": "YourProject",
      "patEncrypted": "...",
      "apiVersion": "5.1"
    }
  ]
}
```

---

## Feature Support Matrix

```
┌──────────────────────────────────────┬──────┬──────┬──────┬──────┐
│ Feature                              │ v5.1 │ v6.x │ v7.0 │ v7.1 │
├──────────────────────────────────────┼──────┼──────┼──────┼──────┤
│ Work Items — list / detail           │  ✅  │  ✅  │  ✅  │  ✅  │
│ Work Item — create (with state)      │  ✅  │  ✅  │  ✅  │  ✅  │
│ Work Item — type-specific fields     │  ✅  │  ✅  │  ✅  │  ✅  │
│ Work Item — inline images (upload)   │  ✅  │  ✅  │  ✅  │  ✅  │
│ Work Item — comments (threaded)      │  ⚠️  │  ✅  │  ✅  │  ✅  │
│ Work Item — comments (img, History)  │  ✅  │  ✅  │  ✅  │  ✅  │
│ Work Item — links / navigation       │  ✅  │  ✅  │  ✅  │  ✅  │
│ Work Item — attachments + download   │  ✅  │  ✅  │  ✅  │  ✅  │
│ Work Item — history                  │  ✅  │  ✅  │  ✅  │  ✅  │
│ Kanban board (drag-and-drop)         │  ✅  │  ✅  │  ✅  │  ✅  │
│ Board quick-create per column        │  ✅  │  ✅  │  ✅  │  ✅  │
│ Bulk actions (state/priority/assign) │  ✅  │  ✅  │  ✅  │  ✅  │
├──────────────────────────────────────┼──────┼──────┼──────┼──────┤
│ Builds — list / trigger              │  ✅  │  ✅  │  ✅  │  ✅  │
│ Builds — live logs                   │  ✅  │  ✅  │  ✅  │  ✅  │
│ Builds — artifact downloads          │  ✅  │  ✅  │  ✅  │  ✅  │
│ Builds — test results                │  ✅  │  ✅  │  ✅  │  ✅  │
│ Build approvals                      │  ⚠️  │  ✅  │  ✅  │  ✅  │
│ Releases — list / create             │  ✅  │  ✅  │  ✅  │  ✅  │
│ Releases — retry failed stage        │  ✅  │  ✅  │  ✅  │  ✅  │
│ Release task logs                    │  ✅  │  ✅  │  ✅  │  ✅  │
│ Release approvals + comment          │  ⚠️  │  ✅  │  ✅  │  ✅  │
│ Approval badge + toast notification  │  ⚠️  │  ✅  │  ✅  │  ✅  │
├──────────────────────────────────────┼──────┼──────┼──────┼──────┤
│ Pull Requests — list / create        │  ✅  │  ✅  │  ✅  │  ✅  │
│ PR — vote (instant list update)      │  ✅  │  ✅  │  ✅  │  ✅  │
│ PR — complete options panel          │  ✅  │  ✅  │  ✅  │  ✅  │
│ PR — complete / abandon (instant)    │  ✅  │  ✅  │  ✅  │  ✅  │
│ PR — reviewer multi-select           │  ✅  │  ✅  │  ✅  │  ✅  │
│ PR — review threads + comments       │  ✅  │  ✅  │  ✅  │  ✅  │
├──────────────────────────────────────┼──────┼──────┼──────┼──────┤
│ Wiki — read pages                    │  ✅  │  ✅  │  ✅  │  ✅  │
│ Wiki — create / edit pages           │  ✅  │  ✅  │  ✅  │  ✅  │
├──────────────────────────────────────┼──────┼──────┼──────┼──────┤
│ Sprint progress card                 │  ✅  │  ✅  │  ✅  │  ✅  │
│ Burndown chart                       │  ✅* │  ✅* │  ✅* │  ✅* │
│ Dashboard — recent work items (@Me)  │  ⚠️  │  ✅  │  ✅  │  ✅  │
│ Dashboard — recent pipeline runs     │  ✅  │  ✅  │  ✅  │  ✅  │
│ Dashboard charts                     │  ✅  │  ✅  │  ✅  │  ✅  │
├──────────────────────────────────────┼──────┼──────┼──────┼──────┤
│ Multi-project connections            │  ✅  │  ✅  │  ✅  │  ✅  │
│ PAT encryption (Windows DPAPI)       │  ✅  │  ✅  │  ✅  │  ✅  │
│ Global search + / shortcut           │  ✅  │  ✅  │  ✅  │  ✅  │
│ Keyboard shortcuts (G+H/W/P/R)       │  ✅  │  ✅  │  ✅  │  ✅  │
│ Inline image upload (byte-safe)      │  ✅  │  ✅  │  ✅  │  ✅  │
└──────────────────────────────────────┴──────┴──────┴──────┴──────┘

✅  = Fully supported
⚠️  = Supported with fallback or limited (see notes below)
*   = Requires sprint start/finish dates set in ADO project settings
```

---

## Fallback Details

### Comments — v5.1 with `-preview` suffix
The work item comments API requires `api-version=X.X-preview.1` on all v5.x servers.
The extension uses `VPreview()` automatically. If the preview endpoint is unavailable, comments fall
back to writing `System.History` directly (which always works but renders as a history entry rather
than a threaded comment).

### Inline Images in Comments
Images in comments always route through `System.History` PATCH regardless of API version — the
threaded comments API on all versions strips `<img>` tags.

Pasted images are uploaded as real Azure DevOps attachments (byte-safe binary upload) and linked to
the work item with an `AttachedFile` relation, so they render correctly in the Azure DevOps web UI
as well. This works identically on v5.1 through v7.1 — the attachment endpoint and relation syntax
are unchanged across versions.

### Pipeline / Release Approvals — v5.1
The build approvals API may not be available on all ADO Server 2019 installations.
The extension silently shows an empty approvals list in that case.

### "Recently Modified by Me" Dashboard Widget — v5.1
Uses `[System.ChangedBy] = @Me` in WIQL. Some ADO Server 2019 configurations do not support `@Me`
in the `ChangedBy` field. The widget returns an empty list silently; no error is shown.

### Sprint Overview — All Versions
Built entirely client-side using two stable endpoints (iteration list + work item query), avoiding
the team-context URL that varies between cloud and on-prem. Works on all supported versions.

---

## Version-Specific Notes

| Version | Limitation |
|---|---|
| v5.0–v5.1 | Pipeline approvals API may be unavailable |
| v5.0–v5.1 | `@Me` in `ChangedBy` WIQL may not be supported — "Recently Modified by Me" shows empty |
| TFS 2018 (v4.1) | Limited WIQL syntax; some filters may not work; approvals not supported |
| All versions | Burndown chart requires sprint start/finish dates configured in ADO |
| All versions | Release retry requires **Release (Write)** PAT scope |
| All versions | Inline image uploads are capped at 2 MB per image (ADO field size budget) |
