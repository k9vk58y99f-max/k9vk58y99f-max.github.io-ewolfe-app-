# eWolfe – Projektübergabe für Claude Code

## Was das ist
eWolfe ist eine persönliche Lebensentwicklungs-App (Deutsch, mit Englisch umschaltbar):
Lebensrad mit sechs Bereichen (Gesundheit, Beziehungen, Finanzen, Berufung, Spiritualität/„sinn", Freude),
Ziele (mit Grund, Maßnahmen, Status aktiv/erreicht), Ereignisse, Tagebuch (Seiten mit Text,
Zeichnungen, Fotos), Achtsamkeit (Meditationen, Links, Audio/Video), Backup als JSON,
Cloud-Sync über Supabase.

Der Nutzer ist kein Entwickler. Erkläre Schritte einfach und auf Deutsch.

## Aufbau
- **Alles steckt in einer Datei: `index.html`** (HTML, CSS und JavaScript, keine Build-Schritte,
  keine externen Bibliotheken). **Schriften seit 1.92.0 lokal** in `fonts/` (`fonts/fonts.css` + woff2, nur Latein/Latein erweitert:
  Inter, Marcellus, Noto Serif Display schmal, Caveat, Cormorant Garamond, Jost) – keine Verbindung mehr zu Google (Datenschutz). Neue Schrift: woff2 von
  Google laden, `@font-face` in `fonts/fonts.css` ergänzen, nie wieder fonts.googleapis.com einbinden. Zusatzdateien: Audios im Ordner `audio/` und die Bilder der
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
  `closeModal`, seit 1.72.1 auch beim Tippen daneben und beim Seitenwechsel) „erledigt“ (ab 30 s) und Stimmung nachher;
  erst danach der Vergleich vorher → jetzt mit Gesichtern und einem Satz (`moodCompare()`). Alles steht in `state.achtsamkeit.log` (wird synchronisiert,
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
- Bei jeder Änderung `APP_VERSION` erhöhen (aktuell `1.96.0`) und `APP_BUILD` anpassen.
  Die Version steht unten in der App.

## Neu seit 1.55.0 (iPad, Wischen, Tagebuch, Sticker, Impulskarten)
- **Volle Breite** auf Tablet/Rechner: `.page-modal > .modal` ohne Breitenbegrenzung; Tagebuch-Bibliothek und Titelblatt groß.
  Die Tagebuchseite wächst als Ganzes (`tbSizeVars()`: `--tb-page-w` = Fensterbreite − 24 px, höchstens 1200; `--tbk` =
  Breite/560; Schrift, Zeilen, Ränder, Linien/Karos/Punkte und Lebensbereiche-Seite rechnen mit `--tbk`). Dadurch passt auf
  jede Seite genau gleich viel Text wie bei 560 px, es wird nichts neu umbrochen. Handys (< 640 px) bleiben wie bisher.
- **Seiten ziehen** (seit 1.65.0, `swipeDragStart/Move/End`): die neue Seite folgt dem Finger und schiebt sich über die alte
  (`.swipe-top`/`.swipe-under`), zurück zum Lebensrad wird die Seite weggezogen; ab ¼ Breite oder schnellem Wisch rastet sie ein.
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
  Von wachsenden Trophäen (`group`) steht nur die höchste Stufe im Regal (`trophyShelf`). Seit 1.67.0 zählt für Trophäen alles höchstens einmal am Tag (`oncePerDay`; Übungen je Übung und Tag). Seit 1.66.0 Knopf „Ins Tagebuch“ in der großen Trophäen-Ansicht (`trophyToDiary`: Bild + Datum, Titel, Text); im
  Tagebuch stehen Trophäen seit 1.74.2 auf einer kleinen nachtblauen Karte mit Goldrand (`tbIsTrophyImg`, `.is-trophy`),
  weil die Freisteller (Glas, Glanz) auf weißem Papier dunkle Ränder zeigen.

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
- `AREAS = ['gesundheit','beziehungen','finanzen','berufung','sinn','freude']` (Freude seit 1.77.0)
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

## Seit 1.78.0
- Startseite: Inhalt scrollt unter dem Rahmen (`body::before` z-index 60 + dunkles Band `.home-veil` z 59), Rad ohne eigenen
  Hintergrund. Neuer Regler-Knopf = Bild `img/knob-gold.webp` (Faltpapier-Goldkugel, ersetzt das SVG); Wunsch-Linie im Rad in
  `KNOB_GOLD` mit diesen Knöpfen an den Ecken.
- Symbole Leiste für 6 Bereiche: `ico-wheel` (Sechseck), `ico-areas` (6 Rauten) – vorläufig selbst gezeichnet; Freude-Symbol
  = Sonne des Nutzers.
- Bilder des Nutzers: Adventskranz `advent-1…4` (rote Kerzen), Baum `stickers/tree`, Stern `trophy-star`, Krippe mit Hund
  `stickers/crib-dog`, liegender Hund `stickers/dog-lying`, Vitrine `vitrine.webp`, Freude-Trophäe `orb-sun`.
- Hundeplatz am Regal: 24.12.–6.1. steht dort die Krippe (größer, der Hund liegt darin), am Wochenende der liegende Hund, sonst
  der sitzende. Krippe ohne Hund als Trophäe (`trophy-crib`) fehlt noch.
- Vitrine ab Stufe 6 (ein Jahr): neben (breit) bzw. unter (Handy) dem Regal, Feiertags-Trophäen (Ostereier, Nikolaus, Stiefel,
  Stern, Krippe) stehen darin; Fächer in % von vitrine.webp.
- Freude-Trophäe (`joy10/20/30`, Gruppe `joy`): wächst, wenn Freude seit dem ersten Wert um 10/20/30 Punkte gestiegen ist –
  ein hoher Startwert allein zählt nicht.

- Seit 1.78.1: auf allen Seiten liegt der Rahmen über dem Inhalt (`::before` z 3) mit dunklem Band darunter (`::after` z 2),
  Inhalt z 1. Gold-Knopf: Box Breathing r 9,75, im Lebensrad 17 px. Kartenbilder des Nutzers: Advent `cards/adv01…23`
  (Adventssonntage weiter mit Kranz), Feiertage `cards/h-<Schlüssel>` (`HOLIDAY_PICS`), Freude `cards/fre1…8`.

- Seit 1.78.2: Krippe ohne Hund `trophy-crib.webp` (Trophäe `xmas2`); Freude-Trophäe in drei eigenen Bildern `orb-sun-1…3`;
  neue Freude-Kartenbilder; mit Vitrine (Stufe 6) stehen die Tagebücher früherer Jahre im obersten Vitrinenfach, sobald es
  mindestens zwei sind (Trophäen dann in den drei Fächern darunter).

- Seit 1.79.0: Leisten-Symbole `ico-wheel`/`ico-areas` sind jetzt Bilder des Nutzers (Sechseck, 6 Rauten). Seite Bereiche:
  aufklappbare Karte „Warum ein Lebensrad?“ (`whyText`, beim ersten Rad offen) und je Bereich „Mehr lesen“ (`areaMore`:
  Was dazugehört, Woran du einen hohen Wert erkennst, Ein kleiner erster Schritt).

## Zwei Designs: „A Gentleman's Study“ und „A Mystery World“ (seit 1.80.0)
- Wahl je Gerät in `localStorage` `ewolfe_design` = `biz` (A Gentleman's Study) oder `classic` (A Mystery World, das bisherige
  Gold-/Papier-Design). Ein kleines Skript im `<head>` setzt vor dem Zeichnen `html.theme-biz` und `window.EW_DESIGN`;
  im Code `BIZ`. Neues Gerät → Business, Gerät mit Daten (`ewolfe_data_v1`) → classic (Bestandsnutzer behalten ihr Design).
- Erste Maske: Sprache und Design wählen, dann „Weiter“ (`firstPickSet`/`firstPickGo`, Vorschau sofort). Einstellungen: Abschnitt
  „Design“ (`designBtns`, `setDesign` speichert und lädt die App neu).
- A Gentleman's Study: Navy/Dunkelgrün, Akzent Cognac #9a6b3f, Text Leinen #dcd4b8, Überschriften und Knöpfe Noto Serif Display
  schmal (wdth 62.5) in Großbuchstaben, keine Rahmen, schlichte Knöpfe. CSS-Block „Design Business“ am Ende des CSS (`:root.theme-biz`).
- Symbole: Linien-SVGs in `img/icons-rl/ico-*.svg` (gleiche Namen wie `img/icons/ico-*.webp`); ein MutationObserver tauscht in BIZ
  jede `img/icons/ico-…webp` gegen die SVG (`designSwapIn`), auch in später gezeichnetem Inhalt. Auf cremefarbenem Papier dunkel gefiltert.
- Rad: Ist in Leinen, Wunsch Cognac mit Cognac-Punkten (`wishKnob`), Lücke Cognac → Grün (`GAP_*`, `IST_COL`, `COMPARE_COLORS`).
- Regal: Nussholzregal `img/rl/shelf.webp` (zugeschnitten 552×1254, `SHELVES.rl` mit `aspect`, `rowCap` rechnet mit dem Seitenverhältnis),
  Vitrine `img/rl/vitrine.webp` (348×1000). Keine Tiere/Feiertagsschmuck (`decorOn()` false, Schalter ausgeblendet).
- Trophäen: gleiche Regeln, andere Bilder in `img/rl/` über `trophyRl()`/`RL_TRO`: Übungen = Medaillen (6 Bänder), Tage in Folge =
  Kristall-Obelisken 1–6, je Übung Glasglocke (lotus, cherry, lemon, stones, bowl, hearts), Lebensräder = Steuerrad, Ziele =
  Silberpokal, Tagebuchseiten = Bücher mit Füller, Treue = Taschenuhr, Freude = Sonnenuhr, +10 = Fernrohr, alle Bereiche = großer
  Pokal, Ostern = Eier (creme, kobalt, grün; Titel „Osterei“), Nikolaus = Holzfigur, Stiefel, Stern. Krippe noch die alte.
- Seit 1.80.1: Name „A Gentleman's Study“. App-Symbol = Monogramm „EW“ (Didot, Leinen auf Navy, Cognac-Linie, `img/rl/app-icon.png`,
  `img/rl/apple-touch-icon.png`, im `<head>`-Skript gesetzt). Stufen = Club-Ränge (de seit 1.80.2: Gast, Anwärter, Mitglied, Ratsmitglied, Vorstand,
  Ehrenmitglied; en: Guest, Candidate, Member, Fellow, Senior Fellow, Honorary Member; `clubRanks`, `clubUp`, `clubProgressText`) auf einem gravierten Messingschild am Regalsockel (`clubPlaqueHtml`,
  `.club-plaque`, je Rang eine Raute). Favicon und Manifest-Symbol (Data-URL) sind noch das alte.
