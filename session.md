# Session: 5-Achsen-Roboterarm in Three.js

**Datum:** 2026-09-27
**Projektordner:** `/Users/haraldbeker/Roboter`
**Dateien:** `prompt.md` (Aufgabenstellung), `robot-arm.html` (Ergebnis, eigenständige Seite), `index.html` (Landingpage), `README.md`, `social-preview.png`

## Aufgabenstellung (aus `prompt.md`)

- Roboter mit 5 Achsen: Greifarm mit zwei Gelenken, um seine Achse drehbar; Greifer um seine Achse drehbar; schließbare Zange.
- Darstellung in Three.js, pro Achse ein Regler.
- Am Boden ein beweglicher, greifbarer Klotz.
- **Zusatz:** Kollision des Greifers mit Boden und Würfel erkennen, bei Kollision blockieren und nur Zurückfahren erlauben; Würfel klein genug zum Greifen.

## Verlauf der Anforderungen

1. Grundgerüst: 5 Achsen mit Reglern, Orbit-Kamera, ziehbarer Klotz, Greifen durch Schließen der Zange nahe am Klotz.
2. Kollisionserkennung Boden/Würfel mit Sperre (nur Zurückfahren), Würfel verkleinert.
3. Boden erst als eisernes Gitterrost, später auf Wunsch als **Schachbrett** aus Stahlplatten.
4. **Kamera am Greifer** (mittig zwischen den Backen) mit kleinem Live-Monitor oben rechts.
5. Langsame **Pick-up-Demo**: eine Achse nach der anderen, je 2 Sekunden, mit Sicherheits-Warnton.
6. Demo-Choreografie verfeinert: über dem Würfel ausrichten, Zange so drehen, dass die Backen waagerecht beidseits des Würfels stehen, absenken, schließen, sofort anheben; beim Loslassen legt sich der Würfel flach auf den Boden.
7. Kein Autostart mehr – Demo startet per Button **„Pick-up starten“**.
8. Auf **GitHub Pages** veröffentlicht (`https://helmutqualtinger.github.io/Roboter/`).
9. **Social-Media-Vorschau**: Open-Graph-/Twitter-Bild und -Beschreibung hinterlegt.
10. **Sternenhimmel mit Galaxien** als Hintergrund statt flacher Farbe.
11. **Schatten** von Roboter und Würfel, zweite schattenwerfende Lichtquelle.
12. Würfel durch ein **Dodekaeder mit zwölf verschiedenfarbigen Seiten** ersetzt und vergrößert (`BLOCK_SIZE` 0,22 → 0,3).
13. **Saturn im Zenit** (gebänderte Kugel, Ringe mit Cassini-Teilung, drei Monde), Ringe und Laufbahnen schräg, Laufbahnen als Linien sichtbar.
14. **Demo-Ende:** Arm richtet sich auf Saturn aus (Rastersuche + Feinsuche über a1–a3), Greiferkamera zoomt in 8 s so weit heran, dass die äußerste Mondbahn das Bild füllt.
15. **Kompaktere Regler** (eine Zeile pro Achse) und neuer **Kamerazoom-Regler** (1×–10×).

## Aktueller Stand von `robot-arm.html`

### Achsen

| Achse | Funktion | Bereich |
| --- | --- | --- |
| 1 | Basisdrehung (um Hochachse) | −180° … 180° |
| 2 | Schulter | −90° … 90° |
| 3 | Ellbogen | −120° … 120° |
| 4 | Greiferdrehung (Rollen um eigene Achse) | −180° … 180° |
| 5 | Zange | 0 % (geschlossen) … 100 % (offen) |

### Funktionen

