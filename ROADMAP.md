# Roadmap: Boxenstopp-Crew-Spiel (Arbeitstitel „Pit Panic“)

Kooperatives Online-Multiplayer-Spiel in Godot 4. Crews aus je 1–4 Spielern arbeiten als Boxencrew eines Rennstalls. Ein KI-Fahrer fährt auf einem Oval. Die Crew ruft ihn per Funk an die Box, wechselt Reifen, tankt nach und schickt ihn wieder raus. Je schneller und sauberer der Stopp, desto besser die Platzierung. Gleichzeitig können sich die Crews gegenseitig sabotieren.

---

## 1. Festgelegte Eckpunkte (aus dem Interview)

| Thema | Entscheidung |
|---|---|
| Engine / Sprache | Godot 4.x (aktuelle stabile Version), GDScript |
| Erfahrung | Godot-Neuling, deshalb enthält die Roadmap Lernphasen |
| Team | Hobby, solo |
| Spieleranzahl | 3–4 Crews, 1–4 Spieler pro Crew (max. 16), **keine Bots** |
| Perspektive | First-Person |
| Steuerung / Gefühl | Physikbasiert und bewusst „wabbelig“: Schlauch, Schlagschrauber und Reifen lassen sich nicht ganz präzise führen (Richtung Gang Beasts / Totally Reliable Delivery Service, aber in First-Person) |
| Netzwerk | Online, Listen-Server (ein Spieler hostet) |
| Boxenstrategie | Die Crew ruft den Fahrer per Funk rein |
| Sabotage | In fremde Boxen schleichen; vorbereitete Teile manipulieren, die erst auf der Strecke wirken |
| Verteidigung | Saboteure erwischen und rauswerfen (Zeitstrafe für deren Team) **und** Teile vor dem Einbau inspizieren (kostet Zeit) |
| Matchlänge | 10–15 Minuten, etwa 5–7 Boxenstopps |
| Grafik | Cartoon / Comic (Toon-Shader, Outlines, übertriebene Animationen) |
| Strecke | Ein Oval |

---

## 2. Spielablauf eines Matches

1. **Lobby:** Der Host erstellt ein Spiel, die anderen treten bei und wählen eine Crew (Farbe). Der Host legt die Rundenanzahl fest.
2. **Vorbereitung (ca. 30 s):** Die Crews legen Reifen bereit, füllen Kanister und besetzen ihre Stationen.
3. **Rennen:** Die KI-Fahrer fahren. In der Box sieht die Crew auf einem Telemetrie-Bildschirm Reifenverschleiß, Sprit, Position und Abstände.
4. **Boxenruf:** Ein Crewmitglied drückt den Funkknopf. Der Fahrer kommt in der nächsten Runde rein.
5. **Boxenstopp:** Das Auto steht in der Box. Die Crew hebt es mit dem Wagenheber an, löst die Radmuttern, wechselt die Reifen, zieht die Muttern fest, tankt und gibt das Auto mit dem Lollipop wieder frei. Die Standzeit wird gemessen.
6. **Fehler haben Folgen:** Eine lose Mutter lässt das Rad auf der Strecke abfallen. Wer zu früh freigibt (Schlauch noch dran), bekommt eine Strafe. Wer den falschen Reifen montiert, verschleißt schneller.
7. **Sabotage:** Wer sich zwischendurch in eine fremde Box schleicht, kann dort Reifen anstechen, Wasser in den Kanister kippen oder Muttern vertauschen. Dafür fehlt diese Person in der eigenen Crew.
8. **Ende:** Sobald alle Runden gefahren sind, folgen Siegerehrung und Statistik (schnellster Stopp, meiste Sabotagen, peinlichster Fehler).

---

## 3. Technische Architektur (Überblick)

### 3.1 Netzwerkmodell