- Seit 1.80.2: Alles, was im neuen Design noch ein altes Bild zeigt, trägt einen roten Schriftzug „Platzhalter – neues Design folgt“
  (CSS `::after` im Block „A Gentleman's Study“, Sprache über `html[lang]`, das `applyLang()` setzt): Übungskarten, Atem-/Player-Bilder,
  Dankbarkeit, Impulskarten, Tagebuch-Cover und Titelseite, Coverwahl, Buchrücken, Sticker-Menü, Trophäen ohne `rl/`-Bild (`rlMissing`).
  Wenn ein neues Bild kommt, die Klasse dort aus der Liste nehmen. Glasglocken sollen wieder drei Stufen bekommen (Prompts bestellt).
- Seit 1.80.4: Das Messingschild sitzt nur auf dem vorderen Sockelbrett (left 23,3 %, top 93,1 %, 49,8 × 3,1 % des Regalbilds,
  vom Nutzer markiert), Rang und Rauten in einer Zeile. Noch per CSS gezeichnet; ein Bild der leeren Plakette ist bestellt
  (dann als Hintergrund von `.club-plaque`). Achtung: `cqh` am Schild selbst misst das Regal, nicht das Schild.
- Seit 1.80.3: Unser Hund gehört zu jedem Design. In A Gentleman's Study sitzt er (am Wochenende liegt er) ab 15 aktiven Tagen rechts
  vor dem Nussholzregal (`bizDog`); bis `img/rl/dog.webp`/`dog-lying.webp` da sind, das alte Bild mit rotem Hinweis (`RL_DOG_READY = false`).
  Sonst keine Tiere und kein Feiertagsschmuck in diesem Design.
- Noch im alten Stil (Bilder folgen): Tagebuch-Cover und Buchrücken, Übungsbilder, Impulskarten, Sticker,
  Box-Breathing-Punkt. Neue Sonnenuhr und Stiefel sind bestellt (Prompts 6b/21b im Sammelordner).

## Bibliothek seit 1.81.0 (beide Designs gleich – Grundsatz: die Funktionen sind in beiden Designs immer gleich, nur die Bilder unterscheiden sich)
- Die Bibliothek ist eine Zeile, die sich seitlich scrollen lässt (`.lib-wrap` > `.lib-scroll` > `.lib-row`): links das Regal,
  rechts später die Vitrine (Stufe 6) und weitere Gegenstände. Pfeil „›“ am rechten Rand, solange es mehr zu sehen gibt (`libScrollHint`).
  Breiten über `--sw` (Breite des großen Regals) an `.lib-row`; Vitrine `--sw × 0,5` (Business × 0,72).
- Schmales Regal (`NARROW`, Bilder `shelf-narrow`, `shelf-clean-narrow`, `shelf-dusty-narrow`, `rl/shelf-narrow`, aus den großen Regalen
  geschnitten, Mitte herausgenommen): erscheint links angebaut, sobald die Tagebücher nicht mehr ins oberste Fach passen. Dann stehen
  alle Tagebücher darin (4–5 je Fach), das große Regal trägt nur Trophäen. `k` = Breite relativ zum großen Regal, `ov` = Überlappung.
- Neue Trophäen in beiden Designs: „Angekommen“ (`arrived`, Flugzeug: in einem Bereich erreicht der Ist-Wert den Wunsch, nachdem er
  darunter lag) und „Ein Klassiker“ (`classic12`, Oldtimer 300 SL: Lebensräder in 12 verschiedenen Monaten). Bilder fehlen noch
  (`TRO_MISSING`): Ersatzbild mit rotem Hinweis `.ph-note` „Platzhalter – neues Bild folgt“ (in beiden Designs).
- Rang-Schild: lange Namen werden kleiner gesetzt (`fit`). Alle offenen Prompts: Sammelordner „2 RL Style/A Grafiken/Offene Prompts …“.
- Seit 1.82.0 Bilder des Nutzers in A Gentleman's Study: Lederhefte als Cover (`img/rl/covers/<farbe>-p|l.webp`, `RL_COVER` ordnet die
  fünf Cover-Plätze zu: navy→navy, blue→charcoal, green→green, violet→cognac, red→burgundy; aus ChatGPT-Kacheln zugeschnitten und
  hochskaliert), Buchrücken `rl/spine-*`, Glasglocken in drei Stufen (`cloche-<x>-1…3`, Box Breathing noch eine Glocke), Bücherstapel
  1–4 (`rl/books-1…4`, Maßstab hält ein Buch gleich hoch), neue Sonnenuhr, Stiefel, Krippe, Flugzeug, Oldtimer, Hund sitzend/liegend,
  Messingplakette als Bild (`rl/plaque.webp`), Monogramm von ChatGPT als App-Symbol. A Mystery World: `trophy-plane`, `trophy-car`.
  Prompt für ein schlankeres Papier-Regal (Maße wie das Nussholzregal) in „Offene Prompts …“ Teil D.
- Seit 1.81.2: A Mystery World bekommt statt der Kugeln dieselben Gegenstände wie A Gentleman's Study im goldenen Papierstil
  (`CLASSIC_NEW`: Steuerrad, Pokal, Taschenuhr, Fernrohr, großer Pokal, wachsende Bücherstapel 1–4). Ein Bild ersetzt seine
  Kugel erst, wenn sein Dateiname in `CLASSIC_READY` steht; Bücherstapel fürs Business-Design ebenso über `RL_BOOKS`/`RL_READY`.
  Die Sonne im Orb (Freude) bleibt. Kristalle, Glasglocken, Eier, Nikolaus, Stiefel, Stern, Krippe bleiben. Prompts: „Offene Prompts …“ Teil C (Nr. 25–31).
- Seit 1.81.1 statt Cognac Bronze (mehr Kontrast): auf dunklem Grund Schrift helle Bronze #d9b679 (`--gold-text`), Linien/Akzente
  Bronze #c9a463 (`--gold`, Wunschlinie, Regler); auf cremefarbenem Papier (Ziele, Tagebuch-Bereichsköpfe) #ab8b54 (Bronze 15 % dunkler).

## Seit 1.83.0
- **Impulskarten in A Gentleman's Study:** kleine englische Gemälde (Constable/Gainsborough/Stubbs-Stil, Bilder des Nutzers) in
  `img/rl/cards/<id>.webp` (48 Impulse, `advSun1…4`, `adv01…20`), auf cremefarbenem Büttenpapier mit Bronze-Linien
  (`img/rl/card-front.webp`, Fenster für das Bild, `CARD_WIN`) und navyfarbener Rückseite mit EW-Monogramm (`img/rl/card-back.webp`).
  `RL_CARDS` listet die vorhandenen; `cardImg()`/`rlCardName()`; seit 1.83.1 alle 92 Karten vorhanden (Advent 1–24, 4 Adventssonntage,
  xmas1/xmas2 = Stall mit Stern wie adv24, 13 Feiertage); fehlende würden das alte Bild mit rotem Hinweis zeigen (`.imp-win.ph-note`). „Ins Tagebuch“ zeichnet die Karte im selben Stil (`impulseCardImage`).
- **A Mystery World: schlanke Papier-Regale** (Bilder des Nutzers, Maße wie das Nussholzregal): `shelf-slim-clean|dusty|ivy.webp`,
  Anbau-Regal aus dem sauberen geschnitten (`shelf-slim-clean-narrow`, `shelf-slim-dusty-narrow`; Efeu nur am breiten Regal),
  neue Vitrine `vitrine-slim.webp` mit fünf Glasfächern. Tiere/Schmuck neu platziert (`html:not(.theme-biz) .shelf-…`).
  Staub-Regal jetzt die ersten zwei aktiven Tage (`appActiveDays() < 3`).
