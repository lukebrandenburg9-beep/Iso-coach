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
- **Gespreiztes Onboarding** über drei Stufen (leicht / mittel / schwer) statt einer.
- **Zonenrotation:** Die App fordert die Zone an, die am längsten zurückliegt — **und erholt
  ist**. Ohne Spreizung wird die Kurve wertlos, ohne Erholung die Messung.
- **Erholung:** Pro Tag genau **ein** Test, nie drei Zonen hintereinander. Danach wartet die
  Zone, bis sie wieder belastbar ist (Startwert zwei Tage, unter *Mehr → Erholung* änderbar).
  Der Tagesabstand gilt **übungsübergreifend**, die Zonenpause je Übung — weil die
  Energiesysteme überlappend arbeiten und nie isoliert. An einem gesperrten Tag zeigt die App
  einen Pausentag mit Begründung und Termin; übergehen lässt er sich bewusst.
- **Progressionsregeln 1–4** inklusive Rückstufung bei zweimaligem Schmerzabbruch.
- **Schmerz** als Häkchen pro Satz plus ein Ampelwert am Sessionende.
- **Sicherheitsregel:** dreimal in Folge Ampelwert 6 oder höher → Hinweis auf ärztliche Abklärung.
- Red-Flag-Abfrage im Onboarding, Timer mit Tonsignalen, 20-Sekunden-Pausen, Export/Import.

**Bewusst nicht drin:** CMF. Begründung steht in der App unter Profil → „Warum hier keine
Maximalkraft steht" und ausführlich in `../Modell-Befunde_Simulation.md`.

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