- **Greifen:** Zange ≤ 12 % geschlossen und TCP (Punkt zwischen den Backen) näher als 0,4 am Würfel → Würfel wird an den Greifer gehängt. Öffnen ≥ 22 % → loslassen, Würfel sinkt ab und richtet sich flach aus (Drehung um die Hochachse bleibt).
- **Kollisionssperre:** Greifer wird durch Kugeln angenähert (Gehäuse, beide Backenspitzen, TCP). Jede Bewegung, die tiefer in Boden oder Würfel führen würde, wird blockiert (Achsen-Box blinkt rot); Zurückfahren bleibt erlaubt. Gilt für Regler und Demo gleichermaßen. Sitzt der Würfel zwischen den Backen, dürfen diese ihn berühren.
- **Greifer-Kamera:** am Greifer montiert, Bild oben rechts im 3D-Bereich.
- **Pick-up-Demo (Button):** Basis zum Würfel → Zange öffnen → über dem Würfel schweben → Zange um −90° drehen (Backen waagerecht) → absenken → schließen → sofort anheben → zum Ablageort (+130°) drehen → knapp über dem Boden absenken → loslassen → zurück in Ruhestellung → auf Saturn ausrichten → Endzoom der Greiferkamera. Warnton (Web Audio) läuft währenddessen, Regler sind gesperrt, Reset bricht ab.
- **Reset:** Ruhestellung, Würfel an Startposition (3,0 / `BLOCK_SIZE`/2 / 0) und Kamerazoom 1×.
- **Hintergrund:** große, von innen sichtbare Kugel (Radius 500) mit prozedural gezeichneter Himmelstextur (2048×1024 Canvas, kein externes Bild) – ca. 1400 Sterne, drei farbige Spiralgalaxien mit Kern/Scheibe/Spiralarmen, schwaches Milchstraßenband. Vom Nebel ausgenommen (`fog:false`), damit er den Himmel nicht verwäscht; dreht sich mit der Kamera mit.

## Wichtige Erkenntnisse und behobene Fehler

- **Geometrie-Grenze:** Effektive Unterarmlänge bis zum TCP ist 2,685 (nicht ~2,04 wie anfangs überschlagen). Liegt der Würfel näher als ca. 2,3 Einheiten an der Basis, müsste der Ellbogen über sein ±120°-Limit einknicken → nicht erreichbar. Ein senkrechter Zugriff von oben ist mit diesen Gliedlängen praktisch nicht möglich; der Greifer steht beim Greifen ca. 40° schräg.
- **Backen-Ausrichtung:** Waagerecht können die Backen nur quer zur Armebene stehen. Damit sie gleichzeitig parallel zu den Würfelkanten sind, liegt der Würfel auf der Linie z = 0.
- **Numerischer Löser:** Handgerechnete Winkel lagen daneben. Ersetzt durch Mustersuche (8 Richtungen, mehrere Startpunkte) gegen die echte Vorwärtskinematik der Szene. Die erste Version (nur 4 Achsrichtungen, veraltete Werte) blieb an Randwerten hängen.
- **Kollisionsradien:** Zu große Kugeln (Gehäuse 0,29, Würfel-Raumdiagonale) blockierten weit vor echtem Kontakt → verkleinert (Gehäuse 0,13, Backen 0,07, Würfel 0,128).
- **Greifen vs. Kollision:** Ein Griff hat Vorrang vor der Kollisionssperre, sonst blockierten die Backenspitzen die letzte Annäherung.
- **Verzögerte Frames:** Ein großer Frame-Sprung in die Kollision ließ eine Achse komplett stehen. Jetzt fährt sie per Bisektion bis an die Grenze.
- **Anheben nach dem Griff:** Der erste Heben-Schritt bewegte eine Achse, die beim Absenken nicht benutzt wurde → Würfel sank kurz in den Boden. Jetzt wird die Absenkbewegung zuerst umgekehrt.
- **Banner blieb sichtbar:** `#demo-banner { display: flex }` überschrieb das `hidden`-Attribut → `#demo-banner[hidden] { display: none }`.
- **Warnton:** Browser blockieren Audio ohne Nutzer-Geste; durch den Start-Button ist das gelöst.
- **GitHub Pages lieferte 404:** Pages-Quelle stand auf `/docs` (Zweig `main`), diesen Ordner gibt es im Repo nicht. Umgestellt auf Repo-Wurzel (`main`, `/`).
- **Weißer Fleck im Screenshot:** Headless-Chromium ohne echte GPU (SwiftShader) erzeugte ein Rendering-Artefakt zwischen den beiden WebGL-Kontexten (Haupt- und Greiferkamera). Mit `--use-gl=angle --use-angle=metal --enable-gpu` (echte GPU) verschwunden – kein Bug der Seite selbst, nur des Screenshot-Verfahrens.

