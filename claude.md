# Löpbelöning

En liten webbapp (PWA) som motiverar till att springa varje vecka genom att
"tjäna" öl av förbrända kalorier. Byggd i samma anda som `elpriskollen`.
UI:t är på svenska. Ägare: Elias (vesterlund97 på GitHub).

## Så fungerar systemet (produktlogik)

Grundmetafor: en **bar-nota**. Du tjänar öl genom att springa och går i skuld
när du missar veckor. Allt nedan är justerbart av användaren inne i appen —
inget är hårdkodat, defaultvärdena är bara startläget.

- **Öl-saldo** = intjänat − druckna öl − straff
  - Intjänat = totala kcal ÷ `kcalPerBeer`
  - Druckna = användaren trycker "Drick 1 öl" (räknas ned)
  - Straff = antal missade veckor × `penaltyBeer`
- **Veckokrav**: minst `needPerWeek` pass mån–sön, annars räknas veckan som
  missad. Måndag är veckostart. Nuvarande (pågående) vecka straffas inte —
  bara avslutade veckor från och med första loggade passets vecka.
- **Streak**: antal veckor i rad (bakåt från nuvarande vecka) som når kravet.
  En missad avslutad vecka bryter streaken.
- **Bank-cap**: `bankCap` = max sparade öl. 0 = ingen gräns. Saldot får gå
  minus (skuld), men det drickbara saldot cappas uppåt.

### Defaultvärden (Inställningar)
- `kcalPerBeer`: 250  (medvetet högre än en 50:a på ~210 kcal → man "överbetalar" i svett)
- `needPerWeek`: 1
- `penaltyBeer`: 2   (missad vecka = −2 öl)
- `bankCap`: 3
- `unit`: "öl"  (kan döpas om, t.ex. till annan belöningsenhet)

### Mål / achievements (helt användardefinierade)
Skapas/redigeras/tas bort i fliken **Mål**. Varje mål: ikon, namn, belöning,
villkor, tröskel, och flagga för repeterbart. Villkorstyper:
- `run_km`    – distans i ett pass (km)      [kan vara repeterbart]
- `run_kcal`  – kalorier i ett pass (kcal)    [kan vara repeterbart]
- `streak`    – veckor i rad
- `total_km`  – total distans
- `total_runs`– antal pass totalt
- `total_kcal`– totala kalorier

Mål låser upp sig själva automatiskt när ett pass eller totalstatistik når
tröskeln. Repeterbara mål räknar antal gånger (visas som ×N). Seedade default-mål:
Specialöl (5 km, repeterbar), Whiskyglas (10 km), Momentum (3 v streak),
Hundralappen (100 km total).

## Teknik / arkitektur

- **En enda fil**, `index.html` (döp om `lopbeloning.html` → `index.html` för
  GitHub Pages). Vanilla JS, inga byggsteg, inga npm-beroenden.
- Typsnitt via Google Fonts: Barlow Condensed (rubriker/siffror),
  Inter (brödtext), Spline Sans Mono (ledger-siffror).
- **Lagring**: liten wrapper (`store.get`/`store.set`) som använder
  `window.storage` när appen körs i Claudes artifact-miljö, annars
  `localStorage` (t.ex. på GitHub Pages). Samma fil funkar på båda ställena.
- All state ligger i ett objekt under nyckeln `lopbeloning_v1`:
  `{ settings, runs, consumed, achievements }`.
- Fyra flikar (bottennav): Översikt, Logga, Mål, Inställningar.
- Signaturelement: ölglas-SVG som fylls efter saldo/bank-cap, plus en
  receipt-liknande "nota" som bryter ned saldot rad för rad.
- Respekterar `prefers-reduced-motion`.

## Att göra härnäst (backlog)

1. **Gör om till installningsbar PWA**: `manifest.json` + service worker
   (cache-first för offline). Ikoner. `<link rel="manifest">` i index.html.
2. **Deploya till GitHub Pages** som `vesterlund97.github.io/lopbeloning/`
   (repo → Settings → Pages → main branch).
3. **Samsung Health-import**: låt användaren slippa knappa in km/kcal manuellt.
   Undersök export-format (troligen CSV/JSON via Samsung Health-export).
4. Ev. redigera loggade pass (inte bara ta bort), och en enkel historik-vy /
   diagram över veckor.

## Konventioner

- Svenska i UI:t. Decimaltecken = komma (`toLocaleString('sv-SE')`).
- Håll allt i en fil så länge det går; enkelhet > ramverk.
- Ändra inte defaultvärdena i logiken — de ska kunna ändras av användaren.