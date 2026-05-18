**Sensitivität:** 🔵 INTERNAL
**Version:** 0.3.0
**Erstellt:** 2026-05-12
**Letzte Aktualisierung:** 2026-05-18
**Quelle:** Claude+Review
**Verantwortlich:** Digitale Geschäftsentwicklung

---

# FocusGuard — Konzeptdokument

> Dieses Dokument beschreibt das konzeptionelle Zielbild von FocusGuard.
> Es enthält keine Implementierungsdetails — diese sind in FG_GOVERNANCE_v03.md.
> Stand: Konzept in Entwicklung — nicht entscheidungsreif.

---

## 1. Problemstellung

Beim Arbeiten an komplexen KI-Anwendungen und UseCases entsteht ein
wiederkehrendes Muster:

1. Eine Aufgabe wird begonnen
2. Während der Bearbeitung tauchen neue Ideen, Probleme oder Features auf
3. Der ursprüngliche Fokus geht verloren (Scope-Creep)
4. Tasks referenzieren Quelldateien, die sich zwischenzeitlich geändert haben
5. Beim Wiederaufnehmen eines Tasks ist unklar, ob der letzte Arbeitsstand
   noch zur aktuellen Quelldatei passt

---

## 2. Zielbild

FocusGuard ist ein leichtgewichtiges, portables Werkzeug das drei Kernprobleme löst:

- **Fokus-Verlust** → aktive Mission mit Parking Lot für zurückgestellte Ideen
- **Datei-Drift** → Awareness wenn referenzierte Quelldateien sich geändert haben
- **Strukturbruch** → erzwungene, LLM-lesbare Datenstrukturen direkt in TODO.md

---

## 3. Leitprinzipien

| Prinzip | Ausprägung |
|---|---|
| Daten gehören in Projektordner | Keine App-eigene Datenhaltung |
| Portabilität | Tool läuft in jedem Projektkontext ohne Rekonfiguration |
| EKB-Konformität | Linking-Konvention und EKB-Governance gelten uneingeschränkt |
| Changelog-Awareness | Tasks wissen welche Dateiversion sie zuletzt referenziert haben |
| LLM-First | Alle Strukturen sind ohne Erklärung von Claude und ChatGPT verarbeitbar |
| Minimale Komplexität | Kein Backend, kein Build, kein Deployment |
| Kein Medienbruch | Nahtlose Integration mit Claude Desktop und ChatGPT |
| Visualisierung als primäre UI | Die interaktive Übersicht ist die Hauptinteraktionsfläche — kein Formular-Tool |
| Single Source of Truth | Alle FG-Daten leben als FG-Blöcke in TODO.md — kein separates Persistenzmedium |

---

## 4. Funktionale Anforderungen

### 4.1 Fokus-Management
- Genau eine aktive Mission pro Projektkontext
- Mission ist lightweight — nur Titel und Status sind Pflicht, alle anderen Felder optional
- Parking Lot für zurückgestellte Ideen mit Herkunfts-Tracking
- Jeder Task muss einer Mission zugeordnet sein

### 4.2 Task-Management
- Vollständiges Task-Management pro Projektkontext via FG-Blöcke in TODO.md
- Tasks können auf externe .md-Quelldateien referenzieren
- Referenz enthält Dateipfad und Versionsstand zum Zeitpunkt der letzten Bearbeitung
- Tasks haben Status, Priorität und optionale Subtasks (Parent/Child)
- Zuordnung zur aktiven Mission oder zum Backlog
- Tasks aus bestehender TODO.md werden per Original-ID referenziert — nie kopiert
- Parking-Lot-Items können per Interaktion in aktive Tasks überführt werden
- Wiederkehrende Tasks: Rhythmus, Faktor und automatisch berechnete nächste Fälligkeit

### 4.3 Changelog-Awareness
- Beim Wiederaufnehmen eines Tasks: automatischer Abgleich der ref-Version
  mit der aktuellen Version der Quelldatei
