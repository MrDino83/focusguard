**Sensitivität:** 🔵 INTERNAL
**Version:** 0.3.0
**Erstellt:** 2026-05-18
**Letzte Aktualisierung:** 2026-05-18
**Quelle:** Claude+Review
**Verantwortlich:** Digitale Geschäftsentwicklung

---

# FocusGuard — Governance v03

> Dieses Dokument ist das normative Regelwerk für FocusGuard.
> Es definiert Architektur, Datenstrukturen, Schreib-Protokoll und
> Konfliktregeln. Es ersetzt FG_GOVERNANCE_v02.md vollständig.
> Zielgruppe: Mensch und LLM gleichwertig.

---

## 1. Architektur-Grundprinzipien (nicht verhandelbar)

```
TODO.md                     ← Führendes Backlog-System
   ↓ FG-Blöcke eingebettet
FocusGuard App (HTML)       ← Läuft lokal im Browser, kein Server
```

| Prinzip | Regel |
|---|---|
| Einziges Persistenzmedium | FocusGuard schreibt ausschließlich in FG-Blöcke innerhalb von TODO.md |
| FOCUS.md entfällt | Kein separates Persistenzmedium — alle FG-Daten leben in TODO.md |
| Kein Backend | Kein Server, kein Build, kein Deployment |
| Offline-fähig | Keine Internetverbindung erforderlich |
| Lokale Datenhaltung | Alle Daten liegen im Projektordner (OneDrive-sync-fähig) |
| Kein localStorage / IndexedDB | Daten nicht im Projektordner, nicht Obsidian-kompatibel — explizit abgelehnt |
| LLM-First | Alle Strukturen sind ohne Erklärung von Claude und ChatGPT verarbeitbar |
| Visualisierung als primäre UI | Die interaktive Übersicht ist die Hauptinteraktionsfläche — kein Formular-Tool |

**Explizit abgelehnte Technologien:**

| Technologie | Ablehnungsgrund |
|---|---|
| Lovable | Cloud-Deployment, kein lokaler Dateizugriff möglich |
| Electron | Entwicklungsaufwand unverhältnismäßig für MVP |
| Supabase / Firebase | Widerspricht Datensouveränität und Offline-Anforderung |
| localStorage / IndexedDB | Daten nicht im Projektordner, nicht Obsidian-kompatibel |
| MCP-Server (eigener) | Valide Ausbaustufe, nicht für initialen Scope |

---

## 2. FG-Block-Typen

FocusGuard kennt genau drei Block-Typen. Jeder Block ist durch
HTML-Kommentar-Marker eindeutig begrenzt. FocusGuard schreibt
**ausschließlich** zwischen diesen Markern.

| Block-Typ | Öffnender Marker | Schließender Marker | Zweck |
|---|---|---|---|
| Mission | `<!-- FG:MISSION id="..." version="..." -->` | `<!-- FG:MISSION:END -->` | Aktive Mission des Projektkontexts |
| Task | `<!-- FG:TASK id="..." version="..." -->` | `<!-- FG:TASK:END -->` | Operativer Task-Zustand |
| Parking Lot | `<!-- FG:PARKING id="..." version="..." -->` | `<!-- FG:PARKING:END -->` | Zurückgestellte Ideen |

### 2.1 Positionierung in TODO.md

- FG:MISSION — am Dateianfang von TODO.md, vor allen Tasks
- FG:TASK — direkt unterhalb der zugehörigen Task-Überschrift oder Tabellenzeile
- FG:PARKING — am Dateiende von TODO.md, nach allen Tasks

Beim ersten FocusGuard-Zugriff auf einen Task wird der FG:TASK-Block
angelegt falls er noch nicht existiert.

### 2.2 Versionierung

Jeder Block trägt eine eigene Versionsnummer im Öffnungs-Marker.
Kein zentraler Changelog — Versionierung ist block-intern.

| Änderungstyp | Versions-Increment |
|---|---|
| Feldwert geändert (Status, Timing) | PATCH |
| Neues Feld hinzugekommen | MINOR |
| Schema-Änderung | MAJOR |

---

## 3. Block-Schemas

### 3.1 FG:MISSION

