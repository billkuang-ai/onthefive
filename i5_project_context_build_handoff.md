I-5 Road Trip App — Project Context & Build Handoff
What This App Is
A hyper-curated, zero-API, static web app (PWA) for the California I-5 corridor (San Diego to Oregon border). Think Oregon Trail meets System 7 Macintosh — retro, quirky, personality-driven. No monetization. No bloat. Just a love letter to the I-5.

The editorial heart of the app is an Editor's Choice POI system — every stop requires the developer's personal stamp of approval. No algorithmic suggestions, no user-generated content, no chain restaurant spam.

Mobile-First Design Requirement
This is a driving companion designed to sit in a phone mount or cupholder. Design for mobile from the first line of CSS.

Primary viewport: portrait-mode phone screen

The red I-5 line runs vertically down the center of a tall, narrow screen

All touch targets must be thumb-reachable

No hover states. No mouse assumptions.

Desktop is a nice-to-have, not a requirement.

Core UX Concept
Main screen: a thick bold red vertical line down the center of the screen representing I-5.

POIs appear as cards branching left and right of the line, like a vertical timeline.

Your GPS dot appears as a blinking marker on the red line, showing real-time position.

Upcoming POIs surface naturally as you drive north or south.

Aesthetic: system.css (retro Apple System 7 CSS library), bitmap fonts, dithered backgrounds, dialog-box-style POI cards.

No map tiles, no turn-by-turn, and no routing. When a user wants navigation to a POI, one button deep-links to Google Maps or Waze.

Product Direction Since the Pivot
The project shifted away from monetization-heavy planning and toward a much smaller, more niche, more personal product. The app is now intentionally editorial, playful, and highly opinionated.

Key decisions made since the pivot:

Keep the experience extremely niche and corridor-specific.

Make POIs exclusive and hand-approved under an Editor's Choice model.

Prioritize cute, quirky, retro UX over feature completeness.

Avoid paid APIs and unnecessary infrastructure.

Treat this as a static, offline-friendly driving companion rather than a full route planner.

Preserve a singular editorial voice instead of opening the app to public submissions.

Tech Stack (Decided)
Component	Solution
Hosting	GitHub Pages (free, static)
Route data	I-5 GeoJSON from OpenStreetMap via Geofabrik
GPS positioning	navigator.geolocation
GPS snapping to line	Turf.js nearestPointOnLine()
POI data	Hand-curated pois.json
UI framework	system.css
Offline support	Service Worker + pre-cached assets
Keep screen on	Screen Wake Lock API
APIs needed	Zero
Monthly cost	$0
Why No APIs
The route is fixed. I-5 does not change. No live route generation, map tiles, or Places APIs needed. Everything runs locally once cached — making dead zones much less of a concern than in a typical map app.

PWA vs Native Decision
Start as a PWA. The concept maps cleanly to the web, distribution is instant via URL, and offline support is handled by a Service Worker. Native iOS remains a future option if true background GPS or App Store packaging become priorities.

POI Philosophy
Every POI should feel chosen, not scraped. Selection principles:

Personally interesting, beloved, odd, iconic, or unexpectedly useful

Strong corridor relevance

Worth stopping for, not just technically present along I-5

Themed around delight, relief, story, or atmosphere

User Favorites Decision
Do not open the app to public POI creation in v1.

Allow users to star existing Editor's Choice POIs locally via localStorage.

No public submissions at launch.

If community input is desired later, use a lightweight "Suggest a Stop" email link instead of a backend moderation system.

Radio Feature Decision
Very on-brand for the retro concept but deprioritized for v1. Reserve optional placeholder fields in the data model for future corridor notes.

POI Data Schema (Draft)
json
{
  "id": "bravo-farms",
  "name": "Bravo Farms",
  "tagline": "The Wild West village that ate the freeway",
  "mile_marker": 278,
  "lat": 35.9876,
  "lng": -119.9723,
  "side": "east",
  "direction": "both",
  "tags": ["food", "kids", "quirky", "iconic", "pet-friendly", "rv-friendly"],
  "kid_friendly": true,
  "pet_friendly": true,
  "ada_accessible": true,
  "rv_friendly": true,
  "editors_note": "Required stop. Non-negotiable.",
  "image": "bravo-farms.jpg",
  "corridor_notes": {
    "radio_dead_zone": false,
    "cell_reception": "good"
  }
}
Recommended First Build Prompt for Claude
Build a working static PWA for a California I-5 road trip app.

Design requirements:

Mobile-first, portrait orientation — this is a phone-in-cupholder driving app

Thick red vertical line centered on a tall narrow screen representing I-5

POI cards branching left and right from the line

GPS dot that snaps to the line using Turf.js nearestPointOnLine()

Retro System 7 Macintosh aesthetic using system.css

All touch targets thumb-reachable, no hover states
Technical requirements:

Static HTML/CSS/JS only — no paid APIs

POIs from local pois.json

I-5 route from local GeoJSON file

Service Worker for offline support

Screen Wake Lock to keep screen on while driving

Start with 3 placeholder POIs
Please create the first working prototype with a clear file structure.

Build Phases
Phase 1: Red line, local POI JSON, alternating POI cards, retro Mac aesthetic, mobile layout

Phase 2: Live GPS, Turf.js snapping, directional awareness, localStorage favorites

Phase 3: Offline caching, navigation deep links, editorial detail views, mobile polish

Product Guardrails for Claude
Design mobile-first from the very first line of CSS

Do not turn this into a generic planner

Do not add account systems, user-generated POIs, or backend complexity

Do not introduce map APIs without a very strong reason

Keep the interface playful, sparse, and opinionated

Respect the Editor's Choice concept as the core of the product

This app should feel like a handmade roadside companion for one specific highway, not a startup attempting to cover every travel use case. The charm comes from restraint.