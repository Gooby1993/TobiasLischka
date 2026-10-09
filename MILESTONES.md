# Meilensteine und UAT-Quality-Gates

Ausführbarer Arbeitsplan zur `ROADMAP.md`. Jede Phase besteht aus kleinen **Aufgaben** (eine Aufgabe ≈ eine Claude-Code-Sitzung) und endet mit einem **UAT** (User Acceptance Test). Das UAT ist ein **Quality Gate**: Die nächste Phase beginnt erst, wenn das Gate der aktuellen Phase bestanden ist.

---

## Status-Übersicht

| Phase | Titel | Status | Gate bestanden am | Git-Tag |
|---|---|---|---|---|
| 0 | Setup und Grundverständnis | ⬜ offen | – | – |
| 1 | Kern-Prototyp Boxenstopp | 🔒 gesperrt | – | – |
| 2 | Rennsimulation und Funk | 🔒 gesperrt | – | – |
| 3 | Multiplayer-Fundament | 🔒 gesperrt | – | – |
| 4 | Mehrere Crews und Match-Flow | 🔒 gesperrt | – | – |
| 5 | Sabotage und Verteidigung | 🔒 gesperrt | – | – |
| 6 | Online-Spiel über das Internet | 🔒 gesperrt | – | – |
| 7 | Grafik, Sound, Juice | 🔒 gesperrt | – | – |
| 8 | Feinschliff und Release | 🔒 gesperrt | – | – |

Statuswerte: 🔒 gesperrt · ⬜ offen · 🔨 in Arbeit · 🧪 im UAT · ✅ bestanden · ❌ UAT nicht bestanden (Nacharbeit)

---

## So arbeitest du mit dieser Datei

### Aufgaben abarbeiten

1. Claude Code lokal im Projektordner starten (Godot-MCP-Server verbunden).
2. `/next-task` aufrufen, oder den Prompt-Vorschlag der Aufgabe kopieren.
3. Claude setzt **genau eine Aufgabe** um, prüft die Akzeptanzkriterien selbst (Tests, Projekt starten, Log lesen), hakt die Aufgabe ab und committet.
4. Du probierst kurz selbst aus, was Claude dir am Ende nennt.

### UAT durchführen (am Ende jeder Phase)

1. `/uat` aufrufen. Claude führt die **automatische Vorprüfung** aus (alle Tests, Projekt starten, Log fehlerfrei, offene Aufgaben = 0) und setzt den Status auf 🧪.
2. **Du** spielst die manuellen Testfälle durch und trägst je Testfall ✅ oder ❌ ein.
3. Gefundene Probleme kommen ins UAT-Protokoll, mit Schweregrad:
   - **Blocker:** Kern-Funktion geht nicht oder stürzt ab
   - **Major:** funktioniert, aber das Phasenziel ist nicht erreicht (zum Beispiel „fühlt sich nicht gut an“)
   - **Minor:** Schönheitsfehler, kann in eine spätere Phase
4. **Gate-Regel:** Bestanden, wenn alle **Muss**-Testfälle ✅ sind und kein Blocker oder Major offen ist. Minor-Punkte wandern in den Backlog am Ende dieser Datei.
5. Bestanden: Status ✅, Git-Tag `gate-m<N>` setzen, nächste Phase auf ⬜.
   Nicht bestanden: Status ❌. Claude legt für jedes Blocker- oder Major-Problem eine Nacharbeitsaufgabe (`M<N>.F1`, `M<N>.F2`, …) in der Phase an. Danach wird das UAT wiederholt.

### Regeln für Claude

- Nur Aufgaben aus der Phase mit Status ⬜, 🔨 oder ❌ bearbeiten. Nie eine Aufgabe aus einer 🔒-Phase beginnen.
- Aufgaben in der angegebenen Reihenfolge abarbeiten, außer der Entwickler sagt etwas anderes.
- Eine Aufgabe gilt erst als erledigt, wenn alle Akzeptanzkriterien geprüft sind.
- Das UAT-Ergebnis (✅/❌ bei manuellen Testfällen) trägt **nur der Entwickler** ein bzw. bestätigt es. Claude setzt den Gate-Status nie selbst auf ✅.

---

## Phase 0: Setup und Grundverständnis

**Ziel:** Werkzeugkette steht, zwei Instanzen verbinden sich, Claude kann Projekt starten und Logs lesen.

### Aufgaben

- [ ] **M0.1 Projekt anlegen**
  - Prompt: „Lege ein Godot-4-Projekt im Repo-Root an: `project.godot`, Ordnerstruktur laut CLAUDE.md, Godot-`.gitignore`, eine leere Hauptszene `core/main.tscn`. Starte das Projekt über den MCP-Server und prüfe das Log.“
  - Akzeptanz: Projekt startet ohne Fehler im Log; Ordner existieren; `.godot/` ist ignoriert.
- [ ] **M0.2 Logger und Debug-Overlay**
  - Prompt: „Baue den Autoload `Log` mit Kategorien (NET, PIT, RACE, SABOTAGE, CORE), Peer-ID in jeder Zeile, und ein ein-/ausblendbares Debug-Overlay (F3) mit FPS.“
  - Akzeptanz: Log-Zeilen haben das Format `[KATEGORIE][peer] Text`; Overlay lässt sich umschalten.
