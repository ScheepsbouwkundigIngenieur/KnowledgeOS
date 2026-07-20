# KnowledgeOS — Engineering & Governance Manual

**Status:** Definitief. Dit is het permanente ontwikkelhandboek.
**Hiërarchie:** `Architecture/Blueprint.md` (bevroren) > `Implementation_Master_Plan.md` > dit Manual. Bij tegenstrijdigheid wint het hogere document.
**Vervangingsverklaring:** dit Manual **vervangt** het Developer Playbook en de Developer Guide. Beide worden gearchiveerd, niet onderhouden. Reden: de vaste documentenset staat geen handboeken naast elkaar toe — één handboek, of geen. De permanente set is vanaf nu: Blueprint · Master Plan · dit Manual · contractdocument · `Current_State.md` · Decision Log · `Future_Ideas.md`. **Deze set groeit niet.**

---

## 1. Engineering Principles

Wat iedere ontwikkelaar altijd onthoudt — alles hieronder is hiervan afgeleid:

1. **De bestanden zijn de waarheid.** Kennis bestaat uitsluitend als bestand in het archief. Code, index en tools zijn bedienend personeel.
2. **Alles wat afgeleid is, is weggooibaar.** Kapot = weggooien en herbouwen, nooit repareren-met-angst.
3. **Niets draait permanent.** Elk onderdeel start koud vanaf bestanden, met één commando.
4. **De mens is de poort.** AI en automatisering stellen voor (staging); alleen een menselijke reviewhandeling maakt iets tot kennis, en elke review laat een append-only beslisrecord na.
5. **Conventie boven code.** Structuur zit in het contractdocument en wordt afgedwongen door de validator — niet door frameworks, niet door goede voornemens.
6. **Bouwen boven beschrijven.** Documentatie ontstaat uit werkende code. Niets bestaat vóór het een echt bestand verwerkt heeft (bestaansregel).
7. **Eén volgende stap.** `Current_State.md` bevat altijd de actuele staat en precies één volgende actie. Elke sessie begint en eindigt daar.
8. **Vervangen boven verbouwen.** Modules zijn wegwerpbaar achter stabiele contracten. Contracten wijzig je met de ernst van een API-break.
9. **Het kleinste werkende ding eerst.** Elke fase levert iets bruikbaars; "bijna af" is geen opleverstatus.
10. **De herstartdrempel is de maat.** Elke keuze wordt getoetst op: pak ik dit na drie weken stilstand binnen tien minuten weer op?

**Sessieritme (verplicht):** open `Current_State.md` → voer de éne stap uit → draai het startcommando (intake → scan → validator) → werk `Current_State.md` bij → commit. Geen sessie eindigt zonder deze afsluiting.

---

## 2. Architecture Governance

**Implementatiewijziging** (bouwen, geen formaliteit):

- alles binnen één module dat zijn contract respecteert;
- de bewust flexibele zaken (Blueprint §G2): indextechnologie en -schema, providerkeuze per taak, beslisrecord-velden (uitbreiden, nooit verwijderen), prompts, UI-vorm, module-interne opbouw, libraries, concrete staging-/inboxstructuur — mét contractupdate in dezelfde sessie;
- nieuwe AI-taken via de bestaande adapter en het bestaande datapad;
- nieuwe connectors die uitsluitend in de inbox schrijven;
- refactors die contracten en extern gedrag ongewijzigd laten.

**Architectuurwijziging** (heropening vereist):

- alles wat een Golden Rule (§10) of een van de zes beslissingen raakt;
- een tweede plek waar waarheid staat naast de bestanden;
- elk permanent draaiend onderdeel;
- elke automatische route van staging naar archief;
- module-communicatie anders dan via bestanden;
- een andere runtime naast of in plaats van Python;
- realtime-gedrag.

**Wanneer de Blueprint heropend mag worden — uitsluitend bij een nieuw feit**, vastgelegd als Decision Log-entry, gevolgd door herbeoordeling van alléén de geraakte beslissing. Geldige feiten zijn waarnemingen uit gebruik of omgeving, bijvoorbeeld: de NAS-mount blijkt structureel onbetrouwbaar; het archief overschrijdt de aangenomen ordegrootte; een provider verdwijnt; maandenlang gebruik toont een behoefte die het huidige model aantoonbaar niet kan dragen.

**Ongeldige heropeningsgronden — direct afwijzen, zonder discussie:**

- ontevredenheid, twijfel of "een beter idee" zonder nieuw feit;
- enthousiasme over een nieuwe tool, taal of hype;
- een AI-voorstel dat een regel schendt ("de AI stelde het voor" is geen feit);
- scopevergroting als route naar consensus ("we moeten X óók meenemen");
- elke "tijdelijke uitzondering" op een Golden Rule — die bestaat niet.