- **A Gentleman's Study:** Hund deutlich größer vor dem Regal (`.shelf-dog` height 34 %).
- **Seit 1.86.4 eigene Achtsamkeits-Bilder in A Gentleman's Study** (`img/rl/medi/`: lotus, cherry, lemon, stones, hearts, bowl,
  bell, blanket; vom Navy-Grund freigestellt, Glasvase durchsichtig, Skript im Sammelordner „Freigestellt/medi“): `imgPic()` tauscht
  in BIZ über `RL_MEDI` (Karten, Lemon-Player, Stille, Dankbarkeit); eigene Übungen zeigen in BIZ die Messingglocke. Roter Hinweis
  bleibt nur auf der Atem-Animation (`.breath-stage`) und den Stickern; in A Lady's Conservatory weiter auf allem Geliehenen.
  Seit 1.86.5 auch in Box Breathing (Lotus) und 4-7-8 (Kirschzweig in der Vase, ohne Himmel-Ausschnitt): das Bild wird beim
  Einatmen sanft größer (`breathLotusSet`, Faktor 0,9–1,04 von `data-base`), kein roter Hinweis mehr in AGS.
  Seit 1.86.6 eigene Sticker in AGS (`img/rl/stickers/<key>.webp`, 17 Stück, `RL_STICKER`, `stickerPath()`; Hund = `rl/dog`);
  nur der Hundekopf hat noch das alte Bild mit rotem Hinweis (`.tb-sticker-btn.st-old`). In A Lady's Conservatory alle Sticker mit Hinweis.
  **Schneiderpuppe im Reitfrack** (`img/rl/dress-form.webp`, `.lib-dress`) steht in AGS rechts neben dem Regal (immer); die Medaillen
  Seit 1.86.8 hängen die Übungs-Medaillen als Orden auf der linken Brust des Fracks (`.dress-medals`, alle erreichten Stufen,
  `DRESS_ALL_MEDALS = true`; false = nur die höchste) und stehen nicht mehr im Regal. Fernrohr und Silberpokal stehen etwas weiter hinten
  (`TRO_LIFT`). Offen für AGS: Hundekopf-Sticker, Favicon/Manifest-Symbol.
- **A Mystery World hat seit 1.86.8 auch eine Plakette** (ganz gold, aus `btn-main.webp` umgefärbt: `img/plaque-gold.webp`) **unter** dem
  Regal (auf dem Sockel sitzt schon die Goldraute), mit Rängen des wachsenden Glanzes (`glowRanks`: Funke, Glut, Flamme, Stern, Komet, Sonne).
  `rankNames(d)` wählt die Ränge je Design.
- **Seit 1.86.9 die Papier-Gegenstände statt Kugeln in A Mystery World** (`CLASSIC_READY` voll): Steuerrad, Pokal, Taschenuhr, Fernrohr,
  großer Pokal, Bücherstapel 1–4 (`img/trophy-*.webp`, goldener Low-Poly-Stil mit Funkeln, von Navy freigestellt; Stapel mit eigener
  Skalierung je Stufe). Flugzeug und Oldtimer gab es schon. Originale: „1 Harry Potter Style/A Grafiken/Originale ChatGPT“.
- **Seit 1.87.0 Bibliothek-Knopf = das eigene Regal des Designs mit Büchern** (`img/shelf-icon.webp` A Mystery World, `img/rl/shelf-icon.webp`,
  `img/rt/shelf-icon.webp`; aus den Regalbildern mit eingesetzten Buchrücken gerechnet, Skript `shelficon.py` im Scratchpad), 40 px hoch,
  ohne Kasten (`.tb-lib-shelf`). Lady-Cover (`rt/covers/*`) neu freigestellt: grauer Schattenrand rechts/unten weggeschnitten, gerade Kanten.
- Seit 1.87.1: Wer auf dem Handy (kürzere Seite < 600 px) beim neuen Tagebuch „Querformat“ wählt, bekommt eine deutliche Rückfrage
  (`diaryLandPhoneWarn`: Seite wird sehr klein, Format gilt fürs ganze Buch; Knöpfe Hochformat / Trotzdem Querformat / Abbrechen).
- **Seit 1.87.2 sind die Regale in allen drei Designs gleich hoch** (Höhe = Fensterhöhe − Leiste − 190 px; `--sw` = Höhe × `--sa`,
  das Seitenverhältnis des Regalbilds, das `tbRenderLibrary` an `.lib-row` setzt). Oben nur 24 px Rand, unten Platz für den Hund
  (`--sw / --sa × 0,07`). Die Katze sitzt nicht mehr oben auf dem Regal von A Mystery World (bekommt später einen anderen Platz); der Hund
  steht dort näher vor dem Regal. A Lady's Conservatory: unten wieder Boden zu sehen (`padding-bottom` `--sw`×0,15, Hintergrund y
  `100% + --sw×0,265`); passt das Orchideen-Tischchen auf dem Handy nicht daneben, beginnt die Zeile beim Regal (Hund ganz sichtbar),
  das Tischchen schaut links herein (Skript in `tbRenderLibrary`, setzt `scrollLeft` und `scroll-margin-left`). Seitlich scrollen bleibt überall.
- **Seit 1.87.3:** Kontrast in A Gentleman's Study 20 % stärker (`:root.theme-biz:not(.theme-retreat)`: Leinen #ece7d8, Bronze #d3b681,
  helle Bronze #e3c99d; WCAG gegen #0c1226 × 1,2). Ist-Punkte im großen Lebensrad haben die Größe der Wunsch-Knöpfe (`ew-knob`, `data-scale` 0,8,
  weil das Knopfbild einen Rand hat); im kleinen Rad auf „Bereiche“ sind die Wunsch-Knöpfe kleiner (`wishKnob(…, 0.7)`). Englisch: die zwei
  Beispiel-Übungen (`SEED_TEXT`, `mediTitle`/`mediNotes`), „Lemon meditation“ (`exLemonTitle`), „Icon“. Bibliothek in allen Designs so breit wie das
  Gewächshaus (4,78 `--sw`), Regal anfangs mittig. **Trophäen nach Perspektive:** die Bilder sind leicht von oben aufgenommen; über Augenhöhe
  (Fächer über dem mittleren, neben den Büchern, auf dem Regal) stehen nur flache/frontale Stücke (`TRO_FRONT`); jede Trophäe ist in jedem Fach gleich
  groß (Höhe relativ zum mittleren Fach), was zu hoch ist, kommt in ein anderes Fach; erst etwas kleiner (bis 86 %), dann auch über Augenhöhe, erst
  zuletzt aufs Regal (`place()` in `tbRenderLibrary`).
- Seit 1.83.2: neue Zitronen-Glocken (Zitronenzweig: Blüten → grüne Frucht → gelbe Zitrone), schmaler (Seitenverhältnis 0,70). Übungsbilder für die Achtsamkeit (Lotus, Kirschzweig, Zitrone, Steine, Herzen, Klangschale, Glocke, Decke) liegen im Sammelordner unter „Originale ChatGPT/Achtsamkeit“, noch nicht eingebaut.

## Drittes Design „A Lady's Conservatory“ (seit 1.84.0, bis 1.86.0 „A Morning Retreat“)
- **Name seit 1.86.1: „A Lady's Conservatory“** (Wahl des Nutzers, Gegenstück zu „A Gentleman's Study“). Intern bleiben der Schlüssel
  `ewolfe_design` = `retreat`, `RETREAT`, `theme-retreat` und der Ordner `img/rt/` – so behalten alle Geräte ihre Wahl.
- `ewolfe_design` = `retreat`. Das `<head>`-Skript setzt `html.theme-biz` **und** `html.theme-retreat`; im Code `RETREAT` und
  `BIZ` (BIZ ist auch hier wahr: bis eigene Bilder kommen, nutzt es alle Bilder und Funktionen von A Gentleman's Study,
  mit rotem Platzhalter-Hinweis überall: Trophäen über `rlMissing`, Regal, Vitrine, Cover, Titelseite, Impulskarten per CSS).
- Hell: Beige #efe6d8, Karten #f8f2e8, Salbei #7f9a80 (Knöpfe, helle Schrift), Salbei dunkel #5a7760 (Unterüberschriften, Ist-Linie),
  Sand #c8b08a (Wunsch, Rahmen), Schrift #3d3a35. Überschriften Cormorant Garamond, Text Jost, runde Knöpfe. CSS-Block
  „Design A Morning Retreat“ am Ende des CSS; Rad-Farben über `RETREAT ? … : BIZ ? … : …` (IST_COL, GAP_*, KNOB_GOLD, Gitter).
- Linien-Symbole sind die von A Gentleman's Study, per `filter:brightness(0.38)` dunkel.
- Design-Auswahl: drei Knöpfe untereinander (`DESIGNS`, `DESIGN_SUB`); Wechsel lädt neu, wenn sich `window.EW_DESIGN` ändert.
- Seit 1.84.1: Regal, Anbau-Regal und Vitrine sind die Bilder von A Gentleman's Study, hell umgefärbt (Eiche, `img/rt/shelf`, `rt/shelf-narrow`,
  `rt/vitrine`; gleiche Maße und Fächer, `SHELVES.rl.img`/`NARROW.rl.img` wählen per `RETREAT`). Umfärben: Helligkeit je Pixel mit
  Gamma 0,9 auf einen Holz-Verlauf gelegt (Skript im Scratchpad `lighten.py`; seit 1.84.2 Variante „gekalkt“, passend zum Tischchen).
- Seit 1.84.2: ab Stufe 3 (wo A Mystery World den Efeu bekommt) steht links vor dem Regal ein Pflanzentisch mit weißer Orchidee
  (`img/rt/orchid-stand.webp`, `.shelf-orchid` in `.shelf-unit`, `.lib-row.has-orchid` macht links Platz, `scroll-margin-left` damit
  das Einrasten ihn nicht verdeckt).