- Bei Abweichung: visuelle Markierung (⚠) direkt am Task und an der Quelldatei
- Einblendung der Changelog-Einträge seit letzter Bearbeitung auf Anforderung
- Nutzer entscheidet bewusst ob er die Änderungen zur Kenntnis nimmt
- Erst nach Bestätigung wird die gespeicherte ref-Version aktualisiert

### 4.4 AI Context Export
- Strukturierter Export des aktuellen Arbeitskontexts als Prompt
- Direkt einfügbar in Claude Desktop oder ChatGPT ohne Nachbearbeitung
- Enthält: aktive Mission, offene Tasks mit Drift-Hinweisen, Parking Lot

### 4.5 Mehrere Projektkontexte
- Tool ist portabel — ein einziges Tool, beliebig viele Projektkontexte
- Jeder Projektkontext hat eine eigene TODO.md mit FG-Blöcken
- Kein Wechsel der App nötig beim Kontextwechsel

---

## 5. Visualisierung als primäre Interaktionsfläche

### 5.1 Designprinzip

Die Visualisierung ist nicht ein Feature der App — sie ist die App.
Alle Interaktionen (Tasks anlegen, Status ändern, Parking Lot verwalten,
Drift prüfen) erfolgen direkt in der visuellen Übersicht.
Es gibt keine separaten Formulare oder Listen-Ansichten als Primär-UI.

### 5.2 Visualisierungsmodell: Focus-Orbit

Das gewählte Modell strukturiert die Übersicht in vier klar getrennten Zonen:

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  [Task A ✓]  ←──┐                  ┌──→ [datei_a.md]  │
│  [Task B   ]  ←──┤   [ MISSION ]   ├──→ [datei_b.md]  │
│  [Task C ⚠]  ←──┘                  └──→ [datei_c.md ⚠]│
│                                                         │
│  ─────────────────── Parking Lot ─────────────────────  │
│  [Parked X]          [Parked Y]          [Parked Z]     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Zone links — Aktive Tasks:**
Alle Tasks die zur aktuellen Mission gehören.
Status (offen, in Arbeit, erledigt) und Drift-Indikator (⚠) direkt am Task.

**Zone Mitte — Mission:**
Die aktive Mission als zentrales Element.
Anker für alle Tasks und Parking-Lot-Verbindungen.

**Zone rechts — Quelldateien:**
Alle referenzierten .md-Dateien der aktiven Tasks.
Versions-Drift wird direkt an der Quelldatei markiert (⚠ + Versionsdelta).

**Zone unten — Parking Lot:**
Zurückgestellte Ideen und Tasks.
Optionale gestrichelte Verbindungslinien zur Mission oder zu aktiven Tasks
zeigen inhaltliche Bezüge ohne den Fokus zu unterbrechen.

### 5.3 Interaktionsmodell

Alle folgenden Aktionen erfolgen direkt in der Visualisierung:

| Aktion | Interaktion |
|---|---|
| Task-Detail öffnen | Task anklicken → Detailansicht mit ref-Datei, Version, Changelog-Delta |
| Quelldatei öffnen | Quelldatei-Node anklicken → Datei öffnen via File System Access API |
| Drift prüfen | ⚠-Markierung anklicken → Changelog-Einträge einblenden, bestätigen |
| Parking Lot → Task | Parking-Lot-Item anklicken → in aktive Tasks überführen |
| Task → Parking Lot | Task per Aktion parken → wandert in Parking-Lot-Zone |
| Task anlegen | Direkt in der Visualisierung, ohne Formular-Umweg |
| Mission setzen | Zentral-Element bearbeiten |
| AI Export | Dedizierter Export-Button → Prompt in Zwischenablage |

### 5.4 Visuelle Kodierung

