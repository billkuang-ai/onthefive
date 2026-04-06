# I-5 California Corridor — Claude Code Handoff

## What This App Is

A hyper-curated, zero-API, static PWA road trip companion for the California I-5 corridor (San Diego to Oregon border). Think Oregon Trail meets System 7 Macintosh — retro, quirky, personality-driven. No monetization. No bloat. Just a love letter to the I-5.

The editorial heart is an **Editor's Choice POI system** — every stop is hand-approved. No algorithmic suggestions, no user-generated content, no chain restaurant spam.

---

## File Structure

```
index.html      — the entire app (single file, ~930 lines)
sw.js           — service worker for offline caching
manifest.json   — PWA manifest for home screen install
CLAUDE.md       — this file
```

The app was originally planned as multi-file (pois.json, separate CSS, etc.) but was intentionally collapsed into a single `index.html` for simplicity and portability. Do not split it back out without a strong reason.

---

## Current Build State

**Phase 1** ✅ — Red line, POI cards, alternating left/right layout, retro aesthetic  
**Phase 2** ✅ — Live GPS, Turf.js snapping, directional awareness, localStorage favorites  
**Phase 3** ✅ (mostly) — Service worker, PWA manifest, safe area insets, offline caching  

### What's been built and is working:
- Vertical red I-5 spine, Oregon at top / San Diego at bottom
- 34 hand-curated POIs with geographic lat/lng snapping via Turf.js
- POI cards branch left/right based on actual freeway side (east/west of I-5 centerline)
- GPS dot snaps to I-5 line using `turf.nearestPointOnLine()`
- NB/SB direction toggle — corridor flips, cards reorder, "coming up next" pill updates
- Bottom sheet POI detail view (slides up, tap anywhere to dismiss)
- ★ Saved favorites filter — dims non-starred cards to 20% opacity
- County display in bottom bar — 16 real California counties with ENTERING/EXITING transitions at 3-mile boundaries
- Navigation app picker — Google Maps (default), Apple Maps, Waze — persisted to localStorage
- Settings sheet (⚙️ button)
- Service worker offline caching
- Safe area insets for notched iPhones (`viewport-fit=cover`)

---

## Key Architecture Decisions

### Single constant controls all scaling
```js
const PX_PER_MILE = 26.81; // ~20,000px total corridor height
```
Everything derives from this — card positions, dot placement, GPS snapping, gap labels, terminus markers. Change this one value to rescale the entire corridor.

### POI positioning
POIs are placed geographically by snapping their lat/lng to the I-5 GeoJSON line via Turf.js. Mile markers are computed at runtime — they are NOT hardcoded. The `I5_COORDS` array is a simplified polyline of ~50 waypoints along the actual I-5 centerline.

### East/West side assignment
`poi.side` is geographic truth — is the physical location east or west of I-5? This determines which side the card appears on. **POI placement bugs are almost always bad coordinate data, not logic errors.** Always audit raw lat/lng against real-world geography before investigating code.

### County system
16 real California counties replace the old editorial regions. County boundaries are calibrated against known POI positions:
```js
const COUNTIES = [
  {name:'SAN DIEGO COUNTY',   mile:0,   endMile:54},
  {name:'ORANGE COUNTY',      mile:54,  endMile:96},
  {name:'LOS ANGELES COUNTY', mile:96,  endMile:162},
  {name:'KERN COUNTY',        mile:162, endMile:233},
  {name:'KINGS COUNTY',       mile:233, endMile:258},
  {name:'FRESNO COUNTY',      mile:258, endMile:330},
  {name:'MERCED COUNTY',      mile:330, endMile:378},
  {name:'STANISLAUS COUNTY',  mile:378, endMile:418},
  {name:'SAN JOAQUIN COUNTY', mile:418, endMile:456},
  {name:'SACRAMENTO COUNTY',  mile:456, endMile:492},
  {name:'YOLO COUNTY',        mile:492, endMile:505},
  {name:'COLUSA COUNTY',      mile:505, endMile:540},
  {name:'GLENN COUNTY',       mile:540, endMile:578},
  {name:'TEHAMA COUNTY',      mile:578, endMile:618},
  {name:'SHASTA COUNTY',      mile:618, endMile:668},
  {name:'SISKIYOU COUNTY',    mile:668, endMile:746},
];
```

### Chip system
Tags and amenity flags are unified into a single chip array via `buildChips(poi)`. Max 5 chips shown in the detail sheet. Editorial tags fill first (priority-sorted), amenity flags fill remaining slots. Card map view shows max 3 emoji-only chips via `cardChips(poi)`.