- **Godot High-Level Multiplayer** (`MultiplayerAPI`, `MultiplayerSpawner`, `MultiplayerSynchronizer`, RPCs).
- **Transport:** zuerst `ENetMultiplayerPeer` (LAN, direkte IP). Später für Internet-Spiele **GodotSteam** (Steam-Lobbys, NAT-Traversal, Freunde einladen). Zum Testen geht das kostenlos mit der Spacewar-App-ID 480. Alternative ohne Steam: ein Relay- oder NAT-Punch-Dienst wie *noray*.
- **Autorität:**
  - **Host-autoritativ:** Rennsimulation, Autos, Boxenstopp-Zustand, Teile (Reifen, Kanister, Muttern), Sabotage-Zustände, Strafen, Punkte.
  - **Client-autoritativ:** nur die *eigene Spielfigur*, also Bewegung und Blickrichtung. Das fühlt sich direkt an, und Schummeln ist unter Freunden in einem Hobbyprojekt kein großes Thema.
  - **Interaktionen** (greifen, schubsen, Schrauber ansetzen) laufen als RPC-Anfrage an den Host. Der Host prüft sie und führt sie aus.
- **Physik der gehaltenen Objekte** läuft auf dem Host. Clients interpolieren. Die gewollt wabbelige Steuerung hilft dabei: Eine Feder mit Verzögerung kaschiert Latenz deutlich besser als präzise Steuerung.
- **Tickrate:** Rennsimulation mit 10–20 Hz synchronisiert (Autos auf Pfaden lassen sich gut interpolieren), Spielfiguren und Objekte mit 20–30 Hz.

### 3.2 Rennsimulation (bewusst einfach)

- Kein echtes Fahrphysik-Modell. Die Autos folgen einem `Path3D` (Oval) mit Fortschritt `t` und Geschwindigkeit `v`.
- Geschwindigkeit = Basistempo × Reifenfaktor × Gewichtsfaktor (Sprit) × Sabotage-Effekte, plus etwas Zufall.
- Überholen: mehrere Spuren (Lane-Offset) und eine simple Regel, die bei geringem Abstand auf die freie Spur ausweicht.
- Boxengasse: eigener Pfad mit Tempolimit. Der Zeitverlust entsteht durch Gassenfahrt plus Standzeit.
- Rundenzeit auf dem Oval etwa 20–25 s, also etwa 30–40 Runden für 10–15 Minuten.

### 3.3 Wabbel-Steuerung

- First-Person-Controller (`CharacterBody3D`) mit leichter Trägheit.
- „Hände“ als physikalischer Anker: Gehaltene Objekte (`RigidBody3D`) werden per Feder oder PD-Regler zu einem Zielpunkt vor der Kamera gezogen, statt fest zu kleben. Daraus entsteht das Wackeln.
- Zusätzlich gezieltes Rauschen (Noise), zum Beispiel beim Schlauch, der unter Druck „zappelt“.
- Schlauch: Kette aus Segmenten (`PinJoint3D`) oder eine Verlet-Seil-Simulation, nur auf dem Host berechnet.
- Alle Wabbel-Parameter zentral in einer `Resource`, damit man sie beim Playtesten schnell justieren kann.

### 3.4 Projektstruktur (Vorschlag)

```
res://
  autoload/        # Game, Network, Settings (Singletons)
  core/            # Match-Flow, Regeln, Punkte
  race/            # Strecke, KI-Auto, Rennsimulation, Telemetrie
  pit/             # Box, Stationen, Teile (Reifen, Mutter, Kanister, Wagenheber, Lollipop)
  player/          # FP-Controller, Interaktion, Hände
  sabotage/        # Manipulationen, Inspektion, Rauswurf
  net/             # Lobby, Peer-Setup, Spawner
  ui/              # HUD, Telemetrie-Screen, Lobby, Ergebnis
  art/ audio/
```

---

## 4. Roadmap in Phasen

Die Zeitangaben sind grobe Schätzungen für ein Solo-Hobbyprojekt mit etwa 8–10 Stunden pro Woche. Jede Phase endet mit einem **Meilenstein**, also etwas Spielbarem oder klar Prüfbarem.

### Phase 0: Lernen und Fundament (3–4 Wochen)

Ziel: Godot und GDScript sicher genug beherrschen, um nicht ständig blockiert zu sein.

- [ ] Godot 4 installieren, Git-Repo einrichten (`.gitignore` für Godot, `.godot/`-Ordner ignorieren)
- [ ] Offizielles Tutorial „Your first 3D game“ durcharbeiten
- [ ] Grundlagen verstehen: Nodes und Scenes, Signale, `_process` und `_physics_process`, Resources, Autoloads
- [ ] Einen einfachen First-Person-Controller selbst bauen (laufen, springen, Maus-Look)
- [ ] Objekte aufheben und werfen (RigidBody plus Raycast)
- [ ] Mini-Multiplayer-Tutorial: zwei Fenster, ENet, Spieler spawnen und synchronisieren
- [ ] Debug-Werkzeuge kennenlernen: Remote-Szenenbaum, Profiler, „Run multiple instances“ im Editor

