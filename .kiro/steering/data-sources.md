---
inclusion: always
---
# Data sources: rules for every adapter

All endpoints below were checked live on Nov 13, 2026. Full results, sample responses and field names are in the verification log:

#[[file:docs/data-check-2026-11-13.md]]

## Service area
- Allowed county FIPS (from the Census geocoder `Counties` layer):
  - NYC: `36005` Bronx, `36047` Kings (Brooklyn), `36061` New York (Manhattan), `36081` Queens, `36085` Richmond (Staten Island).
  - Long Island: `36059` Nassau, `36103` Suffolk.
- Any other county: stop and show "outside our area" (EN/ES). No other source is called.

## Flow for one address
1. Normalize the text (`src/core/address.ts`).
2. `/api/geocode` calls the Census geocoder: `geographies/onelineaddress`, `benchmark=Public_AR_Current`, `vintage=Current_Current`, `layers=Counties,Census Tracts`. It returns matches with `{lon: x, lat: y, countyFips, tractGeoid, matchedAddress}`.
   - 0 matches: "We couldn't find that address" plus tips.
   - 2 or more matches: the user picks one.
3. Then, in parallel:
   - **flood**: FEMA NFHL layer 28 point query (zone) and layer 3 (FIRM panel effective date). Keep only panels whose `FIRM_PAN` starts with `36`.
   - **storms**: fetch `/data/storms/<countyFips>.json`.
   - If the county is in NYC: **NYC GeoSearch** for BBL/BIN, then **dob**, **hpd** and **311**.
   - If the county is Nassau or Suffolk: dob, hpd and 311 return `not_covered` with the reason key `reason.li_building_records`.
   - Stretch: **nri** (point query) and **remediation** (`within_circle`, 800 m).

## GeoSearch guard (bug seen Nov 13; distances re-checked Nov 17)
GeoSearch matched "1 W Chester St, Long Beach" to a **Brooklyn** building. Accept a GeoSearch result only if all of these hold:
- the Census county is one of the five NYC counties;
- its `borough` matches that county (borough names: Manhattan = 36061, Bronx = 36005, Brooklyn = 36047, Queens = 36081, Staten Island = 36085);
- its point is within **300 m** of the Census point. The Census geocoder interpolates along address ranges: on Nov 17, 350 5th Ave was 82 m off and 89 South St 197 m off, while the wrong Long Beach match was 22 km off;
- its house number equals the house number in the Census matched address.

Otherwise dob, hpd and 311 are `unavailable` with the reason key `reason.building_not_matched`. Never guess a building.

**Placeholder BINs:** if `addendum.pad.bin` is `1000000`, `2000000`, `3000000`, `4000000` or `5000000`, there is no real building ID. Those BINs collect hundreds of unrelated DOB records. The BIN-keyed dob card is then `unavailable` with `reason.building_not_matched`. The BBL-keyed hpd and 311 cards may still run.

## FEMA flood zone, in plain words

| FLD_ZONE / ZONE_SUBTY | Category | Plain English |
|---|---|---|
| `V`, `VE` | coastal high hazard | High-risk coastal flood area with wave action (about a 1% chance of flooding each year) |
| `A`, `AE`, `AH`, `AO`, `A99`, `AR` | high risk | High-risk flood area (about a 1% chance of flooding each year) |
| `X` + "0.2 PCT ANNUAL CHANCE FLOOD HAZARD" | moderate | Moderate flood area (about a 0.2% chance each year) |
| `X` + "AREA OF MINIMAL FLOOD HAZARD" | minimal | Minimal flood hazard on FEMA's map |
| `D` | unknown | FEMA has not studied the flood risk here, so it shows as unavailable, not as clear |
| no feature returned | not mapped | No FEMA flood zone at this point, so it shows as unavailable |

- `SFHA_TF = 'T'` means inside FEMA's Special Flood Hazard Area.
- `STATIC_BFE = -9999` means there is no base flood elevation. Show the base flood elevation (in feet) only when it is greater than 0, labeled "base flood elevation on FEMA's map".
- **ArcGIS errors arrive as HTTP 200.** A reply that contains an `error` object (FEMA NFHL and NRI do this) is a failure, not "no feature returned". Retry once, then show "couldn't check". Only a reply with `features: []` means not mapped.
- The card always says: "This is FEMA's map, not an official determination for your building," and links to https://msc.fema.gov.
- Show the map's effective date: NYC 2007-09-05, Nassau 2009-09-11, Suffolk 2009-09-25 at the sample points. Read the real date from layer 3; don't hard-code it.

## NOAA storms (build-time script)
- Source folder: `https://www.ncei.noaa.gov/pub/data/swdi/stormevents/csvfiles/`. Parse the folder listing and pick the newest `StormEvents_details-ftp_v1.0_dYYYY_cYYYYMMDD.csv.gz` for each of the **last 10 full years plus the current year**.
- **`CZ_FIPS` is not zero-padded in the CSV** (Bronx is `5`, not `005`). Parse it as an integer before comparing, for both county and zone rows. A string match on `005` silently drops every county except Suffolk.
- Keep New York rows (`STATE = "NEW YORK"`) where:
  - `CZ_TYPE = C` and `CZ_FIPS`, read as an integer, is one of 5, 47, 61, 81, 85, 59, 103 (county FIPS 36005, 36047, 36061, 36081, 36085, 36059, 36103), or
  - `CZ_TYPE = Z` and `CZ_FIPS`, read as an integer, maps to a county:

    | Zone(s) | County |
    |---|---|
    | 72 | 36061 |
    | 73 | 36005 |
    | 74 | 36085 |
    | 75 | 36047 |
    | 176, 178 | 36081 |
    | 177, 179 | 36059 |
    | 78, 79, 80, 81 | 36103 |

