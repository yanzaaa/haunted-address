# Haunted Address: build plan (Nov 17 to Dec 4)

**Deadline: Friday, Dec 4, 2026, 2:00 PM Pacific (5:00 PM New York).** No extensions. Aim to submit on **Thu Dec 3**, and keep Friday as a buffer.

**After the deadline:** the submission is locked. The app and the public repo must stay working, free and without login until judging ends: **Fri Jan 29, 2027, 1:00 PM PT**. Judging runs Dec 16, 2026 to Jan 29, 2027, and winners are announced around Feb 3, 2027.

**Category:** Frankenstein. It stitches ArcGIS REST (FEMA), Socrata SoQL (NYC Open Data), a bulk CSV archive (NOAA), the Census geocoder and, if built, an LLM narrator into one app.

**Builder:** one person with a day job, about 1.5 hours on weeknights and 4 to 6 hours on weekend days. That's about 35 to 40 hours from Tue Nov 17. **Kiro writes the app code.** You steer, review and test.

Started Tue Nov 17, four days later than first planned. Stretch goals happen only if everything through Fri Nov 27 is done. The Kiro CLI is skipped (WSL isn't installed on this PC).

## What wins points (judging: 5 equal criteria)

| Criterion | What the minimum demo shows |
|---|---|
| Potential impact | A renter, buyer, small landlord or small business owner types an address and understands its flood zone, storm history and building records in under a minute, in English or Spanish. |
| Technological implementation (Kiro use) | 3 specs built Requirements-First; 7 steering files using two inclusion modes; 3 or 4 hooks (tests on save, property tests, wording check, optional README updater); Kiro's property-based testing on the Haunt Meter and the address normalizer; the Playwright MCP to test the UI; the Context7 power; a custom "ghost-auditor" agent. Keep a real diary in `docs/KIRO-DIARY.md`. |
| Design | One clean page: a dark "haunted" look with readable contrast, one card per ghost, four clear states (found / clear / not covered / couldn't check), works on a phone. |
| Innovation (public datasets) | 4 to 6 public datasets in 4 different API styles joined on one address, plus an honest coverage map (NYC vs. Long Island). |
| Presentation | A 2:45 video: the problem, an NYC demo, a Long Island demo, the Spanish toggle, then about 45 seconds of how Kiro built it. |

## Minimum demo (must ship)

1. Address box with example chips. NYC and Nassau/Suffolk addresses work; anything else gets a polite "outside our area".
2. Census geocoder through `/api/geocode` (it has no browser CORS), with no caching or logging of addresses.
3. **Flood ghost** (FEMA NFHL zone and map effective date) for NYC and Long Island.
4. **Storm ghost** (NOAA Storm Events, last 10 years, county plus forecast zones) for NYC and Long Island, from pre-built JSON.
5. **NYC building ghosts**: DOB violations and complaints by BIN, HPD violations and complaints by BBL, and 311 at the address in the last 12 months. Long Island shows an honest "not covered here" card.
6. **Haunt Meter** (0 to 100, with coverage "checked 4 of 5"), clearly labeled "not a safety rating".
7. English/Spanish toggle. Every card cites its source with a link and a "data as of" date.
8. A deployed public URL, and a public GitHub repo with an MIT license and `/.kiro` committed.
9. Property-based tests for the meter and the address normalizer, written through Kiro specs.

## Stretch goals (only if ahead; in this order)

1. **Ghost narrator**: an LLM writes a 3 to 5 sentence spooky summary of the cards in English and Spanish, with a template fallback and a number check.
2. **Long Island extra ghosts**: FEMA National Risk Index ratings and NYS DEC remediation sites nearby (both verified Nov 13 and Nov 17).
3. A print and share view (a "haunt report" one-pager).
4. A small map (point plus flood zone outline).
5. Your own MCP server that exposes the adapters.
6. The Kiro CLI, only if you choose to install WSL (admin rights and a restart). Skipped by default.

Parked: NYS spill records (ZIP only, no coordinates), dollar damage figures and storm narratives (never shown), and map tiles that need a paid key.

## Calendar

| When | Goal | Done when |
|---|---|---|
| **Tue Nov 17 (today)** | Set up (see `NEXT-STEPS.md`) | Joined the hackathon; Devpost name matches your ID; credit request sent (skip if you sent it on Nov 13); Git for Windows installed; repo created and first push done; folder open in Kiro; 7 steering files visible; warm-up question asked |
| Wed Nov 18 | Spec 1 (data adapters): requirements, Review, Analyze Requirements | `requirements.md` approved (still on Free credits) |
| Thu Nov 19 | Spec 1 design (keep the properties) and tasks (property tests required). Redeem the credit code when it arrives. Run task 1 (skeleton and scripts) alone. Create hook 1 | Design and tasks approved; the project builds |
| Fri Nov 20 | Spec 1 tasks one at a time: normalizer + property test, Census proxy, service area. Create hook 2 and watch it fire | `npm test` green; diary line |
| Sat Nov 21 | Run the rest of Spec 1: FEMA, NOAA script, GeoSearch guard, Socrata adapters, retries, fixtures. `npm run smoke` once. Optional: a skeleton deploy to Cloudflare (your clicks, see `NEXT-STEPS.md`), to check that the Census proxy works from Cloudflare | `npm run data:storms` writes 7 county files plus `baseline.json`; smoke passes or failures are understood |
| Sun Nov 22 | Spec 2 (Haunt Meter) end to end with property-based tests. Then Spec 3 requirements (type `#demo-addresses` first) | Kiro's PBT runs; any counterexample recorded |
| Mon Nov 23 | Spec 3 design and tasks approved. Install the Context7 power | |
| Tue Nov 24 | Spec 3: i18n files and `check:copy`, then create hook 3. Cards render with mocked data | Hook 3 seen firing |
| Wed Nov 25 | Real data, the four states, Spanish | All 4 demo addresses work end to end locally |
| Thu Nov 26 (Thanksgiving) | Off (optional: read the Spanish copy) | |
| Fri Nov 27 | Mobile layout and Try again. Playwright MCP plus screenshots. Deploy (your clicks, only when you say go) | The public `pages.dev` URL works on your phone |
| Sat Nov 28 | Fix bugs from the phone test. End-to-end tests. Lighthouse accessibility 90 or higher. Create the ghost-auditor and run one audit. Hook 4 if credits allow. `bilingual.md` checklist | Auditor report saved in the diary |
| Sun Nov 29 | Design polish. Record all raw clips (the app on desktop and phone, plus the Kiro screens). Draft the blog post from the diary. Test the README run steps on a fresh clone | Clips saved |
| **Mon Nov 30** | **Feature freeze.** Bugs only. Fill README Setup, Scripts and Features; remove the narrator line if it wasn't built | |
| **Tue Dec 1** | Edit the video to under 3:00, add captions, upload to YouTube as Public. *(Credit request deadline is 12:00 PM PT = 3:00 PM ET today. It matters only if a request was declined and must be sent again.)* | Video URL saved |
| Wed Dec 2 | Publish the blog post (third-party platform) and the social post (starts with #KiroweenHackathon26) | Both URLs saved |
| Thu Dec 3 | Fill Devpost from `DEVPOST.md` with no brackets left. Repo checks (MIT in About, `/.kiro` present, README run steps). **Submit** | Devpost shows "Submitted"; you can still edit until the deadline |
| **Fri Dec 4** | Buffer: small fixes only. **Redeem the credit code by today if you haven't.** Deadline 2:00 PM PT / 5:00 PM ET. Afterwards: tag the submitted commit and stop pushing to main | |
| Dec 16 to Jan 29 | Judging. Keep the app and the repo up; don't change them | |

Behind schedule? Cut in this order: all stretch goals, then 311 (keep DOB and HPD), then HPD complaints (keep violations). Never cut source citations, the Spanish toggle or the "couldn't check" state.

## Risks and answers

| Risk | Answer |
|---|---|
| Kiro credits run out | Send the credit request today (Join hackathon first). Until the code arrives you are on Kiro Free: 50 credits a month, Free-tier models only, so use Auto. Spend Free credits on requirements and design; don't click Run all Tasks or add hook 4 until the Kiroween credits are redeemed. Agent-prompt hooks cost credits each time; shell-command hooks don't. |
| Git isn't usable outside the Claude app on this PC | Install Git for Windows (step 5 in `NEXT-STEPS.md`), or let Claude run the git steps for you after you say yes. |
| PowerShell blocks `npm` here | `tech.md` tells Kiro to type `npm.cmd`/`npx.cmd` in its terminal. Or change the execution policy yourself (your call). Hooks run through cmd.exe and are unaffected. |
| NYC Open Data returns 503 or is slow (seen Nov 13 and Nov 17) | One retry with backoff, an 8-second timeout, and a "couldn't check" card with a retry button. Unit tests use recorded fixtures, never the live API. |
| The Census geocoder has no CORS | The `/api/geocode` server function (no cache, no logs). Deploy a skeleton early to make sure it answers from Cloudflare. |
| GeoSearch mismatches non-NYC addresses (seen Nov 13) | NYC counties only; borough and house number must match; within 300 m; placeholder BINs rejected. |
| The Kiro CLI needs Windows 11, and WSL isn't installed | Skip the CLI. The IDE runs on Windows 10. |
| Wording slips into advice | The `wording-rules.md` steering, the `check:copy` hook and the ghost-auditor agent before each push. |
| The video runs long | Script at 2:45; rehearse once; cut the "how it works" part first. |
| Kiro use judged insufficient | Kiro writes all app code. Keep the diary, and commit often with clear messages. Disclose that the planning docs and draft configs were made with Claude on Nov 13 to 17, before any code existed. |
| The app breaks during judging (Dec 16 to Jan 29) | Don't push to main after Dec 4 (Cloudflare redeploys on every push). If a live source breaks the app, email the hackathon manager before changing anything. |
