# Papier Piloten – Übergabe für Claude Code

## Projekt
- Browserspiel „Papier Piloten" (früher „Papierflieger Werkstatt"), in 7 Sprachen (Deutsch ist die Ausgangssprache, siehe „Sprachen“).
- Eine einzige Datei: `index.html` (HTML + CSS + Vanilla-JS, Canvas, keine Bibliotheken, nur Google Font Fredoka).
- Repository: `github.com/paper-pilots/paper-pilots-frontend` (Organisation `paper-pilots`).
- Lokal: `C:\Users\bened\OneDrive\Dokumente\Paper Pilots\paper-pilots-frontend` (enthält bisher nur `.git` und eine kleine `README.md`).
- Entstanden als claude.ai-Artifact: https://claude.ai/artifact/RkQxGKGJDdBzs2ds8AyLL7 . Der aktuelle Stand steht in `index.html` (Kopie von `papierflieger_v2.html`, im Ausgabeordner der claude.ai-Sitzung).

## Arbeitsregeln des Nutzers
- Bei „hochladen" oder „veröffentlichen": `git commit` mit aussagekräftiger Nachricht, dann `git push`.
- Bei Merge-Konflikten: `git fetch`, Konflikte selbstständig lösen.
- Mit aktivem Claude-Pro-Abo die eingebaute GitHub-Integration nutzen, sonst die GitHub CLI (`gh`).
- Nutzer schreibt Deutsch, Antworten auf Deutsch, kurz und ohne Fachjargon.

## Spielablauf (Kurzfassung)
1. Erststart: Spitzname-Dialog (min. 2 Zeichen, nicht schließbar). Danach öffnet sich sofort das Faltblatt mit Anleitung (nur neue Spieler, `tutorialDone`).
2. Anleitung: 4 geführte Falten (`GSTEPS`, Ringe als Ziel). Danach „Flieger übernehmen", dann 3 Übungswürfe mit gestrichelter Mittellinie (`throwHint`, `practiceThrows`).
3. Werfen: Flieger mit der Maus greifen, nach hinten ziehen, nach vorne schwingen, loslassen. Es gibt keinen Wurfknopf mehr. Der Flug startet genau an der Loslass-Stelle; Weite wird ab `launchX` gemessen.
4. Münzen = `floor(Bildpunkte/10)` (nicht die angezeigten Meter, siehe „Weite in Metern“). Damit kauft man Wurfstärke, Boosts, besseres Papier, Lack, Sticker. (Doppelmünzen wurden auf Wunsch komplett entfernt; ein alter Wert `coinLevel` im Speicherstand wird ignoriert.)

## Wichtige Konstanten und Formeln
- Terrasse: `SX = 150` (Startmitte), Terrasse 300 px breit, Boden `groundY = 380`, Canvas 800×400. Kamera folgt ab `W/2`.
- Perfekt-Linie unsichtbar bei `x = SX`: ±10 px → Wurf ×1,8, ±30 px → ×1,25 (`throwNote` zeigt Meldung). Meter-Striche unter der Terrasse sind absichtlich ausgeblendet.
- Wurf: `vx = (2 + strength*0.5) * paper.speed * boosts.speed * pm`, `vy = (-1 - strength*0.2 - boosts.lift) * tm`, Schwerkraft `paper.gravity * boosts.gravity`. `pm`/`tm` kommen aus der Zeigergeschwindigkeit beim Loslassen (letzte 120 ms) oder dem Abstand nach vorne.
- Upgrade-Kosten ×1,5 pro Stufe.
- Papiere (`PAPERS`): heft (gratis), glanz 60, perga 250, carbon 800. Die Einträge `eigen` und `order` sind nur Altlast und bleiben unsichtbar.
- Boosts (`BOOSTS`): clip 80, tape 150, rocket 300, sail 700; werden einmal gekauft und gelten dann für jedes Flugzeug, an jedem Flugzeug höchstens 2 gleichzeitig (siehe „Seite eines Flugzeugs“).
- Lack 30 Münzen, Sticker 10 Münzen pro Stück (max. 8). Bezahlt wird erst bei „Flieger übernehmen".

