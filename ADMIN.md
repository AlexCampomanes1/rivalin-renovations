# Client admin page — how it works

This folder ships with `admin-template.html`, copied verbatim from the Ultra
Flow builder (`server/admin-template.html`). It is the page the **site owner**
(Rivalin) uses to add or remove gallery photos themselves — no developer,
no Claude, no rebuild.

This doc covers:

1. What the admin page is (architecture)
2. How the builder produces and deploys it (so you can reproduce it)
3. How the end user actually uses it
4. The state for **this** detached Rivalin folder (important — not plug-and-play)

---

## 1. What it is

A **single static HTML file** (`admin-template.html`) with everything inline:
HTML, CSS, JS. No build step, no framework, no server.

Architecture:

```
┌──────────────────────────────┐
│  admin.html (this template)  │       ┌──────────────────────────────┐
│  ──────────────────────────  │       │  Azure Storage account       │
│   • PIN lock screen          │       │  ┌────────────────────────┐  │
│   • Upload tab (drag/drop)   │ ───▶  │  │  container: gallery    │  │
│   • Manage tab (grid)        │       │  │   ├ photo1.jpg         │  │
│   • Change PIN tab           │       │  │   ├ photo2.jpg         │  │
│   • Stats placeholder        │       │  │   └ gallery-config.json│  │
│                              │       │  └────────────────────────┘  │
│  Talks DIRECTLY to Azure     │       └──────────────────────────────┘
│  Blob REST API via SAS token │       SAS token is baked into admin.html
└──────────────────────────────┘       at deploy time (rwdl perms, expires 2031)
```

Key idea: there is **no backend** sitting between the admin page and storage.
The page makes:

- `GET  /<container>?restype=container&comp=list&prefix=&<SAS>` → list blobs
- `PUT  /<container>/<blob>?<SAS>`                              → upload
- `DELETE /<container>/<blob>?<SAS>`                            → delete
- `GET / PUT /<container>/gallery-config.json?<SAS>`            → load/save PIN hash

The same `gallery` container is also where the public site reads photos
from. So the moment the owner uploads, the photo is live.

### Placeholders

The template has four `{{...}}` tokens filled at deploy time:

| Placeholder         | Filled with                                  |
| ------------------- | -------------------------------------------- |
| `{{BUSINESS_NAME}}` | `project.businessName` (HTML-escaped)        |
| `{{SA}}`            | Azure storage account name (e.g. `rivalin01`) |
| `{{SAS_TOKEN}}`     | container-scoped SAS (rwdl, expires 2031)    |
| `{{SESSION_KEY}}`   | `uf-admin-<projectId>` — keeps each site's unlock session separate in `sessionStorage` |

### PIN model

- Master / break-glass PIN: hardcoded `6969` in the file (so Alex can always get in).
- Owner PIN: starts at `project.galleryPin || '1234'`, hashed with SHA-256, stored
  in `gallery-config.json` inside the blob container.
- The "Change PIN" tab rewrites that file. No server round-trip — the SAS lets
  the page `PUT` directly to the blob.

---

## 2. How the builder produces and deploys it

The relevant code is in `server/index.js` — the `/api/projects/:id/deploy`
handler (around line 2456 onward). Sequence per site:

1. Create resource group + storage account named after the project
   (`ultraflow_<projectId>` / `ultraflow<slug>`).
2. Enable static-website hosting on the account.
3. Create two containers: `$web` (the public site) and `gallery` (user photos
   + admin config).
4. Set blob-level CORS so the admin page can hit the blob REST API from
   `https://<account>.z9.web.core.windows.net` and from any custom domain.
5. Upload generated `index.html` to `$web`. Inline-rewrite the
   `gallery-manifest` fetch URL from the builder-API path
   (`/api/projects/<id>/gallery-manifest`) to the deployed relative path
   `gallery-manifest.json`.
6. Upload `gallery-manifest.json` listing the initial photos.
7. Copy every photo from the builder's local `<DATA_DIR>/gallery/<projectId>/`
   into the `gallery` container.
8. **Render admin.html**:
   - `az storage account keys list ...` → get the account key.
   - `az storage container generate-sas --name gallery --permissions rwdl
     --expiry 2031-01-01T00:00:00Z --https-only` → mint SAS token.
   - Read `admin-template.html`, replace the four placeholders, upload to
     `$web/admin.html`.
9. Hash the initial PIN (SHA-256) and write `gallery-config.json` into the
   `gallery` container with `{ "pinHash": "<hex>" }`.

End result:

