# Kiro prompts for Haunted Address

These are the exact prompts to paste into **Kiro**. Kiro writes all of the app code; these prompts only tell it what to build. The steering files in `.kiro/steering/` load automatically (all except `demo-addresses.md`, which you load by typing `#demo-addresses`).

> Honesty note for the write-up: these prompts, the steering drafts and the draft configs below (the ghost-auditor agent, the Playwright MCP entry and a sample hook) were prepared with Claude (Anthropic's assistant) on Nov 13 to 17, 2026, before any code existed. Kiro then turned them into specs, designs, tasks, code and tests. Say exactly that in the Devpost "How Kiro was used" section.

## Before the first spec: credits
- Until your 4,000 Kiroween credits show on the usage dashboard (look for a "kiroween" prefix), you are on **Kiro Free**: 50 credits a month and Free-tier models only. Keep the model on **Auto**.
- On Free credits, do the warm-up question and Spec 1's requirements and design. **Don't click Run all Tasks, and don't add hook 4** (it costs credits every time it fires) until the Kiroween credits are redeemed. One spec task usually costs more than 1 credit.
- If a model is refused later, switch back to Auto. Neither Kiro nor Devpost says which models the Kiroween credits cover.

## How to run a spec (do this for Specs 1, 2 and 3)

1. In Kiro, open the **Kiro panel** (the Kiro ghost icon in the left bar), go to **Specs** and click **+** (or choose **Spec** in the chat box).
2. Choose **Feature**, then **Requirements-First**.
3. Paste the prompt below, as is.
4. Kiro writes `requirements.md`. Click the floating **Review** button, read it, and comment on anything wrong. Then run **Analyze Requirements** (in the chat options or the Continue dropdown) to catch gaps. Approve.
5. Kiro writes `design.md`. In this phase, Kiro pulls **correctness properties** out of the requirements for property-based testing. Hover each one to see which requirement it comes from. Keep them. Approve.
6. Kiro writes `tasks.md`. **Property-based test tasks are optional by default, and Run all Tasks skips optional tasks.** If Kiro asks whether to keep optional tasks or make everything required, choose required. Otherwise, ask in the spec chat: "Make every property-based test task in tasks.md required." Check that no property-test task is still marked optional, then approve.
7. Run the first task or two one at a time and watch what Kiro does. After that, **Run all Tasks** is fine. It runs every required, incomplete task, with independent tasks in parallel waves.
8. After each spec, write 3 to 5 lines in `docs/KIRO-DIARY.md`: what surprised you, what you corrected, and what a hook or a property test caught.

Spec order: **1, then 2, then 3**. Specs 4 and 5 are stretch goals; skip them unless you are ahead of `PLAN.md`.

---

## Spec 1: `haunt-data-adapters`

Paste this:

```
Create the data layer for Haunted Address, following the steering files (product, tech, structure, data-sources, wording-rules, bilingual) and the verified results in docs/data-check-2026-11-13.md.

Goal: given one free-text address in New York City or Long Island (Nassau or Suffolk County), produce a list of typed GhostCard objects, one per data source. Each card has: id, state (found | clear | not_covered | unavailable), a reason key when not found, facts (label key, value, optional date), and a source citation (agency, dataset name, dataset URL, terms URL, data-as-of date).

Set up the project first: Vite + React + TypeScript strict, Vitest, fast-check, Wrangler for Cloudflare Pages Functions, the folder layout in structure.md, and the npm scripts tech.md lists for this spec (not check:wording, check:i18n or check:copy, which the UI spec adds). Add .env.example with empty values. Never add any line that mentions .kiro to .gitignore. On this Windows PC, run terminal commands with npm.cmd and npx.cmd (see tech.md).

Requirements:
1. Address normalization in src/core/address.ts: trim, collapse spaces, uppercase for matching, expand or contract common suffixes (Street/St, Avenue/Ave, Boulevard/Blvd, Road/Rd, Place/Pl, Drive/Dr), keep the house number exactly (including Queens hyphenated numbers like 104-20), map borough names (Manhattan to New York, Brooklyn to Kings, Staten Island to Richmond), and reject empty input or input over 200 characters. It must never throw.
2. A Cloudflare Pages Function at functions/api/geocode.ts that proxies the Census geocoder (geographies/onelineaddress, benchmark Public_AR_Current, vintage Current_Current, layers Counties and Census Tracts). It returns only {matchedAddress, lon, lat, countyFips, tractGeoid} for up to 5 matches, does not cache, never logs the address, and has an 8-second timeout. The Census geocoder has no browser CORS, so this proxy is required.
3. A service-area check: only the 7 county FIPS codes in data-sources.md are allowed. Any other county gets an "outside our area" result and no further calls.
4. NYC building IDs come from NYC Planning GeoSearch, called only for the 5 NYC counties. Accept a match only when the borough matches the county, the point is within 300 m of the Census point, and the house number matches the Census matched address. (On Nov 13, GeoSearch matched a Long Beach address to a Brooklyn building; 89 South St is 197 m from its Census point.) Treat placeholder BINs 1000000, 2000000, 3000000, 4000000 and 5000000 as no building ID. Otherwise the building ghosts are unavailable with reason building_not_matched.
5. The flood ghost comes from the FEMA NFHL ArcGIS REST service: layer 28 (zone) and layer 3 (FIRM panel effective date, keeping only panels whose FIRM_PAN starts with 36), mapped to the plain categories in data-sources.md. Zone D, no feature, or an ArcGIS error body means unavailable, never clear.
6. The storm ghost uses a build script, scripts/build-storms.ts (npm run data:storms). It:
   - reads the NOAA Storm Events CSV folder listing and downloads the newest details file for each of the last 10 full years plus the current year;
   - parses CZ_FIPS as an integer (the CSV does not zero-pad it);
   - keeps only the 7 counties and the forecast zones mapped in data-sources.md;
   - counts a zone storm once per county per (EPISODE_ID, EVENT_TYPE, begin date);
   - drops every damage, death, injury and narrative field;
   - writes public/data/storms/<countyFips>.json and baseline.json;
   - caches raw downloads in scripts/.cache/ (git-ignored).
   The adapter reads the JSON for the address's county and says the data is county-level.
7. The NYC ghosts come from NYC Open Data (Socrata SoQL), using aggregate queries only:
   - dob: DOB Violations 3h2n-5cm9, ECB 6bgk-3dad and DOB Complaints eabe-havv, all by BIN. For complaints, filter date_entered with ::floating_timestamp, because it is MM/DD/YYYY text.
   - hpd: HPD Violations wvxf-dwi5 and HPD Complaints ygpa-z7cr, both by BBL.
   - 311: erm2-nwe9 by BBL for the last 12 months, falling back to within_circle 75 m.
   Read each dataset's "as of" date from /api/views/<id>.json (rowsUpdatedAt). Send X-App-Token only to data.cityofnewyork.us and data.ny.gov, and only when VITE_SOCRATA_APP_TOKEN is set.
8. Long Island addresses get dob, hpd and 311 as not_covered with reason li_building_records (each town or village keeps those records).
9. src/adapters/http.ts: fetch with an 8-second timeout and one retry, with 400 to 1,200 ms jitter, on 429, on 5xx, or on an HTTP 200 body with an "error" key (ArcGIS sends errors that way). Socrata returned 503 on 3 of 10 calls on Nov 13. After the retry fails, the card is unavailable. A failure must never produce clear.
10. src/core/sources.ts: a registry with agency, dataset name, dataset URL, terms URL and coverage for every source, used by every card.
11. Tests:
    - unit tests for every adapter, using recorded fixtures in tests/fixtures (no live network in unit tests);
    - property-based tests for the address normalizer: idempotent; case- and whitespace-insensitive; house number preserved; never throws on any string;
    - a manual npm run smoke script that calls each live source once for the demo addresses (#demo-addresses).
12. Privacy: no address is stored, cached on a server or logged anywhere. No cookies.

Out of scope for this spec: UI components, the Haunt Meter math, and the narrator.
```

---

## Spec 2: `haunt-meter`

Paste this:

```
Create the Haunt Meter engine for Haunted Address in src/core/levels.ts and src/core/haunt.ts. It is pure TypeScript with no I/O, and its input is the GhostCard list from the haunt-data-adapters spec, with at most one card per ghost id. Follow the steering files, especially product.md (the meter is "just for fun", not a safety rating) and wording-rules.md.

Requirements:
1. Each ghost gets a level from 0 to 4 (shown as skulls) from a single threshold table in src/core/levels.ts. Each threshold has a one-line comment explaining why. Starting thresholds (Anthony may tune them later):
   - flood: minimal X = 0; moderate X (0.2%) = 1; A, AE, AH, AO, A99, AR = 3; V, VE = 4.
   - storms: ratio of the county's yearly average to the 7-county median in baseline.json. Under 0.5 = 0, under 0.9 = 1, under 1.1 = 2, under 1.5 = 3, otherwise 4.
   - dob: open ECB violations plus DOB violations in the last 3 years. 0 = 0, 1-2 = 1, 3-9 = 2, 10-24 = 3, 25 or more = 4.
   - hpd: weighted open violations (class C x3, B x2, A x1, I x0). 0 = 0, 1-4 = 1, 5-14 = 2, 15-39 = 3, 40 or more = 4.
   - 311: requests in the last 12 months. 0-4 = 0, 5-19 = 1, 20-49 = 2, 50-99 = 3, 100 or more = 4.
2. Only cards in state found or clear count as "checked". Cards that are not_covered or unavailable do not count and do not change the meter.
3. Meter = round(100 x sum of checked levels / (4 x number of checked ghosts)). With fewer than 2 checked ghosts, the meter is null and the UI shows "not enough data".
4. Output: the meter (0-100 or null), per-ghost levels, coverage {checked, notCovered, unavailable, total}, and a mood band key. The bands are meter.band.whisper (0-19), creaks (20-39), restless (40-59), haunted (60-79) and very_haunted (80-100). Band labels never say safe or unsafe.
5. explainMeter() returns i18n keys and parameters, so the UI can show "how this was calculated" in English or Spanish.

Correctness properties (write property-based tests with fast-check, at least 200 runs each; generate card lists with unique ghost ids):
- The meter is an integer from 0 to 100, or null.
- It is deterministic, and shuffling the card order does not change it.
- Raising one ghost's level, with the others fixed, never lowers the meter.
- Each threshold function is monotonic: a higher count never gives a lower level.
- Flood ordering: V/VE level >= A-family level >= moderate X level >= minimal X level.
- Adding a not_covered or unavailable card never changes the meter or the number of checked ghosts.
- The meter is 0 exactly when every checked level is 0, and 100 exactly when every checked level is 4.
- Fewer than 2 checked ghosts gives a null meter.
- The result does not depend on the UI language.
```

---

## Spec 3: `haunt-report-ui`

Paste this (type `#demo-addresses` first, so the example chips use the right addresses):

```
Create the Haunted Address user interface: one page in React that uses the adapters from haunt-data-adapters and the engine from haunt-meter. Follow every steering file, especially product.md, wording-rules.md and bilingual.md.

Requirements:
1. Header with the app name, a short line in the current language ("What do public records say about this address?"), and an English/Español toggle that remembers only the language choice.
2. Address form with a label, a submit button and example chips for the demo addresses. While geocoding, show a calm loading state. With 2 or more matches, show a pick list. With 0 matches, or outside the service area, show a friendly message with tips.
3. Results: the Haunt Meter at the top (the number, skulls, the mood band, coverage "checked X of Y", the "just for fun, not a safety rating" line and a "how this was calculated" disclosure), then one card per ghost.
4. Each ghost card shows:
   - the spooky name and the plain title;
   - a state badge (found / clear / not covered / couldn't check);
   - 1 to 3 plain sentences and up to 4 key facts;
   - a "what this does and doesn't mean" line;
   - the citation line "Source: agency, dataset · data as of date", with a link that opens in a new tab.
   The not-covered state says where such records live. The couldn't-check state has a Try again button that reloads only that card.
5. Each card loads on its own, with a skeleton and a gentle animation. All motion stops under prefers-reduced-motion.
6. Footer:
   - the full disclaimer from wording-rules.md in the current language, and the non-endorsement line;
   - a privacy note: "We don't keep the addresses you type. To look one up, the app sends it to the U.S. Census Bureau and, for NYC addresses, to NYC Planning." (ES: "No guardamos las direcciones que escribe. Para buscarla, la app la envía a la Oficina del Censo de EE. UU. y, si está en NYC, a NYC Planning.");
   - "Built with Kiro for Kiroween 2026", and a link to the GitHub repo.
7. Look:
   - a dark "haunted" theme with WCAG 2.2 AA contrast, and one accent color per ghost;
   - original inline SVG ghost icons (no copyrighted art, no agency logos);
   - a display font only for the spooky names, and a readable font for everything else;
   - it works from 360 px wide to desktop.
8. Accessibility: semantic landmarks, visible focus, full keyboard use, an aria-live region that announces when results are ready, an accessible name for every icon, and lang switching on <html>.
9. All copy lives in src/i18n/en.json and es.json. Add:
   - scripts/check-i18n.ts: keys and placeholders must match;
   - scripts/check-wording.ts: the banned terms in wording-rules.md, whole words only, with its listed exceptions;
   - npm run check:copy, which runs both.
10. Tests:
    - component tests for each card state in both languages;
    - a property test that en.json and es.json have identical keys and placeholders;
    - Playwright end-to-end tests with a mocked network for: Midtown Manhattan (NYC, all ghosts), Long Beach (Long Island: building ghosts not covered), outside the area, and a Socrata failure (the card shows couldn't check, and Try again works).
11. Performance: no layout shift when cards arrive; Lighthouse mobile accessibility 90 or higher.
```

---

## Spec 4 (stretch): `ghost-narrator`

```
Add an optional AI narrator to Haunted Address. Follow wording-rules.md, "Narrator (LLM) limits".

Requirements:
1. A Cloudflare Pages Function at functions/api/narrate.ts that accepts {language: "en" | "es", cards: GhostCard[]}. It never accepts the raw address.
2. It calls one LLM through a small provider interface. Default: Cloudflare Workers AI through a binding named AI (no API key in code), using a model on Workers AI's free allowance (not one that needs paid billing). Keep the provider swappable through configuration. If the model's license asks for attribution (for example "Built with Llama"), add it to the footer and README.
3. The prompt asks for 3 to 5 sentences: spooky in tone, using only the facts in the cards, with no advice, no prices and no insurance words.
4. src/narrate/validate.ts rejects output that contains a number or date not present in the card facts, a banned term from wording-rules.md, or more than 5 sentences.
5. If validation fails, the call errors, or it takes more than 8 seconds, return the deterministic template summary from src/narrate/template.ts instead, and say so in the response.
6. The UI shows the summary under the meter, labeled "AI-written summary of the cards above" (or the Spanish label), with a toggle to hide it.
7. Property tests: the validator rejects any injected number not in the facts; the template summary always passes the validator; the output language matches the request.
```

## Spec 5 (stretch): `long-island-extra-ghosts`

```
Add two more ghosts that cover Long Island and NYC, following data-sources.md, "Stretch sources":
- nri: FEMA National Risk Index ratings by point (overall, coastal flooding, hurricane, inland flooding, winter weather). Rating words only, never dollar fields. Use field IFLD_RISKR (RFLD_RISKR does not exist).
- remediation: NYS DEC Environmental Remediation Sites within 800 m (data.ny.gov c6ci-rzpg, within_circle on georeference). Count distinct program_number (rows repeat per operable unit), and show the 3 closest distinct facility names.
Add level thresholds for both to levels.ts, with property tests for monotonicity, plus fixtures, unit tests, EN/ES copy and citations.
```

---

## Agent hooks

Hooks live in `.kiro/hooks/*.json`. The easiest way to create one is to **ask Kiro in chat**, using the "Ask Kiro" text below. You can also use the **Agent Hooks** section of the Kiro panel. Things to know:
- **File hooks fire only on changes the agent makes**, not on your manual edits.
- **Shell-command hooks cost no credits; agent-prompt hooks do.** Prefer command hooks.
- Hooks activate when a session starts. If a new hook doesn't fire, start a new chat session before deciding it's broken.
- Kiro runs hook commands through `cmd.exe` on Windows, so plain `npm`/`npx` works there (the PowerShell block in `tech.md` affects the agent's terminal, not hooks). If a hook ever reports "running scripts is disabled", change its command to `npm.cmd`/`npx.cmd`. Keep each command to one `npm run ...`.
- Trigger names in the JSON files: File Save = `PostFileSave`, File Create = `PostFileCreate`, Post Task Execution = `PostTaskExec`, Pre Task Execution = `PreTaskExec`, Pre Tool Use = `PreToolUse`, Post Tool Use = `PostToolUse`, Agent Stop = `Stop`, Prompt Submit = `UserPromptSubmit`.

| # | When to create it | Hook | Trigger | Ask Kiro |
|---|---|---|---|---|
| 1 | After Spec 1 task 1 | Tests on save | File Save, `.ts`/`.tsx` | "Create a hook that runs `npx vitest run --changed --passWithNoTests` whenever you save a .ts or .tsx file, with a 120-second timeout." |
| 2 | After the normalizer task | Property tests for the core | File Save, `address.ts`, `levels.ts`, `haunt.ts` | "Create a hook that runs `npm run test:property` whenever you save src/core/address.ts, levels.ts or haunt.ts, with a 180-second timeout. Make the matcher work with Windows backslash paths as well as forward slashes." |
| 3 | After Spec 3's check:copy task | Wording and i18n check | File Save on `src/i18n/*.json`, `src/narrate/**`, `src/ui/**` | "Create a hook that runs `npm run check:copy` whenever you save an i18n JSON file, a narrator file or a UI component. Make the matcher work with Windows backslash paths." |
| 4 | Only after the Kiroween credits arrive (it costs credits each time) | README updater | Post Task Execution (agent prompt) | "Create a hook that runs after each spec task finishes. If the task changed setup steps, npm scripts, environment variables, data sources or user-visible features, update the matching section of README.md (Setup, Scripts, Data sources, Features). Keep the Disclaimer and License sections word for word. If nothing user-facing changed, change nothing." |
| 5 | Optional | `.kiro` guard (good for the write-up) | Pre Tool Use on `write` | "Create a hook that blocks any write to .gitignore that would make it ignore .kiro, because the hackathon rules disqualify that. Use a small Node script in scripts/hooks/ that reads the hook event JSON from STDIN (tool_input). When the write would ignore .kiro, it prints the reason to STDERR and exits with code 2. Otherwise it exits 0. Only exit code 2 blocks; other non-zero codes just warn." |

For reference, Kiro's documented hook file format, with hook 1 as the example (Kiro writes the real file):

```json
{
  "version": "v1",
  "hooks": [
    {
      "name": "Tests on save",
      "trigger": "PostFileSave",
      "matcher": "\\.(ts|tsx)$",
      "action": { "type": "command", "command": "npx vitest run --changed --passWithNoTests" },
      "timeout": 120
    }
  ]
}
```

---

## MCP server worth using: Playwright

It lets Kiro open your running app in a real browser, type the demo addresses, click the Español toggle, check keyboard focus, and take screenshots for the README and Devpost. It is the most useful MCP for a UI-heavy entry. It uses your installed Google Chrome by default.

Ask Kiro: "Add the Playwright MCP server to this workspace." Or create `.kiro/settings/mcp.json` with:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"],
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

If the server won't start on Windows, change it to `"command": "cmd", "args": ["/c", "npx", "-y", "@playwright/mcp@latest"]`. If it still doesn't show in the Kiro panel, open Settings (Ctrl+,), search for MCP, and check that MCP is enabled.

Then try: "Start the dev server, open it with the Playwright MCP, search each demo address in English and Spanish, and save screenshots to docs/screenshots/. Tell me anything that looks broken or hard to read."

## Kiro powers: Context7

As of Nov 17, the powers registry has a **Context7** power (current library docs). It also has a Netlify power, but no Cloudflare power.
- Install Context7 from the **Powers** panel: the Ghosty-with-lightning icon, search "Context7", **Install**. Kiro then loads current docs for Vite, Wrangler or fast-check only when they come up.
- It gives you a true answer to the judges' "which powers did you use" question. Write down in the diary what it changed. If it changed nothing, say so or leave the line out.
- If it ever asks for an API key, keep the key out of `mcp.json` by using Kiro's `${VAR}` environment expansion.

Bigger stretch idea: have Kiro build a small **Haunted Address MCP server** (stdio) that exposes the adapters as tools (`geocode`, `flood_zone`, `storms_for_county`, `nyc_building_records`). Then Kiro itself can "ask the ghosts" when it writes fixtures. That's a good "Frankenstein" story for judges.

---

## Custom agent: `ghost-auditor`

A reviewer you run before every push and before recording the video. It has read tools plus a few pre-approved check commands; any other command asks you first. Ask Kiro: "Create a custom agent called ghost-auditor in .kiro/agents/ from this draft," then paste:

```json
{
  "name": "ghost-auditor",
  "description": "Reviewer that checks Haunted Address copy, cards, citations and Spanish against the steering rules. Never edits files.",
  "tools": ["read", "shell"],
  "resources": [
    "file://.kiro/steering/**/*.md",
    "file://docs/data-check-2026-11-13.md"
  ],
  "permissions": {
    "rules": [
      { "capability": "shell", "match": ["npm run check:*", "npm.cmd run check:*", "npm run test*", "npm.cmd run test*", "npx vitest run*", "npx.cmd vitest run*"], "effect": "allow" }
    ]
  },
  "prompt": "You are the Ghost Auditor for Haunted Address. Never edit files. Review src/i18n/en.json, src/i18n/es.json, src/narrate, src/ui and README.md against wording-rules.md, bilingual.md and data-sources.md. Run npm.cmd run check:copy and npm.cmd run test. Report a table with these columns: file, line, rule broken, the exact text, and a suggested fix in English and Spanish. Check that every ghost card cites agency, dataset, link and data-as-of date, that no failure path can show 'clear', and that no dollar amount appears anywhere. End with PASS or FAIL."
}
```

Kiro's docs disagree on whether custom agents inherit steering by default, so the `resources` line makes sure the auditor always loads every steering file and the data-check log. To use it, pick **ghost-auditor** in the agent picker at the bottom of the chat box (under WORKSPACE), ask "Audit the app," and save the report in the diary.

---

## Kiro CLI: skipped by default

- Kiro's docs list **Windows 11** for the native Windows CLI. Your PC runs Windows 10, and **WSL is not installed**. The IDE works on Windows 10, so the plan skips the CLI.
- If you are ahead of schedule and want one CLI story for the write-up (your call; installing WSL needs admin rights and a restart):
  1. In an admin PowerShell, run `wsl --install -d Ubuntu`, then restart.
  2. In Ubuntu, install Node 22.12 or newer and the CLI (`curl -fsSL https://cli.kiro.dev/install | bash`), then run `kiro-cli login` with the same account you use in the IDE.
  3. Work in a clone inside Linux, not under `/mnt/c`, because Windows-installed `node_modules` won't run there: `git clone https://github.com/<you>/haunted-address ~/haunted-address`, then `cd ~/haunted-address`, then `npm ci`.
  4. Run `kiro-cli --agent ghost-auditor` and ask for an audit.
- The CLI's V3 harness (the default for interactive `kiro-cli`) reads the same `.kiro/` steering, hooks, agents and MCP config. A headless run would need `--v3` and `--trust-tools`, and may ask for `KIRO_API_KEY`, which needs a paid plan. Skip headless.

## Not recommended for this entry
- **Kiro Web** needs a paid Kiro plan (Pro or higher).
- **Kiro Crew** is a separate personal-agent app. The "Best Kiro Crew App" bonus needs a Crew app of its own, so don't chase it now.
