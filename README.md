# Chase The Banana 🍌

Løbetræningsapp bygget på egne intervals.icu/Garmin-data. Statisk PWA — ingen build-trin, ingen server, bare filer.

## Struktur
- `index.html` — hele appen (HTML/CSS/JS i én fil)
- `manifest.json` — PWA-manifest (navn, ikoner, farver)
- `sw.js` — service worker (offline-cache, netværk-først-strategi)
- `icon-*.png` — app-ikoner

## Kom i gang lokalt
Ingen afhængigheder at installere. Åbn `index.html` i en browser, eller kør en simpel lokal server:
```bash
python3 -m http.server 8080
```

## Sæt Git + Netlify op (én gang)

**1. Opret et GitHub-repo**
```bash
cd chasethebanana
git remote add origin https://github.com/<dit-brugernavn>/chasethebanana.git
git branch -M main
git push -u origin main
```
(Opret selve repoet på github.com først — "New repository", kald det fx `chasethebanana`, lad det være tomt uden README/gitignore, siden vi allerede har et lokalt repo.)

**2. Forbind Netlify til GitHub-repoet**
- Gå til [app.netlify.com](https://app.netlify.com) → dit eksisterende site (`chasethebanana`) → **Site configuration → Build & deploy → Link repository** (eller opret et nyt site via "Import an existing project" og vælg GitHub-repoet)
- Build command: (tom — ingen build-trin nødvendigt)
- Publish directory: `/` (roden af repoet)
- Netlify redeployer nu automatisk, hver gang der pushes til `main`

**3. Fremadrettede opdateringer**
Når appen skal ændres — enten manuelt eller via Claude Code lokalt:
```bash
git add -A
git commit -m "Beskrivelse af ændringen"
git push
```
Netlify bygger og deployer automatisk indenfor et minuts tid. Ingen manuel zip-upload nødvendig længere.

## Vigtigt ved fremtidige ændringer af sw.js
Bump `CACHE_NAME` (fx `v14` → `v15`) i `sw.js`, hver gang `index.html` ændres — ellers risikerer allerede installerede PWA'er at hænge fast på en ældre version, indtil cachen naturligt udløber.
