---
name: uat
description: Führt das UAT-Quality-Gate der aktuellen Phase aus MILESTONES.md durch (automatische Vorprüfung plus geführter manueller Test). Nur auf ausdrücklichen Aufruf verwenden.
disable-model-invocation: true
---

# UAT-Quality-Gate der aktuellen Phase

1. **Phase bestimmen:** Aktive Phase aus der Status-Übersicht in `MILESTONES.md`. Sind noch Aufgaben offen, liste sie auf und brich ab.
2. **Status auf 🧪 setzen.**
3. **Automatische Vorprüfung** laut Abschnitt „UAT Phase <N>“ ausführen (Tests, Projektstart über den Godot-MCP-Server, Log-Prüfung, Netzwerk- und Lasttests der Phase). Ergebnis je Punkt melden. Schlägt etwas fehl: Status auf ❌, Nacharbeitsaufgaben `M<N>.F*` anlegen, Protokolleintrag schreiben, abbrechen.
4. **Manuellen Test vorbereiten:** Dem Entwickler eine Checkliste der Testfälle U<N>.* geben, mit konkreten Startbefehlen (z. B. `tools/run_multi.sh 2`, `--scenario pitstop`) und worauf er achten soll. Muss-Testfälle hervorheben.
5. **Auf Ergebnisse warten.** Der Entwickler meldet je Testfall ✅/❌ und Befunde. Setze nie selbst ein ✅ für einen manuellen Testfall.
6. **Befunde einordnen:** Mit dem Entwickler Schweregrad festlegen (Blocker / Major / Minor).
7. **Gate-Entscheidung** nach der Gate-Regel in `MILESTONES.md`, vom Entwickler bestätigen lassen:
   - **Bestanden:** Ergebnisse in die Tabelle eintragen, Phase auf ✅ mit Datum, nächste Phase auf ⬜, Minor-Befunde in den Backlog, Git-Tag `gate-m<N>` setzen.
   - **Nicht bestanden:** Phase auf ❌, je Blocker/Major eine Aufgabe `M<N>.F<k>` (mit Prompt und Akzeptanz) in der Phase anlegen.
8. **Protokoll:** Eintrag nach Vorlage im Abschnitt „UAT-Protokoll“ ergänzen (neuester oben), committen (`UAT M<N> Durchlauf <k>: bestanden|nicht bestanden`).
