# KnowledgeOS Architecture Blueprint

**Status:** Definitief — bevroren na eindreview (zie §G1 en Decision Log)
**Datum:** 19-07-2026 · Eindreview: 19-07-2026
**Functie van dit document:** vastleggen van de architectuurbeslissingen die na maanden bouwen niet meer terug te draaien zijn. Niets anders.
**Detailregel:** elk onderdeel wordt behandeld op het niveau dat nodig is om de zes beslissingen (Deel B) te onderbouwen. Dunne secties zijn correct gedrag.

---

# Deel A — Analyse

## A1. Probleemherformulering

**Het fundamentele probleem:** kennis, context en beslissingen bestaan, maar zijn niet terugvindbaar en niet herbruikbaar op het moment dat ze nodig zijn. Het gevolg is herhaald opstartwerk, contextverlies en cognitieve uitputting. Het probleem wordt versterkt door ADHD: elke drempel om het systeem opnieuw op te pakken vergroot de kans dat het systeem zelf ook een verlaten project wordt.

**Wat het probleem niet is:**

- Geen opslagprobleem. De NAS bewaart alles al.
- Geen tekort aan tools. Er zijn eerder te veel losse tools.
- Geen AI-probleem. AI is een middel; het probleem bestond vóór AI.
- Geen reviewprobleem. De Review Workbench loste één deelprobleem (CSV-review) op onder een ouder, smaller probleemframe.

**Uitkomst:** de architectuur moet worden ontworpen rond *terugvindbaarheid, contextbehoud en herstartbaarheid* — niet rond opslag, niet rond review, niet rond AI.

## A2. Informatiegaten

| # | Ontbrekende informatie | Blokkeert een van de zes beslissingen? |
|---|---|---|
| 1 | Exact datavolume en aantal bestanden op de NAS | **Nee** — ordegrootte volstaat (aanname 3) |
| 2 | Volledige lijst bestandsformaten in het archief | Nee |
| 3 | Frequentie waarmee reviews plaatsvinden | Nee |
| 4 | Is de NAS betrouwbaar bereikbaar vanaf het werkstation (mount)? | **Ja** — raakt Beslissing 3 (SSoT-locatie). Afgedekt via aanname 1 |
| 5 | Privacy-/kosteneisen voor cloud-AI per datacategorie | **Ja** — raakt Beslissing 6. Afgedekt via aanname 2 |
| 6 | Back-upstrategie van de NAS | Nee — operationeel, implementatiefase |
| 7 | Toekomstige UI-vorm (web, desktop, CLI) | Nee — expliciet uitgesteld (§G2) |

Items met "nee" worden in de rest van deze Blueprint genegeerd.

## A3. Aannames

| # | Aanname | Status | Effect |
|---|---|---|---|
| 1 | De NAS is als netwerk-mount betrouwbaar bereikbaar vanaf het primaire werkstation | **Onbevestigd** | Verlaagt betrouwbaarheidsklasse van Beslissing 3 naar "hoog, mits aanname klopt" |
| 2 | Gevoelige persoonlijke data mag niet naar cloud-AI; niet-gevoelige data wel | **Onbevestigd** | Verlaagt betrouwbaarheidsklasse van Beslissing 6 |
| 3 | Ordegrootte archief: ≤ ~1 miljoen bestanden, ≤ enkele TB | **Onbevestigd** | Onderbouwt keuze voor lichte, herbouwbare index (A9) |
| 4 | Eén gebruiker; geen multi-user, geen gelijktijdige schrijvers | **Geverifieerd** (projectdocumenten) | Onderbouwt afwezigheid van servicelaag |
| 5 | Python-vaardigheid is aanwezig en blijft de werktaal | **Geverifieerd** (bestaande scripts, NAS V2.2) | Onderbouwt Beslissing 2 |

**Verificatiepad per onbevestigde aanname (eindreview):**

- Aanname 1 en 3 worden geverifieerd door de eerste implementatiestap (§G3): slaagt de scan, dan is de mount bevestigd én is de ordegrootte van het archief bekend. Faalt de scan, dan is dat een nieuw feit voor het Decision Log.
- Aanname 2 vereist geen bouwwerk maar één beslismoment: vóór de eerste cloud-AI-aanroep wordt de dataclassificatieregel expliciet bevestigd of aangescherpt. Tot dat moment geldt de veilige standaard uit Beslissing 6 (alles lokaal). De architectuur is daarmee robuust ongeacht de uitkomst van deze aanname.

