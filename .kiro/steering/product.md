---
inclusion: always
---
# Product: Haunted Address

## One line
Type a New York City or Long Island address and get a "haunt report": one ghost card per public record about that place (flood zone, past storms, building violations and complaints). Each card is in plain English or Spanish and links to its public source.

## Who it is for
- Renters checking an apartment before signing a lease.
- Home buyers before making an offer.
- Small landlords checking a building they own or might buy.
- Small business owners checking a storefront or office before signing a commercial lease.

Write for a busy adult reading on a phone: plain words, short sentences, about an 8th-grade reading level. Many users read Spanish first.

## What it is not
- Informational only. Not insurance, legal, real estate, engineering or financial advice.
- No insurance quotes, prices, premiums or coverage recommendations. No "safe" or "unsafe" verdicts. No lead capture, sign-up or contact form.
- Not an official flood zone determination. FEMA's Flood Map Service Center (https://msc.fema.gov) is the official source.
- No agency branding and no call to action for any business. The footer credits the builder by name only.

## Hackathon context
- Kiroween 2026, **Frankenstein** category: it stitches together incompatible tech (ArcGIS REST from FEMA, Socrata SoQL from NYC Open Data, a bulk CSV archive from NOAA, the Census geocoder and, if built, an LLM narrator) into one calm, useful page.
- The app must stay online, free and without login until judging ends (**Jan 29, 2027, 1:00 PM PT**).
- Deadline: Friday, Dec 4, 2026, 2:00 PM Pacific. Judges score potential impact, Kiro use, design, innovation (use of public datasets) and presentation equally.
- Built by one person, Anthony Yanza Tomala, with Kiro writing the code.

## The ghosts (one card each)

| id | Spooky name (EN / ES) | Plain title (EN / ES) | Source | NYC | Long Island |
|---|---|---|---|---|---|
| `flood` | The Drowned Ghost / El Fantasma Ahogado | Flood zone / Zona de inundación | FEMA National Flood Hazard Layer | yes | yes |
| `storms` | The Storm Wraith / El Espectro de las Tormentas | Past storms nearby / Tormentas pasadas en la zona | NOAA Storm Events Database | yes (county) | yes (county) |
| `dob` | The Building Poltergeist / El Poltergeist del Edificio | Building violations and complaints / Infracciones y quejas del edificio | NYC Dept. of Buildings (NYC Open Data) | yes | not covered |
| `hpd` | The Cold Radiator Ghost / El Fantasma del Radiador Frío | Housing conditions / Condiciones de la vivienda | NYC HPD (NYC Open Data) | yes | not covered |
| `311` | The Whispering Neighbors / Los Vecinos que Susurran | 311 reports at this address / Reportes al 311 en esta dirección | NYC 311 (NYC Open Data) | yes | not covered |
| `nri` (stretch) | The Shadow Map / El Mapa de Sombras | Neighborhood hazard ratings / Calificación de riesgos del vecindario | FEMA National Risk Index | yes | yes |
| `remediation` (stretch) | The Buried Secret / El Secreto Enterrado | Cleanup sites nearby / Sitios de limpieza ambiental cercanos | NYS DEC (data.ny.gov) | yes | yes |

## Card states (always one of four)
1. **found**: the source has records here. Show the facts.
2. **clear**: the source was checked and has nothing here. Say what was checked and when.
3. **not covered**: the source does not cover this place (for example, NYC building records on Long Island). Say where such records live instead.
4. **couldn't check**: the source failed or timed out. Offer a retry. **Never show this as clear.**

## The Haunt Meter
A playful 0 to 100 summary of how much the public records turned up, shown with its coverage ("checked 4 of 5 ghosts"). It is **not** a safety rating, a property value signal or an insurance score, and the UI says so right next to it.

## Product principles
1. Every card cites its source: the agency, the dataset name, a link and the "data as of" date.
2. Unknown is never shown as safe.
3. Spooky is decoration. The facts are calm and plain. No fear words like "dangerous" or "deadly".
4. Bilingual from day one: English and Spanish, with the same facts.
5. Privacy: no accounts, no stored addresses, no server-side caching or logging of addresses, no analytics that record addresses, nothing stored beyond the language choice. To look an address up, the app sends it to the U.S. Census Bureau geocoder and, in NYC, to NYC Planning GeoSearch. The privacy note says so.
6. Accessible: WCAG 2.2 AA, full keyboard use, screen reader labels, respects reduced motion, works at 360 px wide.
