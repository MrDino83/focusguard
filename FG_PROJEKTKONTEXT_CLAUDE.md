**Sensitivität:** 🔵 INTERNAL
**Version:** 0.2.0
**Erstellt:** 2026-05-13
**Letzte Aktualisierung:** 2026-05-18
**Quelle:** Claude+Review
**Verantwortlich:** Digitale Geschäftsentwicklung

---

# FocusGuard — Projektkontext für Claude

> Dieses Dokument ist das vollständige Briefing für einen neuen Claude-Kontext
> ohne Vorgeschichte. Einmalig lesen — danach operativ arbeitsfähig.
> Es fasst alle Governance-, Architektur- und Designentscheidungen zusammen
> die in der Entstehungsgeschichte von FocusGuard getroffen wurden.
> Normatives Regelwerk: FG_GOVERNANCE_v03.md (maßgeblich bei Widersprüchen).

---

## 1. Was ist FocusGuard?

FocusGuard ist eine **browser-basierte, lokale Single-Page-App** (kein Backend,
kein Deployment, kein Build-Schritt) die drei Kernprobleme beim Arbeiten mit
KI-Anwendungen löst:

| Problem | FocusGuard-Lösung |
|---|---|
| Fokus-Verlust beim Arbeiten | Aktive Mission mit Parking Lot für zurückgestellte Ideen |
| Datei-Drift | Erkennt wenn referenzierte `.md`-Quelldateien sich verändert haben |
| Fehlende LLM-Lesbarkeit | Erzwungene, maschinenlesbare FG-Blöcke direkt in TODO.md |

FocusGuard ist **kein eigenes Task-Management-System**. Es ist ein
**Fokus-Layer** über einer bestehenden Projektstruktur (z.B. der EKB).

---

## 2. Architektur — Grundprinzipien (nicht verhandelbar)

```
TODO.md                     ← Führendes Backlog-System
   ↓ FG-Blöcke eingebettet (FocusGuard schreibt nur in diese)
FocusGuard App (HTML)       ← Läuft lokal im Browser, kein Server
```

**Absolut geltende Regeln:**
- FocusGuard schreibt **ausschließlich in FG-Blöcke innerhalb von TODO.md**
- FOCUS.md entfällt — kein separates Persistenzmedium
- Kein Backend, kein Build, kein Deployment
- Alle Daten liegen lokal im Projektordner (OneDrive-sync-fähig)
- Offline-fähig, keine Internetverbindung erforderlich
- Kein localStorage / IndexedDB (Daten nicht im Projektordner)
- TODO.md-Inhalte außerhalb von FG-Blöcken werden **niemals** verändert
- EKB-IDs (B-01, FW-01, C-01 etc.) werden **1:1 übernommen**, keine neue ID vergeben
- FocusGuard interpretiert keine EKB-internen Governance-Regeln

**Tech-Stack (festgelegt):**
- React UMD, kein Build-Schritt
- D3.js für Netzwerkgraph
- File System Access API für lokalen Dateizugriff
- VS Code Live Server zum Starten
- Plattform: Windows (primär), Chrome/Edge

**Explizit abgelehnte Technologien:** Lovable, Electron, Supabase/Firebase,
localStorage/IndexedDB, eigener MCP-Server (für MVP).

---

## 3. Dateistruktur

```
OneDrive/
├── FocusGuard/                         ← App (einmalig, projektübergreifend)
│   ├── focusguard_11.html              ← Aktuelle App-Version
│   ├── CHANGELOG.md
│   ├── FGD_Konzept_v02.md
│   ├── FG_GOVERNANCE_v03.md            ← Normatives Regelwerk (aktuell)
│   ├── FG_PROJEKTKONTEXT_CLAUDE.md     ← Dieses Dokument
│   ├── fiktive_todo_100_tasks_v2.md
│   └── projects/                       ← Projektdaten
│
└── Enterprise Knowledgebase (EKB)/
    ├── TODO.md                         ← Führendes System — FG schreibt nur in FG-Blöcke
    ├── 00_SYSTEM/
    │   ├── ekb_governance.md           ← Übergeordnet zu FG-Governance
    │   ├── linking_convention.md
    │   └── maintenance_guide.md
    └── 02_DATENINFRASTRUKTUR/systems/
        ├── sys_cross.md
        └── sys_kdb.md
```

**Hinweis:** FOCUS.md entfällt als Datei. Alle FocusGuard-Daten
(Mission, Tasks, Parking Lot) leben als FG-Blöcke in TODO.md.

---

## 4. FG-Blöcke — Übersicht

