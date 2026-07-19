# KnowledgeOS — Implementation Master Plan

**Hoort bij:** `Architecture/Blueprint.md` (definitief, bevroren 19-07-2026)
**Functie:** het optimale bouwplan van eerste script tot en met V1. Dit document voegt geen architectuur toe en wijzigt de Blueprint niet. Volgorde op moduleniveau volgt de bevroren roadmap: scan → Review Workbench → Inbox Automation → Search → AI Assistant → Connectors.

---

## Definitie van V1

V1 is bereikt wanneer de volledige kernlus werkt:

> nieuw bestand → inbox → staging → menselijke review (met beslisrecord) → archief → index → terugvindbaar → AI-ondersteund, met minstens één connector.

En bovendien: alles start koud vanaf bestanden, met één commando, na weken stilstand.

---

## Spelregels voor het bouwen (afgeleid uit de Blueprint, geen nieuwe regels)

- **Eén fase tegelijk.** De volgende fase begint pas als de vorige gereed is. Geen parallel werk.
- **Bestaansregel geldt per fase:** geen mappen, templates of documentatie voor iets dat nog geen echt bestand verwerkt. Contractregistratie hoort bij de bouwzitting zelf (max. één alinea).
- **Elke fase eindigt met:** (1) werkende code die echte archiefbestanden verwerkt, (2) contract-entry indien de fase een nieuwe in-/output introduceert, (3) bijgewerkte `Current_State.md` met de éne volgende stap.
- **Fasegrootte:** 1–3 zittingen. Groter = fase splitsen, nooit oprekken.
- **Planwijziging:** dit plan is goedkoop aanpasbaar op basis van bouwervaring. Alleen wijzigingen die een Blueprint-beslissing raken vereisen heropening via het Decision Log.

---

# Fase 0 — Inventaris & mount-verificatie

*(= §G3 van de Blueprint, hier onveranderd overgenomen als startpunt)*

- **Doel:** eerste afgeleide index (`index/catalog.jsonl`) + verificatie van aannames 1 en 3.
- **Waarom eerst:** vastgelegd in §G3; levert direct bruikbaar resultaat en toetst het fundament (NAS-mount) vóór er iets op gebouwd wordt.
- **Afhankelijkheden:** geen.
- **Oplevercriteria:** `scan_archive.py` draait foutloos over de echte archiefmap; `catalog.jsonl` bevat per bestand pad, naam, extensie, grootte, wijzigingsdatum; totaaltelling geprint; outputlocatie als eerste regel in het contractdocument.
- **Belangrijkste bestanden:** `engine/scan_archive.py` · `index/catalog.jsonl` · contractdocument (eerste entry).
- **Risico's:** mount faalt of is traag → nieuw feit voor Decision Log, niet omheen werken. Zeer grote mappen → telling per submap loggen, geen optimalisatie starten.
- **Gereed wanneer:** de scan tweemaal achter elkaar identiek resultaat geeft (idempotent) en de telling plausibel is tegen aanname 3.

---

# Fase 1 — Engine-basis (B2)

## 1a. Conventiemodule, gedestilleerd uit Fase 0

- **Doel:** één gedeelde module (logging, configuratie, foutafhandeling, exit-codes) waar elk volgend script op bouwt; `scan_archive.py` er als eerste naartoe gerefactord.
- **Waarom nu:** B2 maakt de conventieset bindend. Dit is het laatste moment waarop dat goedkoop is — bij 1 script is refactoren een uur, bij 10 scripts een week. Conventies worden *gedestilleerd uit werkende code*, niet vooraf ontworpen (bestaansregel).
- **Afhankelijkheden:** Fase 0.
- **Oplevercriteria:** `engine/common.py` (of vergelijkbaar) bestaat; `scan_archive.py` gebruikt het; gedrag ongewijzigd (zelfde catalog-output); logregels in vast formaat.
- **Belangrijkste bestanden:** `engine/common.py` · aangepaste `scan_archive.py`.
- **Risico's:** over-engineering van de gedeelde module ("framework-drang"). Grens: alleen wat scan_archive vandaag nodig heeft.
- **Gereed wanneer:** scan draait ongewijzigd via de conventiemodule en de conventies staan in max. één pagina beschreven in het contractdocument.