- Prompts und Entscheidungen: Sammelordner „3 Morning Retreat Style“.
- **Seit 1.86.2 eigene Möbel und das Gewächshaus:** Regal `img/rt/shelf.webp` (cremefarben, Rückwand Salbei, Efeu links; 681×1379,
  Fächer `x:[16.7, 90.6]`, Reihen gemessen), Anbau-Regal `rt/shelf-narrow` (aus der rechten Hälfte gespiegelt, ohne Efeu, `k` 400/681,
  `ov` 0,12, weil links 9 % Efeu-Rand sind), Vitrine `rt/vitrine` (Messing und Glas mit Farnen, 518×1304, Glas per Farbe-zu-Alpha
  gegen das Grau durchsichtig; Fächer `VX [13, 87]`, vier Reihen über den Farnen; Breite `--sw`×0,76), Messingschild auf dem
  cremefarbenen Sockel. Hintergrund der Bibliothek `rt/conservatory.webp` („Gewächshaus 4“): `.lib-row` ist so breit wie das Bild
  (4,78 `--sw`), Wand hinter den Möbeln, Bodenlinie an ihren Füßen, rechts scrollt man zu den offenen Türen in den Wald.
  Freistell-Skript und Bilder: Sammelordner „Fertige Bilder/Gewächshaus (1.86.2)“. Trophäen sind noch die von A Gentleman's Study.
- **Seit 1.86.3:** Steht das Anbau-Regal da, wandert der Efeu dorthin: `rt/shelf-narrow` = linker Teil des Efeu-Regals (bis x 300)
  plus ein Stück rechts davon ohne eigenen rechten Pfosten (Füßchen dort entfernt); das große Regal wird `SHELVES.rtPlain`
  (`rt/shelf-plain`, aus seiner rechten Hälfte gespiegelt, ohne Efeu, 630×1379, `x:[10, 89.8]`). Beide teilen sich eine
  Zwischenwand (den linken Pfosten des großen), `ov` 41/630. Umschalten in `tbRenderLibrary` (`if(RETREAT && NAR) SH = SHELVES.rtPlain`).
  Seit 1.86.6 Vitrine 4 mit Glasdach und cremefarbener Rückwand (`rt/vitrine`, 386×1200, Glasböden 24,4/36,9/49,5/61,9 %,
  `VX [11, 89]`, Breite `--sw`×0,68) und links das **sechseckige eingelegte Tischchen mit Orchidee** (`rt/hex-table`, ab Stufe 3,
  ersetzt das alte Orchideen-Tischchen).
- Seit 1.86.7 stehen die Möbel weiter vorn auf dem Boden (`.lib-row` padding-bottom `--sw`×0,035). Alle fehlenden Bilder für
  A Lady's Conservatory (Hund nach dem AGS-Hund als Vorlage, Trophäen, Tagebuch-Cover, Achtsamkeit, Sticker, Impulskarten):
  „3 Morning Retreat Style/Alle offenen Prompts – A Lady's Conservatory.md“. „Regal 3 (schlicht, Sockel)“ liegt bereit, noch nicht eingebaut.
- **Seit 1.86.8 eigene Trophäen, Cover und Plakette in A Lady's Conservatory** (`img/rt/`, `RT_TRO`, `RT_BOOKS`, in `trophyRl` zuerst):
  Übungen = 6 Emaille-Broschen, Tage in Folge = 6 Rosenquarze, Glasglocken je Übung in 3 Stufen (auch Box Breathing), Lebensräder =
  Armillarsphäre, Ziele = Porzellanpokal, Treue = Anhängeruhr, Freude = Sonnenuhr mit Jasmin, +10 = Gießkanne, alle besser = Blumenschale,
  Angekommen = Ballon, Klassiker = Damenrad, Bücherstapel 1–4. Fehlend (Nikolaus, Stiefel, Stern, Krippe, Ostereier) = Bilder von
  A Gentleman's Study mit Hinweis (`rlMissing` prüft `rt/`). Cover/Buchrücken aus Leinen (`rt/covers/<farbe>-p|l`, `rt/spine-*`, `RL_COVER`
  zeigt in LC auf sage, ivory, forest, blush, celadon; forest/celadon von ChatGPT im falschen Format, auf 860×1204 bzw. 1000×750 gebracht).
  Eigene goldene Plakette `rt/plaque.webp` und **Damen-Ränge** (`ladyRanks`: Gast, Gärtnerin, Botanikerin, Kuratorin, Patronin, Ehrenpatronin;
  `ladyUp`). Freistell-Skript: Sammelordner „Fertige Bilder/Trophäen und Cover (1.86.8)“.
- **Seit 1.86.0 neue Richtung „Gewächshaus am Wald“** (Nutzer: edel, vintage, verziert und schlicht, Pflanzen, ladylike in Grün
  zwischen Wellness und Waldbaden, mehr Weiß/Creme, Bibliothek in einem Gewächshaus, das sich in den Wald öffnet; Lieblingsstück
  sechseckiges eingelegtes Beistelltischchen). Farben Variante A „Gewächshaus am Morgen“: Grund Creme #f7f3ea (nach unten leicht
  grünlich bis #e8eedf), Karten #fffdf8, Schrift #262622, gedämpft #625e55, Waldgrün #2f4f3e (Knöpfe, Ist-Linie, aktive Leiste),
  Grün #3f6150 (Unterüberschriften), Salbei #8ea88a, Messing #b08d57 (Wunsch, Rahmen), Messing-Schrift #86683a. Lücke im Rad
  Salbei → helles Messing-Creme. Kontraste alle über der 30-%-Regel von 1.84.3. Bilder (Gewächshaus-Hintergrund, Regal, Wardscher
  Kasten als Vitrine, sechseckiges Tischchen, Pouf, Rattan, Terrarium, Teppich): Prompts „Bild-Prompts Morning Retreat – Gewächshaus.md“,
  Inspiration „Inspiration Gewächshaus“, Farbvergleich „Farbvergleich Gewächshaus.html“. Name seit 1.86.1 „A Lady's Conservatory“.
- Seit 1.84.3 Kontrast mindestens 30 % stärker (Wunsch des Nutzers; WCAG-Kontrast gegen Beige #efe6d8 je Farbe × 1,3, Farbton bleibt,
  nur dunkler): Schrift #292723, gedämpft #666058, Salbei #69856a, Salbei dunkel #4b6350, Sand #b69664, Sand dunkel #826a45,
  Knopf #a58552, Vergleich #a7734f, Platzhalter #90877a; feine Linien rgba(41,39,35,…) mit höherer Deckkraft (0,13 → 0,25,
  0,18 → 0,29 usw.); Linien-Symbole `brightness(0.26)` statt 0,38. Hintergründe unverändert. Neue Bilder aus den Prompts
  werden erst eingebaut, wenn sie da sind.

## Seit 1.88.0: neue Farben für A Lady's Conservatory, Vitrinen so hoch wie Regale, schmales Regal immer
- **Neue Bilder in `img/rt2/`** (die alten in `img/rt/` bleiben für Armillarsphäre, Glasglocken, Plakette, Cover-Reste). Originale und Freisteller:
  Sammelordner „3 Morning Retreat Style/Originale ChatGPT/Neue Farben (21.10.2026)“ und „Fertige Bilder/Neue Farben (21.10.2026)“.
- **Farben „bunter“** (`:root.theme-biz.theme-retreat`): Knöpfe/Ist-Linie Waldgrün #1B4D2A, Überschriften Burgund #7A0B25 (`--rt-burgundy`,
  Regel `html.theme-retreat.theme-retreat :is(.modal-title, …)`), Unterüberschriften #B81F57 (`--rt-sage-d`), Wunsch/Regler/aktive Leiste
  Himbeere #D62C68 (`--gold`, `KNOB_GOLD`, Knopf `img/rt2/knob.webp`), Seladon #79B8A7 (`--rt-celadon`, Lücke im Rad), Grund Creme mit Rosé-Hauch.
- **Bibliothek:** Himbeer-Regal `rt2/shelf` (454×1212, `x:[12, 86.5]`), schmales Regal `rt2/shelf-narrow` (Mitte herausgenommen, `ov` 0,1 = eine
  gemeinsame Wand), Vitrine `rt2/vitrine` (435×1192, fünf Fächer `VX [21, 79]`), Hintergrund `rt2/conservatory` (grüne Wand, roter Teppich; nur
  540 px breit, deshalb unscharf – neues Bild bestellt). Hintergrund in Regalhöhen gemessen (`--lh` = `--sw / --sa`). Regal und Vitrine dicht
  beieinander (`gap` `--lh`×0,012). Rosen-Tischchen `rt2/rose-table` steht vorn vor der linken Regalkante (`.shelf-orchid`, z-index 4).
  Bibliothek-Symbol `rt2/shelf-icon`. Hund gemalt `rt2/dog`, `rt2/dog-lying` (Sticker-Kopf `rt2/dog-head` liegt bereit).
- **Trophäen stehen richtig** (nur LC): Reihen enden auf der Mitte des Bodens (`rows` …23,6 / 40,9 / 56,8 / 73,4 / 90,2), kein Versatz mehr
  (`transform:none`). Über Augenhöhe (Fächer 1 und 2) liegt die Vorderkante des Bodens als Bild darüber (`.shelf-fronts`, `rt2/shelf-fronts`,
  `rt2/shelf-narrow-fronts`) und verdeckt ein Stück vom Fuß. Vitrine: Füße genau auf dem Glas (`VR0` …21,9 / 35,5 / 49,1 / 62,5 / 76).
