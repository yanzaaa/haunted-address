---
inclusion: always
---
# English and Spanish

## Rules
- Every visible string lives in `src/i18n/en.json` and `src/i18n/es.json`, with **identical key sets** and identical `{placeholders}`. `scripts/check-i18n.ts` and a property test enforce this.
- English is the default. The toggle sits in the header, says "Español" or "English", and remembers the choice in `localStorage` (only the language, never an address).
- Switching language changes `<html lang>` and re-renders the cards **without new network calls**. The facts are the same; only the words change.
- Dates and numbers use `Intl` with `en-US` or `es-US`. Example: "Nov 13, 2026" becomes "13 nov 2026".
- Spanish uses **usted**, neutral Latin American Spanish. Avoid Spain-only words. Keep the spooky names playful, and keep the bodies as plain as the English.
- Data values from sources stay as published, in English (complaint types, violation text). Show them in quotes with a short Spanish label. Example: Tipo de queja: "Noise - Commercial" (ruido comercial).
- Never machine-translate at runtime. All translations are written into `es.json` and reviewed by a human (Anthony) before the deadline.
- The narrator (stretch) writes Spanish directly from the card facts. It does not translate the English summary.

## Glossary (use these exact terms)

| English | Español |
|---|---|
| Haunt Meter | Medidor Embrujado |
| ghost | fantasma |
| haunt report | reporte embrujado |
| flood zone | zona de inundación |
| high-risk flood area | área de alto riesgo de inundación |
| FEMA's flood map | el mapa de inundación de FEMA |
| base flood elevation | elevación de inundación base |
| storm | tormenta |
| county | condado |
| building violation | infracción del edificio |
| complaint | queja |
| housing conditions | condiciones de la vivienda |
| 311 report | reporte al 311 |
| public record | registro público |
| source | fuente |
| data as of | datos al |
| found | encontrado |
| clear (no records found) | sin registros |
| not covered here | no cubierto aquí |
| couldn't check | no se pudo consultar |
| try again | intentar de nuevo |
| outside our area | fuera de nuestra área |
| for information only | solo informativo |

## Review checklist (Anthony, before the video)
- [ ] Read every Spanish card state for all 4 demo addresses.
- [ ] The disclaimer matches `wording-rules.md` word for word.
- [ ] No English left on the Spanish screen, except quoted source data.
