# ISO Coach — Prototyp

Eine einzelne HTML-Datei. Keine Installation, keine Abhängigkeiten, kein Server nötig.

---

## Lokal starten

`index.html` doppelklicken. Fertig.

Der Browser darf dabei offline sein. Deine Daten liegen ausschließlich in diesem einen Browser
auf diesem einen Gerät — sie gehen nirgendwohin.

## Auf dem Handy nutzen

Am Handy ist die App eigentlich erst brauchbar — du liegst beim Liegestütz auf dem Boden und
brauchst die Tonsignale.

**Die Adresse:** https://lukebrandenburg9-beep.github.io/Iso-coach/

Diese Adresse **am iPhone in Safari** öffnen (Android: Chrome) → Teilen → **„Zum
Home-Bildschirm"**. Sie startet dann im Vollbild, ohne Browserleiste, mit eigenem Symbol.

**Chrome auf dem iPhone kann „Zum Home-Bildschirm" grundsätzlich nicht** — das kann unter iOS
nur Safari. Ein Artifact-Link funktioniert dafür ebenfalls nicht: der öffnet die
Claude-Oberfläche mit der App darin, und die lässt sich nicht installieren.

## Wie Änderungen ankommen

Der Ordner auf dem Mac ist ein Git-Repository und hängt an
[github.com/lukebrandenburg9-beep/Iso-coach](https://github.com/lukebrandenburg9-beep/Iso-coach).
Eine Änderung geht so hoch:

```bash
git add -A && git commit -m "kurze Beschreibung" && git push
```

GitHub Pages baut die Seite danach in etwa einer Minute neu. Du musst nichts hochladen.

**Woran du erkennst, ob das Update angekommen ist:** Unten in der App steht die Fassung, zum
Beispiel „Fassung 2026-09-14 a". Die wird bei jeder Änderung hochgezählt.

**Wenn noch die alte Fassung erscheint:** GitHub Pages lässt Browser die Seite bis zu zehn
Minuten zwischenspeichern, und eine installierte App auf dem iPhone hält manchmal länger daran
fest. App aus dem App-Umschalter wegwischen und neu öffnen.

**Deine Messwerte überleben die Updates.** Sie liegen unter dem Schlüssel `isocoach.v1` im
Browser, gebunden an die Adresse. Solange die Adresse gleich bleibt, bleiben die Sessions
erhalten; geänderte Übungsstufen werden über `EX_VERSION` sauber nachgezogen. Vor größeren
Umbauten trotzdem einmal über **Mehr → Sichern (Datei)** sichern.

**Das Repository ist öffentlich**, weil GitHub Pages ohne bezahltes Konto nur so funktioniert.
Unbedenklich: In den Dateien stehen keine persönlichen Daten — die Messwerte entstehen erst
beim Benutzen und bleiben im Browser des jeweiligen Nutzers. Das Handoff mit den Zugängen liegt
eine Ebene höher und ist nicht Teil des Repositorys.

---

## Was drinsteckt

Der Prototyp setzt Abschnitt 4 des Handoffs um — er ist kein Timer mit Deko, sondern die
Rechenlogik:

- **Die Kurve** `t = W'/(P − CP)`, gefittet über eine eindimensionale Suche, ältere Messpunkte
  werden mit 3 % je neuerem Punkt abgewertet.
- **Gespreiztes Onboarding** über drei Stufen (leicht / mittel / schwer) statt einer. Der
  Einstieg hängt an der **Kurve**, nicht am Workout: Eine Übung, die später dazukommt, holt ihre
  fehlenden Zonen nach. Gezählt werden dabei leere Zonen, nicht Messungen — zwei gespreizte
  Punkte sind mehr wert als fünf aus derselben Zone.
- **Die Stufe wählst du**, nicht die App. Sie schlägt eine vor und zeigt je Stufe die erwartete
  Haltezeit sowie die Zone, in der sie voraussichtlich landet — entscheiden tust du. Der
  Messpunkt zählt so, wie er herauskommt: Er wird der Zone zugeordnet, in der er tatsächlich
  liegt, und den Richtwert zu verfehlen ist bei selbst gewählter Stufe kein Fehlschlag.
- **Zonenrotation:** Die App fordert die Zone an, die am längsten zurückliegt — **und erholt
  ist**. Ohne Spreizung wird die Kurve wertlos, ohne Erholung die Messung.
- **Erholung:** Dieselbe Muskelgruppe **dreimal pro Woche**, jedes Energiesystem — also jede
  Zone — **einmal pro Woche**. Beides zusammen ist genau eine Rotation: drei Zonen, drei
  Einheiten, ein Mo/Mi/Fr-Rhythmus. Umgesetzt als zwei Schranken: mindestens zwei Tage zwischen
  zwei Einheiten derselben Muskelgruppe, sieben Tage bis dieselbe Zone wiederkommt.
  Während der drei Einstiegstests genügen **24 Stunden** — sonst zöge sich das Onboarding über
  drei Wochen. Übungen **ohne Überschneidung** (Liegestütz, Ausfallschritt, Seitstütz, Schrägzug,
  Fersenbrücke) dürfen am selben Tag getestet werden; die App sagt am Pausentag, welche Gruppe noch frei ist. Der Pausentag
  nennt Grund und Termin und lässt sich bewusst übergehen. Werte unter *Mehr → Erholung*.
- **Höchstens sechs Blöcke** je Workout. Fünf Übungen, davon drei zweiseitig, wären acht Blöcke
  — in Zone C über eine Stunde. Die App nimmt deshalb die freigegebenen Übungen in der
  Reihenfolge, wie lange ihre **Kurve** nicht mehr gemessen wurde, und stellt den Rest auf die
  nächste Sitzung zurück; dort stehen sie oben. Wer selbst entscheiden will, benutzt auf der
  Startseite *Heute selbst zusammenstellen* — das gilt nur für diese eine Sitzung und hebt den
  Deckel auf.
- **Progressionsregeln 1–4** über zwei Bahnen: Leiter und Zusatzgewicht. Am oberen Ende der
  Leiter kommt Gewicht dazu statt gar nichts, am unteren geht erst Gewicht weg, bevor die Stufe
  fällt. Zweimal Schmerzabbruch stuft zurück — und wenn nichts Leichteres mehr da ist, sagt die
  App das ausdrücklich, statt zu schweigen.
- **Die Kurve gilt nur, wo gemessen wurde.** Sie merkt sich die kleinste und größte Last, aus der
  sie stammt. Außerhalb zeichnet das Profil gestrichelt und schreibt „außerhalb" statt einer Zahl
  — keine Vorhersage über Lasten, bei denen du nie warst.
- **Kontrollposition.** Liegen die verfügbaren Stufen einer Übung weniger als Faktor 1,25
  auseinander, gibt es **keine Kurve**, sondern einen Verlauf der Haltezeit über die Wochen. Zwei
  zu eng beieinander liegende Lasten ergeben rechnerisch keine Kurve — sie ergeben eine, die
  perfekt aussieht und falsch ist.
- **Schmerz** als Häkchen pro Satz plus ein Ampelwert am Sessionende.
- **Sicherheitsregel:** dreimal in Folge Ampelwert 6 oder höher → Hinweis auf ärztliche Abklärung.
- Red-Flag-Abfrage im Onboarding, Timer mit Tonsignalen, 20-Sekunden-Pausen, Export/Import.
- **Sicherheitsfrage, wenn eine Übung eine neue Körperregion öffnet.** Der Fragebogen läuft sonst
  nur im Onboarding — vor der ersten Fersenbrücke wird die Frage zur proximalen Hamstring-Sehne
  nachgeholt.

**Bewusst nicht drin:** CMF. Die Begründung stand bis zum 17.09. auch in der App und ist dort
wieder raus — sie erklärt ein Verfahren, das der Nutzer nie zu sehen bekommt. Nachzulesen in
`../Modell-Befunde_Simulation.md` und als P3 in `../ISO-Coach_Probleme-und-Aenderungen.md`.

---

## Was du an Gerät brauchst

Unter **Mehr → Setup → Geräte** hakst du an, was du hast. Drei Übungen kommen mit Möbeln aus
— Arbeitsplatte, Tisch, Stuhl, Hocker, Kiste, Kissen —, zwei nicht:

| Gerät | wofür | voreingestellt |
|---|---|---|
| Etwas zum Daranziehen | Schrägzug, **die ganze Übung** | an — eine stabile Tischkante zählt mit |
| Faszienrolle | Fersenbrücke, obere Stufe | aus |
| Turnringe | Fersenbrücke, oberste Stufe | aus |

Stufen, die ein fehlendes Gerät brauchen, erscheinen nicht in der Auswahl — aber die
**Nummerierung bleibt stehen**. Sonst verschöbe sich die gespeicherte Stufe deiner alten
Messungen, sobald du ein Gerät anhakst. Eine Übung, von der keine einzige Stufe ausführbar ist,
taucht im Workout gar nicht erst auf.

---

## Was du einstellen solltest

Unter **Mehr → Stufen und Lastfaktoren**.

Die Lastfaktoren sind Verhältniszahlen, keine Kilogramm — die leichteste Stufe ist 1,00, alles
andere bezieht sich darauf. Sie sind **grob geschätzte Startwerte** und liegen in deinem
Fachbereich.

Ein Punkt, der sich beim Testen gezeigt hat und den du kennen solltest: **Wie fein die Stufen
abgestuft sind, entscheidet, welche Zonen überhaupt trainierbar sind.** Mit den ursprünglich
fünf groben Stufen war Zone B (60–90 s) für einen durchschnittlichen Nutzer gar nicht
erreichbar — der Sprung von Stufe zu Stufe übersprang sie. Deshalb jetzt acht Stufen mit
einem Lastverhältnis von etwa 1,15 zwischen benachbarten Stufen. Wenn du die Stufen änderst,
prüfe unter **Profil → „Was du auf jeder Stufe schaffst"**, ob noch alle drei Zonen vorkommen.

Der Ausfallschritt ist mein Vorschlag und nicht abgestimmt — Tiefe und Unterstützungsgrad als
Progression. Der Liegestütz folgt der Stufenfolge aus dem Handoff, nur feiner unterteilt.

Die Seitstütz-Leiter ist **gerechnet**, nicht geschätzt: Das Biegemoment an der Taille
(Segmentmodell, Schnitt L4/L5) fällt in jeder Schräglage mit cos(Neigungswinkel). Zwei Folgen,
die dem üblichen Rat widersprechen:

- **Unterarm erhöhen** ist die saubere Regression — der Hebel bleibt, nur der Winkel dreht sich.
  Messbar in Zentimetern.
- **Füße erhöhen ist keine Progression.** Es verschiebt Last auf die Stützschulter (die im
  flachen Seitstütz schon rund 70 % des Körpergewichts trägt); das Rumpfmoment sinkt dabei sogar
  leicht. Die Stufe steht deshalb bewusst nicht in der Leiter.

Oberhalb des Standards bleibt nur Zusatzlast, und die wirkt nur nah an der Taille: 10 kg an der
Hüfte sind +25 %, dieselben 10 kg am Brustkorb +3 %. Darum steht der Ort in der Bedienung.

**Der Schrägzug folgt demselben Gesetz.** Last = 0,746 · cos θ, wobei θ der Körperwinkel über der
Waagerechten ist — gegen vier publizierte Kraftmessungen am Gurt geprüft, größte Abweichung 5,1 %.
Die Stufen sind deshalb **Winkelmarken**, getroffen über Fußabstand (trägt bis etwa 37°) und
Gurtlänge (übernimmt ab dort). Und damit zum zweiten Mal: **Füße erhöhen ist keine Progression.**
Es dreht dich nur zur Waagerechten, und dort hat cos θ sein Maximum — darüber hinaus fällt die
Last wieder. Oberhalb geht es nur mit Zusatzgewicht weiter.

Der **einarmige** Schrägzug ist keine Stufe davon, sondern eine andere Übung: rund 1,4
Körpergewichte pro Arm plus ein Drehmoment um die Längsachse, das der Rumpf gegenhält.

Die **Fersenbrücke** skaliert über den Fersenabstand — wie weit die Ferse vom Gesäß weg steht.
Kurzer Hebel leicht, langer Hebel schwer; die „Long-Lever-Brücke" ist keine zweite Übung, sondern
das obere Ende derselben Leiter. Die unteren vier Stufen sind von einem zweiten, unabhängigen
Segmentmodell bestätigt. **Faszienrolle und Ringe sind geschätzt** und stehen in der App als
solche gekennzeichnet: Ob die Rolle die Last wirklich hebt oder nur den Stabilisierungsaufwand,
ist offen — bei gleicher Fersenhöhe ist das Hüftmoment dasselbe. Das ist genau das Muster, an dem
beim Seitstütz die Fußerhöhung als Scheinstufe aufgeflogen ist, und steht deshalb als Erstes auf
der Liste der Dinge, die nachgemessen gehören.

**Zusatzgewicht ist eine eigene Achse, keine Leitersprosse.** Die Leiter bleibt reines
Körpergewicht; Kilogramm kommen ab einer Stufe je Übung dazu (Liegestütz ab „Boden",
Ausfallschritt ab „Hinterer Fuß ~20 cm erhöht", Seitstütz ab „Standard"). Wie stark ein Kilo
wirkt, ist übungsabhängig: Bei Liegestütz und Ausfallschritt wächst die Last mit `X/BW`, beim
Seitstütz mit `2·X/BW` — dort ist die Last ein Biegemoment und das Gewicht sitzt auf der Hüfte,
dem wirksamsten Hebelpunkt. Beim Schrägzug sind es `1,3·X/BW` (Rucksack hoch auf den
Schulterblättern; rutscht er aufs Becken, wirkt dasselbe Gewicht nur zu drei Vierteln), bei der
Fersenbrücke `1,4·X/BW` (Gewicht auf dem Becken, dicht am Hüftgelenk). Dafür braucht die App das Körpergewicht: *Mehr → Messung*.

---

## Was der Prototyp nicht kann

- **Die Lastfaktoren selbst lernen.** Das braucht mehrere hundert Nutzer und einen Server
  (Handoff 4.2). Ein einzelner Nutzer kann seine eigene Lastskala nicht bestimmen — bei drei
  Messungen und zwei Modellparametern bleibt dafür rechnerisch zu wenig übrig.
- **Zuverlässig bei gesperrtem Display weiterlaufen.** Eine Webseite darf das nicht. Genau
  darum geht es in Abschnitt 6.1 des Handoffs, und genau deshalb wird die echte App
  wahrscheinlich nativ.
- Konten, Sync, Push, Videos, Bezahlung — alles außerhalb des MVP.

---

## Datensicherung

**Mehr → Sichern (Datei)** legt eine JSON-Datei ab. Das ist die einzige Sicherung, die es gibt:
Leerst du die Browserdaten oder wechselst das Gerät, sind die Messwerte weg. Vor jedem
Browser-Aufräumen einmal sichern.

---

*Kein Medizinprodukt. Keine Diagnose, keine Therapieempfehlung.
Prototyp vom 13.09.2026, installierbar seit 14.09.2026 · Luke Brandenburg*
