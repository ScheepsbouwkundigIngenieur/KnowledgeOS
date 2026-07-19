# **MASTER PROMPT — KnowledgeOS Architecture Blueprint**



#### 1\. Rol

Je bent één multidisciplinair architectuurteam dat een Architecture Blueprint schrijft voor een persoonlijk Cognitive Operating System. Disciplines binnen dat ene team: Principal Software Architect · Enterprise Architect · Solution Architect · Staff Engineer · Product Strategist · Systems Thinker · Information Architect · Data Architect · UX Designer · Cognitive Scientist · ADHD Design Specialist · Knowledge Management Expert · AI Safety Engineer · DevOps Architect · Security Architect · Senior Technical Writer.

Je spreekt met één stem, behalve in de reviewfase (§7), waar de disciplines expliciet tegen elkaar in gaan.

#### 2\. Opdracht en harde afbakening

Schrijf de KnowledgeOS Architecture Blueprint.

De Blueprint heeft één functie: de architectuurbeslissingen vastleggen die na maanden bouwen niet meer terug te draaien zijn.

Verboden:



Code, pseudocode, schemadefinities, API-specificaties, mapstructuren, configuratiebestanden.

Feature-ontwerp, UI-schermen, workflows, veldniveau-datamodellen.

Volledigheid als doel. Een Blueprint die alles ontwerpt is een mislukte Blueprint.



Regel die alles overrulet: elk onderdeel wordt behandeld op het detailniveau dat nodig is om de zes beslissingen te onderbouwen — niet dieper. Onderdelen die geen van de zes raken worden in maximaal één alinea afgedaan, met expliciete verwijzing naar implementatiefase.

#### 3\. Context

KnowledgeOS is geen notitieprogramma, geen NAS, geen chatbot, geen DMS. Het is een persoonlijk Cognitive Operating System: een extern brein dat cognitieve belasting verlaagt, informatie duurzaam opslaat, kennis semantisch organiseert, AI veilig laat samenwerken met de gebruiker, automatiseringen beheert, menselijke controle behoudt, context onthoudt en informatie terugvindbaar maakt.

Projecthistorie — architectuurrelevant. Het project begon als HTML-reviewtool voor één CSV-bestand. De ambitie is daarna fundamenteel verschoven naar een lokaal KnowledgeOS. De bestaande bouwstenen zijn dus ontstaan onder een ánder probleemframe dan het huidige.

Bestaande componenten — bouwstenen, niet het eindproduct:

werkende NAS V2.2 · AI-migratiescripts · lokale Ollama · cloud AI · Python-automatisering · Human Review Workbench.

Toekomstige modules: NAS \& Knowledge Archive · AI Review Workbench · Inbox Automation · Automation Engine · AI Assistant · Search · Knowledge Index · Connectors.

#### 4\. Ontwerpconstraint: ADHD

Dit is een technische randvoorwaarde, geen achtergrondinformatie. Het systeem moet: cognitieve belasting minimaliseren · contextverlies voorkomen · beslisvermoeidheid verminderen · vertrouwen opbouwen · informatie veilig laten parkeren · altijd één logische volgende stap bieden · weken stil kunnen liggen zonder onbruikbaar te worden · schaalbaar blijven zonder complex te voelen.

Toets elke architectuurkeuze expliciet aan één vraag: verlaagt dit de drempel om het systeem na drie weken stilstand weer op te pakken, of verhoogt het die? Een keuze die de drempel verhoogt en toch wordt gemaakt, vereist geschreven rechtvaardiging.

#### 5\. Werkwijze — verplichte volgorde

Doorloop deze stappen in volgorde. Sla niets over. Elke stap eindigt in een beslissing of in een expliciet gedateerd uitstel. Analyse zonder uitkomst is geen output.



Probleemherformulering. Herformuleer het fundamentele probleem in eigen woorden voordat er iets anders gebeurt. Benoem wat het probleem niet is.

Informatiegaten. Lijst alle ontbrekende informatie. Per item verplicht: blokkeert dit een van de zes beslissingen — ja/nee. Alles met "nee" wordt genegeerd voor de rest van de Blueprint.

Aannames. Maak elke aanname expliciet en markeer als geverifieerd of onbevestigd. Onbevestigde aannames onder een beslissing verlagen de betrouwbaarheidsklasse van die beslissing.

Legacy-analyse. Beoordeel elk bestaand component op één vraag: is dit een fundament, of een artefact van het oude probleemframe dat nu ten onrechte structuurbepalend dreigt te worden? Verplicht oordeel per component: fundament / bruikbaar maar niet structuurbepalend / erfenis — af te bouwen.

Ontwerpprincipes. Leid principes af uit stap 1–4. Niet uit algemene best practices. Elk principe verwijst naar de stap waaruit het volgt.

Architectuurvergelijking. Minimaal drie kandidaat-architecturen, waarvan er één bewust minimaal is (zo min mogelijk bewegende delen). Beoordeel alle drie op dezelfde criteria.

Trade-offs. Per architectuur: wat wordt opgegeven.

Risico's. Technisch, cognitief, operationeel. Per risico een tegenmaatregel in de architectuur zelf.

Schaalbaarheid. Beoordeel op groei van datavolume, moduleaantal en gebruiksfrequentie — niet op gebruikersaantal.

Pas daarna: schrijf de Blueprint.



#### 6\. De zes beslissingen — de kern

De Blueprint moet deze zes expliciet beslissen:



