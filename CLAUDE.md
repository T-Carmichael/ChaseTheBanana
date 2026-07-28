# Chase The Banana 🍌

Løbetræningsapp til en løbeklub, bygget på medlemmernes egne intervals.icu/Garmin-data. Genererer periodiserede ugeplaner, zone-baserede pas, et begynder-spor og en ernæringsguide — og kan sende/hente data direkte til/fra intervals.icu.

## Arkitektur — vigtigt at forstå først
Dette er **ingen build-pipeline, ingen npm, ingen framework**. Det er tre statiske filer:
- `index.html` — hele appen. Al HTML, CSS og JavaScript ligger i denne ene fil (bevidst, for enkel deploy uden build-trin).
- `manifest.json` — PWA-manifest (navn, ikoner, farver, standalone-display).
- `sw.js` — service worker. **Netværk-først, cache kun som offline-fallback** (var tidligere cache-først, hvilket forårsagede at opdateringer ikke slog igennem — ret ikke dette tilbage).
- `icon-*.png` — app-ikoner (banan + løbesko-motiv, genereret med PIL, ligger som binære filer).

Ingen bundler, ingen transpiler. Test af JS-syntaks: `node --check` på det udtrukne script-indhold (se "Test" nedenfor).

## Kritisk convention: bump CACHE_NAME ved enhver ændring
Hver gang `index.html`, `manifest.json` eller `sw.js` ændres, **skal** `CACHE_NAME` i `sw.js` bumpes (fx `hm-program-v14` → `v15`). Uden det bliver allerede installerede PWA'er hængende på gammelt indhold hos brugerne. Glem aldrig dette trin.

## Nøglekomponenter i index.html (i omtrentlig rækkefølge)
- **DATA / EMPTY_DATA** — al brugerdata (løb, zoner, fitness/trætheed) ligger i det globale `DATA`-objekt. `EMPTY_DATA` er en neutral standard uden nogens personlige tal (vigtigt: indeholder IKKE rigtige brugerdata — det blev bevidst renset for at gøre appen delbar).
- **storageGet/storageSet** — universel lagrings-wrapper: bruger `window.storage` hvis kørende som Claude-artifact, ellers `localStorage` (til den selvstændige PWA). Al persistens skal gå igennem disse, aldrig direkte `window.storage` eller `localStorage`.
- **GOAL / PROGRAM / SESSIONS_PER_WEEK** — brugerens mål (distance, dato, niveau, ugentlige pas), og den periodiserede programtidslinje beregnet derudfra.
- **ZONE_BOUNDS / paceForZone / zoneRangeStr** — tempo-zoner (Z1-Z7) ankret til brugerens tærskelfart, kalibreret til at matche intervals.icu's egne zonegrænser præcist.
- **buildEasySegs / buildTempoSegs / buildLongSegs** — genererer de ugentlige træningspas som segment-lister (bruges både til visning og til `intervals.icu`-eksport-tekst).
- **processCSVRows / processGarminRows** — parser hhv. intervals.icu's og Garmins egne CSV-eksportformater. `simulateCTLATLFromRuns` er en fallback, når kilden (fx API'et) ikke selv leverer Fitness/Trætheed.
- **setupApiExperiment / sendWorkoutToIntervals** — direkte API-integration mod intervals.icu (både læse aktiviteter og skrive planlagte pas). Athlete ID + API-nøgle gemmes lokalt, aldrig delt.
- **Guide-modal** (`#guideModal` i HTML) — den indbyggede brugerguide. Hold den i sync med `funktionsbeskrivelse.md`, hvis den findes i repoet, når indholdet ændres.

## Test / verificér ændringer
Der er ingen automatiske tests. Før enhver commit:
```bash
# Udtræk JS og tjek syntaks
python3 -c "
content = open('index.html').read()
script = content.split('<script>')[-1].split('</script>')[0]
open('/tmp/check.js','w').write(script)
"
node --check /tmp/check.js && echo "SYNTAX OK"
```
Tjek også at `<style>`-tags har lige mange `{` og `}` ved større CSS-ændringer (kopier CSS-blokken og tæl).

## Deploy
`git push` til `main` deployer automatisk via Netlify (forbundet direkte til dette repo — ingen manuel zip-upload). Husk CACHE_NAME-bump (se ovenfor) i samme commit.

## Ting der bevidst er som de er (ikke bugs)
- Ingen backend, ingen database — alt er client-side og gemmes lokalt pr. bruger/enhed.
- `EMPTY_DATA` skal forblive tom/neutral — tilføj aldrig rigtige brugerdata som standardværdier igen.
- Belastnings- og zone-estimater er bevidst forenklede tilnærmelser, ikke intervals.icu's fulde interne model — det er tydeligt kommunikeret til brugerne i appen.