| Element | Visuelle Sprache |
|---|---|
| Mission | Zentrales, hervorgehobenes Element |
| Aktive Tasks | Verbundene Nodes links der Mission |
| Erledigte Tasks | Gedimmt, mit Häkchen |
| Wiederkehrende Tasks | Eigenes Symbol am Task-Node |
| Versions-Drift | ⚠-Symbol am Task und an der Quelldatei |
| Quelldateien | Nodes rechts, gestrichelte Verbindung zum Task |
| Parking Lot | Eigene Zone unten, optionale gestrichelte Verbindung zur Mission |
| Subtasks | Untergeordnete Nodes am jeweiligen Task (Baum-Drill-down) |

---

## 6. Nicht-funktionale Anforderungen

| Anforderung | Beschreibung |
|---|---|
| Datensouveränität | Alle Daten liegen lokal im Projektordner (OneDrive-Sync eingeschlossen) |
| Offline-Fähigkeit | Keine Internetverbindung erforderlich |
| Plattform | Windows (primär), Browser-basiert |
| Obsidian-Kompatibilität | FG-Blöcke sind HTML-Kommentare — in Obsidian nicht sichtbar, nicht störend |
| EKB-Governance | DSGVO-Konformität durch Datenhaltung in bestehender EKB-Infrastruktur |

---

## 7. Abgrenzung zu bestehenden Tools

| Tool | Zweck | Verhältnis zu FocusGuard |
|---|---|---|
| TODO.md (EKB) | Vollständiger Task-Backlog — führendes System | FocusGuard erweitert um Fokus-Layer via FG-Blöcke — schreibt nie außerhalb dieser |
| MapApp (ProblemMap) | Visuelle Netzwerkansicht aller Tasks | Komplementär — MapApp zeigt Gesamtbild, FocusGuard den aktiven Fokus |
| Claude Desktop | KI-Arbeitsumgebung | FocusGuard liefert strukturierten Kontext via AI Export, kein Ersatz |
| Obsidian | Wissensbasis-Viewer und Graph | FG-Blöcke transparent — FocusGuard-Daten stören Obsidian-Darstellung nicht |
| Claude Projects | Kontext-Workspace per Projekt | FocusGuard ist projektordner-nativ, kein Claude-Projekt erforderlich |

---

## 8. Architektur — Grundprinzip

FocusGuard besteht aus zwei Komponenten:

**Komponente A — die App**
Ein portables, browser-basiertes Frontend ohne eigene Datenhaltung.
Es liest und schreibt ausschließlich FG-Blöcke innerhalb der TODO.md
des gewählten Projektkontexts.
Die App selbst liegt in einem eigenen, von Projekten getrennten Ordner.
Die interaktive Visualisierung ist die primäre UI-Schicht.

**Komponente B — FG-Blöcke in TODO.md**
Strukturierte HTML-Kommentar-Blöcke direkt in der projektspezifischen TODO.md.
Drei Block-Typen: `FG:MISSION`, `FG:TASK`, `FG:PARKING`.
Sie sind das einzige Persistenzmedium — kein separates Persistenzmedium existiert.
Block-interne Versionierung ersetzt einen zentralen Changelog.

---

## 9. Dateistruktur (konzeptionell)

```
OneDrive/
│
├── FocusGuard/                         ← App (einmalig, projektübergreifend)
│   ├── [App-Dateien]
│   ├── FG_GOVERNANCE_v03.md            ← Normatives Regelwerk
│   └── FG_PROJEKTKONTEXT_CLAUDE.md     ← LLM-Briefing
│
├── Enterprise Knowledgebase (EKB)/
│   ├── TODO.md                         ← Führendes System + FG-Blöcke
│   └── [weitere EKB-Dateien]
│
└── [AndererProjektordner]/
    └── TODO.md                         ← Führendes System + FG-Blöcke
```

**Hinweis:** FOCUS.md entfällt vollständig. Alle FocusGuard-Daten
leben als FG-Blöcke in der jeweiligen TODO.md des Projektkontexts.

---

## 10. Technologie-Stack

