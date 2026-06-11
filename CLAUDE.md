# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file static trip planning page (`index.html`) for a 6-day Basque Country road trip (15–20 Jul 2026). No build step, no dependencies to install. Deployed via GitHub Pages at https://bramvancamp.github.io/basque-trip.

## Development

Open `index.html` directly in a browser, or serve it locally to avoid Firebase Auth restrictions:

```bash
npx serve .
# then open http://localhost:3000
```

Deploy by pushing to `main` — GitHub Pages deploys automatically.

## Architecture

Everything lives in `index.html` as a single file: CSS in `<style>`, content as HTML, and two script blocks at the bottom.

**Script block 1 & 2 — Leaflet maps (plain `<script>`):**  
Two separate Leaflet map initialisations — one for the trip route (`#trip-map`) and one for the events section (`#events-map`). Both use CartoDB light tiles and custom div-icon pins.

**Script block 3 — Firebase module (`<script type="module">`):**  
Handles auth, access control, and ratings. Key flow:
- On load, `showLogin()` hides the entire site behind a full-screen overlay
- `onAuthStateChanged` checks the user's doc in Firestore `users/{uid}`
- UID `5wklihfuHxT8FtCZgWzvhqTfUqh2` (admin) is auto-approved; all others need `approved: true` in their Firestore doc
- Approved users see the site; unapproved users see a pending screen
- `initRatings()` attaches `onSnapshot` listeners to `ratings/{itemId}/votes` for every `[data-rating-id]` element and renders star widgets in real time

## Firebase

- **Project:** `trip-512bc`
- **Database:** named `trip` (not the default) — always pass `getFirestore(app, 'trip')`
- **Auth:** Google Sign-In via popup; domain `bramvancamp.github.io` must stay in Firebase Auth → Authorized Domains
- **Firestore collections:**
  - `users/{uid}` — `{ displayName, email, approved: bool, requestedAt, admin?: bool }`
  - `ratings/{itemId}/votes/{uid}` — `{ score, uid, displayName, updatedAt }`
- **Firestore rules:** reads open, writes require `request.auth != null`

## Ratable items

Any HTML element with a `data-rating-id` attribute gets a star widget automatically. Current IDs follow the pattern `gure-toki`, `la-cuchara`, `sopelana-surf`, `event-jazz-vitoria`, etc. Add the attribute to any new element to make it rateable — no other wiring needed.

## Approving users

Firestore → Data → `users` collection → open the document → set `approved` field to `true`.
