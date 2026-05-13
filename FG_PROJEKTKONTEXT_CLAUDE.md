**Sensitivität:** 🔵 INTERNAL
**Version:** 0.1.0
**Erstellt:** 2026-05-13
**Letzte Aktualisierung:** 2026-05-13
**Quelle:** Claude+Review
**Verantwortlich:** Digitale Geschäftsentwicklung

---

# FocusGuard — Projektkontext für Claude

> Dieses Dokument ist das vollständige Briefing für einen neuen Claude-Kontext
> ohne Vorgeschichte. Einmalig lesen — danach operativ arbeitsfähig.
> Es fasst alle Governance-, Architektur- und Designentscheidungen zusammen
> die in der Entstehungsgeschichte von FocusGuard getroffen wurden.

---

## 1. Was ist FocusGuard?

FocusGuard ist eine **browser-basierte, lokale Single-Page-App** (kein Backend,
kein Deployment, kein Build-Schritt) die drei Kernprobleme beim Arbeiten mit
KI-Anwendungen löst:

| Problem | FocusGuard-Lösung |
|---|---|
| Fokus-Verlust beim Arbeiten | Aktive Mission mit Parking Lot für zurückgestellte Ideen |
| Datei-Drift | Erkennt wenn referenzierte `.md`-Quelldateien sich verändert haben |
| Fehlende LLM-Lesbarkeit | Erzwungene, maschinenlesbare Datenstrukturen in FOCUS.md |

FocusGuard ist **kein eigenes Task-Management-System**. Es ist ein
**Fokus-Layer** über einer bestehenden Projektstruktur (z.B. der EKB).

---

## 2. Architektur — Grundprinzipien (nicht verhandelbar)

```
TODO.md                     ← Führendes Backlog-System (nur lesend)
   ↓ referenziert per ID
FOCUS.md                    ← Einzige Datei die FG schreibt (pro Projekt)
   ↓ liest und zeigt
FocusGuard App (HTML)       ← Läuft lokal im Browser, kein Server
```

**Absolut geltende Regeln:**
- FocusGuard schreibt **niemals** in TODO.md zurück
- Kein Backend, kein Build, kein Deployment
- Alle Daten liegen lokal im Projektordner (OneDrive-sync-fähig)
- Offline-fähig, keine Internetverbindung erforderlich
- Kein localStorage / IndexedDB (Daten nicht im Projektordner)
- TODO.md wird **nicht** migriert oder reformatiert
- EKB-IDs (B-01, FW-01, C-01 etc.) werden **1:1 übernommen**, keine neue ID vergeben

**Tech-Stack (festgelegt):**
- React UMD, kein Build-Schritt
- D3.js für Netzwerkgraph
- File System Access API für lokalen Dateizugriff
- VS Code Live Server zum Starten
- Platform: Windows (primär), Chrome/Edge

**Explizit abgelehnte Technologien:** Lovable, Electron, Supabase/Firebase,
localStorage/IndexedDB, eigener MCP-Server (für MVP).

---

## 3. Dateistruktur

```
OneDrive/
├── FocusGuard/                     ← App (einmalig, projektübergreifend)
│   ├── focusguard_11.html          ← Aktuelle App-Version
│   ├── CHANGELOG.md
│   ├── FGD_Konzept_v02.md
│   ├── FG_GOVERNANCE_v02.md        ← Normatives Regelwerk
│   ├── FG_PROJEKTKONTEXT_CLAUDE.md ← Dieses Dokument
│   ├── fiktive_todo_100_tasks_v2.md
│   └── projects/                   ← Projektdaten
│
└── Enterprise Knowledgebase (EKB)/
    ├── TODO.md                     ← Bestehend, unverändert, nur lesend
    ├── FOCUS.md                    ← Neu durch FocusGuard (noch nicht angelegt)
    ├── 00_SYSTEM/
    │   ├── ekb_governance.md       ← Übergeordnet zu FG-Governance
    │   ├── linking_convention.md
    │   └── maintenance_guide.md
    └── 02_DATENINFRASTRUKTUR/systems/
        ├── sys_cross.md
        └── sys_kdb.md
```