**Beslisregel bij twijfel:** raakt dit een van de zes beslissingen of een Golden Rule? Ja of onduidelijk → behandelen als architectuurwijziging tot het tegendeel beargumenteerd is. Eén onnodige log-entry kost niets; één sluipende drift kost het systeem.

---

## 3. Architecture Decision Checklist

Doorloop vóór iedere belangrijke wijziging. Eén "fout" antwoord = stoppen en de aangegeven route volgen.

| # | Vraag | Goed | Fout → actie |
|---|---|---|---|
| 1 | Raakt dit een van de zes beslissingen of Golden Rules? | Nee | Ja → heropeningsprocedure of afwijzen |
| 2 | Ontstaat er een tweede plek met waarheid naast de bestanden? | Nee | Ja → afwijzen |
| 3 | Moet er iets permanent draaien? | Nee | Ja → afwijzen of heropenen |
| 4 | Kan AI-output hiermee het archief bereiken zonder menselijke handeling? | Nee | Ja → afwijzen |
| 5 | Blijft alles afgeleide met één commando herbouwbaar? | Ja | Nee → herontwerpen |
| 6 | Verhoogt dit de herstartdrempel na drie weken stilstand? | Nee | Ja → geschreven rechtvaardiging vóór de bouw |
| 7 | Is de in-/output als contract-entry vast te leggen in dezelfde sessie? | Ja | Nee → wijziging is onaf gedefinieerd; niet starten |
| 8 | Past dit binnen de huidige fase van het plan? | Ja | Nee → één regel naar `Future_Ideas.md` |
| 9 | Is een nieuwe dependency echt nodig? | Nee / gemotiveerd in één regel | Ongemotiveerd → standaardbibliotheek |
| 10 | Kan het in 1–3 zittingen met een testbaar eindresultaat? | Ja | Nee → opsplitsen |

---

## 4. Definition of Done

Een module, feature of automatisering is **gereed** wanneer álles hieronder waar is:

- verwerkt echte archiefbestanden (geen demo-data als eindbewijs);
- contract-entry bestaat en klopt; validator draait groen;
- idempotent: tweemaal draaien geeft identiek resultaat;
- koud startbaar: nieuwe sessie, niets draait, één commando → werkt;
- afgeleide output is aantoonbaar weggooibaar en herbouwbaar;
- raakt het AI-output: de volledige lus staging → menselijke review → archief/afgewezen is gedemonstreerd, mét beslisrecord;
- de oplevercriteria van de betreffende planfase slagen als test, niet als indruk;
- draait mee in het ene startcommando waar van toepassing;
- `Current_State.md` is bijgewerkt met de volgende stap;
- geen open einden: rest-ideeën staan als één regel in `Future_Ideas.md`, bewuste schuld staat in het Decision Log (§7).

Niet vereist voor "done": perfecte prompts, optimale ranking, mooie UI, volledige documentatie. Dat zijn flexibele zaken; "werkt" is de norm, "best" is dat niet.

---

## 5. Code Review Playbook

Elke wijziging — door mens of AI geschreven — wordt vóór commit beoordeeld door de mens. Beoordeel tegen deze lijst, in deze volgorde.

**Blockers (nooit committen):**

- schrijft naar het archief buiten de menselijke reviewhandeling om;
- directe AI-provider-call buiten de adapterlaag;
- introduceert een permanent draaiend onderdeel;
- creëert of muteert data die niet als bestand in het archief herleidbaar is (tweede waarheid);
- wijzigt of verwijdert bestaande beslisrecords (append-only geschonden);
- leest of schrijft bestanden buiten het geregistreerde contract;
- omzeilt de conventiemodule (eigen logging/config/foutafhandeling);
- gevoelige data of secrets in code, config, prompts of testdata;
- destructieve operaties op het echte archief in tests;
- wijzigt Blueprint, Decision Log of dit Manual mee in een implementatie-commit.

**Waarschuwingen (bespreken, motiveren of terugbrengen):**

- nieuwe dependency zonder één-regel-motivatie;
- geen idempotentie aantoonbaar;
- complexiteit voorbij wat de huidige fase vraagt (framework-drang);
- functionaliteit buiten de oplevercriteria van de fase ("nu we toch bezig zijn");
- onduidelijke naamgeving of code die de reviewer niet in één lezing begrijpt;
- documentatiegroei buiten de vaste set;
- foutafhandeling die stilzwijgend doorgaat waar de keten hoort te stoppen.

**Altijd controleren:**

1. klopt de code met de contract-entry — en andersom;
2. koude-start-test slaagt;
3. herbouwtest slaagt (indien afgeleide output geraakt wordt);
4. validator groen;
5. de wijziging is uitlegbaar in twee zinnen — lukt dat niet, dan is hij te groot of niet begrepen.