FocusGuard kennt drei Block-Typen. Alle sind HTML-Kommentar-Marker
und für Obsidian transparent (nicht sichtbar, nicht störend).

| Block-Typ | Marker | Zweck | Position in TODO.md |
|---|---|---|---|
| `FG:MISSION` | `<!-- FG:MISSION id="..." version="..." -->` | Aktive Mission | Dateianfang |
| `FG:TASK` | `<!-- FG:TASK id="..." version="..." -->` | Operativer Task-Zustand | Direkt unter Task-Header |
| `FG:PARKING` | `<!-- FG:PARKING id="..." version="..." -->` | Zurückgestellte Ideen | Dateiende |

Vollständige Block-Schemas, Pflichtfelder und Schreib-Protokoll:
→ **FG_GOVERNANCE_v03.md §3 und §5** (maßgeblich)

### 4.1 Mission

- Genau eine aktive Mission pro TODO.md (`fg-status: AKTIV`)
- Zweite aktive Mission: harter Fehler, kein Schreiben
- **Lightweight erlaubt:** nur `fg-titel`, `fg-status` und
  `fg-letzte-fg-aenderung` sind Pflicht — alle anderen Felder `n/a`
- `fg-letzte-fg-aenderung` trägt den Timestamp des letzten
  FocusGuard-Schreibvorgangs im gesamten Projektkontext

### 4.2 Tasks

- Jeder Task bekommt einen `FG:TASK`-Block beim ersten FocusGuard-Zugriff
- Tasks aus TODO.md: Original-ID übernehmen (z.B. `B-01`)
- Native Tasks (nur in FocusGuard erstellt): `FG-NATIV-[YYYYMMDD]-[NN]`
- Native Tasks erzeugen einen minimalen Task-Header in TODO.md
- Jeder Task muss einer Mission zugeordnet sein (`fg-mission-ref`)
- Missionslose Tasks sind nicht erlaubt

**Timing-Felder:**

| Feld | Typ | Beschreibung |
|---|---|---|
| `fg-start` | YYYY-MM-DD | Arbeitsbeginn |
| `fg-deadline` | YYYY-MM-DD | Fälligkeit |
| `fg-ende` | YYYY-MM-DD | Tatsächliches Ende |
| `fg-wiederkehrend` | Enum | TÄGLICH / WÖCHENTLICH / MONATLICH / QUARTALSWEISE / JÄHRLICH |
| `fg-wiederkehrend-faktor` | Integer | Multiplikator — z.B. `2` bei WÖCHENTLICH = alle 2 Wochen |
| `fg-naechste-faelligkeit` | YYYY-MM-DD | **Berechnet** — nie manuell setzen |

Berechnungslogel: `fg-naechste-faelligkeit = fg-ende + (fg-wiederkehrend-faktor × Rhythmus)`

### 4.3 Parking Lot

- Parking-Lot-Einträge werden nicht gelöscht wenn sie zu Tasks werden
- `fg-herkunft` wird ergänzt wenn ein Parking-Lot-Item zu einem Task wird

### 4.4 Versionierung

Jeder Block trägt eine eigene Versionsnummer im Öffnungs-Marker.
Kein zentraler Changelog — Versionierung ist block-intern.

---

## 5. Führungs- und Konfliktregeln

| Konfliktfeld | Führendes System |
|---|---|
| Priorität, Zuweisung, Beschreibungstext, Labels | TODO.md — FocusGuard liest, schreibt nie |
| Operativer Status (`fg-status`) | FG:TASK-Block |
| Drift-Zustand (`fg-drift`, `fg-ref-version`) | FG:TASK-Block |
| Parking-Lot | FG:PARKING-Block |
| Aktive Mission | FG:MISSION-Block |
| Timings (`fg-start`, `fg-deadline`, `fg-ende`) | FG:TASK-Block |
| Alle EKB-weiten Governance-Regeln | ekb_governance.md (übergeordnet zu FG) |
| Dateistruktur, Präfixe, Wiki-Links | linking_convention.md v1.2.0 |

Bei Widerspruch zwischen FG-Governance und ekb_governance.md gilt
**immer** ekb_governance.md.

**FocusGuard interpretiert keine EKB-internen Governance-Regeln.**
EKB-Regeln regeln EKB-interne Prozesse. FocusGuard-Regeln regeln
FocusGuard-Verhalten. Die Regelungsbereiche überschneiden sich nicht.

**Konflikt bei mehreren Projektkontexten:** FG-Blöcke sind nicht an
einen Projektkontext gebunden. Bei konkurrierenden Schreibvorgängen
gilt Last-Write-Wins. Bekannte Einschränkung — adressierbar in
späteren Ausbaustufen.