**Meilenstein M0:** Zwei lokale Instanzen, in denen zwei First-Person-Figuren herumlaufen und eine Kiste aufheben können.

### Phase 1: Kern-Prototyp „Ist der Boxenstopp lustig?“ (4–6 Wochen, Einzelspieler)

Ziel: Den Spielspaß des Kernmechanismus beweisen, bevor Zeit in Netzwerk oder Grafik fließt. Nur Greybox-Grafik (Würfel und Zylinder).

- [ ] Eine Box als Greybox: Stellplatz, Reifenregal, Tanksäule, Wagenheber, Lollipop
- [ ] Wabbel-Hände: Feder- und PD-gezogene gehaltene Objekte, Parameter-Resource
- [ ] Reifenwechsel: Mutter lösen (Schlagschrauber halten, Taste gedrückt halten), Rad abziehen, neues Rad aufsetzen, Mutter anziehen
- [ ] Tanken: Schlauch zum Tankstutzen führen und halten, solange der Füllstand steigt. Rutscht er ab, wird verschüttet und es geht langsamer
- [ ] Wagenheber und Lollipop-Freigabe
- [ ] Ein Dummy-Auto fährt ein, wartet und fährt raus. Die Standzeit wird angezeigt
- [ ] Fehlerzustände: lose Mutter, Schlauch noch dran, falscher Reifen
- [ ] **Ein bewusster Schritt:** Code von Anfang an so schreiben, dass alle Spielzustandsänderungen über *eine* Stelle laufen (zum Beispiel `PitStop.request_action(player, action)`). Das macht Phase 3 viel leichter

**Meilenstein M1:** Allein einen kompletten Boxenstopp durchspielen. Er soll sich chaotisch, aber beherrschbar anfühlen. Freunde ausprobieren lassen: Lachen sie?

### Phase 2: Rennsimulation und Funk (3–4 Wochen, Einzelspieler)

- [ ] Oval-Strecke als `Path3D`, Boxengasse als zweiter Pfad mit Abzweig
- [ ] KI-Auto: Geschwindigkeit, Reifenverschleiß, Spritverbrauch, Spurwechsel
- [ ] 4 Autos, davon 3 mit Platzhalter-Crews (feste Standzeit), damit es ein Rennen gibt
- [ ] Funkknopf in der Box: „Box diese Runde“
- [ ] Telemetrie-Bildschirm in der Box (`SubViewport`): Reifen, Sprit, Position, Abstand
- [ ] Rennende, Rangliste, Ergebnisbildschirm
- [ ] Folgen auf der Strecke: Rad fällt ab (Auto humpelt zur Box), leerer Tank (Auto rollt aus)
- [ ] Blick auf die Strecke: Boxenmauer oder Fenster, damit man das Rennen sieht

**Meilenstein M2:** Ein komplettes Rennen von 10–15 Minuten allein spielen, mit eigenen Boxenstopps und Strategieentscheidungen.

### Phase 3: Multiplayer-Fundament (6–8 Wochen, schwierigste Phase)

Ziel: Eine Crew mit 2–4 Spielern spielt gemeinsam über das Netzwerk (zunächst LAN oder Localhost).

- [ ] `Network`-Autoload: Host oder Join per IP, Peer-Verwaltung, Verbindungsabbrüche
- [ ] Einfache Lobby: Name, Bereit-Status, Host startet
- [ ] Spielfiguren spawnen (`MultiplayerSpawner`), eigene Bewegung client-autoritativ
- [ ] Interaktionen als RPCs an den Host (greifen, loslassen, benutzen)
- [ ] Gehaltene Objekte host-simuliert, für Clients interpoliert
- [ ] Rennsimulation läuft nur auf dem Host, Autos werden synchronisiert
- [ ] Boxenstopp-Zustand synchronisiert (Muttern, Füllstand, Wagenheber)
- [ ] Latenz-Tests: künstliche Verzögerung oder Paketverlust simulieren (zum Beispiel mit `clumsy` unter Windows oder `tc netem` unter Linux)

