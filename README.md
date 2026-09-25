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
- **Tonight’s pick.** Answer six quick questions — worked out today, work or off tomorrow, TV / gaming / social, time until bed, format, and whether you want something new — and the app ranks what’s in stock, top three, each with a one-line reason drawn from the actual COA numbers (e.g. “Most Linalool in stock (0.409), for calm”). Defaults are set from the clock and day of the week, so on a typical night it’s one tap. Terpene weights shift with context: sleep soon favours Myrcene and Linalool and penalises Limonene; gaming penalises the calming terpenes that cost reflexes; a workout adds Caryophyllene. Products without lab terpene data (mostly edibles) aren’t ranked, and the result says how many were left out.
- **Two independent sliders:**
  - **Lineage** (Sativa ↔ Indica) — the dispensary's own label, set via a dropdown on each item (Sativa / Sativa-leaning / Hybrid / Indica-leaning / Indica).
  - **Effect** (Energizing ↔ Sedating) — estimated from the item's terpene profile, independent of the label. Surfaces the real paradox strains — a "sativa" that reads sedating on the terp sheet, and vice versa.

    The estimate is a concentration-weighted score across 23 terpenes, using **how each terpene is commonly reported to feel in the cannabis community and industry** — not lab-measured, and not clinically established. Human trial evidence for terpene-driven effects is genuinely thin; the strongest result to date (a 2024 double-blind RCT) covers d-limonene reducing THC-induced anxiety, which is not the same as an energizing/sedating axis. Terpenes with strong, near-universal consensus (myrcene, linalool, limonene, pinene, terpinolene) carry real weight; terpenes with thin or contradictory reports (camphene, farnesene, 3-carene) are deliberately weighted at or near zero so they don't fabricate a signal. Each item shows which terpenes drove its score, and flags "low confidence" when there isn't enough well-understood terpene mass to say much.
- **Terpene bars** weighted against realistic live-rosin ceilings per terpene, so a strong Limonene reads as strong even though no terpene hits double digits.
- **Detail sheet** with the full COA in priority order: THC → CBD → Total Terps → Myrcene → Limonene → Caryophyllene → Linalool, then the rest, plus the computed Effect %. Cannabinoids (CBG/CBGA) listed separately so they never displace terpenes. Tap any terpene name for a plain-language tooltip (~23 terpenes covered).
- **Ratings.** 1–5 stars per strain; unrated shows "Strain Not Rated."
- **Session log.** “Had tonight” on any card or in the detail sheet records the night, and a session after midnight counts toward the night before (until 5am). Cards show when you last had something; the detail sheet shows the full history and, for jars, a rough usage estimate at about 0.1 g a night, which says plainly that anything used before you started logging isn’t counted. “Had it before” covers products you’ve used but never logged. Nothing is ever labelled untried — only “not in your log yet,” because missing data isn’t evidence you haven’t had it.
- **Abbreviation tag** — optional 6-character shorthand shown next to the strain name (e.g. SKNK, TDAZ).
- **Lifecycle.** Finish opens a short “how was it?” step — optional stars and a one-line note — then archives with the date. The archive keeps a full snapshot of the product, including its terpene profile and session log, so Restore brings it back exactly as it was. (Entries archived before snapshots existed kept only name, THC, and effects; restoring those still needs details filled in.) Archive is browsable via toggle.
- **Add products** via the + button:
  - **Scan COA** — upload a COA PDF or photo; Claude reads it and returns structured data (name, brand, type, size, THC, CBD, total terps, full terpene list, other cannabinoids, batch date, lab) for your review before anything is added. *Only works when the app runs inside Claude (published artifact or chat preview) — see Hosting below.*
  - **Paste COA** — on the self-hosted app, run the **Entourage COA** shortcut (Apple Intelligence) on a COA — share the COA to it, or tap **Open Entourage COA** in the Scan COA sheet and pick the photo or file — then tap Paste COA in the Scan COA sheet. The app reads the JSON the shortcut left on the clipboard and shows the same review card as Scan COA. Clipboard contents without a strain name or a terpenes list are rejected with an error; nothing partial is added. See **Apple Intelligence via Shortcuts** below.
  - **Manual entry** — works everywhere. All 23 terpenes COAs commonly report are enterable (the 7 majors up front, the rest behind a "More terpenes" toggle), plus lineage, abbreviation, and every other field. The Effect slider position calculates live as you type.