---

## 6. LLM-Operationsregeln

### Beim Lesen einer TODO.md mit FG-Blöcken
1. Alle `FG:MISSION`-Blöcke einlesen — genau einer sollte `fg-status: AKTIV` haben
2. Tasks mit `fg-status: IN_ARBEIT` oder `OFFEN` und `fg-kontext: MISSION` identifizieren
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

### Fehlerbehandlung
- **ID-Mismatch:** ID im Marker stimmt nicht mit Task-ID überein → harter Fehler, kein Schreiben
- **Zweite aktive Mission:** FG:MISSION mit `fg-status: AKTIV` bereits vorhanden → harter Fehler, kein Schreiben
- **Drift erkannt:** `fg-drift: DRIFT` setzen, visuelles ⚠-Signal — nie automatisch korrigieren

---

## 7. FG_INDEX.md — Projektkontext-Definition

Jedes Projekt das von FocusGuard navigierbar sein soll bekommt eine `FG_INDEX.md`.
Sie definiert **Regeln** (nicht Dateilisten) und wird von FocusGuard beim Start
eingelesen um den Verzeichnisbaum dynamisch zu traversieren.

**Pflichtfelder:**
- Root-Pfad des Projekts
- Relevante Ordner / Präfixe (Positivliste oder Ausschlussliste)
- Klassifizierungsregeln (EKB-Standard oder abweichend)
- Pfad zur TODO.md (sofern vorhanden)
- Semantische Bedeutung von Datei-Präfixen (z.B. `sys_` = Systemprofil)

**Indizierungs-Trigger:**
1. Beim Öffnen eines Projektkontexts (immer, vollständig)
2. Manuell via `↺ Indizieren`-Button
3. Bei Datei-Öffnung (nur die geöffnete Datei, kein Full-Scan)
4. Beim Speichern von FG-Blöcken (partiell: nur referenzierte Dateien)

Kein Hintergrund-Polling (kein Backend → kein eigener Prozess möglich).

**RESTRICTED-Dateien** (`🔴`): Sichtbar im Graph mit Sperr-Symbol, Inhalt
nicht ladbar. Gilt für `role_registry.md`, `PERS_*.md` und alle Dateien
mit `🔴 RESTRICTED` im Header. Regel gilt auch ohne EKB-Governance.

---

## 8. Linking Convention — FG-spezifische Erweiterungen

Ergänzt `linking_convention.md v1.2.0`. Bestehende Regeln gelten unverändert.

| Link-Typ | Syntax | Wo zulässig |
|---|---|---|
| Mission-Referenz | `MISSION-[KÜRZEL]-[DATUM]` | `fg-mission-ref`-Feld, Notiz-Felder |
| Task-Referenz auf TODO.md | `REF:[Original-ID]` | Freitext-Felder in FG-Blöcken |
| Parking-Lot-Bezug | `PARK-[KÜRZEL]-[DATUM]-[NN]` | `fg-bezug`-Feld |

Wiki-Links (`[[sys_cross]]`) nur in Freitext-Feldern (`fg-notiz`, `fg-warum`,
`fg-mvp`). Personennamen nur als `[[PERS_xxx|Name]]`, nie als Freitext.

Cross-Kontext-Referenzen: über `fg-ref-datei` mit absolutem Pfad — kein direkter Wiki-Link.

**Hinweis:** `[[FOCUS]]`-Link entfällt — FOCUS.md existiert nicht mehr.

---

## 9. AI Context Export — Format

Strukturierter Prompt der direkt in Claude Desktop oder ChatGPT eingefügt
werden kann. Kein Personenname (EKB-Governance §1.1).

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

## 10. App-Versionsstand

**Aktuelle Version:** focusguard_11.html