**Meilenstein M3:** 2–4 Spieler machen zusammen einen Boxenstopp, und es fühlt sich auch mit 100 ms Latenz noch spielbar an.

### Phase 4: Mehrere Crews und Match-Flow (3–4 Wochen)

- [ ] Boxengasse mit 3–4 nebeneinanderliegenden Garagen
- [ ] Crew-Zuweisung in der Lobby (Farben), 1–4 Spieler pro Crew
- [ ] Ein Auto pro Crew, Platzhalter-Crews entfallen
- [ ] Vollständiger Match-Flow: Lobby → Vorbereitung → Rennen → Ergebnis → zurück zur Lobby
- [ ] **Balancing für kleine Crews (ohne Bots):** zum Beispiel automatische Vorbereitung bei kleinen Crews, schnelleres Werkzeug oder Werkzeug, das mehrere Schritte kombiniert. Das muss in Playtests ermittelt werden
- [ ] Performance-Test mit 16 Spielern (mehrere Instanzen auf einem oder zwei Rechnern)

**Meilenstein M4 (Vertical Slice ohne Sabotage):** 3–4 Crews fahren ein komplettes Rennen gegeneinander.

### Phase 5: Sabotage und Verteidigung (4–6 Wochen)

**Sabotage (in fremde Box schleichen und Teile manipulieren):**
- [ ] Reifen anstechen: Er verliert auf der Strecke Luft, der Verschleiß steigt stark
- [ ] Wasser in den Kanister: Motorstottern, das Auto ist langsamer
- [ ] Muttern vertauschen oder lockern: Das Rad sitzt nicht fest und kann abfallen
- [ ] Falsche Reifenmischung ins Regal legen
- [ ] Manipulierte Teile sehen auf den ersten Blick normal aus (wichtig für die Spannung)

**Verteidigung:**
- [ ] Schubsen oder Fangen: Ein Saboteur in der eigenen Box wird rausgeworfen. Seine Crew bekommt eine Zeitstrafe (zum Beispiel ein Durchfahrtsgebot für ihren Fahrer)
- [ ] Inspektionsstation: Ein Teil dort ablegen zeigt nach X Sekunden, ob es manipuliert ist
- [ ] Hinweise auf Einbruch: Fußspuren, verschobene Teile, ein Geräusch

**Balance-Hebel:**
- [ ] Laufweg zur fremden Box (Risiko: eigene Crew unterbesetzt)
- [ ] Sabotage dauert einige Sekunden und ist an Animation und Geräusch erkennbar
- [ ] Abklingzeit oder Begrenzung pro Spieler und Rennen
- [ ] Schutzzone: Während des eigenen Boxenstopps ist die Box tabu (oder gerade nicht, das ist ein Playtest-Thema)

**Meilenstein M5:** Vollständiges Spielkonzept spielbar. Erster großer Playtest mit Freunden (8–16 Leute).

### Phase 6: Online-Spiel über das Internet (3–5 Wochen)

- [ ] Entscheidung treffen: GodotSteam (Steam-Lobbys, Einladungen, NAT-Traversal) oder Relay- bzw. Join-Code-Lösung (noray)
- [ ] Lobby-Browser oder Join-Code
- [ ] Host-Abbruch sauber behandeln (Match beenden mit Hinweis; Host-Migration ist für ein Hobbyprojekt zu aufwendig)
- [ ] Reconnect für Clients während eines laufenden Matches (optional)
- [ ] Bandbreite messen und optimieren (Sync-Raten, nur Änderungen senden)

**Meilenstein M6:** Freunde an verschiedenen Orten spielen ohne Portfreigabe zusammen.

### Phase 7: Grafik, Sound, Juice (6+ Wochen, auch parallel möglich)

- [ ] Cartoon-Look: Toon-Shader (Cel-Shading), Outlines, kräftige Farben pro Crew
- [ ] Assets: Kenney (CC0) als Basis, eigene einfache Modelle in Blender (Lernkurve einplanen)
- [ ] Spielfiguren: einfache Cartoon-Mechaniker mit sichtbaren Händen im FP-Bild, Ganzkörper für andere Spieler
- [ ] Übertriebenes Feedback: Funken am Schlagschrauber, Sprit-Spritzer, wegfliegende Räder, Rauch
- [ ] Sound: Motoren (Doppler), Schlagschrauber, Funksprüche des Fahrers, Jubel
- [ ] UI: Lobby, HUD, Telemetrie-Screen, Ergebnisbildschirm mit lustigen Awards

