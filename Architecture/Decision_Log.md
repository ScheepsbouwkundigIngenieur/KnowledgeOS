# Decision Log

Voortzetting van het log uit `Architecture/Blueprint.md` (Deel F). Nieuwe entries komen onderaan. Entries worden nooit gewijzigd of verwijderd (append-only).

| Datum | Besluit / gebeurtenis | Klasse |
|---|---|---|
| 19-07-2026 | B1: Review Workbench = module; kern = archief + Knowledge Model | Duur omkeerbaar |
| 19-07-2026 | B2: Python = vaste Automation Engine met bindende conventieset | Duur omkeerbaar |
| 19-07-2026 | B3: SSoT = bestanden op NAS; index afgeleid; Knowledge Model = conventielaag | Praktisch onomkeerbaar |
| 19-07-2026 | B4: modulecommunicatie via bestandssysteem-contracten, batchgewijs | Duur omkeerbaar |
| 19-07-2026 | B5: leren via append-only beslisrecords in de SSoT; geen finetuning | Duur omkeerbaar |
| 19-07-2026 | B6: AI achter adapterlaag; staging → menselijke review → archief; gevoelig = lokaal | Duur omkeerbaar |
| 19-07-2026 | Veto ADHD Design Specialist op permanent draaiende onderdelen — opgelost via B4 | Veto, opgelost |
| 19-07-2026 | Veto Staff Engineer op graph-database als Knowledge Model-drager — opgelost via B3 | Veto, opgelost |
| 19-07-2026 | Huidige HTML-workbench geclassificeerd als erfenis, af te bouwen | Legacy-oordeel |
| 19-07-2026 | Eindreview afgerond: geen nieuwe beslissingen, geen scopewijziging; Blueprint definitief | Review |
| 20-07-2026 | Implementation Master Plan definitief (Fase 0 t/m 8, V1-definitie) | Plan |
| 20-07-2026 | Engineering & Governance Manual definitief; vervangt Developer Guide en Developer Playbook (beide gearchiveerd onder `Archive/`) per Golden Rule 14 | Governance |
| 20-07-2026 | Architect-overdracht vastgelegd als `CLAUDE.md` (AI-sessiecontext per Manual §8.1) | Governance |
| 20-07-2026 | **Ontwerpfase afgesloten.** Documentenset bevroren: Blueprint · Master Plan · Manual · contractdocument (ontstaat in Fase 0) · Current_State · Decision Log · Future_Ideas. Eenheid van voortgang is vanaf nu werkende code over echte bestanden | Mijlpaal |

---

## Openstaand (te verifiëren tijdens implementatie)

| Aanname | Verificatiemoment |
|---|---|
| 1 — NAS betrouwbaar bereikbaar als mount | Fase 0 (`scan_archive.py`) |
| 3 — Ordegrootte archief ≤ ~10⁶ bestanden | Fase 0 (telling) |
| 2 — Dataclassificatie cloud-AI | Gate Fase 5b, vóór eerste cloud-aanroep |
