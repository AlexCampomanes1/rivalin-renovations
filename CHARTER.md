# Site Charter — Rivalin Renovations

This is the **contract** every Ultra Flow site (including this detached one)
adheres to. It exists so a future agent — Claude, Copilot, a human dev —
can drop into this folder cold and know how the site is supposed to behave
without reading the builder's source.

Read this *before* you touch [index.html](index.html). Read
[SPECS.md](SPECS.md) for what *this* site is. Read [HANDOFF.md](HANDOFF.md)
for *why* it's detached.

---

## 1. Philosophy

- **Single-file static HTML.** All CSS in a `<style>` block at the top,
  all JS in a `<script>` at the bottom. No build step, no bundlers, no
  framework. If you can't open it with `open index.html`, you've broken it.
- **Mobile-first.** Every section must work at 360 px wide before it works
  at 1440 px.
- **No remote dependencies.** No CDN fonts, no analytics, no external CSS,
  no Google Maps embeds unless explicitly requested. System fonts only.
- **Accessibility is not optional.** Real `<button>` and `<a>` elements,
  semantic landmarks (`<nav> <main> <section> <footer>`), alt text on every
  image, focus styles preserved.
- **Performance budget.** Page weight ≤ 2 MB on first load. No image larger
  than 400 KB. Lazy-load anything below the fold.
- **One color accent.** A single `--accent` CSS custom property drives every
  CTA / heading / link. For Rivalin: `#2c5f8d`.

---

## 2. The standard section structure

Every Ultra Flow site uses the same top-to-bottom skeleton. Some sections
are **required**, some are **optional**. The contract is: *if a section is
present, it follows the rules below; if it's absent, the site still feels
complete.*

| # | Section | Required? | ID | Purpose |
|---|---------|-----------|----|---------|
| 1 | **Nav** | required | `<nav>` | Logo + 3–5 anchor links |
| 2 | **Hero** | required | `#hero` | One-screen pitch + primary CTA |
| 3 | **Services** | required | `#services` | What you sell, 2–6 cards |
| 4 | **About** | required | `#about` | Story + face/logo |
| 5 | **Highlights** | **optional** | `#highlights` | 2–4 standout claims (years, projects, reviews, certifications) |
| 6 | **Gallery** | **optional** | `#gallery` / `#work` | Portfolio / work samples |
| 7 | **Before & After** | **optional, vertical-specific** | `#before-after` | Transformation pairs (see §4) |
| 8 | **Testimonials** | optional | `#testimonials` | Quotes from real customers |
| 9 | **Contact** | required | `#contact` | Email, location, primary CTA |
| 10 | **Footer** | required | `<footer>` | Logo, copyright, secondary links |

**"Optional" means optional.** A site for an accountant probably has no
gallery and no before/after. A site for a renovator (this one) has both.
The site must look intentional either way — no empty divs, no "Coming
soon" placeholders, no `display:none` skeletons.

### Section ordering

The order above is canonical and should not change. The reasoning:
- Highlights, when present, sits **between About and Gallery** because it
  bridges story (About) and proof (Gallery).
- Before & After, when present, sits **after the standard Gallery** because
  it's a more powerful proof — you scroll past nice photos *into* dramatic
  transformations.