---

## 4. FOCUS.md — Vollständiges Schema

### 4.1 Pflicht-Header (EKB-Standard)

```markdown
**Sensitivität:** [🔵 INTERNAL | 🟡 CONFIDENTIAL | 🔴 RESTRICTED]
**Version:** [MAJOR.MINOR.PATCH]
**Erstellt:** [YYYY-MM-DD]
**Letzte Aktualisierung:** [YYYY-MM-DD]
**Quelle:** [Manuell | Claude+Review]
**Verantwortlich:** [Rolle oder Team]
```

Versionierung: MAJOR = Schemaänderung, MINOR = neuer Inhalt, PATCH = Statusänderung.

### 4.2 Block-Reihenfolge (verbindlich, Parser-kritisch)

```
[EKB-Header]
---
# FOCUS — [Projektkontext-Name]
[Kontext-Kommentar: 1–3 Sätze. Pflicht.]
---
## MISSION
---
## TASKS
---
## PARKING LOT
---
## Changelog
```

Abschnittsbezeichner exakt in dieser Schreibweise. Abweichung = Parser-Fehler.
Leerer Block: `<!-- Keine Einträge -->`

### 4.3 Mission-Block

```markdown
## MISSION

**ID:** MISSION-[PROJEKTKÜRZEL]-[YYYYMMDD]
**Titel:** [Max. 80 Zeichen]
**Status:** [AKTIV | PAUSIERT | ABGESCHLOSSEN]
**Gesetzt am:** [YYYY-MM-DD]
**Warum jetzt:** [1–3 Sätze]
**MVP-Definition:** [1–5 Sätze]
**Abgeschlossen am:** [YYYY-MM-DD | n/a]
```

Genau eine Mission pro FOCUS.md. Alle Felder Pflicht. Kein Feld leer — `n/a` wenn
nicht anwendbar. Gleicher Tag, neue Mission: Suffix `-B`, `-C`.

### 4.4 Task-Eintrag

```markdown
- **ID:** [Original-ID aus TODO.md | FG-NATIV-[YYYYMMDD]-[NN]]
  **Titel:** [Freitext]
  **Status:** [OFFEN | IN_ARBEIT | ERLEDIGT | GEPARKT]
  **Kontext:** [MISSION | BACKLOG]
  **Quelle:** [REF:[Original-ID] | NATIV]
  **ref-Datei:** [./relativer/pfad.md | /absoluter/pfad.md | n/a]
  **ref-Version:** [x.y.z | n/a]
  **ref-Typ:** [INTERN | EXTERN | n/a]
  **Letzte Bearbeitung:** [YYYY-MM-DD]
  **Notiz:** [Freitext | n/a]
```

**ID-Regeln:**
- Task aus TODO.md → Original-ID übernehmen (z.B. `B-01`), `Quelle: REF:B-01`
- Task nur in FOCUS.md → `FG-NATIV-20260513-01`, `Quelle: NATIV`
- Native Tasks sind die Ausnahme; wiederkehrende Tasks gehören in TODO.md

**Drift-Mechanismus** (ref-Datei + ref-Version + ref-Typ):

| Zustand | ref-Datei | ref-Version | ref-Typ |
|---|---|---|---|
| Keine Referenz | `n/a` | `n/a` | `n/a` |
| Intern | `./pfad/datei.md` | `x.y.z` | `INTERN` |
| Extern | `/absoluter/pfad.md` | `x.y.z` | `EXTERN` |

`ref-Version` wird **nur** aktualisiert wenn Nutzer Drift explizit bestätigt.
LLM darf `ref-Version` nie eigenständig aktualisieren.

### 4.5 Parking-Lot-Eintrag

```markdown
- **ID:** PARK-[PROJEKTKÜRZEL]-[YYYYMMDD]-[NN]
  **Titel:** [Max. 120 Zeichen]
  **Herkunft:** [MANUELL | TASK:[Task-ID] | MISSION:[Mission-ID]]
  **Geparkt am:** [YYYY-MM-DD]
  **Bezug:** [Task-ID | Mission-ID | n/a]
  **Notiz:** [Freitext | n/a]
```