- [ ] **M0.3 Test-Framework**
  - Prompt: „Richte GUT (oder gdUnit4) ein, schreibe einen Beispieltest und ein Skript `tools/run_tests.sh`, das alle Tests headless ausführt. Trage den Befehl in CLAUDE.md ein.“
  - Akzeptanz: `tools/run_tests.sh` läuft durch und meldet einen grünen Test; Exit-Code 0.
- [ ] **M0.4 First-Person-Controller**
  - Prompt: „Baue einen First-Person-Controller (`player/player.tscn`): laufen, springen, Maus-Look, Maus einfangen/freigeben mit Esc. Bewegungswerte in einer Resource `player/player_tuning.tres`. Testraum `core/test_room.tscn` mit Boden und Wänden.“
  - Akzeptanz: Figur bewegt sich im Testraum; Werte kommen aus der Resource.
- [ ] **M0.5 Objekt aufheben (noch ohne Wabbeln)**
  - Prompt: „Ermögliche es, `RigidBody3D`-Kisten per Raycast anzuschauen (Hervorhebung), mit E aufzuheben, mit E abzulegen und mit Linksklick zu werfen.“
  - Akzeptanz: Kiste lässt sich aufheben, ablegen, werfen; keine Fehler im Log.
- [ ] **M0.6 Netzwerk-Grundgerüst und Multi-Instanz-Start**
  - Prompt: „Baue den Autoload `Network` (ENet, Host/Join per Kommandozeilen-Argumenten `--host`, `--join <ip>`, `--autostart`) und `tools/run_multi.sh <N>`, das Host plus N−1 Clients startet. Spieler werden per `MultiplayerSpawner` gespawnt, die eigene Bewegung ist client-autoritativ und wird synchronisiert. Trage den Befehl in CLAUDE.md ein.“
  - Akzeptanz: `tools/run_multi.sh 2` öffnet zwei Fenster, beide sehen beide Figuren sich bewegen; das Log zeigt Verbindungsaufbau mit Peer-IDs.
- [ ] **M0.7 Gehaltene Kiste im Netzwerk**
  - Prompt: „Mache das Aufheben netzwerkfähig: Aufheben/Ablegen/Werfen als RPC-Anfrage an den Host, die Kistenphysik läuft nur auf dem Host, Clients interpolieren.“
  - Akzeptanz: Host und Client sehen die Kiste an derselben Stelle; der Client kann sie aufheben und werfen.
- [ ] **M0.8 CLAUDE.md vervollständigen**
  - Prompt: „Trage Godot-Version, MCP-Server und alle Befehle in CLAUDE.md ein.“
  - Akzeptanz: Keine TODOs mehr in CLAUDE.md.

### UAT Phase 0

**Automatische Vorprüfung (Claude):** alle Tests grün · Projekt startet ohne Fehler/Warnungen · `run_multi.sh 2` verbindet sich · alle Aufgaben abgehakt.

| ID | Muss | Testfall | Erwartet | Ergebnis |
|---|---|---|---|---|
| U0.1 | ✔ | Projekt im Godot-Editor öffnen und mit F5 starten | Testraum erscheint, keine Fehler im Ausgabefenster | |
| U0.2 | ✔ | Herumlaufen, springen, umschauen | Steuerung reagiert direkt und ohne Ruckeln | |
| U0.3 | ✔ | Kiste aufheben, ablegen, werfen | Funktioniert zuverlässig | |
| U0.4 | ✔ | `tools/run_multi.sh 2`, in beiden Fenstern bewegen | Jede Figur ist im jeweils anderen Fenster sichtbar und bewegt sich flüssig | |
| U0.5 | ✔ | Im Client-Fenster die Kiste werfen | Kiste fliegt in beiden Fenstern gleich | |
| U0.6 | ✔ | Claude bitten, das Projekt über den MCP-Server zu starten und das Log zusammenzufassen | Claude kann beides | |
| U0.7 | ✔ | **Verständnis:** Erkläre dir selbst (oder Claude) Node, Szene, Signal, Autoload und den Unterschied Host/Client | Du kannst es in eigenen Worten | |
| U0.8 | | F3 drücken | Debug-Overlay mit FPS erscheint | |

---

## Phase 1: Kern-Prototyp „Ist der Boxenstopp lustig?“

**Ziel:** Ein kompletter Boxenstopp ist allein spielbar und fühlt sich chaotisch, aber beherrschbar an. Nur Greybox-Grafik.

### Aufgaben

- [ ] **M1.1 Wabbel-Hände**
  - Prompt: „Ersetze das feste Halten durch eine Feder/PD-Regelung: gehaltene Objekte werden zu einem Zielpunkt vor der Kamera gezogen, mit Verzögerung und leichtem Noise-Wackeln. Alle Parameter (Federstärke, Dämpfung, Noise-Amplitude/-Frequenz, Max-Gewicht) in `player/hands_tuning.tres`.“
  - Akzeptanz: Objekte folgen sichtbar verzögert und wackeln; Parameter wirken live im Inspector.
- [ ] **M1.2 Box-Greybox**
  - Prompt: „Baue `pit/pit_box.tscn` als Greybox: Stellplatz mit Markierung, Reifenregal mit 4 neuen Reifen, Tanksäule mit Schlauch-Anschluss, Wagenheber-Position, Lollipop-Halter, Funkknopf-Platzhalter. Interaktive Teile bekommen eine Outline beim Anschauen.“
  - Akzeptanz: Szene lädt; alle Stationen sind erreichbar und markiert.
