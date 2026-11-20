# Data source check: Nov 13, 2026 (re-checked Nov 17)

**Nov 17 re-check:** an independent second pass re-ran every query below. Everything still works. It found these corrections, and they are applied in this file and in `.kiro/steering/data-sources.md`:
1. **NOAA `CZ_FIPS` is not zero-padded** (Bronx is `5`, not `005`). Compare as integers, or six of the seven counties lose their county-level storms.
2. **Zone storms repeat once per forecast zone.** Count one event per (`EPISODE_ID`, `EVENT_TYPE`, begin date) per county. In 2025, Suffolk had 34 zone rows but 14 distinct events.
3. **NOAA narratives can describe deaths and name victims.** The app keeps no narratives and no casualty or damage fields.
4. **GeoSearch distance:** 89 South St's building point is 197 m from its Census point, and 350 5th Ave's is 82 m. The guard is 300 m plus a house-number match. GeoSearch also returns **placeholder BINs** (`1000000` to `5000000`, for example `1000000` for 102 South St). Those BINs collect unrelated DOB records, so the dob card treats them as "no building ID".
5. **`date_entered` (DOB complaints) is text.** Use `date_entered::floating_timestamp > '...'`. A text comparison runs without error but miscounts (20 vs. the correct 4 for BIN 1015862).
6. **ArcGIS errors arrive as HTTP 200** with an `error` object. Treat them as failures, not as "no flood zone".
7. **ECB `severity` mixes two label sets** (`CLASS - 1/2/3` and `Hazardous`/`Non-Hazardous`, plus `Unknown`). Status values are `ACTIVE`, `RESOLVE` and `Unknown`.
8. **FEMA's CORS preflight allows only `Content-Type`.** Send `X-App-Token` only to the Socrata hosts.
9. **DEC remediation rows repeat per operable unit.** Count `distinct program_number`: Long Beach has 8 rows but 5 sites.
10. **HPD has 0 rows for 350 5th Ave** (a commercial building). That is expected.
11. The GeoSearch attribution link moved (see section 2).
12. dev.socrata.com does **not** say app tokens are public. It says to keep them over HTTPS, and that anyone who copies one uses your quota (see section 5).

These are live, read-only checks of every data source Haunted Address plans to use. They ran on Nov 13, 2026, before any app code existed, so the build starts from facts and not guesses. Every sample address is a **public building** (a village hall, a city hall, a landmark address), never a private home.

| Sample | Address | Why |
|---|---|---|
| NYC, Manhattan | 350 5th Ave, New York, NY 10118 | Well-known public address with many building records |
| NYC, waterfront | 89 South St, New York, NY 10038 (Census point -74.003858, 40.705874) | Inside a FEMA high-risk flood zone |
| Long Island, Nassau | Long Beach City Hall, 1 W Chester St, Long Beach, NY 11561 | Coastal, inside a FEMA high-risk flood zone |
| Long Island, Suffolk | Patchogue Village Hall, 14 Baker St, Patchogue, NY 11772 | Inland Suffolk |

"CORS" below means a web page on another site can call the source directly from the visitor's browser. When CORS is not open, the app needs a small server function in the middle.

## Results at a glance

| Source | Works for NYC | Works for Long Island | Browser CORS | Plan |
|---|---|---|---|---|
| Census Geocoder (address to map point, county, tract) | Yes | Yes | **No** (no `Access-Control-Allow-Origin`; JSONP works) | Call it through a server function `/api/geocode` |
| NYC Planning GeoSearch (BBL/BIN building IDs) | Yes | **No**: wrongly matched the Long Beach address to a Brooklyn building | Yes (`*`) | NYC counties only, with a distance check |
| FEMA NFHL flood zones (ArcGIS REST) | Yes | Yes | Yes (echoes the origin) | Call it from the browser |
| NOAA Storm Events (bulk CSV) | Yes (county and zone rows) | Yes (county and zone rows) | Yes (`*`), but 12.6 MB gzipped per year | Pre-build small JSON files per county |
| NYC Open Data DOB/HPD/311 (Socrata SoQL) | Yes | **No** (NYC only) | Yes (`*`) | Call it from the browser; retry on 503 |
| FEMA National Risk Index, tract ratings (ArcGIS) | Yes | Yes | Yes (`*`) | Stretch: extra Long Island ghost |
| NYS DEC Environmental Remediation Sites (Socrata) | Yes | Yes | Yes (`*`) | Stretch: extra Long Island ghost |
| NYS DEC Spill Incidents (Socrata) | ZIP only | ZIP only | Yes | **Parked**: no coordinates, ZIP-level only |
| Long Island building violations | n/a | **None found** | n/a | **Dropped**: Long Island shows a "not covered here" card |