## 1b. Conventievalidator

- **Doel:** `validate_conventions.py` — leest het contractdocument en controleert dat geregistreerde artefacten bestaan en aan vorm voldoen; meldt afwijkingen.
- **Waarom nu:** dit is de tegenmaatregel uit A8.2 (conventie-erosie). Hij moet bestaan vóór het aantal contracten groeit, anders wordt hij nooit ingehaald.
- **Afhankelijkheden:** 1a (gebruikt de conventiemodule).
- **Oplevercriteria:** validator draait groen op de huidige staat; bewust geïntroduceerde fout (bestand hernoemen) wordt gedetecteerd en gemeld.
- **Belangrijkste bestanden:** `engine/validate_conventions.py`.
- **Risico's:** validator te streng maken → frictie → omzeilen. Start met "bestaat het en is het leesbaar", niet meer.
- **Gereed wanneer:** de valse-fout-test slaagt en de validator onderdeel is van de standaard werkwijze (zie 3b).

---

# Fase 2 — Module 1: Review Workbench (B1, B5)

## 2a. Staging-gebied en reviewcontract

- **Doel:** vaste locaties `inbox/` en `staging/` op de NAS, geregistreerd in het contractdocument, met één echt bestand door de flow als bewijs.
- **Waarom nu:** het mens-gecontroleerde datapad (staging → review → archief) is de poort waar elke latere AI-functie doorheen moet (B6). Zonder poort geen veilige module 3, 5 of 6.
- **Afhankelijkheden:** Fase 1 (conventies + validator).
- **Oplevercriteria:** locaties bestaan; contract-entry beschrijft wie schrijft (alles behalve de mens schrijft alléén hier) en wie verplaatst (alleen de mens); één echt bestand handmatig door inbox → staging gehaald.
- **Belangrijkste bestanden:** contractdocument (staging-entry) · `inbox/` · `staging/`.
- **Risico's:** mappenstructuur-perfectionisme. Grens: twee mappen, één alinea contract, klaar.
- **Gereed wanneer:** validator kent de nieuwe locaties en draait groen.

## 2b. Beslisrecord v1

- **Doel:** append-only beslisrecord (plain Markdown, naast de content in het archief): wat voorgesteld, wat besloten, waarom, datum.
- **Waarom nu:** B5 maakt reviewuitkomsten zelf tot kennis in de SSoT. Het record moet bestaan vóór de eerste review, anders gaat de eerste geleerde kennis verloren.
- **Afhankelijkheden:** 2a.
- **Oplevercriteria:** schrijffunctie in de engine maakt een record aan; record is leesbaar zonder tooling; append-only afgedwongen (nooit overschrijven); één echt record geschreven voor het testbestand uit 2a.
- **Belangrijkste bestanden:** `engine/decision_record.py` · eerste record in het archief.
- **Risico's:** veldenschema-discussie (zit bewust in de uitstellijst). Start met de vier velden hierboven; uitbreiden mag later, verwijderen niet.
- **Gereed wanneer:** record geschreven, teruggelezen en door de validator herkend.

## 2c. Review-flow (CLI)

- **Doel:** één commando dat staging-items toont en per item de menselijke keuze afhandelt: accepteren (→ archief + beslisrecord) of afwijzen (→ afgewezen-locatie + beslisrecord).
- **Waarom nu:** hiermee is Module 1 functioneel af: de poort werkt. CLI eerst, omdat het reviewcontract — niet de interface — het duurzame deel is (uitstellijst).
- **Afhankelijkheden:** 2a, 2b.
- **Oplevercriteria:** drie echte bestanden gereviewd; elk heeft een beslisrecord; archief bevat de geaccepteerde bestanden op contractconforme locaties; geen enkel geautomatiseerd pad kan de verplaatsing doen.
- **Belangrijkste bestanden:** `engine/review.py`.
- **Risico's:** functionaliteit stapelen (tags, filters, bulk). Grens: tonen, beslissen, vastleggen, verplaatsen. Meer is Fase 2d of `Future_Ideas.md`.
- **Gereed wanneer:** de drie-bestanden-test slaagt en de oude HTML-workbench formeel niet meer gebruikt wordt (afbouw per A4).

