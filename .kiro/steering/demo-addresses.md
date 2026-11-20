---
inclusion: manual
---
# Demo and test addresses

Use these when writing fixtures, smoke tests, example chips and the demo video. Type `#demo-addresses` in chat to load this file. They are all **public buildings**. Never use a private home or a small landlord's building in tests, screenshots or the video. Show addresses only, never building names or logos (trademarks).

| Chip label | Address | Expected (checked Nov 13, 2026) |
|---|---|---|
| Midtown Manhattan | 350 5th Ave, New York, NY 10118 | County 36061. Flood zone X (minimal). GeoSearch BIN 1015862, BBL 1008350041, 82 m from the Census point. DOB violations 691, ECB 241, DOB complaints 142 (4 in the last 3 years). 311 at BBL: about 520 since 2020. **HPD: 0 rows** (a commercial building), so the HPD card shows "clear"; don't promise HPD data here. |
| Lower Manhattan waterfront | 89 South St, New York, NY 10038 | Census point -74.003858, 40.705874, county 36061. Flood zone AE (high risk, inside the SFHA), base flood elevation 11 ft on FEMA's map. NYC map effective 2007-09-05. GeoSearch BIN 1090548, BBL 1000730010, **197 m** from the Census point (it passes the 300 m guard). |
| Long Beach (Nassau) | 1 W Chester St, Long Beach, NY 11561 | County 36059, tract 36059416500. Flood zone AE, base flood elevation 10 ft, map effective 2009-09-11. Building ghosts: not covered. Stretch: NRI coastal flooding "Relatively High"; 5 DEC remediation site codes within 800 m (BCP 2, HW 2, VCP 1) at 2 properties (8 rows, because rows repeat per operable unit). |
| Patchogue (Suffolk) | 14 Baker St, Patchogue, NY 11772 | County 36103, tract 36103159000. Flood zone X (minimal), map effective 2009-09-25. Building ghosts: not covered. Stretch: NRI overall "Very Low"; 0 remediation sites within 800 m. |
| Outside the area | 1600 Pennsylvania Ave NW, Washington, DC 20500 | Census matches county 11001 (DC), outside the 7 counties, so the app says "outside our area" and calls nothing else. |
| Bad input | `asdf` | 0 matches, so the app shows "We couldn't find that address" plus tips. |

Counts change daily. Tests use recorded fixtures, not these numbers. The numbers are here so a human can tell if a live result looks wrong.
