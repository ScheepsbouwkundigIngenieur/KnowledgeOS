# KnowledgeOS — Developer Playbook

**Status:** Definitief
**Hoort bij:** `Architecture/Blueprint.md` (bevroren 19-07-2026) · `Implementation_Master_Plan.md`
**Doelgroep:** iedere ontwikkelaar die aan KnowledgeOS werkt — inclusief de oorspronkelijke ontwikkelaar na maanden afwezigheid.
**Functie:** de bestaande architectuur vertalen naar ontwikkelrichtlijnen. Dit document neemt geen beslissingen en introduceert geen architectuur. Bij tegenstrijdigheid wint de Blueprint, daarna het Master Plan, daarna dit Playbook.

---

## 1. Hoe er binnen deze architectuur ontwikkeld wordt

- **Werk in fases, één tegelijk.** Het Master Plan bepaalt wat er nu gebouwd wordt. `Foundation/02_Current_State.md` bevat altijd de actuele fase en precies één volgende stap. Elke werksessie begint daar en eindigt met het bijwerken ervan.
- **Sessieritme:** open `Current_State.md` → voer de éne stap uit → draai het startcommando (intake → scan → validator) → werk `Current_State.md` bij → commit. Geen sessie eindigt zonder deze afsluiting; dat is wat koude herstart mogelijk maakt.
- **Bouwen boven beschrijven.** Documentatie ontstaat uit werkende code, nooit ervoor. De bestaansregel geldt onverkort: geen mappen, templates of documenten voor iets dat nog geen echt bestand verwerkt heeft.
- **Klein opleveren.** Elke fase eindigt met werkende code over echte archiefbestanden, een contract-entry indien er nieuwe in-/output is, en groene validator. "Bijna af" bestaat niet als opleverstatus.
- **Elke discussie eindigt in een beslissing of een gedateerd uitstel.** Open eindes zijn geen geldige uitkomst; uitgestelde punten gaan als één regel naar `Future_Ideas.md`.

## 2. Architectuurregels die nooit overtreden worden

Deze regels volgen direct uit de zes bevroren beslissingen. Overtreding is per definitie een architectuurwijziging (zie §7) — er bestaat geen "kleine uitzondering".

1. **De bestanden op de NAS zijn de enige waarheid** (B3). Geen module bezit data die niet als bestand in het archief bestaat.
2. **Alles wat afgeleid is, is weggooibaar** (B3). Index, caches en tussenresultaten moeten op elk moment verwijderd en met één commando herbouwd kunnen worden. Code die dat breekt, is fout — ook als hij werkt.
3. **Niets draait permanent** (principe 3). Geen daemons, services, schedulers of processen die "aan" moeten staan. Alles is koud startbaar.
4. **AI schrijft uitsluitend naar staging** (B6). Verplaatsing naar het archief gebeurt uitsluitend door een menselijke reviewhandeling. Geen script, taak of tool krijgt die bevoegdheid — ook niet tijdelijk, ook niet voor tests.
5. **Elke AI-aanroep loopt via de adapterlaag** (B6). Directe provider-calls in modulecode zijn verboden; de validator controleert hierop.
6. **Elke review levert een append-only beslisrecord** (B5). Records worden nooit gewijzigd of verwijderd, alleen toegevoegd.
7. **Modules communiceren uitsluitend via bestandssysteem-contracten** (B4), geregistreerd in het ene contractdocument. Een module die ongeregistreerd andermans bestanden leest of schrijft, is defect.
8. **Python is de runtime; de conventieset is bindend** (B2). Elke module gebruikt de gedeelde conventiemodule voor logging, configuratie en foutafhandeling.
9. **Gevoelige data blijft lokaal** (B6 + classificatiegate). Cloud is opt-in per taak, nooit standaard.
10. **De Blueprint wijzigt alleen via heropening** met een nieuw feit in het Decision Log (§G1).

## 3. Bewust flexibele keuzes

Deze zaken zijn expliciet vrij (Blueprint §G2 en Deel E). Ze mogen zonder heropening veranderen, mits de onbreekbare regels intact blijven:

- indextechnologie en indexschema (weggooibaar per regel 2);
- providerkeuze per AI-taak en modelkeuze;
- velden van het beslisrecord (uitbreiden mag; verwijderen niet — append-only geldt ook voor het formaat);
- prompts en promptbeheer;
- UI-vorm en -technologie van elke module;
- module-interne opbouw en gebruikte libraries;
- concrete mappenstructuur van staging/inbox (vastgelegd in het contractdocument, aanpasbaar mét contractupdate).

Vuistregel: flexibel betekent *goedkoop vervangbaar*, niet *vrijblijvend*. Wie iets flexibels wijzigt, werkt het contractdocument in dezelfde sessie bij.

## 4. Hoe nieuwe modules ontworpen worden

Een module is een los startbaar Python-programma op de kern (archief + index), niets meer.

Ontwerpvragen, in volgorde:

1. **Bestaansvraag:** welk echt bestand gaat deze module verwerken in de eerste bouwzitting? Geen antwoord = geen module; het idee gaat als één regel naar `Future_Ideas.md`.
2. **Contractvraag:** welke bestanden leest hij, welke schrijft hij, waar? Dit is het volledige externe ontwerp — meer interface bestaat niet in deze architectuur.
3. **Padvraag:** raakt de module AI-output? Dan is staging de enige schrijfbestemming en de review-flow de enige route naar het archief.
4. **Koude-startvraag:** werkt de module na maanden stilstand met alleen de bestanden en één commando? Elke afhankelijkheid van draaiende staat is een ontwerpfout.
5. **Toetsvraag:** verlaagt dit de herstartdrempel na drie weken stilstand, of verhoogt hij die? Verhogen vereist geschreven rechtvaardiging vóór de bouw start.

`Architecture/Module_Template.md` wordt pas ingevuld tijdens de eerste bouwzitting, niet ervoor.

## 5. Levensloop van een module: idee → oplevering

1. **Idee** → één regel in `Future_Ideas.md`. Punt. Uitwerken telt als starten.
2. **Selectie** → alleen wanneer het Master Plan (of, na V1, het dan geldende plan) de module aan de beurt stelt. Ideeën springen de rij niet voorbij.
3. **Eerste bouwzitting** → contract-entry (max. één alinea) + code die minstens één echt bestand verwerkt. Vanaf dit moment bestaat de module.
4. **Uitbouw** → in zittingen van 1–3, elk eindigend met werkende staat, groene validator en bijgewerkte `Current_State.md`.
5. **Oplevering** → de oplevercriteria uit het plan slagen als test, niet als indruk. De module draait mee in het ene startcommando waar van toepassing.
6. **Afronding** → `Current_State.md` wijst naar de volgende fase. Restideeën terug naar `Future_Ideas.md`.

Een module die na zijn eerste zitting drie sessies lang geen echt bestand meer verwerkt, wordt hardop ter discussie gesteld: bouwen we hem, of parkeren we hem?

## 6. Organisatie van repository, code, documentatie en tests

De Blueprint schrijft geen mappen of tooling voor. De repository voldoet daarom aan principes, niet aan een voorgeschreven structuur:

**Repositoryprincipes**

- **Bron en afgeleide strikt gescheiden.** Alles wat herbouwbaar is (index, caches, gegenereerde output) is herkenbaar als weggooibaar en wordt niet versiebeheerd. Wat in versiebeheer staat, is per definitie bron.
- **Structuur volgt modulegrenzen.** De indeling van de code weerspiegelt de contracten: wat samen één module is, staat bij elkaar; gedeeld is alleen de conventiemodule. Een lezer moet uit de structuur de architectuur kunnen aflezen.
- **Eén ingang.** Er is één voor de hand liggend startpunt (het startcommando) en één statusdocument (`Current_State.md`). Wie de repository voor het eerst opent, hoeft niets anders te weten om te beginnen.
- **Het contractdocument is de kaart.** Elke in-/output van elke module staat erin; het is de enige plek waar dat vastligt. Structuurwijziging zonder contractupdate is per definitie onaf.
- **Documentatie is minimaal en dwingend.** De vaste set: Blueprint, Master Plan, dit Playbook, contractdocument, `Current_State.md`, Decision Log, `Future_Ideas.md`. Nieuwe permanente documenten worden alleen toegevoegd als ze een van deze vervangen of een contractfunctie hebben — niet ernaast.