- **Export / Import JSON** — full inventory, for safekeeping or moving between installs. Export saves a file or copies to the clipboard; Import takes a file or pasted text.
- **Copy for Claude.** A compact plain-text summary of what’s actually in stock right now — COA numbers, effect scores, ratings, session history, and recent finishes with their notes — for pasting into a chat when you want a second opinion. From inside Tonight’s pick it also includes your answers to the six questions. This exists because the app’s live data lives on your device, so anything reasoning from an older copy of the inventory will recommend products you’ve already finished.
- **Ask.** Type a question; the app copies the same in-stock summary as Copy for Claude plus your question to the clipboard and opens the **Entourage Ask** shortcut, which answers with Apple Intelligence. See **Apple Intelligence via Shortcuts** below.

## How index.html is generated

`index.html`, the file in this repo, is not hand-written. It's generated from `entourage.html` — the single source of truth for both the app's code and the real inventory data — by stripping the two seed arrays (`SEED_ACTIVE`, `SEED_ARCHIVE`) down to empty. That's the only difference between the two files; everything else is identical.

That generation step (`build.py`) lives outside this repository, alongside `entourage.html`, and is never uploaded here — so don't expect to find it in this file listing. It's mentioned for context: if you're wondering why `index.html` has no data and no unminified "source," this is why, and it's intentional (see Storage below for the privacy reasoning). Every feature update to this app happens in `entourage.html` first; `index.html` is regenerated and re-uploaded after.

## Hosting & install

Two ways to run it. They have different tradeoffs:

| | Published on Claude | Self-hosted (GitHub Pages etc.) |
|---|---|---|
| Opens as standalone app from Home Screen | No — wrapped in Claude site chrome | **Yes** |
| COA scanning (AI) | **Yes** | Via the Entourage COA shortcut + Paste COA (Apple Intelligence iPhone) |
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

## Apple Intelligence via Shortcuts

Two shortcuts give the self-hosted app AI features without any API key: **Entourage COA** turns a COA into the JSON that Paste COA reads, and **Entourage Ask** answers questions about what's in stock. Both run Apple's model on Private Cloud Compute.

**Requirements:** iOS 26 or later on an iPhone that supports Apple Intelligence, with Apple Intelligence turned on (Settings → Apple Intelligence & Siri). Action names below are as Apple documents them; if a label on your phone reads slightly differently, search the action list for the key word (e.g. "Model", "Extract Text").

The shortcut names must match exactly — the app opens `shortcuts://run-shortcut?name=Entourage%20COA` and `shortcuts://run-shortcut?name=Entourage%20Ask&input=clipboard`, so shortcuts named anything other than `Entourage COA` and `Entourage Ask` won't be found.

### Entourage COA