Tag priority order: `iconic → hidden-gem → food → scenic → quirky → historic → shopping → rest-area → hotel → ev-charging → clean-restrooms → showers → fuel`

Amenity flags: `🎈 kids · 🐾 pets · ♿ ADA · 🚛 RV · 📶 good signal · 📵 weak signal`

Tags `kids`, `pet-friendly`, and `detour` are filtered out of chip rendering (covered by amenity flags or removed intentionally).

---

## Product Guardrails

- **Mobile-first always.** This is a phone-in-cupholder app. No hover states. No mouse assumptions. All touch targets thumb-reachable.
- **Zero APIs.** I-5 does not change. No live routing, no map tiles, no Places API.
- **No accounts, no backend, no user-generated POIs.**
- **Editor's Choice model is sacred.** Every POI is hand-approved. Don't add algorithmic suggestions.
- **Geographic accuracy > visual convenience.** If a card position looks wrong, check the coordinates first.
- **Restraint is the charm.** Don't add features just because you can. Every addition should earn its place.

---

## LocalStorage Keys

| Key | Value | Purpose |
|-----|-------|---------|
| `i5_fav` | JSON array of POI ids | Starred favorites |
| `i5_nav` | `'google'` \| `'apple'` \| `'waze'` | Navigation app preference |

---

## State Variables

```js
let dir = 'north';          // driving direction
let favs = [];              // starred POI ids
let favFilter = false;      // fav filter toggle state
let navApp = 'google';      // selected nav app
let gpsActive = false;      // GPS fix acquired
let gpsMile = null;         // current mile position on I-5
let gpsLat, gpsLng = null;  // raw GPS coordinates
let gpsWatchId = null;      // navigator.geolocation watch ID
let cardPositions = [];     // rendered card positions for next-pill logic
let nextTarget = null;      // next upcoming POI
```

---

## Navigation URL Schemes

```js
google: `https://www.google.com/maps/dir/?api=1&destination=${lat},${lng}&travelmode=driving`
apple:  `maps://maps.apple.com/?daddr=${lat},${lng}&dirflg=d`
waze:   `https://waze.com/ul?ll=${lat}%2C${lng}&navigate=yes&zoom=17`
```

---

## POI Data Schema

```js
{
  id: 'bravo-farms',
  name: 'Bravo Farms',
  tagline: 'The Wild West village that ate the freeway',
  town: 'Kettleman City',
  lat: 36.008,
  lng: -119.962,
  side: 'east',           // geographic east or west of I-5
  tags: ['food','kids','quirky','iconic'],
  kid: true,
  pet: true,
  ada: true,
  rv: true,
  ec: true,               // Editor's Choice badge
  note: 'Required stop. Non-negotiable...',
  cell: 'good'            // 'good' | 'weak' | 'poor'
}
```

---

## Known Gotchas

- **Animation vs opacity bug**: `.card` uses `animation:fadeIn forwards`. The animation fill overrides CSS class opacity rules. `.card.dimmed` must use `opacity:.2 !important` to win against the animation engine.
- **`let dir`** must be declared separately from any data arrays — it was accidentally deleted once when removing a nearby block and broke the entire app silently.
- **`yToMile()`** is defined after `updateCountyDisplay()` in source order but works fine because function declarations are hoisted. Don't move things around carelessly.
- **Turf.js `nearestPointOnLine`** returns distance in miles when `{units:'miles'}` is passed. The `.properties.location` value IS the mile marker — no scaling needed.
- **`I5_COORDS` spine waypoints** must stay in sync with POI coordinates. Mismatches cause GPS snapping errors.

---

## Remaining Work

- [ ] Test GPS snapping on a real device driving I-5
- [ ] Test offline behavior after service worker caches (disconnect from network, reload)
- [ ] Test county ENTERING/EXITING transitions at real county boundaries
- [ ] Verify safe area insets on notched iPhone (status bar, home bar)
- [ ] Consider adding a `MILE_MAX` console.log check to verify Turf.js route length (~746mi)
- [ ] Optional: richer editorial detail view (photos, hours) — deprioritized

---

## Deployment

**GitHub Pages** — drop all three files in repo root, enable Pages from Settings → Pages → main branch / root. Requires HTTPS for service worker to activate (GitHub Pages provides this automatically).

Service worker cache version is `i5-v1` in `sw.js`. Bump to `i5-v2` when pushing updates so users get fresh content.