- **One storm, many zone rows.** NWS lists a zone-type storm (winter weather, high wind, coastal flood) once per forecast zone. Suffolk has 4 zones, and Queens and Nassau have 2 each. For zone rows, treat rows with the same (`EPISODE_ID`, `EVENT_TYPE`, begin date) as one event for that county: count it once in `countsByType`, `countsByYear` and `baseline.json`, and list it once in `recent`. County (`C`) rows are separate local reports and count once per row. Without this, Suffolk looks over twice as stormy as it is (2025: 34 zone rows, 14 distinct events).
- Each `public/data/storms/<countyFips>.json` contains: `countyFips`, `countyName`, `years: [from, to]`, `countsByType`, `countsByYear`, the `recent` 10 events (`date`, `type`, zone or county name), `sourceFiles` (with their `c` dates), `generatedAt`.
- Also write `public/data/storms/baseline.json`: the median per-year count across the 7 counties, used by the storm level.
- **Drop `DAMAGE_PROPERTY`, `DAMAGE_CROPS`, `EVENT_NARRATIVE`, `EPISODE_NARRATIVE`, and every `DEATHS_*` and `INJURIES_*` field.** No dollar figures, no narratives (some describe deaths and name victims), and no casualty counts. A spooky ghost card is no place for them.
- The card says the data is "for [County] County, not your exact street".

## NYC Open Data (Socrata SoQL)
- Base URL (SODA2): `https://data.cityofnewyork.us/resource/<id>.json`. All of these work without a token today. The newer SODA3 API (`/api/v3/views/<id>/query.json`) is not needed.
- Optional header `X-App-Token`, sent **only** to `data.cityofnewyork.us` and `data.ny.gov`. FEMA's CORS preflight allows only `Content-Type`, so a custom header on a FEMA call breaks it in the browser.
- Use aggregates (`$select=count(*)`, `$group`) and small `$limit`s. Never download more than 20 raw rows for display.

| Ghost | Dataset | Key | What to show |
|---|---|---|---|
| dob | DOB Violations `3h2n-5cm9` | `bin` | Total; count in the last 3 years; latest `issue_date` (YYYYMMDD text) |
| dob | DOB ECB Violations `6bgk-3dad` | `bin` | Open (`ACTIVE`) vs. resolved (`RESOLVE`) from `ecb_violation_status`. Counts by `severity`, which mixes two label sets (`CLASS - 1/2/3` and `Hazardous`/`Non-Hazardous`, plus `Unknown`); give each its own EN/ES label |
| dob | DOB Complaints Received `eabe-havv` | `bin` | Count in the last 3 years. `date_entered` is MM/DD/YYYY **text**: query with `date_entered::floating_timestamp > '<date 3 years ago>T00:00:00'`. Never compare it as text: that runs without error and gives a wrong count (20 instead of 4 for BIN 1015862 on Nov 17) |
| hpd | HPD Violations `wvxf-dwi5` | `bbl` | Open violations by `class` (A non-hazardous, B hazardous, C immediately hazardous, I informational); latest `inspectiondate` |
| hpd | HPD Complaints and Problems `ygpa-z7cr` | `bbl` | Last 3 years: top `major_category` values (for example HEAT/HOT WATER) |
| 311 | 311 Service Requests 2020 to Present `erm2-nwe9` | `bbl` | Last 12 months: top 5 `complaint_type` values with counts. Fallback: `within_circle(location, lat, lon, 75)`. The fallback can be slow (one took 21.7 s on Nov 17), so the BBL query stays the main path |

- The "as of" date comes from `https://data.cityofnewyork.us/api/views/<id>.json` (`rowsUpdatedAt`, Unix seconds). Cache it for 24 hours.
- Some queries take over 20 seconds (seen Nov 13). Avoid `$order` on large unfiltered sets. Always filter by `bin` or `bbl` first.

## Stretch sources
- **FEMA NRI**: query the ArcGIS FeatureServer by point. Fields `COUNTY, TRACTFIPS, RISK_RATNG, CFLD_RISKR, HRCN_RISKR, IFLD_RISKR, WNTW_RISKR, NRI_VER`. Show rating words only. Never use expected-annual-loss dollar fields. `RFLD_RISKR` does not exist.
- **NYS DEC remediation sites**: `https://data.ny.gov/resource/c6ci-rzpg.json`. The dataset has **one row per operable unit or project**, not one per site, so count distinct sites:
  - Counts: `$select=program_type, count(distinct program_number) as sites&$where=within_circle(georeference, lat, lon, 800)&$group=program_type`
  - The 3 closest: `$select=program_number, program_facility_name, min(distance_in_meters(georeference, 'POINT(lon lat)')) as d&$where=within_circle(georeference, lat, lon, 800)&$group=program_number, program_facility_name&$order=d&$limit=3`
  - The class field is `siteclass`. No field holds a link, so cite https://data.ny.gov/d/c6ci-rzpg.

## Citations (every card)
`sources.ts` holds, for each source: agency, dataset name, dataset URL, terms URL and coverage. Each card shows: "Source: [agency], [dataset] · data as of [date] · [link]".

The footer adds: "Not endorsed by FEMA, NOAA, the U.S. Census Bureau, the City of New York or New York State."