## A4. Legacy-analyse

Toetsvraag per component: fundament, of artefact van het oude probleemframe dat nu ten onrechte structuurbepalend dreigt te worden?

| Component | Oordeel | Motivering |
|---|---|---|
| NAS V2.2 | **Fundament** | Duurzame opslaglaag; onafhankelijk van elk probleemframe |
| Python-automatisering | **Fundament** | Bewezen runtime, aanwezige vaardigheid, geen extra afhankelijkheid |
| Human Review Workbench (HTML-tool) | **Erfenis — af te bouwen** | Gebouwd voor één CSV-bestand onder het oude frame. Het *reviewprincipe* is waardevol (zie Beslissing 5); de huidige implementatie mag de structuur niet bepalen |
| AI-migratiescripts | **Bruikbaar, niet structuurbepalend** | Eenmalige werktuigen; leveren conventie-lessen, geen architectuur |
| Ollama (lokaal) | **Bruikbaar, niet structuurbepalend** | Provider, hoort achter een adapter (Beslissing 6) |
| Claude / OpenAI (cloud) | **Bruikbaar, niet structuurbepalend** | Idem |

**Kernconclusie:** het grootste architectuurrisico is dat de Review Workbench — het meest zichtbare werkende artefact — impliciet de kern wordt. Dat is expliciet afgewezen in Beslissing 1.

## A5. Ontwerpprincipes (afgeleid uit A1–A4)

1. **Bestanden boven databases** (uit A1: duurzaamheid en herstartbaarheid; uit A4: NAS is fundament). Kennis leeft in open bestandsformaten; alles wat afgeleid is, is weggooibaar.
2. **Alles afgeleide is herbouwbaar** (uit A1: vertrouwen; uit A3.3). Index kapot = index opnieuw bouwen, nooit dataverlies.
3. **Niets draait permanent** (uit A1: herstartbaarheid na weken stilstand). Geen daemons, geen services die "aan" moeten staan om het systeem geldig te houden.
4. **Legacy bepaalt geen structuur** (uit A4). Bestaande tools zijn bouwstenen op afroep, geen dragende muren.
5. **AI aan de rand, mens in het pad** (uit A1 + missiedocumenten). AI produceert voorstellen; alleen een menselijke handeling maakt iets tot kennis.
6. **Conventie boven code** (uit A3.5 + A4). Structuur wordt afgedwongen door afgesproken naamgeving en locaties, gevalideerd door lichte scripts — niet door frameworks.
7. **Eén zichtbare volgende stap** (uit A1, ADHD-constraint). Het systeem zelf bevat altijd de actuele staat en precies één volgende actie (bestaand mechanisme: `Foundation/02_Current_State.md`).

## A6. Architectuurvergelijking

Drie kandidaten, beoordeeld op dezelfde criteria: herstartdrempel na 3 weken stilstand · duurzaamheid van kennis · onderhoudslast · schaalbaarheid (data/modules/frequentie) · AI-integreerbaarheid · omkeerbaarheid.

**Kandidaat A — Minimaal: "bestanden + losse scripts."** Alles is een bestand op de NAS; zoeken via bestandsnamen en handmatig doorzoeken; scripts ad hoc.
Herstartdrempel: zeer laag. Onderhoudslast: zeer laag. **Faalt op de kernmissie:** terugvindbaarheid en contextbehoud schalen niet voorbij enkele duizenden bestanden zonder index; precies het probleem uit A1 blijft bestaan.

**Kandidaat B — File-core met afgeleide index (gekozen).** Bestanden op de NAS zijn de Single Source of Truth. Eén lokale, volledig herbouwbare index (SQLite met full-text search) maakt zoeken en context mogelijk. Modules zijn los startbare Python-programma's die communiceren via bestandssysteem-contracten. AI zit achter een adapterlaag en schrijft uitsluitend naar een staging-gebied.
Herstartdrempel: laag (niets hoeft te draaien; index desnoods herbouwen). Duurzaamheid: maximaal. Onderhoudslast: laag. Schaalt binnen aanname 3 ruim.