## Selbstgebaute Flieger
- Freies Falten: Ecke ziehen → Falte an der Mittelsenkrechten von Start und Ziel (`foldLayers`, Lagen als konvexe Polygone `{pts, back}`).
- Werte aus der Form (`analyze`): Symmetrie, Schlankheit, Stabilität, Auftrieb → `speed`, `gravity`.
- Gespeichert in `custom[]`: `{id, name, speed, gravity, shape, paint, stickers, color, custom:true}`. Löschen per Zwei-Klick-Knopf („Wirklich löschen?").
- Im Spiel gezeichnet über `drawShape(shape, entry)` (Form um 90° gedreht, Spitze = Flugrichtung).
- 3D-Ansicht (`draw3d`): Canvas-Painter-Algorithmus, Lagen von unten nach oben, V-Flügel, Kiel, Perspektive. Schleife `loop3d` startet erst am Ende des Skripts (Falle bei `const`/`let` und Reihenfolge der Deklarationen beachten).
- Katalog A1–Z20 und Flugzeug-Suche im Basteln-Fenster wurden entfernt (Wunsch des Nutzers). Nur `applyOrder`/`orderQ` bleiben, damit alte Spielstände mit bestellten Fliegern weiter geladen werden. Bereits bestellte Flieger liegen als normale `custom[]`-Einträge vor.

## Meine Flugzeuge (früher „Hangar“)
- Abschnitt „Meine Flugzeuge“ direkt über dem Flugfeld (`#hangar`, `renderHangar`, `drawHangar`; die Namen stammen noch aus der Hangar-Version, die Hallen selbst wurden auf Wunsch entfernt): Jeder selbstgebaute Flieger mit eigener Form steht als Karte mit dem Flugzeug (Lack, Sticker) da. Die anderen Standardpapiere (Glanz, Pergament, Carbon) und Flieger ohne Form werden bewusst nicht gezeigt, Die Reihe scrollt seitlich. Den früheren Papier-Laden und Boost-Laden unter dem Flugfeld gibt es nicht mehr; alles dafür steht auf der Seite des Flugzeugs. Unter dem Flugfeld steht nichts mehr außer „Spielstand zurücksetzen“ und dem besten Wurf.
- Klick auf eine Karte öffnet die Seite des Flugzeugs (siehe unten); dort wählt man ihn aus. Ein Flieger im Windkanal ist blass mit „Im Windkanal“ markiert.
- Wird mit `updateUI()` neu gezeichnet. Im Querformat am Handy ausgeblendet (nur das Spielfeld bleibt).

## Kein Standardflieger mehr
- Das Schulheftpapier-Flugzeug („heft“) wurde auf Wunsch entfernt: Es steht nicht mehr in „Meine Flugzeuge“ und kann nicht geworfen werden. `equipped` ist die Id eines selbstgebauten Fliegers oder leer (`""`). `paper()` liefert nur selbstgebaute Flieger (oder `null`), `ownPlanes()` die in „Meine Flugzeuge“, `ensureEquipped()` (am Anfang von `updateUI`) wählt automatisch den ersten Flieger, der nicht im Windkanal trainiert.
- Ohne Flieger passiert beim Werfen nichts; das Spielfeld und die Flugzeug-Reihe zeigen `noPlane` („Tippe oben auf Flieger basteln …“), trainieren alle im Windkanal, zeigt das Spielfeld `allTraining`. Beim ersten Start führt die Anleitung sowieso zuerst zum Falten.
- `PAPERS` bleibt als Liste der Papier-Sorten für das Papier-Upgrade (`heft` = Grundpapier jedes Fliegers), nicht mehr als Flugzeug.
- Alte Spielstände: Wer „heft“ ausgewählt hatte, bekommt den ersten selbstgebauten Flieger. Globale Boosts (`attached`) gehen an den ausgewählten oder ersten Flieger, das beste früher gekaufte Papier wird bei allen selbstgebauten Fliegern zum Papier-Upgrade, wenn es besser ist als ihr jetziges.
## Weite in Metern (realistisch gestaucht)
- Der Flieger fliegt weiter in Bildpunkten (`px = plane.x − launchX`), angezeigt wird `toMeters(px) = 75 · (1 − e^(−px/1450))` (`DIST_MAX` 75, `DIST_L` 1450). Ein normaler Wurf (450 px) gibt etwa 20 m, 1500 px 48 m, 5000 px 72,6 m; die Grenze 75 m liegt knapp über dem Weltrekord für Papierflieger (69 m). Auf Wunsch des Nutzers, weil 5 km „voll unrealistisch“ waren.
- Münzen zählen weiter nach den Bildpunkten: `floor(px / 10)`, damit sich die Spielwirtschaft nicht ändert. `best` und Duell-Weiten (`cDist`, `oDist`) sind in Metern (eine Nachkommastelle, `round1`, Anzeige mit `fmtM` im Zahlenformat der Sprache).
- Entfernungsmarken am Boden stehen bei 10, 20, … 70, 73 m an der Stelle `SX + toPx(m)`; der Abstand wächst, je näher man der Grenze kommt.
- Höhenanzeige: 20 Bildpunkte = 1 m (Sterne dann etwa ab 70 bis 90 m Höhe).
- Alte Spielstände (ohne `distScale: 2`): der Rekord wird beim Laden von Bildpunkten in Meter umgerechnet. Alte Duell-Weiten in der Datenbank bleiben unverändert.
## Höhe: Ballons, Wolken, Sterne
- Steigt der Flieger über den oberen Spielfeldrand, fährt die Kamera mit (`camY`, ≤ 0; in `animate`: Ziel `plane.y − 160`, weich nachgeführt, ohne Flug 0). Alles Gezeichnete läuft in `drawWorld` in Weltkoordinaten (`ctx.translate(0, −camY)`); y = 0 ist der obere Rand am Start, je kleiner y, desto höher.
- Schichten (`drawHighLayers`, deterministisch per `hash2`, mit Parallaxe): Heißluftballons etwa 170 bis 690 über dem Rand (bunt gestreift, wippen), dicke Wolken 800 bis 1800, darüber Sterne (erscheinen ab ca. 1400, funkeln) und ein Mond (Sichel, ca. 2750). Der Himmel (`skyAt`) wird mit der Höhe erst satter blau, ab ca. 1500 dunkel.
- Anzeige `#alt-hud` („↑ n m“, Höhe über der Terrasse) unter der Weite, sobald der Flieger weit genug oben ist (`plane.y ≤ 150`).
- Erreichbar nur bei viel Schwung nach oben (hohe Wurfstärke, Raketenstart, wenig Fall durch Papier und Windkanal), sonst bleibt der Flieger im Bild. Zum Testen: nach `throwPlane` in der Konsole `plane.y`, `plane.vy` und `flightGravity` setzen.
## Ton
- Beim Fliegen hört man ein leises Windpfeifen (`ensureAudio`, `windSound`, `windTick`). Es wird per Web Audio erzeugt, ohne Audiodatei: weißes Rauschen → schmaler Bandpass (Pfeifen, wabert mit 0,45 Hz) plus weiches tiefes Rauschen darunter. Lautstärke und Tonhöhe wachsen mit dem Tempo des Fliegers (`hypot(vx, vy)`), davor und danach blendet es weich aus. Gemessener Pegel beim Fliegen etwa −42 bis −27 dB.
- Der Browser erlaubt Ton erst nach einer Berührung: `ensureAudio` läuft bei jedem `pointerdown`/`keydown` (Capture) und legt den AudioContext dann an. Beim Wechsel des Tabs wird er angehalten (`visibilitychange`).
- Knopf `#sound-btn` unten auf der Hauptseite (🔊 Ton an / 🔇 Ton aus), gespeichert unter `pf_sound` im localStorage (eigener Schlüssel). Im Querformat am Handy ist er ausgeblendet; die Einstellung gilt trotzdem.
- Klack: Jeder aktive Knopf (`button`, nicht der Ton-Knopf selbst, nicht gesperrte) macht beim Drücken ein kleines Mausklick-Geräusch (`klack()`, Capture-Listener auf `click`): scharfer, kurzer Knacks (Rauschen durch Bandpass um 2,8 kHz, 12 ms) plus Körper (Bandpass 1,1 kHz) und ein tiefes Pochen (Sinus 160 Hz ohne Tonhöhenänderung), 65 ms später ein leiseres „Loslass“-Klicken. Keine gleitenden Töne: die klangen nach Laser (Versuch 2 war außerdem „viel zu hell“, Versuch 1 zu dumpf). Gemessen: Spitze etwa −20 dB, Schwerpunkt etwa 3,4 kHz, hörbar rund 35 ms.
- Idee für später: Wurf-Geräusch, Landung, Ventilatoren im Windkanal.
## Einstellungen (Zahnrad)
- Unten auf der Hauptseite steht nur noch das Zahnrad ⚙ (`#settings-btn`) neben dem besten Wurf. Es öffnet `settings-dialog` mit Ton an/aus (`#sound-btn`), Sprache (`#lang-select`), „Spielstand zurücksetzen“ (`#reset-btn`, Zwei-Klick) und der Anleitung „So wird gespielt“ (Schlüssel `set.title`, `set.howto`, `how.1` bis `how.7`, in allen 7 Sprachen). Die IDs sind gleich geblieben. Bei neuen Funktionen die Anleitungstexte mitpflegen. Die Sprachknöpfe im Anmeldefenster bleiben.
## Sprachen
- Sieben Sprachen: Deutsch, English, Español, Français, Italiano, Português, Türkçe (`LANGS`). Alle Texte stehen in `I18N[sprache][schlüssel]` am Anfang des Skripts; `t("schlüssel", {param: wert})` liefert den Text, Platzhalter heißen `{n}`, `{d}` usw. Fehlt ein Schlüssel in einer Sprache, gilt Deutsch.
- Feste Texte im HTML tragen `data-i18n="schlüssel"` (Text), `data-i18n-ph` (Platzhalter) oder `data-i18n-aria` (aria-label); `applyLang()` setzt sie. Texte, die der Code zusammenbaut (Meldungen, Laden, Fenster), rufen `t()` beim Zeichnen auf; `setLang()` zeichnet neu (`updateUI` usw.).
- Sprachwahl: Beim ersten Start als Knöpfe im Anmeldefenster (`#lang-chips`, vorgewählt nach Browsersprache, sonst Englisch), später als Auswahlfeld `#lang-select` unten auf der Hauptseite. Gespeichert unter `pf_lang` im localStorage (eigener Schlüssel, nicht im Spielstand). Alte Spielstände ohne `pf_lang` starten auf Deutsch.
- Namen von Papieren und Boosts: `PAPERS[].name`, `BOOSTS[].name/desc` werden von `refreshNames()` je Sprache gesetzt (Schulheftpapier nur, wenn es nicht selbst umbenannt wurde). Selbst vergebene Flieger-Namen werden nicht übersetzt.
- Namensvorschläge für Flieger: `NAMEDATA[sprache]` mit `mode` `pre` (Deutsch: zusammengeschrieben), `ap` (Englisch, Türkisch: „Silver Eagle“) oder `ba` (Romanisch: „Águila Relámpago“).
- Neue Sprache hinzufügen: Eintrag in `LANGS`, vollständiges `I18N[xx]` (alle Schlüssel von `I18N.de`) und `NAMEDATA[xx]`. Prüfen: Alle Schlüssel von `I18N.de` müssen in jeder Sprache vorkommen und dieselben Platzhalter haben (Test in der Konsole).
- Übersetzungen stammen vom Assistenten, nicht von Muttersprachlern: bei Gelegenheit gegenlesen lassen.
## Seite eines Flugzeugs (Upgrades)
- Klick auf eine Karte in „Meine Flugzeuge“ öffnet `plane-dialog` (`openPlane`, `renderPlane`): großes Bild, Werte, Knopf „Auswählen“ (gesperrt beim Fliegen, im Duell-Wurf und im Windkanal) und „Upgrades“.
- Upgrades pro Flugzeug: bis zu `MAX_UP` = 2 Boosts (Büroklammer-Nase, Klebeband-Flügel, Raketenstart, Windsegel; kaufen oder anbringen/entfernen direkt dort) und besseres Papier (`pap`: Glanz 60, Pergament 250, Carbon 800 Münzen, bezahlt wird der Unterschied zum aktuellen Papier). Bei selbstgebauten Fliegern liegen sie am Flieger (`c.boosts`, `c.pap`), bei Standardpapieren in `pupg[id]`. `upOf(p)` liefert sie, `effStats(p)` die Werte mit Papier und Boosts; `throwPlane` rechnet damit.
- Boosts werden einmal gekauft (`ownedBoosts`) und gelten für alle Flugzeuge. Das alte globale `attached` wird beim Laden dem ausgewählten Flieger zugeteilt (höchstens 2) und dann nicht mehr gespeichert. Im Duell wandern `boosts`, `pap` und `lvl` mit dem Flieger (`planePack`, `addPlaneCopy`).
- Die Boost-Auswahl im Bastel-Fenster gibt es nicht mehr. Selbstgebaute Flieger löscht man auf ihrer Seite („Flieger löschen“, zwei Klicks, nicht im Windkanal).
- Alte Spielstände: Das beste früher gekaufte Papier (Glanz/Pergament/Carbon) wird beim Laden zum Papier-Upgrade des Schulheftpapiers (`pupg.heft.pap`); war so ein Papier ausgewählt, wird das Schulheftpapier ausgewählt. `buyPaper`/`renderShop` und die zwei Läden sind entfernt.
## Windkanal
- Knopf „Windkanal“ zwischen „Duell“ und „Spieler“. Dialog `tunnel-dialog` mit Canvas (Glasröhre, Ventilator vorne und hinten, Flieger schwebt in der Mitte).
- Ein selbstgebauter Flieger liegt im Windkanal (`tunnel = {id, since, focus}`, gespeichert). Beim Hineinlegen wählt man, was trainiert wird: **Tempo** (`focus: "speed"`), **Gleitflug** (`"glide"`, weniger Fall) oder **Abwurf** (`"launch"`, höherer Abwurf wie beim Raketenstart), alles als Knöpfe (`FOCI`) in der Zeile des Fliegers (der aktive trägt ein ✓; ein anderer Knopf auf demselben Flieger wechselt nur das Ziel, die angefangene Zeit bleibt). Alle `TUNNEL_MS` (3 Minuten) steigt er eine Stufe (`lvl`, höchstens `TUNNEL_MAX` = 20), und der gewählte Wert wird um **3 %** besser: Tempo `speed × 1,03`, Gleitflug `gravity ÷ 1,03` (mindestens 0,02), also Gleitflug-% × 1,03, Abwurf `c.lvlLift`++ (jede Stufe = 3 % des Grund-Abwurfs `1 + Wurfstärke·0,2`, in `effStats` als `lift` eingerechnet, ausgewiesen als „Abwurf +n %“ auf der Flugzeug-Seite; geht beim Duell mit). Alte Windkanal-Einträge ohne `focus` trainieren Tempo. Die Zeit läuft per Zeitstempel auch bei geschlossenem Fenster und beim Neuladen weiter (`tickTunnel`).
- Es liegt höchstens ein Flieger im Windkanal. Beim Herausnehmen bleibt die angefangene Zeit im Flieger (`prog`). Ein Flieger im Windkanal ist gesperrt: Auf seiner Seite steht „Trainiert gerade" statt „Auswählen", er kann nicht gelöscht und nicht ins Duell gesetzt werden. Beim Hineinlegen wird er abgewählt (Standardpapier). Herausnehmen gibt ihn wieder frei. `lvl` wird beim Duell mitgegeben (`planePack`).
- Idee für später: Symmetrie/Stabilität als weitere Werte (heute nur beim Falten berechnet und nicht gespeichert), Offline-Obergrenze.

## Namen neuer Flieger
- Sobald ein Flieger fertig gefaltet ist (Bereich `#craft-style` wird sichtbar), schlägt das Spiel einen coolen Namen vor (`suggestName`: je Sprache `NAMEDATA[lang]`: Vorsilbe + Endung, z. B. „Blitzfalke“, „Sturmschwalbe“). Schlanke Flieger bekommen Tempo-Namen (`fast`), Flieger mit viel Auftrieb Gleiter-Namen (`glide`). Bereits vergebene Namen und schon vorgeschlagene Namen (`usedNames`) kommen nicht nochmal: pro Typ gibt es rund 640 (schnell, Gleiter) bzw. 1700 (normal) Möglichkeiten; erst wenn alle durch sind, beginnt es von vorn.
- Im Feld `#craft-name` kann man den Namen ändern; „🎲 Neuer Vorschlag“ würfelt neu. Der Vorschlag kommt einmal pro Blatt (`craftNameSet`), spätere Falten ändern ihn nicht. Bleibt das Feld leer, wird ein neuer Vorschlag genommen. Umbenennen im Nachhinein: Auf der Seite eines selbstgebauten Fliegers (`#pl-rename`, `renamePlane`) steht oben ein Namensfeld mit „Umbenennen“ (auch Enter) und 🎲 für einen neuen Vorschlag; höchstens 24 Zeichen, leer ändert nichts. Auch das Schulheftpapier kann umbenannt werden (Name in `pupg.heft.name`, beim Laden auf `PAPERS` angewendet, beim Zurücksetzen auf den übersetzten Standardnamen zurück (`refreshNames`)). Gespeichert wird auch beim Verlassen des Felds (außer beim Wechsel auf 🎲); eine Meldung `#pl-rename-note` bestätigt es. Achtung: `.name-row[hidden]` braucht eine eigene `display:none`-Regel, sonst überschreibt `display:flex` das `hidden`-Attribut. Das Feld wird beim Tippen nicht von Neuzeichnen überschrieben. „Eigenbau n“ gibt es nicht mehr (alte Flieger behalten ihren Namen).
## Speicherstand (localStorage)
- Schlüssel `papierflieger` mit: `coins, strength, upgradeCost, best, owned, equipped, craftQ, ownedBoosts, orderQ, shapes (alt), custom, nick, tutorialDone, throwHint, practiceThrows, tunnel, pupg`.
- Weiterer Schlüssel: `pf_claimed` (schon verbuchte Duelle).
- Die früheren Testlinks `#muenzen0` und `#plus10k` sind entfernt. Zum Testen Münzen in der Konsole setzen (`coins = 10000; save(); updateUI()`).
- Alte Spielstände ohne `custom` werden beim Laden migriert (`eigen`/`order` → `custom[]`).

## Duell-Modus (hängt an claude.ai!)
- Duell-Knopf oben links. Man setzt einen eigenen Flieger ein, wirft, und teilt einen Link `…/artifact/<id>#d_<docId>`.
- Gegner wählt auch einen Flieger. Wer weiter fliegt, bekommt den Flieger des anderen (Unentschieden: jeder behält seinen).
- Technik: Capability `db` (Sammlung `duels`, Felder `cId, cNick, cPlane, cDist, status, oId, oNick, oPlane, oDist, result, claimed, created`), `user` (`id()`).
- Der Einsatz des Herausforderers liegt im Duell (aus `custom` entfernt) und kommt über `claimDuels()` bzw. `cancelDuel()` zurück oder geht weg.
- Gilt nur im claude.ai-Artifact. Beide Spieler brauchen ein Konto und mindestens Mitwirkenden-Rechte (sonst schlägt `set` fehl). Ohne `db` bleibt nur die einfache Herausforderungs-Nachricht mit Teilen-Knopf.
- `DUEL_LINK` ist fest auf die Artifact-URL gesetzt. **Bei Hosting auf GitHub Pages anpassen** und dort ein eigenes Backend einplanen, sonst funktionieren die Einsatz-Duelle nicht.

## Plattform-Einschränkungen (aus dem Artifact-Sandbox-Betrieb)
- Kein `alert`, `confirm`, `prompt` (werden blockiert). Bestätigungen laufen über Zwei-Klick-Knöpfe.
- Die Spielfläche hat `touch-action: none` (Ziehen am Handy; Scrollen nur außerhalb).
- Handy im Querformat (`orientation: landscape` und `max-height: 520px`): Nur das Spielfeld (`.stage`) ist sichtbar, so groß wie die Höhe es zulässt, ohne Scrollen. Knöpfe, Laden und Anzeigen gibt es nur im Hochformat.
- Hash-Links nur als einfaches Token (`#d_…`).

## Stand und offene Punkte
- **Nichts davon wurde im Browser getestet.** Es gab nur Syntaxprüfungen mit `node --check`. Vor dem ersten Release alles einmal durchspielen: Anmeldung, Anleitung, Wurf mit Perfekt-Linie, Basteln, Lack/Sticker, 3D-Ansicht, Boosts, Löschen, Duell (mit zweitem Konto).
- Duell-Nachricht heißt jetzt „Papier Piloten" (erledigt). Im Browser getestet (ohne Duell mit zwei Konten): Anmeldung, Anleitung, Würfe, Laden, Boosts, Basteln, Lack/Sticker, Löschen, Handy-Ansicht.
- Ideen: Treffer-Linie nach Treffer kurz aufblitzen lassen, fertige Flieger nachträglich umlackieren, Boosts am Flieger sichtbar zeichnen, Datei in Module aufteilen, echte Bestenliste und Konten.
- Das Spiel wurde bisher nur als Artifact veröffentlicht, noch nicht ins GitHub-Repository gepusht (siehe Arbeitsregeln). `index.html` muss zuerst aus der claude.ai-Sitzung ins Repository kopiert werden.