## Verifikation

Mit Playwright (Chromium, headless) im echten Browser geprüft, Testskripte lagen nur im Scratchpad:

- Beim Laden kein Autostart, Ruhestellung.
- Zwei aufeinanderfolgende Button-Läufe: jeweils gegriffen und abgelegt, keine Kollisionssperre ausgelöst, keine Seitenfehler.
- Würfel landet flach (0° Neigung) auf y = 0,11.
- Dreifache Wiederholung mit identischem Ergebnis (deterministisch).
- Live-Version auf GitHub Pages: Startseite und `robot-arm.html` liefern 200, `social-preview.png` liegt als `image/png` vor, Pick-up-Demo direkt auf der veröffentlichten Seite ausgelöst und geprüft (gegriffen + abgelegt, keine Fehler).
- MD5-Abgleich zwischen lokaler und veröffentlichter `social-preview.png` nach dem Update stimmt exakt überein.

## Veröffentlichung

- **Repo:** `HelmutQualtinger/Roboter` auf GitHub, Branch `main`.
- **GitHub Pages:** Quelle auf Repo-Wurzel umgestellt (siehe oben); erreichbar unter
  - `https://helmutqualtinger.github.io/Roboter/` (Startseite `index.html`)
  - `https://helmutqualtinger.github.io/Roboter/robot-arm.html` (Simulator)
- **Social-Media-Vorschau:** `social-preview.png` (1200×630, Playwright-Screenshot der laufenden Szene) plus Open-Graph-/Twitter-Meta-Tags (Titel, Beschreibung, Bild) in `index.html` und `robot-arm.html`. Der veraltete `screenshot.png` (zeigte den entfernten Autostart-Banner) wurde entfernt, README entsprechend aktualisiert.
- GitHubs eigene Repo-Social-Preview (Vorschau beim Teilen des reinen `github.com`-Links) lässt sich nicht per API setzen, nur manuell unter *Settings → General → Social preview*.

## Bekannte Grenzen / offene Punkte

- Liegt der Würfel außerhalb der erreichbaren Zone (zu nah an der Basis), greift die Demo daneben – es gibt noch keine Meldung dafür.
- Liegt der Würfel nicht auf der Armachse bzw. ist gedreht, können die Backen nicht gleichzeitig waagerecht und kantenparallel stehen; der Löser wählt dann den besten Kompromiss.
- Die Backen sind auf die Kanten eines Würfels ausgelegt (`solveGripperRoll`, 90°-Symmetrie); beim Dodekaeder passt das nur näherungsweise.
- Saturn liegt bei y = 100 in der Szene, damit der Endzoom das Gesamtsystem einrahmt; die Monde sind dafür überproportional groß.
- Der Würfel sinkt beim Loslassen linear ab (keine echte Physik).
- Die Orbit-Kamera lässt bei sehr nahem Zoom kombiniert mit starker Neigung ein Abtauchen unter die Bodenplatte zu (man sieht dann die Unterseite des Sockels). Vorbestehend, nicht durch den Sternenhimmel verursacht, noch nicht behoben.

## Benutzung

- **Online:** `https://helmutqualtinger.github.io/Roboter/robot-arm.html`
- **Lokal:** `robot-arm.html` direkt im Browser öffnen (Three.js r128 wird von `cdn.jsdelivr.net` geladen, Internet nötig).

Regler bedienen, im 3D-Bild ziehen zum Drehen, scrollen zum Zoomen, Würfel mit der Maus verschieben, „Pick-up starten“ für die Demo.