| Feature | Status |
|---|---|
| Dashboard (Metriken, Fortschritt, Drift-Liste) | ✅ Implementiert |
| Netzwerkgraph mit D3.js | ✅ Implementiert |
| Task-Nodes mit Pie-Chart-Fortschritt (kumuliert) | ✅ Implementiert |
| Datei-Nodes als Datei-Icons im Graph | ✅ Implementiert |
| View Overlay: 3 Gruppen (Tasks/Dateien/Kontext) | ✅ Implementiert |
| Gravitations-Slider (Tasks, Dateien, Kontext) | ✅ Implementiert |
| Kurvenstärke-Regler für Kontext-Linien (magenta) | ✅ Implementiert |
| Sidebar: Task-Detail mit Drift-Box, Fortschrittsbalken | ✅ Implementiert |
| Sidebar: Datei-Detail mit vollständigem Pfad | ✅ Implementiert |
| Statusbar (Mission, Drift, Tasks, Indizierungs-Timestamp) | ✅ Implementiert |
| Filter-Overlay (Priorität, Status, Zuweisung, Ebene etc.) | ✅ Implementiert |
| Globale Suche im Header | ✅ Implementiert |
| Projektfarben-Editor im View-Overlay | ✅ Implementiert |
| TODO.md-Parser (Markdown-Block + Tabellenformat) | ✅ Implementiert |
| FG-Block-Parser (FG:MISSION, FG:TASK, FG:PARKING) | ❌ Noch nicht |
| File System Access API (echte Datei-I/O) | ❌ Noch nicht — simuliert |
| FG-Blöcke in TODO.md schreiben (echte Persistenz) | ❌ Noch nicht |
| FG_INDEX.md lesen und Verzeichnis traversieren | ❌ Noch nicht |
| Drift-Prüfung gegen echte Datei-Header | ❌ Noch nicht — simuliert |
| Wiederkehrend-Logik und fg-naechste-faelligkeit berechnen | ❌ Noch nicht |
| Multi-User / Zuständigkeiten | ❌ Geplante Ausbaustufe |
| MCP-Server-Integration | ❌ Geplante Ausbaustufe |

---

## 11. Offene Architekturentscheidungen

| # | Frage | Stand |
|---|---|---|
| 11.1 | TRAINING.md: eigenständiges LLM-Dokument oder ist FG_GOVERNANCE_v03.md die Trainingsdatei? | Offen |
| 11.7 | Visualisierung bei >10 aktiven Tasks: Filterlogik oder Zoom-/Drill-down? | Schwellwert offen |

**Geschlossene Punkte (zur Dokumentation):**

| # | Frage | Entscheidung |
|---|---|---|
| 11.2 | FOCUS.md in `00_INDEX.md` referenzieren? | Obsolet — FOCUS.md entfällt |
| 11.3 | `maintenance_guide.md` um FG-Pflegeprozesse erweitern? | EKB-Governance, nicht FG |
| 11.4 | Gilt EKB §4.2 für `fg-ref-datei`-Links? | EKB-Governance, nicht FG — Regelungsbereiche überschneiden sich nicht |
| 11.5 | Verhältnis FOCUS.md zu `interview_backlog.md`? | EKB-Governance, nicht FG |
| 11.6 | Mehrpersonen-Zuständigkeitskonzept? | Nicht MVP — Last-Write-Wins als bekannte Einschränkung dokumentiert |
| 11.8 | Visuelles Design | Entschieden: MapApp-Design (DM Sans, #0f0f11, Projekt-Farbsystem) |

---

## 12. Bekannte Einschränkungen (Produktivbetrieb)

- FG-Block-Parser noch nicht implementiert — App arbeitet mit Mock-Daten
- File System Access API noch nicht implementiert — kein echtes Datei-I/O
- FG-Blöcke werden noch nicht in TODO.md geschrieben
- Drift-Erkennung ist simuliert (statische Felder)
- Wiederkehrend-Logik und `fg-naechste-faelligkeit`-Berechnung noch nicht implementiert
- View-Einstellungen und Projektfarben sind Runtime-State, nicht persistent
- Fortschrittsmodell ist deterministisch/fiktiv — kein echter Arbeitsaufwand hinterlegt
- TODO.md-Parser liest per `fetch()` vom lokalen Server — erfordert VS Code Live Server
- Last-Write-Wins bei mehreren Projektkontexten die denselben Task referenzieren —
  kein Konfliktschutz, bekannte Einschränkung für Mehrbenutzer-Szenarien

---

## Changelog

| Version | Datum | Änderung | Quelle |
|---|---|---|---|
| 0.1.0 | 2026-05-13 | Initiale Erstellung — Vollständige Destillation aller Governance- und Architekturentscheidungen aus der FocusGuard-Entwicklungshistorie | Claude+Review |
| 0.2.0 | 2026-05-18 | Architektur-Update: FOCUS.md entfällt; FG-Blöcke in TODO.md als einziges Persistenzmedium; drei Block-Typen dokumentiert; Timing-Felder inkl. Wiederkehrend mit Faktor und berechneter Nächste-Fälligkeit; Führungsregeln aktualisiert; §11 bereinigt (alle geschlossenen Punkte dokumentiert); §12 Bekannte Einschränkungen aktualisiert; Linking Convention angepasst | Claude+Review |
