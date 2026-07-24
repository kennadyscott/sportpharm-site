# SportPharm Website — Project Handoff

_Last updated: 2026-07-24 · Continuation guide for picking this project up in a fresh Claude session/account._

---

## 0. TL;DR — what to do first

1. **The GitHub repo is the single source of truth.** Everything built is pushed there. Nothing lives only on a local machine.
   - Repo: **`kennadyscott/sportpharm-site`** (branch `main`)
   - Live site (GitHub Pages): **https://kennadyscott.github.io/sportpharm-site/**
2. On the new account, in a fresh session, just say: _"Clone kennadyscott/sportpharm-site and read HANDOFF.md."_ That's enough to resume.
3. **Read this whole file** + the two knowledge docs in **`docs/`** (the detailed decision log) before editing.

---

## 1. What the project is

A custom marketing + content + light‑commerce website for **SportPharm** — a sports‑injury recovery brand (educational guidance + products that help athletes recover and return to play). President: **Brandon Welch**. It is **problem/solution first**, not product‑first (browse by injury / body part / sport).

Two related surfaces:
- **SportPharm main site** — the big content site (homepage, injury guides, recovery/performance hubs, product pages, sports‑medicine/professional page, articles, an admin console).
- **WasabiRub** — the flagship topical‑rub product's own landing page (`wasabirub.html`). The lineup: **WasabiRub** ($29.95, balanced), **IcetraRub** ($39.95, cooling), **Super Hot** ($39.95, max heat), **LidoRub** (coming soon).