**Kandidaat C — Database-/service-centrisch.** Centrale database als SSoT, interne API-laag, services, webapplicatie.
Sterkste zoek- en UI-mogelijkheden, maar: kennis raakt gekoppeld aan schema en tooling; het systeem is "kapot" wanneer de services niet draaien; onderhoudslast structureel hoog. Herstartdrempel na stilstand: hoog (migraties, dependencies, draaiende onderdelen). **ADHD Design Specialist-veto** (zie Deel C).

**Keuze: Kandidaat B.** A verworpen omdat het de missie niet haalt; C verworpen omdat het de ADHD-toets faalt en kennis aan tooling koppelt.

## A7. Trade-offs van de gekozen architectuur

Wat wordt opgegeven met Kandidaat B:

- **Realtime-gedrag.** Batchgewijze verwerking; geen live-synchronisatie tussen modules. Aanvaard: geen enkel scenario in A1 vereist realtime.
- **Rijke query-mogelijkheden van een relationeel schema als bron.** De index kan veel, maar de bron blijft plat. Aanvaard: de index mag rijk zijn, zolang hij weggooibaar blijft.
- **Strakke afdwinging van datamodellen.** Bestanden dwingen minder af dan databaseschema's. Tegenmaatregel: conventievalidatie door de Automation Engine (A8, risico 2).
- **Kant-en-klare UI.** Uitgesteld; genoteerd bezwaar van UX Designer in Deel C.

## A8. Risico's en tegenmaatregelen (in de architectuur zelf)

| # | Risico | Type | Tegenmaatregel in de architectuur |
|---|---|---|---|
| 1 | Index loopt uit de pas met bestanden | Technisch | Index is per definitie herbouwbaar; bij twijfel wint het bestand; herbouw is één commando |
| 2 | Conventie-erosie (naamgeving, locaties verwateren) | Technisch/cognitief | Eén contractdocument in het archief; validatiescript als vast onderdeel van de engine, meldt afwijkingen |
| 3 | AI-output sluipt ongereviewd het archief in | Veiligheid | AI heeft alleen schrijfrecht op het staging-gebied; verplaatsing naar archief kan uitsluitend via een menselijke reviewhandeling (Beslissing 5+6) |
| 4 | Systeem vervalt tijdens weken stilstand | Cognitief | Geen draaiende onderdelen; actuele staat + één volgende stap in `Current_State.md`; alles start koud vanaf bestanden |
| 5 | Ontwerpen vervangt bouwen | Cognitief | Zie Deel E — verplichte faalmodus, met structurele tegenmaatregel |
| 6 | Provider-lock-in (AI) | Operationeel | Adapterlaag; providerkeuze per taak configureerbaar (Beslissing 6) |

## A9. Schaalbaarheid

- **Datavolume:** binnen aanname 3 (≤ ~10⁶ bestanden) is SQLite + FTS ruim toereikend; het model schaalt tot vele miljoenen indexrecords. Overschrijding heropent geen SSoT-beslissing — alleen de indextechnologie, die weggooibaar is.
- **Moduleaantal:** bestandssysteem-contracten schalen lineair. Risico bij >~10 modules is contract-wildgroei; tegenmaatregel is het ene contractdocument (A8, risico 2).
- **Gebruiksfrequentie:** batchmodel dekt dagelijks tot continu handmatig gebruik. Zou realtime ooit nodig blijken, dan is dat een expliciete heropening (§G1) — geen sluipende toevoeging.
- Gebruikersaantal is per missie geen schaal-as (aanname 4).

---

# Deel B — De zes beslissingen

## Beslissing 1 — De Review Workbench is een module, niet de kern

- **Gekozen optie:** de kern van KnowledgeOS is het Knowledge Archive (bestanden op de NAS) plus het Knowledge Model (de conventielaag daaroverheen). De Review Workbench is één vervangbare module die op die kern werkt.
- **Verworpen alternatieven:** *Workbench als kern* — dit zou het oude probleemframe (CSV-reviewtool) structuurbepalend maken (A4); alle latere modules zouden dan afhangen van een artefact dat voor een ander probleem is gebouwd.
- **Omkeerbaarheidsklasse:** duur omkeerbaar. Wordt de workbench toch kern, dan groeit elke module eraan vast en is latere ontvlechting maandenwerk.
- **Wat hierdoor vastligt:** Search, AI Assistant, Automation Engine en Connectors bouwen op het archief en de index — nooit op de workbench. De huidige HTML-workbench wordt afgebouwd en vervangen door een module die het reviewcontract van Beslissing 5 implementeert.
- **Onderbouwing:** A1 (het probleem is terugvindbaarheid, niet review), A4 (workbench = erfenis van oud frame), principe 4.

