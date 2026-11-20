# Haunted Address 👻

**What do public records say about a place before you sign?**

Type any New York City or Long Island address and get a "haunt report": one ghost card per public record about that place (flood zone, past storms, building violations and complaints). Each card is in plain English or Spanish and links to its public source.

Built for **Kiroween 2026** (Frankenstein category) with [Kiro](https://kiro.dev). Kiro writes the app code from the specs in `.kiro/specs/` (added as the build goes), guided by the steering files in [`.kiro/steering/`](.kiro/steering/).

> **Status:** in development during the Kiroween 2026 submission period (Oct 30 to Dec 4, 2026). The sections marked *(to be filled)* are completed as the build goes.

## Who it's for
Renters, home buyers, small landlords and small business owners checking an apartment, house, building or storefront before they sign.

## The ghosts (planned)

| Ghost | What it shows | Source | NYC | Long Island |
|---|---|---|---|---|
| 🌊 The Drowned Ghost | FEMA flood zone and the map's effective date | FEMA National Flood Hazard Layer | ✓ | ✓ |
| ⛈️ The Storm Wraith | Storms recorded in the county over the last 10 years | NOAA Storm Events Database | ✓ | ✓ |
| 🏚️ The Building Poltergeist | Building violations and complaints | NYC Department of Buildings (NYC Open Data) | ✓ | not covered |
| 🔥 The Cold Radiator Ghost | Housing violations and complaints | NYC HPD (NYC Open Data) | ✓ | not covered |
| 📞 The Whispering Neighbors | 311 reports at the address, last 12 months | NYC 311 (NYC Open Data) | ✓ | not covered |

On Long Island, building records are kept by each town or village, and no open dataset covers them, so the app says "not covered here" instead of guessing. Every source was checked live before building started (Nov 13 and Nov 17, 2026): see [docs/data-check-2026-11-13.md](docs/data-check-2026-11-13.md).

The **Haunt Meter** (0 to 100) is just for fun. It counts public records; it is not a safety rating.

## Why "Frankenstein"
One address gets stitched across four API styles that were never meant to meet:
- **ArcGIS REST** (FEMA flood zones)
- **Socrata SoQL** (NYC Open Data)
- a **bulk CSV archive** (NOAA, reduced at build time to small JSON files)
- the **Census geocoder** (through a tiny server proxy, because it has no browser CORS)
- plus, if it gets built, an **LLM narrator** that may only repeat facts already on the cards.

## Features *(to be filled)*

## Setup *(to be filled)*

## Scripts *(to be filled)*

## How Kiro builds it
- **Specs:** `.kiro/specs/`, Requirements-First: data adapters, Haunt Meter, UI.
- **Steering:** [`.kiro/steering/`](.kiro/steering/): product, tech, structure, data sources, wording rules, bilingual, demo addresses.
- **Hooks:** `.kiro/hooks/`.
- **Custom agent:** `.kiro/agents/`.
- **Diary:** [docs/KIRO-DIARY.md](docs/KIRO-DIARY.md).

The planning documents, the data-source checks, the first steering drafts and the draft agent and MCP configs were prepared with Claude (Anthropic's assistant) on Nov 13 to 17, 2026, before any code was written. The app code is written in Kiro.

## Data sources and attribution
- FEMA National Flood Hazard Layer: https://hazards.fema.gov/ (official maps: https://msc.fema.gov)
- NOAA NCEI Storm Events Database: https://www.ncdc.noaa.gov/stormevents/
- U.S. Census Bureau Geocoder: https://geocoding.geo.census.gov/
- NYC Open Data (DOB, HPD, 311): https://opendata.cityofnewyork.us/
- NYC Planning GeoSearch: https://geosearch.planninglabs.nyc/ ([attribution](https://geosearch.planninglabs.nyc/v2/attribution)), with building IDs from NYC Planning's [PAD](https://www.nyc.gov/content/planning/pages/resources/datasets/pad)

## Disclaimer
Haunted Address is for information only. It is not insurance, legal, real estate or financial advice, and it is not an official flood zone determination. Data comes from public sources that can be incomplete or out of date; check each source. Not endorsed by FEMA, NOAA, the U.S. Census Bureau, the City of New York or New York State.

## Privacy
No accounts, and no record of the addresses you type: nothing is stored, cached on a server or logged. To look an address up, the app sends it to the U.S. Census Bureau geocoder and, for NYC addresses, to NYC Planning GeoSearch.

## License
[MIT](LICENSE) © 2026 Anthony Yanza Tomala
