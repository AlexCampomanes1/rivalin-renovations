# Rivalin Renovations

Standalone, hand-edited static site for Rivalin Renovations (Winnipeg). Detached from the Ultra Flow builder on 2026-05-23.

## Run it

```bash
# Just open
open index.html

# Or serve (better for relative-path debugging)
python3 -m http.server 8765
# → http://localhost:8765/
```

No build step. No dependencies. Single-file HTML + `assets/`.

## Read this first

- [SPECS.md](SPECS.md) — what the site is, who it's for, the visual system, what's in it
- [HANDOFF.md](HANDOFF.md) — why it exists, what was changed vs. the builder output, what's next, what *not* to do

## Files

```
index.html               ← the site
assets/                   ← all images (logo + 9 stock photos), fully local
project.json              ← original intake answers (read-only context)
SPECS.md, HANDOFF.md      ← docs
```

## Next up

Before & after gallery → richer standard gallery → tiny local admin uploader. Once proven here, port back into the Ultra Flow builder as an opt-in module. Details in [SPECS.md §Roadmap](SPECS.md#roadmap-post-detachment).
