# CLAUDE.md

Kooperatives Online-Multiplayer-Spiel in Godot 4: Crews aus 1–4 Spielern arbeiten als Boxencrew, KI-Fahrer fahren auf einem Oval, Crews sabotieren sich gegenseitig. Konzept: `ROADMAP.md`. Arbeitsplan mit Aufgaben und UAT-Gates: `MILESTONES.md`.

## Rahmen

- **Engine:** Godot 4.x — TODO: exakte Version eintragen (z. B. 4.5.1). Ausschließlich Godot-4-APIs verwenden, keine Godot-3-Syntax (`yield`, `export var`, `onready var`, `KinematicBody` usw.).
- **Sprache:** GDScript mit statischen Typen (`var speed: float = 5.0`, `func foo() -> void`).
- **Entwickler:** Solo, Godot-Neuling. Änderungen kurz erklären (was und warum), besonders bei Netzwerk und Physik.
- **Godot-MCP-Server:** TODO: Name und Funktionen eintragen. Zum Starten des Projekts und Lesen der Debug-Ausgabe verwenden.

## Architekturregeln

- **Netzwerk:** Listen-Server, Godot High-Level Multiplayer.
  - Host-autoritativ: Rennsimulation, Autos, Boxenstopp-Zustand, Teile, Sabotage, Strafen, Punkte.
  - Client-autoritativ: nur Bewegung und Blickrichtung der eigenen Spielfigur.
  - Interaktionen sind RPC-Anfragen an den Host; der Host prüft und führt aus.
  - Physik gehaltener Objekte läuft nur auf dem Host, Clients interpolieren.
- **Logik vom Rendering trennen:** Regeln (Boxenstopp, Rennen, Punkte, Sabotage) in eigenen Klassen ohne Abhängigkeit von Szenen, damit sie headless testbar sind.
- **Zustandsänderungen über eine Stelle:** z. B. `PitStop.request_action(player, action)` statt verstreuter direkter Änderungen.
- **Tuning-Werte in Resources (`.tres`):** Wabbel-Parameter, Verschleiß, Standzeiten usw. nie als Magic Numbers im Code.
- **Kleine Szenen:** Lieber viele kleine `.tscn`-Dateien als eine große.
- **Logging:** Zentraler Logger mit Kategorien (`NET`, `PIT`, `RACE`, `SABOTAGE`), jede Zeile mit Peer-ID.

## Ordnerstruktur

```
autoload/  core/  race/  pit/  player/  sabotage/  net/  ui/  art/  audio/  tests/  tools/
```

## Befehle

TODO: ergänzen, sobald eingerichtet.

- Tests headless ausführen: `godot --headless ...`
- Host plus N Clients starten: `tools/run_multi.sh <N>` (Argumente `--host`, `--join <ip>`, `--autostart`)

## Arbeitsweise

- **Phasen-Gates einhalten:** Nur Aufgaben der aktiven Phase aus `MILESTONES.md` bearbeiten, nie aus einer gesperrten (🔒). Eine Phase ist erst nach bestandenem UAT abgeschlossen; manuelle UAT-Ergebnisse trägt nur der Entwickler ein. Skills: `/next-task`, `/uat`.
- Pro Aufgabe ein klares Ziel; nach der Umsetzung Projekt starten und Log auf Fehler prüfen.
- Neue Spiellogik bekommt Tests.
- Netzwerkänderungen mit mindestens Host plus einem Client prüfen.
- Am Ende sagen, was der Entwickler selbst im Spiel ausprobieren sollte (Gefühl, Timing).