**Meilenstein M7:** Das Spiel sieht aus wie ein Spiel und nicht mehr wie ein Prototyp.

### Phase 8: Feinschliff und Release (4–6 Wochen)

- [ ] Tutorial oder Übungsbox (Einzelspieler)
- [ ] Optionen: Mausempfindlichkeit, Lautstärke, Grafik, Tastenbelegung
- [ ] Balancing anhand der Playtest-Daten (Standzeiten, Sabotage-Erfolgsquote, Gewinnrate nach Crewgröße)
- [ ] Bugfixing, Absturzsicherheit, Export für Windows und Linux
- [ ] Release auf itch.io (kostenlos oder „pay what you want“). Steam später optional (100 $ App-Gebühr)

**Meilenstein M8:** Öffentliche Version 1.0.

**Gesamtaufwand grob:** 9–14 Monate im Hobby-Tempo.

---

## 5. Größte Risiken und Gegenmaßnahmen

| Risiko | Gegenmaßnahme |
|---|---|
| Netzwerk-Physik (Schlauch, gehaltene Objekte) ruckelt oder desynchronisiert | Physik nur auf dem Host, Clients interpolieren. Die Wabbeligkeit ist Teil des Designs und kaschiert Latenz. Schlauch notfalls nur visuell simulieren und lediglich das „Ende“ synchronisieren |
| Listen-Server mit 16 Spielern überlastet den Host-PC | Früh messen (Phase 4). Sync-Raten senken. Rennsimulation ist billig (Pfade statt Physik) |
| First-Person mit 4 Leuten in einer engen Box ist unübersichtlich | Klare Stationen, farbige Outlines für interaktive Teile, kontextabhängige Hinweise |
| Kleine Crews (1–2 Spieler) sind chancenlos | Balancing-Hebel aus Phase 4; im Zweifel optionale Gleichstellung in der Lobby |
| Sabotage dominiert und frustriert | Abklingzeiten, Strafen, Inspektion, Playtests. Ein Lobby-Schalter für die Sabotage-Intensität |
| Scope Creep als Solo-Entwickler | Strikt nach Meilensteinen arbeiten. Neue Ideen in die Liste „Später“ schreiben und nicht sofort bauen |
| Motivation sinkt (Solo, lange Laufzeit) | Nach jeder Phase etwas Spielbares mit Freunden testen. Fortschritt öffentlich zeigen (Devlog) |

---

## 6. Bewusst NICHT in Version 1.0 (Ideenliste „Später“)

- Weitere Strecken (Rundkurse mit Kurven, Wetter)
- Bots als Mechaniker oder KI-Crews
- Verkleidung oder Tarnung für Saboteure
- Fahrzeug-Upgrades und Progression zwischen Matches
- Host-Migration, Dedicated Server
- Ranglisten und Matchmaking
- Konsole und Controller-Feinschliff

---

## 7. Offene Fragen (für spätere Gespräche)

1. **Sieht man die Saboteure?** Teamfarbe immer sichtbar (einfach), oder können sie sich unbemerkt nähern?
2. **Dürfen Saboteure während eines laufenden Stopps eingreifen** oder nur in der Vorbereitungszeit zwischen den Stopps?
3. **Gibt es Rollen** (Reifen-Mann, Tanker, Lollipop) mit Spezialfähigkeiten, oder kann jeder alles?
4. **Wie stark wirkt sich die Standzeit aus?** Reicht ein guter Stopp, um Plätze gutzumachen, oder ist die Strategie (Timing, Reifenwahl) wichtiger?
5. **Reifenmischungen** (weich, mittel, hart) als Strategie-Ebene, oder nur „neu oder alt“?
6. **Name des Spiels** 🙂

---

## 8. Nützliche Ressourcen

- Godot-Dokumentation: „High-level multiplayer“, „MultiplayerSynchronizer“, „Physics introduction“
- GodotSteam (Steam-Integration für Godot)
- noray (NAT-Punchthrough und Relay für Godot)
- Kenney.nl (kostenlose CC0-Assets)
- Inspiration: *Overcooked* (Koop-Chaos), *Gang Beasts* / *Human: Fall Flat* (wabbelige Steuerung), *Totally Reliable Delivery Service*, *Among Us* (Sabotage-Spannung)
