# Papier Piloten – Übergabe für Claude Code

## Projekt
- Browserspiel „Papier Piloten" (früher „Papierflieger Werkstatt"), komplett auf Deutsch.
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
4. Münzen = `floor(Weite/10) * 2^coinLevel`. Damit kauft man Wurfstärke, Papier, Boosts, Doppelmünzen, Lack, Sticker.

## Wichtige Konstanten und Formeln
- Terrasse: `SX = 150` (Startmitte), Terrasse 300 px breit, Boden `groundY = 380`, Canvas 800×400. Kamera folgt ab `W/2`.
- Perfekt-Linie unsichtbar bei `x = SX`: ±10 px → Wurf ×1,8, ±30 px → ×1,25 (`throwNote` zeigt Meldung). Meter-Striche unter der Terrasse sind absichtlich ausgeblendet.
- Wurf: `vx = (2 + strength*0.5) * paper.speed * boosts.speed * pm`, `vy = (-1 - strength*0.2 - boosts.lift) * tm`, Schwerkraft `paper.gravity * boosts.gravity`. `pm`/`tm` kommen aus der Zeigergeschwindigkeit beim Loslassen (letzte 120 ms) oder dem Abstand nach vorne.
- Upgrade-Kosten ×1,5 pro Stufe. Doppelmünzen: `3000 * 2^coinLevel`.
- Papiere (`PAPERS`): heft (gratis), glanz 60, perga 250, carbon 800. Die Einträge `eigen` und `order` sind nur Altlast und bleiben unsichtbar.
- Boosts (`BOOSTS`): clip 80, tape 150, rocket 300, sail 700; höchstens 2 gleichzeitig (`attached`).
- Lack 30 Münzen, Sticker 10 Münzen pro Stück (max. 8). Bezahlt wird erst bei „Flieger übernehmen".

## Selbstgebaute Flieger
- Freies Falten: Ecke ziehen → Falte an der Mittelsenkrechten von Start und Ziel (`foldLayers`, Lagen als konvexe Polygone `{pts, back}`).
- Werte aus der Form (`analyze`): Symmetrie, Schlankheit, Stabilität, Auftrieb → `speed`, `gravity`.
- Gespeichert in `custom[]`: `{id, name, speed, gravity, shape, paint, stickers, color, custom:true}`. Löschen per Zwei-Klick-Knopf („Wirklich löschen?").
- Im Spiel gezeichnet über `drawShape(shape, entry)` (Form um 90° gedreht, Spitze = Flugrichtung).
- 3D-Ansicht (`draw3d`): Canvas-Painter-Algorithmus, Lagen von unten nach oben, V-Flügel, Kiel, Perspektive. Schleife `loop3d` startet erst am Ende des Skripts (Falle bei `const`/`let` und Reihenfolge der Deklarationen beachten).
- Katalog A1–Z20 und Flugzeug-Suche im Basteln-Fenster wurden entfernt (Wunsch des Nutzers). Nur `applyOrder`/`orderQ` bleiben, damit alte Spielstände mit bestellten Fliegern weiter geladen werden. Bereits bestellte Flieger liegen als normale `custom[]`-Einträge vor.

## Windkanal
- Knopf „Windkanal“ zwischen „Duell“ und „Spieler“. Dialog `tunnel-dialog` mit Canvas (Glasröhre, Ventilator vorne und hinten, Flieger schwebt in der Mitte).
- Ein selbstgebauter Flieger (`custom[]`) liegt im Windkanal (`tunnel = {id, since}`, gespeichert). Alle `TUNNEL_MS` (3 Minuten) steigt er eine Stufe (`lvl`, höchstens `TUNNEL_MAX` = 20): ungerade Stufen Tempo +0,03, gerade Stufen Gleitflug +5 % (`gravity × 0,95`, mindestens 0,02). Die Zeit läuft per Zeitstempel auch bei geschlossenem Fenster und beim Neuladen weiter (`tickTunnel`).
- Es liegt höchstens ein Flieger im Windkanal. Beim Herausnehmen bleibt die angefangene Zeit im Flieger (`prog`). Ein Flieger im Windkanal ist gesperrt: Im Laden steht „Trainiert gerade" statt „Auswählen", er kann nicht gelöscht und nicht ins Duell gesetzt werden. Beim Hineinlegen wird er abgewählt (Standardpapier). Herausnehmen gibt ihn wieder frei. `lvl` wird beim Duell mitgegeben (`planePack`).
- Idee für später: Symmetrie/Stabilität als weitere Werte (heute nur beim Falten berechnet und nicht gespeichert), Offline-Obergrenze.

## Speicherstand (localStorage)
- Schlüssel `papierflieger` mit: `coins, strength, upgradeCost, best, owned, equipped, craftQ, ownedBoosts, attached, orderQ, shapes (alt), custom, coinLevel, nick, tutorialDone, throwHint, practiceThrows, tunnel`.
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
- Hash-Links nur als einfaches Token (`#d_…`).

## Stand und offene Punkte
- **Nichts davon wurde im Browser getestet.** Es gab nur Syntaxprüfungen mit `node --check`. Vor dem ersten Release alles einmal durchspielen: Anmeldung, Anleitung, Wurf mit Perfekt-Linie, Basteln, Lack/Sticker, 3D-Ansicht, Boosts, Löschen, Duell (mit zweitem Konto).
- Duell-Nachricht heißt jetzt „Papier Piloten" (erledigt). Im Browser getestet (ohne Duell mit zwei Konten): Anmeldung, Anleitung, Würfe, Laden, Boosts, Basteln, Lack/Sticker, Löschen, Handy-Ansicht.
- Ideen: Treffer-Linie nach Treffer kurz aufblitzen lassen, fertige Flieger nachträglich umlackieren, Boosts am Flieger sichtbar zeichnen, Datei in Module aufteilen, echte Bestenliste und Konten.
- Das Spiel wurde bisher nur als Artifact veröffentlicht, noch nicht ins GitHub-Repository gepusht (siehe Arbeitsregeln). `index.html` muss zuerst aus der claude.ai-Sitzung ins Repository kopiert werden.