- [ ] **M1.3 Boxenstopp-Logik (ohne Szene)**
  - Prompt: „Implementiere `core/pit_stop_rules.gd` als reine Logik-Klasse: Zustände je Rad (Mutter fest/lose/ab, Rad alt/neu/fehlt, Mischung), Tankfüllstand, Wagenheber oben/unten, Schlauch dran/ab, Freigabe. Zentrale Methode `request_action(player_id, action) -> Result`. Fehlerzustände: lose Mutter bei Freigabe, Schlauch dran bei Freigabe, falsche Mischung. Mit vollständigen Unit-Tests.“
  - Akzeptanz: Tests decken korrekten Ablauf und alle Fehlerfälle ab; alle grün.
- [ ] **M1.4 Dummy-Auto**
  - Prompt: „Ein Greybox-Auto fährt auf Tastendruck (Debug) in die Box, hält am Stellplatz, wartet auf Freigabe und fährt raus. Standzeit wird gemessen und groß angezeigt.“
  - Akzeptanz: Auto fährt rein/raus; Standzeit stimmt mit Stoppuhr überein (±0,1 s).
- [ ] **M1.5 Wagenheber und Reifenwechsel**
  - Prompt: „Verbinde Szene und Logik: Wagenheber bedienen (Taste halten), Schlagschrauber greifen und an eine Mutter halten (Taste halten, dauert X s laut Tuning, Abrutschen bricht ab), Rad abziehen, neues Rad an die Nabe führen (Einrasten in Toleranzradius).“
  - Akzeptanz: Ein Rad lässt sich komplett wechseln; Logik-Zustand passt zur Szene (Debug-Overlay zeigt ihn).
- [ ] **M1.6 Tanken mit Schlauch**
  - Prompt: „Schlauch als Segmentkette oder Verlet-Seil, nur das Ende ist greifbar. Am Tankstutzen halten füllt den Tank; außerhalb der Toleranz wird verschüttet (Partikel-Platzhalter) und es läuft langsamer. Schlauch-Zappeln per Noise. Werte in `pit/fuel_tuning.tres`.“
  - Akzeptanz: Tank lässt sich füllen; Abrutschen ist möglich und sichtbar.
- [ ] **M1.7 Freigabe und Fehlerfolgen**
  - Prompt: „Lollipop-Freigabe; Fehlerzustände sichtbar machen: lose Mutter → Rad fällt beim Rausfahren ab, Schlauch dran → Schlauch reißt + Strafe-Anzeige, falsche Mischung → Hinweis im Ergebnis.“
  - Akzeptanz: Jeder Fehlerfall ist reproduzierbar und sichtbar.
- [ ] **M1.8 Debug-Szenario und Reset**
  - Prompt: „Startargument `--scenario pitstop` startet direkt mit Auto in der Box; Taste R setzt den Stopp zurück. Ergebnis-Panel nach dem Stopp: Standzeit, Fehler, Bestzeit der Sitzung.“
  - Akzeptanz: Szenario startet in < 3 s; Reset funktioniert beliebig oft.

### UAT Phase 1

**Automatische Vorprüfung (Claude):** alle Tests grün · `--scenario pitstop` startet fehlerfrei · 10× automatischer Reset ohne Fehler im Log · keine Magic Numbers für Tuning-Werte im Code.

| ID | Muss | Testfall | Erwartet | Ergebnis |
|---|---|---|---|---|
| U1.1 | ✔ | Kompletten Boxenstopp fehlerfrei durchspielen | Möglich, ohne Bugs oder Hänger | |
| U1.2 | ✔ | 5 Stopps hintereinander, Zeiten notieren | Zeiten werden besser (Lernkurve spürbar) | |
| U1.3 | ✔ | Schlauch halten | Ungenau und wackelig, aber mit Übung kontrollierbar | |
| U1.4 | ✔ | Schlagschrauber an Mutter halten | Erfordert Konzentration, ist nicht frustrierend | |
| U1.5 | ✔ | Absichtlich eine Mutter lose lassen und freigeben | Rad fällt ab, gut sichtbar und witzig | |
| U1.6 | ✔ | Absichtlich mit Schlauch dran freigeben | Strafe wird angezeigt | |
| U1.7 | ✔ | Tuning-Werte im Inspector ändern | Wirkung sofort spürbar | |
| U1.8 | ✔ | **Spaßtest:** 2–3 Freunde je 3 Stopps spielen lassen | Mehrheit lacht oder will nochmal. Notizen ins Protokoll | |
| U1.9 | | Standzeit-Anzeige gegen Stoppuhr | Abweichung < 0,1 s | |

> Wenn U1.8 nicht besteht, ist das kein Versagen, sondern der wichtigste Erkenntnisgewinn des ganzen Projekts. Dann mit Claude Varianten der Steuerung durchspielen, bevor es weitergeht.

---

## Phase 2: Rennsimulation und Funk

**Ziel:** Ein komplettes Rennen (10–15 Min.) allein spielbar, mit eigenen Stopps und Strategieentscheidungen.

### Aufgaben

