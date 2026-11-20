---
inclusion: always
---
# Project structure

```
/.kiro/                     Kiro config. Always committed; NEVER add it to .gitignore (the hackathon rules disqualify that).
  steering/                 These rules
  specs/                    One folder per spec (requirements.md, design.md, tasks.md)
  hooks/                    Agent hooks (*.json)
  agents/                   Custom agents (ghost-auditor)
  settings/mcp.json         MCP servers (no secrets in here)
/src/
  core/
    address.ts              Normalize and validate free-text addresses (pure)
    cards.ts                Ghost card types and helpers (pure)
    levels.ts               Per-ghost level thresholds, with a short reason for each (pure)
    haunt.ts                Haunt Meter math (pure)
    sources.ts              Source registry: agency, dataset, URLs, terms, coverage
    result.ts               Result type and helpers
  adapters/                 One file per source; each returns Result<GhostCard>
    census.ts               Calls /api/geocode
    nycGeosearch.ts
    femaNfhl.ts
    noaaStorms.ts           Reads /data/storms/<countyFips>.json
    nycDob.ts
    nycHpd.ts
    nyc311.ts
    femaNri.ts              (stretch)
    nysRemediation.ts       (stretch)
    http.ts                 fetch with timeout and retry
  narrate/                  (stretch) template.ts, validate.ts, client.ts
  i18n/
    en.json
    es.json
    index.ts
  ui/                       React components (App, AddressForm, HauntMeter, GhostCard, CardStates, Footer, LanguageToggle)
  styles/
/functions/api/
  geocode.ts                Census proxy (Cloudflare Pages Function)
  narrate.ts                (stretch) LLM narrator
/scripts/
  build-storms.ts           NOAA CSV to per-county JSON (run by hand, commit the output)
  check-wording.ts          Fails on banned phrases (see wording-rules.md)
  check-i18n.ts             Fails when en/es keys or placeholders differ
  smoke.ts                  Live check of every source (manual only)
/public/data/storms/        Generated NOAA JSON (committed, small)
/tests/
  unit/
  property/                 fast-check tests
  fixtures/                 Recorded API responses
  e2e/                      Playwright tests with mocked network
/docs/
  data-check-2026-11-13.md  Live source verification
  KIRO-DIARY.md             What Kiro did, for the hackathon write-up
```

## Naming
- Files: `camelCase.ts` for modules, `PascalCase.tsx` for React components.
- Ghost ids: `flood`, `storms`, `dob`, `hpd`, `311`, `nri`, `remediation`.
- i18n keys: `ghost.<id>.title`, `ghost.<id>.spooky`, `ghost.<id>.body.<state>`, `meter.*`, `footer.*`, `error.*`.
- Tests mirror the source path: `src/core/haunt.ts` is tested by `tests/unit/core/haunt.test.ts` and `tests/property/haunt.property.test.ts`.

## Imports
- `src/core` imports nothing from `adapters`, `ui` or `narrate`.
- `src/ui` never calls `fetch` directly. It goes through adapters.
- Relative imports within a folder; the `@/` alias for `src/`.
