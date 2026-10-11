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
  keine externen Bibliotheken außer Google Fonts „Inter"). Einzige Zusatzdateien: Audios im Ordner `audio/`.
- Eingebaute Meditationen mit Audio (`builtIn:true` in `DEFAULT_MEDITATIONEN`, Feld `audio` = Pfad zur Datei,
  z. B. „Lemonmeditation" → `audio/lemonmeditation.m4a`) werden per `ensureBuiltInMeditations()` bei allen
  Nutzern ergänzt und aktuell gehalten. Sie lassen sich nicht löschen und kosten keinen App-Speicher.
  Ihre Einleitung steht zweisprachig im `I18N` (Feld `notesKey`, z. B. `mediLemonNote`).
  „Box Breathing" (`seed-box`, Feld `tool:'box'`) öffnet eine geführte Atemübung (`breathOpen()`, Fenster
  `#breath-modal`: Punkt wandert ums Quadrat, je 4 s ein/halten/aus/halten).
- Symbole: eigene Linien-Icons statt Emojis über `ico(name)` / `UI_ICONS` (Tagebuch, Löschen …).
- Fenster (`.modal-backdrop` ohne `.page-modal`) liegen immer über den Seiten (z-index 110 statt 100).
- Veröffentlicht über **GitHub Pages** aus diesem Repository (Branch `main`, Ordner `/`):
  https://k9vk58y99f-max.github.io/k9vk58y99f-max.github.io-ewolfe-app-/
- Ein Commit + Push auf `main` ist die Veröffentlichung; nach 1–2 Minuten ist es live.
- Bei jeder Änderung `APP_VERSION` erhöhen (aktuell `1.24.0`) und `APP_BUILD` anpassen.
  Die Version steht unten in der App.

## Wichtige Stellen im Code
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
  Seite löschen: runder Knopf unten links (`.tb-del-fab`). Auf dem Blatt selbst nur noch „Bild". Die Seitengröße NICHT von der Bildschirmhöhe abhängig machen: der Seitenumbruch
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