## 2d. Eenvoudigste bruikbare interface

- **Doel:** het genoteerde UX-bezwaar (Deel C) adresseren: een interface met lagere drempel dan de kale CLI, bovenop hetzelfde reviewcontract.
- **Waarom hier:** pas nadat het contract bewezen werkt; de interface is wisselbaar, het contract niet. Technologiekeuze is vrij (uitstellijst) en wordt bij aanvang van deze fase gekozen, niet eerder.
- **Afhankelijkheden:** 2c.
- **Oplevercriteria:** één reviewronde volledig via de interface uitgevoerd; interface bezit geen eigen data (alles via contract); systeem blijft volledig werkend zonder de interface.
- **Belangrijkste bestanden:** afhankelijk van gekozen vorm; geregistreerd in contractdocument als consument van het reviewcontract.
- **Risico's:** dit is de grootste rabbit-hole-kandidaat van het hele plan. Harde grens: 2 zittingen. Niet af in 2 zittingen → terug naar CLI, interface naar `Future_Ideas.md`, door naar Fase 3.
- **Gereed wanneer:** de reviewronde-test slaagt binnen de zittingslimiet — of de fase expliciet is afgebroken en genoteerd. Beide uitkomsten zijn geldig.

---

# Fase 3 — Inbox Automation

## 3a. Intake-script

- **Doel:** `intake.py` — verplaatst nieuwe bestanden uit `inbox/` naar `staging/` met minimale metadata (herkomst, datum, checksum), idempotent.
- **Waarom nu:** de poort (Fase 2) heeft aanvoer nodig. Instroom vóór zoeken: er moet eerst iets binnenkomen voordat terugvinden waarde heeft.
- **Afhankelijkheden:** Fase 2 (a–c volstaan; 2d niet vereist).
- **Oplevercriteria:** tien echte bestanden via inbox → staging → review → archief zonder handwerk buiten de reviewhandeling; dubbel aangeboden bestand wordt herkend, niet gedupliceerd.
- **Belangrijkste bestanden:** `engine/intake.py` · contract-entry inbox-formaat.
- **Risico's:** formaat-specifieke verwerking per bestandstype toevoegen ("nu we toch bezig zijn"). Grens: verplaatsen + metadata, geen conversie.
- **Gereed wanneer:** de tien-bestanden-test slaagt inclusief duplicaattest.

## 3b. Eén startcommando

- **Doel:** `run.py` — draait in vaste volgorde: intake → scan → validator, en print daarna de actuele staat en de éne volgende stap.
- **Waarom nu:** dit ís principe 7 (één zichtbare volgende stap) in uitvoerbare vorm en de herstartgarantie van A8.4: na weken stilstand is er precies één commando.
- **Afhankelijkheden:** 3a, 1b.
- **Oplevercriteria:** koude start (nieuwe terminal, niets draait) → één commando → systeem actueel + statusregel; faalt een stap, dan stopt de keten met leesbare melding.
- **Belangrijkste bestanden:** `engine/run.py`.
- **Risico's:** orkestratie-drang (scheduling, parallellisme). Grens: sequentieel, handmatig gestart, klaar.
- **Gereed wanneer:** de koude-start-test slaagt.

---

# Fase 4 — Search & Index (B3)

## 4a. Herbouwbare index

- **Doel:** `build_index.py` — bouwt lokale SQLite met full-text search uit het archief (metadata uit catalog + tekstinhoud van leesbare formaten). Volledig herbouwbaar met één commando.
- **Waarom nu:** pas nu is er een gevuld, gereviewd archief om te indexeren. Eerder bouwen = indexeren van ruis.
- **Afhankelijkheden:** Fase 3 (gevulde kernlus).
- **Oplevercriteria:** index verwijderen + herbouwen levert identiek resultaat; herbouwtijd acceptabel voor de werkelijke archiefomvang (bekend uit Fase 0); index staat op een locatie die per contract als "weggooibaar" gemarkeerd is.
- **Belangrijkste bestanden:** `engine/build_index.py` · `index/knowledge.db` (afgeleid).
- **Risico's:** schema-perfectionisme en embeddings-verleiding (beide in de uitstellijst). Grens: FTS over naam, pad en platte tekst.
- **Gereed wanneer:** de weggooien-en-herbouwen-test slaagt.

