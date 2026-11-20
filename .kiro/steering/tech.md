---
inclusion: always
---
# Tech stack and rules

## Stack
- **TypeScript** in strict mode everywhere.
- **Vite + React** (current stable) for one single-page app. Plain CSS with custom properties; no UI framework.
- **Vitest** for unit tests, **fast-check** for property-based tests, **Playwright** for a few end-to-end tests.
- **Cloudflare Pages** for hosting, with **Pages Functions** in `functions/api/` for the two server endpoints. Use **Wrangler** for local dev of functions.
- Node.js 22.12 or newer on an LTS line (22, 24 or 26), plus npm. Current Wrangler, Vite and Vitest require 22.12+, and Vitest 5 rejects Node 25. The build PC has Node v26 (LTS) and npm 11.20.0. Cloudflare Pages builds with Node 22 by default, which is fine.
- No database, no login, no server-side storage.
- Stay on **Pages Functions** (`functions/api/*.ts`, the `PagesFunction` type, `wrangler pages dev dist`). Cloudflare's docs now suggest Workers for new projects, so ignore Workers-only APIs. Keep functions only under `functions/api/`: no `functions/_middleware.ts` and no root catch-all route, so static requests never count toward the Functions quota. Set headers with `public/_headers`.
- In dev, Vite proxies `/api` to `wrangler pages dev` (Vite `server.proxy`), or the app is built and served with `wrangler pages dev dist`.

## Windows build PC: npm in PowerShell
This PC's Windows PowerShell blocks `npm.ps1` and `npx.ps1` ("running scripts is disabled on this system") unless Anthony changes the execution policy. So when you (Kiro) run a command in the terminal, **use `npm.cmd` and `npx.cmd`** (for example `npm.cmd install`, `npx.cmd vitest run`). Hook commands and `package.json` scripts keep plain `npm`/`npx`. Never change the execution policy yourself; that is Anthony's decision.

## Why these choices
- Static hosting plus two tiny functions keeps the app free to run and fast.
- Cloudflare's free plan serves static files without a request cap and allows 100,000 function requests a day. Commercial use is allowed, so the URL keeps working through judging.
- fast-check lets Kiro's property-based testing express rules like "the meter never goes above 100".

## How each source is called (verified Nov 13, 2026; see `docs/data-check-2026-11-13.md`)

| Source | From | Why |
|---|---|---|
| Census Geocoder | `functions/api/geocode.ts` (server proxy) | No browser CORS |
| NYC Planning GeoSearch | browser | CORS `*` |
| FEMA NFHL and FEMA NRI (ArcGIS REST) | browser | CORS allowed |
| NYC Open Data and data.ny.gov (Socrata SoQL) | browser, optional `X-App-Token` header (only to these two hosts) | CORS `*` |
| NOAA Storm Events | build time: `scripts/build-storms.ts` writes `public/data/storms/<countyFips>.json` | 12.6 MB gzipped per year is too big for browsers |
| LLM narrator (stretch) | `functions/api/narrate.ts` | Keeps model credentials on the server |

## Reliability rules
- Every network call has an **8-second timeout**.
- Retry **once** on HTTP 429, on 5xx, or on an HTTP 200 body with an `error` key (ArcGIS sends its errors that way), with 400 to 1,200 ms jittered backoff. Socrata returned 503 on 3 of the first 10 calls on Nov 13, and the retries worked.
- After the retry fails, the card state is `unavailable` ("couldn't check"). **Never** turn a failure or timeout into `clear`.
- Each card loads on its own. One slow source never blocks the others.
- **No server-side caching of anything that contains an address.** `/api/geocode` does not cache and never logs. The browser memoizes results for the session only. Dataset "as of" dates (no address in them) may be cached in the browser for 24 hours.

## Secrets and config
- Environment variables only. `.env` files are never committed; `.env.example` lists the names with empty values.
- `VITE_SOCRATA_APP_TOKEN` is optional. A token in the browser bundle is visible to every visitor, and anyone who copies it uses its quota. That is an accepted trade-off for a free, read-only app token. Never put a Socrata *secret* token or any real secret in the browser.
- LLM credentials, if any, live only in Cloudflare environment variables or bindings, never in client code.
- Never put API keys in `.kiro/settings/mcp.json` or in steering files.

## Testing rules
- Unit tests never touch the live network. They use recorded fixtures in `tests/fixtures/`, saved from the verified sample queries.
- Property-based tests (fast-check, at least 200 runs each) cover the address normalizer, the Haunt Meter, level thresholds and English/Spanish key parity.
- `npm run smoke` is a manual script that calls each live source once for the demo addresses. It never runs in CI or hooks.
- Tests must pass before a spec task is marked complete.

## Code rules
- Pure functions in `src/core/` (no I/O). Adapters in `src/adapters/` return a typed `Result` (`{ ok: true, value } | { ok: false, error }`) and do not throw across module boundaries.
- Dates are ISO strings in data. Format for display with `Intl.DateTimeFormat` using `en-US` or `es-US`.
- Every user-visible string goes through the i18n files. No hard-coded copy in components.
- Keep dependencies few. Prefer the platform `fetch`, `URL` and `Intl`.
- No dollar amounts anywhere in the UI. Drop NOAA damage dollars and FEMA NRI expected-loss dollars at the adapter level.

## npm scripts (Kiro creates them)
- From the data-adapters spec: `dev`, `build`, `preview`, `test`, `test:property`, `data:storms`, `smoke`.
- From the UI spec: `check:wording`, `check:i18n`, `check:copy` (runs both checks).
