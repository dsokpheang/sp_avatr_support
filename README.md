# SP Avatr — support & releases

The public site for the **SP Avatr** software: the support and privacy pages the
iOS app links to, and the release folder every SP app updates itself from.

Served by GitHub Pages at **https://dsokpheang.github.io/sp_avatr_support/**.

> Nothing private lives here. No certificates, no keys, no keystores, no
> research notes — those stay in the private toolkit repo. The `.gitignore`
> carries guard rules for that reason.

## Layout

| Path | What it is |
|---|---|
| `index.html` | Landing page — what the software is, and where everything else is |
| `support/index.html` | Support page. **Support URL** for the App Store listing |
| `support/privacy.html` | Privacy policy. **Privacy Policy URL** for the App Store listing |
| `assets/site.css` | The whole site's styling. One file, no build step, no webfonts, no external requests |
| `assets/icon-*.png` | The app mark, downscaled from the iOS app icon (see below) |
| `sp_suites/` | The published release folder: every app's APK, `sp-broker.jar`, and `index.json` |
| `LICENSE` | The licence the software ships under. Copied from the toolkit repo — do not edit here |
| `.nojekyll` | Stops Pages from reinterpreting `sp_suites/`. **Load-bearing** — without it filenames with dots can 404 |

## The look

Deliberately plain: a single 660px text column, hairline rules instead of cards, and
one accent colour. There is no framework, no JavaScript, and nothing loaded from
another host — every page is three requests (HTML, CSS, one PNG).

The palette is **sampled from the app icon**: the deep navy of its ground and the
cyan of the wireframe glow. Dark is the product's look so it is the default; a light
palette is defined under `prefers-color-scheme: light` for readers whose OS asks for
one. Every colour is a token declared on bare `:root` first — none is only ever set
inside a media query, which is what stops one mode rendering unstyled.

The mark is the **iOS app icon**, the master at
`sp_avatr_toolkit/sp_avatr_ios/App/Assets.xcassets/AppIcon.appiconset/icon-1024.png`.
The sizes here are downscales of it, so the site, the phone home screen and the App
Store listing are all visibly the same product:

```bash
SRC=…/AppIcon.appiconset/icon-1024.png
for s in 180 96 32 16; do sips -s format png -Z $s "$SRC" --out "assets/icon-$s.png"; done
```

`icon-96` is the masthead mark (36 CSS px, so it stays sharp at 2×), `icon-180` is the
`apple-touch-icon` and the Open Graph image, `icon-32`/`icon-16` are the favicons.
**Regenerate all four when the app icon changes** — a stale mark here is the one place
the branding can silently drift.

## `sp_suites/` — the release folder

Three consumers read this folder, and none of them negotiates the path:

- **SP Service Starter** (desktop) — `SUPPORT_DIR` in `servicestarter/catalog.py`
- **SP Avatr Kit** (on-car, *Updates* panel) — the three base URLs in `Catalog.java`
- **SP Avatr** (iOS) — compares the car's broker against what is published here

`index.json` describes the whole suite: package name, `versionName`/`versionCode`,
size and **SHA-256** per artifact. Those hashes are the reason pulling over the
open internet is safe — a truncated or substituted download is refused before it
can reach a car. The folder holds exactly one build per app; superseded APKs are
removed on publish.

**Renaming this folder breaks every copy of every app already handed out.** There
is no fallback path. If it ever has to move, all three consumers change in the
same release, and every client in the field has to be updated by hand first.

## Publishing

Never edit `sp_suites/` by hand. It is written from a verified local build in the
private toolkit repo:

```bash
# from sp_avatr_toolkit/
./tools/build/build_all.sh                                            # build the suite
python3 tools/release/publish_support_site.py ~/development/sp_avatr_support --dry-run
python3 tools/release/publish_support_site.py ~/development/sp_avatr_support --push
```

`--push` is opt-in on purpose: this puts binaries on the public internet, which
is a decision, not a build step. The same command refreshes `LICENSE` from the
toolkit root, so the site and the shipped binaries can never quote different terms.

Pages usually serves a new commit within a minute. Clients cache-bust their
requests, but Fastly still honours Pages' `max-age=600`.

## Pages settings

Deploy from branch `main`, folder `/` (root). No build step, no Jekyll, no
Actions — these are hand-written static files.

---

© 2026 Sokpheang Duong. All rights reserved. See [LICENSE](LICENSE).
