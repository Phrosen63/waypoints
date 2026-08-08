# Waypoints

Waypoints är innehållsrepot för **Waylight** - ett självbyggt, webbläsarbaserat verktyg för att förbereda och bläddra i TTRPG-kampanjer (Drakar och Demoner). Det här repot innehåller *bara* innehåll (`.md`- och `.yaml`-filer, samt bilder) - ingen kod. Waylight läser in filträdet direkt från GitHub och renderar allt i webbläsaren.

Den här filen dokumenterar hur innehållet ska struktureras och formateras för att Waylight ska tolka det korrekt.

---

## Innehållsförteckning

- [Filstruktur](#filstruktur)
- [Frontmatter](#frontmatter)
- [aventyr.yaml](#aventyryaml)
- [Länkning](#länkning)
- [Specialtaggar i brödtexten](#specialtaggar-i-brödtexten)
  - [Om {.rå} och nästling](#om-rå-och-nästling)
- [Bilder](#bilder)
- [Låssystemet ("master password")](#låssystemet-master-password)
- [Delning via URL](#delning-via-url)
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
├── klasser/           (globala klasser, kan ha undermappar per klasstyp, t.ex. besvarjare/)
├── bilder/            (bilder, undermappar matchar innehållstyp)
└── aventyr/
    └── <äventyrsnamn>/
        ├── aventyr.yaml       (namn + "filer"-index över allt innehåll i äventyret)
        ├── platser/
        ├── monster/
        ├── karaktarer/
        └── foremal/
```

**Globalt innehåll** (`regler/`, `monster/`, `karaktarer/`, `foremal/`, `klasser/`) är alltid synligt för alla, oavsett låst/upplåst status - om inte en enskild fil har `konfidentiell: true` i sin frontmatter (se [Låssystemet](#låssystemet-master-password)).

**Äventyrsinnehåll** (`aventyr/<namn>/...`) är *implicit konfidentiellt* - allt innehåll i en äventyrsmapp är dolt för oupplåsta användare, förutom `aventyr.yaml` själv, vars namn och filindex alltid är synligt (så att äventyret syns i trädet, om än låst).

Undermappar under `monster/`, `karaktarer/`, `klasser/` etc. (både globalt och per äventyr) är fritt valbara och används för att gruppera innehåll visuellt i trädet (t.ex. `monster/boss/`, `monster/svaga/`, `klasser/besvarjare/`). Namnet på undermappen blir rubriken i trädvyn.

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
konfidentiell: true
status: draft
toc: true
toc_nivaer: [2, 3]
delbar: true
tillbaka_knapp: false
---
```

Allt efter den avslutande `---` tolkas som markdown-brödtext.

### Fält

| Fält | Typ | Beskrivning |
|---|---|---|
| `type` | sträng | Innehållstyp. Styr ikonen som visas i trädet och länkpanelen samt "typ-badgen" högst upp på sidan. Kända typer: `regel` (§), `monster` (☠), `karaktär` (☺), `plats` (⌂), `föremål` (◆), `klass` (✦). Okänd/saknad typ visar `•` som ikon. |
| `namn` | sträng | Visningsnamn för sidan. Om utelämnat används filnamnet (utan `.md`) istället, i tabbar, träd, länkchips och sidtitel. |
| `länkar` | objekt (kategori → lista) | Explicita, kategoriserade länkar till annat innehåll. Varje nyckel är en fri textkategori (t.ex. `regler`, `personer`, `platser`) och värdet är en lista med kortnamn (filnamn utan `.md`). Visas i länkpanelen till höger, grupperat per kategori. |
| `relaterat` | lista | En platt lista med kortnamn på relaterat innehåll, utan kategori. Visas i en egen sektion i länkpanelen. |
| `taggar` | lista | Fria etiketter för sökning/filtrering. Matchas mot filnamn och `namn` när man söker i trädet (sökrutan matchar mot filnamn + `namn` + `taggar` sammanslaget). |
| `konfidentiell` | bool | Om `true`, döljs sidans innehåll bakom låssystemet tills upplåst (se nedan). Sätts oftast manuellt bara på enskilda **globala** filer (t.ex. en global boss som är tänkt som en spoiler) - allt inuti `aventyr/` är redan implicit konfidentiellt utan att detta behöver sättas. |
| `status` | sträng | Om satt till `draft`, visas en "✎ utkast"-badge bredvid typ-badgen högst upp på sidan. Rent visuellt, påverkar inget annat. |
| `toc` | bool | Om `true`, genereras automatiskt en innehållsförteckning ("Innehåll") överst i dokumentet. Vilka rubriknivåer som inkluderas styrs av `toc_nivaer` (se nedan) - standard är bara H2. Klickbara länkar som skrollar till rätt sektion. |
| `toc_nivaer` | lista med tal | Styr vilka rubriknivåer (H1–H6) som tas med i innehållsförteckningen, t.ex. `[2, 3]` för att inkludera både H2 och H3. **Default är `[2]`** (bara H2) om fältet saknas eller innehåller ogiltiga värden - bakåtkompatibelt med sidor som redan har `toc: true` men inte detta fält. Kräver `toc: true` för att ha någon effekt. Om flera nivåer anges, indenteras djupare rubriker (t.ex. H3) visuellt under närmast föregående rubrik på en lägre nivå (t.ex. H2) i innehållsförteckningen. |
| `delbar` | bool | Om `true`, visas en "🔗 Kopiera delningslänk"-knapp högst upp på sidan (bara synlig för den som redan är upplåst). Knappen genererar en länk som visar **just den här sidan** olåst för mottagaren, utan att låsa upp resten av äventyret eller sessionen i övrigt. Se [Delning via URL](#delning-via-url) för hur detta fungerar och när det är lämpligt att använda. Sätts manuellt per sida - ärvs inte automatiskt av t.ex. alla monster eller allt äventyrsinnehåll. |
| `tillbaka_knapp` | bool | Styr om den flytande "scrolla till toppen"-knappen ska visas på sidan. **Default är `true`** (knappen visas) om fältet helt saknas - sätt `tillbaka_knapp: false` för att medvetet stänga av den på enstaka sidor (t.ex. mycket korta sidor där den inte fyller något syfte). Knappen dyker upp automatiskt efter att man scrollat en bit ner i sidans innehåll, oavsett om sidan har `toc: true` eller ej. |

**Kortnamn**: när man refererar till en fil i `länkar`/`relaterat`, eller i en `[[wikilänk]]`, används alltid filnamnet utan `.md`-ändelse och utan sökväg - t.ex. `reva`, inte `monster/starka/reva.md`. Se [Länkning](#länkning) för hur upplösningen fungerar.

---

## aventyr.yaml

Varje äventyrsmapp under `aventyr/` måste innehålla en `aventyr.yaml` i sin rot. Den fyller två syften:

1. Ger äventyret ett **visningsnamn** i trädet (annars visas mappnamnet rakt av).
2. Fungerar som ett **filindex** - en lista över allt innehåll som finns i äventyret, oavsett om det redan är hämtat från GitHub eller inte. Det här är vad som gör att `resolveLink()` kan känna igen och peka mot innehåll i ett äventyr som ännu inte laddats (t.ex. ett annat, olåst äventyr som länkar till något i ett låst äventyr).

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
- `filer`: en platt lista med **fullständiga sökvägar** (relativt repots rot) till alla `.md`-filer i äventyret. Måste hållas i synk manuellt med det faktiska filinnehållet - om en fil läggs till eller tas bort i äventyrsmappen, uppdatera listan här också.

`aventyr.yaml` hämtas alltid direkt (även om äventyret är låst) - det är vad som gör att äventyret syns (låst, kursiverat, med 🔒) i trädet innan man loggat in. Resten av äventyrets filer hämtas först när man låser upp *och* klickar sig in i äventyret (lat laddning, se [Cache och uppdatering](#cache-och-uppdatering)).

---

## Länkning

Det finns tre sätt att länka mellan sidor:

### 1. Wikilänkar i brödtexten

```markdown
Revan använder ofta [[dimridåer]] för att skapa förvirring.
```

`[[kortnamn]]` i löptexten görs om till en klickbar länk. Länktexten blir automatiskt sidans `namn`-fält (eller filnamnet om `namn` saknas) - du kan alltså inte skriva en egen länktext med bara `[[kortnamn]]`, vilket kan ge grammatiskt klumpiga meningar om sidans namn inte råkar böjas som du vill ha det i just den meningen.

**Anpassad visningstext:** lägg till ett `|` och egen text efter kortnamnet för att styra exakt vad som visas, utan att det påverkar vilken sida länken pekar mot:

```markdown
En präst börjar med en [[vapenfardigheter|vapenfärdighet]].
```

Här pekar länken fortfarande mot filen `vapenfardigheter.md` (kortnamnet, före `|`, avgör alltid *målet*), men den synliga länktexten blir "vapenfärdighet" istället för sidans faktiska `namn`-fält (t.ex. "Vapenfärdigheter", bestämd form plural) - användbart för att böja ord grammatiskt korrekt i löpande text.

Om målet inte kan hittas visas texten olänkad men markerad (`Länk saknas: ...` som tooltip) - med anpassad visningstext visas då den texten istället för kortnamnet, så meningen fortfarande läser naturligt även när länken är trasig.

**Använd ALDRIG vanlig markdown-länksyntax (`[text](fil.md)`) för att länka mellan sidor** - det tolkas som en riktig extern URL av markdown-parsern (`target="_blank"`, `<a href="fil.md">`), inte som en Waylight-intern länk, och resulterar i en 404 när man klickar på den. Wikilänkar (`[[...]]`) är det enda sättet att skapa fungerande interna länkar i brödtexten.

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

1. **Exakt sökväg** - om strängen redan är en fullständig, existerande filsökväg, används den direkt.
2. **Samma äventyr** - om den länkande filen ligger i `aventyr/<namn>/...`, letas först i just det äventyrets redan inlästa filer.
3. **Globala mappar** - `regler/`, `monster/`, `karaktarer/`, `foremal/`, `klasser/` (i den ordningen), inklusive undermappar.
4. **Alla äventyrs filindex** - om inget hittats i redan inläst innehåll, söks i samtliga äventyrs `filer`-listor i `aventyr.yaml`. Om en träff hittas där, men äventyret ännu inte är hämtat/upplåst, returneras en **låst länk-descriptor** istället för en sökväg - detta är vad som ger 🔒-ikonen på wikilänkar/chips som pekar mot låst, ohämtat innehåll.
5. Hittas inget alls, betraktas länken som trasig (visas med varningsfärg / "saknas").

### Automatiska backlinks

Sektionen "Omnämnd av" i länkpanelen beräknas automatiskt genom att skanna **alla** inlästa filers `länkar`- och `relaterat`-fält och se vilka som pekar på den aktuella sidan. Det finns inget separat fält att fylla i för detta - det sköts helt av Waylight vid rendering.

---

## Specialtaggar i brödtexten

Utöver vanlig markdown (rubriker, listor, tabeller, bilder, citat, etc.) stöds följande specialsyntax:

### Wrapper-taggar: `{.klass}...{/}`

Generell syntax för att wrappa ett stycke text - eller ett helt block med rubriker, listor och tabeller - i en stylbar container.

Varje tagg är **block** som default (egen ram/rad, bryter medvetet layouten - kan inte stå mitt i en mening). Lägg till suffixet **`-inline`** på klassnamnet - t.ex. `{.viktigt-inline}` - för att tvinga fram **inline-läge** istället: ingen egen rad, kan aldrig bryta det omgivande stycket, avsedd att sitta mitt i en mening. Suffixet fungerar på vilken klass som helst, både de fördefinierade nedan och egna/anpassade klassnamn.

```markdown
Det här är {.viktigt-inline}en viktig fras{/} mitt i en mening.

{.spelledare}
## Attacker och förmågor
* **FV:** 16
* ...
{/}

Initiativ avgör turordning i strid: alla slår 1T10 och lägger till sin INIT-bonus.

{.exempel}
Anna har INIT-bonus +2 och slår en 6:a på tärningen, vilket ger totalt 8.
Björn har INIT-bonus +4 och slår en 3:a, vilket också ger totalt 7.
Anna går alltså före Björn i turordningen.
{/}
```

**Viktigt att komma ihåg när du skriver:** en blocktagg (utan `-inline`) mitt i en mening ger ogiltig, trasig HTML - stycket bryts sönder istället för att bara visa innehållet inramat. Använd blockvarianten bara när taggen ska stå på egen rad/eget stycke; använd `-inline` när den ska sitta mitt i löptext.

Fördefinierade klasser:

| Klass | Effekt |
|---|---|
| `spelledare` | Spelledarinnehåll. Visas alltid inramat (brun/guld ruta), både låst och upplåst. I låst läge visas en låsnotis istället för det verkliga innehållet ("🔒 SL: Låst innehåll, lås upp för att visa." som block, en kortare "🔒 SL" som inline). Använd för SL-specifika hintar, hemligheter och taktikråd som ändå ska synas som tydligt markerade även efter upplåsning. Finns i både block- och `-inline`-variant. |
| `konfidentiellt` | Döljer innehållet helt tills upplåst (visar samma sorts låsnotis som `spelledare` gör i låst läge). **Skillnaden**: när innehållet väl är upplåst renderas det helt normalt, utan någon ram eller specialstyling - som om taggen inte fanns. Gäller oavsett block eller `-inline`. Använd när du bara vill hindra spoilers innan upplåsning, utan att permanent markera innehållet som "SL-material" i layouten. |
| `bildtext` | Centrerad, kursiv, mindre text - för bildtexter direkt under en bild. Konceptuellt alltid block (står under en bild) - `-inline`-varianten finns men har sällan naturlig användning. |
| `viktigt` | Framhäver text med guld-understrykning och fetare vikt, utan att dölja något. Blockvarianten är ett helt, avskilt stycke; `-inline` är en enstaka fras mitt i en mening. |
| `effekt` | Framhäver text med lila understrykning och fetare vikt, utan att dölja något. Visuellt släkt med `viktigt` (samma stil, annan färg) - använd för att särskilja spelmekaniska effekter (skada, bonusar, statuseffekter) från allmänt viktig text. Block/`-inline` fungerar som `viktigt`. |
| `citat` | Kursiv, serif-stil (display-fonten) - för in-universe-citat eller stämningsfulla rader. Blockvarianten är ett fristående, indraget citatstycke; `-inline` smälter in i den omgivande meningen. |
| `exempel` | Ett tydligt avgränsat block (grön vänsterkant, ljust bakgrundstonad ruta) med en automatisk "Exempel:"-etikett överst (`-inline`-varianten får en kortare "Ex:"-prefix istället, ingen egen etikettrad). Använd för att ge ett konkret, förklarande exempel direkt efter en regel- eller mekanikbeskrivning - döljer inget, bara visuellt separerar exemplet från den omgivande löptexten. |
| `rå` | Visar sitt innehåll **ordagrant** - ingen markdown- eller taggparsning sker alls inuti. Använd för att visa bokstavlig `{.klass}`-syntax (eller annan text som annars skulle tolkas) som exempel i dokumentationen, utan att den faktiskt aktiveras. Blockvarianten renderas som ett monospace-kodblock; `-inline` som `kod` mitt i en mening. Kan nästlas i eller runt andra taggar - se [Om `{.rå}` och nästling](#om-rå-och-nästling) nedan för en viktig begränsning. |
| `nyckelord` | Framhäver ett ord eller en kort fras med gyllene, fetstil text - utan att dölja något. **Alltid inline oavsett suffix**: den körs aldrig genom markdown-parsern och renderas alltid som en `<span>` mitt i löpande text, vilket garanterar att den aldrig bryter stycket den står i - oavsett hur kort eller lång texten är. `-inline`-suffixet har ingen effekt på den här klassen (den är redan permanent inline) men skadar inte om det råkar sättas ändå. Använd för att lyfta fram enstaka spelmekaniska termer (t.ex. statuseffekter) direkt i en mening. |

**OBS:** all `aventyr/`-mappinnehåll är redan implicit konfidentiellt på filnivå (se [Låssystemet](#låssystemet-master-password)) - `{.spelledare}`/`{.konfidentiellt}` behövs bara för att dölja *delar* av en sida, eller för att markera SL-material inuti en annars publik, global sida (t.ex. en global regel- eller monstersida som har en spoiler-sektion).

### Om `{.rå}` och nästling

`{.rå}` (och `{.rå-inline}`) kan innehålla andra taggars syntax som ren text, t.ex.:

```markdown
Skriv {.rå-inline}{.viktigt-inline}text{/}{/} för att markera något som viktigt.
```
→ visar bokstavligen `{.viktigt-inline}text{/}` i en kodliknande markering, utan att taggen faktiskt aktiveras.

**Begränsning:** ett *komplett* `{.klass}...{/}`-par kan inte skrivas rakt av inuti `{.rå}` om det innehåller egen text mellan öppning och stängning på det sättet - parsern parar alltid ihop en `{/}` med senast öppnade taggen, så en extra inre stängning kan "läcka ut" och råka stänga `{.rå}` för tidigt, eller tvärtom fortsätta leta efter sin egen stängning längre fram i dokumentet än du tänkt dig. Skriv istället `{.rå}`s egen stängning `{/}` **direkt efter** den inre taggens `{/}`, som i exemplet ovan (`{.rå-inline}{.viktigt-inline}text{/}{/}`) - då fungerar det pålitligt. `[[wikilänkar]]` och enstaka `{`/`}`-tecken kräver ingen sådan försiktighet och kan skrivas rakt av inuti `{.rå}` utan problem.

Inga HTML-entities (`&#123;` etc.) behövs för att visa bokstavlig tagg-syntax - `{.rå}` hanterar escapingen automatiskt.

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

- Använd **riktiga, filsystem-relativa sökvägar** räknat från den aktuella filens plats - exakt samma sätt som GitHubs egen markdown-preview tolkar relativa bildlänkar. Waylight löser upp `../`- och `./`-segment mot filens faktiska sökväg i repot.
- Bilder lat-laddas (`loading="lazy"`) och hämtas aldrig i förväg - de laddas av webbläsaren först när de skrollas in i synfältet.
- Lägg bilder i `bilder/`, med undermappar som gärna matchar innehållstypen de hör till (`bilder/monster/`, `bilder/platser/`, etc.) för att hålla ordning, men detta är inget krav som Waylight kontrollerar - sökvägen i `![]()`-taggen är sanningen.

---

## Låssystemet ("master password")

Det här är en **UX-spärr, inte riktig säkerhet**. Allt i repot är publikt läsbart av vem som helst determinerad nog (view source, devtools, direkt via `raw.githubusercontent.com`). Syftet är att hålla spelare borta från spoilers medan de fritt kan bläddra i regler/monster tillsammans med spelledaren - inte att kryptografiskt skydda innehållet.

### Vad som räknas som konfidentiellt

- **Allt** under `aventyr/<namn>/...` (utom `aventyr.yaml` självt) räknas automatiskt som konfidentiellt - inget behöver anges manuellt.
- Enskilda **globala** filer (`regler/`, `monster/`, etc.) kan markeras manuellt med `konfidentiell: true` i sin frontmatter.

### Hur upplåsning fungerar

- Lösenordet lagras aldrig i klartext - bara dess SHA-256-hash finns i koden.
- Vid korrekt lösenord sparas ett upplåst-flagga i `sessionStorage`, vilket innebär att upplåsningen **rensas automatiskt** när fliken/webbläsaren stängs (måste låsas upp på nytt varje ny session).
- Låst innehåll visas i trädet med en kursiv rad och 🔒-ikon. Att klicka på en låst rad, en låst wikilänk, eller en låst länk-chip triggar lösenordsprompten direkt.

### Lat laddning av äventyrsinnehåll

Även om ett äventyr är låst, hämtas **`aventyr.yaml`** (namn + filindex) alltid direkt vid inläsning - annars skulle äventyret inte ens synas i trädet. Det faktiska innehållet i äventyret (alla `.md`-filer under mappen) hämtas **först** när användaren klickar sig in i äventyret *och* anger rätt lösenord. Detta minimerar onödig nätverkstrafik och håller spoiler-innehåll borta från webbläsarens minne tills det verkligen efterfrågas.

---

## Delning via URL

Waylight håller applikationens vy synkad med webbläsarens adressfält via query-parametrar, vilket gör att man kan kopiera och dela en URL för att ge någon annan exakt samma vy. Detta gäller både för vanlig navigering och för att medvetet avslöja specifikt innehåll för spelare - se nedan.

### Parametrar

| Parameter | Innehåll | Beskrivning |
|---|---|---|
| `tabs` | kommaseparerad lista med fullständiga sökvägar | Vilka flikar som är öppna, i öppningsordning. |
| `active` | en fullständig sökväg | Vilken av `tabs` som är den aktiva/synliga fliken. |
| `search` | fri text | Speglar trädets sökfält (mappar/filnamn/taggar). |
| `page_search` | fri text | Speglar sökningen inom den aktiva sidan (highlightning). Bara meningsfull tillsammans med `active`. |
| `reveal` | kommaseparerad lista med fullständiga sökvägar | Se [Delbara sidor](#delbara-sidor-reveal) nedan. |

Sökvägarna i `tabs`/`active`/`reveal` är alltid **fullständiga filsökvägar** (t.ex. `aventyr/dysterhamn/karaktarer/ryvok.md`), inte kortnamn - detta för att undvika all tvetydighet kring namnkrockar (se [Namnkrockar](#namnkrockar)).

URL:en uppdateras automatiskt (via webbläsarens historik-API, utan att skapa nya bakåtknapp-poster) varje gång man öppnar/stänger en flik, byter aktiv flik, eller söker. Det innebär att adressfältet alltid går att kopiera och dela rakt av för att ge mottagaren exakt samma vy - inga separata "dela"-knappar behövs för detta grundläggande beteende.

### Delbara sidor (`reveal`)

En sida med `delbar: true` i sin frontmatter (se [Frontmatter](#frontmatter)) kan delas så att den visas **olåst för mottagaren, utan att låsa upp resten av äventyret eller sessionen**. Detta är tänkt för sådant en spelledare vill ge spelarna direkt tillgång till efter att de själva upptäckt det i spelet - typiska exempel: en ledtråd i ett äventyr, eller ett monster spelarna just analyserat med en besvärjelse.

Så här går det till:

1. Sätt `delbar: true` i frontmatter på den specifika sidan.
2. Öppna sidan i Waylight (kräver att du själv är upplåst). En knapp, "🔗 Kopiera delningslänk", visas högst upp på sidan.
3. Klicka knappen - en URL kopieras till urklipp. Den innehåller `tabs`, `active` och `reveal`, alla satta till just den sidans sökväg.
4. Skicka länken till spelarna (Discord, SMS, etc.). När de öppnar den ser de **bara den sidan**, olåst - resten av äventyret (och appen i övrigt) förblir låst för dem tills de själva anger master-lösenordet.

**Säkerhetsspärren är `delbar: true` självt** - en `reveal`-sökväg i en manuellt ihopskriven URL som pekar på en fil *utan* `delbar: true` ignoreras helt av Waylight (loggas som en varning i webbläsarkonsolen, ingen effekt). Det gör att man inte råkar avslöja godtyckligt innehåll bara genom att känna till eller gissa en sökväg - sidan måste vara medvetet flaggad som delningsbar av den som skrev innehållet, i förväg.

**Begränsningar värda att känna till:**
- Det här är fortfarande bara en UX-spärr, precis som resten av låssystemet - filen är redan publikt läsbar för den som verkligen vill (se [Låssystemet](#låssystemet-master-password)).
- Avslöjandet gäller bara **just den länken**, inte en varaktig upplåsning i mottagarens egen instans av appen. Öppnar spelaren Waylight på nytt utan att ha kvar den specifika länken, är sidan låst igen som vanligt.
- Det finns inget sätt att "återkalla" en redan utskickad reveal-länk - samma begränsning som allt annat innehåll i repot.

---

## Namnkrockar

Eftersom länkning sker via kortnamn (filnamn utan sökväg/ändelse) måste filnamn vara unika **inom sitt scope**:

- Inom samma äventyr (`aventyr/<namn>/...`), oavsett undermapp.
- Inom samma globala topp-mapp (`regler/`, `monster/`, `karaktarer/`, `foremal/`, `klasser/`), oavsett undermapp.

Om två filer i samma scope råkar heta likadant (t.ex. `monster/starka/reva.md` och `monster/boss/reva.md`), varnar Waylight om detta automatiskt vid inläsning (en banderoll högst upp i appen) eftersom länkar till det namnet kan peka fel. Döp om en av filerna för att lösa krocken - det finns inget sätt att disambiguera i själva länksyntaxen.

---

## Cache och uppdatering

Det här avsnittet är mest relevant att känna till som skribent, inte som strikt skrivregel:

- Waylight cachar allt inläst innehåll i webbläsarens `localStorage`, nyckat på repots senaste commit-SHA.
- Så länge repots `main`-gren inte har nya commits sedan senaste inläsningen, laddas allt från cache - inga nya nätverksanrop görs, även vid siduppdatering.
- Efter att du pushat ändringar till Waypoints, måste den som tittar i Waylight klicka på uppdatera-knappen (⟳) för att tvinga fram en ny hämtning direkt från GitHub, förbi cachen.
