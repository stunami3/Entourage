# Entourage

A single-file web app for cannabis users who read their COAs. Tracks live rosin jars, cartridges, edibles, and flower with full terpene profiles, star ratings, and an archive — built around the entourage effect, not just THC percentages.

Copyright © 2026 stunami3. All Rights Reserved. Personal-use software; do not redistribute.

---

## What it is

- **Two generated HTML files, one source of truth.** `entourage.html` holds every feature and the real inventory data — it's what gets pasted into the Claude artifact. `index.html` is generated from it automatically by `build.py`, with the inventory stripped to empty, for self-hosting. See **Build workflow** below.
- **Phone-first.** Designed for iPhone: sticky toolbar, bottom-sheet detail view, 44px tap targets, safe-area insets, reduced-motion support.
- **Private by design.** The self-hosted file never carries inventory data — it starts empty and you Import your own backup. See Storage below.

## Features

- **Inventory sections** in order: Jars → Carts → Flower → Edibles, with collapse/expand all.
- **Home / On-the-go modes.** On-the-go filters to portable products — carts, distillates, and edibles; dabs and flower are home-only.
- **Goal chips** for quick filtering by intent.
- **Two independent sliders:**
  - **Lineage** (Sativa ↔ Indica) — the dispensary's own label, set via a dropdown on each item (Sativa / Sativa-leaning / Hybrid / Indica-leaning / Indica).
  - **Effect** (Energizing ↔ Sedating) — estimated from the item's terpene profile, independent of the label. Surfaces the real paradox strains — a "sativa" that reads sedating on the terp sheet, and vice versa.

    The estimate is a concentration-weighted score across 23 terpenes, using **how each terpene is commonly reported to feel in the cannabis community and industry** — not lab-measured, and not clinically established. Human trial evidence for terpene-driven effects is genuinely thin; the strongest result to date (a 2024 double-blind RCT) covers d-limonene reducing THC-induced anxiety, which is not the same as an energizing/sedating axis. Terpenes with strong, near-universal consensus (myrcene, linalool, limonene, pinene, terpinolene) carry real weight; terpenes with thin or contradictory reports (camphene, farnesene, 3-carene) are deliberately weighted at or near zero so they don't fabricate a signal. Each item shows which terpenes drove its score, and flags "low confidence" when there isn't enough well-understood terpene mass to say much.
- **Terpene bars** weighted against realistic live-rosin ceilings per terpene, so a strong Limonene reads as strong even though no terpene hits double digits.
- **Detail sheet** with the full COA in priority order: THC → CBD → Total Terps → Myrcene → Limonene → Caryophyllene → Linalool, then the rest, plus the computed Effect %. Cannabinoids (CBG/CBGA) listed separately so they never displace terpenes. Tap any terpene name for a plain-language tooltip (~23 terpenes covered).
- **Ratings.** 1–5 stars per strain; unrated shows "Strain Not Rated."
- **Abbreviation tag** — optional 6-character shorthand shown next to the strain name (e.g. SKNK, TDAZ).
- **Lifecycle.** Finish (archives with date), Delete, Restore. Archive is browsable via toggle.
- **Add products** via the + button:
  - **Scan COA** — upload a COA PDF or photo; Claude reads it and returns structured data (name, brand, type, size, THC, CBD, total terps, full terpene list, other cannabinoids, batch date, lab) for your review before anything is added. *Only works when the app runs inside Claude (published artifact or chat preview) — see Hosting below.*
  - **Manual entry** — works everywhere. All 23 terpenes COAs commonly report are enterable (the 7 majors up front, the rest behind a "More terpenes" toggle), plus lineage, abbreviation, and every other field. The Effect slider position calculates live as you type.
- **Export / Import backup** — full inventory as a JSON file, for safekeeping or moving between installs.

## How index.html is generated

`index.html`, the file in this repo, is not hand-written. It's generated from `entourage.html` — the single source of truth for both the app's code and the real inventory data — by stripping the two seed arrays (`SEED_ACTIVE`, `SEED_ARCHIVE`) down to empty. That's the only difference between the two files; everything else is identical.

That generation step (`build.py`) lives outside this repository, alongside `entourage.html`, and is never uploaded here — so don't expect to find it in this file listing. It's mentioned for context: if you're wondering why `index.html` has no data and no unminified "source," this is why, and it's intentional (see Storage below for the privacy reasoning). Every feature update to this app happens in `entourage.html` first; `index.html` is regenerated and re-uploaded after.

## Hosting & install

Two ways to run it. They have different tradeoffs:

| | Published on Claude | Self-hosted (GitHub Pages etc.) |
|---|---|---|
| Opens as standalone app from Home Screen | No — wrapped in Claude site chrome | **Yes** |
| COA scanning (AI) | **Yes** | No — manual entry only |
| Storage | Claude account (syncs across your devices) or device, depending on where it runs | This device only (localStorage) |
| URL | claude.ai public link | Your own clean URL |

### GitHub Pages setup (iPhone-friendly)