- Public site:  `https://<account>.z9.web.core.windows.net/`
- Admin page:   `https://<account>.z9.web.core.windows.net/admin.html`

Both behind the same custom domain once DNS is pointed at the storage account
(or via Azure Front Door for HTTPS on apex).

---

## 3. How the end user uses it

The site owner gets:

- A URL: `<their-domain>/admin.html`
- A starter PIN (default `1234`; can be set per project at build time)

Flow:

1. Open admin URL on phone or laptop.
2. Type 4-digit PIN. If correct, the lock screen fades and the app tabs
   appear. Session persists in `sessionStorage` until the tab is closed (or
   until they hit the lock icon in the header).
3. **Upload tab**: drag/drop photos (or tap to pick from camera roll). Files
   are queued, each shows its own progress bar, and each is `PUT` directly
   to blob storage. As soon as upload completes, the file is live on the
   public site.
4. **Manage tab**: shows a grid of every photo in the `gallery` container,
   newest first. Tap to select (multi-select supported). Hit "Delete
   selected" → confirm → `DELETE`s straight from blob. Grid reloads.
5. **Change PIN tab**: enter current PIN, new PIN twice. SHA-256 of the new
   PIN is written to `gallery-config.json`. Next session uses the new hash.
6. Lock icon (top right) clears the session immediately.

Notes:

- Videos (`.mp4`, `.mov`, `.webm`) work too — manage grid shows a hover-play
  preview.
- No file-size validation, no image resizing on the client. Stored as-is.
- The public-facing site reads either `gallery-manifest.json` (built sites)
  or lists the container directly (depends on the gallery loader Claude
  emitted).

---

## 4. State for **this** detached Rivalin folder

**The `admin-template.html` in this folder will not work as-is.**

It's the template, not a rendered admin page. Two things are missing:

1. The four `{{...}}` placeholders are still literal placeholder text.
2. There is no Azure storage account behind this folder — Rivalin is
   currently a pure static repo. There is no SAS token to fill in because
   there is nothing to talk to.

You have two practical paths forward. Pick one when you're ready:

### Path A — Wire Rivalin up to Azure blob (recommended)

Same architecture as every other Ultra Flow site. One-time setup:

1. `az group create -n rivalin-rg -l canadacentral`
2. `az storage account create -n rivalinsite -g rivalin-rg --sku Standard_LRS --kind StorageV2`
3. `az storage blob service-properties update --account-name rivalinsite --static-website --index-document index.html`
4. `az storage container create --account-name rivalinsite -n gallery --public-access blob`
5. Set CORS on the storage account (see builder code line ~2517).
6. Generate a SAS for the `gallery` container with `rwdl` perms.
7. Render `admin-template.html` locally: substitute `{{SA}}`, `{{SAS_TOKEN}}`,
   `{{BUSINESS_NAME}}` = `Rivalin Renovations`, `{{SESSION_KEY}}` = `uf-admin-rivalin`.
8. Save the rendered file as `admin.html`, upload to the `$web` container.
9. Upload a starter `gallery-config.json` to the `gallery` container with the
   SHA-256 of `1234` (or whatever PIN you want):
   `{"pinHash":"03ac674216f3e15c761ee1a5e255f067953623c8b388b4459e13f978d7c846f4"}`
10. Update Rivalin's `index.html` `loadGallery()` to either list the gallery
    container directly (with a read-only SAS) or fetch a `gallery-manifest.json`
    that the admin page maintains.

After that, the workflow in §3 above just works.

### Path B — Pure-static "manifest-only" admin (no Azure)

If you want to stay 100% static (no Azure, no SAS), the admin page can be
rewritten to read/write a JSON file in this repo via the GitHub API using a
fine-scoped PAT. This is what `HANDOFF.md §11` already gestures at with
`assets/photos/gallery/manifest.json`. The trade-offs:

- Pro: zero infra.
- Pro: the owner's uploads land in this Git repo — versioned, recoverable.
- Con: needs a GitHub PAT baked into the admin page (or a tiny serverless
  function to mint short-lived tokens), which is a different security model
  than the SAS approach.
- Con: image storage in Git is fine for a few dozen photos; not for hundreds.

Recommend Path A. Path B is documented only as a fallback.

---

## File reference

- `admin-template.html` — copied from `ultraflow/server/admin-template.html`.
  Do not edit this in place if you plan to keep it in sync with the builder.
  Treat it as a snapshot; future builder improvements should be pulled in
  by re-copying.
- Source of truth in the builder repo: `server/admin-template.html`
- Deploy logic that fills the template:
  `server/index.js` → `/api/projects/:id/deploy` handler (~line 2456+).
