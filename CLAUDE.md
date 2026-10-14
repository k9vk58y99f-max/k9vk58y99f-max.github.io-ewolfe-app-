# eWolfe – Projektübergabe für Claude Code

## Was das ist
eWolfe ist eine persönliche Lebensentwicklungs-App (Deutsch, mit Englisch umschaltbar):
Lebensrad mit fünf Bereichen (Gesundheit, Beziehungen, Finanzen, Berufung, Spiritualität/„sinn"),
Ziele (mit Grund, Maßnahmen, Status aktiv/erreicht), Ereignisse, Tagebuch (Seiten mit Text,
Zeichnungen, Fotos), Achtsamkeit (Meditationen, Links, Audio/Video), Backup als JSON,
Cloud-Sync über Supabase.

Der Nutzer ist kein Entwickler. Erkläre Schritte einfach und auf Deutsch.

## Aufbau
- **Alles steckt in einer Datei: `index.html`** (HTML, CSS und JavaScript, keine Build-Schritte,
  keine externen Bibliotheken außer Google Fonts „Inter"). Zusatzdateien: Audios im Ordner `audio/` und die Bilder der
  Achtsamkeits-Übungen im Ordner `img/` (seit 1.52.0, siehe unten).
- Bilder der Übungen (seit 1.52.0): Papier-Bilder des Nutzers, freigestellt (transparenter Hintergrund) als WebP in `img/`:
  `lotus` (neuer Blauer Lotus, Bild der Box-Breathing-Karte und Sticker, seit 1.58.1), `lotus1-3` (Knospe, halb, offen), `cherry1-6` (Knospen → volle Blüte), `stones`, `heart`, `bowl`. `imgPic(datei)` packt
  ein Bild in ein kleines SVG (Karten, Player). `lotusPictureSvg`/`blossomPictureSvg`/`stonesPictureSvg`/`heartsPictureSvg`/
  `bowlPictureSvg` liefern jetzt diese Bilder (Karten, Player). Die Atem-Animation dagegen ist gezeichnet, damit sich die
  Blätter fließend bewegen (bloßes Überblenden der Bilder gefiel nicht). Box Breathing: der ursprüngliche gezeichnete
  Lotus (`lotusMarkup`); ein Origami-Umbau (1.52.1) gefiel nicht und wurde in 1.54.1 zurückgenommen. Für einen Lotus in
  der Qualität der Bilder braucht es ein Video vom Aufblühen (wie beim Kirschzweig).
- Box Breathing (seit 1.59.0): neuer Blauer Lotus aus den 20 Bildern des Nutzers (Raster `images/52`): Bilder 10–16
  (Knospe → offen), gemeinsame Seerosenblätter aus einem Knospenbild (Blüte und dunkles Kelchblatt darunter entfernt und
  übermalt, cv2.inpaint), Blüte 20 px (von 560) höher, Zwischenbilder per optischem Fluss: 25 Bilder `img/lotus-v/00…24`.
  Hintergrund: Papier-Wasser `img/water.webp` statt des gezeichneten Sees (Ringe ausgeblendet). Gleicher Mechanismus wie
  4-7-8 (`BREATH_FRAMES`, `breathLotusSet`). Werkzeuge: Python-venv mit Pillow, NumPy, OpenCV im Scratchpad.
- 4-7-8 (seit 1.53.0): Kirschzweig aus dem Video des Nutzers (`IMG_1031.mov`, 512 px, 30 fps): das Stück 1,67–3,33 s, in dem die
  Knospen aufgehen, als 26 Einzelbilder in `img/cherry-v/00…25.webp`, seit 1.54.0 freigestellt vor dem blauen Himmel
  (Durchsichtigkeit beim Überblenden gegen das erste/letzte Bild gerechnet, äußerster Pixelrand weggeschnitten). `breathLotusSet` läuft gleichmäßig durch die Phase: 4 s einatmen = Bild 0→25, 7 s halten = offen,
  8 s ausatmen = zurück; zwei Bilder übereinander, das nächste wird eingeblendet (`CHERRY_FRAMES`, `cherryFrameUrl`,
  `cherryPreload`). Einzelbilder aus Videos: kleines Swift-Werkzeug mit AVAssetImageGenerator (ffmpeg ist nicht installiert).
  Die Knospe unten in der Mitte (links neben der unteren rechten Blüte) bleibt in allen Bildern wie im ersten Bild
  (dort machte das Video aus einer Knospe zwei und ließ lose Blütenblätter liegen; Ausschnitt mit weichem Rand ersetzt).
  Die Zitrone ist weiterhin gezeichnet (`lemonMarkup`). Neue Bilder freistellen: Hintergrund ist ein gleichmäßiges
  Dunkelblau, per Farbabstand zum Rand-Median entfernen, weiche Schatten nur nahe am Motiv behalten.
- Eingebaute Meditationen mit Audio (`builtIn:true` in `DEFAULT_MEDITATIONEN`, Feld `audio` = Pfad zur Datei,
  z. B. „Lemonmeditation" → `audio/lemonmeditation.m4a`, die erste Aufnahme mit ihrer ursprünglichen Musik; die überarbeitete Fassung `audio/lemonmeditation_v2.m4a` liegt für später bereit) werden per `ensureBuiltInMeditations()` bei allen
  Nutzern ergänzt und aktuell gehalten. Sie lassen sich nicht löschen und kosten keinen App-Speicher.
  Ihre Einleitung steht zweisprachig im `I18N` (Feld `notesKey`, z. B. `mediLemonNote`).
  „Box Breathing" (`seed-box`, Feld `tool:'box'`) öffnet eine geführte Atemübung (`breathOpen()`, Fenster
  `#breath-modal`: Punkt wandert ums Quadrat, je 4 s ein/halten/aus/halten; in der Mitte ein Blauer Lotus
  (Seerose von oben, Blau-Violett: außen dunkel, innen hell, gelbe Mitte, auf grünen Seerosenblättern), der sich beim Einatmen öffnet und
  beim Ausatmen schließt: Zeichnung `lotusMarkup(prefix, offen)`, Bewegung `lotusApply()`, Farben `LOTUS_RINGS`/`LOTUS_LEAF`; leichte Abweichungen je
  Blütenblatt über `lotusPetalVar()`). Darunter ein See in Türkis/Hellblau (`lakeWater`) mit breiten, weichen
  Lichtringen (`#breath-ripples`, in `breathFrame()` bewegt); Punkt, Zahl und „Beenden" in Lila (#6d4aff).
  Dazu Handpan-Musik, live im Browser erzeugt (Web Audio, `hpStart()`/`hpStop()`, D-Kurd-Skala, festes Stück
  `hpPiece()` mit 38 Atemrunden = 608 s: fließende Arpeggien, ein Akkord je Atemphase, Ein-/Ausatem-Geräusche
  aus gefiltertem Rauschen (`hpBreath()`), weicher Übergang beim Neustart). Stummschalten
  wird je Gerät gemerkt (`ewolfe_breath_music`). Spielt auf dem iPhone auch im Lautlos-Modus
  (`navigator.audioSession` bzw. stilles <audio>-Element).
- Achtsamkeit (seit 1.30.0): eingebaute Übungen als Karten mit Bild + Titel (`mediToggleCard()`), aufgeklappt
  Text + gleicher „Starten"-Knopf. Bilder als SVG: `lotusPictureSvg()` (derselbe Lotus, offen), `lemonPictureSvg()` (ohne Schein; Zeichnung in `lemonMarkup()`).
  Im Player liegt die Zitrone auf einem hellblauen Quadrat wie beim Box Breathing, mit diagonalem Strich und Farbverlauf
  wie beim App-Icon (`lemonPlayerSvg()`). Lemonmeditation
  spielt im Fenster `#medi-player-modal` (`mediPlayerOpen()`). Das Formular zum Hinzufügen ist eingeklappt
  (`mediShowForm()`), mit „Abbrechen".
- Längere Texte zu einer Übung (seit 1.51.0, `MEDI_INFO_KEYS` → `mediBoxInfo`, `ex478Info` im `I18N`: Einleitung,
  Schritte, „Gut zu wissen"): stehen aufklappbar unter „Starten" (`mediInfoHtml()`, `mediToggleInfo()`). Nur beim
  allerersten Öffnen einer Karte auf einem Gerät stehen sie offen (Merker `ewolfe_medi_info_seen`), danach eingeklappt.
- Achtsamkeit 2.0 (seit 1.38.0, Abschnitt „Achtsamkeit 2.0“ im Code): weitere eingebaute Übungen (`MEDI_ORDER`):
  4-7-8-Atmung (`breathOpen('b478')`, gleiches Fenster wie Box Breathing, aber statt des Sees ein blauer Himmel mit
  Wolken (`.breath-sky`, der Zweig wird am Quadrat abgeschnitten über `#breath-clip`); Blüten öffnen sich höchstens zu 90 %
  (`BLOSSOM_MAX_OPEN`)
  und mit einem Kirschzweig statt des Lotus (`blossomMarkup()`/`blossomApply()`, nach einem Foto des Nutzers):
  schlanker dunkler Zweig diagonal, zartrosa Blüten nur von oben (zwei ganz leicht schräg), große Blüte in der Mitte,
  kleine Gruppen geschlossener Knospen (`budTop`: prall eiförmig, matt, rotbrauner Kelch, zeigt vom Zweig weg über
  `awayAngle`), keine Blätter; ein Ästchen läuft von der Mitte nach rechts unten zu einer weiteren Blüte; der ganze Zweig ist nach links unten
  verschoben (`SHIFT`), oben rechts eine Zweigspitze mit Blüte und Knospen; `blossomMarkup(P, open, {labels:true})`
  nummeriert die Knospen mit festen Nummern (`num`; `labels:[…]` zeigt nur bestimmte); jede Blüte/Knospe hängt an
  einem kurzen braunen Stiel zum nächsten Zweigpunkt (`stalkTo`), Ästchen beginnen exakt auf dem Ast (`TW`); alle
  Blüten im Farbton `pt` mit Glanz, heller Randkante und feinen Adern; Papier-Stil wie der Lotus. Blüten öffnen sich aus runden Knospen (`.lt-bud`), leicht versetzt (`lag`). Sieben „späte“ (`late:1`,
  `data-late`, `blossomLateOpen()`) beginnen erst, wenn die anderen halb offen sind, und sind am Ende halb offen: nach
  dem Einatmen drei Stadien (offen, halb offen, Knospe). Ein Katalog-/Auswahl-
  Verfahren mit Stellen A–H wurde ausprobiert und wieder entfernt; Musik je Rhythmus `hpPiece(key)`), 5-4-3-2-1 (`groundOpen()`), Dankbarkeit am Abend (`gratOpen()`, auf Wunsch ins Tagebuch dieses
  Jahres: `gratToDiary()`), Stille Meditation (`timerOpen()`, Gong `hpGong()`, Musik „calm“ ohne Atemgeräusche).
  Start immer über `mediRun(id)`: Stimmung vorher (`moodAsk()`), Übung, beim Schließen (`achSessionEnd()` aus
  `closeModal`) „erledigt“ (ab 30 s) und Stimmung nachher. Alles steht in `state.achtsamkeit.log` (wird synchronisiert,
  Zusammenführen nach `id` in `mergeBackupInto`). Oben auf der Seite: Impuls für den niedrigsten Bereich
  (`impulses` im `I18N`), „Tage in Folge“ (`achStreak()`) und „Tägliche Erinnerung“ (`remindAdd()`: .ics-Datei mit
  täglichem Termin für den eigenen Kalender, kein Server nötig). Unten: Auswertung der Stimmung (`achMoodHtml()`).
  Papier-Stil wie der Lotus: jede Lage (Blätter, Schale, helles Fruchtfleisch, Spalten …) eine eigene Gruppe mit weichem
  Schlagschatten (`feDropShadow`); so auch Zitrone (`lemonMarkup()`, Spalten mit kleinen Saftzellen) und Kirschzweig (4-7-8). Dankbarkeit zeigt zwei Origami-Herzen
  übereinander, jedes aus einem gefalteten Papier (`heartsPictureSvg()`/`origamiHeart()`, beide Hälften gleich
  gefaltet, sanfte Farbstufen); weitere Herzformen zur Auswahl in `heartShapeSvg(kind)`
  (round, slim, origami, petals, plump). Ein Glas voller Glücksmomente (`jarPictureSvg()`) liegt als Alternative bereit. Weitere Bilder (Verläufe, Blätter mit Adern: `natLeaf()`, die Adern sind per clipPath auf die Blattform zugeschnitten): Mond (für eigene Übungen), Lavendel
  (für einen späteren Body-Scan), Bachkiesel mit Keimling (5-4-3-2-1: `stonesPictureSvg()`, Kieselform `pebblePath()`, jeder Kiesel drei flache Papierlagen in den Blau-Violett-Tönen des Lotus, Vorschlag „K2“), Klangschale auf Kissen mit Klöppel, Klang als zwei mintgrüne Papier-Sicheln (Stille: `bowlPictureSvg()`, flache Papierlagen ohne Glanz, Vorschlag „S3“).
  Der Body-Scan mit Computerstimme wurde in 1.38.2 wieder entfernt (`RETIRED_BUILTINS` nimmt ihn aus allen Listen);
  er soll später mit einer eigenen Aufnahme zurückkommen (wie die Lemonmeditation, Audio im Ordner `audio/`). Eigene Übungen sind
  Karten mit wählbarem Bild (`MEDI_ICONS`/`ownPictureSvg()`, Feld `icon`).
- Sprachwahl beim allerersten Öffnen eines neuen Geräts im Anmeldefenster (`syncNeedsLangPick()`,
  Merker `ewolfe_lang_chosen`).
- Symbole: eigene Linien-Icons statt Emojis über `ico(name)` / `UI_ICONS` (Tagebuch, Löschen …).
- Fenster (`.modal-backdrop` ohne `.page-modal`) liegen immer über den Seiten (z-index 110 statt 100).
- Hauptfarbe (seit 1.33.0): Mintgrün `#58d48f` (rgba 88,212,143) statt des früheren Türkis `#33e6c4`.
  Die CSS-Variable heißt weiterhin `--ew-turquoise`.
- Ziele-Seite: Hinweis unter der Überschrift (`goalsSub`: bis zu 5 Ziele je Bereich, kleine, mittlere, höchstens ein
  langfristiges). Die Tabellen laufen dort von Rand zu Rand; jede Bereichsspalte ist so breit, dass die nächste
  rechts gerade noch hereinschaut (`#goals-page … col.col-area`, Breite über `cqw`).
- Lebensräder vergleichen: Nach „Anzeigen" endet der Auswahlmodus automatisch (Rahmen verschwinden).
- Nur drei Textgrößen (seit 1.34.0, Variablen `--fs-h` 22px, `--fs-sub` 17px, `--fs-text` 15px, Block „Three text sizes
  only" am Ende des CSS): Überschriften groß und weiß, Unterüberschriften kleiner und grün, sonst normaler Text in weiß.
  Ausnahmen: Beschriftung der unteren Leiste, Versionszeile, Atemzähler, die Tagebuchseiten selbst.
- Lebensräder (seit 1.36.0): statt Wörtern Symbole je Bereich (`AREA_ICONS`/`areaIco()`: Gesundheit Herzschlag,
  Beziehungen Herz, Finanzen steigender Chart, Berufung Kompass, Spiritualität Lotus) mit Prozentwert daneben
  (`wheelLabelsSvg()`, Platzierung `wheelLabelsFit()`, Tippflächen `wheelLabBox()`). Größe aus der angezeigten Breite:
  Symbol 26px, Zahl 15px darunter (seit 1.37.0). Größe des Rads und Höhe der Zeichnung werden je Rad aus der
  angezeigten Breite berechnet (`wheelGeo()`/`wheelGeoFor()` setzen `CX`, `CY`, `MAXR`). Die Symbole stehen auch auf
  der Seite Lebensbereiche, in den Kopfzeilen der Ziele-Tabellen und auf der Bewertungsseite.
- „Neues Lebensrad erstellen" hat ein ✕ (`#eval-close-btn`), außer beim allerersten Lebensrad.
- Vergleich: nur das neueste Rad gefüllt, ältere als Linien darüber (Mintgrün und Gelb, beide durchgezogen) mit Punkten;
  an jedem Bereich der neueste Wert und die Veränderung seit dem ältesten verglichenen Rad (+ grün / − rot).
  Das Fenster reicht fast bis oben; darunter ein Satz, je zehn Varianten (`cmpUpV`/`cmpUp1V`/`cmpUpAllV`, bei Rückschritten `cmpDownV`, sonst `cmpSameV`); welche, hängt von den verglichenen Rädern ab (gleicher Vergleich = gleicher Satz).
- Tagebuch (seit 1.37.0): Werkzeuge → „Lebensrad einfügen" (`tbInsertWheel()`, Bild als SVG `wheelPictureSvg()`,
  verschieben/vergrößern/löschen wie Fotos). „Neue Seite ▾ → Lebensbereiche" legt eine Seite mit fünf Feldern an
  (`page.bereiche = { bereich: Text }`, je Feld Symbol + Name als kleine Überschrift; ein volles Feld nimmt nichts mehr an).
  Beim Zusammenführen und Zählen beschriebener Seiten zählt dieser Text mit (`tbAreaTextLen()`).
- Ränder (seit 1.36.0): Seiten 14px seitlich, Karten auf der Startseite 10px, Ziele-Tabellen mit schmalem Rand (8px).
- Tagebuch-Titelblatt: Das Buch füllt den Platz, nichts scrollt (`.tb-body-cover`). Antippen des Titelblatts schlägt
  das Tagebuch auf (`.tb-cover-tap`); keine eigenen Knöpfe mehr (Bibliothek oben links in der Kopfzeile). Nur bei neuen
  Tagebüchern liegt das Zahnrad auf dem Buch. Schrift der ganzen App: Inter (Caveat nur für Handschrift im Tagebuch).
- Fotos im Tagebuch: antippen → Rahmen, roter Löschknopf (mit Rückfrage) und grüner Griff unten rechts zum Vergrößern/
  Verkleinern (`.tb-img-box`, `tbSelImg`, `tbSelectImg()`); Verschieben wie bisher. Breite bleibt in `wPct` gespeichert.
- Veröffentlicht über **GitHub Pages** aus diesem Repository (Branch `main`, Ordner `/`):
  https://k9vk58y99f-max.github.io/k9vk58y99f-max.github.io-ewolfe-app-/
- Ein Commit + Push auf `main` ist die Veröffentlichung; nach 1–2 Minuten ist es live.
- Bei jeder Änderung `APP_VERSION` erhöhen (aktuell `1.63.3`) und `APP_BUILD` anpassen.
  Die Version steht unten in der App.

## Neu seit 1.55.0 (iPad, Wischen, Tagebuch, Sticker, Impulskarten)
- **Volle Breite** auf Tablet/Rechner: `.page-modal > .modal` ohne Breitenbegrenzung; Tagebuch-Bibliothek und Titelblatt groß.
  Die Tagebuchseite wächst als Ganzes (`tbSizeVars()`: `--tb-page-w` = Fensterbreite − 24 px, höchstens 1200; `--tbk` =
  Breite/560; Schrift, Zeilen, Ränder, Linien/Karos/Punkte und Lebensbereiche-Seite rechnen mit `--tbk`). Dadurch passt auf
  jede Seite genau gleich viel Text wie bei 560 px, es wird nichts neu umbrochen. Handys (< 640 px) bleiben wie bisher.
- **Wischen** (`NAV_ORDER`, `swipeGo()`, `swipeInit`): links/rechts = nächste/vorige Seite der Leiste, an den Enden Schluss;
  am Rechner seitliches Scrollen mit zwei Fingern. Seitlich scrollbare Tabellen (Ziele) scrollen zuerst (`swipeScrollsX`).
  Tagebuch: Titelblatt links = Buch öffnen, rechts = zurück zu Ziele; im Buch nur im Ansichtsmodus (blättert Tagebuchseiten),
  beim Schreiben nichts. Fenster über den Seiten blockieren es (`swipeBlocked`).
- Startseite: „Ziele bearbeiten“ entfernt, „Frühere Lebensräder“ direkt unter „Neues Lebensrad erstellen“.
- **Tagebuch**: „Jetzt schreiben“ steht seit 1.58.0 in den Tagebuch-Einstellungen (`.tb-start-btn`), nicht mehr unter dem
  Titelblatt; das Buch öffnet immer auf der letzten Seite; auf Tablets ist die Seite so groß, dass sie hochkant ganz passt; Tipp auf die Seitenzahl öffnet die Seitenwahl
  (`tbPagePickHtml`, `tbGoToPage`); Seitentyp „Lebensbereiche“ in den Einstellungen (`defaultPageType = 'areas'`, neue Seiten
  bekommen dann die fünf Felder; Text, der auf die nächste Seite überläuft, bleibt normale Seite: `tbTextType`).
  Text aus der Achtsamkeit (Dankbarkeit, Impulskarte) beginnt unterhalb des tiefsten Bildes der Seite (`tbAppendBlock`).
- **Sticker** (so heißen die Bilder der Übungen jetzt): im Werkzeug-Menü „Sticker“ (`TB_STICKERS`, `tbInsertSticker`):
  Lotus, Zitrone, Kirschzweig, Steinturm, die zwei gefalteten Herzen (das neue Papierherz gehört zur Dankbarkeit), Klangschale,
  Blatt. Auf der Seite ohne Fotoschatten (`sticker:true`, Klasse `is-sticker`).
- **Impulskarten** (`IMPULSE_CARDS`, 40 Karten, 8 je Bereich, eigene Sätze und kurze Zitate mit Namen, de/en): Knopf
  „Impulskarte ziehen“ oben auf der Achtsamkeits-Seite (ersetzt die alte Impulszeile), einmal am Tag beim Start als Karte
  (`maybeShowImpulseCard`, Merker `ewolfe_impulse_day`), nur per Knopf zu schließen; „Ins Tagebuch“ legt Bild + Spruch auf
  eine Seite. Seit 1.58.0: Karte ohne Rahmenfenster (Format 5:7, `impulseCardHtml`), Startkarte schließt per Antippen, auf der
  Achtsamkeits-Seite: daneben tippen schließt, zur Seite wischen zieht eine neue (`impulseNext`), kleines Buch unten rechts
  überträgt die ganze Karte als Bild ins Tagebuch (`impulseCardSvg`). Oben auf der Achtsamkeits-Seite eine Mini-Karte (die
  zuletzt gezogene, `impulseLast`) mit Text daneben. Platzhalter-Bilder sind ältere Zeichnungen (`IMPULSE_PH`, `img:'ph0…7'`),
  nicht die Bilder der Übungen. Bilder sind noch Platzhalter (`IMPULSE_PLACEHOLDER = true`, roter Schriftzug); echte Bilder: Feld `img` je Karte
  setzen (Datei in `img/`) und `IMPULSE_PLACEHOLDER` auf `false`.

## Bücherregal, mehrere Tagebücher, Trophäen (seit 1.60.0)
- Mehrere Tagebücher pro Jahr ohne Grenze: Schlüssel in `state.tagebuch.jahre` = "2026", "2026/2", … (`tbBookKeys`,
  `tbLatestBook`, `tbNewBook`, `tbNewBookAsk`). Jedes Buch hoch- oder querformatig (`diary.format`, `tbIsLand`; Querseite 4:3,
  Klasse `tb-land-page`, `--tb-page-wl`/`--tbk-l` auf Basis 746 px; Seitenumbruch misst im Format des Buchs: `tbMeasureLand`).
  Dankbarkeit/Impulskarten gehen ins neueste Buch des Jahres. Das Zahnrad liegt jetzt immer auf dem Titelblatt.
- Bibliothek = Holzregal (`tbRenderLibrary`, `tbSpineHtml`): Buchrücken (Breite nach Seitenzahl), Platz mit Plus = neues Buch,
  darunter Trophäen-Böden (je Reihe ein Brett). Tablet größer (`tbShelfScale`).
- Trophäen (`trophyDefs`/`trophyList`, aus vorhandenen Daten berechnet, zählen rückwirkend, auf allen Geräten gleich): Übungen
  1/5/10/25/50/100 (Kugeln), je Übung 10× unter Glasglocke (Dankbarkeit 7×), Tage in Folge 3/5/7/14/30/100 (Kristalle, wachsen),
  Lebensräder 1/5/10, +10 in einem Bereich, alle Bereiche besser, Ziele 1/5/10, Tagebuchseiten 1/10/50/100, erste Dankbarkeit.
  Neue erscheinen einmal groß (`trophyCheck`, Merker `ewolfe_trophies_seen`; beim allerersten Mal werden alle still einsortiert).
  Seit 1.60.0 Bilder des Nutzers (`kind:'img'`, Dateien in `img/`): Regal `shelf.webp` mit fünf Fächern (`SHELF_X`/`SHELF_ROWS`,
  Bücher hinten, Trophäen etwas weiter vorne), Übungen = farbige Kristalle (`crystal-mint…gold`), Tage in Folge = wachsender
  Kristall (`crystal-1…6`), Box/4-7-8 = Glasglocken in 3 Stufen (`trophy-lotus-1…3`, `trophy-cherry-1…3`, nach 3/10/25),
  Lebensräder/Ziele/Seiten/Dankbarkeit = Orbs (violett/gold/blau/rosa, wachsen), +10 = grüner, alle Bereiche = Regenbogen-Orb.
  Von wachsenden Trophäen (`group`) steht nur die höchste Stufe im Regal (`trophyShelf`).

- Seite **Lebensbereiche** (seit 1.26.0, ganz links in der Leiste, Beschriftung „Bereiche"): erklärt die fünf
  Bereiche und wie das Lebensrad funktioniert (`renderAreasPage()`, Texte `areasText` usw. im `I18N`). Sie öffnet
  sich einmal von selbst (`maybeShowAreasIntro()`, Merker `ewolfe_areas_seen_v1` je Gerät): für Neue nach der
  Anmeldung als Startseite (Knopf „Erstes Lebensrad erstellen"), für alle anderen einmal nach dem Update
  (ohne Knopf: wer schon ein Lebensrad hat, sieht keinen Knopf). Unter „Neues Lebensrad erstellen" gibt es „Ziele bearbeiten".
- Aufbau (seit 1.20.0): Leiste am unteren Rand (`#bottom-nav`) mit vier Seiten: **Lebensrad** (Startseite:
  Rad, Knopf „Neues Lebensrad erstellen" → Bewertungsseite `#eval-modal`, darunter „Frühere Lebensräder" mit
  Vergleichen und Löschen), **Ziele** (`#goals-page`: Ziele, erreichte Ziele, Ereignisse im Überblick),
  **Tagebuch**, **Achtsamkeit**. Seiten haben kein ✕; gewechselt wird über die Leiste (`navGo()`/`navSync()`).
  Zoomen nur im Tagebuch. Tippen aufs App-Icon oben links spielt die Animation (`replayIntro()`).
- Oben rechts das Konto-Symbol (`#sync-btn`) → „Einstellungen" (`#settings-modal`): Mitteilungen, Konto &
  Cloud-Sync, Sprache, Backup. Der türkise Punkt erscheint nur bei Mitteilungen (`appNotes()`): neue
  App-Version (`checkForUpdate()` liest `APP_VERSION` der veröffentlichten Seite, „Jetzt aktualisieren")
  oder Sync-Fehler.
- Gelöschte Lebensräder: Grabstein `b:<bereich>:<ts>` in `state.geloescht`.
- `STORE_KEY = 'ewolfe_data_v1'`: der gesamte App-Stand liegt als ein Objekt `state` in localStorage.
  Felder: `bewertungen`, `ziele`, `ereignisse`, `meditationen`, `tagebuch.jahre`, `settings`.
- `saveState()` speichert lokal, rendert die Übersichtstabellen und stößt den Sync an.
- `AREAS = ['gesundheit','beziehungen','finanzen','berufung','sinn']`
- Texte: Objekt `I18N` mit `de` und `en`; neue Texte immer in beiden Sprachen anlegen
  (in englischen Texten keine Apostrophe in einfach gequoteten Strings ohne Escape).
- Dialoge: `uiDialog()`, `uiConfirm()`, `uiNotice()` statt `alert`/`confirm`.
- Backup: `exportBackup()`, `importBackupFile()`, `mergeBackupInto(cur, inc)` (Vereinigung,
  löscht nie etwas). Beim Import mit „Ersetzen" bleibt ein volleres Tagebuch automatisch erhalten.
- Tagebuch-Bedienung (seit 1.23.0): Seitenpfeile „‹ 2/5 ›" in der Kopfzeile (`tbPageNavHtml()`), Werkzeuge
  über den runden türkisen Knopf unten rechts (`tbToolsHtml()`, Menü: Schreiben mit Pfeil → Druck-/
  Handschrift, Zeichnen, Radierer, Neue Seite ▾, Seiten, Seitenansicht ▾ Leer/Liniert/Kariert/Gepunktet).
  Seite löschen: runder Knopf unten links (`.tb-del-fab`). Auf dem Blatt selbst nur noch „Bild".
- Tagebuch-Einstellungen (Titel, Farbe, Material, Schrift, Seitentyp, Coverbild) als Pop-up
  (`tbCoverSettingsHtml()`): auf dem Titelblatt per Zahnrad nur, solange das Tagebuch neu ist
  (`tbCoverSetupOpen()`, `diary.setupDone` wird beim ersten Aufschlagen gesetzt), sonst über das Zahnrad
  am Buch in der Bibliothek (`tbEditYearCover()`). Die Seitengröße NICHT von der Bildschirmhöhe abhängig machen: der Seitenumbruch
  misst an der Seitengröße und würde sonst bestehende Seiten neu umbrechen.
- Tagebuch: immer einseitige Ansicht (`tbIsSpread()` liefert `false`), Seitenumbruch über
  `tbHandleOverflow()` / `tbRepaginateDiary()`. Bilder werden in Prozent der Seite gespeichert
  (`xPct`, `yPct`, `wPct`).
- Speicherplatz: localStorage ca. 5 MB. Fotos werden verkleinert (`shrinkImageFile`,
  `shrinkStoredImages`). Audio/Video-Anhänge prüfen den tatsächlich freien Platz.

## Cloud-Sync (Supabase)
- Projekt „ewolfe", Ref `egoeosnqzrzikbwwoouo`, Region eu-central-1 (Frankfurt), kostenloser Plan.
- Tabelle `public.user_data`: `user_id` (uuid, PK, verweist auf auth.users), `data` (jsonb, ganzer
  App-Stand ohne `settings`), `revision` (bigint), `updated_at`.
  Row-Level-Security: jede Person liest/schreibt nur ihre eigene Zeile; `anon` hat keinen Zugriff.
  Ein Trigger erhöht `revision` und setzt `updated_at` bei jedem Update.
- Login per E-Mail + Passwort, „Confirm email" ist im Dashboard ausgeschaltet.
- **Anmeldung ist Pflicht** (seit 1.15.0): Ohne Sitzung wird das Sync-Fenster nach der Intro-Animation
  zur nicht schließbaren Sperre (`syncShowGate()`, CSS-Klasse `gate`). Neues Gerät → „Konto erstellen",
  Gerät mit früherer Anmeldung (`ewolfe_last_email` bzw. Sync-Meta) → „Anmelden" mit vorausgefüllter
  E-Mail. Auch nach Abmelden oder abgelaufener Sitzung erscheint die Sperre. Ohne Internet kann man
  sich nicht neu anmelden (bewusst so gewünscht).
- **Passwort vergessen** (seit 1.16.0): „Passwort vergessen?" im Anmeldefenster → `POST /auth/v1/recover`
  mit `redirect_to` = App-Adresse. Der Mail-Link führt zurück mit `#access_token=…&type=recovery`;
  `syncHandleAuthLink()` liest das aus, entfernt es aus der Adresse, meldet an und verlangt ein neues
  Passwort (`syncSaveNewPassword()` → `PUT /auth/v1/user`). Abgelaufener Link → Hinweis im Fenster.
  Achtung: Ohne eigenen SMTP-Server verschickt Supabase Mails nur an Mitglieder des Supabase-Teams
  und nur wenige pro Stunde. Für echte Nutzer muss im Dashboard ein eigener SMTP-Dienst eingetragen werden.
  Site URL im Dashboard = die GitHub-Pages-Adresse oben (bei Adresswechsel anpassen!).
- Im Code: Abschnitt „Cloud-Sync (Supabase)", ohne Bibliothek direkt per `fetch` gegen
  `/auth/v1/...` und `/rest/v1/user_data`. Der Schlüssel im Code ist der öffentliche
  „publishable" Key, das ist so gewollt. Niemals einen geheimen Schlüssel in die Datei schreiben.
- `syncNow()` – Sicherheitsregeln, die erhalten bleiben müssen:
  1. Jeder Schreibvorgang nennt die Revision, auf der er beruht (`revision=eq.X`); hat jemand
     dazwischen geschrieben, wird nichts überschrieben, sondern neu geholt und zusammengeführt.
  2. Ein Gerät ohne eigene Daten (`localIsPristine()`) lädt nur herunter, nie hoch.
  3. Schrumpfen Lebensräder oder beschriebene Tagebuchseiten plötzlich deutlich gegenüber dem
     letzten Sync (`syncCounts`), wird nicht hochgeladen, sondern mit der Cloud zusammengeführt.
  4. Während Tagebuch oder ein Bearbeitungsfenster offen ist, wird nichts Fremdes eingespielt.
  5. Jedes Gerät behält seine eigene Sprache (`settings` wird nicht synchronisiert).
  6. Löschungen reisen mit: `state.geloescht` (`'z:'/'e:'/'m:'/'s:' + id` → `{t, d}`) wird
     synchronisiert, `syncMerge()` entfernt Gelöschtes (`delApply`). Ein eingespieltes Backup
     holt Gelöschtes bewusst zurück (`delReviveAll`).
- Bekannte Grenzen: es wird jeweils der ganze Stand übertragen.

## Testen
Bisher wurde mit Playwright (Chromium, headless) getestet: App als `file://` laden, ca. 4 Sekunden
Intro-Animation abwarten (danach erscheint die Anmelde-Sperre, solange keine Sitzung besteht), dann über `page.evaluate` und Klicks prüfen. Für den Sync wurde der
Supabase-Server mit `context.route(...)` nachgebaut (Konten, Revisionen, abgelaufene Tokens).
Vor jedem Push: Syntax prüfen und die betroffenen Abläufe testen. Daten des Nutzers nie riskieren.

## Offene Wünsche
- Eigenen SMTP-Dienst in Supabase eintragen, damit Passwort-Mails alle Nutzer erreichen
- Offline-Modus (Service Worker) und dauerhaften Speicher anfordern
- Eventuell eigene Domain statt der GitHub-Adresse
- Später: Community, Onboarding, Kalender-Export, Teilen, Admin-Bereich, KI-Funktionen, Abo