## Beslissing 2 — Python is de vaste Automation Engine

- **Gekozen optie:** Python is niet "tijdelijk script" maar de permanente automation-runtime. De Automation Engine is een verzameling los startbare Python-programma's onder één gedeelde conventieset (logging, configuratie, foutafhandeling, contractvalidatie).
- **Verworpen alternatieven:** *Workflow-tooling (n8n e.d.)* — permanente draaiende dienst, strijdig met principe 3, ADHD-veto-categorie; *herbouw in ander ecosysteem* — vernietigt bewezen fundament (A4) zonder probleem op te lossen; *status "tijdelijk" handhaven* — leidt aantoonbaar tot scriptwildgroei zonder conventies, precies de versnippering uit WHY.md.
- **Omkeerbaarheidsklasse:** duur omkeerbaar. Na tientallen modules is een runtime-wissel een herbouw.
- **Wat hierdoor vastligt:** alle modules en connectors zijn Python; de conventieset is bindend voor elke nieuwe module; vaardigheden en bestaande scripts blijven bruikbaar.
- **Onderbouwing:** A3.5 (geverifieerd), A4 (fundament-oordeel), principes 3 en 6.

## Beslissing 3 — Single Source of Truth: de bestanden op de NAS

- **Gekozen optie:** de SSoT is het bestandsarchief op de NAS, in open formaten (Markdown voor kennis en beslisrecords, originelen ongewijzigd ernaast). De **indexdatabase** is een afgeleide, lokaal herbouwbare cache. Het **Knowledge Model** is géén aparte opslag maar de logische laag: naamgevings-, locatie- en metadataconventies over de bestanden heen.
- **Verworpen alternatieven:** *Indexdatabase als SSoT* — koppelt alle kennis aan een schema en een draaiend stuk software; corrupt of verouderd = kennis onbereikbaar; onacceptabel tegen A1. *Knowledge Model als aparte graph-store* — **Staff Engineer-veto**: geen concreet implementatiepad binnen de huidige middelen; opgelost door het model als conventielaag te definiëren (Deel C).
- **Omkeerbaarheidsklasse:** praktisch onomkeerbaar. Zodra jaren kennis in deze vorm staat, is migratie naar een ander SSoT-paradigma het duurste denkbare project.
- **Wat hierdoor vastligt:** elke module leest en schrijft bestanden; de index mag altijd worden weggegooid en herbouwd; back-up = bestandsback-up; geen enkele module mag data bezitten die niet als bestand in het archief bestaat.
- **Onderbouwing:** A1 (duurzaamheid, herstartbaarheid), A3.1 (mits mount-aanname klopt — betrouwbaarheidsklasse: hoog onder aanname), A4 (NAS = fundament), principes 1–2.

## Beslissing 4 — Modules communiceren via bestandssysteem-contracten

- **Gekozen optie:** modules wisselen uitsluitend uit via afgesproken bestandslocaties en -formaten (o.a. inbox → staging → archief; index als leesbaar afgeleid product). Verwerking is batchgewijs; elke module is koud startbaar. Alle contracten staan in één contractdocument in het archief.
- **Verworpen alternatieven:** *Interne API's/services* — vereist draaiende processen, strijdig met principe 3 (ADHD-veto-categorie, zie Deel C); *message bus/queue* — infrastructuur zonder probleem: er is één gebruiker en geen realtime-eis (A7).
- **Omkeerbaarheidsklasse:** duur omkeerbaar. Contracten verspreiden zich door alle modules; wisselen naar API's later betekent elke module aanpassen.
- **Wat hierdoor vastligt:** elke toekomstige module (Inbox Automation, Search, AI Assistant, Connectors) definieert zijn in- en output als bestanden en registreert dat in het contractdocument; volgordelijkheid ontstaat door bestandsstaat, niet door orkestratie.
- **Onderbouwing:** A1 (herstartbaarheid), A3.4 (één gebruiker), principes 2–3, veto-afhandeling in Deel C.