No core source was dropped. The Census geocoder works, but it needs a proxy. The only coverage gap is building records on Long Island.

## 1. Census Geocoder

- Endpoint: `https://geocoding.geo.census.gov/geocoder/geographies/onelineaddress?address=...&benchmark=Public_AR_Current&vintage=Current_Current&layers=Counties,Census%20Tracts&format=json`
- 350 5th Ave: matched `350 5TH AVE, NEW YORK, NY, 10118`, x=-73.985076, y=40.747851, county GEOID `36061`, tract `36061007600`.
- Long Beach City Hall: matched, x=-73.665982, y=40.589448, county `36059` (Nassau), tract `36059416500`.
- Patchogue Village Hall: matched, x=-73.012742, y=40.761807, county `36103` (Suffolk), tract `36103159000`.
- CORS: a request with an `Origin` header got `Vary: Origin` but **no** `Access-Control-Allow-Origin`, so browsers block it. `format=jsonp&callback=cb` returns `/**/cb({...})`. Use a server proxy (cleaner than JSONP).
- Rate limits: none published for single-address lookups; batch geocoding takes files of up to 10,000 addresses (not needed here).
- Terms: U.S. Census Bureau public service. Show a non-endorsement line ("not endorsed by the U.S. Census Bureau").
- Gotchas: `addressMatches` can be empty or hold several matches, so let the user pick. `x` is longitude and `y` is latitude.

## 2. NYC Planning GeoSearch (for NYC building IDs)

- Endpoint: `https://geosearch.planninglabs.nyc/v2/search?text=350%205th%20Ave%20Manhattan&size=1`
- Returned BBL `1008350041` and BIN `1015862` in `features[0].properties.addendum.pad`, point -73.985656, 40.748441, borough Manhattan.
- CORS: `access-control-allow-origin: *`.
- **Problem found:** `1 W Chester St Long Beach NY` returned `1 CHESTER STREET, Brooklyn` (BBL 3034997501, placeholder BIN 3000000). GeoSearch only knows NYC and will force a match. Rules:
  - Call it only when the Census county is one of the five NYC counties.
  - Accept the match only if its borough matches that county, it lies within 300 m of the Census point (89 South St is 197 m off), and the house number matches.
  - Treat placeholder BINs (`1000000` to `5000000`) as "no building ID".
- Every GeoSearch result, good or bad, comes back with `match_type: fallback` and `confidence: 0.8`, so those fields can't tell a good match from a bad one.
- Attribution: https://geosearch.planninglabs.nyc/v2/attribution (the old http link is dead). Building IDs come from NYC Planning's PAD: https://www.nyc.gov/content/planning/pages/resources/datasets/pad

## 3. FEMA National Flood Hazard Layer (ArcGIS REST)

- Flood zones, layer 28: `https://hazards.fema.gov/arcgis/rest/services/public/NFHL/MapServer/28/query?geometry=LON,LAT&geometryType=esriGeometryPoint&inSR=4326&spatialRel=esriSpatialRelIntersects&outFields=FLD_ZONE,ZONE_SUBTY,SFHA_TF,STATIC_BFE,DFIRM_ID&returnGeometry=false&f=json`

| Sample | FLD_ZONE | ZONE_SUBTY | SFHA_TF | STATIC_BFE |
|---|---|---|---|---|
| 350 5th Ave | X | AREA OF MINIMAL FLOOD HAZARD | F | -9999 (none) |
| South Street Seaport point | AE | null | T | 11 |
| Long Beach City Hall | AE | null | T | 10 |
| Patchogue Village Hall | X | AREA OF MINIMAL FLOOD HAZARD | F | -9999 (none) |