- [ ] **M2.1 Oval und Boxengasse**
  - Prompt: „Baue `race/oval.tscn`: Oval als `Path3D` mit 3 Spuren (Lane-Offset), Boxengasse als zweiter Pfad mit Ein- und Ausfahrt und Tempolimit-Zone. Greybox-Geometrie, Kamera-Debugansicht von oben (Taste F4).“
  - Akzeptanz: Pfade sind geschlossen bzw. korrekt verbunden; Debugansicht zeigt alles.
- [ ] **M2.2 Rennsimulation (ohne Szene)**
  - Prompt: „Implementiere `race/race_sim.gd` als reine Logik: Autos mit Fortschritt, Geschwindigkeit, Reifenverschleiß, Sprit, Gewicht; Überholen per Spurwechsel; Boxengasse; Rundenzählung; Rangliste. Werte in `race/race_tuning.tres`. Unit-Tests inklusive ‚ganzes Rennen in Sekunden headless simulieren‘.“
  - Akzeptanz: Headless-Simulation eines kompletten Rennens < 5 s; plausible Ergebnisse; Tests grün.
- [ ] **M2.3 Autos visualisieren**
  - Prompt: „4 Greybox-Autos folgen der Simulation auf dem Oval, mit Interpolation. Das eigene Auto in Crew-Farbe.“
  - Akzeptanz: Bewegung flüssig, keine Sprünge.
- [ ] **M2.4 Platzhalter-Crews**
  - Prompt: „Die 3 fremden Autos haben Platzhalter-Crews: Sie fahren bei Verschleiß-Schwelle in die Box, Standzeit zufällig im konfigurierbaren Bereich.“
  - Akzeptanz: Fremde Autos stoppen plausibel oft.
- [ ] **M2.5 Box an die Simulation anbinden**
  - Prompt: „Das eigene Auto fährt aus der Simulation in die Box aus Phase 1 und nach der Freigabe zurück. Reifen und Sprit aus dem Stopp übernehmen. Fehlerfolgen (Rad ab, leerer Tank) wirken auf der Strecke.“
  - Akzeptanz: Stopp verändert Verschleiß und Sprit korrekt; Fehler wirken sich auf das Tempo aus.
- [ ] **M2.6 Funk und Telemetrie**
  - Prompt: „Funkknopf in der Box: ‚Box diese Runde‘ (bestätigt per Funk-Text). Telemetrie-Bildschirm (`SubViewport`) in der Box: Reifen je Rad, Sprit, Position, Abstand vorne/hinten, Runde X/Y.“
  - Akzeptanz: Ruf vor der Einfahrtslinie → Auto kommt in dieser Runde; danach → nächste Runde.
- [ ] **M2.7 Blick auf die Strecke**
  - Prompt: „Die Box hat eine offene Seite zur Boxenmauer, von der man einen Teil des Ovals sieht. Passe Strecke/Box so an, dass das eigene Auto beim Vorbeifahren sichtbar ist.“
  - Akzeptanz: Vom Stellplatz aus ist die Strecke sichtbar.
- [ ] **M2.8 Rennende und Ergebnis**
  - Prompt: „Countdown-Start, Rennende nach Rundenzahl (Debug-Option zum Vorspulen ×4), Ergebnisbildschirm mit Rangliste, Stopp-Anzahl, bester Standzeit, Fehlern.“
  - Akzeptanz: Rennen endet korrekt; Vorspulen funktioniert.

### UAT Phase 2

**Automatische Vorprüfung (Claude):** alle Tests grün · 20 headless simulierte Rennen ohne Fehler, Auswertung der durchschnittlichen Stopps pro Auto (Ziel 5–7) · Spiel startet und läuft 2 Min. ohne Fehler im Log.

| ID | Muss | Testfall | Erwartet | Ergebnis |
|---|---|---|---|---|
| U2.1 | ✔ | Ein komplettes Rennen ohne Vorspulen spielen | 10–15 Min., keine Bugs, Ergebnis erscheint | |
| U2.2 | ✔ | Den Fahrer per Funk reinrufen | Kommt in der erwarteten Runde | |
| U2.3 | ✔ | Telemetrie während des Rennens beobachten | Verständlich; man weiß, wann man stoppen sollte | |
| U2.4 | ✔ | Einen sehr schnellen vs. einen langsamen Stopp | Der Unterschied ist in der Position spürbar | |
| U2.5 | ✔ | Zu spät stoppen (Sprit leer) | Auto rollt aus, Folge ist klar erkennbar | |
| U2.6 | ✔ | Rad-ab-Fehler provozieren | Auto humpelt zurück in die Box | |
| U2.7 | ✔ | **Strategietest:** Gibt es eine echte Entscheidung, wann man stoppt? | Ja, nicht immer „so spät wie möglich“ | |
| U2.8 | | Fremde Autos beobachten | Verhalten wirkt plausibel | |

---

## Phase 3: Multiplayer-Fundament

**Ziel:** Eine Crew mit 2–4 Spielern macht über das Netzwerk zusammen Boxenstopps; auch mit 100 ms Latenz spielbar.

### Aufgaben

- [ ] **M3.1 Netzwerk-Architektur planen**
  - Prompt: „Plan-Modus: Analysiere den aktuellen Code und erstelle einen Plan, wie Box, Teile, Schlauch, Rennsimulation und Telemetrie host-autoritativ werden (welche Nodes, welche Synchronizer, welche RPCs, welche Sync-Raten). Schreibe den Plan nach `docs/network_plan.md`.“
  - Akzeptanz: Plan liegt vor und ist von dir gelesen und freigegeben.