**Testprincipes**

- Tests zijn zelf batchscripts onder dezelfde conventieset: koud startbaar, geen draaiende afhankelijkheden.
- De kerngaranties worden getest, niet de details: idempotentie (tweemaal draaien = zelfde resultaat), herbouwbaarheid (index weg → herbouw → identiek), padveiligheid (AI-output kan het archief niet bereiken zonder reviewhandeling), contractnaleving (validator groen).
- Testdata staat buiten het archief. Het echte archief is nooit testomgeving voor destructieve operaties.

**Git-principes**

- De hoofdbranch is altijd werkend: koude-start-test slaagt op elke commit die erop landt.
- Commits zijn klein en benoemen de fase of het contract dat ze raken. Eén commit vermengt geen implementatie met architectuurdocument-wijziging.
- Wijzigingen aan Blueprint, Playbook of Decision Log staan in eigen commits met de heropeningsgrond in het bericht.
- Meetsignaal (Blueprint Deel E): meer commits in documentatie dan in code over een langere periode is een alarmsignaal, geen stijlkeuze.

## 7. Implementatie of architectuurwijziging?

**Gewone implementatie** — bouwen zonder verdere formaliteit:

- alles binnen één module dat het contract van die module respecteert;
- alles uit §3 (flexibele keuzes), inclusief contractupdate in dezelfde sessie;
- nieuwe AI-taken via de bestaande adapter en het bestaande datapad;
- nieuwe connectors die uitsluitend in de inbox schrijven;
- refactors die extern gedrag en contracten ongewijzigd laten.

**Architectuurwijziging** — vereist heropening via het Decision Log (nieuw feit, herbeoordeling van uitsluitend de geraakte beslissing):

- alles wat een van de tien regels in §2 raakt;
- een tweede plek waar waarheid komt te staan naast de bestanden;
- elk permanent draaiend onderdeel;
- elke automatische route van staging naar archief;
- elke communicatievorm tussen modules anders dan bestanden;
- een runtime naast of in plaats van Python;
- realtime-gedrag.

**Beslisregel bij twijfel:** stel de vraag "raakt dit een van de zes beslissingen of tien regels?" Ja of onduidelijk → behandelen als architectuurwijziging tot het tegendeel is beargumenteerd. De kosten van één onnodige log-entry zijn verwaarloosbaar; de kosten van één sluipende architectuurwijziging niet.

## 8. Veilig gebruik van Claude Code en andere AI-tools tijdens ontwikkeling

Het runtime-principe (AI stelt voor, mens beslist) geldt onverkort voor het bouwproces zelf:

- **AI-gegenereerde code is een voorstel.** Niets wordt gecommit zonder dat de mens het gelezen en begrepen heeft. Code die je niet kunt uitleggen, ga je niet onderhouden — dus komt hij er niet in.
- **AI-tools krijgen nooit schrijftoegang tot het archief.** Ontwikkelwerk gebeurt op de repository en op testdata, nooit rechtstreeks op de NAS-kennislaag. Ook een agent met terminaltoegang werkt binnen die grens.
- **Elke AI-sessie krijgt de kaders mee:** de tien regels uit §2 en het relevante contract. Een AI-voorstel dat een regel schendt, wordt afgewezen — niet "tijdelijk geaccepteerd".
- **AI mag geen scope toevoegen.** Voorstellen voor extra features, dependencies of frameworks worden standaard geweigerd of geparkeerd in `Future_Ideas.md`. De verleiding "de AI had nog een goed idee" is dezelfde ADHD-trap als altijd, met hogere productiesnelheid.
- **AI wijzigt nooit Blueprint, Decision Log of dit Playbook.** Die documenten worden uitsluitend door de mens aangepast, na de heropeningsprocedure.
- **Geen gevoelige archiefinhoud in cloud-AI-context tijdens ontwikkeling.** De dataclassificatieregel geldt ook voor ontwikkelsessies: bouwen en testen met neutrale of synthetische data.
- **Klein houden.** Eén fase-stap per AI-sessie. Lange sessies waarin een agent "alles even afmaakt" produceren precies de ongecontroleerde wildgroei waar dit systeem tegen gebouwd is.
- **Na elke AI-sessie:** validator draaien + koude-start-test. Groen of terugdraaien.

