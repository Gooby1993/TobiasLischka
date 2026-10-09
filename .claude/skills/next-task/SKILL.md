---
name: next-task
description: Arbeitet die nächste offene Aufgabe aus MILESTONES.md ab, mit Prüfung des Phasen-Gates. Nur auf ausdrücklichen Aufruf verwenden.
disable-model-invocation: true
---

# Nächste Meilenstein-Aufgabe umsetzen

Optionales Argument: eine Aufgaben-ID (z. B. `M1.3`). Ohne Argument die nächste offene Aufgabe nehmen.

1. **Gate prüfen:** Lies die Status-Übersicht in `MILESTONES.md`. Bestimme die aktive Phase (Status ⬜, 🔨 oder ❌). Gehört die gewünschte Aufgabe zu einer Phase mit 🔒, brich ab und erkläre, welches Gate zuerst bestanden werden muss. Steht die aktive Phase auf 🧪, brich ab und verweise auf `/uat`.
2. **Aufgabe wählen:** Die erste nicht abgehakte Aufgabe der aktiven Phase (Nacharbeitsaufgaben `M<N>.F*` zuerst), oder die als Argument übergebene.
3. **Status setzen:** Phase auf 🔨, falls sie ⬜ oder ❌ war.
4. **Kurz planen:** Nenne dem Entwickler in 3–5 Zeilen, was du tun wirst. Bei Aufgaben mit „Plan-Modus“ im Prompt: erst Plan vorlegen und auf Freigabe warten.
5. **Umsetzen** nach den Regeln in `CLAUDE.md`. Nur diese eine Aufgabe, kein Vorgriff auf spätere Aufgaben.
6. **Akzeptanzkriterien prüfen:** Tests ausführen, Projekt über den Godot-MCP-Server starten, Log lesen; bei Netzwerkthemen Host plus mindestens einen Client. Jedes Kriterium einzeln als erfüllt/nicht erfüllt melden.
7. **Abschließen** nur wenn alle Kriterien erfüllt sind: Checkbox in `MILESTONES.md` abhaken, committen (`M<N>.<x>: <Titel>`).
8. **Übergabe an den Entwickler:**
   - Was geändert wurde (kurz, verständlich für einen Godot-Neuling)
   - Was er selbst im Spiel ausprobieren soll (Gefühl, Timing, Tuning-Werte in welcher `.tres`)
   - Ist das die letzte Aufgabe der Phase: darauf hinweisen, dass jetzt `/uat` dran ist.
9. **Vorgemerkte Skills empfehlen:** Prüfe die Tabelle „Skills und Hooks“ in `MILESTONES.md`. Für jeden Eintrag mit Status 📌, dessen Auslöser jetzt erreicht ist (aktive Phase bzw. gerade abgeschlossene Aufgabe), dem Entwickler in 1–2 Sätzen empfehlen, was er bringt, und fragen, ob er angelegt werden soll. Nur nach Zustimmung anlegen (`.claude/skills/<name>/SKILL.md` bzw. Hook in `.claude/settings.json`), Status auf ✅ setzen, committen (`skill: <name> anlegen`). Bei Ablehnung Status ⏭. Bereits abgelehnte Einträge nicht erneut empfehlen.