- [ ] **M3.2 Lobby**
  - Prompt: „Einfache Lobby-Szene: Name eingeben, Hosten oder per IP beitreten, Spielerliste mit Bereit-Status, Host startet. `--autostart` überspringt die Lobby weiterhin.“
  - Akzeptanz: 4 Instanzen können sich über die Lobby verbinden und gemeinsam starten.
- [ ] **M3.3 Teile host-autoritativ**
  - Prompt: „Setze den Plan für Reifen, Muttern, Schlagschrauber, Wagenheber und Lollipop um: alle Interaktionen als RPC-Anfrage, Host prüft per `pit_stop_rules`, Zustand wird synchronisiert, gehaltene Objekte host-simuliert und interpoliert.“
  - Akzeptanz: Zwei Spieler können parallel an verschiedenen Rädern arbeiten; Zustand ist auf allen Instanzen gleich.
- [ ] **M3.4 Schlauch im Netzwerk**
  - Prompt: „Schlauch im Netzwerk: Simulation nur auf dem Host, Clients sehen eine synchronisierte bzw. lokal nachsimulierte Darstellung; Füllstand host-autoritativ.“
  - Akzeptanz: Client kann tanken; Schlauch sieht auf allen Instanzen ähnlich aus.
- [ ] **M3.5 Rennen und Telemetrie im Netzwerk**
  - Prompt: „Rennsimulation läuft nur auf dem Host, Autos und Telemetrie werden mit niedriger Rate synchronisiert und interpoliert. Funkruf als RPC.“
  - Akzeptanz: Alle Instanzen zeigen dieselbe Rangliste und Position.
- [ ] **M3.6 Bot-Input für Tests**
  - Prompt: „Test-Modus `--test-bot <skript>`: die Spielfigur führt eine Skriptsequenz aus (gehe zu, greife, halte, lasse los). Damit ein automatischer Test: Host + 1 Client headless, Bots führen einen Boxenstopp aus, alle Logs werden auf Desync geprüft (Prüfsumme des Box-Zustands).“
  - Akzeptanz: `tools/run_net_test.sh` läuft grün durch.
- [ ] **M3.7 Latenz und Verbindungsabbruch**
  - Prompt: „Debug-Option für künstliche Latenz/Paketverlust (oder Anleitung für clumsy/netem in `docs/`). Verbindungsabbruch eines Clients und des Hosts sauber behandeln (Meldung, zurück zur Lobby). Ping im Debug-Overlay.“
  - Akzeptanz: Abbrüche führen nicht zu Absturz oder Hänger.

### UAT Phase 3

**Automatische Vorprüfung (Claude):** alle Tests grün · `run_net_test.sh` 5× hintereinander grün · `run_multi.sh 4` läuft 5 Min. ohne Fehler im Log.

| ID | Muss | Testfall | Erwartet | Ergebnis |
|---|---|---|---|---|
| U3.1 | ✔ | Mit einer zweiten Person (zweiter Rechner, LAN) ein Rennen als Crew spielen | Komplett spielbar | |
| U3.2 | ✔ | Gleichzeitig an zwei Rädern arbeiten | Kein Durcheinander, Zustände stimmen | |
| U3.3 | ✔ | Ein Spieler hält den Schlauch, der andere wechselt Reifen | Beides funktioniert parallel | |
| U3.4 | ✔ | Als Client spielen (nicht Host) | Greifen und Halten fühlt sich fast genauso an wie beim Host | |
| U3.5 | ✔ | Mit 100 ms künstlicher Latenz spielen | Noch gut spielbar | |
| U3.6 | ✔ | Client mitten im Rennen schließen | Host läuft weiter, keine Fehler | |
| U3.7 | ✔ | Host mitten im Rennen schließen | Client bekommt Meldung und landet in der Lobby | |
| U3.8 | | 4 Instanzen in einer Crew | Funktioniert, Box nicht zu eng | |

---

## Phase 4: Mehrere Crews und Match-Flow

**Ziel:** 3–4 Crews fahren ein komplettes Rennen gegeneinander (Vertical Slice ohne Sabotage).

### Aufgaben

- [ ] **M4.1 Boxengasse mit 4 Garagen**
  - Prompt: „Boxengasse mit 4 nebeneinanderliegenden Box-Instanzen, jede mit eigener Crew-Farbe, eigenem Stellplatz und Telemetrie. Die Boxen sind zu Fuß erreichbar (wichtig für Phase 5).“
  - Akzeptanz: 4 Boxen funktionieren unabhängig voneinander.
- [ ] **M4.2 Crew-Wahl in der Lobby**
  - Prompt: „Lobby: Crew wählen (Farbe), 1–4 Spieler pro Crew, mindestens 2 Crews mit Spielern zum Starten. Host stellt Rundenzahl ein.“
  - Akzeptanz: Zuordnung wird korrekt ins Spiel übernommen.
- [ ] **M4.3 Ein Auto pro Crew**
  - Prompt: „Platzhalter-Crews entfernen (bzw. nur noch für leere Crews im Debug). Jedes Auto gehört einer Spieler-Crew, Funk und Telemetrie nur für die eigene Crew.“
  - Akzeptanz: Jede Crew steuert nur ihr Auto.