## Beslissing 5 — Het systeem leert via expliciete, append-only beslisrecords

- **Gekozen optie:** elke menselijke review levert een gestructureerd, leesbaar beslisrecord op (wat werd voorgesteld, wat werd besloten, waarom), append-only opgeslagen in het archief zelf — naast de content waarop het betrekking heeft. "Leren" betekent: AI-stappen krijgen relevante beslisrecords als context en afgeleide regels mee. Geen finetuning, geen impliciet leren.
- **Verworpen alternatieven:** *Model-finetuning* — geen implementatiepad, niet uitlegbaar, koppelt geleerde kennis aan één provider (strijdig met Beslissing 6); *reviews alleen in de workbench-opslag* — koppelt kennis aan één module, strijdig met Beslissing 1 en 3.
- **Omkeerbaarheidsklasse:** duur omkeerbaar. Het recordformaat-principe (plain, append-only, in de SSoT) draagt alle toekomstige review- en AI-modules; de exacte velden zijn implementatie en blijven vrij.
- **Wat hierdoor vastligt:** elk AI-voorstel doorloopt review vóór het archief; reviewuitkomsten zijn zelf kennis in de SSoT; explainability is structureel (elke geleerde regel is herleidbaar tot records).
- **Onderbouwing:** A1 (beslissingen vastleggen was een kernvraag: "waarom had ik deze keuze gemaakt?"), A4 (reviewprincipe behouden, implementatie afbouwen), principes 1 en 5.

## Beslissing 6 — AI-providers zitten achter één adapterlaag, buiten het datapad van de SSoT

- **Gekozen optie:** alle AI-aanroepen (Ollama, Claude, OpenAI, toekomstige providers) lopen via één adapterlaag binnen de Automation Engine. Providerkeuze is per taak configureerbaar. AI schrijft uitsluitend naar het staging-gebied; alleen de menselijke reviewhandeling verplaatst iets naar het archief. Dataclassificatieregel: gevoelige data uitsluitend lokaal (Ollama).
- **Verworpen alternatieven:** *Directe provider-calls in losse scripts* — lock-in en onvervangbaarheid, aantoonbaar duur bij providerwissel; *AI met schrijfrechten op het archief* — schendt principe 5 en de missie ("mens in controle").
- **Omkeerbaarheidsklasse:** duur omkeerbaar. Verspreide directe calls zijn later overal terug te vinden en te herschrijven; de adapterlaag voorkomt dat.
- **Wat hierdoor vastligt:** providers zijn verwisselbaar zonder module-aanpassingen; het mens-gecontroleerde datapad (staging → review → archief) geldt voor elke huidige en toekomstige AI-functie.
- **Onderbouwing:** A3.2, A4 (providers niet structuurbepalend), principes 5 en 4. Missiedocument: "Technologie is een middel, nooit het doel." Eindreview: omdat lokaal de standaard is en cloud opt-in per taak, blijft deze beslissing geldig bij élke uitkomst van aanname 2 — betrouwbaarheidsklasse daarmee: hoog.

**Filterregel toegepast:** geen van de zes is goedkoop omkeerbaar; alle zes horen in de Blueprint. Goedkoop omkeerbare deelkeuzes die tijdens de analyse opkwamen zijn verwijderd en staan in de Uitstellijst (§G2).

---

# Deel C — Multidisciplinaire review

