# Pocket Caddie ⛳

A personal golf GPS, scorecard, and shot tracker that runs entirely in the browser. One HTML file, no server, no accounts, no subscriptions. All data stays on your phone.

**Live app:** open the GitHub Pages URL for this repo in Safari, then Share → **Add to Home Screen** to install it like an app.

Current version: **v1.0** (shown in the app header — use it to check whether a cached copy is stale).

---

## Features

- **Live GPS yardages** — tap anywhere on the satellite map to drop a flag; the yardage plate shows the live distance from you to it as you walk.
- **Shot tracking** — mark your position before each swing. First mark per hole is labeled the tee; every segment between points shows its yardage on the map. Tag each shot with a club, miss tags (chunked, skulled, thin, fat, push, pull, slice, hook), and a free-text note — all optional.
- **Drops & relief** — penalty drops with type (point of entry, drop area/other side, or redo/stroke-and-distance e.g. re-teeing after OB), selectable penalty strokes (0/1/2, default 1), and optional reason; the shot that caused the drop is flagged in the data. Free relief (GUR / cart path / casual water / embedded / casual obstacle) marks with no stroke.
- **Scorecard** — 18 holes, editable pars, strokes, putts (putts feed the stroke total), Tee / FW / GIR chips, running totals and +/− vs par.
- **Putting notes** — tap the putts number: first-putt length, per-putt miss direction + note, read, and green condition chips.
- **Scramble mode** — team format with player names and whose-ball tagging.
- **Round history** — saved rounds with map replay of the full shot trail.
- **Exports** — per-round formatted scorecards (with machine-readable data embedded) and a full JSON export of everything.
- **Quality of life** — sun mode for bright light, customizable club list, auto-save with crash-resume, first-visit tour, in-app Help tab.

## Setup (GitHub Pages)

1. Put `index.html` at the root of this repo (the name matters — Pages serves `index.html` at the base URL).
2. Repo **Settings → Pages** → Branch: `main`, folder `/ (root)` → Save.
3. Wait a minute or two, then visit `https://<username>.github.io/<repo>/`.
4. On iPhone: open in Safari → allow location → Share → **Add to Home Screen**.

### Updating

Commit a new `index.html`. Note the caching layers:

- GitHub Pages CDN caches ~10 minutes after a commit.
- Safari and the Home Screen icon cache their own copies. Check the version number in the app header; if stale, load the URL with a cache-buster (`...?v=2`), or delete and re-add the Home Screen icon (saved rounds survive this — they belong to the site, not the icon).
- ⚠️ **Clearing Safari website data deletes all saved rounds.** Export first (Rounds tab → Export all data).

## Using the app