- [ ] **M4.4 Match-Flow**
  - Prompt: „Ablauf als Zustandsautomat: Lobby → Vorbereitung (30 s) → Rennen → Ergebnis → Lobby. Ergebnis mit Crew-Rangliste und Awards (schnellster Stopp, meiste Fehler).“
  - Akzeptanz: Mehrere Matches hintereinander ohne Neustart möglich.
- [ ] **M4.5 Balancing kleiner Crews**
  - Prompt: „Implementiere konfigurierbare Ausgleichs-Hebel für Crews mit 1–3 Spielern (z. B. schnellere Werkzeugzeit, vorbereitete Teile), alle in `core/crew_balance.tres`, in der Lobby ein-/ausschaltbar.“
  - Akzeptanz: Hebel wirken messbar; per Lobby abschaltbar.
- [ ] **M4.6 Last- und Performancetest**
  - Prompt: „Lasttest mit 16 Instanzen (headless wo möglich, Bot-Inputs): Messung von Bandbreite pro Client, Host-CPU, Frame-Zeit. Ergebnis nach `docs/perf_m4.md`. Optimiere, falls Ziele (Host ≥ 60 FPS, < 50 KB/s pro Client) verfehlt werden.“
  - Akzeptanz: Ziele erreicht oder Abweichung dokumentiert und mit dir abgestimmt.

### UAT Phase 4

**Automatische Vorprüfung (Claude):** alle Tests grün · Lasttest erfüllt die Ziele · 3 automatische Matches hintereinander ohne Fehler.

| ID | Muss | Testfall | Erwartet | Ergebnis |
|---|---|---|---|---|
| U4.1 | ✔ | Spieleabend: mindestens 3 Crews mit echten Spielern, ein komplettes Match | Läuft von Lobby bis Ergebnis durch | |
| U4.2 | ✔ | Direkt ein zweites Match starten | Funktioniert ohne Neustart | |
| U4.3 | ✔ | Crew mit 1 Spieler gegen Crew mit 4 | Kleine Crew hat mit Ausgleich eine faire Chance | |
| U4.4 | ✔ | Spieler fragen: Wollt ihr nochmal? | Mehrheit ja | |
| U4.5 | ✔ | Host-PC beobachten | Kein spürbares Ruckeln | |
| U4.6 | | Awards im Ergebnis | Sorgen für Gesprächsstoff | |

---

## Phase 5: Sabotage und Verteidigung

**Ziel:** Vollständiges Spielkonzept spielbar; Sabotage sorgt für Spannung statt Frust.

### Aufgaben

- [ ] **M5.1 Sabotage-Logik (ohne Szene)**
  - Prompt: „Erweitere die Logik um manipulierbare Teile: Reifen angestochen, Kanister/Tank mit Wasser, Mutter gelockert/vertauscht, falsche Mischung im Regal. Effekte wirken in `race_sim` erst auf der Strecke. Unit-Tests für jeden Effekt.“
  - Akzeptanz: Jeder Effekt ist getestet und messbar.
- [ ] **M5.2 Sabotage-Interaktion**
  - Prompt: „In einer fremden Box kann man Teile manipulieren: Aktion dauert X s (Tuning), mit eindeutiger Animation/Geräusch-Platzhalter. Manipulierte Teile sehen normal aus. Abklingzeit pro Spieler.“
  - Akzeptanz: Sabotage funktioniert im Netzwerk; Teile sind optisch nicht unterscheidbar.
- [ ] **M5.3 Rauswurf**
  - Prompt: „Schubsen: Spieler können andere Spieler schubsen (Impuls per Host-RPC an den Besitzer). Wird ein fremder Spieler in der eigenen Box geschubst, wird er an seine Box zurückgesetzt und seine Crew erhält eine Strafe (Durchfahrt).“
  - Akzeptanz: Rauswurf und Strafe funktionieren und sind für alle sichtbar.
- [ ] **M5.4 Inspektionsstation**
  - Prompt: „Inspektionsstation in jeder Box: Teil ablegen, nach X s zeigt sie ‚OK‘ oder ‚manipuliert‘.“
  - Akzeptanz: Erkennt alle Manipulationsarten korrekt.
- [ ] **M5.5 Einbruchsspuren**
  - Prompt: „Hinweise auf Einbruch: verschobene Teile, Fußspuren in Crew-Farbe (verblassen), Geräusch beim Manipulieren.“
  - Akzeptanz: Spuren sind sichtbar, aber nicht aufdringlich.
- [ ] **M5.6 Balance-Schalter**
  - Prompt: „Lobby-Option Sabotage-Intensität (aus / wenig / normal / chaos), steuert Abklingzeiten, Dauer und Effektstärke über `sabotage/sabotage_tuning.tres`. Statistik im Ergebnis: Sabotagen versucht/erwischt/erfolgreich.“
  - Akzeptanz: Stufen wirken spürbar unterschiedlich.

### UAT Phase 5

**Automatische Vorprüfung (Claude):** alle Tests grün · Netzwerktest mit Sabotage-Bot-Skript grün · 3 Matches mit Sabotage ohne Fehler.

