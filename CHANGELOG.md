# Changelog

---

## [11.0.0] - 2026-05-13

### Zweck dieser Version
Version 11 integriert den Network-Graph-Prototyp (v4) in die bestehende FocusGuard-Architektur. Schwerpunkt ist die vollständige Überarbeitung des View-Overlays, die Einführung von Datei-Nodes als eigenen visuellen Entitätstyp, die Korrektur des Edge-Rendering-Bugs (unsichtbare Verbindungslinien) sowie die korrekte Skalierung der Gravitations-Kräfte.

### Dateien / Ablage
- Neue App-Version: `FocusGuard/focusguard_11.html`
- Vorherige Version bleibt erhalten: `FocusGuard/Archiv/focusguard_10_1.html` (Archiv)
- Changelog fortgeschrieben: `FocusGuard/CHANGELOG.md`
- Governance-Dokument: `FocusGuard/FGD_Konzept_v02.md`

### Added

**View Overlay — 3-Gruppen-Struktur**
- Vollständige Neustrukturierung des View-Overlays in drei semantische Gruppen mit farbigen Gruppen-Headern:
  - **Tasks** (grüner Dot): Icons (Sichtbarkeit), Verbindungslinien, Gravitation (Parent-Child)
  - **Dateien** (blaugrauer Dot): Icons (Sichtbarkeit), Verbindungslinien, Gravitation (Anziehung)
  - **Kontext / Dep.** (magenta Dot): Verbindungslinien, Gravitation, Kurvenstärke
- Kurvenstärke-Regler für Kontext-Linien: 0% = gerade Linie, 100% = starke Bézierkurve
- Overlay scrollbar bei kleinen Viewports

**Datei-Nodes (neu)**
- `.md`-Dateien werden im Netzwerkgraph als eigenständige Nodes mit Datei-Icon (SVG-Pfad mit Dog-Ear) dargestellt — kein Kreis
- Datei-Status visuell kodiert: `drift` = amber, `aktiv` = EKB-Grün, `ok` = grün, `ref` = blaugrau
- Drift-Symbol (⚠) wird direkt im Datei-Icon angezeigt
- Versionsnummer (`vX.X.X`) unterhalb des Icons
- Datei-Nodes haben eigene D3-Force-Gruppe: sanfte Soft-Force Richtung rechtes Drittel des Canvas
- Datei-Gravitations-Regler steuert die Anziehungskraft zu referenzierenden Tasks
- Klick auf Datei-Node öffnet Sidebar mit vollständigem Dateipfad und Liste referenzierender Tasks

**Sidebar — Datei-Nodes**
- Neuer Sidebar-Typ für Datei-Nodes: zeigt vollständigen Dateipfad, Version, Status, referenzierende Tasks
- Drift-Box wird bei Datei-Nodes mit Status `drift` angezeigt
- Copy-Button für Dateipfad

**Sidebar — Task-Nodes**
- Fortschrittsbalken mit Prozentzahl (kumuliert aus Child-Elementen)
- Quelldatei-Bereich mit vollständigem Pfad (`refFile`) und Versionsvergleich bei Drift
- Abhängigkeiten in magenta dargestellt (← blockiert durch / → ermöglicht)

**Statusbar (neu)**
- Permanente Statusleiste am unteren Rand der Graph-Ansicht
- Zeigt: aktive Missionen, Drift-Warnungen, offene/erledigte Tasks, letzten Indizierungszeitpunkt

**Reindex-Button (neu)**
- `↺ Indizieren`-Button erscheint in der Header-Leiste wenn der Netzwerk-Tab aktiv ist
- Aktualisiert den Indizierungszeitpunkt in der Statusleiste

**Kontext-Linien**
- Magenta, gestrichelt, Bézierkurve
- Kurvenstärke live steuerbar ohne Simulation-Rebuild

### Fixed

**Edge-Rendering-Bug (kritischer Fix)**
- In v10.1 wurden Edge-Selections mit `l.source` und `l.target` referenziert bevor D3 `forceLink` die String-IDs zu Objekt-Referenzen aufgelöst hatte. Das Ergebnis: alle Koordinaten `undefined`, keine Verbindungslinie sichtbar.
- Fix: SVG-Elemente werden weiterhin vor dem ersten Tick erstellt, aber alle Koordinaten-Zuweisungen (`x1/y1/x2/y2`) erfolgen ausschließlich im `tick`-Handler. Dort sind `l.source` und `l.target` garantiert Objekte mit `.x` und `.y`.