Parking-Lot-Items werden **nicht gelöscht** wenn sie zu Tasks werden.
`Herkunft` wird ergänzt: `MANUELL → TASK:FG-NATIV-20260513-01`.

### 4.6 Changelog-Block (immer letzter Abschnitt)

```markdown
## Changelog

| Version | Datum | Änderung | Quelle |
|---|---|---|---|
| 0.1.0 | YYYY-MM-DD | Initiale Erstellung | [Quelle] |
```

Neueste Einträge unten. Nie kürzen oder löschen.

---

## 5. Führungs- und Konfliktregeln

| Konfliktfeld | Führendes System |
|---|---|
| Priorität, Fälligkeit, Zuweisung, Backlog-Status | TODO.md |
| Aktiver Status, Drift-Zustand, Parking-Lot | FOCUS.md |
| Alle EKB-weiten Governance-Regeln | ekb_governance.md (übergeordnet zu FG) |
| Dateistruktur, Präfixe, Wiki-Links | linking_convention.md v1.2.0 |

Bei Widerspruch zwischen FG-Governance und ekb_governance.md gilt **immer**
ekb_governance.md.

---

## 6. LLM-Operationsregeln

### Beim Lesen einer FOCUS.md
1. Header einlesen, Version notieren
2. MISSION-Block: `Status: AKTIV`? → aktive Mission vorhanden
3. Tasks mit `IN_ARBEIT` oder `OFFEN` + `Kontext: MISSION` identifizieren
4. Für jeden: `ref-Datei != n/a` → Datei zugänglich? → Version aus Header lesen
5. Abweichung von `ref-Version` → **⚠ DRIFT** im Output markieren, **nicht** automatisch korrigieren
6. PARKING LOT für vollständigen Kontext lesen

### Beim Schreiben / Ändern
1. Aktuelle Version aus Header lesen
2. Änderung durchführen
3. Version nach Semantic-Versioning erhöhen
4. `Letzte Aktualisierung` auf aktuelles Datum setzen
5. Changelog-Eintrag anhängen
6. Kein Pflichtfeld weglassen (auch `n/a` muss stehen)
7. IDs niemals ändern oder neu vergeben
8. `ref-Version` nur mit expliziter Nutzerbestätigung aktualisieren

### Beim Erstellen einer neuen FOCUS.md
1. Header setzen, Version `0.1.0`
2. Alle vier Blöcke anlegen mit `<!-- Keine Einträge -->`
3. Ersten Changelog-Eintrag anlegen

### Beim Überführen eines TODO.md-Tasks
1. Original-ID übernehmen (z.B. `B-01`)
2. `Quelle: REF:B-01` setzen
3. `Status: OFFEN` (unabhängig vom TODO.md-Status)
4. TODO.md bleibt unverändert

### Beim Parken eines Tasks
1. `Status: GEPARKT` setzen
2. Parking-Lot-Eintrag anlegen mit `Herkunft: TASK:[ID]`
3. Task bleibt in TASKS-Liste (ID muss erhalten bleiben)

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
4. Beim Speichern von FOCUS.md (partiell: nur referenzierte Dateien)

Kein Hintergrund-Polling (kein Backend → kein eigener Prozess möglich).

**RESTRICTED-Dateien** (`🔴`): Sichtbar im Graph mit Sperr-Symbol, Inhalt
nicht ladbar. Gilt für `role_registry.md`, `PERS_*.md` und alle Dateien
mit `🔴 RESTRICTED` im Header. Regel gilt auch ohne EKB-Governance.

---

## 8. Linking Convention — FG-spezifische Erweiterungen

Ergänzt `linking_convention.md v1.2.0`. Bestehende Regeln gelten unverändert.

| Link-Typ | Syntax | Wo zulässig |
|---|---|---|
| FOCUS.md selbst | `[[FOCUS]]` | Andere EKB-Dateien |
| Mission-Referenz | `MISSION-[KÜRZEL]-[DATUM]` | Innerhalb FOCUS.md, Notiz-Felder |
| Task-Referenz auf TODO.md | `REF:[Original-ID]` | `Quelle`-Feld in FOCUS.md Tasks |
| Parking-Lot-Bezug | `PARK-[KÜRZEL]-[DATUM]-[NN]` | `Bezug`-Feld |