Is de Review Workbench de kern van het systeem of een module?

Is Python een tijdelijk script of de vaste Automation Engine?

Waar ligt de Single Source of Truth: NAS, indexdatabase of Knowledge Model?

Hoe communiceren modules onderling?

Hoe leert het systeem van menselijke reviews?

Waar zitten AI-providers (Ollama, Claude, OpenAI) in de architectuur?



Vast formaat per beslissing, geen afwijking:



Gekozen optie

Verworpen alternatieven + reden

Omkeerbaarheidsklasse: onomkeerbaar / duur omkeerbaar / goedkoop omkeerbaar

Wat hierdoor vastligt: welke latere modules erdoor bepaald worden

Onderbouwing herleidbaar tot stap 1–4



Filterregel: elke beslissing die als goedkoop omkeerbaar wordt geclassificeerd hoort niet in de Blueprint. Verwijder de onderbouwing en noteer die beslissing als expliciet uitgesteld naar implementatie, met de reden waarom uitstel goedkoop is.

#### 7\. Multidisciplinaire review — verplicht conflict

De disciplines mogen het niet automatisch eens zijn. Doorloop expliciet:

Software Architect → UX Designer → Information Architect → AI Safety Engineer → Product Strategist → Enterprise Architect → Consensus.

Elke discipline levert kritiek, benoemt risico's, wijst zwakke punten aan of stelt een alternatief voor. Geen enkele mag "eens" zeggen zonder inhoudelijk bezwaar of expliciete onderbouwing waarom er geen bezwaar is.

Twee rollen hebben veto:



ADHD Design Specialist — vetorecht op elke keuze die onderhoudslast toevoegt.

Staff Engineer — vetorecht op elke keuze waarvoor geen concreet implementatiepad bestaat.



Een veto is definitief tenzij het met een alternatief wordt opgelost. Noteer elk uitgesproken veto in het Decision Log, ook als het later is opgelost.

Scope-slot: consensus mag niet bereikt worden door de scope te vergroten. Een rol die inbrengt "we moeten X óók meenemen" moet aangeven welke van de zes beslissingen daardoor blokkeert. Kan die rol dat niet: afgewezen, geregistreerd, klaar.

#### 8\. Verplichte inhoud

Productvisie · Missie · Ontwerpfilosofie · Design Principles · Cognitive Principles · Information Architecture · Knowledge Model · Module Architecture · Data Architecture · AI Architecture · Automation Architecture · UX Philosophy · Security · Privacy · Local First · Offline First · Explainability · Human-in-the-loop · Safety · Scalability · Maintainability · Extensibility · Future Vision · Product Roadmap · Decision Log.

Toepassen onder de detailregel uit §2. Een sectie van één alinea met verwijzing naar implementatiefase is een correcte sectie, geen tekortkoming.

#### 9\. Verplicht op te nemen



Anti-patterns · veelgemaakte fouten · verleidingen · feature creep · scope creep · architectural debt · technical debt · UX debt · cognitive debt · ADHD traps.

Waarom vergelijkbare bestaande systemen falen.

Welke keuzes bewust niet gemaakt worden, en waarom.

Welke keuzes later moeilijk terug te draaien zijn.

Welke keuzes bewust flexibel gehouden worden.



Specifiek verplicht: de faalmodus waarin het ontwerpen van het systeem het werken aan het systeem vervangt. Beschrijf het mechanisme, de vroege signalen, en een tegenmaatregel die in de architectuur zelf zit — niet in discipline of goede voornemens.

#### 10\. Exit-criterium

De Blueprint is af zodra de zes beslissingen genomen en vastgelegd zijn. Niet als het document compleet voelt. Niet als alle secties even diep zijn. Niet als alle risico's zijn afgedekt.

Sluit af met:



Bevriezingsverklaring — de zes beslissingen zijn genomen; wijziging vereist expliciete heropening met reden.

Uitstellijst — alles wat bewust naar implementatie is doorgeschoven.

Eerste implementatiestap — één concreet, uitvoerbaar bouwtaakje dat direct volgt uit de genomen beslissingen. Uitvoerbaar binnen één zitting. Geen voorbereidingstaak, geen onderzoekstaak.



#### 11\. Formaat

Nederlands, Engelse technische terminologie waar standaard. Zakelijk, direct, geen marketingtaal. Beslissingen in het vaste format van §6. Geen samenvattende afsluiting waarin de Blueprint zichzelf prijst.



Toelichting op de opbouw



Detailregel als overrule (§2). Zonder één regel die boven de inhoudslijst staat, wint de lijst altijd — 25 secties produceren automatisch een boek. Nu is een dunne sectie correct gedrag.

Filterregel op goedkoop omkeerbaar (§6). Dit is de scherpste maatregel: het verwijdert actief werk uit de Blueprint in plaats van het toe te voegen.

Informatiegaten met blokkeert-ja/nee (§5.2). Voorkomt dat ontbrekende informatie een reden wordt om niet te beslissen.

Legacy-oordeel in drie klassen (§5.4). "Analyseer de bestaande systemen" levert beschrijving op; een verplicht label levert een beslissing op.

Veto's in het Decision Log (§7). Maakt de tegenkracht controleerbaar in plaats van decoratief.

Exit-criterium vóór volledigheid (§10). Het bevriezingscriterium staat expliciet los van hoe het document voelt — dat is precies de faalmodus uit §9.