## 4b. Zoekcommando

- **Doel:** `search.py` — query in, gerangschikte paden + snippets uit.
- **Waarom nu:** dit is het eerste moment waarop de kernmissie ("waar stond dit ook alweer?") aantoonbaar is opgelost.
- **Afhankelijkheden:** 4a.
- **Oplevercriteria:** vijf realistische vragen uit het eigen verleden ("waar staat X?") leveren binnen één zoekopdracht het juiste bestand.
- **Belangrijkste bestanden:** `engine/search.py`.
- **Risico's:** relevantie-tuning zonder einde. Grens: FTS-ranking van SQLite volstaat voor V1.
- **Gereed wanneer:** de vijf-vragen-test slaagt; indexverversing toegevoegd aan `run.py`.

---

# Fase 5 — AI-adapterlaag (B6)

## 5a. Adapterinterface + lokale adapter

- **Doel:** één interface voor AI-aanroepen; eerste implementatie: Ollama (lokaal). Output uitsluitend naar `staging/`.
- **Waarom nu:** de poort bestaat (Fase 2), dus AI-output kan nergens anders heen dan het veilige pad. AI eerder bouwen had het datapad omzeild.
- **Afhankelijkheden:** Fase 2; Fase 4 nuttig maar niet vereist.
- **Oplevercriteria:** één echte AI-aanroep via de adapter levert een voorstel in staging; het voorstel is via de bestaande review-flow te accepteren/afwijzen; nergens in de codebase een directe provider-call buiten de adapter (validatorcontrole).
- **Belangrijkste bestanden:** `engine/ai/adapter.py` · `engine/ai/ollama_adapter.py` · contract-entry AI-output.
- **Risico's:** promptontwerp als tijdvreter (uitstellijst: vrij te itereren). Grens: één werkende prompt, niet de beste.
- **Gereed wanneer:** het voorstel de volledige lus staging → review → archief/afgewezen heeft doorlopen met beslisrecord.

## 5b. Beslismoment dataclassificatie (gate, geen bouwwerk)

- **Doel:** de in de Blueprint vastgelegde gate: vóór de eerste cloud-aanroep wordt aanname 2 bevestigd of aangescherpt (welke data mag naar cloud, welke niet).
- **Waarom nu:** dit is het laatste moment waarop de keuze nog alles beschermt; na de eerste cloud-call is gedeelde data niet terug te halen (bezwaar Enterprise Architect).
- **Afhankelijkheden:** 5a.
- **Oplevercriteria:** één korte notitie met de regel + Decision Log-entry (aanname 2 → geverifieerd).
- **Belangrijkste bestanden:** Decision Log · classificatienotitie in het archief.
- **Risico's:** de gate overslaan "omdat het maar een test is". Niet doen.
- **Gereed wanneer:** de log-entry bestaat. Duur: één zitting of minder.

## 5c. Cloud-adapters

- **Doel:** Claude- en OpenAI-adapter achter dezelfde interface; providerkeuze per taak in configuratie; standaard blijft lokaal.
- **Afhankelijkheden:** 5b (harde gate).
- **Oplevercriteria:** dezelfde taak draait met alleen een configuratiewijziging op een andere provider; classificatieregel technisch afgedwongen (gevoelig gemarkeerde input weigert cloud).
- **Belangrijkste bestanden:** `engine/ai/claude_adapter.py` · `engine/ai/openai_adapter.py` · taakconfiguratie.
- **Risico's:** provider-features gebruiken die de interface breken → lock-in via achterdeur. Grens: alleen wat de interface definieert.
- **Gereed wanneer:** de provider-wissel-test en de weigering-test slagen.

---

# Fase 6 — AI Assistant (eerste echte AI-taken)

## 6a. Eerste taak met leerlus

