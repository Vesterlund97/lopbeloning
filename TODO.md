# Galopp – TODO

Löpande backlog. Kryssa av med `- [x]` när klart.

---

## Snabbfix (låg risk, litet ingrepp)

- [x] **Teckengräns på inmatningsfält**
  - `maxlength` satt på mål-namn (20) och mål-belöning (80), samt nya dryckesprofil-etiketten (20).
  - Visuell räknare ("18/20") wired via `wireCharCount()`.

- [x] **Cap på rutstorlek**
  - `overflow-wrap:anywhere` + `min-width:0` på `.tab-num`, `.stat .v`, `.led-row .r` så stora tal/etiketter wrappar istället för att svälla ut rutorna.
  - Testat med extremvärden (10 000+, långa etiketter) i Selenium.

---

## Samsung Health-import – avaktiverad, inte riven

- [x] Feature flag: `const ENABLE_SAMSUNG_HEALTH=false;`
- [x] Bara UI-ingången (knappen i Logga) gömd via flaggan.
- [x] Parse-/importlogiken ligger kvar orörd i koden.
- [x] Kommentar i koden om varför + datum (2026-07-23).

---

## Redigerbar dryckestyp (öl / vin / grogg)

**Beslut: Alt B – dryckesprofiler.** Klart.

- [x] Datamodell: `settings.drinkProfiles: [{id, label, icon, kcalPerUnit}]` + `settings.activeDrinkId`.
  `unit`/`kcalPerBeer` finns kvar i settings men är nu deriverade (synkas alltid från aktiv profil via `syncActiveDrink()`), så all befintlig läskod förblev oförändrad.
- [x] Default-profiler: öl (250 kcal), vin (150 kcal), grogg (100 kcal). Egna drycker läggs till via "+ Ny dryck" i Inställningar.
- [x] Export/import-schema fick versionsnummer 2 (`BACKUP_VERSION`), migreringskedja via `BACKUP_MIGRATIONS`.
- [x] Migrering av befintlig data: gammal anpassad `unit`/`kcalPerBeer` bevaras som den nya "öl"-profilens värden (inte tyst återställd till hårdkodade defaults) — verifierat med en handskriven v1-backup i test.

## Consumed → datumstämplade händelser (upptäckt under historik-arbetet)

- [x] `consumed` bytt från ett globalt tal till `[{id, date}]`, samma mönster som `runs`.
- [x] Migrering: gammalt tal N → N synteriserade poster daterade till migreringsdagen (totalen bevaras, men inte historiskt *när*-info för gammal data — okej, invänd och godkänt).
- [x] Kombinerad med dryckesprofil-migreringen i samma v2-steg.

---

## Historik

- [x] Tomt läge ("inga pass ännu") — fanns redan sedan tidigare.
- [x] Gruppering per vecka med delsummor (km, kcal, öl, pass, klar/missad) — fanns redan.
- [x] **Tydligare staplar**: gridlinjer + siffer-etiketter (värde per vecka, inte bara på tryck).
- [x] **Graf över intjänat vs spenderat över tid** — ny sektion under huvuddiagrammet, grön/röd stapelpar per vecka.
- [ ] Filtrering (tidsintervall, endast intjäning, endast uttag) — inte byggt än.
- [ ] Kronologisk lista med tydligare typografisk hierarki — oklart om detta fortfarande behövs utöver vecko-listan; stäm av vid behov.

---

## Backlog / senare

- [~] Google Play-publicering via TWA (Bubblewrap) — **pågår**: keystore skapad, `.aab` byggd, `assetlinks.json` live, innehållsformulär nästan klart. Väntar på sluten testning (Googles 20-testare/14-dagarskrav).
- [x] Åldersrating – alkoholtemat: IARC-formuläret ifyllt ärligt, landade på alla åldrar/PEGI 3 (marknadsför/säljer inte alkohol, bara tema).
- [ ] iOS-väg: Capacitor + Codemagic (500 gratis byggminuter/mån)
  - Kräver Apple Developer Program, $99/år
  - Måste passera riktlinje 4.2.2 – native-funktioner utöver ren webbwrapper