- **Trophäen-Zuordnung LC:** Freude = Fahrrad, Ein Klassiker = Auto, Erste Dankbarkeit = Rosenkarte, Angekommen = Segelboot, +10 = Kerzenleuchter,
  Ziele = roter Pokal, Treue = Uhr, alle besser = Blumenschale, Nikolaus/Stiefel/Stern/Ostereier neu (`RT_TRO`, `RT_EGGS`, `trophyRl`).
  Broschen und Uhr klein (kommen später auf die Büste). Glasglocken passen nicht mehr: roter Hinweis (`rlMissing` erkennt nur `rt2/` und die
  Armillarsphäre als fertig). Krippe fehlt (Prompt K2 mit heiliger Familie).
- **Achtsamkeit LC:** eigene Bilder `rt2/medi/*` (`RT_MEDI` in `imgPic`, Atem-Animation, Glocke für eigene Übungen), kein roter Hinweis mehr.
- **Impulskarten LC:** 48 gemalte Szenen `rt2/cards/<id>` (`RT_CARDS`), Vorderseite `rt2/card-front` (Bildfenster oben durchsichtig),
  Rückseite `rt2/card-back`; Bild quer oben im Fenster (`CARD_WIN` mit `cover:true`, auch beim Übertragen ins Tagebuch), Text darunter.
  Advent und Feiertage fehlen noch (zeigen das alte Bild mit rotem Hinweis).
- **Cover LC:** `rt2/covers/<forest|celadon|sage|raspberry|burgundy>-p|l`, Rücken `rt2/spine-*` (`COVER_DIR` = `rt2`, `RL_COVER`). Nur kleine
  Kacheln geliefert, deshalb weich.
- **Alle Designs:** Jede Vitrine ist genau so hoch wie das Regal (Breite = `--sw / --sa` × Seitenverhältnis der Vitrine). Das schmale Regal für
  die Tagebücher steht immer da (`NAR = true`), das große trägt nur Trophäen.

## Seit 1.89.0: ein Regal pro Design (Bücher- und Trophäenteil), gleiche Vitrinen, Lady-Kleid
- **Neue Regale in allen drei Designs** (Bilder des Nutzers, gerade von vorn, Augenhöhe = mittleres Fach): je ein Möbel mit schmalem Bücherteil
  links und Trophäenteil rechts, an der Mittelwand geschnitten (`NARROW` = Bücherteil, `SHELVES` = Trophäenteil, `ov` 0, `k` = Breitenverhältnis).
  Dateien: A Mystery World `img/mw2/shelf(-ivy)(-narrow)`, A Gentleman's Study `img/rl2/shelf(-plant)(-narrow)` (ab Stufe 3 mit Monstera:
  `rlPlant`), A Lady's Conservatory `img/rt2/shelf(-narrow)`. Überhängende Pflanzen als eigenes Bild links am Bücherteil (`NARROW.*.plant`,
  `.shelf-plant`). Gemeinsame Fächer `SHELF_ROWS2`; die Reihen enden auf der Mitte der Böden. Über Augenhöhe liegt die Bodenkante vor den
  Trophäen (`*-fronts.webp`, `.shelf-fronts`, Bild über `SH.fronts`/`NS.fronts`). Das Bücherteil steht immer da (`NAR = true`).
  Ein Regal mit Spinnweben für A Mystery World fehlt noch (`dusty` zeigt vorerst das saubere).
- **Vitrinen** in allen Designs aus derselben Vorlage, gerade von vorn: `mw2/vitrine`, `rl2/vitrine`, `rt2/vitrine`; Fächer überall
  `VX [17.2, 83.9]`, `VR0` = fünf Glasböden; Höhe = Regalhöhe (Breite = `--sw / --sa` × Seitenverhältnis).
- **Plakette** in allen Designs auf dem Sockel des Trophäenteils (`.club-plaque` 19 % / 94,2 %).
- **A Lady's Conservatory:** scharfer Hintergrund `rt2/conservatory` (1536×1024); grünes besticktes Kleid mit Schuhen (`rt2/dress-shoes`,
  Schuhe im Maßstab Puppe ≈ 160 cm, Schuh ≈ 25 cm) statt der Schneiderpuppe; die Broschen (Übungen) sitzen als kleine Gruppe am Ausschnitt,
  die Uhr (Treue) an der Taille (`.dress-watch`), beide nicht mehr im Regal. Neue Glasglocken Zitrone, Kirsche, Klangschale (`rt2/cloche-*`);
  Lotus, Steine, Herzen fehlen noch. Ostereier auf Goldständern (`RT_EGGS`: Himbeere, Burgund, Rot nur mit Goldband; Grün folgt).
  Advent- und Feiertagskarten `rt2/cards/<adv01…24|advSun1…4|xmas1|xmas2|newyear|epiphany|palm|maundy|goodfri|holysat|easter|eastermon|martin>`
  (`RT_CARDS` = alle Dateien im Ordner); es fehlen ascension, pentecost, pentecostmon, harvest. Rosen-Tischchen ab Stufe 3 (wie der Efeu).
- **1.89.1:** Räume auch für A Gentleman's Study (`rl2/room`, Navy-Bibliothek mit Parkfenster) und A Mystery World (`mw2/room`, Zaubersalon mit
  Sternenfenster), gleiche Größe/Lage wie das Gewächshaus. Lady: Krippe mit heiliger Familie (`rt2/crib`), neue Armillarsphäre (`rt2/armillary`),
  fünf Eier auf Goldständern (kleiner: 0,42–0,46), Schuhe kleiner und tiefer vor dem Kleid.

## Seit 1.90.0
- A Mystery World: Regal mit Spinnweben (`mw2/shelf-dusty`, `-narrow`, `-fronts`) für die ersten zwei aktiven Tage.
- A Lady's Conservatory: Weihnachtsbaum im himbeerfarbenen Topf (`rt2/tree`) steht vom 24.12. bis 6.1. an der Stelle des Rosen-Tischchens
  (`xmasTree`, `.shelf-xmas`); sechste Cover-Farbe Rot (`TB_COVERS` hat in LC zusätzlich `crimson`, `RL_COVER.crimson = 'red'`);
  alle Glasglocken, Cover, Buchrücken, Advent- und Feiertagskarten in den neuen Farben. Kleid steht tiefer auf dem Teppich
  (`.lib-dress-wrap` margin-bottom `--dh`×−0,13).

## Seit 1.91.0
- A Lady's Conservatory: alle 19 Sticker eigene Bilder (`rt2/stickers/<key>`, Hund `rt2/dog`, Hundekopf `rt2/dog-head`; `stickerPath`), kein roter Hinweis mehr.
  Rose = pinke Rose mit Band, Musik = Stimmgabel mit Band (Wunsch des Nutzers).
- A Mystery World: Versuch, die Trophäen vom Navy-Rand zu befreien (1.91.0), gefiel dem Nutzer gar nicht und wurde in 1.91.1 komplett
  zurückgenommen (alte Bilder, nachtblaue Karte im Tagebuch bleibt). Nicht noch einmal per Skript versuchen; nur mit neuen Bildern vor Grau.

## Seit 1.92.1
- **Lebensfreude** statt „Freude“ (de; en „Joy in life“): Bereichsname, Sonne, Karte, Vergleich, Trophäentext. Die Frage (`joyFeelQ`)
  macht klar, dass es um das Leben geht, nicht um einen Tag („denk an die letzten Wochen“). Stimmung des Moments = Achtsamkeit.
- **Buchrücken:** Titel und Jahr stehen in einer senkrechten Zeile (`.shelf-key` steckt in `.shelf-title`), die Schrift wird
  kleiner, wenn die Zeile nicht passt (`--spine-fit` aus der Zeichenzahl, Hoch-/Querformat). Nichts wird mehr abgeschnitten.
- 1.92.2: Broschen am Lady-Kleid mittig unter der Spitze des Ausschnitts (`html.theme-retreat .dress-medals` left 41,5 %, top 17,8 %),
  nicht mehr auf der Brust. Prompt für sechs neue, kostbarere Broschen: „3 Morning Retreat Style/Broschen – Prompts (24.10.2026).md“.
- 1.92.3: keine Tagebücher mehr in der Vitrine (alle im Bücherteil, `oldKeys = []`); Mystery: Fuchs vor dem Bücherteil (`left:-74%`),
  nie vor dem Hund; Lady: Rosentischchen/Weihnachtsbaum links neben dem Regal (`translateX(-98%)`, Zeile `padding-left` 0,46 `--lh`,
  Raum `background-position-x` mitgeschoben); AGS: Medaillen 20 % kleiner, linker Rand bleibt, untere Reihe halb unter der oberen
  (`nth-child(n+4)` margin-top), obere linke per `clip-path` am Revers abgeschnitten; AGS-Trophäen im Tagebuch auf Navy-Karte mit
  Bronzerand (`tbIsRlTrophyImg`, `.is-trophy-rl`), weil die Freisteller Navy-Ränder haben.
- 1.92.4: Lady-Regal, Bücherteil und Vitrine (`rt2/shelf`, `rt2/shelf-narrow`, `rt2/vitrine`) hatten halb durchsichtige Rückwände
  (Rest vom Freistellen) – durch die Löcher schien die grüne Wand als helle Bögen. Löcher geschlossen (Alpha 255 im Inneren), Regal-Rückwände
  zusätzlich glatt nachgerechnet. Andere Designs geprüft: dort nur Randpixel. Neue Bilder nach dem Freistellen immer vor einer
  farbigen Fläche prüfen.