## 9. Veelvoorkomende valkuilen en preventie

| Valkuil | Herkenning | Preventie (structureel) |
|---|---|---|
| Ontwerpen vervangt bouwen | Nieuwe templates/mappen zonder inhoud; documenten herstructureren; meer doc-commits dan code-commits | Bestaansregel; bevriezing; sessie eindigt altijd in werkende code |
| Rabbit hole in één deelprobleem | Derde zitting aan dezelfde subfase zonder geslaagde test | Zittingslimiet per fase; F2d-regel als voorbeeld: afbreken is een geldige uitkomst |
| Systeemdrang onder stress | Behoefte aan "nieuwe opzet" zonder nieuw feit | Heropeningsprocedure; ideeën als één regel parkeren |
| "Tijdelijke" uitzondering op een contract | "Even snel", "alleen voor nu" | Uitzonderingen bestaan niet (§2); validator faalt = werk onaf |
| AI-output omzeilt review | Script dat staging automatisch doorschuift, ook in tests | Padveiligheidstest in de testset; regel 4 |
| Scriptwildgroei buiten conventies | Los script "naast" de engine | Alles via de conventiemodule; validator kent alle geregistreerde artefacten |
| Feature-toevoeging tijdens bouw | "Nu we toch bezig zijn…" | Oplevercriteria van de fase zijn de grens; rest naar `Future_Ideas.md` |
| Perfectioneren van flexibele delen | Eindeloos tunen van prompts, schema's, ranking | §3-vuistregel: flexibel = vervangbaar; V1-criteria zijn "werkt", niet "best" |
| Herstart via documentatie i.p.v. systeem | Na stilstand eerst alles herlezen | Startcommando + `Current_State.md` zijn de enige herstartroute; herstart-test uit Fase 8 als norm |

## 10. Meerjarige onderhoudbaarheid, uitbreidbaarheid en consistentie

- **Duurzaamheid zit in de bestanden, niet in de code.** Code mag verouderen en vervangen worden; zolang regel 1 en 2 gelden, overleeft de kennis elke technologiewissel. Onderhoud richt zich daarom eerst op het intact houden van contracten en datapad, pas daarna op codekwaliteit.
- **Uitbreiden = module + contract-entry.** Als een uitbreiding méér vraagt dan dat, is het geen uitbreiding maar een architectuurwijziging (§7).
- **Vervangen boven verbouwen.** Modules zijn bedoeld om weggegooid en herschreven te worden achter een stabiel contract. Een module herschrijven is goedkoop; een contract wijzigen is dat niet — behandel contractwijzigingen met dezelfde ernst als API-breaks.
- **Dependencies minimaal en gemotiveerd.** Elke externe library is toekomstig onderhoud. Standaardbibliotheek eerst; elke toevoeging in één regel gemotiveerd bij het contract of in de commit.
- **Periodieke gezondheidscheck = de drie eindtesten van Fase 8** (koude start, herstarttijd ≤10 min, missie-test). Zakt er één, dan is dát het onderhoudswerk — niet nieuwe features.
- **Consistentie is afgedwongen, niet afgesproken.** De validator, de testset en het ene startcommando zijn de dragers; goede voornemens tellen niet als maatregel. Wie een inconsistentie vindt die de validator mist, breidt eerst de validator uit en lost daarna de inconsistentie op.
- **Het Decision Log is het geheugen van het waarom.** Elke toekomstige ontwikkelaar leest bij twijfel eerst het log en het bijbehorende beslisrecord voordat hij iets "verbetert". De duurste fout in dit systeem is een goed bedoelde wijziging die een bewust genomen beslissing ongedaan maakt zonder het te weten.

---

**Leesvolgorde voor een nieuwe ontwikkelaar (of jezelf na 6 maanden):** `Current_State.md` → dit Playbook §1–2 → het contractdocument → aan het werk. Blueprint en Master Plan zijn naslag bij twijfel, geen verplichte herlezing per sessie.
