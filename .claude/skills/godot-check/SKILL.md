---
name: godot-check
description: Prüft geänderte GDScript-, Szenen- und Resource-Dateien auf Godot-3-Syntax, fehlende statische Typen, Magic Numbers und Verstöße gegen die Architekturregeln aus CLAUDE.md. Vor einem Commit oder auf Aufruf verwenden.
---

# Godot-4-Regelprüfung

Argument (optional): Pfade oder ein Git-Bereich (z. B. `HEAD~3`). Standard: alle gegenüber `HEAD` geänderten und neuen Dateien (`git diff --name-only HEAD` plus ungetrackte), gefiltert auf `.gd`, `.tscn`, `.tres`.

Prüfe jede Datei auf folgende Punkte und melde Funde als `pfad:zeile – Problem – Vorschlag`:

1. **Godot-3-Syntax und -APIs** (Blocker):
   - `yield(` → `await`; `export var` → `@export var`; `onready var` → `@onready var`; `tool` → `@tool`
   - `KinematicBody`, `Spatial`, `RigidBody` (ohne 3D/2D), `Area` ohne 3D → Godot-4-Klassen
   - `move_and_slide(` mit Argumenten → `velocity` setzen, dann `move_and_slide()`
   - `connect("signal", self, "method")` → `signal.connect(method)`
   - `remote`/`master`/`puppet`/`remotesync` → `@rpc(...)`; `rpc_id`-Aufrufe auf Methoden ohne `@rpc`
   - `get_tree().network_peer`, `NetworkedMultiplayerENet` → `multiplayer.multiplayer_peer`, `ENetMultiplayerPeer`
   - `setget` → Properties mit `set`/`get`
2. **Statische Typen** (Major): Variablen, Parameter und Rückgabewerte ohne Typ (`var x = …` ohne `:`/`:=`, `func f(a)` ohne Typ, fehlendes `-> Typ`).
3. **Magic Numbers** (Major): Zahlen für Spielgefühl oder Balancing (Kräfte, Zeiten, Geschwindigkeiten, Verschleiß, Strafen) direkt im Code statt in einer `.tres`-Resource. Ausgenommen: 0, 1, -1, 2 in offensichtlicher Mathematik, Array-Indizes, Konstanten mit sprechendem Namen für technische Werte.
4. **Architekturregeln** (Major), siehe `CLAUDE.md`:
   - Regel-Klassen in `core/` (Boxenstopp, Rennen, Punkte, Sabotage) dürfen keine Szenen, Knoten-Pfade (`$`, `get_node`) oder Rendering verwenden.
   - Spielzustand (Teile, Autos, Boxenstopp, Sabotage, Punkte) wird nur auf dem Host geändert; Client-Code ändert ihn nicht direkt, sondern schickt eine RPC-Anfrage.
   - Zustandsänderungen laufen über eine zentrale Methode (z. B. `request_action`), nicht verstreut.
   - Ausgaben über den `Log`-Autoload mit Kategorie statt `print()`.
5. **Kleinkram** (Minor): sehr große Szenen (> ~300 Zeilen `.tscn`), Dateinamen nicht in `snake_case`, Klassen ohne `class_name` wo andere Dateien sie brauchen.

Ausgabe:
- Zusammenfassung: Anzahl Funde je Schweregrad.
- Liste der Funde, Blocker zuerst.
- Für Blocker und eindeutige Fälle anbieten, sie direkt zu beheben. Nur nach Zustimmung ändern, danach Tests ausführen und Projekt über den MCP-Server starten.
- Keine Funde: kurz bestätigen, welche Dateien geprüft wurden.