```markdown
<!-- FG:MISSION id="MISSION-[PROJEKTKÜRZEL]-[YYYYMMDD]" version="x.y.z" -->
fg-titel: [Max. 80 Zeichen] ← PFLICHT
fg-status: [AKTIV | PAUSIERT | ABGESCHLOSSEN] ← PFLICHT
fg-gesetzt-am: [YYYY-MM-DD | n/a]
fg-warum: [Text | n/a]
fg-mvp: [Text | n/a]
fg-abgeschlossen-am: [YYYY-MM-DD | n/a]
fg-letzte-fg-aenderung: [YYYY-MM-DD] ← PFLICHT — Timestamp des letzten FG-Schreibvorgangs im gesamten Projektkontext
<!-- FG:MISSION:END -->
```

**Invariante:** In einer TODO.md darf zu jedem Zeitpunkt genau ein
FG:MISSION-Block mit `fg-status: AKTIV` existieren. Beim Versuch eine
zweite aktive Mission anzulegen: harter Fehler, kein Schreiben.

**Lightweight-Mission:** Nur `fg-titel`, `fg-status` und
`fg-letzte-fg-aenderung` sind Pflichtfelder. Alle anderen Felder
können `n/a` sein. Eine Mission muss nicht vollständig ausformuliert
sein um gültig zu sein.

### 3.2 FG:TASK

```markdown
<!-- FG:TASK id="[Task-ID]" version="x.y.z" -->
fg-status: [OFFEN | IN_ARBEIT | ERLEDIGT | GEPARKT] ← PFLICHT
fg-kontext: [MISSION | BACKLOG] ← PFLICHT
fg-mission-ref: [MISSION-ID | n/a] ← PFLICHT
fg-letzte-bearbeitung: [YYYY-MM-DD] ← PFLICHT
fg-parent: [Task-ID | n/a]
fg-child: [Task-ID, Task-ID | n/a]
fg-start: [YYYY-MM-DD | n/a]
fg-deadline: [YYYY-MM-DD | n/a]
fg-ende: [YYYY-MM-DD | n/a]
fg-wiederkehrend: [TÄGLICH | WÖCHENTLICH | MONATLICH | QUARTALSWEISE | JÄHRLICH | n/a]
fg-wiederkehrend-faktor: [Integer | n/a]
fg-naechste-faelligkeit: [YYYY-MM-DD | n/a] ← READ-ONLY — berechnet, nie manuell setzen
fg-ref-datei: [./relativer/pfad.md | /absoluter/pfad.md | n/a]
fg-ref-version: [x.y.z | n/a]
fg-drift: [CLEAN | DRIFT | n/a]
<!-- FG:TASK:END -->
```

**Pflichtfelder:** `fg-status`, `fg-kontext`, `fg-mission-ref`,
`fg-letzte-bearbeitung`. Alle anderen Felder optional — `n/a` wenn
nicht anwendbar. Kein Feld darf leer sein.

**Berechnungslogik `fg-naechste-faelligkeit`:**
Wird automatisch berechnet wenn `fg-ende`, `fg-wiederkehrend` und
`fg-wiederkehrend-faktor` gesetzt sind.

```
fg-naechste-faelligkeit = fg-ende + (fg-wiederkehrend-faktor × Rhythmus)
```

FocusGuard schreibt dieses Feld automatisch. Der Nutzer schreibt
nie manuell in `fg-naechste-faelligkeit`.

### 3.3 FG:PARKING

```markdown
<!-- FG:PARKING id="PARK-[PROJEKTKÜRZEL]-[YYYYMMDD]-[NN]" version="x.y.z" -->
fg-titel: [Max. 120 Zeichen] ← PFLICHT
fg-herkunft: [MANUELL | TASK:[Task-ID] | MISSION:[Mission-ID]] ← PFLICHT
fg-geparkt-am: [YYYY-MM-DD] ← PFLICHT
fg-bezug: [Task-ID | Mission-ID | n/a]
fg-notiz: [Freitext | n/a]
<!-- FG:PARKING:END -->
```

Parking-Lot-Einträge werden **nicht gelöscht** wenn sie zu Tasks
werden. `fg-herkunft` wird ergänzt:
`MANUELL → TASK:FG-NATIV-20260518-01`.

---

## 4. ID-Regeln

| Task-Ursprung | ID-Format | Beispiel |
|---|---|---|
| Task aus TODO.md | Original-ID übernehmen | `B-01` |
| Task nativ in FocusGuard erstellt | `FG-NATIV-[YYYYMMDD]-[NN]` | `FG-NATIV-20260518-01` |
| Mission | `MISSION-[PROJEKTKÜRZEL]-[YYYYMMDD]` | `MISSION-EKB-20260518` |
| Gleicher Tag, neue Mission | Suffix `-B`, `-C` | `MISSION-EKB-20260518-B` |
| Parking Lot | `PARK-[PROJEKTKÜRZEL]-[YYYYMMDD]-[NN]` | `PARK-EKB-20260518-01` |

