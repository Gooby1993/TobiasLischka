---
name: explain
description: Erklärt eine Datei, Szene oder ein Godot-Konzept (Signal, RPC, Autoload, MultiplayerSynchronizer usw.) einsteigerfreundlich am Code dieses Projekts. Verwenden, wenn der Entwickler fragt, wie etwas funktioniert oder warum es so gebaut ist.
---

# Godot und Projektcode verständlich erklären

Der Entwickler ist Solo-Entwickler und Godot-Neuling. Argument: ein Pfad (`.gd`, `.tscn`, `.tres`), ein Begriff (z. B. „RPC“, „Autoload“) oder eine Frage („Warum läuft die Kistenphysik nur auf dem Host?“).

1. **Kontext sammeln:** Die genannte Datei bzw. die Stellen im Projekt lesen, an denen das Konzept vorkommt. Bei Szenen auch die angehängten Skripte und die Knotenhierarchie ansehen.
2. **Erklären, in dieser Reihenfolge:**
   - **Was ist das?** Ein bis zwei Sätze ohne Fachjargon, mit einer Alltagsanalogie, wenn sie passt.
   - **Wo steckt es in unserem Projekt?** Konkrete Dateien und Zeilen (`pfad:zeile`), kurze Code-Ausschnitte (höchstens ~15 Zeilen).
   - **Warum ist es so gebaut?** Bezug auf die Architekturregeln in `CLAUDE.md` (z. B. Host-Autorität, Logik vom Rendering getrennt, Tuning in Resources).
   - **Typische Stolperfallen** für Einsteiger, inklusive Godot-3-Altlasten in Tutorials (z. B. `yield` → `await`, `export var` → `@export var`).
   - **Selbst ausprobieren:** Eine kleine Änderung oder Beobachtung, mit der der Entwickler es im Editor oder im Spiel nachvollziehen kann (z. B. „Setze einen Haltepunkt in …“, „Ändere X und beobachte Y“).
3. **Optional vertiefen:** Auf den passenden Abschnitt der offiziellen Godot-4-Dokumentation verweisen (docs.godotengine.org, stabile Version).
4. **Nichts ändern:** Dieser Skill erklärt nur. Code wird nur angepasst, wenn der Entwickler das danach ausdrücklich wünscht.
