# Notefox add-on — project notes for Claude

Fork of the official **Notefox: Websites notes** Firefox/Chrome extension
(upstream: Sav22999, published on the add-on stores). Manifest v2, version
tracks upstream (currently 4.7.2.x). Plain JS, no build step.

## Most important constraint: DO NOT diverge from upstream

The owner **uses the official store build of this add-on, unchanged**, and
wants to keep it that way (a fork would mean maintaining and re-publishing it).
So:

- **Default to NOT modifying this repo.** Behaviour changes — especially
  anything about sync, backups, or data safety — belong in the **backend**
  (`notefox-api`), not here.
- This repo is kept mainly as a reference/mirror and to read how the client
  behaves. Treat it as read-only unless the owner explicitly asks for an add-on
  change and accepts the maintenance cost.

## How it works (reference for backend work)

- `js/api-service.js` — talks to the API. Base URL `API_ENDPOINT_DEFAULT =
  https://www.notefox.eu/api/v1`, overridable via Settings → "Change API
  endpoint" (stored in `settings["api-endpoint"]`). Paths like `/login/`,
  `/data/insert/`, `/data/get/` are appended — so a custom endpoint must end in
  `/api/v1` with **no** trailing slash.
- `js/background.js` → `sendLocalDataToServer()` builds the sync payload:
  `JSON.stringify({notefox, settings, websites, "sticky-notes", storage,
  last-update})`. **Per-page notes live under `websites`** (keyed by URL, each
  with a `notes` string); sticky notes under `sticky-notes`. The server stores
  this as one encrypted blob per sync.
- The payload is sent as plaintext in the request body; the server encrypts it.

This payload shape is what the backend's empty-overwrite guard
(`countNotesInPayload()` in `notefox-api`) parses. **If this shape ever changes,
update that guard** so it doesn't mis-count and false-block saves.

## Data-loss context

The owner once lost all notes (likely a Firefox storage conflict / unexpected
logout caused an empty sync to overwrite the visible data). The fix lives in the
backend fork (empty-overwrite guard + backups + version restore), not here —
precisely to avoid forking the add-on.