- FIRM panel effective dates, layer 3 (`outFields=FIRM_PAN,EFF_DATE,PANEL_TYP`): NYC panel `3604970184F` effective **2007-09-05**, Nassau `36059C0308G` **2009-09-11**, Suffolk `36103C0694H` **2009-09-25**.
- Gotcha: the Seaport point also hit two New Jersey panels (`34017...`, `34013...`). Keep only panels whose `FIRM_PAN` starts with `36` (New York).
- CORS: `Access-Control-Allow-Origin` echoes the calling origin.
- Rate limits: none published. Cache results per point.
- Terms: FEMA public data. Not an official flood zone determination. Official maps: FEMA Flood Map Service Center, https://msc.fema.gov
- Note: NYC's effective maps date from 2007. FEMA has worked on newer NYC maps for years. The card says "map effective since [date]" and links to msc.fema.gov for the latest status, without guessing.

## 4. NOAA Storm Events Database (bulk CSV)

- Folder: `https://www.ncei.noaa.gov/pub/data/swdi/stormevents/csvfiles/`, with 77 yearly "details" files from 1950 to 2026. The newest were `d2024_c20260728`, `d2025_c20260819` and `d2026_c20261026`.
- The README says the current-year file is updated each month, and a new `c` date in the filename means new data.
- The 2025 file is 12.6 MB gzipped and 71 MB unzipped, with 72,360 rows nationally and 2,093 New York rows. That is too big for each visitor's browser, so a build-time script reduces it to small per-county JSON files.
- County rows (`CZ_TYPE=C`, `CZ_FIPS`, **not zero-padded in the CSV**): Bronx 5, Kings 47, New York 61, Queens 81, Richmond 85, Nassau 59, Suffolk 103. Compare as integers.
- Forecast-zone rows (`CZ_TYPE=Z`): 72 New York (Manhattan), 73 Bronx, 74 Richmond (Staten Is.), 75 Kings (Brooklyn), 176 Northern Queens, 178 Southern Queens, 177 Northern Nassau, 179 Southern Nassau, 78 Northwest Suffolk, 79 Northeast Suffolk, 80 Southwest Suffolk, 81 Southeast Suffolk. Winter weather, high wind and coastal flooding are mostly zone rows, so a county-only filter misses them.
- 2025 examples: Richmond had 15 Thunderstorm Wind events, Queens 9 Flash Flood, Suffolk 7 Thunderstorm Wind and 7 Flash Flood. A sample narrative: "ASOS site KISP and Islip Airport measured a 60 mph wind gust." (Narratives are not shown in the app; some describe deaths.)
- One storm appears once per forecast zone. Count distinct (`EPISODE_ID`, `EVENT_TYPE`, begin date) per county: in 2025, Suffolk had 34 zone rows but 14 events, Queens 11 rows but 7, and Nassau 13 rows but 9.
- CORS: `*` on the folder.
- Terms: NOAA/NCEI public data. Storms are recorded by **county or forecast zone, not by street**, and the card must say so.
- Rule: leave the `DAMAGE_PROPERTY` and `DAMAGE_CROPS` dollar figures out of the app.

## 5. NYC Open Data (Socrata SoQL)

The tests used BIN `1015862` and BBL `1008350041` (350 5th Ave).

| Dataset | ID | Test | Result | Updated |
|---|---|---|---|---|
| DOB Violations | `3h2n-5cm9` | `count(*) where bin='1015862'` | 691 | Nov 12, 2026 |
| DOB ECB Violations | `6bgk-3dad` | count by BIN | 241 (503 on the first try, OK on retry) | Nov 12, 2026 |
| DOB Complaints Received | `eabe-havv` | count by BIN | 142 (503 on the first try, OK on retry) | Nov 12, 2026 |
| HPD Housing Maintenance Code Violations | `wvxf-dwi5` | counts by `class` for one Brooklyn building | A 15, B 11, C 16, I 2 | Nov 12, 2026 |
| HPD Complaints and Problems | `ygpa-z7cr` | sample row | OK (503 on the first try) | Nov 12, 2026 |
| 311 Service Requests from 2020 to Present | `erm2-nwe9` | `count(*) where bbl='1008350041'` | 520 | Nov 13, 2026 |
| 311, `within_circle(location, lat, lon, 100)`, last 12 months, grouped by type | `erm2-nwe9` | top 5 | Noise - Commercial 45, Water Maintenance 30, Noise - Street/Sidewalk 27, Illegal Parking 26, Consumer Complaint 20 | |