Wiki-Links (`[[sys_cross]]`) nur in Freitext-Feldern (`Notiz`, `Warum jetzt`,
`MVP-Definition`). Personennamen nur als `[[PERS_xxx|Name]]`, nie als Freitext.

Cross-Kontext-Referenzen zwischen FOCUS.md-Dateien: über `ref-Datei` mit
absolutem Pfad und `ref-Typ: EXTERN` — kein direkter Wiki-Link.

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
Titel: [Titel]
Warum jetzt: [Text]
MVP-Definition: [Text]

--- AKTIVE TASKS ---
ID: [ID] ([Quelle])
Titel: [Titel]
Status: [Status]
⚠ DRIFT: [ref-Datei] — zuletzt v[ref-Version], aktuell v[aktuelle Version]
   ↳ Geändert seit letzter Bearbeitung: [Changelog-Einträge wenn bekannt]

--- PARKING LOT ---
ID: [PARK-ID] | [Titel] | Bezug: [Bezug]

=== END FOCUSGUARD CONTEXT ===
```

Drift-Zeile nur wenn tatsächlich Drift vorliegt.

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
| File System Access API (echte Datei-I/O) | ❌ Noch nicht — simuliert |
| FG_INDEX.md lesen und Verzeichnis traversieren | ❌ Noch nicht |
| Drift-Prüfung gegen echte Datei-Header | ❌ Noch nicht — simuliert |
| FOCUS.md schreiben (echte Persistenz) | ❌ Noch nicht |
| Multi-User / Zuständigkeiten | ❌ Geplante Ausbaustufe |
| MCP-Server-Integration | ❌ Geplante Ausbaustufe |

---

## 11. Offene Architekturentscheidungen

| # | Frage | Stand |
|---|---|---|
| 11.1 | TRAINING.md: eigenständiges LLM-Dokument oder ist FG_GOVERNANCE_v02.md die Trainingsdatei? | Offen |
| 11.2 | FOCUS.md in `00_INDEX.md` und `ekb_governance.md` referenzieren? | Offen |
| 11.3 | `maintenance_guide.md` um FG-Pflegeprozesse erweitern? | Offen |
| 11.4 | Gilt EKB §4.2 (keine `07_FEEDBACK/zyklen/`-Inhalte als Wissensquelle) auch für `ref-Datei`-Links? | Offen |
| 11.5 | Wie verhält sich FOCUS.md zu `interview_backlog.md`? | Abgrenzung empfohlen, noch nicht formalisiert |
| 11.6 | FocusGuard für mehrere Personen: Zuständigkeits-Konzept in FOCUS.md? | Geplant, nicht spezifiziert |
| 11.7 | Visualisierung bei >10 aktiven Tasks: Filterlogik oder Zoom-/Drill-down? | Schwellwert offen |
| 11.8 | Visuelles Design: Anlehnung an MapApp oder eigenständig? | Entschieden: MapApp-Design (DM Sans, #0f0f11, Projekt-Farbsystem) |

---

## 12. Bekannte Einschränkungen (Produktivbetrieb)

- Dateiinhalte werden noch nicht über File System Access API gelesen — Graph arbeitet mit Mock-Daten
- Drift-Erkennung ist simuliert (statische `refVer`/`refVerAkt`-Felder)
- FOCUS.md wird noch nicht zurückgeschrieben
- View-Einstellungen und Projektfarben sind Runtime-State, nicht persistent
- Fortschrittsmodell ist deterministisch/fiktiv — kein echter Arbeitsaufwand hinterlegt
- TODO.md-Parser liest per `fetch()` vom lokalen Server — erfordert VS Code Live Server

---

## Changelog

| Version | Datum | Änderung | Quelle |
|---|---|---|---|
| 0.1.0 | 2026-05-13 | Initiale Erstellung — Vollständige Destillation aller Governance- und Architekturentscheidungen aus der FocusGuard-Entwicklungshistorie | Claude+Review |