**IDs werden niemals geändert oder neu vergeben.**

---

## 5. Schreib-Protokoll

### 5.1 Grundregel

FocusGuard schreibt in TODO.md **ausschließlich innerhalb definierter
FG-Blöcke**. Alles außerhalb dieser Marker ist für FocusGuard nicht
existent — kein Lesen für Schreibzwecke, kein Verändern, kein Löschen.

### 5.2 Zulässige Schreib-Operationen

| Operation | Auslöser | Felder die geschrieben werden |
|---|---|---|
| FG:TASK-Block erstellen | Task erstmals in FocusGuard geöffnet | Alle Pflichtfelder, optionale Felder als `n/a` |
| Status ändern | Nutzer ändert Status im Frontend | `fg-status`, `fg-letzte-bearbeitung`, Block-Version |
| Kontext ändern | Task wird Mission zugeordnet oder zurück | `fg-kontext`, `fg-mission-ref`, `fg-letzte-bearbeitung`, Block-Version |
| Timing setzen | Nutzer setzt Start / Deadline / Ende | Entsprechendes Timing-Feld, `fg-letzte-bearbeitung`, Block-Version |
| Wiederkehrend setzen | Nutzer setzt Rhythmus und Faktor | `fg-wiederkehrend`, `fg-wiederkehrend-faktor`, `fg-naechste-faelligkeit` (berechnet), Block-Version |
| Parent/Child setzen | Subtask-Beziehung in FG definiert | `fg-parent` / `fg-child`, `fg-letzte-bearbeitung`, Block-Version |
| Drift bestätigt | Nutzer bestätigt Drift-Warnung explizit | `fg-drift: CLEAN`, `fg-ref-version` aktualisieren, `fg-letzte-bearbeitung`, Block-Version |
| Nativ-Task anlegen | Neuer Task in FocusGuard ohne TODO.md-Ursprung | Neuer FG:TASK-Block + minimaler Task-Header in TODO.md |
| Mission anlegen | Neue Mission gesetzt | Neuer FG:MISSION-Block, `fg-letzte-fg-aenderung` |
| Parking-Lot-Eintrag anlegen | Nutzer parkt Idee oder Task | Neuer FG:PARKING-Block |
| Task parken | Task-Status auf GEPARKT | `fg-status: GEPARKT`, neuer FG:PARKING-Block mit `fg-herkunft: TASK:[ID]` |

### 5.3 Verbotene Operationen (absolut)

- Schreiben außerhalb von FG-Block-Markern
- Löschen eines FG-Blocks
- Verändern der Task-ID im Marker
- Ändern von Inhalten außerhalb der FG-Felder (Priorität, Zuweisung, Beschreibungstext, Labels)
- Automatisches Aktualisieren von `fg-drift` oder `fg-ref-version` ohne explizite Nutzerbestätigung
- Manuelles Setzen von `fg-naechste-faelligkeit`
- Schreiben in TODO.md wenn ID-Mismatch vorliegt (siehe §6.1)

### 5.4 Nativ-Task in TODO.md anlegen

Wenn FocusGuard einen nativen Task erstellt, wird in TODO.md ein
minimaler Eintrag angelegt:

```markdown
### FG-NATIV-20260518-01 — [Titel]

<!-- FG:TASK id="FG-NATIV-20260518-01" version="0.1.0" -->
fg-status: OFFEN
fg-kontext: MISSION
fg-mission-ref: [aktive Mission-ID]
fg-letzte-bearbeitung: 2026-05-18
fg-parent: n/a
fg-child: n/a
fg-start: n/a
fg-deadline: n/a
fg-ende: n/a
fg-wiederkehrend: n/a
fg-wiederkehrend-faktor: n/a
fg-naechste-faelligkeit: n/a
fg-ref-datei: n/a
fg-ref-version: n/a
fg-drift: n/a
<!-- FG:TASK:END -->
```

FocusGuard schreibt **nur** diesen Block und den Task-Header.
Kein Fließtext, keine Beschreibung, keine weiteren TODO.md-Felder.
Der Nutzer befüllt den Rest manuell.