**Play tab.** Blue dot = you. Tap the map to set a measuring flag; the plate shows live yards to it. Buttons: **📍** re-center · **Mark shot** log your GPS position (then an optional sheet for club / miss tags / note; Skip keeps the shot untagged) · **Ball moved** penalty drop or free relief · **↺** undo the last mark (confirmed — GPS points can't be recreated) · **⋯** sun mode, edit club list, clear target pin, clear the hole's marks. The hole strip has stroke and putt counters (putts add to the stroke total automatically), Tee/FW/GIR chips cycling ✓ → ✗ → blank, and hole navigation. Tap any shot dot on the map to edit or delete it. Tap the putts number for the putting notes sheet.

**Scorecard tab.** Editable course name and pars, per-hole scores/putts/± and totals. **New round** (Solo or Scramble) · **Download** a named, formatted scorecard file · **Save round** to history.

**Rounds tab.** All saved rounds. **Map** replays a round's shot trail; **⬇** downloads its scorecard; **Export all data (JSON)** downloads everything for backup or analysis.

**Help tab.** The in-app version of these directions, plus a replayable welcome tour.

## Data & storage

All persistence is browser `localStorage` on the device (with a transparent fallback to a sandboxed storage API when the file runs inside certain preview environments). Keys:

| Key | Contents |
|---|---|
| `caddie-rounds` | JSON array of all saved rounds |
| `caddie-active` | Auto-saved in-progress round (debounced ~700 ms after every change) |
| `caddie-bag` | JSON array of the user's club list |
| `caddie-seen` | Flag that the welcome tour was shown |

### Round schema (version 3)

Both export formats carry this schema. The full export is `{app, schema: 3, exported, rounds: [...]}`; each downloaded scorecard embeds `{app, schema: 3, round}` in a `<script type="application/json" id="caddie-round-data">` block inside the HTML.

```json
{
  "version": 3,
  "id": "mcp3k9x-f83ka1z7q",     // unique per round (timestamp + random); use for dedup
  "mode": "solo",                 // "solo" | "scramble"
  "course": "My Course",
  "date": "2026-07-06T14:00:00.000Z",
  "pars":   [4,4,3, ...18],
  "scores": [5,null, ...18],      // total strokes per hole (swings + putts); null = not played
  "putts":  [2,null, ...18],
  "fir":    [true,false,null, ...18],   // fairway hit; null = unrecorded (or par 3)
  "gir":    [true,null, ...18],
  "tee":    [true,null, ...18],         // tee-shot quality chip
  "putting": [                    // per hole; null = no notes taken
    {
      "len": "mid",               // first-putt length: "short" (<6ft) | "mid" (6–20) | "long" (20+)
      "read": "misread",          // "misread" | "speed" | "unlucky"
      "green": "fast",            // "fast" | "slow" | "bumpy"
      "note": "",
      "putts": [ {"miss": "left", "note": ""} ]   // one entry per counted putt
    }
  ],
  "scramble": { "players": ["Me","P2"] },   // null unless mode = "scramble"
  "shots": [                      // 18 arrays of shot records, in order hit
    [
      {
        "lat": 34.05, "lng": -118.24,
        "t": 1720300000000,       // epoch ms when marked (null on migrated v1 data)
        "type": "tee",            // "tee" | "shot" | "drop" | "relief"
        "club": "Dr",             // from the user's bag; null if untagged
        "tags": ["push"],         // miss tags
        "note": "",
        "penalty": null,          // optional reason on drops: "ob" | "water" | "unplayable"
        "dropType": null,         // on type "drop": "entry" (point of entry / lateral / unplayable-nearby)
                                  //                 | "zone" (designated drop area, or across the obstacle)
                                  //                 | "redo" (stroke & distance, e.g. re-tee after OB)
        "penaltyStrokes": null,   // on type "drop": 0 | 1 | 2 (0 supports casual play; added to the hole score)
        "ledToDrop": true,        // set on the tee/shot whose ball required the following drop
        "player": null            // scramble: whose ball
      }
    ]
  ]
}
```

Analysis notes: shot distance = haversine distance between consecutive points in a hole's `shots` array (the app displays yards, `meters × 1.09361`). **Drop semantics matter for distance stats:** a shot flagged `ledToDrop: true` has no measurable distance — its ball was lost, OB, or unplayable — so exclude it from club distance averages but include it in attempt/accuracy counts (a drive that goes OB is still a driver swing). Any segment *ending* at a `type: "drop"` point is ball transport, not a shot. A `dropType: "redo"` means the next swing replays from (approximately) the previous spot; `"entry"` means play continues from where the ball entered trouble. `penaltyStrokes` (0–2) is the drop's score cost; `relief` points never cost strokes. `id` makes deduplication across export files exact. Fields are nullable throughout — untagged data is null, never fabricated.

### Versioning & migration

Every round carries `version`; exports carry top-level `schema`. On load, `migrate()` upgrades older records sequentially (v1 → v2 wraps bare `{lat,lng}` shots into full shot records; v2 → v3 backfills `penaltyStrokes: 1` and `dropType: "entry"` on old drops and flags the preceding shot with `ledToDrop`). Any future schema change should bump the version, extend `migrate()`, and never destroy fields — so old exports remain analyzable forever.

## Code architecture

Everything lives in `index.html`: CSS in `<style>`, markup, and one `<script>` at the bottom. Vanilla JS, no build step. The only dependency is [Leaflet 1.9.4](https://leafletjs.com) from cdnjs, with Esri World Imagery tiles (World Topo in sun mode).

Script layout, top to bottom:

- **Constants** — default pars, default bag, tag/option lists (`MISS_TAGS`, `PUTT_*`, `PENALTIES`, `RELIEFS`).
- **State & model** — `freshRound()`, `newShot()`, `makeId()`, and `migrate()` (schema upgrades).
- **Storage** — `stGet/stSet/stDel` (localStorage with sandbox fallback), `loadRounds/saveRounds`, `scheduleAutosave()` (debounced write of the active round), `tryResume()`.
- **Map** — Leaflet init, tile layers, marker icons by shot type, `yards()` haversine, `updateYardage()`, `drawShots()` (dots, tooltips, segment yardage labels, tap-to-edit).
- **GPS** — `watchPosition` wiring with a graceful fallback banner when unavailable.
- **Bottom sheet framework** — `openSheet/closeSheet`, `chipGridHTML/wireChips/chipValues` (reused by every dialog).
- **Sheets** — `openShotSheet` (tag/edit/delete a shot), `openMovedSheet` (drop/relief), `openMoreSheet` (options), `openPuttSheet` (putting notes), new-round flow, `showWelcome` (first-visit tour, `caddie-seen` flag).
- **Hole strip / scorecard / history / exports** — `syncHoleStrip`, `renderCard`, `renderHistory`, `exportRound` (scorecard HTML with embedded JSON), export-all, `downloadBlob`.
- **Init** — load bag, render, attempt resume, show welcome on first visit.

Conventions: destructive actions on GPS points always `confirm()` (they can't be recreated); every mutation calls `scheduleAutosave()`; all user tagging is optional and null when skipped; the header version string is bumped on every change so cached copies are identifiable.

## Privacy

No analytics, no accounts, no network calls except map tiles and the Leaflet CDN. Location is used live for yardage and stored only inside your own shot records on your device. Data leaves the phone only when you export it.