**Gravitations-Skalierung**
- In v10.1 wurde der Slider-Wert mit `* 0.016` multipliziert, was faktisch keine wahrnehmbare Gravitationsänderung erzeugte.
- Fix: Slider-Wert (0–100%) mappt direkt auf D3 Link-Strength (0.0–1.0). Bei 50% ist die Gravitationswirkung spürbar, bei 100% kollabiert der Baum eng um die Mission.

**Datei-Node-Separation**
- Datei-Nodes wurden in v10.1 nicht gerendert; sie kollabieren mit anderen Nodes übereinander.
- Fix: Eigene `forceX`/`forceY` Soft-Forces für `isFile`-Nodes; eigene Collide-Radien; eigene Charge-Stärken.

**Hierarchische Gravitation**
- Missions werden mit `forceX` auf fixe Horizontalpositionen stabilisiert (EKB links, FG rechts) damit sie sich nicht überlagern.
- Parent-Child- und Mission-Child-Links haben separate Stärken aus dem Task-Gravitations-Regler.
- Dependency- und Kontext-Links haben eigene, deutlich schwächere Stärken aus dem Kontext-Regler.

### Changed

**View Overlay**
- Alle bisherigen Slider aus v10.1 bleiben erhalten (Parent-Opacity, Dep-Opacity, Link-Strength, Kontext-Strength, Kontext-Width, Projektfarbe)
- Umbenennung und Neuordnung in 3 Gruppen für semantische Klarheit
- Gravity-Slider triggern einen vollständigen Simulation-Rebuild; alle anderen Slider wirken live ohne Rebuild

**`updateViewSettings()`**
- Vollständig überarbeitet: liest jetzt 9 Slider statt 7
- Gravity-Werte werden direkt als D3-Stärken gespeichert (nicht als Prozent-Multiplikatoren)

**`buildGraph()`**
- Simulation-Setup überarbeitet: Kräfte werden bei jedem Rebuild aus den aktuellen `viewSettings`-Werten berechnet
- File-Nodes werden in separaten D3-Layer-Gruppen verwaltet (Render-Reihenfolge: Kontext → File-Edges → Task-Edges → File-Nodes → Task-Nodes)
- Alle Edge-Selections werden ausschließlich im `tick`-Handler befüllt
- Neue Datei-Node-Rendering-Logik (SVG-Path statt Circle)

**`lineOpacity()`**
- Unterscheidet jetzt zwischen Task-Edges (`parentTransparency`), File-Edges (`fileEdgeTransparency`) und Kontext-Edges (`contextTransparency`)

**Graph-Höhe**
- `graph-view` Höhe auf `calc(100vh - 52px - 28px)` angepasst um Platz für die neue Statusbar zu schaffen

### Technische Hinweise für Claude als FocusGuard-Architekt

- `FILE_NODES_DEF` und `FILE_LINKS_DEF` sind neue Top-Level-Konstanten. Sie werden beim `buildGraph()`-Aufruf geklont, damit D3 Positions-Properties darauf mutieren kann ohne die Originaldaten zu verändern.
- Gravity-Slider (`link-strength-slider`, `file-grav-slider`, `context-strength-slider`) lösen `gravRebuild()` aus, das `updateViewSettings()` + `buildGraph()` kombiniert. Alle anderen Slider rufen nur `updateViewSettings()` auf.
- Die Kontext-Kurvenstärke (`contextCurvature`) wird im `tick`-Handler live ausgewertet — kein Rebuild nötig, da die Bezier-Kurve bei jedem Tick neu berechnet wird.
- `openFileSidebar(d)` ist eine neue Funktion analog zu `openSidebar(node)` für Task-Nodes.
- `updateStatusbar()` wird am Ende von `buildGraph()` aufgerufen.
- `reindex()` aktualisiert nur den Zeitstempel in der Statusbar; echte Datei-I/O ist noch nicht implementiert (Architekturentscheidung offen).

