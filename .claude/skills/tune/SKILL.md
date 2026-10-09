---
name: tune
description: Passt Tuning-Werte (.tres-Resources) nach einer Gefühlsbeschreibung des Entwicklers an, z. B. „Schlauch zu träge“ oder „Reifen fliegen zu weit“. Nur auf ausdrücklichen Aufruf verwenden.
disable-model-invocation: true
---

# Spielgefühl über Tuning-Werte einstellen

Argument: eine Beschreibung in Alltagssprache, was sich falsch anfühlt (z. B. „Schlauch zu träge“, „Wagenheber zu schnell“). Ohne Argument: alle Tuning-Resources auflisten und fragen, was sich falsch anfühlt.

1. **Resources finden:** Alle `*_tuning.tres` und andere Tuning-Resources im Projekt suchen (z. B. `player/player_tuning.tres`, `hands_tuning.tres`, `fuel_tuning.tres`, `race_tuning.tres`, `sabotage_tuning.tres`). Dazu die zugehörige Resource-Klasse (`.gd`) lesen, um zu verstehen, was jeder Wert bewirkt.
2. **Zuordnen:** Bestimmen, welche Werte das beschriebene Gefühl beeinflussen, und in 1–2 Sätzen erklären, warum (z. B. „Federsteifigkeit bestimmt, wie stark der Schlauch zur Hand gezogen wird; Dämpfung, wie schnell er zur Ruhe kommt“).
3. **Vorschlag machen:** Höchstens 1–3 Werte gleichzeitig ändern, in kleinen Schritten (typisch 15–30 %). Als Tabelle zeigen: Datei · Wert · alt · neu · erwartete Wirkung.
4. **Ändern:** Nur die `.tres`-Datei(en) anpassen, keinen Code. Fehlt ein benötigter Wert in der Resource und steht er als Zahl im Code, diesen Verstoß gegen CLAUDE.md melden und anbieten, ihn in die Resource zu verschieben (nicht ungefragt tun).
5. **Prüfen:** Projekt bzw. passendes Szenario über den Godot-MCP-Server starten (z. B. `--scenario pitstop`) und das Log auf Fehler prüfen. Betroffene Tests ausführen, falls vorhanden.
6. **Übergabe:** Dem Entwickler sagen, was er gezielt ausprobieren soll, und anbieten:
   - „mehr davon“ (nächster Schritt in dieselbe Richtung),
   - „zu viel“ (halber Schritt zurück),
   - „zurück“ (alte Werte wiederherstellen).
7. **Festhalten:** Erst wenn der Entwickler zufrieden ist, committen (`tune: <was> (<Datei>)`) und die Werte mit kurzer Begründung im Commit-Text nennen. Zwischenschritte nicht committen.