| ID | Muss | Testfall | Erwartet | Ergebnis |
|---|---|---|---|---|
| U5.1 | ✔ | Großer Playtest (8–16 Leute), 2–3 Matches mit Sabotage „normal“ | Läuft stabil durch | |
| U5.2 | ✔ | Eine fremde Box erfolgreich sabotieren | Machbar, aber riskant | |
| U5.3 | ✔ | Einen Saboteur erwischen und rauswerfen | Fühlt sich befriedigend an | |
| U5.4 | ✔ | Manipuliertes Teil per Inspektion entdecken | Funktioniert; Abwägung Zeit vs. Sicherheit ist spürbar | |
| U5.5 | ✔ | Sabotage-Effekt auf der Strecke | Für die betroffene Crew nachvollziehbar, was passiert ist | |
| U5.6 | ✔ | Spieler-Feedback: Ist Sabotage frustrierend? | Mehrheit: spannend statt frustrierend | |
| U5.7 | ✔ | Crew, die nur sabotiert, gegen Crew, die nur arbeitet | Keine Strategie dominiert klar | |
| U5.8 | | Statistik im Ergebnis | Sorgt für Gesprächsstoff | |

---

## Phase 6: Online-Spiel über das Internet

**Ziel:** Freunde an verschiedenen Orten spielen ohne Portfreigabe zusammen.

### Aufgaben

- [ ] **M6.1 Entscheidung Transport**
  - Prompt: „Plan-Modus: Vergleiche GodotSteam (Steam-Lobbys) und noray (Relay/NAT-Punch) für dieses Projekt (Aufwand, Kosten, Abhängigkeiten, Godot-Version). Empfehlung nach `docs/online_decision.md`.“
  - Akzeptanz: Du hast entschieden; Entscheidung ist dokumentiert.
- [ ] **M6.2 Transport integrieren**
  - Prompt: „Integriere die gewählte Lösung als alternativen Peer im `Network`-Autoload; ENet/LAN bleibt als Option erhalten.“
  - Akzeptanz: Beide Wege funktionieren; Tests grün.
- [ ] **M6.3 Beitreten per Code oder Einladung**
  - Prompt: „Lobby: Spiel per Join-Code bzw. Steam-Einladung beitreten; Fehlermeldungen bei ungültigem Code, voller Lobby, Versionskonflikt (Spielversion wird beim Beitritt geprüft).“
  - Akzeptanz: Alle Fehlerfälle zeigen verständliche Meldungen.
- [ ] **M6.4 Bandbreite und Robustheit**
  - Prompt: „Bandbreite unter realen Bedingungen messen und optimieren (nur Änderungen senden, Raten anpassen). Optional: Client-Reconnect während eines laufenden Matches.“
  - Akzeptanz: Ziele aus M4.6 auch über das Internet.
- [ ] **M6.5 Export-Builds**
  - Prompt: „Export-Presets für Windows und Linux, Skript `tools/build.sh`, Versionsnummer im Hauptmenü.“
  - Akzeptanz: Builds starten auf einem frischen Rechner.

### UAT Phase 6

**Automatische Vorprüfung (Claude):** alle Tests grün · Builds werden erzeugt · Versionsprüfung getestet.

| ID | Muss | Testfall | Erwartet | Ergebnis |
|---|---|---|---|---|
| U6.1 | ✔ | Freunde an anderen Orten treten mit dem Build bei, ohne Portfreigabe | Funktioniert | |
| U6.2 | ✔ | Komplettes Match über das Internet mit mindestens 8 Spielern | Gut spielbar | |
| U6.3 | ✔ | Ungültigen Code eingeben, alte Version beitreten lassen | Verständliche Fehlermeldung | |
| U6.4 | ✔ | Ein Spieler mit schlechter Verbindung (WLAN, weit weg) | Spielbar, stört andere nicht | |
| U6.5 | | Reconnect nach Abbruch | Klappt (falls umgesetzt) | |

---

## Phase 7: Grafik, Sound, Juice

**Ziel:** Das Spiel sieht aus und klingt wie ein Spiel, im Cartoon-Stil.

### Aufgaben

- [ ] **M7.1 Art-Direction festlegen** (**Du**, mit Claude als Sparringspartner)
  - Prompt: „Hilf mir, eine Stil-Referenz zu erstellen: Farbpalette pro Crew, Shader-Look, Proportionen. Ergebnis nach `docs/art_direction.md`.“
  - Akzeptanz: Du bist mit der Richtung zufrieden.
- [ ] **M7.2 Toon-Shader und Outlines**
  - Prompt: „Cel-Shading-Shader mit 2–3 Lichtstufen und Outline-Pass; auf alle Greybox-Materialien anwendbar; Parameter als Resource.“
  - Akzeptanz: Look passt zur Art-Direction; Performance-Ziele halten.
- [ ] **M7.3 Assets einbinden**
  - Prompt: „Ersetze Greybox-Objekte durch Assets aus `art/` (Kenney o. ä., von mir bereitgestellt), Kollisionen anpassen.“
  - Akzeptanz: Alle Kernobjekte haben finale bzw. gute Platzhalter-Modelle.
- [ ] **M7.4 Figuren und Hände**
  - Prompt: „First-Person-Hände, die gehaltene Objekte sichtbar greifen; Ganzkörper-Figur für andere Spieler mit einfachen Animationen (laufen, greifen, schubsen, fallen).“
  - Akzeptanz: Andere Spieler sind gut lesbar.
