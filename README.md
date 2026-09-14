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

Die Datei bringt alles mit, was sie zu einer installierbaren App macht: Symbol, Name und
Vollbildstart stecken bereits in der HTML-Datei. Was noch fehlt, ist eine Internetadresse,
unter der sie liegt.

### Warum der Artifact-Link dafür nicht reicht

Ein Artifact-Link öffnet immer die Claude-Oberfläche mit der App darin. Die lässt sich nicht
als eigenständige App installieren — in der Claude-App gibt es das Teilen-Menü dafür gar nicht,
und **Chrome auf dem iPhone kann „Zum Home-Bildschirm" grundsätzlich nicht**, das kann auf iOS
nur Safari. Der Artifact-Link ist gut zum schnellen Öffnen, nicht zum Installieren.

### Der Weg, der funktioniert

1. Auf **github.com** ein neues Repository anlegen, **Public**, mit Häkchen bei „Add a README".
2. **Add file → Upload files** → `index.html` hineinziehen → **Commit changes**.
3. **Settings → Pages** → unter „Branch" **`main`** und **`/ (root)`** wählen → **Save**.
4. Nach ein bis zwei Minuten liegt die App unter
   `https://<dein-github-name>.github.io/<repo-name>/`.
5. Diese Adresse **am Handy in Safari** öffnen (Android: Chrome) → Teilen → **„Zum
   Home-Bildschirm"**. Sie startet dann im Vollbild, ohne Browserleiste, mit eigenem Symbol.

Das Repository muss öffentlich sein, damit GitHub Pages ohne bezahltes Konto funktioniert.
Das ist unbedenklich: In der Datei stehen keine persönlichen Daten — die Messwerte entstehen
erst beim Benutzen und bleiben im Browser des jeweiligen Nutzers.

### Ohne GitHub

Jeder Webspace tut es, auf den du eine Datei legen kannst. Es muss **https** sein, sonst
verweigert das Handy die Installation. Eine Datei auf dem Handy selbst (per AirDrop in die
Dateien-App) lässt sich nicht installieren und verliert unter iOS außerdem gespeicherte Daten.

---

## Was drinsteckt

Der Prototyp setzt Abschnitt 4 des Handoffs um — er ist kein Timer mit Deko, sondern die
Rechenlogik:

- **Die Kurve** `t = W'/(P − CP)`, gefittet über eine eindimensionale Suche, ältere Messpunkte
  werden mit 3 % je neuerem Punkt abgewertet.
- **Gespreiztes Onboarding** über drei Stufen (leicht / mittel / schwer) statt einer.
- **Zonenrotation:** Die App fordert immer die Zone an, die am längsten zurückliegt. Ohne das
  verliert die Kurve ihre Spreizung und wird wertlos.
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