| Komponente | Technologie | Begründung |
|---|---|---|
| Frontend-Laufzeit | Browser (Chrome/Edge) | File System Access API für lokalen Dateizugriff |
| Frontend-Framework | React (UMD, kein Build-Schritt) | Kompatibel mit bestehender MapApp-Architektur |
| Visualisierung | D3.js | Interaktive, datengetriebene Graphdarstellung |
| Datenhaltung | FG-Blöcke in lokalen TODO.md-Dateien | Projektordner-nativ, OneDrive-sync-fähig, Obsidian-kompatibel |
| Datei-I/O | File System Access API | Lesen und Schreiben lokaler Dateien direkt aus dem Browser |
| Starten | VS Code Live Server | Bestehender Workflow, kein neues Tool |
| Hosting | Keines | Vollständig lokal |
| Backend | Keines | Bewusste Architekturentscheidung |

**Explizit abgelehnte Technologien:**

| Technologie | Ablehnungsgrund |
|---|---|
| Lovable | Cloud-Deployment, kein lokaler Dateizugriff möglich |
| Electron | Entwicklungsaufwand unverhältnismäßig für MVP |
| Supabase / Firebase | Widerspricht Datensouveränität und Offline-Anforderung |
| localStorage / IndexedDB | Daten nicht im Projektordner, nicht Obsidian-kompatibel |
| MCP-Server (eigener) | Valide Ausbaustufe, nicht für initialen Scope |

---

## 11. Offene konzeptionelle Fragen

1. **TRAINING.md:** Soll ein eigenständiges, reduziertes LLM-Dokument entstehen,
   oder ist FG_GOVERNANCE_v03.md selbst die Trainingsdatei?

2. **Visualisierung bei vielen Tasks:** Das Focus-Orbit-Modell skaliert bis ca. 8-10
   aktive Tasks lesbar. Darüber hinaus braucht es eine Filterlogik oder
   eine Zoom-/Drill-down-Ebene. Schwellwert und Verhalten noch offen.

**Geschlossene Fragen (zur Dokumentation):**

| Frage | Entscheidung |
|---|---|
| Verhältnis FOCUS.md zu TODO.md | FOCUS.md entfällt — FG-Blöcke in TODO.md sind das einzige Persistenzmedium |
| Cross-Ordner-Referenzen | `fg-ref-datei` akzeptiert absolute Pfade — kein Projektordner-Constraint |
| Task-Attribute | Definiert in FG_GOVERNANCE_v03.md §3.2 |
| Visuelles Design | MapApp-Design: DM Sans, #0f0f11, Projekt-Farbsystem |
| Wiederkehrende Tasks | Rhythmus + Faktor + berechnete `fg-naechste-faelligkeit` |
| Mission als eigener Typ oder Parent-Task | Eigener Block-Typ `FG:MISSION` — semantisch von Tasks getrennt |

---

## Changelog

| Version | Datum | Änderung | Quelle |
|---|---|---|---|
| 0.1.0 | 2026-05-12 | Initiale Erstellung aus Konzeptgespräch | Claude+Review |
| 0.2.0 | 2026-05-12 | Visualisierung als primäre UI verankert; Focus-Orbit-Modell; Interaktionsmodell; visuelle Kodierung; D3.js in Stack; offene Frage 5 ergänzt | Claude+Review |
| 0.3.0 | 2026-05-18 | Architektur-Update: FOCUS.md entfällt; FG-Blöcke in TODO.md als einziges Persistenzmedium; §2 Zielbild angepasst; §3 Leitprinzip "Single Source of Truth" ergänzt; §4 Task-Management um Wiederkehrend-Logik und Mission-Pflicht erweitert; §5.4 visuelle Kodierung für wiederkehrende Tasks ergänzt; §6 Obsidian-Kompatibilität präzisiert; §7 Abgrenzung aktualisiert; §8 Architektur neu beschrieben; §9 Dateistruktur ohne FOCUS.md; §11 offene Fragen bereinigt | Claude+Review |