- [ ] **M7.5 Effekte (Juice)**
  - Prompt: „Funken am Schlagschrauber, Sprit-Spritzer, wegfliegende Räder mit Übertreibung, Rauch, Kamerawackeln bei Ereignissen (abschaltbar).“
  - Akzeptanz: Ereignisse sind deutlich und witzig.
- [ ] **M7.6 Sound**
  - Prompt: „Audio-Bus-Struktur, Motoren mit Doppler, Werkzeug-Sounds, Funksprüche (Text + Platzhalter-Sound), Jubel; Lautstärken in Settings.“
  - Akzeptanz: Wichtige Ereignisse sind hörbar unterscheidbar.
- [ ] **M7.7 UI-Überarbeitung**
  - Prompt: „Lobby, HUD, Telemetrie, Ergebnis im Cartoon-Stil, einheitliches Theme (`ui/theme.tres`).“
  - Akzeptanz: Einheitlicher Look.

### UAT Phase 7

**Automatische Vorprüfung (Claude):** alle Tests grün · Performance-Ziele aus M4.6 weiterhin erfüllt · keine fehlenden Ressourcen im Log.

| ID | Muss | Testfall | Erwartet | Ergebnis |
|---|---|---|---|---|
| U7.1 | ✔ | Screenshot an jemanden zeigen, der das Projekt nicht kennt | „Sieht aus wie ein Spiel“ | |
| U7.2 | ✔ | Match spielen | Alle Teile, Stationen und Spieler sofort erkennbar | |
| U7.3 | ✔ | Mit geschlossenen Augen zuhören | Wichtige Ereignisse am Sound erkennbar | |
| U7.4 | ✔ | Auf schwächerem PC spielen | Noch flüssig | |
| U7.5 | | Juice-Effekte | Machen Spaß, stören nicht | |

---

## Phase 8: Feinschliff und Release

**Ziel:** Öffentliche Version 1.0 auf itch.io.

### Aufgaben

- [ ] **M8.1 Tutorial / Übungsbox**
  - Prompt: „Einzelspieler-Übungsbox mit kurzen Hinweisen, die Schritt für Schritt einen Boxenstopp erklärt.“
  - Akzeptanz: Neue Spieler schaffen den ersten Stopp ohne Erklärung von außen.
- [ ] **M8.2 Einstellungen**
  - Prompt: „Optionsmenü: Mausempfindlichkeit, Invertieren, FOV, Lautstärken, Grafikqualität, Tastenbelegung; gespeichert in `user://`.“
  - Akzeptanz: Einstellungen bleiben nach Neustart erhalten.
- [ ] **M8.3 Balancing-Auswertung**
  - Prompt: „Werte die Match-Statistiken der Playtests aus (Logs/Export): Standzeiten, Sabotage-Quoten, Gewinnrate nach Crewgröße. Vorschläge für Tuning-Änderungen nach `docs/balancing.md`.“
  - Akzeptanz: Du hast die Änderungen freigegeben, sie sind umgesetzt.
- [ ] **M8.4 Bugfixing und Stabilität**
  - Prompt: „Arbeite den Backlog ab (alle Minor-Punkte aus den UATs), priorisiert nach Auswirkung.“
  - Akzeptanz: Backlog leer oder bewusst verschoben.
- [ ] **M8.5 Release vorbereiten**
  - Prompt: „itch.io-Seite vorbereiten: Beschreibung, Screenshots-Liste, Systemanforderungen, Changelog, README für Spieler.“
  - Akzeptanz: Alles liegt in `docs/release/`.

### UAT Phase 8 (Release-Gate)

**Automatische Vorprüfung (Claude):** alle Tests grün · alle früheren automatischen Vorprüfungen erneut grün (Regression) · Builds erzeugt.

| ID | Muss | Testfall | Erwartet | Ergebnis |
|---|---|---|---|---|
| U8.1 | ✔ | Neue Spieler ohne Erklärung spielen lassen | Kommen über das Tutorial ins Spiel | |
| U8.2 | ✔ | Kompletter Ablauf: Download → Start → Online-Match → Beenden | Ohne Probleme | |
| U8.3 | ✔ | Alle Muss-Testfälle der Phasen 1–7 kurz wiederholen (Regression) | Weiterhin bestanden | |
| U8.4 | ✔ | Kein offener Blocker oder Major | Erfüllt | |
| U8.5 | | Einstellungen nach Neustart | Bleiben erhalten | |

---

## UAT-Protokoll

Für jeden UAT-Durchlauf einen Eintrag anlegen (neuester oben).

```markdown
### UAT Phase <N>, Durchlauf <k>, <Datum>
- Commit: <hash>
- Tester: <Namen>
- Automatische Vorprüfung: ✅ / ❌ (Details)
- Testfälle: U<N>.1 ✅, U<N>.2 ❌, ...
- Befunde:
  - [Blocker|Major|Minor] <Beschreibung> → <Aufgabe M<N>.F1 | Backlog>
- Entscheidung: Gate bestanden / nicht bestanden
- Notizen (Spielgefühl, Zitate von Testern):
```

<!-- Einträge hier einfügen -->

---

## Backlog (Minor-Befunde und Ideen)

| Herkunft | Beschreibung | Ziel-Phase |
|---|---|---|
| – | – | – |