- **1.93.0 Lady-Kleider und Broschen:** bis Stufe 6 das weite Kleid (`rt2/dress-shoes`), ab einem Jahr das schmale Smaragd-Kleid
  (`rt2/dress-emerald`), jedes Jahr 24.12.–1.1. das Seladon-Kleid mit Goldstickerei (`rt2/dress-celadon`, `isXmasDressTime`); Klasse
  `dress-<art>` an `.lib-dress-wrap` setzt die Lage der Brosche (`--bx`/`--by`, waagerecht mittig im Ausschnitt). Nur **eine** Brosche
  (`.dress-brooch`): die in der Trophäen-Ansicht gewählte (`broochPickHtml`, `broochWear`, je Gerät `state.settings.brooch`), sonst die
  kostbarste erreichte (`broochWorn`). Neue Broschen `rt2/brooch-n1…6` (neue Namen gegen alte Zwischenspeicher; die alten `brooch-1…6`
  bleiben für Tagebuchseiten liegen). Medaillen in A Gentleman's Study unverändert alle am Frack.
- 1.93.1: Die Treue-Uhr (60/150/250 aktive Tage) hängt nicht mehr am Lady-Kleid, sie steht im Regal; eine kleine Kaminuhr ist bestellt
  (Prompt in „3 Morning Retreat Style/Alle offenen Prompts – A Lady's Conservatory.md“), dann `RT_TRO.loyal` umstellen.
- 1.93.2: Treue in A Lady's Conservatory = himbeerrote Kaminuhr mit Rosen und Gold (`rt2/clock`, `RT_TRO.loyal`), wächst je Stufe etwas (0,62/0,70/0,78). Original: „3 Morning Retreat Style/Originale ChatGPT/Kaminuhr (24.10.2026)“.
- **1.94.0 Regal mit angebauter Vitrine (ab Stufe 6, alle Designs):** ein Möbel (Bilder des Nutzers „… – Regal mit Vitrine (neu)“), an den
  Mittelwänden in drei Bilder geschnitten: `mw2|rl2|rt2/sv-narrow`, `sv-shelf`, `sv-vitrine` (+ `sv-narrow-fronts`/`sv-shelf-fronts`: Bodenkanten über
  Augenhöhe und die Pflanzen Efeu/Monstera, die vor dem Bücherteil hängen). Daten in `SHELF_SV` (Fächer, `VX`/`VR` der Vitrine, Plakette `plaque`
  [top, Höhe] → CSS `--plt`/`--plh`). Die Vitrine steckt dann in `.shelf-unit` (`.vit-joined`, ohne Lücke, gleiche Höhe). Hund und Mystery-Krippe stehen
  vor der Vitrine, nicht vor der Plakette. Skripte (Freistellen vom Grau, Schneiden, Messen): Sammelordner „Bibliothek nach 1, 2, 3 Jahren (24.10.2026)“.
  Neue Weihnachtsbäume (`mw2/tree`, `rl2/tree`, `rt2/tree`; schmiedeeiserner Ständer, alter Glasschmuck) links neben dem Regal (`.shelf-xmas-side`),
  jetzt auch in A Gentleman's Study. Adventskränze bleiben die alten. Mystery und Gentleman: Regal hinten an der Wand und weiter links (`.lib-row.has-left`,
  Raum nach unten verschoben). Lady: neuer Hintergrund „Gewächshaus mit Laterne“ (`rt2/conservatory`), Kleid weiter links, Hund 30 % größer,
  Bücherstapel und Auto getauscht (Tausch nach dem Einräumen in `tbRenderLibrary`), Ziele-Pokal halb so groß.
- 1.94.1: Weihnachtsbäume 40 % größer, davor die Krippe (Mystery: `stickers/crib-dog`; Gentleman/Lady: die Krippen-Trophäe `xmas2` wandert
  in der Weihnachtszeit aus der Vitrine vor den Baum, `cribFront`, `.shelf-crib-side`), Zeile hat dann links mehr Platz (`.has-xmas`).
  Mystery-Fuchs rechts neben der Vitrine (wo sonst die Puppe steht). Hund in allen Designs so groß wie in A Gentleman's Study (34 % / 12,5 %).
  Herren-Schneiderpuppe so hoch wie das Lady-Kleid (`--dh` = Regalhöhe × 0,96).
- 1.94.2: Bäume und Krippen in allen Designs an derselben Stelle und gleich groß (Baum 89,6 % Regalhöhe, Krippe 20,4 %, rechts vor dem Baum);
  Lady-Rosentischchen weiter hinten (bottom −4 %); nach dem Tausch Bücher/Auto wird das eingetauschte Stück kleiner, wenn die Reihe sonst überläuft.
- 1.95.1 Lady-Regal: obere zwei Reihen enden tiefer im Boden, die ganze Bodenkante liegt davor (`rt2/sv-*-fronts`, Perspektive); Bücherstapel ×0,65,
  Kerzenständer ×1,2, Fahrrad ×0,8, Bouquet ×1,2, Glasglocken ×0,85, Rosenquarz-Obelisk ×0,8 (`trophyRl`). Fach mit dem Bücherstapel: Boot links, Bouquet
  mittig, Bücher ganz rechts (`row.spread` → `.tro-spread`); passt es nicht, gibt erst das Bouquet (bis 75 %), dann die Bücher (bis 60 %) nach.
- Prompts Weihnachtsbaum (schmiedeeiserner Ständer, alter Glasschmuck) und Adventskranz für alle drei Designs:
  Sammelordner „Weihnachtsbaum und Adventskranz – Prompts (24.10.2026).md“.
- **1.95.0 Ende-zu-Ende-Verschlüsselung (gebaut):** Abschnitt „End-to-end encryption“ vor `syncPayload`. Ein zufälliger Datenschlüssel (AES-256-GCM)
  je Konto verschlüsselt alles, was in `user_data.data` geht (Umschlag `{ e2e:1, v, z (gzip), kv, kdf, pw, rc, code, iv, ct }`). Der Schlüssel liegt im Konto
  zweimal verschlossen: mit dem Passwort (PBKDF2-SHA256, 600 000 Runden, `pw`/`kdf`) und mit dem Wiederherstellungs-Code (24 Zeichen, `rc`); der Code selbst
  liegt mit dem Datenschlüssel verschlüsselt im Umschlag (`code`), damit jedes entsperrte Gerät ihn in den Einstellungen wieder zeigen kann. Jedes Gerät hält
  den Schlüssel lokal (`ewolfe_e2e_v1`). `kv` = Version der Schlösser: neues Passwort → neue `kv`, andere Geräte übernehmen neuere Schlösser beim Lesen
  (`e2eOpen`); `syncRemember` merkt sich `kv`, eine Abweichung erzwingt einen Schreibvorgang. Ohne Schlüssel kein Sync: Sperre „Daten entsperren“
  (Passwort, oder Code + Passwort, oder „Neu beginnen“ mit Warnung: `e2eStartOver`). Alte unverschlüsselte Zeilen werden beim ersten Sync verschlüsselt.
  Passwort vergessen: neues Passwort → auf einem Gerät mit Schlüssel neu verschlossen (`e2eRewrapPw`), sonst Code nötig (`e2ePendingPw`).
  Passwort-Regeln (`pwChecks`/`pwOk`, Häkchen `#pw-rules`): 10 Zeichen, groß, klein, Zahl, Sonderzeichen – bei „Konto erstellen“ und neuem Passwort.
  Einstellungen → „Datenschutz“: Hinweis, „Wiederherstellungs-Code anzeigen“, Datenschutzerklärung (`PRIVACY` de/en, Fenster `#privacy-modal`, ENTWURF
  mit Platzhaltern [Betreiber, Anschrift, E-Mail, AVV mit Supabase]). Getestet mit nachgebautem Supabase (Skript `e2etest.py` im Sammelordner).
- **1.96.0 Passkey (Face ID / Touch ID) statt Code:** Abschnitt „Passkey“ vor der Datenschutzerklärung. WebAuthn mit PRF-Erweiterung: der Passkey liefert ein
  Geheimnis, das den Datenschlüssel ein drittes Mal verschließt (`pk` im Umschlag: je Passkey `{ id, salt, iv, ct }`, gehört zu `kv`). `pkCreate` (rpId =
  Hostname, `user.id` = Supabase-Nutzer-ID), `pkGetOutput` (`evalByCredential`), `e2ePkUnlock` im Entsperr-Fenster (auch nach „Passwort vergessen“, dann
  wird mit dem neuen Passwort neu verschlossen). Angebot einmal nach Anmeldung/Entsperren (`pkOffer`, „Später“ je Gerät: `ewolfe_pk_later`), in den
  Einstellungen einrichten/weitere/entfernen. Braucht z. B. iOS 18 / macOS 15; sonst Hinweis `pkNoPrf`. Die Anmeldung am Konto bleibt E-Mail + Passwort.
  Getestet mit virtuellem Authenticator (Chrome, `hasPrf`): Skript `pktest.py` im Sammelordner „Datenschutz (24.10.2026)“.
- **Noch offen (Datenschutz):** Speicher-Erinnerung (80 %/90 %), verschlüsseltes Backup + monatliche Erinnerung, IndexedDB statt localStorage, Fotos einzeln
  im Supabase-Speicher, Konto in der App löschen; Platzhalter der Datenschutzerklärung füllen und rechtlich prüfen lassen; AVV und SMTP bei Supabase.