---

## 6. Fehlerbehandlung

### 6.1 ID-Mismatch

Wenn FocusGuard einen `FG:START`-Marker findet dessen `id`-Attribut
nicht mit der Task-ID in TODO.md übereinstimmt:

**→ Harter Fehler. FocusGuard schreibt nicht.**

Fehlermeldung im Frontend mit Angabe der betroffenen ID und Zeilennummer.
Kein stilles Überschreiben. Kein Fallback.

### 6.2 Zweite aktive Mission

Wenn beim Anlegen einer neuen Mission bereits ein FG:MISSION-Block
mit `fg-status: AKTIV` existiert:

**→ Harter Fehler. FocusGuard schreibt nicht.**

Fehlermeldung im Frontend mit Hinweis auf die bestehende aktive Mission.
Nutzer muss bestehende Mission explizit pausieren oder abschließen.

### 6.3 Drift-Erkennung

Wenn `fg-ref-datei` gesetzt ist und die Version der referenzierten
Datei von `fg-ref-version` abweicht:

**→ Visuelles ⚠-Signal am Task und an der Quelldatei. Kein automatisches Korrigieren.**

`fg-drift` wird auf `DRIFT` gesetzt. `fg-ref-version` wird
**nur** aktualisiert wenn der Nutzer die Drift explizit bestätigt.

---

## 7. Führungs- und Konfliktregeln

| Konfliktfeld | Führendes System |
|---|---|
| Priorität, Zuweisung, Beschreibungstext, Labels | TODO.md — FocusGuard liest, schreibt nie |
| Operativer Status (`fg-status`) | FG:TASK-Block |
| Drift-Zustand (`fg-drift`, `fg-ref-version`) | FG:TASK-Block |
| Parking-Lot | FG:PARKING-Block |
| Aktive Mission | FG:MISSION-Block |
| Timings (`fg-start`, `fg-deadline`, `fg-ende`) | FG:TASK-Block |
| Alle EKB-weiten Governance-Regeln | ekb_governance.md (übergeordnet zu FG-Governance) |
| Dateistruktur, Präfixe, Wiki-Links | linking_convention.md v1.2.0 |

Bei Widerspruch zwischen FG-Governance und ekb_governance.md gilt
**immer** ekb_governance.md.

**FocusGuard interpretiert keine EKB-internen Governance-Regeln.**
EKB-Regeln (z.B. §4.2) regeln EKB-interne Prozesse.
FocusGuard-Regeln regeln FocusGuard-Verhalten. Die Regelungsbereiche
überschneiden sich nicht.

### 7.1 Konflikt bei mehreren Projektkontexten

FG-Blöcke sind nicht an einen Projektkontext gebunden. Derselbe
Task kann von mehreren Projektkontexten referenziert werden.

**Konfliktauflösung: Last-Write-Wins.**

Kein Besitzer-Mechanismus, kein Merge, keine Warnung bei
konkurrierenden Schreibvorgängen. Bekannte Einschränkung für
Mehrbenutzer-Szenarien — adressierbar in einer späteren Ausbaustufe.

---

## 8. `fg-ref-datei` — Referenzierungsregeln

`fg-ref-datei` kann auf jede erreichbare `.md`-Datei zeigen.
FocusGuard macht keine Aussage über erlaubte Referenzziele.
Welche Dateien innerhalb der EKB referenziert werden dürfen ist
Sache der EKB-Governance — nicht von FocusGuard.

| Zustand | fg-ref-datei | fg-ref-version | fg-drift |
|---|---|---|---|
| Keine Referenz | `n/a` | `n/a` | `n/a` |
| Referenz vorhanden, kein Drift | `./pfad/datei.md` | `x.y.z` | `CLEAN` |
| Referenz vorhanden, Drift erkannt | `./pfad/datei.md` | `x.y.z` (alt) | `DRIFT` |

---

## 9. LLM-Operationsregeln

### Beim Lesen einer TODO.md mit FG-Blöcken
1. Alle `FG:MISSION`-Blöcke einlesen — genau einer sollte `fg-status: AKTIV` haben
2. Alle `FG:TASK`-Blöcke mit `fg-status: IN_ARBEIT` oder `OFFEN` und `fg-kontext: MISSION` identifizieren
3. Für jeden Task mit `fg-ref-datei != n/a`: Datei zugänglich? → Version aus Header lesen
4. Abweichung von `fg-ref-version` → **⚠ DRIFT** im Output markieren, **nicht** automatisch korrigieren
5. `FG:PARKING`-Blöcke für vollständigen Kontext lesen