- If a site has Before & After but no standard Gallery, B/A takes the
  Gallery slot (#6).

---

## 3. Section-by-section rules

### Nav
- Logo on left, links on right (or hamburger on mobile ≤ 768 px)
- Logo image: white pill background (`#fff`), hairline accent border
  (`1px solid rgba(accent, 0.15)`), soft shadow, `4px 8px` padding,
  `border-radius: 6px`. Height: `50px` desktop, `44px` mobile.
- Links scroll-anchor to sections; sticky-positioned with `backdrop-filter`
  blur if the site has a photo-heavy hero.

### Hero
- One screen tall (`min-height: 100vh` or `90vh` if the nav is tall)
- Photo background OR colored gradient — never both
- Headline ≤ 8 words, sub-headline ≤ 20 words
- Exactly **one** primary CTA. A secondary CTA may appear as a ghost
  button next to it but never two filled buttons.

### Services
- 2–6 cards. 1-column mobile, 2-up tablet, 3-up desktop (4 if needed).
- Each card: icon or single-color graphic, title (≤ 4 words), description
  (≤ 25 words). No prices on cards — prices live on Contact / a separate
  pricing page if relevant.

### About
- Two-column desktop: copy on one side, image on the other. Stack on mobile.
- Image is either a real photo of the owner OR a treatment of the logo.
- For logo treatments: `object-fit: contain`, `padding: 32px`, white
  background, framed border — never `object-fit: cover` (cropping a logo
  is a cardinal sin).
- Copy ≤ 150 words. First sentence is a hook.

### Highlights (optional)
- 2–4 stat cards: big number + label + optional one-line context.
- Examples: "28 years experience", "200+ projects", "4.9★ Google rating".
- Numbers must be **true**, sourced from the intake. Never fabricate.
- Visual treatment: large display font for the number (3rem+), accent
  color, plain label below.
- If you can't fill at least 2 honest highlights, **omit the section.**

### Gallery (optional)
- 4–16 photos. Below 4, it's not a gallery — fold them into About or
  Services. Above 16, it needs lightbox + pagination.
- Treatments allowed:
  - **Tile grid** (current default): square crops, hover lift
  - **Masonry**: varying heights, more portfolio-feel
  - **Filmstrip**: horizontal scroll, full-bleed
  - **Lightbox**: tap-to-zoom with arrow navigation (always available as
    a layer on top of any of the above)
- Pick **one** treatment per site. Don't mix.

### Before & After (optional, vertical-specific) — see §4

### Contact
- Email link (`mailto:`), phone link (`tel:`), location text (city + region)
- Optional: simple form (`<form>` posting to `mailto:` or a real endpoint),
  WhatsApp CTA, Google Maps **link** (not embed)
- The CTA from Hero and the CTA in Contact should match in label and color.

### Footer
- Dark background (`--ink`), light text
- Logo (in a white pill, same treatment as nav but heavier shadow)
- © year + business name
- Optional: secondary links (Privacy, etc.), social icons, attribution
  ("Built by Ultra Flow" or similar — strip for client-owned sites)

---

## 4. Before & After gallery (new module)

A vertical-specific section for businesses that sell **transformation**:
home renovations, painting, cleaning, landscaping, auto detailing,
dental, fitness, hair, tattoo cover-ups, junk removal, pressure washing.

### Interaction model
- **Stacked images** — "before" and "after" are the same dimensions,
  same camera angle (within reason), same lighting if possible.
- **Drag handle** — a vertical handle sits over the image. Drag it left
  to reveal more of the "after", right to reveal more of the "before".
- **Clip-path on the after image** — the after layer is clipped to a
  rectangle whose right edge follows the handle position. Pure CSS, no
  canvas.
- **Touch-friendly** — `pointer` events, not `mouse` events. Handle must
  be large enough to grab on mobile (≥ 44 px target).
- **Keyboard-driven** — a hidden `<input type=range>` underneath gives
  arrow-key + screen-reader access. Tab-focusable.
- **Auto-animate on first scroll-into-view** — once per page load, slide
  from 100% before → 50/50 over ~800 ms easing. Pure dopamine. After
  that, the user controls it.
- **No libraries.** Vanilla JS, ~30 lines. No JQuery, no Cocoen, no
  TwentyTwenty.

### Data model
Each pair:
```json
{
  "id": "kitchen-renovation-east-st-paul",
  "before": "assets/before-after/kitchen-east-st-paul-before.jpeg",
  "after": "assets/before-after/kitchen-east-st-paul-after.jpeg",
  "label": "Kitchen renovation, East St. Paul",
  "durationDays": 21,
  "category": "kitchen"
}
```
- `before` and `after` paths are required.
- `label` is optional but recommended (a caption beneath the slider).
- `durationDays` is optional; when present, render as "21 days" badge.
- `category` is optional; used for filtering when there are > 4 pairs.

The data lives in `assets/before-after-manifest.json` as an array of
objects in display order. The page reads it on load via `fetch()` and
renders each pair.

### Layout
- 1 pair per row on mobile, 2 pairs per row ≥ 1024 px.
- Each pair: image (or slider), then label + optional duration badge below.
- If there are > 6 pairs, add category filter chips at the top of the
  section.

### Open design questions (parked)

1. **Default state on load** — 50/50 split, or 100% before then animate to
   50/50 when scrolled into view? (Leaning: 100% → 50/50, once per session.)
2. **Per-service tabs** — for a renovator with 4 service types, do we
   group pairs under Kitchens / Bathrooms / Basements / Other tabs, or
   just show them all flat? (Leaning: flat below 6 pairs, tabbed above.)
3. **Video before/after** — a 5-second drone flyover before → after could
   be powerful for exteriors. But video doubles the asset weight and
   complicates the data model. (Leaning: skip for v1, revisit when a
   client has actual exterior video.)
4. **Watermark / attribution on the photos** — small unobtrusive logo in
   one corner, or none? (Leaning: none. The site already says who built it.)
5. **"View original sizes" affordance** — should clicking a pair open a
   lightbox with both images full-size, side by side? (Leaning: yes,
   but v2.)

These get resolved as we build, then this doc gets updated.

---

## 5. Assets — folder convention

Logo lives at the **root** of `assets/`. Everything else is bucketed by
section so swaps in true-edit mode are local and reviewable. Decided
2026-05-23.

```
assets/
├── logo-*.jpeg / logo-*.png           ← THE logo (one, maybe two variants)
├── hero/                               ← hero background/slideshow
├── about/                              ← About-section visuals (owner photo, treated logo)
├── highlights/                         ← optional graphics for stat cards
├── gallery/                            ← standard portfolio
│   └── gallery-manifest.json
├── before-after/                       ← B/A pairs
│   └── before-after-manifest.json
└── icons/                              ← inline SVGs or custom icons (rare)
```

Rules:
- File names must be **descriptive**: `kitchen-east-st-paul-after.jpeg`,
  not `IMG_2049.jpeg`.
- Use lowercase, hyphens, no spaces, no underscores.
- Prefer `.jpeg` for photos (≤ 85% quality, mozjpeg if you have it),
  `.png` only for logos with transparency, `.svg` for icons.
- Strip EXIF data before committing.
- Never commit a photo larger than 2000 px on the long edge. Resize first.

Migration: this site started flat (all 10 files in `assets/`). The split
happens the next time we touch image references — likely when we add the
gallery manifest loader. Until then, flat is fine.

---

## 6. CSS conventions

- **One `<style>` block.** Top of `<head>`, no external sheet.
- **Custom properties at the top:**
  ```css
  :root {
    --accent: #2c5f8d;
    --ink: #1a1a1a;
    --paper: #ffffff;
    --muted: #6b7280;
    --radius: 6px;
    --container: 1200px;
  }
  ```
- **Sections use class names that match their semantic role** — `.hero`,
  `.services`, `.about`, `.highlights`, `.gallery`, `.before-after`,
  `.contact`. No utility classes (no Tailwind, no Bootstrap).
- **No `!important`** except for utility-class-like one-offs (`.hidden`).
- **Media query breakpoints:**
  - Mobile-first base styles
  - `@media (min-width: 640px)` — large mobile / small tablet
  - `@media (min-width: 1024px)` — desktop
  - Avoid mid-range breakpoints unless a specific section needs them.

---

## 7. JS conventions

- **One `<script>` block** at the bottom of `<body>`.
- **No frameworks.** Plain DOM API.
- **No globals.** Wrap everything in an IIFE or use `const` / `let`.
- **Initialize on `DOMContentLoaded`**, not on `load`.
- **Defer image-heavy work** (lightbox, before/after manifest fetch)
  until first user interaction or scroll-into-view via `IntersectionObserver`.
- **Never trust external input.** If a manifest JSON file is fetched, treat
  every field as untrusted (escape strings before inserting into HTML, or
  use `textContent` not `innerHTML`).

---

## 8. SEO + metadata baseline

Every site has, in `<head>`:
- `<title>` ≤ 60 chars, format: "Business Name | What They Do in Location"
- `<meta name="description">` ≤ 160 chars
- `<meta property="og:title">`, `og:description`, `og:image`, `og:url`
- `<meta name="viewport" content="width=device-width, initial-scale=1">`
- `<link rel="canonical">`
- A favicon (32 × 32 PNG at minimum)
- A `JSON-LD` `LocalBusiness` block with name, address, phone, hours
  (if known), URL, and image

`sitemap.xml` and `robots.txt` are generated when the site has a real
domain — not yet for Rivalin.

---

## 9. What this charter is NOT

- It's **not** a design system. There's no shared component library
  between Ultra Flow sites. Each site is a single HTML file with its own
  inline CSS, tuned to its vertical and brand.
- It's **not** a CMS spec. There's no DB, no admin UI required. Content
  is hand-edited (or admin-uploader-edited) directly in `index.html` and
  the manifest JSONs.
- It's **not** the builder's prompt. The builder generates *something
  close* to this — but it's the builder's job to follow this charter, not
  the other way around. If the builder ever diverges, this charter wins.

---

## 10. When to update this document

- When a new vertical-specific section gets added (after B/A: a menu
  section for restaurants, a booking section for salons, etc.)
- When a design rule changes (e.g. switching from 50px nav logo to
  60px globally)
- When an open design question (§4) gets resolved
- **Not** for one-site exceptions. If Rivalin needs something special,
  it lives in SPECS.md, not here.

Last reviewed: 2026-05-23.