## Freude wächst aus dem Rad (seit 1.85.0) – ersetzt „Sechster Bereich Freude“ unten
- Das Lebensrad hat wieder **fünf** Ecken: `WHEEL_AREAS` (= `AREAS` ohne `freude`) für alles, was gezeichnet oder als Rad gezählt
  wird (Rad, Bereiche-Rad, Vergleich, Verlauf, Mini-Räder, Tagebuch-Bild, Schnappschüsse, Trophäen „alle besser“/„Angekommen“).
  `AREAS` behält alle sechs für Daten, Sync, Ziele (Ziele für Freude bleiben), Fragen, Impulskarten (alle Freude-Karten bleiben),
  Ereignis-Bereiche, Tagebuch-Bereichsseite.
- **Erwartete Freude** (`joySeries`): beginnt bei 50 %, ändert sich mit jedem Rad um die durchschnittliche Veränderung der fünf
  Bereiche, +5 wenn alle fünf wachsen, 0–100. **Gefühlte Freude**: auf der Bereiche-Seite tippt man eine von fünf Sonnen an
  (`JOY_SUNS` 20…100, `areasJoyHtml`, `areasJoy`), gespeichert wie bisher in `state.bewertungen.freude` mit der Zeit des Rads
  (`joyFeltAt`, 60 s Fenster). Alte Freude-Werte zählen als gefühlte Freude. Keine Pflicht: ohne Antippen zeigt die Sonne die erwartete.
- **Sonne in der Mitte** des Startseiten-Rads (`joySunSvg`): Größe und Schein nach der Freude (gefühlt, sonst erwartet), Wert
  darunter, Antippen erklärt beides (`joyInfo`).
- **Karte unter dem Rad** (`#joy-deep-wrap`, `joyDeepHtml`): gefühlt ≥ 15 Punkte (`JOY_GAP`) unter erwartet → „Dein Rad wächst –
  deine Freude noch nicht“ mit drei Fragen, „Ins Tagebuch“ (`joyDeepToDiary`) und „Freude-Impuls“ (`joyImpulse`). Wächst sie mit
  (Rad oder gefühlte Freude seit dem letzten Rad gewachsen) → kleiner Feier-Satz (`joyGrowV`, Wunsch des Nutzers); deutlich voraus → `joyAheadV`.
- Vergleich: Zeile „Freude: a % → b %“. Die Frage `joyAsk` entfällt. Freude-Trophäe (`joy10/20/30`) zählt weiter die gefühlte Freude.
- Leisten-Symbole wieder mit fünf Ecken: A Mystery World die alten Goldbilder (aus 1.77), A Gentleman's Study/A Morning Retreat neue
  Linien-SVGs. Vorschlagsseite (nicht im Commit): `vorschlag/freude.html`.

## Seit 1.84.4: Knöpfe überall gleich groß, Wunsch im Vergleich
- **Alle Regler-Knöpfe sind auf dem Bildschirm 26 px groß** (`KNOB_PX`), in allen Designs und auf allen Seiten (Wunsch des Nutzers,
  gilt immer): Regler (`input[type=range]`, `box-sizing:border-box`), Box-Breathing-Punkt (`#breath-dot`) und die Wunsch-Ecken in
  jedem Lebensrad (`wishKnob`, auch im kleinen Rad auf „Bereiche“). Knöpfe in Zeichnungen tragen die Klasse `ew-knob`; `knobFix()`
  rechnet 26 px in die Einheiten der jeweiligen Zeichnung um (MutationObserver + ResizeObserver, `breathOpen` ruft es auch auf).
  Der Ring-Knopf von A Gentleman's Study/A Morning Retreat: `r = 0,45·s`, Rand `s/10` (außen genau s).
- **Aussehen der Knöpfe in A Gentleman's Study und A Morning Retreat ist noch offen:** Vergleichsseite `knopf-vergleich.html` (nicht
  im Commit) zeigt A heute / B ohne hellen Rand / C runde Scheibe der Medaille (`img/rl/knob-medal.webp`, aus `rl/medal-cognac`
  ausgeschnitten; beides wieder entfernt). Box Breathing zeigt dort bisher noch den goldenen Faltpapier-Knopf von A Mystery World.
- **Seit 1.84.5 (Entscheidung des Nutzers):** Knopf „B ohne Rand“ überall und 30 % kleiner: `KNOB_PX` = 18. Ein Bild je Design
  (`KNOB_IMG`): A Gentleman's Study `img/rl/knob.webp`, A Morning Retreat `img/rt/knob.webp` (Bronze- bzw. Sand-Verlauf ohne Rand,
  mit Pillow gerechnet), A Mystery World weiter `img/knob-gold.webp`. Gleiches Bild für Regler, Box-Breathing-Punkt (`#knobPat` wird in
  `breathOpen` gesetzt) und Wunsch-Ecken. Ausnahme: der Box-Breathing-Punkt ist 20 % größer (`data-scale="1.2"`). Die Varianten
  A (mit Rand) und C (Medaille) sind verworfen.
- **Lebensräder vergleichen:** der Wunsch des neuesten Rads als gepunktete Linie in `KNOB_GOLD` (Schnappschüsse tragen jetzt
  `wishes`), in der Legende „Wunsch“ mit gepunktetem Strich (`.compare-swatch-wish`, Text `cmpWish`).