- **Doel:** één nuttige AI-taak end-to-end, bijv. metadata-/samenvattingsvoorstel per staging-item, waarbij relevante eerdere beslisrecords als context meegaan (B5: zo "leert" het systeem).
- **Waarom nu:** alle onderdelen bestaan; dit is de eerste keer dat archief, index, adapter, staging en review sámen waarde leveren.
- **Afhankelijkheden:** Fasen 2–5.
- **Oplevercriteria:** twintig items verwerkt; aantoonbaar dat een afwijzing met reden het volgende voorstel beïnvloedt (record in de context); elk voorstel herleidbaar tot prompt + records (explainability).
- **Belangrijkste bestanden:** `engine/tasks/propose_metadata.py` (of vergelijkbaar) · contract-entry taakoutput.
- **Risico's:** de leerlus "slim" willen maken (regels afleiden, scoren). Grens voor V1: records meesturen als context, meer niet.
- **Gereed wanneer:** de beïnvloedingstest slaagt.

## 6b. Tweede taaktype

- **Doel:** een tweede, ander soort taak (bijv. duplicaat-/verwantschapssignalering) om te bewijzen dat taak-toevoegen goedkoop is: configuratie + taakscript, geen kernwijziging.
- **Afhankelijkheden:** 6a.
- **Oplevercriteria:** tweede taak gebouwd in ≤1 zitting zónder wijziging aan adapter, contractstructuur of review-flow. Lukt dat niet, dan is dát het signaal dat er een kernprobleem is — noteren, niet omheen bouwen.
- **Belangrijkste bestanden:** tweede taakscript.
- **Risico's:** taken blijven toevoegen. Twee is genoeg voor V1; verdere ideeën → `Future_Ideas.md`.
- **Gereed wanneer:** de één-zitting-test slaagt.

---

# Fase 7 — Connectors

## 7a. Eerste connector

- **Doel:** één externe bron automatisch laten landen in `inbox/` (keuze: de bron met het meeste handmatige sleepwerk op dit moment).
- **Waarom laatst:** een connector is per architectuur niets anders dan "een script dat bestanden in de inbox zet" — alles stroomafwaarts bestaat en is getest. Eerder bouwen had niets opgeleverd.
- **Afhankelijkheden:** Fase 3 (inbox-contract); volledige lus voor de test.
- **Oplevercriteria:** bron → inbox → staging → review → archief → vindbaar via search, zonder handwerk vóór de reviewhandeling; connector is koud startbaar en draait via `run.py`.
- **Belangrijkste bestanden:** `engine/connectors/<bron>.py` · contract-entry.
- **Risico's:** connectorverzamelwoede. Eén connector is de V1-eis; elke volgende alleen bij aantoonbare behoefte.
- **Gereed wanneer:** de end-to-end-test slaagt.

---

# Fase 8 — V1-afsluiting

- **Doel:** formeel vaststellen dat V1 af is en het systeem de succesdefinitie uit `01_Project_Context.md` haalt.
- **Afhankelijkheden:** Fasen 0–7 (2d mag expliciet vervallen zijn).
- **Oplevercriteria — de drie eindtesten:**
  1. **Koude-start-test:** verwijder de volledige index, open een nieuwe sessie, draai `run.py` → systeem volledig werkend en actueel.
  2. **Herstart-test:** raak het systeem ≥2 weken niet aan; meet daarna de tijd tot zinvol werk. Norm: ≤10 minuten, zonder documentatie te herlezen.
  3. **Missie-test:** vijf echte "waar stond dit ook alweer?"-vragen → vijf juiste antwoorden via search.
- **Belangrijkste bestanden:** Decision Log (V1-verklaring) · `Current_State.md` (nieuw milestone).
- **Risico's:** V1 blijven oprekken met "nog één ding". De definitie bovenaan dit plan is de grens; al het overige is V2-input via `Future_Ideas.md`.
- **Gereed wanneer:** de drie testen slagen en de V1-verklaring in het Decision Log staat.

---

## Overzicht: afhankelijkheidsketen

```
F0 → F1a → F1b → F2a → F2b → F2c → [F2d optioneel] → F3a → F3b
                                                        → F4a → F4b
F2c + F5a → F5b → F5c
F4b + F5c → F6a → F6b
F3a (+ volledige lus) → F7a
alles → F8
```

Praktische volgorde blijft strikt sequentieel (spelregel 1); de keten toont alleen wat wat blokkeert.
