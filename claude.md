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
Lägg till backup via export/import av all appdata. Vi är kvar på lokala spåret
(ingen backend, ingen konto). Följ arkitekturen i CLAUDE.md: all lagring går via
store-wrappern, allt i index.html, vanilla JS, svenskt UI.

## Format
Använd ett versionsmärkt JSON-format så det håller för framtida sync/delning:

{
  "app": "lopbeloning",
  "version": 1,
  "exportedAt": "<ISO-datum>",
  "data": { settings, runs, consumed, achievements }
}

Lägg en migrate(parsed)-funktion som kan uppgradera äldre versioner till nuvarande.
Nu finns bara version 1, men strukturera så framtida versioner slottar in.

## Export
- Serialisera nuvarande state till formatet ovan.
- Filnamn: lopbeloning-backup-YYYY-MM-DD.json
- Ladda ner via Blob + URL.createObjectURL + <a download> + revokeObjectURL.
- TWA-safe fallback: om Web Share API med filer finns (navigator.canShare med
  files), erbjud "Dela" som alternativ. Sista fallback: visa JSON i en textarea
  med en "Kopiera"-knapp. Testa att nedladdning funkar både i vanlig Chrome och
  i en TWA-context.

## Import
- <input type="file" accept="application/json,.json">, läs med FileReader.
- Validera: rätt "app", känd version, och att data har förväntad form (runs är
  array osv). Kör migrate() vid behov.
- Innan overwrite: bekräftelse-modal som säger vad som ersätts, t.ex.
  "Detta ersätter din nuvarande data (X pass, Y mål). Fortsätt?"
- Vid OK: skriv till store, spara, re-rendera alla flikar.

## Felhantering (i gränssnittets röst, inte ursäktande)
- Fel filtyp / trasig JSON: "Filen gick inte att läsa. Välj en
  lopbeloning-backupfil (.json)."
- Fel app/version: "Den här filen kommer inte från Löpbelöning."
- Aldrig krascha eller nolla befintlig data på ett misslyckat import.

## UI
Två knappar i fliken Inställningar: "Exportera data" och "Importera data".
Håll stilen konsekvent med resten (samma btn-klasser, samma modal-mönster).

## Klart när
- Export laddar ner en giltig JSON-fil med all state.
- Import av den filen på en "tom" enhet återställer allt exakt (pass, mål,
  inställningar, druckna öl).
- Trasig eller främmande fil ger vänligt fel och rör inte befintlig data.
- Fungerar i TWA (nedladdning eller delning), inte bara i desktop-Chrome.
Lägg till en historik- och diagramvy. Följ CLAUDE.md: allt i index.html,
vanilla JS, inga externa bibliotek (rita diagram med inline-SVG, inget chart-lib),
svenskt UI, decimaltecken komma via toLocaleString('sv-SE').

## Ny flik: Historik
Lägg till en femte flik "Historik" i bottennav (välj en passande SVG-ikon i
samma stil som övriga). Den ska innehålla:

## Veckoaggregering
Skriv en funktion som grupperar runs per vecka (måndag som veckostart, återanvänd
mondayKey-logiken som redan finns). Per vecka: summa km, summa kcal, antal pass,
netto intjänade öl, och om veckokravet (needPerWeek) nåddes. Sortera senaste först.

## Innehåll i vyn
- Topprad med nyckeltal: nuvarande streak, längsta streak någonsin, totalt antal
  pass, total distans.
- Ett stapeldiagram (inline-SVG) med en stapel per vecka de senaste ~12 veckorna.
  Låt användaren växla vad stapeln visar: km / kcal / öl (enkel toggle-rad).
  Veckor som nådde kravet färgas grönt, missade veckor rött (återanvänd
  --green/--red). Hovra/tryck på en stapel visar värdet.
- Under diagrammet: en enkel lista per vecka (veckonr + datumspann, km, pass,
  öl, en liten prick grön/röd för krav uppnått).

## Tomt läge
Om inga pass finns: vänlig tom-vy i gränssnittets röst som pekar mot Logga-fliken.

## Krav
- Respektera prefers-reduced-motion (ingen animation om satt).
- Diagrammet ska skala snyggt på mobil (max-bredd 520px som resten).
- Rör inte befintlig lagring eller logik i andra flikar.

## Klart när
- Historik-fliken visar korrekta veckosummor mot loggade pass.
- Toggeln växlar km/kcal/öl och staplarna uppdateras.
- Grön/röd färg matchar om veckokravet nåddes.
- Streak och längsta streak stämmer med Översikt.
Paketera appen som en Trusted Web Activity (TWA) för Google Play. Appen ligger
på https://vesterlund97.github.io/lopbeloning/ (HTTPS via GitHub Pages).
Jag kör Linux Mint. Följ CLAUDE.md.

## Steg A – förberedelser i repot (du gör detta)
1. Kontrollera/skapa manifest.json med giltiga fält för TWA:
   name, short_name, start_url ("."), scope ("."), display "standalone",
   theme_color "#0d0b07", background_color "#0d0b07", lang "sv",
   och icons med minst 192x192 och 512x512, varav en maskable.
   Generera ikonerna om de saknas (enkel öl/löp-ikon i appens färger räcker).
   Länka manifestet i index.html (<link rel="manifest" href="manifest.json">).
2. Verifiera att en service worker registreras och ger offline (cache-first på
   appens skal). Skapa en om den saknas.
3. Kör en Lighthouse-audit (PWA + prestanda) och åtgärda tills PWA-kraven är gröna
   och prestanda >= 80 — det är Google Plays tröskel för TWA.

## Steg B – generera Android-projektet
4. Använd Bubblewrap (Googles officiella TWA-verktyg).
   - Lista exakt vilka förkrav jag behöver installera på Linux Mint (Node, JDK,
     Android SDK/cmdline-tools) med kommandon.
   - Kör `bubblewrap init --manifest https://vesterlund97.github.io/lopbeloning/manifest.json`
     och gå igenom valen (applicationId t.ex. io.github.vesterlund97.lopbeloning,
     signeringsnyckel osv). Förklara varje val.
   - Kör `bubblewrap build` för att få en signerad .aab.
   - VIKTIGT: förklara tydligt att signeringsnyckeln måste sparas säkert –
     tappar jag den kan jag aldrig uppdatera appen.

## Steg C – Digital Asset Links
5. Skapa .well-known/assetlinks.json i repot så länken verifieras och webb-URL:en
   döljs i appen. Visa var jag hämtar appens SHA-256-fingeravtryck från
   signeringsnyckeln och fyll i det. Bekräfta att GitHub Pages serverar
   /.well-known/assetlinks.json korrekt.

## Vad jag gör manuellt (lista bara stegen, du kan inte göra dem)
- Skapa Google Play-utvecklarkonto (engångsavgift ~25 USD).
- Ladda upp .aab i Play Console, fylla i butikstext, ikon, skärmdumpar.
- Svara på innehålls-/åldersformuläret – OBS: appen har öl/whisky som
  belöningstema, så flagga alkoholreferenser ärligt (kan ge högre åldersrating).
- Lägga in en integritetspolicy (poäng: all data är lokal, inget skickas någonstans).

## Klart när
- `bubblewrap build` producerar en signerad .aab utan fel.
- assetlinks.json ligger rätt och verifieras.
- Lighthouse PWA grön, prestanda >= 80.
- Jag har en tydlig manuell checklista för själva Play Console-uppladdningen.