- **Software Architect** — Bezwaar: bestandssysteem-contracten creëren impliciete koppeling; zonder discipline ontstaat "iedereen leest overal". Oplossing (geaccepteerd): één bindend contractdocument in het archief plus een validatiescript in de engine (A8.2). Bezwaar opgelost.
- **UX Designer** — Bezwaar: een file-core zonder UI-visie levert op termijn een systeem dat alleen via terminal bruikbaar is, wat de drempel juist verhoogt. Toets aan scope-slot: welk van de zes beslissingen blokkeert dit? Geen — UI raakt geen onomkeerbare keuze. Bezwaar geregistreerd; UI expliciet uitgesteld (§G2). Genoteerd als geldig aandachtspunt voor Module 1.
- **Information Architect** — Bezwaar: zonder afgedwongen naamgeving en minimale metadata verrot de vindbaarheid, en daarmee de hele missie. Oplossing (geaccepteerd): het Knowledge Model ís de conventielaag (Beslissing 3) en conventievalidatie is een vast engine-onderdeel — geen optioneel goed voornemen.
- **AI Safety Engineer** — Bezwaar: een staging-gebied beschermt niet als scripts AI-output automatisch mogen doorschuiven. Oplossing (geaccepteerd, aangescherpt in Beslissing 6): verplaatsing van staging naar archief kan uitsluitend via een menselijke reviewhandeling; geen enkele module krijgt die bevoegdheid geautomatiseerd.
- **Product Strategist** — Bezwaar: risico dat het systeem pas "waarde" levert als alles af is, wat motivatie doodt. Oplossing (geaccepteerd): de eerste implementatiestap (§G3) levert direct een bruikbaar product op (doorzoekbare inventaris), geen infrastructuur.
- **Enterprise Architect** — Bezwaar: cloud-AI zonder dataclassificatie is een privacyrisico dat later niet te repareren is (data is dan al gedeeld). Oplossing (geaccepteerd): classificatieregel opgenomen in Beslissing 6; standaard is lokaal, cloud is opt-in per taak.

**Uitgesproken veto's** (tevens in Decision Log):

