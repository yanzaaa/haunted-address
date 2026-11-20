---
inclusion: always
---
# Wording rules (English and Spanish)

Haunted Address explains public records. It never advises. These rules apply to every string in `src/i18n/*.json`, every card template, the narrator's prompts and outputs, the README, and the alt text.

## Never write (either language)
- Insurance quotes, prices, premiums, rates, deductibles or "how much you'll pay". **No dollar amounts at all.**
- Coverage advice: "you need flood insurance", "get covered", "you should buy", "protect yourself with a policy".
  - Spanish: "necesita seguro", "compre una póliza", "obtenga cobertura".
- Savings or bargain claims: "save", "lowest", "cheapest", "best rate", "discount", "free quote".
  - Spanish: "ahorre", "el más barato", "la mejor tarifa", "cotización gratis".
- The words "broker", "brokerage", "insurance agent", "insurance agency", "quote", "cotización", "corredor", "agente de seguros".
- Verdicts on the place: "safe", "unsafe", "dangerous", "deadly", "avoid this building", "don't rent here", "good deal".
  - Spanish: "seguro" meaning safe, "peligroso", "mortal", "evite".
- Guarantees: "guaranteed", "certified", "official determination" (except to say this is *not* one), "100%".
- Implied government endorsement: "FEMA says you should…", "approved by NYC", agency logos.
- Personal judgments about owners or landlords ("bad landlord", "slumlord"). Describe records, not people.
- Fear tactics or urgency: "act now", "before it's too late".

## Always do
- **Cite the source on every card**: "Source: [agency], [dataset] · data as of [date]" plus a link. Spanish: "Fuente: [agencia], [conjunto de datos] · datos al [fecha]".
- **Say what a record means and what it doesn't.** Example: "A violation is a record that an inspector found a problem. It may already be fixed. Check its status at the link."
- For storms: "These storms were recorded for [County] County, not your exact street."
- For flood: "This is FEMA's flood map, not an official determination for your building. Official maps: msc.fema.gov."
- Show the four states honestly: found / clear / not covered / couldn't check. "Clear" means "this source shows no records here as of [date]", never "safe".
- Neutral next step, at most one per card, pointing to the public source or a general professional. Example: "To learn more, see the full record at the source, or ask a licensed professional you trust."

## Tone
- Spooky titles, calm bodies. The spooky name decorates the card; the plain title and the body carry the facts.
- Sentences of 20 words or fewer. Active voice. No jargon without a short explanation (BBL, BIN, SFHA).
- Humor never mocks a place, an owner or a neighborhood.

## Required disclaimer (footer, both languages, word for word)
- EN: "Haunted Address is for information only. It is not insurance, legal, real estate or financial advice, and it is not an official flood zone determination. Data comes from public sources that can be incomplete or out of date; check each source. Not endorsed by FEMA, NOAA, the U.S. Census Bureau, the City of New York or New York State."
- ES: "Haunted Address es solo informativo. No es asesoría de seguros, legal, inmobiliaria ni financiera, y no es una determinación oficial de zona de inundación. Los datos vienen de fuentes públicas que pueden estar incompletas o desactualizadas; verifique cada fuente. No está respaldado por FEMA, NOAA, la Oficina del Censo de EE. UU., la Ciudad de Nueva York ni el Estado de Nueva York."

Next to the Haunt Meter (both languages):
- EN: "Just for fun: this counts public records. It is not a safety rating."
- ES: "Solo por diversión: esto cuenta registros públicos. No es una calificación de seguridad."

## Narrator (LLM) limits (stretch)
- Input: only the card JSON (facts, states, sources). Never the raw address string.
- Output: 3 to 5 sentences in the requested language, spooky in tone, facts unchanged.
- It may only use numbers and dates that appear in the card facts. A validator rejects any other number.
- The output must pass `check:wording`. If it fails or times out, show the deterministic template summary instead.
- Label it: "AI-written summary of the cards above" / "Resumen escrito por IA de las tarjetas de arriba".

## Machine check
`scripts/check-wording.ts` scans everything a visitor can read in the app (`src/i18n/*.json` and `src/narrate/**`) for the banned terms above (case-insensitive, **whole words only**, so "safety" does not match "safe"). The README is reviewed by the ghost-auditor agent instead, because it talks about Kiro "agents". The script allows these approved exceptions:
- the Haunt Meter lines ("Just for fun: this counts public records. It is not a safety rating." / "Solo por diversión: esto cuenta registros públicos. No es una calificación de seguridad.")
- "not an official determination"
- "no es una determinación oficial"
- the disclaimer sentences
- "seguro" in the sense of insurance inside the disclaimer only

It exits non-zero and lists file, line and term.
