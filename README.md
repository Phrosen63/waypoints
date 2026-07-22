# Waypoints

Waypoints är innehållsrepot för **Waylight** — ett självbyggt, webbläsarbaserat verktyg för att förbereda och bläddra i TTRPG-kampanjer (Drakar och Demoner). Det här repot innehåller *bara* innehåll (`.md`- och `.yaml`-filer, samt bilder) — ingen kod. Waylight läser in filträdet direkt från GitHub och renderar allt i webbläsaren.

Den här filen dokumenterar hur innehållet ska struktureras och formateras för att Waylight ska tolka det korrekt.

---

## Innehållsförteckning

- [Filstruktur](#filstruktur)
- [Frontmatter](#frontmatter)
- [aventyr.yaml](#aventyryaml)
- [Länkning](#länkning)
- [Specialtaggar i brödtexten](#specialtaggar-i-brödtexten)
- [Bilder](#bilder)
- [Låssystemet ("master password")](#låssystemet-master-password)
- [Namnkrockar](#namnkrockar)
- [Cache och uppdatering](#cache-och-uppdatering)

---

## Filstruktur

```
waypoints/
├── regler/           (globala regelsidor)
├── monster/          (globala monster, kan ha undermappar som boss/, svaga/)
├── karaktarer/        (globala NPCs)
├── foremal/           (globala föremål)
├── bilder/            (bilder, undermappar matchar innehållstyp)
└── aventyr/
    └── <äventyrsnamn>/
        ├── aventyr.yaml       (namn + "filer"-index över allt innehåll i äventyret)
        ├── platser/
        ├── monster/
        ├── karaktarer/
        └── foremal/
```

**Globalt innehåll** (`regler/`, `monster/`, `karaktarer/`, `foremal/`) är alltid synligt för alla, oavsett låst/upplåst status — om inte en enskild fil har `confidential: true` i sin frontmatter (se [Låssystemet](#låssystemet-master-password)).

**Äventyrsinnehåll** (`aventyr/<namn>/...`) är *implicit konfidentiellt* — allt innehåll i en äventyrsmapp är dolt för oupplåsta användare, förutom `aventyr.yaml` själv, vars namn och filindex alltid är synligt (så att äventyret syns i trädet, om än låst).

Undermappar under `monster/`, `karaktarer/` etc. (både globalt och per äventyr) är fritt valbara och används för att gruppera innehåll visuellt i trädet (t.ex. `monster/boss/`, `monster/svaga/`). Namnet på undermappen blir rubriken i trädvyn.

---

## Frontmatter

Varje `.md`-fil ska inledas med YAML-frontmatter, omgärdad av `---`:

```yaml
---
type: monster
namn: Reva
länkar:
  regler: [styrkor_svagheter_och_element]
relaterat: [ryvok]
taggar: [monster, starka]
confidential: true
status: draft
toc: true
---
```

Allt efter den avslutande `---` tolkas som markdown-brödtext.

### Fält

| Fält | Typ | Obligatoriskt | Beskrivning |
|---|---|---|---|
| `type` | sträng | Nej (men rekommenderas) | Innehållstyp. Styr ikonen som visas i trädet och länkpanelen samt "typ-badgen" högst upp på sidan. Kända typer: `regel` (§), `monster` (☠), `karaktär` (☺), `plats` (⌂), `föremål` (◆). Okänd/saknad typ visar `•` som ikon. |
| `namn` | sträng | Nej | Visningsnamn för sidan. Om utelämnat används filnamnet (utan `.md`) istället, i tabbar, träd, länkchips och sidtitel. |
| `länkar` | objekt (kategori → lista) | Nej | Explicita, kategoriserade länkar till annat innehåll. Varje nyckel är en fri textkategori (t.ex. `regler`, `personer`, `platser`) och värdet är en lista med kortnamn (filnamn utan `.md`). Visas i länkpanelen till höger, grupperat per kategori. |
| `relaterat` | lista | Nej | En platt lista med kortnamn på relaterat innehåll, utan kategori. Visas i en egen sektion i länkpanelen. |
| `taggar` | lista | Nej | Fria etiketter för sökning/filtrering. Matchas mot filnamn och `namn` när man söker i trädet (sökrutan matchar mot filnamn + `namn` + `taggar` sammanslaget). |
| `confidential` | bool | Nej | Om `true`, döljs sidans innehåll bakom låssystemet tills upplåst (se nedan). Sätts oftast manuellt bara på enskilda **globala** filer (t.ex. en global boss som är tänkt som en spoiler) — allt inuti `aventyr/` är redan implicit konfidentiellt utan att detta behöver sättas. |
| `status` | sträng | Nej | Om satt till `draft`, visas en "✎ utkast"-badge bredvid typ-badgen högst upp på sidan. Rent visuellt, påverkar inget annat. |
| `toc` | bool | Nej | Om `true`, genereras automatiskt en innehållsförteckning ("Innehåll") överst i dokumentet, baserad på alla `##`-rubriker (H2) i brödtexten. Klickbara länkar som skrollar till rätt sektion. |

**Kortnamn**: när man refererar till en fil i `länkar`/`relaterat`, eller i en `[[wikilänk]]`, används alltid filnamnet utan `.md`-ändelse och utan sökväg — t.ex. `reva`, inte `monster/starka/reva.md`. Se [Länkning](#länkning) för hur upplösningen fungerar.

---

## aventyr.yaml

Varje äventyrsmapp under `aventyr/` måste innehålla en `aventyr.yaml` i sin rot. Den fyller två syften:

1. Ger äventyret ett **visningsnamn** i trädet (annars visas mappnamnet rakt av).
2. Fungerar som ett **filindex** — en lista över allt innehåll som finns i äventyret, oavsett om det redan är hämtat från GitHub eller inte. Det här är vad som gör att `resolveLink()` kan känna igen och peka mot innehåll i ett äventyr som ännu inte laddats (t.ex. ett annat, olåst äventyr som länkar till något i ett låst äventyr).

### Struktur

```yaml
namn: Dysterhamn
filer:
  - aventyr/dysterhamn/dysterhamn.md
  - aventyr/dysterhamn/platser/skuggkvarteren/skuggornas_grand.md
  - aventyr/dysterhamn/platser/skuggkvarteren/martas_gomstalle.md
  - aventyr/dysterhamn/monster/reva.md
  - aventyr/dysterhamn/karaktarer/ryvok.md
```

- `namn`: äventyrets visningsnamn i trädet.
- `filer`: en platt lista med **fullständiga sökvägar** (relativt repots rot) till alla `.md`-filer i äventyret. Måste hållas i synk manuellt med det faktiska filinnehållet — om en fil läggs till eller tas bort i äventyrsmappen, uppdatera listan här också.

`aventyr.yaml` hämtas alltid direkt (även om äventyret är låst) — det är vad som gör att äventyret syns (låst, kursiverat, med 🔒) i trädet innan man loggat in. Resten av äventyrets filer hämtas först när man låser upp *och* klickar sig in i äventyret (lat laddning, se [Cache och uppdatering](#cache-och-uppdatering)).

---

## Länkning

Det finns tre sätt att länka mellan sidor:

### 1. Wikilänkar i brödtexten

```markdown
Revan använder ofta [[dimridåer]] för att skapa förvirring.
```

`[[kortnamn]]` i löptexten görs om till en klickbar länk. Om målet inte kan hittas visas texten olänkad men markerad (`Länk saknas: ...` som tooltip).

### 2. `länkar` i frontmatter

```yaml
länkar:
  regler: [styrkor_svagheter_och_element, initiativ]
  personer: [ryvok]
```

Kategoriserade länkar, grupperade och visade i länkpanelen till höger.

### 3. `relaterat` i frontmatter

```yaml
relaterat: [ryvok, skuggornas_grand]
```

En okategoriserad lista, visas i en egen sektion i länkpanelen.

### Hur länkar löses upp (`resolveLink`)

Ett kortnamn (t.ex. `ryvok`) letas upp i denna ordning:

1. **Exakt sökväg** — om strängen redan är en fullständig, existerande filsökväg, används den direkt.
2. **Samma äventyr** — om den länkande filen ligger i `aventyr/<namn>/...`, letas först i just det äventyrets redan inlästa filer.
3. **Globala mappar** — `regler/`, `monster/`, `karaktarer/`, `foremal/` (i den ordningen), inklusive undermappar.
4. **Alla äventyrs filindex** — om inget hittats i redan inläst innehåll, söks i samtliga äventyrs `filer`-listor i `aventyr.yaml`. Om en träff hittas där, men äventyret ännu inte är hämtat/upplåst, returneras en **låst länk-descriptor** istället för en sökväg — detta är vad som ger 🔒-ikonen på wikilänkar/chips som pekar mot låst, ohämtat innehåll.
5. Hittas inget alls, betraktas länken som trasig (visas med varningsfärg / "saknas").

### Automatiska backlinks

Sektionen "Omnämnd av" i länkpanelen beräknas automatiskt genom att skanna **alla** inlästa filers `länkar`- och `relaterat`-fält och se vilka som pekar på den aktuella sidan. Det finns inget separat fält att fylla i för detta — det sköts helt av Waylight vid rendering.

---

## Specialtaggar i brödtexten

Utöver vanlig markdown (rubriker, listor, tabeller, bilder, citat, etc.) stöds följande specialsyntax:

### Wrapper-taggar: `{.klass}...{/}`

Generell syntax för att wrappa ett stycke text — eller ett helt block med rubriker, listor och tabeller — i en stylbar container. Fungerar både **inline** (kort text mitt i en mening) och **blocknivå** (flera stycken/rubriker/tabeller):

```markdown
Det här är {.viktigt}en viktig mening{/} mitt i texten.

{.spelledare}
## Attacker och förmågor
* **FV:** 16
* ...
{/}
```

Fördefinierade klasser:

| Klass | Effekt |
|---|---|
| `spelledare` | Spelledarinnehåll. Visas alltid inramat (brun/guld ruta), både låst och upplåst. I låst läge visas texten "🔒 SL: Låst innehåll, lås upp för att visa." istället för det verkliga innehållet. Använd för SL-specifika hintar, hemligheter och taktikråd som ändå ska synas som en tydlig markerad ruta även efter upplåsning. |
| `konfidentiellt` | Döljer innehållet helt tills upplåst (visar samma låsnotis-ruta som `spelledare` gör i låst läge). **Skillnaden**: när innehållet väl är upplåst renderas det helt normalt, utan någon ram eller specialstyling — som om taggen inte fanns. Använd när du bara vill hindra spoilers innan upplåsning, utan att permanent markera innehållet som "SL-material" i layouten. |
| `bildtext` | Centrerad, kursiv, mindre text — för bildtexter direkt under en bild. |
| `viktigt` | Framhäver text med guld-understrykning och fetare vikt, utan att dölja något. |
| `citat` | Kursiv, serif-stil (display-fonten) — för in-universe-citat eller stämningsfulla rader. |

**OBS:** all `aventyr/`-mappinnehåll är redan implicit konfidentiellt på filnivå (se [Låssystemet](#låssystemet-master-password)) — `{.spelledare}`/`{.konfidentiellt}` behövs bara för att dölja *delar* av en sida, eller för att markera SL-material inuti en annars publik, global sida (t.ex. en global regel- eller monstersida som har en spoiler-sektion).

### TODO-markering

```markdown
> **TODO:** Skriv klart den här sektionen.
```

Ett citatblock som inleds med `**TODO:**` (fetstil) renders med en distinkt gul/brun bakgrund istället för vanlig citat-styling, för att visuellt flagga ofärdigt innehåll under skrivandet.

---

## Bilder

```markdown
![Reva](../../bilder/monster/starka/reva.webp "Reva")
```

- Använd **riktiga, filsystem-relativa sökvägar** räknat från den aktuella filens plats — exakt samma sätt som GitHubs egen markdown-preview tolkar relativa bildlänkar. Waylight löser upp `../`- och `./`-segment mot filens faktiska sökväg i repot.
- Bilder lat-laddas (`loading="lazy"`) och hämtas aldrig i förväg — de laddas av webbläsaren först när de skrollas in i synfältet.
- Lägg bilder i `bilder/`, med undermappar som gärna matchar innehållstypen de hör till (`bilder/monster/`, `bilder/platser/`, etc.) för att hålla ordning, men detta är inget krav som Waylight kontrollerar — sökvägen i `![]()`-taggen är sanningen.

---

## Låssystemet ("master password")

Det här är en **UX-spärr, inte riktig säkerhet**. Allt i repot är publikt läsbart av vem som helst determinerad nog (view source, devtools, direkt via `raw.githubusercontent.com`). Syftet är att hålla spelare borta från spoilers medan de fritt kan bläddra i regler/monster tillsammans med spelledaren — inte att kryptografiskt skydda innehållet.

### Vad som räknas som konfidentiellt

- **Allt** under `aventyr/<namn>/...` (utom `aventyr.yaml` självt) räknas automatiskt som konfidentiellt — inget behöver anges manuellt.
- Enskilda **globala** filer (`regler/`, `monster/`, etc.) kan markeras manuellt med `confidential: true` i sin frontmatter.

### Hur upplåsning fungerar

- Lösenordet lagras aldrig i klartext — bara dess SHA-256-hash finns i koden.
- Vid korrekt lösenord sparas ett upplåst-flagga i `sessionStorage`, vilket innebär att upplåsningen **rensas automatiskt** när fliken/webbläsaren stängs (måste låsas upp på nytt varje ny session).
- Låst innehåll visas i trädet med en kursiv rad och 🔒-ikon. Att klicka på en låst rad, en låst wikilänk, eller en låst länk-chip triggar lösenordsprompten direkt.

### Lat laddning av äventyrsinnehåll

Även om ett äventyr är låst, hämtas **`aventyr.yaml`** (namn + filindex) alltid direkt vid inläsning — annars skulle äventyret inte ens synas i trädet. Det faktiska innehållet i äventyret (alla `.md`-filer under mappen) hämtas **först** när användaren klickar sig in i äventyret *och* anger rätt lösenord. Detta minimerar onödig nätverkstrafik och håller spoiler-innehåll borta från webbläsarens minne tills det verkligen efterfrågas.

---

## Namnkrockar

Eftersom länkning sker via kortnamn (filnamn utan sökväg/ändelse) måste filnamn vara unika **inom sitt scope**:

- Inom samma äventyr (`aventyr/<namn>/...`), oavsett undermapp.
- Inom samma globala topp-mapp (`regler/`, `monster/`, `karaktarer/`, `foremal/`), oavsett undermapp.

Om två filer i samma scope råkar heta likadant (t.ex. `monster/starka/reva.md` och `monster/boss/reva.md`), varnar Waylight om detta automatiskt vid inläsning (en banderoll högst upp i appen) eftersom länkar till det namnet kan peka fel. Döp om en av filerna för att lösa krocken — det finns inget sätt att disambiguera i själva länksyntaxen.

---

## Cache och uppdatering

Det här avsnittet är mest relevant att känna till som skribent, inte som strikt skrivregel:

- Waylight cachar allt inläst innehåll i webbläsarens `localStorage`, nyckat på repots senaste commit-SHA.
- Så länge repots `main`-gren inte har nya commits sedan senaste inläsningen, laddas allt från cache — inga nya nätverksanrop görs, även vid siduppdatering.
- Efter att du pushat ändringar till Waypoints, måste den som tittar i Waylight klicka på uppdatera-knappen (⟳) för att tvinga fram en ny hämtning direkt från GitHub, förbi cachen.