### Beim Schreiben / Ändern eines FG-Blocks
1. Block-Version aus Öffnungs-Marker lesen
2. Änderung durchführen
3. Block-Version nach Semantic Versioning erhöhen
4. `fg-letzte-bearbeitung` auf aktuelles Datum setzen
5. Bei Mission-Block: `fg-letzte-fg-aenderung` auf aktuelles Datum setzen
6. Kein Pflichtfeld weglassen — auch `n/a` muss stehen
7. IDs niemals ändern oder neu vergeben
8. `fg-ref-version` und `fg-drift` nur mit expliziter Nutzerbestätigung aktualisieren
9. `fg-naechste-faelligkeit` niemals manuell setzen — immer berechnen

### Beim Erstellen eines neuen FG:TASK-Blocks
1. ID prüfen: stimmt sie mit der Task-ID in TODO.md überein? Nein → Fehler, abbrechen
2. Alle Pflichtfelder setzen
3. Alle optionalen Felder auf `n/a` setzen
4. Block-Version auf `0.1.0` setzen

### Beim Parken eines Tasks
1. `fg-status: GEPARKT` setzen
2. Neuen `FG:PARKING`-Block anlegen mit `fg-herkunft: TASK:[ID]`
3. FG:TASK-Block bleibt erhalten — ID muss erhalten bleiben

---

## 10. AI Context Export — Format

Strukturierter Prompt der direkt in Claude Desktop oder ChatGPT
eingefügt werden kann. Kein Personenname (EKB-Governance §1.1).

```
=== FOCUSGUARD CONTEXT EXPORT ===
Projektkontext: [Name]
Stand: [YYYY-MM-DD HH:MM]

--- AKTIVE MISSION ---
ID: [Mission-ID]
Titel: [fg-titel]
Warum jetzt: [fg-warum]
MVP-Definition: [fg-mvp]

--- AKTIVE TASKS ---
ID: [Task-ID]
Titel: [Titel aus TODO.md]
Status: [fg-status]
⚠ DRIFT: [fg-ref-datei] — zuletzt v[fg-ref-version], aktuell v[aktuelle Version]
   ↳ Geändert seit letzter Bearbeitung: [Changelog-Einträge wenn bekannt]

--- PARKING LOT ---
ID: [PARK-ID] | [fg-titel] | Bezug: [fg-bezug]

=== END FOCUSGUARD CONTEXT ===
```

Drift-Zeile nur wenn `fg-drift: DRIFT` vorliegt.

---

## 11. Abgrenzung zu bestehenden Tools

| Tool | Zweck | Verhältnis zu FocusGuard |
|---|---|---|
| TODO.md (EKB) | Vollständiger Task-Backlog — führendes System | FocusGuard erweitert um Fokus-Layer via FG-Blöcke — schreibt nie außerhalb dieser |
| MapApp (ProblemMap) | Visuelle Netzwerkansicht aller Tasks | Komplementär — MapApp zeigt Gesamtbild, FocusGuard den aktiven Fokus |
| Claude Desktop | KI-Arbeitsumgebung | FocusGuard liefert strukturierten Kontext via AI Export, kein Ersatz |
| Obsidian | Wissensbasis-Viewer und Graph | FG-Blöcke sind als HTML-Kommentare Obsidian-kompatibel — nicht sichtbar aber nicht störend |
| Claude Projects | Kontext-Workspace per Projekt | FocusGuard ist projektordner-nativ, kein Claude-Projekt erforderlich |

---

## Changelog

| Version | Datum | Änderung | Quelle |
|---|---|---|---|
| 0.1.0 | 2026-05-12 | Initiale Erstellung | Claude+Review |
| 0.2.0 | 2026-05-12 | Visualisierung als primäre UI; Interaktionsmodell; visuelle Kodierung | Claude+Review |
| 0.3.0 | 2026-05-18 | Vollständige Neustrukturierung: FOCUS.md entfällt; FG-Blöcke in TODO.md als einziges Persistenzmedium; drei Block-Typen (MISSION, TASK, PARKING); Schreib-Protokoll; Timing-Felder inkl. Wiederkehrend mit Faktor und berechneter Nächste-Fälligkeit; Fehlerbehandlung; Konfliktregeln; LLM-Operationsregeln | Claude+Review |
