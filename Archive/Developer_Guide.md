# KnowledgeOS — Developer Guide

**Hoort bij:** `Architecture/Blueprint.md` (definitief, bevroren 19-07-2026)
**Functie:** de Blueprint in gewone taal, plus de bouwaanpak. Dit document neemt geen beslissingen; die staan in de Blueprint.

---

## 1. Wat de Blueprint betekent, in gewone taal

- **Jouw bestanden zijn de waarheid.** Alles wat KnowledgeOS weet, staat als gewoon bestand op de NAS. Geen enkele database, tool of AI bezit kennis die niet ook als bestand bestaat.
- **Al het andere is weggooibaar.** De zoekindex, caches, tussenresultaten: allemaal afgeleid. Kapot of verouderd? Weggooien en opnieuw opbouwen. Nooit dataverlies, nooit paniek.
- **Niets hoeft te draaien.** Geen servers, geen achtergrondprocessen. Elk onderdeel start koud op vanaf de bestanden. Drie weken niet aangeraakt = niets kapot.
- **AI stelt voor, jij beslist.** AI schrijft alleen naar een wachtruimte (staging). Alleen jouw expliciete reviewhandeling maakt iets tot kennis in het archief. Elke beslissing wordt zelf ook een bestand.
- **Python is het gereedschap.** Losse, klein houdbare programma's onder één set afspraken. Modules praten met elkaar via bestanden op vaste plekken — niet via API's.

## 2. Hoe je vanaf nu over KnowledgeOS denkt

Eén mentaal model, vier delen:

- **Archief** (NAS-bestanden) = het brein. Permanent, heilig.
- **Index** = de geheugensteun. Handig, weggooibaar.
- **Engine** (Python-scripts) = de handen. Doen werk, bezitten niets.
- **Review** = de poort. Alles wat AI maakt, gaat door jou heen.

Toets bij elk idee: *verlaagt dit de drempel om het systeem na drie weken stilstand weer op te pakken?* Zo nee: niet doen, of parkeren in `Future_Ideas.md` (één regel, geen uitwerking).

## 3. Hoe we beginnen

- **Nu:** bouw en draai `scan_archive.py` (Blueprint §G3). Scant het archief, schrijft `index/catalog.jsonl`, print het aantal bestanden. Eén zitting.
- Dit ene script doet drie dingen tegelijk: het levert direct een doorzoekbare inventaris, het verifieert dat de NAS-mount werkt (aanname 1) en het toont de echte omvang van het archief (aanname 3).
- De outputlocatie wordt de eerste regel van het contractdocument. Daarmee bestaat óók dat document vanaf dag één met echte inhoud.
- Faalt de mount: niet fixen-en-vergeten. Noteer het als nieuw feit in het Decision Log — dat is precies waarvoor het log bestaat.

## 4. Bouwvolgorde en waarom

1. **`scan_archive.py`** — eerste index, verifieert aannames, direct bruikbaar.
2. **Module 1: Review Workbench (nieuw)** — implementeert het reviewcontract: staging lezen → menselijk besluit → beslisrecord + verplaatsing naar archief. Eerst, omdat élke latere AI-functie deze poort nodig heeft.
3. **Inbox Automation** — nieuwe input laten landen in staging, klaar voor review.
4. **Search** — bouwt op de index; pas zinvol als er iets te doorzoeken valt.
5. **AI Assistant en Connectors** — laatst; zij consumeren alles hierboven.

Logica van de volgorde: eerst zien wat er is (1), dan de poort (2), dan instroom (3), dan terugvinden (4), dan intelligentie (5). Elke stap levert zelfstandig iets bruikbaars op.

**Vaste bouwregel:** een module bestaat pas als hij minstens één echt bestand verwerkt heeft. Tot die tijd: geen mappen, geen templates, geen documentatie ervoor.

## 5. Fouten die je absoluut moet voorkomen

- **De Blueprint heropenen zonder nieuw feit.** Twijfel, onvrede of een beter idee telt niet. Alleen een nieuw feit, vastgelegd in het Decision Log.
- **Ontwerpen in plaats van bouwen.** Vroege signalen: nieuwe templates zonder inhoud, prompts die prompts verbeteren, mappen hernoemen, meer werk in `Foundation/` dan in code. Herken het, stop, ga terug naar de ene volgende stap in `Current_State.md`.
- **De oude HTML-workbench de kern laten worden.** Die is erfenis. Niets nieuws mag erop bouwen.
- **AI-output rechtstreeks in het archief laten schrijven.** Ook niet "even snel", ook niet via een script dat het automatisch doorschuift. De menselijke reviewhandeling is de enige route.
- **Scripts buiten de conventieset.** Eén losse uitzondering is het begin van de wildgroei waar dit project juist tegen gebouwd is.
- **Een nieuwe tool of nieuw systeem willen bouwen onder stress.** Dat is het patroon uit WHY.md, niet de oplossing. Parkeren in `Future_Ideas.md`, doorwerken aan de volgende stap.
- **Gevoelige data naar cloud-AI sturen vóór de classificatieregel bevestigd is.** Standaard is lokaal (Ollama). Cloud is bewust opt-in, per taak.

---

**Kern in één zin:** bestanden zijn de waarheid, jij bent de poort, en er is altijd precies één volgende stap.