---

## 6. Failure Playbook

De vijf faalwijzen waaraan KnowledgeOS het meest waarschijnlijk ten onder gaat, met vroege herkenning en structurele preventie.

**F1 — Ontwerpen vervangt bouwen** *(het hoofdrisico, Blueprint Deel E)*
- *Vroeg herkennen:* nieuwe documenten of templates zonder verwerkte bestanden; herstructureren en hernoemen; prompts die prompts verbeteren; meerdere opeenvolgende sessies zonder code-commit; heropenen van documenten zonder nieuw feit.
- *Voorkomen:* bestaansregel; bevroren documentenset (kop van dit Manual); sessieritme dat in werkende code eindigt; de meetregel "meer doc-commits dan code-commits over een langere periode = alarm".
- *Cruciale regels:* bestaansregel, bevriezing §G1, vaste documentenset.

**F2 — Verlating na stilstand**
- *Vroeg herkennen:* herstart kost >10 minuten; je herleest documentatie om te durven beginnen; het startcommando faalt of is vergeten.
- *Voorkomen:* de drie eindtesten van Fase 8 als periodieke gezondheidscheck; `Current_State.md` als enige herstartroute; niets dat draait.
- *Cruciale regels:* koude start, één volgende stap, herstartdrempel als maat.

**F3 — Sluipende architectuurdrift**
- *Vroeg herkennen:* het woord "tijdelijk" in een commit of gedachte; een script "even naast" de engine; een module die ongeregistreerd bestanden van een ander leest; validator die rood staat en genegeerd wordt.
- *Voorkomen:* validator als hard onderdeel van elke sessie-afsluiting; review-blockers; beslisregel bij twijfel (§2).
- *Cruciale regels:* contracten als enige communicatie, validator groen = werk af.

**F4 — AI-wildgroei**
- *Vroeg herkennen:* gecommitte code die je niet kunt uitleggen; AI-sessies die "alles even afmaken"; features die niemand gepland heeft; AI-output die zonder review in het archief blijkt te staan.
- *Voorkomen:* AI-regels in §8; padveiligheidstest in de testset; één fase-stap per AI-sessie; validator + koude start na elke AI-sessie.
- *Cruciale regels:* mens is de poort, adapterlaag verplicht, AI voegt geen scope toe.

**F5 — Vertrouwensverlies in het systeem**
- *Vroeg herkennen:* twijfel of index en archief overeenkomen; angst om afgeleide data weg te gooien; handmatige "reparaties" buiten de scripts om.
- *Voorkomen:* bij twijfel wint altijd het bestand; herbouw is één commando en wordt periodiek daadwerkelijk gedaan (weggooien-en-herbouwen als test, niet als noodgreep).
- *Cruciale regels:* bestanden zijn de waarheid, afgeleide is weggooibaar.

---

## 7. Technical Debt Policy

- **Voorkomen:** de Definition of Done (§4) en de review-blockers (§5) zijn de eerste verdedigingslinie. Schuld ontstaat vrijwel altijd als "tijdelijk" — en dat woord is per §2 verboden zonder registratie.
- **Aangaan:** bewuste schuld mag, mits geregistreerd als Decision Log-entry met klasse **Schuld**, inclusief: wat er niet goed is, waarom dat nu acceptabel is, en de afbetaalconditie (het feit of moment waarop hij opgelost wordt). Ongeregistreerde schuld is geen schuld maar een defect.
- **Niet toegestaan als schuld:** alles uit de blocker-lijst (§5). Golden Rules kennen geen schuldvorm.
- **Oplossen:** bij het aanraken van een module worden eerst diens openstaande schuld-entries bekeken; aflossen gaat vóór nieuwe functionaliteit in dezelfde module. Afgeloste schuld krijgt een afsluitende log-entry.
- **Bewaken:** meer dan een handvol open schuld-entries is zelf een signaal (F3); dan is de eerstvolgende zitting een aflos-zitting, geen bouwzitting.

---

## 8. AI Collaboration Guidelines

Het runtime-principe — AI stelt voor, de mens beslist — geldt onverkort voor het bouwen zelf.

**Geschikt voor AI (met menselijke review vóór commit):**

- modulecode schrijven binnen een gegeven contract en fase-stap;
- refactors binnen de conventieset;
- tests en testdata (synthetisch) genereren;
- foutanalyse, log-interpretatie, debugging-suggesties;
- prompts voor AI-taken opstellen en itereren;
- documentatie-concepten voor de vaste set (mens redigeert en commit).

**Altijd uitsluitend door de mens:**

- elke commit (lezen, begrijpen, dan pas committen — onuitlegbare code komt er niet in);
- elke reviewhandeling staging → archief;
- elke wijziging aan Blueprint, Decision Log of dit Manual;
- elke heropenings- of schuldbeslissing;
- elke contract-wijziging (AI mag voorstellen, mens legt vast);
- de dataclassificatie en alles wat gevoelige data raakt.