The shortcut works two ways: with a COA already attached (from the Share Sheet), or with nothing attached (from the app's **Open Entourage COA** button, the Shortcuts app, or Siri), in which case it asks you to pick a photo or a file first.

1. Open **Shortcuts** → **+** (new shortcut). Tap the name at the top → **Rename** → `Entourage COA`.
2. Tap the **ⓘ** (Details) button → turn on **Show in Share Sheet** → Done. A **Receive** block appears at the top. Tap its input types and leave only **Images** and **PDFs** selected. Set "If there's no input" to **Continue**.
3. Add **If**. Condition: **Shortcut Input** **does not have any value**.
4. Inside that **If** branch, add **Choose from Menu**. Set its prompt to `Scan a COA from` and make two options: `Photo` and `File`.
   - Under **Photo**, add **Select Photos**, then **Set Variable** with the name `COA` and the selected photo as its input.
   - Under **File**, add **Select File**, then **Set Variable** with the name `COA` and the selected file as its input.
5. Inside the **Otherwise** branch of the step 3 **If**, add **Set Variable** with the name `COA` and **Shortcut Input** as its input.
6. After that **End If**, add **Get Details of Files**. Set it to get **File Extension** of the **COA** variable.
7. Add **If**. Condition: **File Extension** **is** `pdf`.
8. Inside the **If** branch, add **Get Text from Input** with the **COA** variable as its input.
9. Inside the **Otherwise** branch, add **Extract Text from Image** with the **COA** variable as its input.
10. After **End If**, add a **Text** action. Paste the prompt below into it, then on a new line after it type `COA text:` and insert the **If Result** variable from the step 7 **If** after that. (There are two **If** blocks, so check you picked the one after the pdf check.)
11. Add **Use Model**. Choose **Private Cloud Compute** as the model. Set its request to the **Text** from step 10. (If the action offers a Follow Up option, leave it off.)
12. Add **Copy to Clipboard**. Its input should be the output of Use Model (Shortcuts usually fills this in; if not, tap the input and pick the Use Model variable).
13. Optional: add **Show Notification** with `COA copied — open Entourage and tap Paste COA`.

Three ways to start it:
- **From the Share Sheet:** open the COA (PDF in Files/Mail, or a photo) → Share → **Entourage COA**. It uses that file directly.
- **From the Shortcuts app, a Home Screen shortcut icon, or Siri:** run **Entourage COA**; it asks Photo or File, then lets you pick the COA.
- **From inside Entourage:** **+** → **Scan COA** → **Open Entourage COA**. The app opens the shortcut with nothing attached, so it asks Photo or File as above.

Whichever way you start it, when the notification appears, go back to Entourage **from its Home Screen icon** → **+** → **Scan COA** → **Paste COA**, and review the card before adding. iOS will ask to allow pasting; tap **Allow Paste**.

A scanned PDF with no text layer produces no text at step 8. If that happens, screenshot the COA page and use the screenshot instead (Share it, or pick **Photo** from the menu), so step 9 reads it.

The prompt to paste in step 7 (identical to the one the app's built-in Scan COA uses):

```
You are extracting data from a cannabis Certificate of Analysis (COA). Read the document and output ONLY a single JSON object — no markdown fences, no explanation, nothing else. Schema:
{
 "name": strain/cultivar name as shown,
 "brand": brand or product line (e.g. MPX, The Vault, Sunshine State, Curaleaf, GrowHealthy),
 "productType": one of "cart","jar","flower","edible","distillate" (infer from product description — "Live Rosin"/"Rosin"/"Derivative" in a small jar format = jar, in a cartridge = cart, "Whole Flower" = flower, gummy/chocolate/RSO edible = edible),
 "size": e.g. "0.5g", "1g", "3.5g",
 "thc": Total THC percent as shown, e.g. "72.8%",
 "cbd": Total CBD percent as shown, e.g. "0.147%" (or null if not tested/shown),
 "totalTerps": Total Terpenes percent as shown, e.g. "6.79%" (or null if not tested),
 "terpenes": array of {"name":.., "pct":number} for every individual terpene with a positive percent value. Use these exact names when the terpene matches (strip d-/l-/beta- prefixes only for these four): Myrcene, Limonene, Caryophyllene, Linalool. For all other terpenes keep standard naming e.g. Humulene, Farnesene, Guaiol, Bisabolol, Ocimene, trans-Nerolidol, cis-Nerolidol, alpha-Pinene, beta-Pinene, alpha-Phellandrene, alpha-Terpineol, Fenchyl Alcohol, Camphene, Caryophyllene Oxide, alpha-Terpinolene, Borneol, Fenchone, alpha-Cedrene, 3-Carene.
 "otherCannabinoids": short string summarizing notable minor cannabinoids e.g. "CBGa 5.04%, THCV 0.574%" (or empty string),
 "batchDate": batch date as MM/DD/YY if shown, else null,
 "lab": testing lab name if shown,
 "cultivator": cultivation facility if shown
}
If a field is not present in the document, use null (or empty array for terpenes). Output nothing except the JSON object.
```

Paste COA tolerates the model wrapping its answer in ```` ```json ```` fences, lab-style terpene names (`beta-Myrcene`, `d-Limonene`, …, normalized the same way as Scan COA), and percents given as text (`"0.8%"`). It rejects anything without a strain name or a terpenes list, or with a terpene entry missing a name or a numeric percent. Always check the review card against the COA: an AI read of a lab sheet can be wrong.

### Entourage Ask

1. Open **Shortcuts** → **+**. Rename it to `Entourage Ask` (exact spelling and capitalization).
2. Add **Get Clipboard**.
3. Add **Use Model**. Choose **Private Cloud Compute**. Set its request to the **Clipboard** from step 2.
4. Add **Show Result** with the output of Use Model as its input.

To use it: in Entourage tap **Ask**, type your question in the sheet that opens, and tap **Ask** (or Send on the keyboard). The app copies the in-stock summary plus your question to the clipboard and opens the shortcut, which shows the answer. If the app can't write to the clipboard, it shows the text in a panel instead so you can copy it and run the shortcut yourself.

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

  The logo is generated from a parametric script kept with the project source (alongside `entourage.html`, outside this repo); `entourage-logo.svg` is the vector master it emits, and the PNGs plus the three copies embedded in the app — header, favicon, apple-touch-icon — are all rendered from that. The generator solves for the leaf-tip angle rather than hard-coding the rotation, so Myrcene stays vertical if the geometry is ever retuned.

  Note for iOS: the Home Screen caches app icons aggressively. After an artwork change, delete the shortcut and re-add it — reloading the page won't update it.