1. Get `index.html` — this is generated by `build.py` from `entourage.html` (see Build workflow above); it's already named correctly and has no personal data baked in.
2. Create a free account at github.com.
3. New repository → name it `entourage` → **Public** → Create.
4. Add file → Upload files → pick `index.html` → Commit. (Use Safari's ᴬA → Request Desktop Website if menus are missing.)
5. Settings → Pages → Deploy from a branch → `main` / root → Save.
6. After a minute or two, open `https://YOURUSERNAME.github.io/entourage/`.
7. Share → **Add to Home Screen**. Launch from the icon: full-screen, no browser chrome.
8. Do your backup **Import inside the home-screen app** (see Storage below for why).

**Updating:** upload a new `index.html` to the repo — same name replaces the old one. Your data survives updates (it's stored on the device, keyed to the domain, not in the file). If the app icon artwork ever changes, delete and re-add the Home Screen shortcut; iOS caches icons.

## Storage — how it actually works

At launch the app probes storage backends in priority order and uses the first one that passes a live write-read-delete self-test:

1. **Claude account storage** — only exists inside Claude's artifact runtime. Syncs with your Claude account.
2. **localStorage** — the normal case in Safari and home-screen installs.
3. **IndexedDB** — fallback if localStorage is blocked.
4. **Memory only** — last resort (e.g., fully locked-down webviews). Nothing persists; a red banner warns you to use Export backup.

The status line under the title always tells you which mode is live: "Synced to your Claude account," "Saved on this device," or the red warning.

Mechanics: state loads once at launch from two keys (`entourage:active`, `entourage:archive`) and lives in memory. Every change — rate, finish, delete, restore, add — immediately writes the full JSON back. Synchronous, no batching; the last completed action is always saved. Inventory JSON is tens of KB against a ~5MB localStorage quota.

**iOS specifics worth knowing:**

- The Home Screen app and the Safari tab of the same URL have **separate** storage containers. Data added in one is invisible in the other. Pick one (the home-screen app) and do everything there.
- Safari purges storage for sites unvisited for 7 days under Intelligent Tracking Prevention — **home-screen web apps are exempt.** Another reason to install it.
- Settings → Safari → Clear History and Website Data erases localStorage for all sites, including this one. Export backups are your protection.
- Different domains = different storage. Moving from the Claude-published version to self-hosted (or between hosts) requires Export on the old, Import on the new.

## Backup & restore

- **Export backup** downloads a timestamped JSON file containing active + archive. Save it to Files/iCloud.
- **Import backup** restores from that file, replacing current data after confirmation.
- Make an export after any big session of edits. It is the only copy that exists outside your device.

### Moving data between installs

Every unique combination of browser + launch method is its own storage silo: a Safari tab, a Safari home-screen icon, Chrome, a Chrome home-screen icon, another device — each is blind to the others. Nothing syncs automatically between them. The one partial exception is the Claude account storage backend, which is meant to sync by account rather than by device — but on the iOS Claude in-app viewer specifically, that backend fails its self-test and falls back to device storage, so don't assume it's syncing. Check the status line under the title in each install ("Synced to your Claude account" vs "Saved on this device") to know what you're actually dealing with there.

**Practical approach:** pick one install as your source of truth (the GitHub Pages home-screen app, for daily use) and treat every other install as a satellite you sync *into* it, right after each session on that satellite — don't let two installs both accumulate changes between syncs.

**To move data from install A to install B:**
1. Open install A **from its own Home Screen icon**, not a Safari tab (see iOS specifics above — tab and icon have separate storage even for the same URL).
2. **Export backup** → save the JSON.
3. Open install B, also from its Home Screen icon.
4. **Import backup** → pick that JSON file.

Import always **replaces** current data after a confirmation prompt. Export from whichever install has the changes you want to keep *before* importing into another, or those changes are lost. Typical flow: scan a COA on the Claude-published version → Export → Import into the GitHub Pages version you actually use day to day.

## Debug mode

The console is silent in normal use. Append `?debug=1` to the URL to enable full diagnostic logging: storage backend probing, environment report, migration attempts, COA parse errors.

## Tech notes

- Vanilla JS, no frameworks. Fonts: Bricolage Grotesque, Inter, JetBrains Mono.
- Terpene name normalization maps lab COA formats (`beta-Myrcene`, `d-Limonene`, `trans-Caryophyllene`, etc.) to canonical names.
- All user-entered and AI-parsed text is HTML-escaped before rendering.
- The seven-leaf logo: one vesica leaf per major terpene (Myrcene, Limonene, Caryophyllene, Linalool, Pinene, Humulene, Terpinolene) in an interlocking weave — the entourage effect, drawn. The amber Myrcene leaf points straight up.

  Seven is an odd number, so a strictly alternating over/under weave around a ring is geometrically impossible — one crossing always ends up frustrated. The mark resolves this by reversing the stacking order inside a central disc: each leaf passes **over** its neighbours out at the petals and **under** them at the core. Every leaf therefore goes both over and under, with no leaf permanently on top or buried, and no broken seam. Each leaf carries a background-coloured halo beneath its outline, which interrupts the strand passing below and makes the crossings read as woven rather than merely overlapped.

  `entourage-logo.svg` is the vector master; the PNGs and the three copies embedded in the app (header, favicon, apple-touch-icon) are rendered from it. The generator solves for the leaf-tip angle rather than hard-coding the rotation, so Myrcene stays vertical if the geometry is ever retuned.
