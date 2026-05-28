# Handoff — Rivalin Renovations standalone

For: future-self / collaborator picking up this folder.
Audience: someone who knows the Ultra Flow builder context but is now working
on this site as a standalone artifact.

## 1. Why this folder exists

The Rivalin site was originally built by the Ultra Flow AI builder
(`proj_4e713c3e`, status `built`, source `ai-intake`). We then decided to
detach it because:

- The client wanted **before & after** + a **richer gallery** treatment, and
  the builder's templated output isn't the right place to experiment with
  visual design — it's a generator, not an editor.
- Iterating in the builder means re-running Claude, paying tokens, and
  risking regressions in unrelated sections each time.
- Once the new gallery patterns are proven *here*, by hand, they get ported
  back into the builder as a clean opt-in module — predictable and cheap.

The user's quote that sparked it: *"I'd like to detach it. And start working
on it on its own, and then I'll probably do a before-and-after gallery and
then the admin which allows you to upload for the standard gallery will also
allow to load for the before-and-after gallery. Then once I get that figured
out then we can build it as a module in the builder."*

## 2. Where the data came from

| Source | What |
|--|--|
| `GET /api/projects/proj_4e713c3e` (builder) | `generatedHtml` → `index.html`; rest → `project.json` |
| `ultraflowuploads.blob.core.windows.net/uploads/proj_4e713c3e/logo-*.jpeg` | Downloaded → `assets/logo-1779489623548.jpeg` |
| 9× `images.pexels.com/photos/...` | Downloaded → `assets/pexels-photo-*.jpeg` |

All remote URLs have been rewritten to relative paths. Verify with:
```bash
grep -Eo 'https?://[^"\)\s]+' index.html | sort -u
```
The only remaining `https://` hits should be the `wa.me/` rewriter string
literals — no asset traffic leaves the page.

## 3. What was changed vs. the builder's raw output

In chronological order:

1. **Logo CSS polish** (patched via builder PATCH endpoint before detachment)
   - `.nav-logo img` → white pill, hairline blue border, soft shadow, 4×8 padding, rounded 6px
   - `.about-visual img` → `cover` → `contain`, 32px white padding, framed border (was cropping the logo)
   - `.footer-logo img` → white pill with deeper shadow (visible on dark footer)
2. **Nav logo size** — 40px → 50px desktop, with `@media (max-width: 768px) { .nav-logo img { height: 44px } }` so it doesn't crowd the hamburger
3. **All images localized** into `assets/` (Pexels + logo)
4. **Ultra Flow analytics block stripped** — the `<script>` that POSTed to
   `https://ultraflow-builder.azurewebsites.net/api/track` is gone. The
   WhatsApp deeplink rewriter (`wa.me/` → `whatsapp://`) was bundled with
   that block and got removed too — if a real WhatsApp CTA gets wired up
   later, re-add the deeplink swap inline near the CTA.

Everything else in `index.html` is the original Claude-generated output.

## 4. State of the builder relative to this folder

As of 2026-05-23, the builder's stored copy of `proj_4e713c3e` **still exists**.
It's the same `generatedHtml` we exported (post logo polish), and the builder
*could* technically rebuild it.

To fully lock the builder out, do one of:

### Option A — Soft detach (recommended for now)
Leave the project in the builder's `projects.json` but stop treating it as
buildable. Do nothing — just don't trigger a rebuild.

### Option B — Hard detach
Delete it from the builder:
```bash
source /tmp/uf-tok   # or refresh: POST /api/auth/login with {"password":"UF2026!"}
curl -X DELETE "https://ultraflow-builder.azurewebsites.net/api/projects/proj_4e713c3e" \
     -H "x-builder-token: $TOKEN"
```
Pros: zero risk of accidental rebuild, removes the project from the admin
list. Cons: irreversible from the builder side — but **all of the intake
data is preserved here in `project.json`**, so it's recoverable. The blob
container still holds the logo and any uploaded photos.

### Option C — Soft detach + flag
Add a `status: 'detached'` value to the builder's enum and make the build,
PATCH, and asset endpoints refuse to mutate detached projects. This is the
"right" answer if we ever detach more than a handful of sites. Not done yet.

## 5. How to work in this folder

```bash
cd ~/Repositories/rivalin-renovations

# Just open it
open index.html

# Or serve it (better — relative paths behave correctly + reload-on-save)
python3 -m http.server 8765
# → http://localhost:8765/
```

No build step. No dependencies. No package.json. Edit `index.html` directly.

For the upcoming gallery + before/after work, plan to:
1. Add `assets/gallery/` and `assets/before-after/` folders
2. Add `assets/gallery-manifest.json` + `assets/before-after-manifest.json`
3. Replace the current `#our-work` section's hardcoded `<img>` tags with a
   `fetch('./assets/gallery-manifest.json').then(render)` block
4. Add a new `#before-after` section using a small clip-path slider component
5. Build a one-file `admin.html` that lets Sheldon drag-drop new pairs and
   regenerates the manifests (purely client-side, downloads the JSON for
   manual commit — or wire to a tiny Node endpoint later)

## 6. Hosting — current state

**Live at:** `https://rivalinrenovations.z9.web.core.windows.net/`

Azure Blob static website hosting:
- Storage account: `rivalinrenovations` (RG: `rg-ultraflow-clients`, Canada Central)
- Container: `$web`, `indexDocument: index.html`
- Static website: enabled

To redeploy after edits:
```bash
source ~/.azure-cli-venv/bin/activate
az storage blob upload \
  --account-name rivalinrenovations \
  --container-name '$web' \
  --name index.html \
  --file ~/Repositories/rivalin-renovations/index.html \
  --content-type "text/html" \
  --auth-mode key \
  --overwrite
```

