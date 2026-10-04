---
type: regel
namn: Kraftordsmagi
länkar:
  regler: []
relaterat: [besvarjelser, grundegenskaper, handlingar_i_strid, styrkor_svagheter_och_element]
taggar: [magi, grundregler]
toc: true
toc_nivaer: [2, 3]
status: draft
---

En särskild magisk tradition där besvärjaren inte lär sig färdiga besvärjelser, utan kraftord på ett magiskt pseudo-latin. Genom att kombinera orden kan spelaren själv skapa besvärjelser.

## Kraftordens fyra kategorier

### Essens

Beskriver vad **magin består av eller påverkar.**

| Kraftord | Betydelse |
| :--- | :--- |
| ignis | eld |
| aqua | vatten |
| terra | jord |
| ventus | vind |
| lux | ljus |
| tenebrae | mörker |
| fulgur | blixt |

Två Essens-ord kan inte kombineras direkt.

{.felexempel}
ignis + aqua
{/}

### Operation

Beskriver **vad besvärjaren gör med essensen.**

| Kraftord | Betydelse |
| :--- | :--- |
| amplificare | förstora |
| firmare | förstärka |
| dividere | dela |
| mutare | förändra |
| movere | förflytta |
| creare | skapa |
| destruere | förstöra |

En besvärjelse måste innehålla minst en {.nyckelord}essens{/}. Två {.nyckelord}operationer{/} kan inte kombineras direkt.

{.felexempel}
amplificare + firmare
{/}

Lägg till en essens, så fungerar det:

{.exempel}
ignis + amplificare + firmare
{/}

Essensen behöver inte komma från besvärjaren själv. Den kan också hämtas från omgivningen, till exempel genom att dra elden ur en närliggande brasa.

### Form

Beskriver **vilken form den magiska effekten får.**

| Kraftord | Betydelse |
| :--- | :--- |
| sphaera | sfär/klot |
| radius | stråle |
| murus | mur |
| gladius | svärd |
| scutum | sköld |
| linea | linje |

### Parameter

Modifierar exempelvis **storlek, hastighet, räckvidd eller varaktighet.**

| Kraftord | Betydelse |
| :--- | :--- |
| magnus | stor |
| parvus | liten |
| celer | snabb |
| longus | lång |
| brevis | kort |

## Magisk syntax

Den grundläggande principen är: {.viktigt}Essens → Operation → Form → Parameter{/}

En Essens fungerar som besvärjelsens grund. Övriga ord modifierar eller formar den.

### Exempel

{.exempel}
ignis + amplificare = Eld som förstoras.

ignis + amplificare + sphaera = En stor eld formad som ett klot - exempelvis ett eldklot.

aqua + movere = Vatten som förflyttas.

terra + amplificare + murus = En stor jordmur.
{/}

## Besvärjelser kan byggas över flera stridsrundor

Besvärjaren kan bara uttala ett begränsat antal ord per stridsrunda.  
En besvärjelse behöver inte släppas direkt. Besvärjare kan hålla kvar en påbörjad besvärjelse och fortsätta lägga till ord nästa runda.

{.exempel}
Runda 1:  
"ignis"  
En liten eld formas i besvärjarens hand.

Runda 2:  
"amplificare"  
Elden förstoras till storleken av en basketboll ungefär (obs! dock fortfarande formlös).

Runda 3:  
"firmare"  
Elden förstärks och kommer därför göra mer skada.

Runda 4:
"sphaera"
Elden formas till ett klot

Besvärjaren släpper besvärjelsen och kastar ett eldklot mot sitt mål.
{/}

## Spiritus bestämmer magins styrka

Grundegenskapen {.nyckelord}Spiritus{/} (SPI) ligger som grund för hur effektiv en besvärjelse är. Vissa kraftord påverkar effektiviteten.

| Förstärkningar | Skada |
| :--- | :--- |
| 0 | SPI / 4 |
| 1 × firmare | SPI / 2 |
| 2 × firmare | SPI |
| 3 × firmare | 2 × SPI |
| 4 × firmare | 3 × SPI |
| 5 × firmare | 4 × SPI |

{.konfidentiellt}
## Förstärkta Essens-ord

Det finns avancerade former av Essens-orden som innehåller firmare inbyggt. Vilket att ett enda uttalat ord motsvarar två kraftord.

Dessa avancerade former är inte kända av nya karaktärer. De kan upptäckas, läras eller erhållas genom progression.

| Grund | +1 firmare | +2 firmare |
| :--- | :--- | :--- |
| ignis | ignifer | igniferus |
| aqua | aquifer | aquiferus |
| terra | terrifer | terriferus |
| ventus | ventifer | ventiferus |
| lux | lucifer | luciferus |
| tenebrae | tenebrifer | tenebriferus |
| fulgur | fulgifer | fulgiferus |

{.exempel}
ignifer = ignis + firmare  
igniferus = ignis + firmare + firmare
{/}

{/}