**Vaste sessieregels voor AI-tools (Claude Code, Claude, GPT, lokale modellen):**

1. Elke sessie krijgt de kaders mee: de Golden Rules (§10) en het relevante contract.
2. Eén fase-stap per sessie. Geen "maak het hele plan af"-opdrachten.
3. AI-tools krijgen nooit schrijftoegang tot het archief; ontwikkelwerk gebeurt op de repository en synthetische data.
4. Geen gevoelige archiefinhoud in cloud-context; de classificatieregel geldt ook tijdens ontwikkeling. Lokale modellen voor alles wat data raakt waarvan de classificatie onzeker is.
5. AI-voorstellen die scope, dependencies of features toevoegen: standaard weigeren of als één regel parkeren.
6. Een AI-voorstel dat een Golden Rule schendt wordt afgewezen, nooit "tijdelijk geaccepteerd" — ongeacht hoe goed het klinkt.
7. Na elke AI-sessie: validator + koude-start-test. Groen, of terugdraaien.

---

## 9. Long-term Maintainability

- **Duurzaamheid zit in de bestanden.** Code mag verouderen en integraal vervangen worden; zolang Golden Rules 1–2 gelden, overleeft de kennis elke technologiewissel. Onderhoud beschermt eerst contracten en datapad, dan pas codekwaliteit.
- **Repositoryprincipes** (geen voorgeschreven mappen of tooling): bron en afgeleide strikt gescheiden — wat herbouwbaar is, staat niet in versiebeheer; structuur volgt modulegrenzen en is daarmee zelf documentatie; één ingang (startcommando + `Current_State.md`); het contractdocument is de enige kaart van alle in-/output.
- **Documentenset is bevroren** (kop van dit Manual). Een nieuw permanent document mag alleen bestaan als het een bestaand document vervangt of een contractfunctie heeft. Dit Manual verving er twee — dat is de norm, niet de uitzondering.
- **Git:** hoofdbranch altijd werkend (koude start slaagt op elke commit); kleine commits die fase of contract benoemen; implementatie en documentwijziging nooit vermengd; heropeningsgrond in het commitbericht bij governance-wijzigingen.
- **Dependencies:** standaardbibliotheek eerst; elke toevoeging in één regel gemotiveerd. Elke dependency is toekomstig onderhoud.
- **Periodieke gezondheidscheck** = de drie eindtesten van Fase 8 (koude start · herstart ≤10 min · missie-test met vijf echte vragen). Zakt er één, dan is dát het eerstvolgende werk — vóór elke feature.
- **Consistentie is afgedwongen.** Validator, testset en startcommando dragen de consistentie. Wie een gat vindt dat de validator mist: eerst de validator uitbreiden, dan het gat dichten.
- **Het Decision Log is het geheugen van het waarom.** Bij twijfel eerst het log lezen. De duurste fout in dit systeem is een goedbedoelde "verbetering" die ongemerkt een bewust genomen beslissing terugdraait.

---

## 10. Golden Rules

Nooit overtreden. Geen uitzonderingen, geen schuldvorm, geen "tijdelijk".

1. De bestanden op de NAS zijn de enige waarheid.
2. Geen module bezit data die niet als bestand in het archief bestaat.
3. Alles wat afgeleid is, is met één commando weggooibaar en herbouwbaar.
4. Niets draait permanent; alles start koud.
5. AI schrijft uitsluitend naar staging.
6. Alleen een menselijke reviewhandeling verplaatst iets naar het archief.
7. Elke review levert een beslisrecord op; records zijn append-only, voor altijd.
8. Elke AI-aanroep loopt via de adapterlaag.
9. Gevoelige data blijft lokaal; cloud is opt-in per taak.
10. Modules communiceren uitsluitend via geregistreerde bestandssysteem-contracten.
11. Elke module gebruikt de bindende conventieset; de validator hoort groen te zijn aan het eind van elke sessie.
12. Niets bestaat vóór het een echt bestand verwerkt heeft — geen mappen, templates of documenten vooruit.
13. De Blueprint wijzigt alleen via heropening met een nieuw feit in het Decision Log.
14. De vaste documentenset groeit niet; een nieuw document vervangt een bestaand document of bestaat niet.
15. Geen commit die de mens niet gelezen en begrepen heeft — ongeacht wie of wat hem schreef.
16. Elke sessie eindigt met werkende staat, groene validator en een bijgewerkte `Current_State.md`.

---

**Herstartroute (de enige):** `Current_State.md` → de éne stap → startcommando → klaar. Dit Manual is naslag bij twijfel, geen verplichte lectuur per sessie.