1. **ADHD Design Specialist — veto** op elke architectuur met permanent draaiende onderdelen (kandidaat C; interne API's; workflow-daemons). Grond: structurele onderhoudslast en herstartdrempel. **Opgelost** met alternatief: batchgewijze filesystem-contracten (Beslissing 4).
2. **Staff Engineer — veto** op een aparte graph-database als drager van het Knowledge Model. Grond: geen concreet implementatiepad binnen huidige middelen en vaardigheden. **Opgelost** met alternatief: Knowledge Model als conventielaag over bestanden, met de herbouwbare index als query-laag (Beslissing 3).

**Consensus:** bereikt op Kandidaat B met bovenstaande aanscherpingen. Scope is tijdens de review niet vergroot.

---

# Deel D — Verplichte secties (onder de detailregel)

- **Productvisie & Missie.** Vastgelegd in `README.md` en `WHY.md`; deze Blueprint herhaalt ze niet. Eén zin: extern brein dat cognitieve belasting verlaagt en de mens in controle houdt.
- **Ontwerpfilosofie & Design Principles & Cognitive Principles.** De zeven principes in A5 vervangen en verdiepen `Foundation/Design_Principles.md` (simpel · modulair · iteratief blijven geldig als samenvatting). Cognitief kernprincipe: elke keuze wordt getoetst op de herstartdrempel na drie weken stilstand.
- **Information Architecture & Knowledge Model.** Het Knowledge Model is de conventielaag uit Beslissing 3: afgesproken locaties, naamgeving en minimale metadata over de bestanden. Veldniveau-uitwerking is implementatiefase.
- **Module Architecture.** Modules zijn los startbare Python-programma's op de kern (archief + index), gekoppeld via contracten (Beslissing 4). Moduleontwerp per module: implementatiefase, via `Architecture/Module_Template.md`.
- **Data Architecture.** SSoT = bestanden (Beslissing 3); index = afgeleid; staging-gebied voor ongereviewde input. Schema's: implementatiefase.
- **AI Architecture.** Adapterlaag, staging, dataclassificatie (Beslissing 6). Promptbeheer: implementatiefase (`Prompts/`).
- **Automation Architecture.** Python-engine met conventieset en validator (Beslissing 2). Concrete tooling: implementatiefase.
- **UX Philosophy.** Eén principe ligt vast: het systeem toont altijd de actuele staat en één volgende stap. Al het overige UI-werk: uitgesteld (§G2), met het genoteerde UX-bezwaar uit Deel C.
- **Security & Privacy.** Lokaal-eerst beperkt het aanvalsoppervlak; gevoelige data verlaat het lokale domein niet (Beslissing 6). Verdere hardening (NAS-toegang, back-upencryptie): implementatiefase.
- **Local First & Offline First.** Structureel gegarandeerd door Beslissingen 3 en 4: zonder netwerk en zonder cloud blijft alles leesbaar en herbouwbaar. Geen aparte maatregelen nodig.
- **Explainability, Human-in-the-loop & Safety.** Structureel gegarandeerd door Beslissingen 5 en 6: elk AI-resultaat is herleidbaar tot voorstel + menselijk beslisrecord; geen automatische route naar het archief.
- **Scalability, Maintainability & Extensibility.** Zie A9. Uitbreidbaarheid = nieuwe module + contractregistratie; geen kernwijziging nodig.
- **Future Vision & Product Roadmap.** Volgorde na bevriezing: Module 1 Review Workbench (nieuw, op het reviewcontract) → Inbox Automation → Search → AI Assistant → Connectors. Detailroadmap: implementatiefase; ideeën parkeren in `Architecture/Future_Ideas.md`.
- **Decision Log.** Zie Deel F; wordt voortgezet in `Architecture/Decision_Log.md`.

---

# Deel E — Anti-patterns, faalmodi en bewuste niet-keuzes

**Anti-patterns en verleidingen.** Feature creep ("nu we toch bezig zijn…"), scope creep via tooling-nieuwsgierigheid, architectural debt door "tijdelijke" uitzonderingen op de contracten, technical debt door scripts buiten de conventieset, UX debt door terminal-only groei, cognitive debt door documentatie die beslissingen beschrijft zonder ze te nemen. ADHD-traps specifiek: nieuwe-tool-dopamine, systeemdrang onder stress, en perfectioneren van structuur als vermijding van inhoud.

**Waarom vergelijkbare systemen falen.** Persoonlijke kennissystemen (PKM-tools, Zettelkasten-implementaties, home-grown wiki's) falen zelden op opslag en bijna altijd op één van drie punten: (1) de herstartdrempel na stilstand is te hoog (draaiende onderdelen, vergeten conventies), (2) kennis raakt gevangen in één tool en migratie wordt nooit gedaan, (3) het onderhouden van het systeem wordt het hobby-doel in plaats van het gebruiken ervan. Beslissingen 3 en 4 adresseren (1) en (2); de faalmodus hieronder adresseert (3).

**Verplichte faalmodus: ontwerpen vervangt bouwen.**
*Mechanisme:* ontwerpen geeft dezelfde beloning als bouwen (voortgangsgevoel, nieuwheid) zonder faalrisico. Onder ADHD is dit versterkt: een nieuw schema is dopamine, een half werkend script is frictie. Het systeem-over-het-systeem groeit; het systeem zelf niet.
*Vroege signalen:* nieuwe templates of mappen zonder inhoud; prompts die prompts verbeteren; hernoemen en herstructureren van bestaande documenten; heropenen van deze Blueprint zonder nieuw feit; meer commits in `Foundation/` en `Prompts/` dan in code.
*Tegenmaatregel — in de architectuur zelf, niet in discipline:*
1. **Bestaansregel:** een module bestaat architecturaal pas wanneer hij minstens één echt bestand uit het archief verwerkt heeft. Tot dat moment mag er geen mapstructuur, template of documentatie voor worden aangemaakt. Verduidelijking (eindreview): de contractregistratie uit Beslissing 4 telt als onderdeel van de eerste bouwzitting — bouwen dus, geen documenteren vooraf — en is maximaal één alinea per module.
2. **Bevriezing:** wijziging van deze Blueprint vereist een Decision Log-entry met het nieuwe feit dat de heropening rechtvaardigt (§G1). Herformuleren zonder nieuw feit is per definitie ongeldig.
3. **Parkeerplek:** ideeën gaan naar `Future_Ideas.md` — één regel, geen uitwerking. Uitwerken van een geparkeerd idee telt als heropening.

**Bewust niet gemaakt keuzes (en waarom):** geen UI-technologie (raakt geen onomkeerbare beslissing), geen indexschema op veldniveau (index is weggooibaar), geen promptarchitectuur (vrij te itereren), geen back-uptooling (operationeel), geen keuze van specifieke AI-modellen (adapter maakt dit triviaal wisselbaar).

**Moeilijk terug te draaien (bewaakt):** SSoT-paradigma (Beslissing 3), het mens-gecontroleerde datapad (5+6), de runtimekeuze (2).

**Bewust flexibel gehouden:** indextechnologie, providerkeuze per taak, beslisrecord-velden, module-interne opbouw, UI-vorm.

---

# Deel F — Decision Log

| Datum | Besluit / gebeurtenis | Klasse |
|---|---|---|
| 19-07-2026 | B1: Review Workbench = module; kern = archief + Knowledge Model | Duur omkeerbaar |
| 19-07-2026 | B2: Python = vaste Automation Engine met bindende conventieset | Duur omkeerbaar |
| 19-07-2026 | B3: SSoT = bestanden op NAS; index afgeleid; Knowledge Model = conventielaag | Praktisch onomkeerbaar |
| 19-07-2026 | B4: modulecommunicatie via bestandssysteem-contracten, batchgewijs | Duur omkeerbaar |
| 19-07-2026 | B5: leren via append-only beslisrecords in de SSoT; geen finetuning | Duur omkeerbaar |
| 19-07-2026 | B6: AI achter adapterlaag; staging → menselijke review → archief; gevoelig = lokaal | Duur omkeerbaar |
| 19-07-2026 | **Veto ADHD Design Specialist** op permanent draaiende onderdelen — opgelost via B4 | Veto, opgelost |
| 19-07-2026 | **Veto Staff Engineer** op graph-database als Knowledge Model-drager — opgelost via B3 | Veto, opgelost |
| 19-07-2026 | Huidige HTML-workbench geclassificeerd als erfenis, af te bouwen | Legacy-oordeel |
| 19-07-2026 | Eindreview afgerond: geen nieuwe beslissingen, geen scopewijziging. Verduidelijkingen: verificatiepad aannames (A3), betrouwbaarheid B6, bestaansregel vs. contractregistratie (Deel E), contractdocument gekoppeld aan §G3 | Review — Blueprint definitief |

Voortzetting van dit log in `Architecture/Decision_Log.md`.

---

# Deel G — Afsluiting

## G1. Bevriezingsverklaring

De zes beslissingen (Deel B) zijn genomen en bevroren per 19-07-2026. Wijziging vereist expliciete heropening: een Decision Log-entry die het nieuwe feit benoemt dat de heropening rechtvaardigt, plus herbeoordeling van uitsluitend de geraakte beslissing. Onvrede, twijfel of een beter idee zonder nieuw feit is geen geldige heropeningsgrond.

## G2. Uitstellijst (bewust doorgeschoven naar implementatie)

| Onderwerp | Reden dat uitstel goedkoop is |
|---|---|
| UI-vorm en -technologie van de Review Workbench | Bouwt op het reviewcontract; wisselbaar zonder kernwijziging |
| Indexschema en zoektechniek (FTS-configuratie, embeddings) | Index is per B3 weggooibaar en herbouwbaar |
| Velden van het beslisrecord | Append-only plain text laat evolutie toe |
| Promptbeheer en promptversiebeheer | Raakt geen contract of datapad |
| Back-up- en hardeningdetails NAS | Operationeel; wijzigt geen architectuur |
| Concrete mappenstructuur van staging/inbox | Vastgelegd in het contractdocument bij eerste gebruik |
| Modelkeuze per AI-taak | Per B6 per taak configureerbaar |

## G3. Eerste implementatiestap

**Bouw en draai `scan_archive.py`** — het eerste onderdeel van de Automation Engine (B2) dat de eerste afgeleide index oplevert (B3):

- Scant de archiefmap op de NAS-mount.
- Schrijft per bestand één regel (pad, naam, extensie, grootte, wijzigingsdatum) naar `index/catalog.jsonl`.
- Print als afsluiting het totaal aantal gevonden bestanden.

Uitvoerbaar binnen één zitting. Geen voorbereidingstaak, geen onderzoekstaak. Het resultaat is direct bruikbaar (doorzoekbare inventaris) en verifieert tegelijk aannames 1 en 3 (NAS-bereikbaarheid en ordegrootte): faalt de mount, dan is dat het eerste nieuwe feit voor het Decision Log.

De outputlocatie `index/catalog.jsonl` vormt de eerste regel van het contractdocument (B4) — daarmee bestaat het contractdocument vanaf dag één met echte inhoud, niet als lege structuur.