- **Idee Schneiderpuppe (nur A Gentleman's Study):** alte Herren-Schneiderpuppe im Reitfrack rechts neben dem Regal, später die
  Medaillen als Orden auf der Brust; Damen-Fassung später im Kleid mit Broschen und Gemme. Prompts: „Offene Prompts …“ Teil F.
  Erst einbauen, wenn die Bilder da sind.

## Sechster Bereich „Freude“ (seit 1.77.0)
- `AREAS` hat jetzt sechs Einträge, `freude` als letzter (Symbol `img/icons/ico-freude.webp`, gefaltete Goldsonne). Das Rad ist
  allgemein für N Bereiche gebaut (`axisAngle`, `wheelGeoFor` misst die Ecken, Beschriftung nach Richtung). Texte, Fragen,
  Impulse, Kurzbeschreibung je Bereich in de/en; 8 Impulskarten fre1–fre8 (Bild vorerst `stickers/sun`, echte Bilder folgen). Alte Daten: fehlende Listen werden beim Laden ergänzt; alte Räder ohne
  Freude bleiben in Verlauf/Vergleich (Freude dort neutral 50 %). Bisherige Nutzer werden einmal gefragt (`joyAsk`,
  Merker `ewolfe_joy_asked`). Trophäe „alle Bereiche besser“ zählt nur Bereiche mit Werten (mind. fünf).

## Umbau 1.76.0: Lebensrad entsteht in „Bereiche“, Ist & Wunsch, Ziele als Tagebuchseiten
- Seite **Bereiche** (`renderAreasPage`): kleines Live-Rad oben (`areasWheelSvg`), je Bereich aufklappbar (`areasToggle`):
  Text, „Fragen zum Nachdenken“ (`areaQuestions` im I18N), zwei Regler „So ist es jetzt“ (lila) und „So wünsche ich es mir“
  (gold), Entwurf in `areasDraft`; „Lebensrad speichern“ (`areasSave`) legt ein neues Rad an (alte bleiben, Vergleichen bleibt).
  Erstnutzer: „Mein erstes Lebensrad speichern“ + „Backup wiederherstellen“. Der Knopf „Neues Lebensrad erstellen“ ist weg;
  `openEvalModal()` öffnet jetzt die Bereiche-Seite (das alte Fenster `#eval-modal` wird nicht mehr benutzt).
- Wunsch: Feld `wish` je Eintrag in `state.bewertungen[a]` (`addBewertung(a, v, ts, wish)`, `currentWish`, `hasWishes`).
  Wer schon Räder ohne Wunsch hat, wird einmal gefragt (`maybeAskWishes`, Merker `ewolfe_wish_asked`).
- Lebensrad (Startseite) ~15 % größer (`wheelGeoFor(…, big)`: Seitenbeschriftungen über dem Punkt), Wunsch als goldene Linie mit
  Punkten, Fläche zwischen Ist und Wunsch mit Verlauf: größte Lücke Violett #A83DCF, kleinste Gold #ECA817 (`wheelGapSvg`).
- Ziele-Seite im Tagebuch-Look: cremeweißes Papier, dunkle Schrift, goldbraune Überschriften in Marcellus, feine Linien
  (CSS-Block „Ziele as diary pages“). Späterer Schritt: Ziele direkt ins Tagebuch.

## Wachsender Glanz, Tiere, Tagebuch löschen (seit 1.68.0 / 1.69.0)
- Stufen nach aktiven Tagen (Tage mit Übung, Stimmung, Lebensrad, Tagebuchseite, Dankbarkeit, seit 1.73.3 auch Ziel angelegt/
  erreicht und Ereignis; bloßes Öffnen der App zählt bewusst nicht; unter dem Regal steht „Aktive Tage: n · noch m bis zur nächsten
  Glanz-Stufe“ (`levelProgressText()`); `appActiveDays()`, `LEVEL_DAYS`
  = 0/7/30/100), dazu seit 1.69.0 zwei Zeit-Stufen (`LEVEL_MONTHS`): 5 nach einem halben Jahr, 6 nach einem Jahr ab dem
  ersten aktiven Tag (`appFirstActive`), nur wenn Stufe 4 erreicht ist. Klassen `html.lvl-1…6`, ab 4 zusätzlich `lvl-gold`.
  1: tonale Rahmen/Knöpfe, Lavendel, staubiges Regal nur am allerersten Tag (ab 2 aktiven Tagen sauber, seit 1.74.3); 2: goldene Überschriften/Leiste, sauberes Regal; 3: goldene Knöpfe,
  tonaler Rahmen mit dünner goldener Innenlinie (`img/frame-*-mid.png`), Efeu-Regal; 4: ganzer Goldrahmen + Schimmer;
  5: funkelnde Sterne in den Rahmenecken (`#lvl-fx`, `.lvl-star`); 6: mehr Sterne an den Rändern. Nichts wird je
  weggenommen; neue Stufe einmal angesagt (`levelUp`, Merker `ewolfe_level_seen`).
- Regal (seit 1.69.0): nur noch ein Regal. Trophäen stehen enger und werden notfalls etwas kleiner (bis 66 %), bis sie in
  die Fächer passen; der Rest steht oben auf dem Regal (`.shelf-roof`, Dachhöhe `roof` je Bild in `SHELVES`). Seit 1.73.7 sind die Fächer (`x` in `SHELVES`) genau an den Innenwänden der Bilder ausgemessen; Bücher stehen von
  Wand zu Wand (`.shelf-comp-books`), die Breite je Fach rechnet `rowCap()`.
- Tiere am Regal (`.shelf-animal`): Hund ab 15 aktiven Tagen (seit 1.73.9), Katze oben links ab Stufe 4 (`img/stickers/cat.webp`, fehlt
  noch – solange die Datei fehlt, bleibt sie einfach weg), Fuchs ab Stufe 6 links auf dem Boden, so groß wie der Hund.
  Der Hase kommt nur zu Ostern (Palmsonntag bis eine Woche nach Ostern, `isEasterTime()`/`easterSunday()`) und sitzt vor
  dem Regal, daneben Ostereier (`img/stickers/eggs.webp`).
- Kirchenjahr (seit 1.70.0): Fabergé-Ei als Trophäe für eine Übung in der Osterzeit, jedes Jahr ein neues (`easter<Jahr>`,
  ohne Glasglocke): 1. Jahr rosa klein, 2. blau größer, 3. mint am größten, danach große Eier reihum (`FAB`). Nikolaus (Bischof, `img/stickers/nikolaus.webp`) steht am
  5./6. Dezember links neben dem Regal, davor der Stiefel (`isNikolausTime()`); Trophäen für eine Übung am 6. Dezember:
  im ersten Jahr ein kleiner Nikolaus (`nikolaus`), im zweiten der Stiefel (`nikolaus2`, `img/trophy-boot.webp`). Trophäen dürfen seit 1.70.0 auch im Bücherfach neben den Büchern stehen (`pack()`).
- Tagebuch-Linien (seit 1.75.1): Linien, Karos und Punkte im gleichen 28-px-Raster (`--tbk`); je Linie eine Zeile, Text
  sitzt etwas hoch (Druckschrift padding-top 24, Handschrift 20,5 px Schrift, padding-top 19,5), berührt keine Linie.
- Advent (seit 1.75.1): Bibelverse nur an den Adventssonntagen (`advSun1…4`, nicht am 6.12.) und an Heiligabend, sonst Impulse.
- Feiertagskarten (seit 1.71.0, `HOLIDAY_CARDS`, `holidayKey()`, `holidayCard()`): an Feiertagen und im Advent (1.–24.12.)
  kommt zuerst die Feiertagskarte (Tageskarte und erstes Ziehen, `impulseDraw`), Id `'h-' + Schlüssel`. Nur positive
  Bibelverse (Luther 1912 / King James) oder eigene Impulse. Bild: Adventskranz mit passender Kerzenzahl (`advent-1…4`),
  sonst Sticker; fehlt ein Bild, nimmt die Karte den Stern. Feiertage: Advent, Weihnachten 25./26.12., Neujahr,
  Dreikönig, Palmsonntag bis Ostermontag, Himmelfahrt, Pfingsten, Erntedank (1. Oktobersonntag), Martinstag.
- Advent/Weihnachten am Regal: Kranz oben rechts (`adventCandles()`, Bilder `img/advent-1…4.webp`), 24.12.–6.1.
  Weihnachtsbaum links (`img/stickers/tree.webp`) und Schnee (`.shelf-snow`, `isChristmasTime()`). Trophäen für eine Übung
  am 24.–26.12.: 1. Jahr Stern (`xmas`, `img/trophy-star.webp`), 2. Jahr Krippe (`xmas2`, `img/trophy-crib.webp`).
- Schrift Marcellus (seit 1.73.0) auch für Unterüberschriften (h3/h4, Kartentitel, Einstellungs-Köpfe …) und Countdowns
  (Atemzahl, Phase, Stille-Uhr, Player-Zeit), seit 1.73.1 auch Knöpfe; Marcellus hat nur eine Stärke, daher nie fett.
- Querformat-Titelblatt (seit 1.74.0): Titel und Jahr genau mittig im goldenen Band (72,5–87,5 % der Höhe, bei allen
  fünf Covern gleich). Mitteilungspunkt am Konto-Symbol: Lila neben goldener Figur, Gold auf Stufe 1 (lila Figur).
- Untere Leiste seit 1.74.3 etwa 20 % niedriger (Symbole 24 px, `--nav-h`). Seit 1.75.0 Reihenfolge Bereiche · Ziele ·
  Lebensrad (Mitte, etwas größeres Symbol, nicht höher) · Tagebuch · Achtsamkeit (`NAV_ORDER`); Beschriftung überall gleich
  hoch, inaktiv `--ew-text-dim`, aktiv `--gold-text`, kein reines Weiß. Wischen zwischen Seiten ausgeschaltet (`SWIPE_ON`).
- Seit 1.75.0: Regler-Knopf und Box-Breathing-Punkt aus goldenem Faltpapier (`img/knob-gold.svg`); Stille Meditation ohne
  Musik, nur Gong (kein Lautsprecher-Knopf); Atemgeräusche 5 % lauter (`BREATH_VOL`, Box und 4-7-8); 5-4-3-2-1: Zahlen groß
  in Marcellus creme ohne Kugel, am Ende ein großer Haken; Atemzahl in Gold.
- Regal seit 1.75.0: Bücher eng ab der linken Wand, das Plus mit kleinem Abstand und nie auf der Seitenwand; Trophäen zuerst
  in die unteren Fächer, ins oberste Fach höchstens 3 (nur wenn unten zu voll); rutscht das Plus ins 2. Fach, füllen
  Trophäen den Rest davon. Abstände als `margin` an den Kindern (cqh misst das Fach), nicht am Fach selbst.
- Schalter „Tiere & Feiertagsschmuck“ in den Einstellungen (seit 1.72.0, `decorOn()`/`setDecor()`, `state.settings.decor`,
  je Gerät): blendet Tiere, Osterhase, Nikolaus, Kranz, Baum und Schnee aus; Trophäen und Feiertagskarten bleiben.
- Rote Kugel (`orb-pink`, Gruppe `loyal`): wächst wie die anderen Kugeln, 60/150/250 aktive Tage. Kugeln seit 1.69.0 unten rund ausgeschnitten.
- Tagebuch löschen (seit 1.69.0): Papierkorb oben links auf dem Titelblatt jedes Buchs (seit 1.73.6, `.tb-trash-btn`) und
  unten in den Tagebuch-Einstellungen „Tagebuch löschen“ (`tbDeleteBook`, mit Rückfrage).
  Grabstein `'j:' + Schlüssel` in `state.geloescht` plus `'s:'` für jede Seite; `delApply` entfernt das Buch, ein neues
  Buch mit gleichem Schlüssel hebt den Buch-Grabstein auf (`tbEnsureYear`), die alten Seiten bleiben gelöscht.
  Seiten löschen fragt schon immer nach (`tbDeletePage`).

## Offene Wünsche
- Eigenen SMTP-Dienst in Supabase eintragen, damit Passwort-Mails alle Nutzer erreichen
- Offline-Modus (Service Worker) und dauerhaften Speicher anfordern
- Eventuell eigene Domain statt der GitHub-Adresse
- Später: Community, Onboarding, Kalender-Export, Teilen, Admin-Bereich, KI-Funktionen, Abo
- **Altersempfehlung**: ab 16 (DSGVO: eigene Einwilligung ab 16), 12–15 mit Einverständnis der Eltern. Inhalte an sich
  kindgerecht (App-Store-Einstufung eher 4+/9+). Datenschutzerklärung vor öffentlichem Start prüfen lassen.
- **Kinder- und Jugendmodus** (Wunsch des Nutzers: finanzielle Bildung vom ersten Taschengeld an):
  - Gleiche fünf Bereiche, altersgerecht erklärt; später evtl. Bereiche wie Schule, Freunde, Familie, Gesundheit, Glaube.
  - Finanzen nach Alter: 6–9 „Mein Taschengeld“ (Sparschwein-Ziel mit Bild, füllt sich je gespartem Euro; drei Gläser
    Sparen/Ausgeben/Teilen); 10–13 Taschengeld-Tagebuch, Wunsch vs. Bedürfnis, „eine Woche warten vor dem Kauf“,
    Sparziele mit Datum; 14–17 erstes Konto, Nebenjob, Budget, Zinseszins einfach, Handyvertrag, Werbung durchschauen.
  - Kinder-Impulskarten (z. B. „Wer teilt, hat doppelte Freude“, „Jeder große Wunsch beginnt mit dem ersten gesparten
    Euro“), Trophäen (z. B. goldenes Sparschwein bei erreichtem Sparziel), Teilen mit Vers „Geben ist seliger denn
    nehmen“ (Apg 20,35).
  - Unter 16: ein Elternteil meldet sich an und legt Kinderprofile an; Eltern können Taschengeld mit einem Tipp eintragen.
  - Bibelverse in der App immer nur solche, die zu positivem Denken passen (Luther 1912 / King James).