**Eventual production stack (not built yet):** Next.js + Payload CMS + Postgres on Vercel, Stripe for payments (incl. native promo codes), Mailchimp for email/abandoned cart. **No WooCommerce needed** (that's WordPress‑only). The current site is a **high‑fidelity static HTML prototype** that will port into that stack.

---

## 2. How it's built & deployed

- **Static HTML/CSS/JS.** Each page is self‑contained with an inline `<style>` block — there is **no shared external stylesheet**. (This is why editing one page never affects another.)
- **No framework, no build step.** Plain files served as‑is by GitHub Pages.
- **Deploy = git push.** Push to `main` → GitHub Pages rebuilds automatically (~30–60s). CDN caches per file ~60s, so hard‑refresh (Cmd+Shift+R) or incognito when verifying.
- **Preview locally** with a static server on **port 4184** (see §5).

### Pushing to GitHub (auth)
There is **no `gh` CLI**. Auth uses a token stored in the **macOS keychain** (service `gh:github.com`), decoded and passed as a Basic auth header. On the same Mac this keeps working; on a new machine you'll need to re‑add the token to the keychain (or use your own git credentials).

```bash
cd /path/to/sportpharm-site
RAW=$(security find-generic-password -s "gh:github.com" -w)
TOKEN=$(printf '%s' "${RAW#go-keyring-base64:}" | base64 -d)
AUTH=$(printf 'x-access-token:%s' "$TOKEN" | base64)
git add -A && git commit -m "…"
git -c http.extraheader="Authorization: Basic $AUTH" push origin HEAD
```
> Note: the account `kennady-scott` (with a hyphen) is separate; this token only pushes to **`kennadyscott`**.

---

## 3. Current state — what's live

**43 HTML pages.** Highlights:

| Area | Pages | Notes |
|---|---|---|
| **Main homepage** | `index.html` | Everyday‑Athlete homepage. Persona toggle top‑right (Everyday / Pro / Healthcare‑AT → `index` / `pro-athlete` / `sports-medicine`). "What Brings You Here ▾" → pathway anchors. Fonts: **Bebas Neue + Inter**. Brand red **#E0312A**. |
| **Persona homepages** | `pro-athlete.html`, `sports-medicine.html` | Pro Athlete + Healthcare/Athletic‑Trainer landing pages. Each has the persona toggle. `sports-medicine.html` is a **LIGHT** theme (white/navy/red). |
| **Athlete Hub (old main build)** | `athlete-hub.html` | The original content home, now a sub‑hub. |
| **Injury system** | `injuries.html` + 8 guides (`ankle-sprain, acl, hamstring, lower-back, rotator-cuff, tennis-elbow, shin-splints, concussion`) + 8 body‑part pages (`head-neck, shoulder, elbow, wrist-hand, back-spine, hip-thigh, knee, lower-leg-ankle-foot`) | Guides are **DARK** hero cards; regenerate from the generator, don't hand‑edit. |
| **Recovery / Performance hubs** | `recovery.html`, `performance.html` + landing pages (`recovery-timelines, rehab-exercises, return-to-play, recovery-tools, training, injury-prevention, strength-mobility, warmup`) | DARK theme, each with one interactive widget. |
| **Products** | `products.html`, `wasabirub.html`, `icetrarub.html`, `superhot.html` | LIGHT theme retail. `wasabirub.html` = the standalone **FEEL IT WORK** landing (see §6). |
| **Articles** | `articles.html`, `article-recovery-habits.html`, `article-nsaids.html`, `article-return-injury.html` | Blog. |
| **Admin console** | `admin.html` | **Prototype** internal dashboard (localStorage‑backed). Modules: Dashboard, Blog CMS (full CRUD w/ tags/categories/publish flow), Products, Leads, Analytics, Settings. On‑brand navy/red, Bebas+Inter. Demo login (any credentials). This is the blueprint for the eventual Payload build. |
| **Utility** | `coming-soon.html` (`?f=Feature+Name`), `roadmap.html` | Every unbuilt destination points to `coming-soon`. |

---

## 4. Brand & design system

- **Logos** (in `assets/`): `wordmark-dark.png` (red SPORTPHARM wordmark, for light bgs), `wordmark-white.png` (for dark bgs), `logo.png` / `logo-white.png` (full logo w/ "We Take Our Drugs Seriously." tagline).
- **Brand red = `#E0312A`** across the main site. (WasabiRub's own page uses its own red **`#d6202a`** — keep that distinction.)
- **Fonts by surface:**
  - Main site (homepage/persona pages): **Bebas Neue** (display) + **Inter** (body/nav).
  - Injury guides / hubs / older pages: **Oswald** (display) + **Epilogue** (nav/sub‑headings) + Inter body. _(User is picky about letterforms — chose Epilogue over Sora/Archivo because of the lowercase "l"; see `docs/`.)_
  - WasabiRub page: **native Arial/Helvetica weight:1000** display (user rejected Bebas AND Archivo Black here).
- **Theme split (intentional):** browse/landing pages (homepage, injuries hub, recovery/performance hubs) are **DARK**; detail/content pages (injury guides, product pages, sports‑medicine) are **LIGHT**.
- User prefers **white/bright text over grey**; do not reintroduce Comic Sans‑style casual fonts (serious brand).

---

## 5. Local preview (recreate on new machine)

`/tmp` gets wiped between sessions, so the working copy and `launch.json` are **ephemeral** — recreate them by cloning + adding a launch config. The user's Claude Code uses a `launch.json` preview server named **`sportpharm`** on **port 4184** serving the repo dir. Recreate `.claude/launch.json`:

```json
{
  "version": "0.0.1",
  "configurations": [
    { "name": "sportpharm", "runtimeExecutable": "python3",
      "runtimeArgs": ["-m", "http.server", "4184"], "port": 4184 }
  ]
}
```
Serve from the repo directory. (Important: `~/Documents` is **not** readable by the preview server, so images must be copied into the repo's `assets/` — see §7.)

---

## 6. ⚠️ WasabiRub — do not redesign

`wasabirub.html` is the user's **approved "FEEL IT WORK" single‑page mockup**. A multi‑page rebuild + color rebrand was attempted and **rejected** ("this looks awful"). Current, correct state:
- The original FEEL IT WORK layout, restored verbatim.
- Header logo = **SportPharm wordmark + "Perform. Recover. Return."** tagline (red periods).
- Product images = local transparent PNGs `assets/wr-{wasabirub,icetrarub,superhot,duo}.png` (white backgrounds knocked out).
- Palette: `--red:#d6202a`, `--deep:#101820` hero, tier accents blue/green/orange, red announcement bar + red final CTA, green glow behind the hero jar.

**Rule:** only touch this page when asked, and keep it anchored to this mockup. A multi‑page wasabirub.com is on the roadmap but must be built FROM this exact look, not a new design system.

---

## 7. Working conventions & gotchas

- **Generators live in the scratchpad, not the repo.** Multi‑page families (injury guides, body‑part pages, hub landings, articles) were produced by Python generators that emit HTML from data dicts. The scratchpad is ephemeral — **the committed HTML in the repo is the source of truth.** If you need to bulk‑edit a family, re‑derive a generator from an existing page rather than hand‑editing 8+ files.
- **Images:** user drops source images in **`~/Documents/Claude/sportpharm-assets/`** (NOT readable by the preview server). Copy/process them into the repo's **`assets/`** (compress to KB — progressive JPEG for photos, keep PNG for transparency; PIL is available). Product PNGs from `i0.wp.com` have **white backgrounds** — knock them out with `PIL.ImageDraw.floodfill` from the edges if they need to float on dark.
- **Preview screenshot quirk:** scroll‑then‑screenshot can return blank frames. Verify with DOM checks (`javascript_exec`, broken‑image scans) or a tall viewport at scrollY 0.
- **/tmp wipe recovery:** if the working dir is gone, `git clone` the repo fresh, re‑add `.claude/launch.json`, and continue.
- **Every link should resolve** — unbuilt destinations point to `coming-soon.html?f=…`. Keep it that way (no dead anchors).

---

## 8. Roadmap / open items

From the Director of Marketing's priority list + pending work:
1. **WasabiRub → true multi‑page site** (Home, Find Your Rub w/ dropdown, Athlete Hub, How It Works, Professional, Cart) — **build FROM the approved FEEL IT WORK look**, only when asked.
2. **Integrations (confirm, then build):** Stripe (payments + coupon/promo codes), Mailchimp (email capture + abandoned cart). Prototype UI only exists so far. No WooCommerce.
3. **sportpharm.com redesign notes:** make it more functional for the everyday person / brick‑and‑mortar in mind; add **Store** to the menu bar linking to wasabirub.com; move testimonials up; improve ease of use.
4. **Admin console → real backend:** port the `admin.html` prototype's content model to Payload CMS.
5. Wire the site's Contact / "Contact a Rep" forms to feed the admin **Leads** module.
6. Build the 4 EA‑homepage pathway landing pages (currently on‑page anchors).

---

## 9. What to bring to the new account

- **Nothing is required beyond the GitHub repo** — clone `kennadyscott/sportpharm-site` and you have the whole site + this guide + `docs/`.
- **Keep (optional, on your Mac):** `~/Documents/Claude/sportpharm-assets/` — original source images you drop for processing (higher‑res than what's in the repo; includes `everyday-athlete-homepage/`, `lifestyle imagery/`, `new-hero.png`, isolated team photos, product source PNGs). Useful for future edits; not needed to run the site.
- **Auth:** the GitHub push token is in this Mac's keychain (`gh:github.com`). Same machine → still works. New machine → re‑add it or use your own git credentials.

See **`docs/`** for the full decision log (`sportpharm-website-project.md`) and the live‑site inventory (`sportpharm-current-site-inventory.md`).