For a custom domain (`rivalinrenovations.com`), add a CNAME + Azure CDN endpoint.

The site is fully static — no backend.

## 7. Things to NOT do

- ❌ Do not regenerate this site via the builder. If you do and PATCH the
  generatedHtml back, you'll lose every hand-tuned CSS change here.
- ❌ Do not point this folder's `index.html` back at `ultraflowuploads.blob.*`
  URLs. Self-contained is the whole point.
- ❌ Do not silently bring back the analytics tracker. If the client wants
  analytics, install something honest (Plausible / Cloudflare / GA4) with
  consent.
- ❌ Do not rename `assets/logo-1779489623548.jpeg` without updating every
  reference in `index.html`. (Or do — but then `git mv` and grep-replace
  in one commit.)

## 8. Open questions / decisions parked

These came up during scoping; resolve before / during the before-after work:

1. **Shared portfolio pool or per-service pairs?** (e.g. one big "Our Work"
   pool vs. tabs for Kitchens / Bathrooms / Basements)
2. **Slider default state** — 50/50 on load, or auto-animate from "all before"
   → "all after" once when scrolled into view (more dramatic)?
3. **Video before/after?** A drone flyover of an exterior could be a powerful
   add — but doubles the data model complexity (2× MP4 vs. 2× JPG).
4. **Admin uploader scope** — local-only single-file tool, or proper hosted
   admin with auth? For one client, local is fine. If we generalize, need
   auth.
5. **Builder module port-back shape** — once before/after works here, what's
   the minimum viable schema we add to `project.beforeAfter` so Eve / the
   wizard can populate it? (Sketch already in main repo's session notes.)

## 9. Eve / intake fixes (live in the builder, not this folder)

For full traceability — two unrelated Eve bugs were fixed the same day
this folder was created. They're in the Ultra Flow repo, not here:

- **Typed messages not sending while Eve speaks** — `_voiceResume()` was
  clobbering `textInput.value`. Now preserves typed content + adds text
  barge-in (typing silences Eve).
- **First-load click + mic dialog race** — replaced the auto-start +
  on-first-gesture pattern with a proper "Tap to start" overlay that
  awaits `getUserMedia()` before letting Eve speak.

Mentioning these so nobody re-opens them as bugs on Rivalin.

## 10. Contact / continuity

- Original project ID in the builder: `proj_4e713c3e`
- Builder admin login: `POST /api/auth/login` with `{"password":"UF2026!"}`
  (don't commit the password anywhere in this folder)
- Builder URL: `https://ultraflow-builder.azurewebsites.net`
- Eve URL: `https://ultraflow.accelerateiq.ca/ai-intake.html`
- Logo blob (still live, unused here): `ultraflowuploads.blob.core.windows.net/uploads/proj_4e713c3e/`

If this folder gets stale and someone needs the absolute current builder
copy to compare: fetch it the same way it was bootstrapped (see §2).

---

## 11. Assets folder structure — DONE (2026-05-23)

This task is complete. The flat `assets/` layout has been migrated.

### What was done

- All 9 AI-placed Pexels photos moved into **section-named subfolders**
  with descriptive filenames (no `pexels-` prefix):
  - `assets/hero/` — 3 hero slideshow photos
  - `assets/services/` — 3 service-card photos
  - `assets/highlights/` — 3 highlights-card photos
- All 9 `<img src>` / `background-image` refs in `index.html` updated.
- `assets/gallery/` and `assets/before-after/` created as **intentionally
  empty placeholders**. Production photos for these sections live in Azure
  blob containers (`gallery`, `before-after`), not in Git.
- `assets/photo-attribution.json` added — maps each renamed file to its
  original photographer + Pexels source URL for license compliance.

### Current layout

```
assets/
├── logo-1779489623548.jpeg          ← logo (stays at root)
├── photo-attribution.json           ← Pexels license attribution
├── hero/                            ← design photos (deploy-managed)
│   ├── kitchen-minimal-1.jpeg
│   ├── kitchen-wood-2.jpeg
│   └── kitchen-contemporary-3.jpeg
├── services/                        ← design photos
│   ├── bathroom-marble.jpeg
│   ├── kitchen-modern.jpeg
│   └── bathroom-elegant.jpeg
├── highlights/                      ← design photos
│   ├── bathroom-glass-shower.jpeg
│   ├── bathroom-modern.jpeg
│   └── kitchen-bright.jpeg
├── about/                           ← empty; About section uses the logo
├── gallery/                         ← OWNER DATA placeholder (empty in dev)
│   └── README.md
└── before-after/                    ← OWNER DATA placeholder (empty in dev)
    └── README.md
```

### Hard rules for any future agent

- **NEVER commit photos into `assets/gallery/` or `assets/before-after/`.**
  Those belong to the site owner and are managed via `admin.html` → Azure blob.
- Design photos (`assets/hero/`, `assets/services/`, `assets/highlights/`,
  `assets/about/`) are deploy-managed — safe to swap, resize, or update.
- The logo at `assets/logo-*` is fine to replace with a new upload.
- Never rename a design photo without updating the corresponding `<img src>`
  or `background-image` ref in `index.html` in the same commit.

### Builder port-back

The full spec for carrying this pattern back to the Ultra Flow builder is in
[BUILDER-SPEC.md](BUILDER-SPEC.md). That document covers the Azure container
layout, deploy handler changes, manifest schemas, admin template changes,
and the migration plan for already-deployed sites.