- Verified fields:
  - DOB violations: `bin`, `issue_date` (YYYYMMDD text), `violation_type`, `violation_category`, `description`.
  - ECB: `bin`, `issue_date`, `severity`, `ecb_violation_status`, `violation_description`.
  - DOB complaints: `bin`, `date_entered` (MM/DD/YYYY text), `complaint_category`, `status`.
  - HPD violations: `bbl`, `bin`, `boroid`, `block`, `lot`, `class`, `violationstatus`, `currentstatus`, `inspectiondate`, `novdescription`.
  - HPD complaints: `bbl`, `received_date`, `major_category`, `minor_category`, `complaint_status`.
  - 311: `created_date`, `complaint_type`, `descriptor`, `status`, `bbl`, `location`.
- CORS: `Access-Control-Allow-Origin: *` on every dataset.
- Speed: 0.2 s to 23 s. One `$order=issue_date DESC` query on DOB violations took 23.6 s. Prefer `count(*)`/`$group` aggregates and small `$limit`s.
- Reliability: **3 of the first 10 calls returned HTTP 503 "Service unavailable"**, and each succeeded on retry. Retry once with backoff, then show "couldn't check". Never show "clear" after a failure.
- Rate limits (from dev.socrata.com/docs/app-tokens): without an app token, requests share a pool per IP address and may be throttled (HTTP 429). With a free app token, requests are not throttled unless abusive. Get a token from an NYC Open Data account.
- Token in the browser: the page says tokens are "not necessarily used for authentication" but should be sent over HTTPS, and anyone who copies one uses your quota. Sending it from the browser in `X-App-Token` (Socrata's CORS preflight allows it) makes it visible to visitors. That is an accepted trade-off for a free read-only token. Never put a secret token in the browser.
- `date_entered` in DOB complaints is text: filter with `date_entered::floating_timestamp`. ECB `severity` mixes `CLASS - 1/2/3` with `Hazardous`/`Non-Hazardous`/`Unknown`.
- HPD returns 0 rows for 350 5th Ave (commercial), as expected.
- Terms: NYC Open Data is free to use, provided "as is" under the nyc.gov Terms of Use. Show "not endorsed by the City of New York".
- 311 covers 2020 to the present only (the dataset's own title).

## 6. Long Island coverage (what really exists)

- Address, flood zone and storms: full coverage (sections 1, 3 and 4).
- **Building violations and complaints: no open dataset found** for Nassau or Suffolk. Each town and village keeps its own records. Searches on Socrata (all domains, including data.ny.gov) and ArcGIS Hub returned nothing usable. The app shows a calm "not covered here" card that explains where those records live (the local building department).
- Extra Long Island options that work:
  - **FEMA National Risk Index**, by point: `https://services.arcgis.com/XG15cJAlne2vxtgt/arcgis/rest/services/National_Risk_Index_Census_Tracts/FeatureServer/0/query?geometry=LON,LAT&geometryType=esriGeometryPoint&inSR=4326&spatialRel=esriSpatialRelIntersects&outFields=COUNTY,TRACTFIPS,RISK_RATNG,CFLD_RISKR,HRCN_RISKR,IFLD_RISKR,WNTW_RISKR,NRI_VER&returnGeometry=false&f=json`
    - Patchogue: overall Very Low; coastal flooding Relatively Moderate; hurricane Relatively Low; inland flooding Very Low; winter weather Relatively Moderate.
    - Long Beach: overall Relatively Moderate; coastal flooding Relatively High; inland flooding Relatively High; winter weather Relatively High.
    - Midtown: overall Relatively Low; coastal flooding No Rating.
    - Version: December 2025. Use the field `IFLD_RISKR` (inland flooding); `RFLD_RISKR` does not exist and makes the query fail. Query by point, not by tract ID. Show **rating words only**, never the expected-annual-loss dollar fields.
  - **NYS DEC Environmental Remediation Sites** (`https://data.ny.gov/resource/c6ci-rzpg.json`), with `within_circle(georeference, lat, lon, 800)`. Long Beach had 8 rows within 800 m, but only **5 distinct sites** by `program_number` (BCP 2, HW 2, VCP 1) at 2 properties, because rows repeat per operable unit. Patchogue had 0. Count `distinct program_number`. No field holds a link, so cite https://data.ny.gov/d/c6ci-rzpg. Updated Nov 12, 2026.
  - NYS DEC Spill Incidents (`u44d-k5fk`): 99 records in ZIP 11772 and 52,968 in Suffolk, but **no coordinates**, so it is parked.

## 7. The narrator (LLM)

Not a data source. It is optional and runs behind `/api/narrate` with a template fallback, so the demo never depends on it. See `.kiro/steering/wording-rules.md` for its limits.