### Bekannte Einschränkungen / offene Architekturpunkte
- File-Nodes sind noch statisch in `FILE_NODES_DEF` definiert. Echte Integration über File System Access API (automatisches Traversieren des Projektordners) ist als nächste Ausbaustufe vorgesehen (FG_INDEX.md-Konzept, siehe FG_GOVERNANCE_v02.md).
- Keine Persistenz von View-Einstellungen oder Gravitations-Konfiguration.
- Drift-Erkennung ist noch simuliert (statische `refVer`/`refVerAkt`-Felder in den Node-Daten). Echte Drift-Prüfung erfordert File System Access API + Header-Parsing der referenzierten Dateien.
- `reindex()` aktualisiert nur den Timestamp; kein echtes Re-Lesen der Dateien.
- File-Node-Sidebar hat noch keinen „Datei öffnen"-Button mit File System Access API-Integration.

---

## [10.1.0] - 2026-05-12

### Zweck dieser Version
Version 10.1 konsolidiert die Graph-Überarbeitungen aus dem Arbeitsstand nach Version 9. Schwerpunkt: Kontextverbindungen als geschwungene Magenta-Linien, steuerbare Anziehungskraft für Kontextlinks, robusterer TODO.md-Parser für kurze IDs.

### Dateien / Ablage
- Neue App-Version: `FocusGuard/focusguard_10_1.html`
- Changelog fortgeschrieben: `FocusGuard/CHANGELOG.md`

### Added
- Kontextverbindungen als geschwungene Magenta-Linien (`stroke: #ff2bd6`, `stroke-dasharray: 8,5`)
- Dedizierter Slider für Kontext-Anziehungskraft (Standard: 0 — visuelle Darstellung ohne Layout-Einfluss)
- Slider für Kontextlinien-Linienstärke
- TODO-Parser: robustere Erkennung kurzer IDs (`B-01`, `C-01`), Kontext-/Bezugs-/Verknüpfungsspalten, freie Kontextzeilen außerhalb der Tabelle
- Dependency-Dubletten werden nicht als Kontextkanten doppelt gerendert

### Fixed
- Kontextverbindungen wurden nicht angezeigt wenn IDs kürzer als `XXX-0000`-Format waren
- Parser-Abbruch bei fehlenden Kontext-Spalten-Headern

### Changed
- View-Overlay um Kontext-Anziehungskraft-Slider erweitert
- Kontext-Stärke default auf 0 gesetzt (explizit durch Nutzer aktivierbar)

---

## [9.0.0] - 2026-05-12

### Zweck dieser Version
Version 9 konsolidiert die seit Version 8.1 entstandenen Bugfixes und UI-Erweiterungen. Schwerpunkt ist die Stabilisierung der Graph-Interaktion, bessere Referenzierbarkeit von Detaildaten und steuerbare visuelle Darstellung von Linien, Fortschritt und Projektfarben.

### Dateien / Ablage
- Neue App-Version: `FocusGuard/focusguard_9.html`
- Changelog fortgeschrieben: `FocusGuard/CHANGELOG.md`

### Added
- Globale Suchfunktion im Header (Task-ID, Titel, Projekt, Priorität, Status, Zuweisung, Typ, Parent, Dependency)
- Projektfarben im View-Fenster änderbar (Dropdown, Standard-Palette, HEX-Eingabe)
- Sidebar um Referenzpfade ergänzt: `TODO.md Pfad`, `FOCUS.md Pfad` mit Copy-Button
- View-Fenster: vereinfachtes Linienmodell (Parent/Child-Transparenz, Kontextlinien-Transparenz, initiale Linienstärke, Link-Anziehungskraft, Projektfarbeneditor)
- Fortschritts-Pie-Chart für Tasks mit Child-Elementen
- Highlighting von ausgewählten Nodes im Graph
- Klickbare Subtasks in der Sidebar
- Dashboard-Task-Klicks öffnen Graph-Ansicht mit gehighlightetem Node

### Fixed
- Rendering-Abbruch durch fehlende Progress-Funktion
- Zoom-/Drag-Verhalten stabilisiert: keine Positions-Sprünge nach Zoom
- Sidebar öffnet wieder nach Klick auf Nodes nach Zoom-Interaktion
- Dependency-Indikator als gelber Kreis entfernt — nur noch gestrichelte Linien
- Projekt-/Mission-Farben von Eigenschaftsfarben entkoppelt
- Parent/Child-Verbindungen: 0% Transparenz = vollständig sichtbar, weiß

### Changed
- Copy-Logik auf referenzrelevante Felder reduziert
- Dashboard-Fortschritt auf `calculateProgress()` umgestellt
- Linienstärke-Modell vereinfacht: ein Regler, automatische Skalierung nach Typ und Ebene

