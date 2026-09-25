# ISO Coach — Probleme und Änderungen

**Für Raffa · Stand 25.09.2026 · Luke Brandenburg**

Ergänzung zum technischen Handoff. Dieses Dokument hält fest, was beim Bau des ersten
Prototyps aufgefallen ist — an Rechenlogik, an Physiologie und an der Struktur der Übungen.
Vieles davon war im Handoff so nicht vorgesehen und verschiebt den Aufwand an Stellen, die
dort noch harmlos aussehen.

**Der Prototyp ist ein Denkwerkzeug, kein Produktvorschlag.** Eine einzelne HTML-Datei,
offline lauffähig, ohne Konto und ohne Server. Er existiert, um die Rechenlogik an echten
Eingaben zu prüfen, bevor sie in eine richtige App wandert.

- Live: https://lukebrandenburg9-beep.github.io/Iso-coach/
- Quelle: https://github.com/lukebrandenburg9-beep/Iso-coach
- Dieses Papier im Repo: https://github.com/lukebrandenburg9-beep/Iso-coach/blob/main/ISO-Coach_Probleme-und-Aenderungen.md
  (die Kopie wird mit jedem Build nachgezogen; der technische Handoff liegt nicht im Repo)

---

## Teil 1 — Probleme, die den Aufwand verschieben

### P1 · Das Rechenmodell ist nicht geheim

Im Handoff stand als Problemstellung Nr. 1, „den mathematischen Algorithmus hinter der Force
Curve zu verstehen und nachzubauen". Das ist erledigt, bevor es begonnen hat: Was dort
beschrieben wird, ist das **Critical-Power-Modell**, publiziert seit Monod & Scherrer 1965,
in der Drei-Parameter-Form von Morton 1996.

```
zwei Parameter:  t = W' / (P − CP)
drei Parameter:  t = W' / (P − CP) − W' / (Pmax − CP)
```

`CP` ist die Asymptote, unterhalb derer theoretisch unbegrenzt gehalten wird. `W'` ist der
endliche Vorrat darüber. Die drei Zonen des Konzepts sind drei Abschnitte auf **einer**
Hyperbel, keine drei Modelle.

Erprobte Open-Source-Implementierungen: GoldenCheetah (GPL, C++, `cp2`- und `cp3`-Fits),
`cyclingtools` (R), PyTindeq (Python, BLE-Kraftmesser).

**Für die Schätzung:** Keine Forschungsleistung, eine Implementierungsaufgabe. Der Fit im
Prototyp ist eine eindimensionale Suche über `CP` mit analytisch eliminiertem `W'` — rund
30 Zeilen.

### P2 · Die Lastskala ist nicht identifizierbar

Aus reinen Haltezeiten sind nur **Verhältnisse** bestimmbar, keine Kilogramm. Schärfer: Die
Skala ist nur bis auf eine **affine Transformation** bestimmt. Setzt man `L → a·L + b` und
transformiert `CP` identisch, bleiben alle Haltezeiten unverändert.

Praktisch heißt das:

- **Ein** Ankerpunkt mit bekanntem Gewicht genügt nicht. Es braucht **zwei**.
- Ein einzelner Nutzer kann seine eigenen Lastfaktoren **nicht** lernen. Bei `K` Messungen und
  zwei Nutzerparametern bleiben `K−2` Freiheitsgrade für die Lastschätzung. Bei `K = 2` ist
  das Problem exakt flach — nicht ungenau, sondern unlösbar.
- Das Lernen der Lastfaktoren aus Daten (Handoff 4.2) braucht eine **Population**, die
  mehrere Varianten übergreifend trainiert. Das ist Item Response Theory und setzt Server,
  Konten und mehrere hundert Nutzer voraus.

**Für die Schätzung:** Im MVP sind die Lastfaktoren feste, redaktionell gesetzte Startwerte.
Das Lernen ist ein eigenes, späteres Vorhaben — nicht Teil des ersten Releases.

### P3 · CMF ist numerisch nicht tragfähig

Das Vorbild zeigt eine „Consolidated Max Force" — den rechnerischen Schnittpunkt der Kurve
mit der Nulllinie. Dieser Wert liegt **außerhalb** des gemessenen Bereichs. In der Simulation
streute er um den Faktor **1,4 bis 1,7**, und zwanzig Messungen machten ihn kaum besser als
fünf.

Zwei unabhängige Bestätigungen: Grip Gains erklärt die eigene Instabilität dieses Werts zum
Merkmal, und der bekannteste Community-Nachbau lässt CMF weg.

**Entscheidung:** Der Prototyp zeigt CMF nicht. Wenn ein Maximalkraftwert gewünscht ist, muss er
**gemessen** werden (Kraftmesser), nicht extrapoliert.

Die Begründung stand bis zum 17.09. als aufklappbarer Kasten im Profil und ist dort wieder raus —
Lukes Einwand: *„Das sind Interna, die der Endkunde eh nicht versteht."* Er hat recht, und zwar
strenger, als es zunächst klingt: Der Kasten erklärte, warum eine Zahl **fehlt**, die der Nutzer
nie gesehen hat und in keiner anderen App vermisst. Er beantwortete eine Frage, die nur stellt, wer
das Vorbild kennt.

### P4 · Die Feinheit der Stufen entscheidet, welche Zonen erreichbar sind

Mit den fünf groben Stufen aus dem Handoff war **Zone B (60–90 s) für einen durchschnittlichen
Nutzer gar nicht erreichbar** — der Sprung von Stufe zu Stufe übersprang sie (150 s → 56 s).

Da Zonenabdeckung die Vorbedingung für eine belastbare Kurve ist, hätte das verhindert, dass
überhaupt ein Profil entsteht. Der Prototyp nutzt deshalb **acht Stufen** mit einem
Lastverhältnis von rund **1,15** zwischen benachbarten Stufen.

**Nachgerechnet (14.09.2026).** Für eine Leiter lässt sich ausrechnen, welcher Anteil möglicher
Nutzer überhaupt alle drei Zonen erreichen kann. Entscheidend ist das Verhältnis benachbarter
Stufen, nicht ihre Zahl:

| Verhältnis benachbarter Stufen | Nutzer mit allen drei Zonen |
|---|---|
| 1,42 | 32 % |
| 1,29 | 47 % |
| 1,22 | 57 % |
| 1,15 | 69 % |
| 1,10 | 78 % |

Die Leitern im Prototyp erreichen **71 %** (Liegestütz) und **62 %** (Ausfallschritt). Für den
Rest sitzt `CP` zu weit oben in der Leiter — es bleiben zu wenige Stufen darüber. Das lässt sich
**nicht** durch mehr Positionen lösen, sondern nur durch **Zusatzlast**: Ein Rucksack verstellt
stufenlos und ist damit die eigentliche Feinjustierung.

**Für die Schätzung:** Die Stufenliste ist kein Redaktionsdetail, sondern ein Parameter der
Rechenlogik. Sie gehört mit der Zonendefinition zusammen geprüft, und die App braucht ein Feld
für Zusatzlast.

### P5 · Spreizung schlägt Anzahl

Zwei Messungen in **derselben** Stufe ergeben keine Kurve — das ist keine Ungenauigkeit,
sondern rechnerisch nicht lösbar. Daraus folgt zweierlei:

- Das Onboarding muss **mehrere verschiedene** Stufen messen, nicht eine.
- Die laufende Planung muss Spreizung **erzwingen**, nicht hoffen. Der Prototyp fordert immer
  die Zone an, die am längsten zurückliegt.

### P6 · Die drei Zonen sind keine drei Energiesysteme

Alle drei Zonen liegen per Modelldefinition **oberhalb** von CP. Es sind drei Geschwindigkeiten,
mit denen derselbe Vorrat `W'` geleert wird — nicht drei getrennte Systeme. Der anaerob/aerobe
Kreuzungspunkt liegt bei etwa 75 Sekunden, also mitten in Zone B.

Folge für die Erholungssteuerung: Die Energiesysteme erholen sich in **Minuten**, nicht Tagen.
PCr etwa 96 % nach 10 Minuten, `W'` zu 86 % nach 15 Minuten.

**Offene Fachentscheidung.** Luke hat als Richtwert gesetzt: dieselbe Muskelgruppe dreimal pro
Woche, jede Zone einmal pro Woche. Die Frequenz ist gut gestützt, die Wochenpause je Zone nicht.
Die Recherche dazu liegt in `Recherche_Erholung-Isometrie.md` (69 Quellen). Der Prototyp setzt
Lukes Zahlen um und macht sie editierbar.

### P7 · Drei Zonenetiketten sind nicht gedeckt

Aus derselben Recherche, unabhängig von der Frequenzfrage:

- **„Hypertrophie"** für Zone B — Hypertrophie tritt über den ganzen Lastbereich auf.
- **„Sehnenkonditionierung"** für Zone B — die einschlägige Studie zeigt das **Gegenteil**:
  Bei gleichem Volumen schlug **3 Sekunden Halten 12 Sekunden Halten deutlich**
  (+57 % gegen +25 % Sehnensteifigkeit). „Lange halten ist gut für die Sehne" ist die Umkehrung
  des Gemessenen.
- **Die %-MVC-Zuordnung stimmt nicht.** Gemessen hält man bei 70 % MVC 86–134 s und bei 50 %
  MVC 148–190 s. Zone A (20–60 s) verlangt also **über 80 % MVC**, Zone C (90–150 s) liegt bei
  **55–70 %** — nicht bei 30–50 %.

Fürs Rechenmodell folgenlos, für **Texte und Begründungen in der App** nicht.

### P8 · Der Ausfallschritt misst kein Bein, sondern ein Paar

Das strukturell schwerste Problem, und es betrifft jede Übung, die beidseitig belastet.

Ein Ausfallschritt verteilt grob **60 % auf das vordere und 40 % auf das hintere Bein**.
Daraus folgen drei Dinge:

1. **Seitenasymmetrie.** Die Zielgruppe hat Gelenk- und Sehnenbeschwerden, und die sind fast
   immer **einseitig**. Links und rechts brauchen **getrennte Kurven** — in einer gemeinsamen
   verschwindet genau der Seitenunterschied, der klinisch am meisten aussagt.
2. **Übertragene Ermüdung.** Wer zuerst links vorne misst und dann rechts vorne, misst rechts
   ein Bein, das eben schon als hinteres Bein gearbeitet hat — im **selben Muskel**, nur bei
   anderem Hüftwinkel.
3. **Die 40 % sind kein Lastfaktor.** Die Position des hinteren Beins ist eine andere, also
   ist das keine saubere Skalierung derselben Übung. Man kann es nicht wegrechnen.

**Lösung im Prototyp:** Ein Workout besteht aus **drei Blöcken** — Ausfallschritt frische
Seite, Liegestütz, Ausfallschritt zweite Seite. Der Liegestütz steht bewusst in der Mitte und
gibt dem ersten Bein fünf bis acht Minuten Erholung. Gemessen wird nur die frische Seite; die
zweite wird trainiert, aufgezeichnet und als *vorermüdet* geführt, geht aber **nicht in den
Fit**. Die Startseite wechselt je Workout.

**Für die Schätzung:** Das Datenmodell braucht `seite` und ein `frisch`-Kennzeichen je
Satzblock, und der Trainingsablauf ist nicht „eine Übung pro Einheit", sondern eine **Folge von
Blöcken**. Das ist keine Kleinigkeit im UI.

**Der Preis, ehrlich gerechnet:** Drei Workouts pro Woche, drei Zonen, zwei Seiten — jede
Seiten-Zonen-Kombination wird **alle zwei Wochen** einmal gemessen. Ein vollständiges
Seitenprofil braucht entsprechend lange.

### P9 · Zustand gehört an die Kurve, nicht an den Nutzer

Ein Fehler, der erst beim Umbau sichtbar wurde: Aktuelle Stufe, Schmerzserie und Fehlserie
lagen global am Nutzer. Damit stufte eine Schmerzserie am **Knie** die **Schulter** mit zurück.

Richtig ist: Jede Messreihe — Liegestütz, Ausfallschritt links, Ausfallschritt rechts — hat
ihren eigenen Stand.

### P10 · Die Zone eines Messpunkts ist die erreichte, nicht die geplante

Wer eine Stufe selbst wählt oder den Richtwert deutlich verfehlt, landet in einer anderen Zone
als geplant. Wird die geplante Zone gespeichert, sperrt die Erholungslogik anschließend die
falsche Zone und die Zonenabdeckung stimmt nicht.

### P11 · Der Richtwert darf keine Prüfung sein

Sobald der Nutzer die Stufe selbst wählt, ist „unter dem Richtwert" **kein Fehlschlag** — es
ist eine Schätzung, die danebenlag, und damit ein ganz normaler Messpunkt. Die Rückstufungsregel
bei dreimaligem Verfehlen darf in diesem Fall nicht greifen. Die **Schmerzregel** bleibt davon
unberührt; sie ist eine Sicherheitsregel.

### P12 · Widersprüche im Handoff zur Schmerzerfassung

Abschnitt 2.4 (Ampel 0–10 steuert alles), M5 (nur Häkchen), 6.2 (24-h-Push, „ohne die
funktioniert die Progressionslogik nicht") und Abschnitt 7 (Ampelwert 6+) widersprachen sich
gegenseitig.

**Geklärt:** Häkchen pro Satz („wegen Schmerz abgebrochen") **plus** ein Ampelwert 0–10 am Ende
des Workouts. Der 24-Stunden-Push ist gestrichen. Abschnitt 7 ist auf den Wert am Workoutende
umgeschrieben.

Nachtrag aus der Recherche: Der gestrichene 24-Stunden-Check ist ausgerechnet die Größe, die
die Reha-Literatur zur Erholungssteuerung vorsieht. Er ließe sich **ohne Push** umsetzen — als
eine Frage beim Start der nächsten Einheit. Das ist eine offene Entscheidung.

### P13 · Was eine Webseite nicht kann

- **Timer bei gesperrtem Display.** Eine Webseite darf das nicht zuverlässig. Genau darum geht
  es in Handoff 6.1 — und genau deshalb wird die echte App wahrscheinlich nativ.
- **Installieren über einen geteilten Link.** Ein Artifact- oder Vorschaulink öffnet eine
  fremde Oberfläche und ist nicht installierbar. Es braucht eine eigene https-Adresse.
- **„Zum Home-Bildschirm" in Chrome auf dem iPhone.** Kann iOS grundsätzlich nicht, nur Safari.
- **Speicher.** `localStorage` hängt an Adresse und Gerät. Kein Gerätewechsel, kein Backup außer
  Export. Für Gesundheitsdaten auf Dauer zu wenig.

### P14 · Eine Stufe muss über eine nachmessbare Größe definiert sein

Beim ersten echten Test fielen beide Stufenlisten durch:

- **„An der Wand, aufrecht"** erzeugt keinen Kraftvektor nach vorne. Wer aufrecht an der Wand
  steht, hat **keine Last auf den Händen** — die Stufe misst nichts. Die Last entsteht erst durch
  den Abstand der Füße zur Wand.
- **„Beide Hände abgestützt"** beim Ausfallschritt hat zwei Fehler auf einmal: Die Handstütze
  belastet genau die Muskulatur, die im nächsten Block den Liegestütz halten soll, und sie ist
  **nicht normierbar** — mit wie viel Kilogramm stützt man sich ab? Das weiß niemand.

**Regel daraus:** Jede Stufe muss über eine Größe definiert sein, die der Nutzer **nachmessen**
kann. Umgesetzt als Handhöhe und Fußabstand in **Fußlängen** (skaliert mit der Körpergröße) beim
Liegestütz, und als Höhe eines Gegenstands unter dem hinteren Knie beim Ausfallschritt.

### P15 · Die Höhe der Abstützfläche hat oben kaum Spielraum

Eine Leiter aus immer niedrigeren Ablagen sieht plausibel aus, trägt aber nicht. Gemessene
Handlasten im Liegestütz (Suprak et al. 2011): Boden rund **69 %** des Körpergewichts, Knie
abgelegt rund **53 %**, Ablage auf 61 cm rund **55 %**.

Von einer 60-Zentimeter-Ablage bis zum Boden ändert sich die Last also nur um den Faktor **1,25**
— über mehrere Stufen hinweg. Nach P4 ist das viel zu fein, um Zonen zu trennen.

**Die schweren Stufen müssen deshalb über erhöhte Füße laufen, nicht über niedrigere Flächen.**

### P16 · Erklärungen dürfen die Pause nicht auffressen

Der Ablauf zeigte nach jedem Satz erst eine Erklärung und die Schmerzfrage und **startete danach**
die 20 Sekunden Pause. Damit war die Pause faktisch so lang, wie der Nutzer zum Lesen brauchte.

Das ist kein Schönheitsfehler: Der **Abfall der Haltezeit über die drei Sätze ist selbst ein
Messwert** (W'-Rekonstitution, Handoff 3.1). Bei ungleich langen Pausen ist er wertlos.

**Regel:** Alles, was den Messablauf taktet, läuft unabhängig von Eingaben des Nutzers. Fragen
stehen **neben** der laufenden Uhr, nie davor.

Dazu gehört auch der umgekehrte Fall: Ohne Vorlauf beginnt die Messung, während der Nutzer sich
noch in die Position aufbaut. Der Prototyp zählt jetzt **drei Sekunden** herunter, bevor die Uhr
läuft.

### P17 · Die Reaktionszeit sitzt in jeder Messung drin

Zwischen dem Moment, in dem der Nutzer die Position verlässt, und dem Knopfdruck vergehen ein bis
drei Sekunden. Die landen als **Haltezeit** in der Messung.

Systematisch, nicht zufällig: Der Fehler liegt immer in dieselbe Richtung und trifft jeden Satz.
Bei einem 30-Sekunden-Satz sind zwei Sekunden **sieben Prozent**, bei einem 150-Sekunden-Satz
eineinhalb. Der Fehler verzerrt also nicht nur das Niveau, sondern **die Form der Kurve** — kurze
Halte werden relativ stärker überschätzt als lange, und genau daraus schätzt das Modell `CP`
und `W'`.

**Umsetzung im Prototyp:** Ein einstellbarer Abzug je Satz (Voreinstellung 2 s). Weil er bei jedem
Satz gleich groß ist, bleiben die Verhältnisse zwischen den Sätzen unberührt.

**Für die echte App:** Sauberer wäre, den Nutzer die Uhr **nicht** per Knopf beenden zu lassen —
ein Kraftmesser oder eine Bewegungserkennung setzt den Endpunkt selbst. Solange der Knopf der
Endpunkt ist, gehört der Abzug konfiguriert und **dokumentiert**, denn er verschiebt alle
absoluten Werte.

### P18 · Vorgaben für Folgesätze müssen mitlaufen

Die Vorgabe für Satz 3 wurde aus Satz 1 mit einem **festen** Faktor gerechnet (0,30). Wer in
Satz 2 deutlich über oder unter der Vorgabe lag, bekam in Satz 3 trotzdem denselben Wert.

Das ist nicht nur unbequem — es verschenkt eine Messung. Der **Abfall über die Sätze ist die
W'-Rekonstitution** und personenabhängig. Ein fester Faktor setzt voraus, was eigentlich gemessen
werden soll.

**Umsetzung:** Satz 2 richtet sich nach der tatsächlich erreichten Zeit aus Satz 1; ab Satz 3 wird
der zwischen den letzten beiden Sätzen **beobachtete** Abfall fortgeschrieben, begrenzt auf 0,35
bis 0,85.

### P19 · Vorlauf gehört nur an den Anfang eines Blocks

Der Drei-Sekunden-Vorlauf lief vor **jedem** Satz. Nach der 20-Sekunden-Pause zählte die App also
zweimal hintereinander herunter, obwohl der Nutzer die Pause längst zum Aufbauen genutzt hatte.

**Regel:** Vorlauf vor Satz 1 eines Blocks, danach nicht mehr.

---

### P20 · Beim Seitstütz ist Fuß-Erhöhung keine Progression

Der übliche Rat lautet: Füße auf eine Bank, dann wird es schwerer. Für den Rumpf stimmt das nicht.

Das Biegemoment an der Taille entsteht aus der Komponente der Schwerkraft, die **quer** zur
Körperlängsachse steht. Neigt man den Körper um den Winkel θ, schrumpft diese Komponente mit
cos θ — und zwar unabhängig davon, ob der Unterarm oder die Füße höher liegen. Rechnet man ein
Segmentmodell durch (Massen und Schwerpunkte nach Winter, Schnitt auf Höhe L4/L5), kommt heraus:

| Lage | Winkel | Rumpfmoment |
|---|---|---|
| Unterarm und Füße am Boden | 0° | 100 % |
| Füße auf einem Stuhl (~45 cm) | 18° | **95 %** |
| Unterarm auf einer Arbeitsplatte (~90 cm) | 39° | **78 %** |

Füße erhöhen macht den Rumpf also *leichter*, nicht schwerer. Was tatsächlich zunimmt, ist die
Last auf der **Stützschulter** — im flachen Seitstütz trägt der Ellenbogen bereits rund 70 % des
Körpergewichts. Wer die Übung als Rumpfmessung führt, misst mit erhöhten Füßen zunehmend die
Schulter.

**Konsequenz:** Die Stufe steht nicht in der Leiter. Oberhalb des Standards bleibt nur Zusatzlast
— und die wirkt nur nah an der Taille: 10 kg an der Hüfte sind +25 %, dieselben 10 kg als
Rucksack am Brustkorb +3 %. Deshalb steht der *Ort* der Last in der Stufenbezeichnung.

**Zum zweiten Mal, beim Schrägzug (17.09.).** Dieselbe Rechnung, dieselbe Konsequenz: Die Last am
Schlingentrainer ist **0,746 · cos θ**, wobei θ der Körperwinkel über der Waagerechten ist. Gegen
publizierte Kraftmessungen am Gurt geprüft:

| Körperwinkel | Modell | gemessen |
|---|---|---|
| 60° | 0,373 | 0,374 ± 0,015 |
| 45° | 0,528 | 0,529 ± 0,006 |
| 30° | 0,646 | 0,681 ± 0,020 |
| waagerecht | 0,746 | 0,695 ± 0,030 |

Größte Abweichung 5,1 %. Und damit gilt auch hier: **Füße erhöhen ist keine Progression.** Es dreht
den Körper nur zur Waagerechten, und dort hat cos θ sein Maximum; darüber hinaus **fällt** die Last
wieder. Gemessen: Füße auf dem Gerät statt am Boden bringen 73,3 % statt 69,5 % — +5,5 % relativ,
keine Stufe. Auch die Stange tiefer zu hängen trägt nur, solange sie den Winkel ändert: von 1,45 m
auf 1,10 m gewinnst du 65 %, von 75 cm auf 44 cm noch 3 %.

**Die Regel dahinter:** Neigung skaliert nur, solange sie den Winkel zur Schwerkraft ändert — nicht,
wenn sie ihn schon ausgereizt hat. Wer eine Leiter aus Erhöhungen baut, sollte vorher ausrechnen,
wo cos θ sein Maximum hat.

---

### P21 · Die Knieversion misst nicht dasselbe wie die Fußversion

Knie statt Füße verkürzt den Hebel und senkt das Rumpfmoment auf 63 %. Das ist der größte
Einzelschritt der Leiter — und zugleich ihr Bruch.

Im EMG-Vergleich Knie- gegen Fußstütz sinkt mit gebeugten Knien der Anteil der Hüftabduktoren,
während die schrägen Bauchmuskeln relativ führend werden. Die Stufe ist also nicht bloß leichter,
sie belastet anders. Wer über diesen Sprung hinweg eine Kurve legt, verbindet zwei Messungen, die
nicht dieselbe Größe erfassen.

**Was das praktisch heißt:** Ein Nutzer, der nur Kniestufen macht, und einer, der nur Fußstufen
macht, sind über diese Leiter nur eingeschränkt vergleichbar. Für die datengetriebene
Stufenschätzung (P2) ist das eine Verkettung, die schlechter trägt als die bei Liegestütz und
Ausfallschritt, wo alle Stufen dieselbe Muskulatur limitiert.

**Offen:** ob Knie- und Fußversion zwei getrennte Kurven sein sollten statt einer.

---

### P22 · Die Einstiegsphase hing am Workout, nicht an der Kurve

Das Onboarding zählte **Workouts**: nach drei Stück galt die Einstiegsphase für alles als
erledigt. Solange die App mit zwei Übungen startete, fiel das nicht auf.

Mit dem Seitstütz fällt es auf. Eine Übung, die später dazukommt, hätte nie eine Einstiegsphase
bekommen — sie wäre auf der leichtesten Stufe gestartet und hätte wochenlang Punkte aus derselben
Zone geliefert, aus denen sich keine Kurve fitten lässt (P5: Spreizung schlägt Anzahl).

**Erster Versuch, und warum er falsch war:** naheliegend ist ein Zähler — weniger als drei
frische Punkte, also Einstiegsphase, also C → B → A. Im Test an einem realistischen Datenstand
fiel auf, dass das die **Ausfallschritt-Seiten** mit in die Einstiegsphase zieht. Jede Seite wird
nur in jedem zweiten Workout frisch gemessen und hat nach dem Onboarding daher erst zwei Punkte —
aber zwei *gespreizte*. Der Zähler hätte sie auf eine Zone geschickt, die längst belegt ist.

Was einer Kurve fehlt, ist eben nie „noch ein Punkt", sondern immer eine bestimmte **leere Zone**.
Genau das sagt P5 schon, nur stand es nicht im Code.

**Regel:** Maßstab ist die Zonenabdeckung der Kurve. Die Rotation bleibt der Normalfall;
abgewichen wird nur, wenn sie eine Zone anfordert, die diese Kurve **schon hat**, während eine
andere leer ist. Dann wird stattdessen die leere Zone gemessen. Eine neue Übung trifft das in
jedem Workout, bis ihre drei Zonen stehen; eine eingefahrene nie.

**Preis:** In einer Sitzung laufen damit unterschiedliche Zonen nebeneinander — der neue
Seitstütz bei 120 s, der eingefahrene Liegestütz bei 35 s. Lukes Kriterium für gleichzeitiges
Testen war *keine überlappenden Muskelgruppen*, und das ist erfüllt. Ob dieselbe Sitzung trotzdem
ein Energiesystem doppelt belastet, ist damit nicht beantwortet — siehe P6. Der Preis fällt jetzt
aber nur noch dort an, wo die Abweichung auch etwas einbringt.

---

### P23 · Fünf Blöcke sind eine lange Sitzung

Drei Übungen, davon zwei zweiseitig, ergeben fünf Blöcke zu je drei Sätzen: **fünfzehn Haltezeiten
plus Pausen** in einer Sitzung. Bei Zielzeiten in Zone C sind das leicht über vierzig Minuten
reine Haltezeit.

Der Liegestütz steht deshalb exakt in der Mitte: Er trennt bei **beiden** zweiseitigen Übungen die
erste von der zweiten Seite, sodass zwischen Seite eins und Seite zwei immer zwei volle Blöcke
liegen.

**Beantwortet in P34:** Sie passt nicht. Ab der vierten Übung entscheidet eine Rotation mit
Deckel, welche der freigegebenen Übungen heute drankommen.

---

### P24 · Erholung je Workout trägt nur, solange das Workout fest ist

Die Erholung wurde zwischenzeitlich auf **Workout-Ebene** gezählt: Ein Workout enthielt ohnehin
alle Muskelgruppen, also genügte der Abstand zwischen zwei Workouts. Das Feld `group` blieb in
den Übungsdaten stehen, ohne benutzt zu werden.

Sobald sich Übungen einzeln zu- und abwählen lassen, ist die Annahme falsch. Wer vormittags
Liegestütz und Ausfallschritt gemacht hat, hat den **Rumpf nicht belastet** — eine Sperre für den
Seitstütz wäre frei erfunden, und zwar in der teuersten Richtung: Sie verhindert Training, das
fachlich zulässig ist.

**Regel:** Gezählt wird wieder je Muskelgruppe — Tagesabstand, Zonenrotation und Einstiegsphase
alle drei. `push` · `legs` · `core` sind überschneidungsfrei, und genau das ist Lukes Kriterium
fürs Testen am selben Tag. Der Startbildschirm zeigt je Übung, ob sie frei ist oder wie lange
ihre Gruppe noch pausiert — und trennt das sichtbar vom Abwählen: **Pause ist keine Abwahl.**

**Folge für die Zonen:** Ein Workout hat damit nicht mehr *eine* Zone. Jede Gruppe fordert die
Zone an, die bei ihr am längsten zurückliegt. Der Startbildschirm zeigt entsprechend so viele
Zonen-Marken, wie im Workout tatsächlich vorkommen.

---

### P25 · Eine einzelne zweiseitige Übung hat keine Zwischenzeit

Die zweite Seite wird aufgezeichnet, aber nicht in die Kurve gelegt — begründet über die
Erholung, die der Block dazwischen verschafft (P8). Läuft nur **eine** zweiseitige Übung im
Workout, gibt es keinen Block dazwischen: Die zweite Seite folgt direkt auf die erste, und die
Begründung fürs Verwerfen ist damit eine andere als vorher.

Gleichzeitig wiegt das Verwerfen schwerer. Bei fünf Blöcken kostet es zwei von fünf Messungen,
bei einer einzelnen Übung die **Hälfte**.

**Umsetzung:** Die Voreinstellung bleibt *nicht messen*. Auf der Blockkarte steht ein Häkchen,
mit dem fachlichen Unterschied daneben — beim Seitstütz arbeitet die andere Rumpfseite mit
eigenen Muskeln, beim Ausfallschritt war dasselbe Bein eben schon hinteres Bein. Die Entscheidung
gehört Luke, nicht der Voreinstellung.

---

### P26 · Das Onboarding konnte sich nicht aus dem Stand bringen

Der schwerste Fehler bisher, und er ist erst im echten Training aufgefallen.

Die Stufenwahl lief so: Gibt es eine gefittete Kurve, wird die Stufe gesucht, deren erwartete
Haltezeit der Zielzeit am nächsten kommt. Gibt es keine, bleibt es bei der gespeicherten Stufe —
beim ersten Mal also **Stufe 1**, die leichteste.

Das ist ein Deadlock:

1. Ohne Kurve bleibt die Stufe stehen.
2. Bei stehender Stufe entstehen nur Messpunkte bei **derselben Last**.
3. Aus Punkten bei einer einzigen Last lässt sich `CP` und `W'` nicht trennen — die Gleichung
   `t = W'/(L − CP)` hat bei nur einem `L` unendlich viele Lösungen.
4. Also entsteht keine Kurve. Zurück zu 1.

Auch **Regel 4** („Stufe rauf, wenn die Stufe über 150 s trägt") half nicht: Sie braucht selbst
einen Fit.

Das Onboarding sollte genau das verhindern — aber es spreizte die **Zielzeiten** (120 / 75 / 35 s)
statt der **Lasten**. Bei gleicher Stufe kommt dreimal ungefähr dieselbe Haltezeit heraus, ganz
gleich welche Zahl danebensteht. Drei Messungen, drei Punkte übereinander. Die README versprach
„gespreiztes Onboarding über drei Stufen (leicht / mittel / schwer)" — die Absicht war richtig
formuliert, der Code tat etwas anderes.

Beim Ausfallschritt wiegt es doppelt: Jede Seite wird nur in jedem zweiten Workout frisch
gemessen. Nach zwei Workouts hat jede Seite **einen** Punkt — für `fit()` zu wenig, und selbst
der dritte hätte nichts geändert, weil er bei derselben Last gelegen hätte.

**Regel, jetzt:**

- **Kein Messpunkt vorhanden** → gespreizte Startstufen nach Einstiegsschritt: 20 % / 50 % / 75 %
  der Leiter. Nicht dreimal die leichteste.
- **Mindestens ein Messpunkt** → der jüngste Punkt derselben Übung dient als **Anker**, über
  beide Seiten hinweg und notfalls auch ein vorermüdeter Satz. Aus `(L₁, t₁)` und der Annahme
  `CP = c · L₁` folgt

  ```
  L_ziel = L₁ · [ c + (1 − c) · t₁ / t_ziel ]     mit c = 0,5
  ```

- **Zwei verschiedene Lasten vorhanden** → echter Fit, die Annahme fällt weg.

**Was daran gesetzt ist:** `c = 0,5`. Das ist keine Messung. Die Blockkarte sagt das im Klartext,
statt die Zahl als Ergebnis auszugeben.

**Was die Formel nicht kann:** Lag die Ankerstufe **unterhalb** der Dauerleistung, war die
Haltezeit nicht durch die Kapazität begrenzt, sondern durch Geduld oder Beschwerden. Dann ist
`t₁` kein Kapazitätsmaß und die Hochrechnung liefert eine Zahl ohne Deckung. Genau dieser Fall
ist bei der leichtesten Stufe der wahrscheinlichste.

**Sprungbegrenzung:** ohne Kurve höchstens drei Stufen auf einmal. Bei Gelenk- und
Sehnenbeschwerden rechtfertigt eine Schätzung keinen Sprung über die halbe Leiter. Die Karte
nennt beide Zahlen — die gerechnete und die gedeckelte —, der Wähler lässt jede Stufe zu.

---

### P27 · Zusatzgewicht ist eine eigene Dimension, keine Leitersprosse

Zwei Leitern hatten das Gewicht eingebacken: der Ausfallschritt als „Hinterer Fuß erhöht +
Zusatzlast", der Seitstütz als „+ 5 / 10 / 15 kg auf der oberen Hüfte". Beides ist falsch
modelliert.

- **„+ Zusatzlast" ist keine Größe.** Wie viel? Die Stufe hat einen festen Lastfaktor (1,45), der
  ein bestimmtes Gewicht unterstellt, ohne es zu nennen. Wer 5 kg nimmt und wer 20 kg nimmt,
  landet im selben Messpunkt.
- **Kilogramm sind ohne Körpergewicht bedeutungslos.** Die Lastfaktoren sind Anteile des
  Körpergewichts. 10 kg sind bei 60 kg Körpergewicht ein Sechstel, bei 95 kg ein Zehntel — die
  Sprossen „+10 kg" mit festem Faktor 1,13 galten still für 80 kg.

**Regel:** Die Leiter bleibt reines Körpergewicht. Zusatzgewicht ist eine zweite, stufenlose
Achse in Kilogramm, angeboten ab einer Stufe je Übung — Liegestütz ab „Boden", Ausfallschritt ab
„Hinterer Fuß ~20 cm erhöht", Seitstütz ab „Standard". Darunter ist die Leiter das richtige
Werkzeug: Wer mehr Last braucht, geht erst eine Stufe hoch.

**Die Wirksamkeit ist übungsabhängig** — und das ist der Punkt, an dem die Modellierung zählt:

| Übung | 1 kg wirkt | warum |
|---|---|---|
| Liegestütz, Ausfallschritt | ×(1 + X/BW) | Die Last ist ein Anteil des Körpergewichts. Wer X kg dazunimmt, trägt BW+X; der Anteil bleibt. |
| Seitstütz | ×(1 + 2·X/BW) | Die Last ist ein **Biegemoment**. Ein Kilo auf der oberen Hüfte sitzt am wirksamsten Hebelpunkt und zählt doppelt. Dasselbe Kilo am Brustkorb wäre fast wirkungslos (P20). |

Deshalb steht der **Ort** der Last in der Bedienung, nicht im Kleingedruckten: Beim Seitstütz
entscheidet er über den Faktor 8 zwischen Hüfte und Brustkorb.

**Neu nötig:** das Körpergewicht, unter *Mehr → Messung*. Für die Lastfaktoren selbst wird es
nicht gebraucht — die sind Verhältniszahlen —, erst für die Umrechnung von Kilogramm.

**Nebenwirkung, bewusst:** Fällt die Stufe unter die Gewichtsgrenze zurück, wird das
Zusatzgewicht auf 0 gesetzt. Sonst wirkte eine Last weiter, deren Bedienfeld nicht mehr sichtbar
ist — unsichtbarer Zustand ist schlimmer als ein verlorener Wert.

---

### P28 · Der laufende Lauf war flüchtig — Datenverlust im echten Training

**Am 16.09.2026 ist Luke ein halbes Workout verloren gegangen.** Seitstütz gemessen, Pause,
dann Liegestütz — und das Gemessene war weg. Kein Bedienfehler, sondern drei Defekte, die alle
dasselbe bewirkten.

Der laufende Lauf (`RUN`) lag nur im Arbeitsspeicher. Gespeichert wurde **erst nach dem letzten
Satz des letzten Blocks**, zusammen mit dem Schmerzwert. Alles davor existierte nirgendwo sonst.

1. **Pause zwischen zwei Blöcken.** Genau das, was das Protokoll vorsieht — fünf bis acht Minuten
   Erholung zwischen erster und zweiter Seite. Auf dem Handy heißt das: Bildschirm aus, Tab im
   Hintergrund. iOS und Android räumen einen Hintergrund-Tab nach wenigen Minuten weg. Beim
   Zurückkommen lädt die Seite neu, `RUN` ist leer, das Workout ist verschwunden. **Die
   Blockstruktur der App hat den Datenverlust also selbst herbeigeführt.**
2. **„Satz verwerfen?"** verwarf den ganzen Lauf. Der Abbruchknopf führte zu
   `if(time===null){ RUN=null; render(); }` — samt aller vorher abgeschlossenen Blöcke. Der
   Bestätigungsdialog versprach einen Satz und nahm alles.
3. **Ein Tipp auf die untere Leiste** während des Laufs rief `renderHome()` auf. `RUN` blieb im
   Speicher, war aber über keinen Weg mehr erreichbar — es sah aus wie verworfen und war faktisch
   verloren.

**Behoben:**

- **Nach jedem Satz und jedem Block wird gesichert** (`S.lauf`). Ein unterbrochener Lauf erscheint
  beim nächsten Start ganz oben — **vor dem Pausentag**, damit Gemessenes nicht hinter einer
  Sperre verschwindet — mit der Zahl der gesicherten Sätze und drei Wegen: weitermachen, die
  fertigen Blöcke abschließen und speichern, oder verwerfen. Letzteres nur mit Rückfrage.
- **Ein abgebrochener Satz ist ein abgebrochener Satz.** Danach stehen zur Wahl: wiederholen,
  Block hier beenden, Block überspringen, Workout abschließen, oder ganz verwerfen.
- **„Heute" führt während eines Laufs zurück ins Workout**, nicht auf den Planungsbildschirm.
- Ein nachträglich abgeschlossener Lauf trägt **sein eigenes Datum**, nicht das von heute — sonst
  wanderte eine Messung von gestern in die Erholungsrechnung von heute.
- **Messpunkt nachtragen** unter *Mehr*: Übung, Seite, Stufe, Zusatzgewicht, bis zu drei
  Haltezeiten, Datum. Für das, was schon verloren ist, und für Notizen auf Papier. Die
  Progressionsregeln laufen dabei bewusst nicht mit.

**Für die native App ist das die wichtigste Lehre des Prototyps.** Ein Messprotokoll mit Pausen
von mehreren Minuten und einem gesperrten Display ist genau der Fall, in dem eine Webseite ihren
Zustand verliert. Die Sicherung nach jedem Satz ist dort kein Komfort, sondern Pflicht — und der
Grund, warum Abschnitt 6.1 des Handoffs (Hintergrundlauf, Benachrichtigungen) kein Nice-to-have
ist.

---

### P29 · Die Einstellungen standen nicht dort, wo die Entscheidung fällt

Bis Fassung `2026-09-16 f` trug die Startseite **pro Block ein vollständiges Formular**: Stufe,
Zusatzgewicht, Häkchen „trotzdem messen", dazu die Begründungstexte. Bei fünf Blöcken waren das
fünf solcher Formulare untereinander, und alle mussten **vor dem ersten Satz** ausgefüllt werden.

Das ist aus zwei Gründen die falsche Stelle.

1. **Zeitlich.** Zwischen dem Ausfüllen und dem fünften Block liegen zwanzig bis dreißig Minuten.
   Was dort vorab eingestellt wurde, ist bis dahin eine Vermutung darüber, wie man sich nach vier
   Blöcken fühlen wird. Die Entscheidung gehört an den Moment, in dem sie wirksam wird.
2. **Räumlich.** Die Startseite ist damit ein langes Formular, durch das man scrollen muss, um an
   den Startknopf zu kommen — obwohl neun von zehn Malen nichts zu ändern ist.

**Behoben:** Vor jeder Übung steht eine eigene Karte mit genau den Einstellungen dieser Übung und
dem Knopf **„Übung starten"**. Die Startseite zeigt nur noch, was geplant ist: Übung, Variante,
Richtwert, Satzzahl, eine Zeile je Block.

Dieselbe Karte erscheint auch nach einer Unterbrechung — wer mit gesperrtem Display Pause macht
und zurückkommt, landet nicht mitten in einer laufenden Uhr, sondern vor der Übung.

**Drei Dinge kamen bei der Gelegenheit dazu**, weil sie an dieselbe Karte gehören:

- **Beide Seiten gleich.** Beim Ausfallschritt und beim Seitstütz galt die Variantenwahl bisher
  je Seite. In der Praxis ist sie fast immer dieselbe — zwei getrennte Einstellungen bedeuten also
  vor allem eine Gelegenheit, es zu vergessen. Jetzt zieht eine Änderung die andere Seite mit,
  ein Häkchen entkoppelt sie wieder (für den Fall, der den Seitenunterschied überhaupt begründet:
  einseitige Beschwerden). **Die Zielzeit wird nicht mitkopiert** — sie kommt aus der Kurve der
  jeweiligen Seite, und die beiden Seiten sind selten gleich stark.
- **Satzzahl je Block statt global.** Drei Sätze waren eine Konstante der App. Fachlich hängt die
  sinnvolle Zahl aber an der Zone: kurze, schwere Haltezeiten vertragen mehr Wiederholungen als
  lange. Voreingestellt sind **5 in Zone A, 4 in Zone B, 3 in Zone C**, wählbar von 1 bis 6.
  Die 4 in Zone B ist interpoliert — siehe Teil 4.
- **Workout vorzeitig beenden.** In jeder Satzpause steht jetzt *Workout hier beenden und
  speichern*. Der gerade gehaltene Satz wird dabei **mitgenommen**, nicht verworfen: ein Block aus
  einem Satz ist ein kurzer Block, kein verlorener. Der Export bekam dafür die Spalte
  `saetze_geplant` (Schemaversion 2) — ohne sie ist ein abgebrochener Fünfer später nicht von
  einer geplanten Einzelmessung zu unterscheiden.

---

### P30 · Zwei Messungen auf derselben Stufe ergaben eine Kurve — eine erfundene

Der Fit prüfte nur die **Anzahl** der Punkte, nie ihre Streuung über die Last. Das ist die
Stelle, an der die App zwei Jahre lang unauffällig falsch gerechnet hätte.

Liegen alle Punkte auf derselben Last `L`, ist `log(L − CP)` für jeden Punkt dieselbe Konstante.
Sie verschiebt alle Residuen gleich weit und **fällt aus der Streuung heraus** — die Fehlersumme
hängt über das ganze Suchgitter nicht mehr von CP ab. Gewonnen hat dann der erste Gitterpunkt,
also CP ≈ 0.

Nachgerechnet mit drei Messungen auf Last 0,60 (95 s / 88 s / 102 s):

| | CP | W′ | rmse | Vorhersage bei 0,30 | bei 0,90 |
|---|---|---|---|---|---|
| **eine Last** (Scheinfit) | 0,006 | 56,4 | **0,061** | 192 s | 63 s |
| drei Lasten (ehrlich) | 0,263 | 27,5 | 0,110 | 735 s | 43 s |

Der Scheinfit hat die **kleinere** rmse — nach der Gütezahl der App sah er *besser* aus als die
ehrliche Kurve. Bei genau zwei identischen Punkten wird die Fehlersumme exakt null: die App
meldete eine **perfekt sitzende** Kurve.

Der Hilfetext im Profil behauptete damals schon das Richtige — „Zwei Messungen in derselben Stufe
ergeben keine Kurve, das ist rechnerisch nicht lösbar" —, aber er stand im Zweig *keine Kurve
vorhanden* und wurde in genau diesem Fall nie angezeigt.

**Was daran hing:** Der Blockbau sprang in den Kurvenzweig statt in den Ankerzweig, womit die
Sprungbremse nie griff. Regel 4 stufte auf Grundlage von Rauschen hoch. Die Kurvenzeichnung
streckte die Lastachse bis auf null.

**Reparatur:** `fit` gibt `null` zurück, sobald alle Punkte dieselbe Last haben. Damit landet der
Blockbau im Ankerzweig — mit der Sprungbremse, also genau richtig.

**Warum es jetzt auffiel und nicht früher:** Bei drei Übungen mit sieben bis neun Sprossen
trifft man den Fall nur, wenn man drei Workouts lang dieselbe Stufe wählt. Eine Position mit
**einer** Variante trifft ihn bei jeder einzelnen Benutzung.

---

### P31 · Eine kurze Leiter forderte Zonen ein, die sie nie erreichen kann

Eine feste Last erzeugt eine feste Haltezeit. Um von Zone C (120 s) nach Zone A (35 s) zu kommen,
muss sich die Last **fast verdoppeln**:

| CP | W′ | Last für 120 s | Last für 35 s | Verhältnis |
|---|---|---|---|---|
| 0,25 | 20 | 0,417 | 0,821 | 1,97× |
| 0,42 | 32 | 0,687 | 1,334 | 1,94× |
| 0,60 | 30 | 0,850 | 1,457 | 1,71× |

**20 kg im Rucksack bringen den Faktor 1,25** (beim Seitstütz mit seinem doppelten Hebel 1,50).
Zusatzgewicht trägt also etwa **eine** Zone weit, nicht zwei. Als Ersatz für eine Leiter taugt es
nur dort, wo die Grundlast klein ist.

Daraus folgten zwei Dauerzustände: Die Zonenabfrage wurde nie leer, also wich der Blockbau in
**jedem** Workout von der Rotation ab und forderte eine Zone an, die die Position nie erreichte.
Und die Karte *Bis zum vollständigen Profil* verschwand nie — eine einzige kurze Leiter blockierte
sie für die **ganze App**, auch wenn alle anderen Kurven längst vollständig waren.

**Es brach leiser, als man denkt.** Die App wich nicht sichtbar von der Rotation ab (die
angeforderte Zone war ja leer). Sie beschriftete den Block als **Zone B** und setzte dann aus dem
Scheinfit aus P30 eine Zielzeit von **96 s** — das ist Zone C. Im Export standen ab da dauerhaft
`zone_geplant = B` neben `zone_erreicht = C`, ohne jede Warnung.

**Reparatur:** Beide Stellen fragen zuerst, welche Zonen die **verfügbaren** Sprossen samt
Zusatzgewicht rechnerisch überhaupt treffen. Gefordert wird nur noch das Erreichbare; die
Profilkarte verschwindet, sobald das Erreichbare belegt ist.

**Was eine zweite Variante wert ist.** Bei zwei Lasten ist der Fit exakt bestimmt und hat
**immer** Residuum null — er sieht immer perfekt aus, egal wie falsch er ist. Seine
Empfindlichkeit hängt allein am Abstand der beiden Lasten; bei 8 % Messfehler auf die zweite Zeit:

| Lastabstand | Fehler in CP |
|---|---|
| ×1,10 | −18 % |
| ×1,25 | −8 % |
| ×1,94 | −5 % |

**Daraus die Schwelle 1,25** — unter diesem Abstand ist ein Zwei-Punkt-Fit nicht belastbar. Die
vorhandenen Leitern haben Sprossenabstände von 1,05 bis 1,85: zwei *benachbarte* Sprossen reichen
oft nicht, es müssen die beiden **Enden** sein. Die App leitet daraus ab, wie sie eine Übung
behandelt (siehe Änderungsprotokoll, Kontrollposition) — die Klasse wird **nicht** von Hand
gesetzt, sonst geriete sie aus dem Tritt, sobald ein Gerät dazukommt oder wegfällt.

---

### P32 · Progression hatte oben und unten eine Sackgasse

Beide Enden der Leiter waren still.

- **Oben:** Regel 4 stieg nur, solange eine höhere Sprosse existierte. Auf der letzten Sprosse
  passierte nichts mehr — obwohl die Übung Zusatzgewicht verträgt und das Feld dafür sichtbar ist.
  Bei einer Ein-Sprossen-Position: nie eine Progression.
- **Unten:** Die Regeln 1 und 3 stuften bis Sprosse 0 zurück und taten dort **gar nichts**. Bei
  zwei Schmerzabbrüchen hintereinander zählte die App den Zähler hoch und schwieg.

Eine stille Nichtreaktion ist bei Schmerz das Schlechteste von allem.

**Reparatur:** Beide Richtungen laufen über zwei Bahnen. Nach oben: höhere Sprosse, sonst 2,5 kg
mehr. Nach unten: erst Gewicht abbauen, dann tiefere Sprosse. Ist beides ausgereizt, sagt die App
es ausdrücklich — *„Hier ist die App am Ende ihrer Mittel; diese Position hat keine leichtere
Stufe mehr."* — und nennt die Alternativen, die außerhalb ihrer Reichweite liegen.

---

### P33 · Sprossen, die Ausrüstung voraussetzen

Die drei ersten Übungen kamen mit Möbeln aus — Arbeitsplatte, Tisch, Stuhl, Hocker, Kiste,
Kissen. Die Leiter durfte deshalb annehmen, dass **jede** Sprosse für jeden erreichbar ist.
Schlingentrainer, Faszienrolle und Ringe brechen diese Annahme, und zwar in beide Richtungen: Der
Schlingentrainer ist beim Schrägzug die *beste* Leiter, die Rolle bei der Fersenbrücke die
interessanteste Progression — aber eine Sprosse, die der Nutzer nicht ausführen kann, darf die App
nicht vorschlagen.

**Lösung: ein Feld `geraet` je Sprosse plus eine Liste im Setup.** Sprossen ohne Feld sind immer
verfügbar.

**Entscheidend für die Datenhygiene: die Leiter wird NICHT umnummeriert.** Nicht verfügbare
Sprossen bleiben im Array stehen und werden nur bei der *Auswahl* übersprungen. Würde man sie
herausfiltern, verschöben sich die gespeicherte Stufe und die Exportspalte `stufe_nr` beim
Anhaken eines Geräts **rückwirkend** — genau die Instabilität, vor der das Datenformat-Dokument
heute schon warnt. Eine Übung, von der keine einzige Sprosse ausführbar ist, taucht im Workout
gar nicht erst auf.

**Voreingestellt vorhanden ist nur der Zugpunkt** — eine stabile Tischkante zählt mit. Rolle und
Ringe sind aus.

---

### P34 · Fünf Übungen sind acht Blöcke (Fortsetzung von P23)

P23 ließ offen, ob eine vierte Übung noch in dieselbe Sitzung passt. Die Antwort mit fünf Übungen,
gerechnet mit 5/4/3 Sätzen je Zone, 20 s Pause und 60 s Umbau je Block:

| Konfiguration | Blöcke | Zone A | Zone B | Zone C |
|---|---|---|---|---|
| Liegestütz, Ausfallschritt, Seitstütz | 5 | 26 min | 35 min | 38 min |
| + Schrägzug (einseitig) | 6 | 32 min | 42 min | 46 min |
| + Fersenbrücke (zweiseitig) | **8** | 42 min | 56 min | **61 min** |

In Zone A sind acht Blöcke **vierzig Sätze**. Die ISO-Stunde ist 08:30–09:30 — das passt nicht.

Der Grund liegt in der Gruppenlogik: Die Tore öffnen **synchron**. Alle Übungen waren am selben
Tag dran, alle sind zwei Tage später wieder frei.

**Lösung: Deckel plus Reihenfolge nach Wartezeit.** Sechs Blöcke, und die freien Übungen kommen in
der Reihenfolge dran, wie lange ihre **Kurve** nicht mehr gemessen wurde — nicht ihre Gruppe, denn
beim Ausfallschritt können die Seiten auseinanderlaufen. Zweiseitige Übungen kosten zwei Blöcke,
einseitige einen; wer nicht mehr unter den Deckel passt, rückt in die nächste Sitzung vor und steht
dort oben. Die Erholung je Muskelgruppe (P24) bleibt die harte Schranke — die Rotation entscheidet
nur, **welche** der freigegebenen Übungen heute drankommen.

**Der falsche Hebel wäre die Satzzahl.** Sie liegt näher, weil sie direkt an der Dauer hängt. Aber
der Abfall über die Sätze ist die einzige Information über die Erholung von W′ innerhalb eines
Blocks. Bei ein bis zwei Sätzen ist sie weg. Kürzen gehört an die Zahl der Blöcke, nicht an die
Zahl der Sätze.

---
### P35 · „CP 0,00" war die Wand des Suchbereichs, keine Messung

Luke schickte am 17.09. ein Bildschirmfoto seines eigenen Profils: **Dauerlast-Schwelle (CP) =
0,00**, W′ = 68, Streuung ±35 %. Die Zahl ist nicht falsch gerundet — sie ist gar keine Messung.

`fit` sucht CP auf einem Gitter von 1499 Punkten zwischen 0 und der kleinsten gemessenen Last.
Das Optimum saß auf dem **ersten** davon (cp = 0,000366), und `toFixed(2)` machte daraus eine
saubere 0,00. Die Suche hatte also kein Minimum gefunden, sondern den Rand ihres Bereichs.

**Warum es dort landet:** Seine beiden Messpunkte waren 0,55 → 82 s und 1,25 → 82 s. Aus
`t₁(L₁−CP) = W′ = t₂(L₂−CP)` folgt `CP = (t₂L₂ − t₁L₁)/(t₂ − t₁)` — bei gleicher Zeit ist der
Nenner null. Es gibt keine endliche Lösung; im Logarithmus fällt die Fehlersumme monoton gegen
`cp → −∞`, und die Box `0 < cp < minL` macht aus diesem Weglauf eine harmlos aussehende Null.

**Der Denkfehler, den ich beim ersten Anlauf selbst gemacht habe** — er gehört hierher, weil er
die naheliegende Reparatur wäre und falsch:

> „Optimum am Gitterrand" heißt **nicht** „die Daten taugen nichts."

Nachgerechnet:

| Messpunkte | CP | rmse | am Rand? |
|---|---|---|---|
| 0,55 / 82 s + 1,25 / 82 s | 0,0004 | **0,411** | ja |
| 0,55 / 124 s + 1,25 / 55 s | 0,0004 | **0,004** | ja |
| 0,55 / 150 s + 1,25 / 55 s | 0,145 | 0,000 | nein |

Die mittlere Zeile ist eine **praktisch perfekte** Kurve, die trotzdem am Rand liegt — weil
`t·L` dort fast konstant ist und CP damit echt nahe null. Für die Vorhersage **innerhalb** des
gemessenen Bereichs ist so ein Fit einwandfrei. Unbrauchbar ist nur die Zahl CP selbst.

**Reparatur, entsprechend eng:** `fit` merkt sich `randUnten`/`randOben`. Die CP-Asymptote
verschwindet dann aus der Zeichnung — mit cp ≈ 0 hatte sie die Lastachse bis fast auf null
gestreckt und alle Stufen an den rechten Bildrand gequetscht. **Blockiert wird nichts:** Zielzeiten
und Stufenvorschläge laufen weiter wie bisher, weil sie im gemessenen Bereich bleiben.

**Die Zahl selbst ist seit Fassung `c` gar nicht mehr zu sehen.** Sie stand zwischenzeitlich als
Strich mit Erklärung in der Kennzahlen-Karte; die ist auf Lukes Zuruf komplett gefallen (*„alle
erstmal nichtssagend und unverständlich"*). Das erledigt das Anzeigeproblem auf dem kürzesten Weg
— eine Zahl, die niemand deuten kann, muss auch nicht richtig angezeigt werden. **Der Befund bleibt
trotzdem stehen**, denn `randUnten` steuert weiterhin die Zeichnung, und sollten die Kennzahlen je
zurückkommen, ist „CP = 0,00" wieder genau die Falle, in die sie getappt sind.

---

### P36 · Die Streuungsangabe war gedeckelt — der Deckel stand in der Anzeige

Im selben Bildschirmfoto: **±35 %**. Das war nicht die Streuung, sondern ihre Obergrenze.

```js
const rel = f => Math.min(.35, Math.max(.05, f.rmse));
```

Der wahre Wert war **0,411**. Ein Fit, der die Messungen um die Hälfte verfehlt, wurde als ±35 %
gemeldet und sah damit aus wie ein mäßiger statt wie ein schlechter. Der Deckel saß in der
**Anzeige**, nicht in der Rechnung — gerechnet wurde weiter mit dem echten Wert, nur der Nutzer
erfuhr ihn nicht.

Dazu ein zweiter, kleinerer Fehler: Die Residuen sind Logarithmen, die Abweichung ist also
**multiplikativ und nicht symmetrisch**. rmse 0,41 heißt **+51 % nach oben und −34 % nach unten**,
nicht ±41 %.

**Reparatur in Fassung `b`:** Deckel weg, Angabe zweiseitig (+51 / −34 %), dazu eine Karte ab 25 %,
die die zwei möglichen Ursachen trennte.

**In Fassung `c` ist beides wieder verschwunden** — erst die Karte (*„Zu viel Text und
unverständlich für den Endverbraucher"*), dann die ganze Kennzahlen-Karte samt Streuungszahl.
Damit ist der Anzeigefehler auf dem kürzesten Weg erledigt, und das ist vertretbar: Eine Zahl, die
niemand deuten kann, hilft auch ungedeckelt niemandem.

**Der Befund gehört trotzdem aufgeschrieben, und zwar als Regel**, denn der Deckel war kein
Schreibfehler, sondern eine Haltung:

> Eine Gütezahl darf in der **Anzeige** nicht beschnitten werden. Wer sie für zu erschreckend
> hält, um sie zu zeigen, soll sie weglassen — nicht kleinrechnen. Ein gedeckelter Wert sieht aus
> wie ein gemessener und ist die schlechtere der beiden Möglichkeiten.

Die Konsequenz aus dem Weglassen steht als offener Punkt 20 in Teil 4: **Die App meldet jetzt gar
nicht mehr, wenn eine Kurve ihre eigenen Messpunkte verfehlt.**

---

### P37 · Die Startseite hing am Workout, nicht an der Übung

Luke fragte: *„Die Kurve stellt nur 2 Punkte dar, aber ich habe schon 3 mal trainiert, oder?"* —
und stieß damit auf einen Fehler, den die Rotation aus P34 erst erreichbar gemacht hat.

Bei einer zweiseitigen Übung wird eine Seite **frisch** gemessen und die andere als
**vorermüdet** geführt; nur die frische trägt die Kurve. Welche das ist, entschied ein einziger
globaler Schalter, der nach jedem Workout kippte:

```js
S.nextSide = (S.nextSide === "rechts") ? "links" : "rechts";
```

Solange jedes Workout jede Übung enthielt, ging das auf. **Seit die Rotation Übungen auslässt,
kippt er auch an Workouts ohne diese Übung.** Kommt sie nur jedes zweite Mal dran, ist er
zwischendurch zweimal gekippt — dieselbe Seite bekommt jedes Mal den frischen Punkt, die andere
nie einen. Der Satz auf der Startseite, *„So bekommt jede Seite gleich oft einen frischen
Messpunkt"*, stimmte dann nicht mehr.

**Reparatur: die Datenlage der Übung entscheidet, nicht ein Zähler.** Es startet die Seite mit
den **wenigsten frischen Messpunkten**. Das ist nicht nur robust gegen ausgelassene Workouts, es
gleicht einen **bereits entstandenen** Rückstand von selbst wieder aus — und zwei Übungen dürfen
in einem Workout verschieden herum starten, was der globale Schalter nie konnte.

**Zur Ausgangsfrage:** Die 2 im Profil war korrekt gezählt, aber sie beantwortete eine andere
Frage als die gestellte. Im Profil stehen die Punkte, die den **Fit** tragen; im Verlauf standen
alle Blöcke, dort aber als „N Satzblocks" — ein Wort, das nirgends erklärt wird. Beide Zahlen
stehen jetzt nebeneinander: **„3 Messungen · 2 davon in der Kurve"**.

---

### P38 · Zwei Erklärtexte, die an der falschen Stelle standen

Auf Lukes Zuruf entfernt, beide aus demselben Grund:

- **„Warum hier keine Maximalkraft steht"** — siehe P3.
- **„Der Ausfallschritt hat zwei Kurven…"** stand über dem Kurvenwähler und damit über **jeder**
  Kurve, auch dem Liegestütz, der gar keine zwei hat. Ein fest verdrahteter Übungsname in einem
  Text, der für alle gilt — derselbe Fehlertyp, der schon beim Zweite-Seite-Grund auffiel.

**Was stattdessen dazukam**, weil es eine echte Lücke war: eine **Legende unter der Zeichnung**.
Die Lastachse ist mit Stufennummern beschriftet, die Tabelle darunter führte die Namen ohne
Nummer — die beiden waren nicht zusammenzubringen. Jetzt tragen beide die Nummer, die aktuelle
Stufe ist hervorgehoben, gemessene Stufen bekommen einen Punkt, und Sprossen ohne das nötige
Gerät sind abgeblendet.

### P39 · Vier Wege zurück zum Startbildschirm, die keiner sein sollte

`render()` zeichnet die App neu — und zeigt dabei **fest verdrahtet den Startbildschirm**. Jede
Stelle, die nach einer Eingabe „bitte neu zeichnen" meinte, warf den Nutzer damit aus dem
Bildschirm, in dem er gerade stand. Innerhalb von zwei Tagen ist derselbe Fehler an **vier**
verschiedenen Stellen aufgeschlagen: beim Abhaken eines Geräts, beim Öffnen einer Erklärung, beim
Auswählen einer Übung in der Liste und beim Umstellen der Satzzahl.

Der Prototyp behilft sich mit einer zweiten Funktion (`render2()`, die den *aktiven* Bildschirm
neu zeichnet) und einem Merker für die Scrollposition, damit ein Häkchen in einer langen Liste
nicht an den Listenanfang zurückspringt.

**Für die Schätzung:** Das ist die Router-Frage, und sie gehört vor die erste Bildschirmzeile.
Ein Bildschirm braucht eine Adresse, „neu zeichnen" muss „**diesen** neu zeichnen" heißen, und die
Scrollposition ist Zustand je Bildschirm, nicht je App. In einem Framework mit Router und
Komponentenzustand kostet das nichts — nachträglich in eine gewachsene Zeichenfunktion
hineinoperiert, kostet es jedes Mal einen Fehlerbericht.

### P40 · Die Personenwaage muss analog sein

Das isometrische Wadenheben liest seine Last von einer Personenwaage unter dem Vorfuß ab. Luke hat
beim Aufbau festgestellt, was die Rechnung nicht vorhersagen konnte: **Digitalwaagen schalten sich
unter einer gleichbleibenden Last ab.** Sie erwarten ein Auftreten, zeigen ein paar Sekunden an und
gehen aus. Ein Halt dauert 20 bis 150 Sekunden — die Anzeige ist dann längst dunkel, und der
Nutzer hält gegen einen Wert, den er nicht mehr sieht.

**Für die Schätzung:** Die Geräteliste ist faktisch eine Einkaufsliste, und sie nennt bisher nur
den Gegenstand („Personenwaage"). Sie muss die **Eigenschaft** nennen, auf die es ankommt
(„analog, mit Zeiger — Digitalwaagen schalten unter Dauerlast ab"). Das gilt für jedes Gerät, das
während eines Halts abgelesen wird. Es ist eine Datenfeld-Frage, keine Textfrage: an der
Gerätekennung hängt künftig ein Beschaffungshinweis.

### P41 · Eine Gerätekennung zu viel versteckt nicht die Sprosse, sondern die ganze Übung

Beim Wadenheben trugen **alle fünf** Sprossen die Kennung „Waage" — auch die beidbeinige
Einstiegssprosse (50 % des Körpergewichts) und die einbeinige Endsprosse (100 %). Beide brauchen
gar keine Waage: bei 50 % steht man auf beiden Vorfüßen, bei 100 % auf einem, und in beiden Fällen
ist die Last der eigene Körper.

Die Folge war größer als „zwei Sprossen abgeblendet". `ausfuehrbar(ex)` gibt `false` zurück, wenn
**keine einzige** Sprosse ohne fehlendes Gerät läuft — die Übung verschwindet dann vollständig aus
der Planung. Zwei zu viel gesetzte Kennungen haben also eine Übung unbenutzbar gemacht, obwohl 40 %
ihrer Leiter gerätefrei sind.

**Für die Schätzung:** Die Verfügbarkeit einer Übung wird aus ihren Sprossen **abgeleitet**, nie
selbst gesetzt. Damit ist jede Gerätekennung eine Aussage über die Sichtbarkeit der ganzen Übung,
und ein Tippfehler darin ist nicht kosmetisch. Das gehört in die Datenpflege-Oberfläche (P33) mit
einer Anzeige „diese Übung bleibt ohne Gerät X nutzbar / nicht nutzbar".

### P42 · Die Erholungssperre kennt nur Gruppen und ist für Übungspaare blind

`MIN_GAP_DAYS` sperrt eine **Muskelgruppe** für 24 Stunden. Die beiden Wadenheben liegen in zwei
verschiedenen Gruppen (`calf` und `arch`), weil sie fachlich zwei verschiedene Strukturen treffen
sollen — also hat der Planer sie ohne Weiteres in dieselbe Sitzung gelegt. Lukes Befund aus dem
Training: **die Muskulatur überlappt zu stark**, die zweite Übung misst dann die Ermüdung der
ersten.

Der Prototyp hat dafür eine zweite Achse bekommen: `nichtMit: ["handtuch"]` je Übung, unabhängig
von der Gruppe, geprüft beim Zusammenstellen des Workouts.

**Für die Schätzung:** Der Ausschluss ist eine Eigenschaft des **Paares**, nicht einer Übung — im
Prototyp steht er von Hand auf beiden Seiten, und eine Datenstruktur, in der man eine Seite
vergessen kann, ist ein Fehler, der auf sein Datum wartet. In der echten App gehört das als
symmetrische Relation modelliert und beim Speichern geprüft. Dasselbe Feld trägt später jede
weitere „diese beiden nicht am selben Tag"-Regel, und davon wird es mehr geben, sobald acht
Übungen im Programm sind.

### P43 · Was eine Übung an Blöcken kostet, steht erst fest, wenn die Stufe feststeht

Zone C bedeutet Haltezeiten von 90 bis 150 Sekunden. Eine einbeinige Übung dort anzufangen ist
unrealistisch — deshalb ist die Einstiegssprosse des Wadenhebens **beidbeinig**. Und damit ist
„links" derselbe Halt wie „rechts": dieselben zwei Füße, dieselbe Last, nur einmal frisch und
einmal vorermüdet.

Der Planer rechnete aber weiterhin mit *Übung × Seiten = zwei Blöcke*. Er muss vor dem Bauen des
ersten Blocks wissen, auf welcher Sprosse der Nutzer gerade steht — die Blockzahl einer Übung ist
eine **Funktion des Trainingsstands**, keine Eigenschaft der Übung. Für den Fußtag hat das aus vier
Blöcken (rund 36 min) einen gemacht (rund 7 min).

**Für die Schätzung:** Der Plan ist keine statische Liste aus Übungen und Seiten. Drei Dinge hängen
daran: die Blockzahl, die Beschriftung (aus „links" wird „beidbeinig"), und die Frage, welche
Seiten überhaupt noch wählbar sind, wenn eine Übung gerade beidbeinig läuft. Im Prototyp waren
das fünf Hilfsfunktionen und ein eigener Zweig im Planer.

### P44 · Ein Fehler im Klick-Handler verschluckt sich spurlos

Beim Aufräumen einer früheren Fassung ist die Funktion `warnKarte()` gelöscht worden, während
`blockIntro()` sie weiter aufrief. Ergebnis: **vier Übungen ließen sich zwei Fassungen lang nicht
starten** — Fersenbrücke, aufrechter Läufer und beide Wadenheben, also genau die Übungen mit einem
Warnhinweis, der noch nicht beantwortet war.

Sichtbar war davon: nichts. Der Knopf „Workout starten" reagierte einfach nicht. Der Grund ist
struktureller Natur:

- `node --check` prüft **Syntax**, nicht Namen. Eine fehlende Funktion ist syntaktisch einwandfrei.
- Der Aufruf steht in einem `onclick`. Eine Ausnahme darin landet in `window.onerror` — und das
  ist in einer einzelnen HTML-Datei ohne geöffnete Entwicklerkonsole **nirgends**.

**Für die Schätzung:** Zwei Dinge, die beide nicht optional sind. Erstens ein Build-Schritt, der
bei einem unaufgelösten Namen abbricht — in einer gebündelten App bekommt man das von Bundler und
Linter geschenkt, hier gab es ihn nicht. Zweitens ein **sichtbarer Fehlerpfad in der laufenden
App**: ohne ihn ist „Ausnahme im Handler" von „Knopf reagiert nicht" nicht zu unterscheiden, und
genau Letzteres meldet der Nutzer. Der Prototyp hat dafür jetzt ein rotes Band am unteren Rand;
in der echten App ist das die Stelle für Fehlerberichte an den Server.

### P45 · Aus einer laufenden Übung gab es keinen Ausstieg

Der Ablauf kannte genau zwei Enden: alle Blöcke durchziehen, oder das ganze Workout abbrechen.
Was fehlte, war das Naheliegende — **diese eine Übung heute lassen, der Rest läuft weiter**.

Der Knopf war der kleinere Teil. Der größere: Wer die erste Seite einer zweiseitigen Übung
überspringt, lässt die zweite Seite mit dem Vermerk „vorermüdet" (`fresh:false`) zurück — und dieser
Vermerk schließt sie von der Stufenregel aus. Ohne Reparatur des Plans hätte die Abbruchmöglichkeit
also eine **stille Datenlücke** eingeführt: gemessen, aber nie gewertet. Der Ausstieg muss die
zweite Seite nachrücken lassen.

**Für die Schätzung:** Jeder Abbruchpfad ist eine Aussage über die schon erhobenen Daten und muss
sie auch treffen — im Prototyp steht auf dem Knopf, dass für Kurve und Stufenregel ohnehin nur der
erste Satz zählt und damit nichts verloren geht. Das ist keine nachträgliche Verzierung: Die
Zustandsmaschine des Workouts gehört **mit** ihren Abbrüchen entworfen, sonst repariert man bei
jedem neuen Ausstieg den Datenpfad hinterher.

### P46 · Die Dauer einer Sitzung stand nirgends — und die App kann sie nicht messen

Luke hat nach einer Zeitangabe gefragt, weil das Training in einem festen Fenster stattfindet. Die
App kann zwei der drei Bestandteile exakt rechnen (Haltezeit und die 20-Sekunden-Pausen innerhalb
eines Blocks) und den dritten **gar nicht**: die Pause zwischen zwei Übungen ist die eigene Pause
des Nutzers, sie wird nirgends gestoppt. Die genannten 180 Sekunden sind eine Annahme.

Die Rechnung dahinter ist unangenehm eindeutig: Sechs Blöcke in Zone C sind rund 36 Minuten reines
Halten plus 15 Minuten angenommene Blockpausen — **55 Minuten**. Die Stellschrauben sind der
Blockdeckel (6/5/4/3 Blöcke → 55/46/36/26 min) und die Blockpause (zwei statt drei Minuten bringen
nur 55 → 50 min). Die Übungsauswahl ist **keine** Stellschraube.

**Für die Schätzung:** Sobald die App eine Dauer behauptet, sollte sie sie auch messen. Ein
Zeitstempel bei Start und Ende eines Workouts macht aus der Annahme nach einer Handvoll Sitzungen
eine gemessene Größe — und die ist zugleich die Grundlage für jede spätere Aussage über Adhärenz.
Das ist ein Feld, kein Feature, aber es muss von Anfang an mitgeschrieben werden.

### P47 · Die App war beim Laden tot, und zwar genau bei den neuen Übungen

Ein Fehler in der ausgelieferten Fassung, gefunden beim Prüfen von etwas anderem. Die Meldung
lautet `Cannot access 'geraetDa' before initialization`, und ihre Wirkung ist total: Das gesamte
Skript bricht ab, die Seite bleibt beim Ladezustand stehen, kein Knopf reagiert.

Die Ursache ist eine Reihenfolge, die man beim Lesen nicht sieht. Direkt nach `load()` läuft eine
Prüfung, die für die zuletzt gewählte Übung ermittelt, ob sie ein- oder beidseitig zu trainieren
ist. Diese Prüfung ruft über vier Zwischenstationen — `sidesOf` → `beidJetzt` → `gemerkteStufe` →
`verfuegbareStufen` — die Hilfsfunktion `geraetDa` auf. Die stand dreihundert Zeilen **weiter
unten** als `const`-Pfeilfunktion. Funktionsdeklarationen werden in JavaScript nach oben gezogen,
`const`-Bindungen nicht: Bis die Zeile erreicht ist, wirft jeder Zugriff darauf.

Warum es niemandem vorher aufgefallen ist: Die Voreinstellung ist der Liegestütz, und der ist
einseitig — die Kette wird gar nicht erst betreten. Sie wird nur betreten, wenn die zuletzt
gewählte Übung **zweiseitig** ist und für sie noch **keine Stufe gemerkt** ist. Das trifft genau
den Fall „neue Übung zum ersten Mal ausgewählt", also aufrechter Läufer und beide Wadenheben, und
jede Übung nach einem Stufen-Reset.

**Für die Schätzung.** Zwei getrennte Lehren, beide unbequem:

1. *Der Fehlerpfad aus P44 hat hier nicht geholfen.* Das rote Band am unteren Rand wird von einem
   `window.addEventListener("error")` gespeist, der in derselben Datei weiter oben registriert ist
   und formal auch gegriffen hat. Nur: Wenn das Skript beim Start abbricht, ist auch die Oberfläche
   nie gebaut worden, an die sich das Band hängen könnte. Ein Fehlerpfad muss **außerhalb** des
   Codes stehen, dessen Fehler er meldet — in der echten App also im HTML-Grundgerüst, nicht im
   Anwendungs-Bundle.
2. *Diese Fehlerklasse gehört einem Werkzeug, nicht einem Menschen.* `node --check` sieht sie nicht
   (P44), ein Namensabgleich gegen die Vorgängerfassung sieht sie nicht — sie ist ja definiert,
   nur zu spät. Was sie sieht, ist ein Linter mit aktivierter Regel gegen Zugriffe vor der
   Definition (`no-use-before-define`), und der kostet in einem gebündelten Projekt eine Zeile
   Konfiguration.

Behoben in `2026-09-18 h`: `geraetDa` ist jetzt eine Funktionsdeklaration.

### P48 · Eine Haltezeit ist nur dann eine Messung, wenn sie eine Maximalzeit ist

Beim ersten isometrischen Wadenheben stand auf der Uhr **244,2 Sekunden**, beidbeinig. Lukes
Kommentar dazu: *„hätte ich mehr als 5 Minuten halten können."* Das ist kein Nebensatz, das ist
der wunde Punkt des ganzen Modells.

`t = W' / (L − CP)` setzt voraus, dass `t` die **maximale** Haltezeit auf dieser Last ist. Wer
aufhört, weil nichts mehr passiert, liefert eine untere Schranke. Die Kurve verarbeitet die aber
wie eine Messung und hält die Übung danach für schwächer, als sie ist — und schlägt in der Folge
**noch leichtere** Stufen vor. Der Fehler verstärkt sich selbst. Gerechnet über den einen Wert
verschiebt ein nicht ausgereizter Halt `W'` um rund ein Viertel nach unten.

Die eingebaute Antwort ist ein Messfenster: **unter 20 und über 300 Sekunden zählt ein Wert nicht
mehr für die Kurve** — er sagt dann nichts über die Belastbarkeit, sondern nur, dass die Stufe zu
schwer oder zu leicht gewählt war. Die Grenzen sind nicht frei gewählt: 20 s ist die Untergrenze
von Zone A, 300 s das Doppelte der Obergrenze von Zone C.

**Was dieses Fenster nicht leistet, und darauf kommt es an.** Der Halt, der die Regel ausgelöst
hat, fällt selbst nicht durch sie durch: 244,2 liegt unter 300. Die Zeitgrenze fasst nur die Fälle,
in denen die Stufe so offensichtlich daneben liegt, dass die Uhr es verrät. **Ob ein Halt ausgereizt
wurde, weiß allein der Trainierende.** Die App kann es nicht messen, nicht schätzen und nicht
ableiten — sie kann nur fragen.

**Für die Schätzung:** Ein Feld am Satzende mit vier Antworten — *Muskel · Form · Gleichgewicht ·
abgebrochen* — ist der kleinste Eingriff, der diese Lücke schließt. Es ist ein Auswahlfeld, aber es
entscheidet über die Gültigkeit **jedes einzelnen Datenpunkts** und gehört deshalb in das
Datenmodell, bevor die erste echte Messreihe entsteht. Es beantwortet zugleich eine Frage, die im
Projekt schon zweimal aufgeschlagen ist: ob ein Halt an der Zielmuskulatur endete oder am
schwächsten Glied der Kette — genau das Argument, mit dem der Seitstütz für die Glutealsehne
ausgeschieden ist.

### P49 · Eine Sprosse, die nur einer Ausführung gehört, sperrt die Übung ein

Das isometrische Wadenheben hat eine Sprosse, die **beidbeinig** gehalten wird: ein Halt, der für
links und rechts zugleich zählt (P8 von der anderen Seite her, ausgeführt in `2026-09-19 g`). Die
Sprosse trägt dafür eine Marke, und der Planer respektierte sie konsequent: Ein Block ohne Seite
durfte **ausschließlich** dort landen.

Konsequent, und genau deshalb eine Falle. Wer auf dieser Sprosse steht, misst nur sie. Eine
einzige Last ergibt keinen Fit — es gibt keine Kurve, also auch keinen rechnerischen Grund,
irgendwohin umzustufen. Die zweite Sicherung, das Messfenster aus P48, greift ebenfalls nicht:
244 Sekunden liegen unter 300 und gelten damit als messbar. Und der dritte Weg, die
Ankerrechnung, bildet über `nahe()` auf die erlaubten Sprossen ab — also wieder auf dieselbe.
Drei unabhängige Wege, und alle drei führen zurück auf die Sprosse, die zu leicht ist.

Sichtbar wurde das als Widerspruch auf einer einzigen Karte: **Zone A** als Vorgabe, darunter die
beidbeinige Sprosse, die diese Zone nachweislich nicht erreichen kann (Teil 4, Punkt 25). Die App
schrieb ein Ziel auf, das ihre eigene Auswahl ausschließt.

**Die allgemeine Form, und darauf kommt es für den Nachbau an.** Sobald eine Leiter Sprossen
enthält, die nur unter einer bestimmten **Ausführung** existieren — beidbeinig statt einbeinig,
zwei Arme statt einem, Knie statt Fuß (P21) —, ist die Ausführung keine Eigenschaft der Sprosse
mehr, sondern eine eigene Dimension neben ihr, wie das Zusatzgewicht in P27. Sie muss dann drei
Dinge können: sich aus den Daten **vorentscheiden**, sich von Hand **überstimmen** lassen, und den
Ablauf **umbauen**, wenn sie kippt — denn beidbeinig ist ein Block, einbeinig sind zwei. Wer sie
nur als Filter über die Sprossenliste behandelt, baut dieselbe Sackgasse noch einmal.

**Und der Umbau endet nicht beim Plan.** Aus einem Block werden zwei — und die beiden gehören
nebeneinander in den vorderen Teil des Workouts, sonst steht die zweite Seite am Ende und ist
vorermüdet, also als Messpunkt wertlos. Das ist der bleibende Teil.

**Wie weit die Nähe im Ablauf geht, ist dagegen keine Frage der Physiologie.** Mit
`2026-09-21 b` liefen die beiden Seiten ohne Zwischenkarte durch, mit `2026-09-21 c` steht wieder
die normale Blockpause dazwischen — Lukes Entscheidung, nicht die der Datenlage. Für den Nachbau
zählt daran zweierlei: Eine Eigenschaft wie `seitenParallel` bündelt mehrere Folgen
(Reihenfolge, Messbarkeit, Pausenlänge), und wird eine davon zurückgenommen, müssen die übrigen
einzeln geprüft werden — ein pauschales Abschalten hätte hier die Messung der zweiten Seite
stillschweigend mit erledigt.

**Zwei Datenlecks derselben Ursache.** Umgekehrt war der Filter gar nicht gesetzt: Ein Block *mit*
Seite konnte auf der beidbeinigen Sprosse landen und hätte „50 % beidbeinig · links" gespeichert —
einen Messpunkt, den es physikalisch nicht gibt. Dasselbe galt für die Variantenliste auf der
Karte, die dem Nutzer diese Sprosse weiter zur Auswahl anbot. Beides ist seit `2026-09-21 a` zu.

---

### P50 · Auf der obersten Sprosse ist das Gewicht die Leiter

Unterhalb der obersten Sprosse steuert die Sprossenwahl die Zone: leichtere Stellung, längere
Haltezeit. Oben gibt es keine schwerere Stellung mehr, dort bleibt als Stellschraube nur das
Zusatzgewicht. Die App hat das Gewicht bis zum 24.09. aber nur bei Leitern aus **einer**
Sprosse (dem aufrechten Läufer) auf die Zone gerechnet. Bei allen anderen blieb es auf dem
Stand, den die Nachsteuerung nach der letzten Messung hinterlassen hatte.

Aufgefallen ist das bei Luke: Zone B gewählt, Wadenheben links auf „100 % — einbeinig
+ 20 kg", und die Karte sagte 112 s voraus. Das ist Zone C. Oben stand die gewählte Zone,
darunter eine Zeit, die zu einer anderen gehört.

**Für den Nachbau:** Auf der obersten Sprosse ist das Zusatzgewicht (P27) keine Beigabe. Es
ist die Fortsetzung der Leiter. Die Zielzeit wird dort über die Umkehrung der Kurve in ein
Gewicht übersetzt. Gedeckelt wird wie überall durch die Gewichtsspanne der Übung und höchstens
ein halbes Körpergewicht. Dazu kommen zwei Grenzen:

- Hat der Nutzer das Gewicht heute selbst gesetzt, bleibt es stehen. Sonst nähme ihm die
  Zonenwahl genau das Gewicht, das er gerade eingestellt hat.
- Über ihren gemessenen Bereich hinaus rechnet die Kurve nur, wenn die Gewichtsspanne dieser
  Sprosse den gemessenen Bereich überhaupt erreicht. Eine Sprosse, die schon mit dem
  kleinsten Gewicht weit über allem Gemessenen liegt, bekommt wie jede andere keine
  Vorhersage.

**Nebenbefund an derselben Stelle.** Unterhalb der Sprosse, ab der Gewicht getragen wird, hat
die Sprossenauswahl trotzdem mit dem gespeicherten Gewicht gerechnet. Stand die Wade oben auf
+ 20 kg, wurden die Waage-Sprossen als „Waage + 20 kg" bewertet, also mit einer Last, die es
nicht gibt. Unterhalb dieser Sprosse gilt jetzt 0 kg.

---

### P51 · Drei Menüs bis zum Timer

Luke, 24.09.: „ich muss mich zusätzlich durch 3 menü´s klicken bis der timer läuft", und: „das
menü ist komplett überladen". Heute zeigte den Plan als Blockliste, dazu die Zonenwahl, die
Ausführungswahl, eine Hilfebox zur Dauer und die Profiltabelle. Danach kam **je Seite** eine
eigene Karte mit:

- Ausführungstext
- Gewichtshinweisen
- Stufen-Knöpfen
- einem Abgleich „Beide Seiten gleich"
- Erklärzeilen

Vieles stand doppelt da, weil dieselbe Entscheidung an zwei Stellen angeboten wurde.

**Für den Nachbau:** Jede Entscheidung hat genau einen Ort, und zwar dort, wo sie fällt.

- Welche Übungen heute: auf der Startseite.
- Welche Zone, Ausführung, Variante, welches Gewicht und wie viele Sätze: auf der Karte vor der
  Übung, für beide Seiten zusammen.

Was die App rechnen kann, steht schon gewählt da. Der Nutzer korrigiert nur. Eine Erklärung,
die man braucht, um eine Zahl zu verstehen, gehört in den Export, nicht auf den
Trainingsbildschirm.

---

### P52 · Waage-Sprossen sind Entlastung auf einer einbeinigen Sprosse

Die Wadenleiter hatte fünf Sprossen: 40 %, 63 % und 79 % mit der Waage unter dem Vorfuß,
dazwischen beidbeinig (50 %), oben einbeinig (100 %). Erst über der einbeinigen kam
Zusatzgewicht. Neue Nutzer begannen auf der leichtesten Sprosse, also mit Waage. Luke, 24.09.:
Einbeinig ist die Kernvariante, viele steigen dort direkt ein, und darüber geht es nur noch mit
Gewicht weiter, stufenlos wie bei Grip Gains. Dazu ein Rucksack vorne an der Brust, 50 bis 60 kg
für Trainierte, „ab 30 kg Zusatzgewicht wird es ordentlich schwer".

Die drei Waage-Sprossen waren physikalisch nie etwas anderes als einbeinig mit Entlastung.
„63 % — Waage" heißt: einbeinig, die Waage zeigt 63 % des Körpergewichts. Das ist dieselbe
Last wie „einbeinig, 29,6 kg entlastet" bei 80 kg. Eine Leiter mit drei festen Punkten auf
dieser Strecke ist gröber als die Strecke selbst.

**Für den Nachbau:** Eine Übung, deren Last sich an einer Anzeige ablesen lässt, braucht keine
Sprossen für die Zwischenwerte. Sie braucht **eine** Sprosse und Gewicht in beide Richtungen:

- Nach oben trägt der Rucksack.
- Nach unten nimmt die Waage ab. Ihre Anzeige **ist** die Last, ohne Hebel dazwischen. Das
  Minus gibt es deshalb nur, wenn die Waage vorhanden ist. Eine Entlastung ohne Anzeige wäre
  eine Schätzung, keine Last.
- Die Entlastung endet dort, wo die nächste Sprosse beginnt. Einbeinig mit halbem
  Körpergewicht entlastet ist beidbeinig. Ein fester Deckel in kg läge bei leichten Nutzern
  darunter, und „leichter" hieße dann, auf eine Sprosse mit **mehr** Last zu wechseln.
- Der Deckel nach oben hängt an der Übung, nicht an einer App-weiten Regel. Beim Wadenheben
  sind es drei Viertel des Körpergewichts, höchstens 60 kg.

Der Preis: Zwischen beidbeinig und einbeinig liegt ein Faktor 2. Überbrückt wird er nur mit der
Waage. Wer keine hat, springt bei Schmerz von einbeinig direkt auf beidbeinig zurück.

### P53 · Der Rucksack trägt beidbeinig bis an einbeinig heran

P52 hat die Lücke zwischen beidbeinig und einbeinig, einen Faktor 2, mit der Waage überbrückt.
Luke, 24.09.: Der Rucksack geht auch beidbeinig, „so lückenlos skalieren bis man stark genug
für einbeinig ist". Eine Personenwaage braucht es dann nicht mehr.

Nachgerechnet bei 80 kg Körpergewicht, Deckel drei Viertel davon, höchstens 60 kg:

| Ausführung | ohne Gewicht | mit vollem Rucksack |
|---|---|---|
| Beidbeinig | 0,50 | 0,875 (+ 60 kg) |
| Einbeinig | 1,00 | 1,75 (+ 60 kg) |

Zwischen 0,875 und 1,00 bleibt ein Rest von Faktor 1,14. Bei 100 kg Körpergewicht deckelt
der Rucksack schon bei 60 kg, beidbeinig endet dann bei 0,80, und der Rest ist Faktor 1,25. Bei
60 kg Körpergewicht endet beidbeinig mit 45 kg ebenfalls bei 0,875. Kein Punkt der Kurve ist
weiter als eine halbe Rasterstufe von einer tragbaren Last entfernt. Liegt die Ziellast im
Rest, nimmt die App die nähere Seite.

**Der Preis.** Die obere Hälfte von beidbeinig kostet viel Gewicht auf der Brust: 0,70 braucht
32 kg, 0,80 braucht 48 kg, 0,875 die vollen 60 kg. Die Waage hätte dieselbe Last mit einer
Anzeige unter dem Fuß erreicht. Und der kleinste Schritt nach unten ist jetzt ein Wechsel:
einbeinig ohne Gewicht (1,00) → beidbeinig mit vollem Rucksack (0,875). Wer bei Schmerz dort
zurückgeht, schnallt 60 kg um, statt Gewicht abzugeben.

**Für den Nachbau:** Tragen zwei Sprossen einer Übung Gewicht, merkt sich jede ihr eigenes.
Beidbeinig + 20 kg und einbeinig + 20 kg sind verschiedene Lasten. Ein gemeinsames Feld würde
beim Wechsel eine davon falsch machen. Beim Wechsel wird umgerechnet, auf dieselbe Last:
beidbeinig + 60 kg ↔ einbeinig ohne Gewicht. Die anderen Übungen, deren Leiter unterhalb der
obersten Sprosse Gewicht erlaubt (Liegestütz, Ausfallschritt, Rudern, Hüftstrecken), behalten
ein Feld für alle Sprossen. Dort entscheidet weiter die Leiter vor dem Gewicht.

### P54 · Mit Waage endet beidbeinig bei 30 kg

P53 hat den Preis genannt: Die obere Hälfte von beidbeinig kostet viel Gewicht auf der Brust,
bis 60 kg. Luke, 25.09.: „alles bis 30kg ist zumutbar dannach kommt die waage zum einsatz".
Wer eine Waage hat, trägt beidbeinig höchstens 30 kg. Die schwereren Lasten laufen einbeinig
mit Entlastung. Die Waage liegt unter dem Vorfuß des Standbeins, das andere Bein ist in der
Luft, die Hände nehmen über eine Auflage Gewicht ab.

Die Naht liegt dort, wo beide Ausführungen dieselbe Last tragen. Einbeinig mit x kg Entlastung
ist 1 − x / Körpergewicht, beidbeinig mit 30 kg ist 0,5 × (1 + 30 / Körpergewicht). Gleich sind
sie bei x = Körpergewicht / 2 − 15 kg:

| Körpergewicht | beidbeinig + 30 kg | einbeinig, dieselbe Last | Waage zeigt |
|---|---|---|---|
| 60 kg | 0,75 | 15 kg entlastet | 45 kg |
| 80 kg | 0,6875 | 25 kg entlastet | 55 kg |
| 100 kg | 0,65 | 35 kg entlastet | 65 kg |

Liegt die Naht zwischen zwei 2,5-kg-Schritten, wird die Entlastung abgerundet. Einbeinig ist
dann nie leichter als beidbeinig mit 30 kg. Bei 81 kg sind das 25 statt 25,5 kg, und es bleibt
ein Sprung unter 1 %. Ohne Waage bleibt alles wie in P53: beidbeinig trägt bis zum vollen
Deckel.

**Was es bringt.** Der kleinste Schritt nach unten ist wieder ein Schritt: Schmerz auf
einbeinig ohne Gewicht heißt 2,5 kg entlastet und nicht beidbeinig mit 60 kg. Zwischen 0,6875
und 1,00 gibt es keinen Rest mehr.

**Der Preis.** Der Haken „Waage" unter „Geräte" entscheidet jetzt über die Ausführung. Wer eine
Waage hat und trotzdem beidbeinig mit 45 kg halten will, muss den Haken abnehmen. Das
betrifft dann auch die Handtuchrolle. Und die Last ist nur so genau, wie die Anzeige während
des Halts stillsteht. Drücken die Hände nach, wandert die Zahl.

**Für den Nachbau:**

- Jede Sprosse hat ihre eigene Gewichtsspanne. Beidbeinig endet bei `beidBis`, aber nur, wenn
  das Gerät da ist (`extraMinGeraet`).
- Die Untergrenze von einbeinig ist kein fester kg-Wert. Sie wird aus der Naht gerechnet
  (`waageTiefe`), sonst gibt es je nach Körpergewicht eine Lücke oder eine Überlappung.
- Die Lastrechnung selbst (`lastVon`) klemmt weiter auf die Vereinigung beider Spannen. Sonst
  bekämen alte Messungen nachträglich eine andere Last.
- Über die Naht hinweg ist der Wechsel lastgleich: beidbeinig + 30 kg ↔ einbeinig 25 kg
  entlastet. Wechselt eine Regel dort die Sprosse, weil es zu leicht war oder weh tat, muss sie
  auf der neuen Sprosse noch einen Schritt weitergehen. Sonst feuert die Regel, und die Last
  bleibt gleich.

### P55 · Der Zwei-Tage-Abstand war eine zweite Regel für denselben Deckel

Luke, 25.09.: „gib mir die möglichkeit Übungen zu trainieren die ich gestern trainiert habe.
Das volumen sollten wir über gewisse zeitintervalle deckeln, es spricht aber nichts dagegen die
gleiche übung 2 tage hintereinander zu machen".

Seine Vorgabe vom 14.09. („dieselbe Muskelgruppe dreimal pro Woche, jede Zone einmal pro Woche")
stand doppelt im Code. Einmal als Abstand: zwei Tage zwischen zwei Einheiten derselben Übung,
also Mo/Mi/Fr. Und einmal als Zonenruhe: Jede Zone kommt erst nach sieben Tagen wieder dran.
Die Zonenruhe allein deckelt schon. Drei Zonen, jede einmal in sieben Tagen, ergeben höchstens
drei Einheiten in sieben Tagen, gleich wie sie liegen. Der Abstand hat nichts gedeckelt. Er hat
nur verboten, zwei davon nebeneinanderzulegen.

Jetzt gilt nur noch: höchstens einmal am Tag. Die Zonenruhe bleibt der Deckel. Ein eigener
Zähler „drei in sieben Tagen" wäre eine dritte Regel für dieselbe Sache.

**Was es bringt.** Mo A, Di B, Mi C geht jetzt, danach ist die Übung bis zum nächsten Montag
zu. Mo/Mi/Fr geht wie bisher. Wer einen Tag verpasst, holt ihn am nächsten nach, statt zwei
Tage zu verlieren.

**Der Preis.** Drei Tage am Stück heißen vier Tage Pause für diese Übung. Ob ein Halt am Tag
nach dem letzten kürzer ausfällt, ist nicht gemessen. Solche Punkte gehen wie alle anderen in
die Kurve.

**Das Nachholen.** Das Nachholfenster für eine halb gemessene Übung bleibt zwei Tage lang,
obwohl die Übung am nächsten Tag ohnehin frei ist. Ohne das Fenster würde die App am
nächsten Tag beide Seiten in der nächsten Zone planen. Die offene Seite bekäme ihre Messung in
der Zone der Gegenseite dann nie. Dabei ist ein alter Fehler aufgefallen. Nach „links gestern,
rechts heute nachgeholt" galt links als offen und am Tag darauf wieder rechts, im Wechsel ohne
Ende. Ein Nachholblock trägt jetzt eine Marke, die Sitzung erbt sie, und eine markierte
Messung reißt kein neues Loch auf.

**Wohin es geht.** Luke im selben Gespräch: Ziel ist, dass die Form entscheidet, nicht eine
Regel, wann und wie oft trainiert werden darf. Fällt die Leistung der letzten Einheiten unter
die eigene Norm der letzten Wochen, ist das funktionelles Überziehen. Hält das an, wird die
Menge reduziert, bis die Form zurückkommt. Das geht erst nach einigen Wochen Messung. Bis
dahin hält die Zonenruhe den Deckel. Der Entwurf dafür folgt getrennt.

**Für den Nachbau:**

- Gezählt wird je Übung und Tag. Zwei Workouts am selben Tag sind eine Einheit
  (`MIN_GAP_DAYS = 1`).
- Der Deckel „drei in sieben Tagen" folgt aus der Zonenruhe. Er hält nur, solange jede Einheit
  der Zone angerechnet wird, die sie wirklich trainiert hat. Das ist `workoutZone` der Sitzung.
  Springt ein Block wegen einer leeren Zone in eine andere (die Lückenlogik bei neuen Übungen),
  bleibt die Zone der Rotation frei. Dann sind in diesen ersten Wochen auch vier Einheiten in
  sieben Tagen möglich.
- Das Nachholfenster (`NACHHOL_TAGE = 2`) hängt nicht mehr an der Sperre. Im Onboarding gilt
  weiter nur der Tag selbst.
- Die Marke `nachhol` sitzt am Block. Sie überlebt den Zonenwechsel auf der Karte und landet
  über `blockSichern` in der Sitzung.

### P56 · Die Handtuchrolle braucht nur eine Sprosse

> **Überholt am selben Tag (Build 2026-09-25 d).** Die Handtuchrolle hat wieder eine
> einbeinige Sprosse, mit Waage ab 40 kg Rucksack. Siehe P59.

Luke, 25.09.: „das wadenheben mit handtuchrolle ist deutlich schwerer ich denke die bilateral
variante + rucksack ist hier erstmal ausreichend".

Bis Build b hatte die Handtuchrolle fünf Sprossen: beidbeinig, drei Stufen an der Waage,
einbeinig mit Gewicht. Jetzt gibt es eine. Man steht beidbeinig, schwerer wird es mit dem
Rucksack vorn an der Brust, bis 60 kg wie beim Wadenheben. Die Last ist
0,5 × (1 + Rucksack / Körpergewicht). Bei 80 kg und 20 kg Rucksack sind das 0,625, mit 60 kg
sind es 0,875.

**Was es bringt.** Für die Handtuchrolle braucht niemand mehr eine Waage. Der Haken „Waage"
unter „Geräte" betrifft nur noch das Wadenheben (P54). Die Progression läuft allein über den
Rucksack, die Karte wechselt keine Sprosse mehr.

**Der Preis.** Über 0,875 geht es nicht hinaus. Wer beidbeinig mit 60 kg noch zu leicht
findet, hat keine nächste Stufe. Die alte einbeinige Sprosse lag bei 1,00 und darüber. Kommt
einbeinig zurück, ist der Platz dafür in der Definition noch da.

**Für den Nachbau:**

- Umzug mit Fassung 22 (`HANDTUCH_BEID_AT`). Jede alte Messung bekommt die Sprossennummer 0,
  Last, Seite und Etikett bleiben. Fehlt einer Messung die Last, wird sie aus der alten Leiter
  nachgetragen (`HANDTUCH_ALT_LAST`). Keine Messung wird verworfen.
- Gespeicherte Handtuch-Gewichte werden gelöscht. Die Karte rechnet das Rucksackgewicht neu
  aus der Kurve.
- In einem angefangenen Workout stehen offene Blöcke danach auf der einen Sprosse ohne
  Gewicht. Gelaufene Blöcke behalten ihre Last.
- Die Seiten links und rechts bleiben in der Definition. Alte einbeinige Messungen behalten so
  ihre Kurve. Beidbeinige Halte zählen für beide Seiten, wie beim Wadenheben.

### P57 · Die Form entscheidet, nicht die Zonenruhe

Luke, 25.09.: „wichtig ist ein nachvollziehbarer chart grip gains gibt hier 0-100 vor, 100 ist
der maximale trend voll erholt mit andauernden bestzeiten 0 ist die schlechteste form die man
haben kann basierend auf historische daten, man kann sich also ruhig auch eine zeit unter 50
aufhalten hier akkumuliert man dann erschöpfung und befindet sich im functional overreaching
und man baut fitness auf". Und: „der user macht das selbst die app spricht nur eine empfehlung
basierend auf dem trend aus (premium funktion)".

P55 hat es angekündigt, Teil 4 Nr. 28 hat den Entwurf. Luke hat ihn am 25.09. übernommen. So
rechnet der Prototyp seit Build c:

1. **Signal je Halt.** Der erste Satz jedes Blocks wird gegen die Kurve aus den Tagen davor
   gelesen, nie gegen eine Kurve, die ihn schon enthält. Gerechnet wird in Kraft, nicht in
   Zeit: Welche Last hätte die Kurve für die gehaltene Zeit vorhergesagt, und wie weit liegt
   die echte Last darüber? 10 % länger gehalten sind in Zone C rund 4 % Kraft, in Zone A rund
   7 %. So wiegt eine Einheit in jeder Zone gleich.
2. **Tagesform.** Das Mittel der Signale eines Tages, beide Seiten zusammen.
3. **Kurzfrist.** Das Mittel der Tagesform über die letzten drei Trainingstage der Übung.
4. **Formwert 0 bis 100.** Der Rang dieses Kurzfrist-Werts unter allen bisherigen derselben
   Übung. 100 ist die stärkste Phase bisher, 0 die schwächste, 50 die Mitte. Jeder Tag wird mit
   dem gerechnet, was an diesem Tag bekannt war. Ein neuer Tag ändert keinen alten Wert.
5. **Drei Zustände.** Unter 20 ist Tal. Das Tal hält, bis der Wert wieder 50 erreicht. Sonst
   Aufbau (ab 50) oder Überziehen (20 bis 49).
6. **Kraftlinie.** Unter dem Formwert steht die Last, die die Kurve der letzten 30 Tage für
   75 Sekunden vorhersagt. Das ist die Mitte von Zone B, Tag für Tag.
7. **Empfehlung.** Ab 30 Tagen nach dem ersten Messtag. Vorher steht auf der Karte
   „vorläufig" und das Datum, ab dem die Empfehlung kommt. Die App sagt nur, was der Trend
   nahelegt. Entscheiden tut der Nutzer. Die Empfehlung trägt die Marke „Premium". Was Premium
   ist, hat Luke am 25.09. entschieden; das steht im Marketing-Kanon, nicht im Repo.
8. **Heute.** Steht eine Übung im Tal und ist das letzte Signal jünger als sieben Tage, steht
   unter ihr „Formtal · entlasten empfohlen".
9. **Die Zonenruhe fällt als Sperre.** Das gilt für eine Gruppe, sobald jede Übung, die in ihr
   in den letzten 30 Tagen trainiert wurde, einen aktiven Trend hat. Einmal am Tag bleibt.
   Die Rotation wählt weiter die Zone, weil die Kurve alle drei braucht. Seit Build d nur mit
   Premium.
10. **Ohne Premium** (seit Build d). Formwert, Zustand, Chart und Tabelle sieht jeder. Es
   fehlen die Empfehlungszeile, der Hinweis auf Heute und die Freigabe der Zonenruhe. Die
   Karte wirbt nicht für Premium.

**Was es bringt.** Die Form fährt Wellen, mal mehr Umfang, mal weniger. Wann entlastet wird,
steht in der eigenen Linie, nicht in einem Blockplan. Die App entscheidet nicht, sie zeigt und
empfiehlt.

**Der Preis.**

- Die Schwellen 20 und 50 sind meine Setzung, nicht Lukes. Beide sind Konstanten und ändern
  sich mit einer Zeile.
- In den ersten Wochen springt der Rang. Mit zwei oder drei Vergleichswerten ist jeder neue
  Wert 0, 50 oder 100. Deshalb kommt die Empfehlung erst nach 30 Tagen. Gezeichnet sind die
  frühen Tage trotzdem.
- Ein Schmerzabbruch gibt kein Signal, denn die Zeit sagt dann nichts über die Form. In der
  Kurve bleibt er.
- Liegt ein Halt außerhalb des Bereichs, den die Kurve gemessen hat (mehr als 12 % über dem
  schwersten oder unter dem leichtesten gemessenen Halt), gibt er kein Signal. Die Kurve rät dort nur.
- Bis Build c fiel die Zonenruhe für jeden, auch für jemanden, der die Empfehlung nicht
  bekäme. Der hätte dann weder Pause noch Rat gehabt. Luke, 25.09.: „ja wie du sagst". Seit
  Build d gilt: ohne Premium bleibt die Zonenruhe.
- Rechenzeit: ein Fit je Messtag und Übung. Das Ergebnis wird zwischengespeichert und nur
  neu gerechnet, wenn sich die Messungen der Übung ändern.

**Für den Nachbau:**

- `formVerlauf(ex)` liefert je Tag Signale, Tagesform, Kurzfrist, Formwert, Zustand und
  Kraftlinie. `formStand(ex)` ist der letzte Tag samt Aktivierung, `formTal(ex)` der Hinweis
  für Heute, `formFreiGruppe(gr)` die Freigabe der Zonenruhe.
- Die Konstanten: `FORM_KURZ = 3`, `FORM_LANG_TAGE = 30`, `FORM_LANG_ZEIT = 75`,
  `FORM_TAL = 20`, `FORM_AUFBAU = 50`, `FORM_FRISCH_TAGE = 7`.
- Der Rang (`formRang`) zählt Gleichstand halb. Ein einziger Wert ist 50.
- Premium steht in `S.premium` und wird über `premiumAn()` gelesen. `formTal`,
  `formFreiGruppe` und die Empfehlungszeile in `formKarte` fragen es ab. Der Prototyp hat keine
  Anmeldung, deshalb steht unter Mehr ein Schalter „Premium freigeschaltet", voreingestellt
  an. In der fertigen App kommt das aus dem Konto. In der App heißt es nur
  „Premium".
- Vorbild und Quelle: Grip Gains, FAQ zum Performance Trend
  (https://gripgains.ca/resources/faq). Die Formeln dort sind nicht veröffentlicht, die
  Übertragung ist meine.

### P58 · Einbeiniges Wadenheben und Aufrechter Läufer am selben Tag

Luke, 25.09.: „isometrisches EINBEINIGES Wadenheben hat starke inteferrenz mit dem aufrechten
läufer die Übungen mit hinweis ausstatten nach möglichkeit nicht an einem tag beide zu
trainieren (gluteus medius, äußere hüfte wird stark gefordert)".

Beide Übungen laden den Gluteus medius des Standbeins. Die Erholungsschranke greift je Gruppe.
Wade und Läufer stehen in verschiedenen Gruppen, deshalb sieht sie das Paar nicht. Der zweite
Halt am selben Tag wäre vorermüdet, und die Kurve bekäme einen zu kurzen Punkt.

Zur Handtuchrolle, Luke, 25.09.: „ja aber nur für handtuchrolle einbeinig wird handtuchrolle
beidbeinig trainier (in der entsprechende zone) kann trainiert werden da hier keine
hüftstabilisation stattfinden muss".

**Gebaut in 2026-09-25 e.** Die Regel hängt an der Ausführung, nicht an der Übung:

| Heute im Plan | Mit dem aufrechten Läufer am selben Tag |
|---|---|
| Wadenheben einbeinig | nach Möglichkeit nicht |
| Wadenheben beidbeinig | ja |
| Handtuchrolle einbeinig | nach Möglichkeit nicht |
| Handtuchrolle beidbeinig | ja |

- Ob heute beidbeinig gehalten wird, entscheidet wie bisher die Zone. Die App dreht die
  Ausführung nicht um, nur um das Paar zu trennen.
- Einbeinig mit Waage-Entlastung zählt als einbeinig. Das Standbein trägt weniger, muss das
  Becken aber genauso halten, und Lukes Maßstab ist die Hüftstabilisation.
- Der Plan legt die beiden nicht auf denselben Tag. Wer länger auf eine Messung wartet,
  kommt dran; der andere rückt auf den nächsten Tag vor.
- Ausnahmen: Hat der Nutzer heute selbst „Einbeinig" gewählt, bleiben beide. Steht bei einer
  der beiden eine Seite offen, geht das Nachholen vor.
- In der Häkchenliste steht an der verschobenen Übung „besser nicht mit … — beide fordern die
  äußere Hüfte". Sie bleibt anhakbar, das Häkchen tauscht nicht. Hakt der Nutzer beide an,
  steht an beiden „zusammen mit … — beide fordern die äußere Hüfte".
- Die Hinweistexte der drei Übungen nennen die Regel. Wade und Handtuchrolle bleiben
  untereinander hart getrennt, wie bisher.

**Der Preis.** Hält der Läufer die Wartezeit-Liste an, rückt das einbeinige Wadenheben einen
Tag weiter, auch wenn sonst nichts im Plan steht als der Läufer. „Nach Möglichkeit" heißt hier:
Die App trennt, solange der Nutzer nichts anderes anhakt. Einen Tag mit nur einer Übung nimmt
sie dafür in Kauf.

**Für den Nachbau:**

- Merkmal `huefteStandbein: true` an Wade, Handtuchrolle und Läufer. Gelesen wird es aus
  `DEFAULT_EX`, nicht aus der gespeicherten Definition. Deshalb gibt es keine neue Fassung und
  keinen Umzug.
- `huefteHeute(ex, bloecke)`: Stehen die Blöcke schon fest, zählt, ob einer davon einseitig
  ist. Sonst `beidJetzt`. `huefteKonflikt(a, b)` gilt nur, wenn beide heute die äußere Hüfte
  fordern und nicht schon `konflikt` sie trennt. `huefteGewollt` ist die eigene Wahl
  „Einbeinig" für heute.
- `auswahlHeute` überspringt im automatischen Plan, nicht beim Zusammenstellen von Hand.
  `planWorkout` gibt die verschobenen als `huefte: [{ex, wegen}]` zurück, getrennt von
  `spaeter` und `paar`.
- Die Häkchenliste schreibt den Satz in eine eigene Zeile. Der Formtal-Hinweis bleibt darunter
  sichtbar.

### P59 · Handtuchrolle: Rucksack bis 40 kg, danach einbeinig mit Waage

Luke, 25.09.: „Zum thema isometrisches wadenheben mit handtuchrolle geh bei der beidbeinigen
ausführung im rücksack bis 40kg dannach einbeinig mit entlasten der waage".

P56 hatte die Leiter auf eine Sprosse gekürzt, beidbeinig mit Rucksack bis 60 kg. Jetzt gibt
es wieder zwei, gebaut wie beim Wadenheben (P54), nur mit einer anderen Grenze: beidbeinig mit
Rucksack bis 40 kg, darüber einbeinig mit Entlastung. Die Waage liegt unter dem Vorfuß des
Standbeins, das Handtuch darauf, das andere Bein ist in der Luft. Die Hände nehmen auf einer
Auflage so viel Gewicht ab, bis die Anzeige die Zahl auf der Karte zeigt.

**Die Naht.** Beidbeinig mit 40 kg trägt 0,5 × (1 + 40 / KG), einbeinig mit x kg Entlastung
trägt 1 − x / KG. Gleich sind beide bei x = KG / 2 − 20:

| Körpergewicht | Entlastung | Waage zeigt | Last |
|---|---|---|---|
| 60 kg | 10 kg | 50 kg | 0,833 |
| 80 kg | 20 kg | 60 kg | 0,750 |
| 100 kg | 30 kg | 70 kg | 0,700 |

Liegt die Naht zwischen zwei 2,5-kg-Schritten, wird die Entlastung abgerundet, wie beim
Wadenheben. Einbeinig ist dann nie leichter als beidbeinig mit 40 kg. Unter 53 kg
Körpergewicht endet der Rucksack schon vorher, bei drei Vierteln des Körpergewichts: Bei 50 kg
geht beidbeinig bis 37,5 kg, die Entlastung beginnt bei 5 kg.

Ohne Waage geht beidbeinig bis zum vollen Deckel (60 kg, höchstens drei Viertel des
Körpergewichts, bei 80 kg also 0,875). Einbeinig beginnt ohne Gewicht bei 1,00 und trägt den
Rucksack. Das ist dieselbe Regel wie beim Wadenheben.

**Was es bringt.** Die Handtuchrolle hat wieder eine Stufe über 0,875, der Preis aus P56 ist
weg. Beidbeinig bleibt die Grundform: Neue Nutzer beginnen dort, weil die Rolle schwerer ist
als das Wadenheben ohne Handtuch. Welche Ausführung eine Zone trifft, rechnet die Kurve
(`beidPerKurve`), wie beim Wadenheben.

**Der Preis.**

- Dieselbe Waage, zwei Grenzen: Das Wadenheben endet beidbeinig bei 30 kg, die Handtuchrolle
  bei 40 kg. Beide stehen in der Definition der Übung (`beidBis`).
- Der Haken „Waage" gilt für beide Übungen. Getrennt einstellen lässt sich das nicht.
- Einbeinig lädt die Handtuchrolle das Standbein wie das Wadenheben. Deshalb gilt P58
  (nicht am selben Tag wie der aufrechte Läufer) für sie mit, seit Build e.

**Für den Nachbau:**

- Die Übung bekommt dieselben Felder wie das Wadenheben: zwei Sprossen, `beidBis: 40`,
  `extraMinGeraet: "waage"`, `kgJeSprosse`, `extraSeiten` mit `waage: true`. Die Rechnung
  dahinter ist allgemein (`extraBereich`, `waageTiefe`, `waageSpanne`, `kgKey`) und brauchte
  keine Änderung.
- Umzug mit Fassung 23 (`HANDTUCH_WAAGE_AT`). Messungen mit Seite waren einbeinig (die Leiter
  vor Build c) und ziehen auf die Sprosse 1. Last, Seite und Etikett bleiben. Beidbeinige
  Messungen haben keine Seite und bleiben auf 0. Keine Messung wird verworfen.
- Der gemerkte Rucksack für beidbeinig zieht auf sein eigenes Feld (`handtuch|links#0`). Mit
  Waage und mehr als 40 kg steht der Stand danach einbeinig auf derselben Last. Das rechnet
  `wadeWaage`, jetzt mit einem Schlüssel-Filter für beide Übungen. Bei 80 kg wird aus
  beidbeinig + 45 kg einbeinig mit 17,5 kg Entlastung.
- Angefangenes Workout: Ein offener beidbeiniger Block über 40 kg bekommt 40 kg (nur mit
  Waage). Gelaufene einbeinige Blöcke ziehen wie Messungen um.
- Nebenbei behoben: Der Umzug aus Fassung 22 ließ bei offenen Blöcken die alte vorgeschlagene
  Sprosse stehen, etwa eine 4 auf einer Leiter mit einer Sprosse. Jetzt zieht sie mit.

---

## Teil 2 — Änderungsprotokoll

Die Fassung steht unten in der App und wird bei jeder Änderung hochgezählt.

### 2026-09-25 e — Einbeinig nicht am selben Tag wie der aufrechte Läufer

**Befund (Luke).** Einbeiniges Wadenheben und der aufrechte Läufer fordern beide die äußere
Hüfte des Standbeins, „nach möglichkeit nicht an einem tag beide". Für die Handtuchrolle „nur
für handtuchrolle einbeinig", beidbeinig „kann trainiert werden da hier keine
hüftstabilisation stattfinden muss" (P58).

**Gebaut.** Der automatische Plan legt einbeiniges Wadenheben, mit und ohne Handtuchrolle,
nicht auf denselben Tag wie den aufrechten Läufer. Wer länger wartet, kommt dran. Beidbeinig
gilt die Regel nicht; welche Ausführung heute dran ist, entscheidet weiter die Zone. Einbeinig
mit Waage-Entlastung zählt als einbeinig. Die eigene Wahl „Einbeinig" für heute und eine offene
Nachholseite heben die Trennung auf. Von Hand angehakt bleiben beide, und an beiden steht der
Grund. Die Hinweistexte der drei Übungen nennen die Regel.

**Geprüft.** 793 von 793 Prüfungen, 42 davon neu:

- Merkmal an genau drei Übungen; Wade und Handtuchrolle bleiben hart getrennt
- welche Ausführung zählt: einbeinig ja, beidbeinig nein, fertige Blöcke vor dem Leiterstand,
  Merkmal auch aus alten gespeicherten Definitionen
- Plan: einbeinig und Läufer nur einer, wer länger wartet gewinnt, beidbeinige Handtuchrolle
  und Läufer beide, eigene Wahl „Einbeinig" beide, von Hand beide, zwei offene Seiten beide,
  allein nie leer
- Häkchenliste: „besser nicht mit …" an der verschobenen Übung, anhakbar; „zusammen mit …" an
  beiden, wenn beide angehakt sind; beidbeinig kein Satz
- Hinweistexte

Im Browser (live, Wadenheben einbeinig, Läufer und Liegestütz aktiv): Der Plan nimmt Läufer und
Liegestütz, das Wadenheben steht darunter mit „besser nicht mit Aufrechter Läufer — beide
fordern die äußere Hüfte". Angehakt kommt es dazu, der Läufer bleibt, und beide Zeilen sagen
„zusammen mit …" in Warnfarbe. Die Testdaten sind gelöscht.

### 2026-09-25 d — Handtuchrolle einbeinig mit Waage ab 40 kg · Premium-Schalter · dieses Papier im Repo

**Befund (Luke).** „geh bei der beidbeinigen ausführung im rücksack bis 40kg dannach einbeinig
mit entlasten der waage" (P59). Was Premium ist, hat Luke entschieden (steht im Marketing-Kanon).
Zur Zonenruhe ohne Empfehlung: „ja wie du sagst". Zu diesem Papier: „ja pack es ins repo".

**Handtuchrolle (P59).** Zwei Sprossen. Mit Waage bei 80 kg: beidbeinig 0 bis 40 kg Rucksack,
einbeinig ab 20 kg entlastet (die Waage zeigt 60 kg) bis 60 kg Rucksack. Ohne Waage:
beidbeinig bis 60 kg, einbeinig ab 0 kg. Hinweistexte und Geräteverzeichnis nennen die Waage
wieder. Fassung 23 zieht alte Messungen um.

**Premium (P57).** Unter Mehr steht ein Schalter „Premium freigeschaltet", voreingestellt an.
Aus heißt: kein Tal-Hinweis auf Heute, keine Empfehlungszeile auf der Formtrend-Karte, und die
Zonenruhe sperrt wie vor Build c. Formwert, Zustand, Chart und Tabelle bleiben.

**Dieses Papier im Repo.** Es liegt jetzt neben `index.html` und wird mit jedem Build
nachgezogen. Der technische Handoff bleibt draußen.

**Geprüft.** 751 von 751 Prüfungen, 62 davon neu:

- Leiter: zwei Sprossen, Grenze 40 kg, Waage schaltet frei, eigene Felder je Sprosse
- Naht bei 50, 60, 80 und 100 kg; beidbeinig + 40 kg und einbeinig mit Entlastung tragen
  dieselbe Last; Umrechnung über die Naht
- ohne Waage: beidbeinig bis zum vollen Deckel, einbeinig ohne Entlastung
- Umzug aus Fassung 22 mit und ohne Waage: Messungen, Stand, Rucksackfelder, offene und
  gelaufene Blöcke, zweiter Durchlauf ändert nichts
- Premium: voreingestellt an, aus bleibt aus; ohne Premium kein Hinweis, keine Empfehlung,
  kein Werbesatz, Zonenruhe sperrt; mit Premium frei über den Trend

Die Tests zur einsprossigen Handtuchrolle aus Build c prüfen jetzt die zweisprossige.

Im Browser (80 kg, Waage angehakt): Die Karte der Handtuchrolle bietet Beidbeinig und
Einbeinig an. Einbeinig mit Waage lässt sich bis „Waage zeigt 60 kg" herunterstellen, nicht
weiter. Beidbeinig endet der Rucksack bei 40 kg. Beide Endpunkte sagen dieselbe Zeit voraus
(64 s). Mit Premium aus fehlen auf Heute der Tal-Hinweis und im Profil die Empfehlung, Formwert
und Chart stehen da. Die Testdaten sind gelöscht.

### 2026-09-25 c — Formtrend von 0 bis 100 · Handtuchrolle beidbeinig mit Rucksack · Körpergewicht freiwillig

**Befund (Luke).** „wichtig ist ein nachvollziehbarer chart grip gains gibt hier 0-100 vor"
(P57). „die bilateral variante + rucksack ist hier erstmal ausreichend" (P56). „frag das
körpergewicht ab, aber nur optional nicht zwingend notwendig".

**Formtrend (P57).** Im Profil steht unter der Kurve eine neue Karte „Formtrend":

- der Formwert groß, daneben der Zustand (Aufbau, Überziehen, Tal)
- ein Chart mit zwei Feldern: oben der Formwert von 0 bis 100 mit dem Tal-Band unter 20 und
  einer gestrichelten Linie bei 50, unten die Kraft bei 75 s
- die Empfehlung mit der Marke „Premium", oder vor Tag 30 „Empfehlung ab dem …"
- eine Tabelle der letzten acht Tage: Tag, erster Satz mit Sekunden und Erwartung („links
  42 s, erwartet 38 s"), Tagesform in Prozent Kraft, Formwert

Auf Heute steht unter einer Übung im Tal „Formtal · entlasten empfohlen". Für Gruppen mit
aktivem Trend sperrt die Zonenruhe nicht mehr.

**Handtuchrolle (P56).** Eine Sprosse, beidbeinig, Rucksack von 0 bis 60 kg. Fassung 22 zieht
alte Messungen um, ohne eine zu verwerfen. Die Waage im Geräteverzeichnis nennt die
Handtuchrolle nicht mehr.

**Körpergewicht.** Der Einstieg fragt es einmal, freiwillig, ohne Pflichtfeld. Leer, Unsinn
oder Werte außerhalb von 30 bis 200 kg zählen als „nicht angegeben", dann bleibt es bei 80 kg
(`bwLesen`). Ändern geht wie bisher unter Mehr.

**Geprüft.** 695 von 695 Prüfungen, 81 davon neu:

- Körpergewicht: leer, Komma, Unsinn, Grenzen, kein Pflichtfeld
- Handtuchrolle: eine Sprosse, Rucksack, Last 0,625 bei 20 kg und 80 kg Körpergewicht
- Umzug aus Fassung 21: Sprossennummer, Last, Seite, nachgetragene Last, offene und gelaufene
  Blöcke, zweiter Durchlauf ändert nichts
- Formtrend: Rang, Kraft statt Zeit, Signal nur mit Kurve davor und im gemessenen Bereich,
  starker Tag 100, drei schwache Tage 0 und Tal, Tal hält bis 50, nichts ändert sich
  rückwirkend, Schmerzabbruch ohne Signal, vorläufig vor Tag 30, Karte, Zonenruhe je Gruppe

Die Tests zum alten fünfsprossigen Handtuch prüfen jetzt dieselben Regeln am Wadenheben.

Im Browser: Die App lädt mit Build c, der Einstieg zeigt das Feld ohne Pflicht. Mit 40 Tagen
Testdaten zeigt das Profil die Karte (Formwert 10, Tal) und Heute den Hinweis. Die Testdaten
sind gelöscht.

### 2026-09-25 b — Zwei Tage hintereinander erlaubt · Nachholen ohne Wechselspiel

**Befund (Luke).** „gib mir die möglichkeit Übungen zu trainieren die ich gestern trainiert
habe … es spricht aber nichts dagegen die gleiche übung 2 tage hintereinander zu machen" (P55).

**Die Regel.** Jede Übung höchstens einmal am Tag. Der Abstand von zwei Tagen ist weg. Die
Zonenruhe von sieben Tagen je Zone bleibt und deckelt auf drei Einheiten pro Woche. In der
Liste steht eine heute trainierte Übung auf „morgen", eine gestern trainierte ist anhakbar.
Unter Mehr → Erholung steht der neue Satz: höchstens einmal am Tag, zwei Tage hintereinander
erlaubt, die Pause je Zone begrenzt die Menge.

**Das Nachholen.** Am Tag nach einem abgebrochenen Workout ist die Übung frei. Geplant wird
trotzdem nur die offene Seite, in der Zone der Gegenseite. Der Nachholblock trägt die Marke
`nachhol`. Die Sitzung erbt sie, und danach gilt die andere Seite nicht wieder als offen. Der
Grund steht jetzt auch im Plan, wenn die Übung nicht gesperrt ist.

**Geprüft.** 614 von 614 Prüfungen, 24 davon neu:

- einmal am Tag
- gestern trainiert, heute frei
- A/B/C an drei Tagen, danach vier Tage zu
- drei Einstiegstests an drei Tagen hintereinander
- Nachholen am Folgetag mit genau einem Block
- die Marke wandert in die Sitzung
- kein Wechselspiel nach dem Nachholen
- ein echter neuer Abbruch bleibt ein Loch
- die Liste zeigt „morgen" nur für heute

Im Browser: Liegestütz heute trainiert steht auf „morgen". Die Wade von gestern ist frei.
Beim Ausfallschritt, gestern nur links, beginnt der Plan mit „Ausfallschritt rechts".

### 2026-09-25 a — Wadenheben: mit Waage endet beidbeinig bei 30 kg

**Befund (Luke).** „übernimm die schweren progressionen dann doch einbeinig mit waage ich denke
alles bis 30kg ist zumutbar dannach kommt die waage zum einsatz" (P54).

**Die Leiter.** Weiterhin zwei Sprossen. Ist unter „Geräte" eine Waage angehakt, gilt:

| Sprosse | leichteste Last (80 kg) | schwerste Last (80 kg) |
|---|---|---|
| Beidbeinig | ohne Gewicht, 0,50 | + 30 kg, 0,6875 |
| Einbeinig | 25 kg entlastet, 0,6875 | + 60 kg, 1,75 |

Ohne Waage bleibt es bei Build c: beidbeinig bis zum vollen Deckel, einbeinig ab 0 kg. Diese
Grenze ist gesetzt, nicht erfragt. Luke kann sie umdrehen. Die anderen Übungen ändern sich
nicht. Neu an der Übung: `beidBis: 30` und `extraMinGeraet: "waage"`. Die Spanne je Sprosse
steht in `extraBereich(ex, level)`, die Tiefe der Entlastung rechnet `waageTiefe`.

**Die Kurve wählt die Ausführung** wie in Build c (`beidPerKurve`), nur mit der neuen Obergrenze.
Bei einer schwachen Kurve (kritische Last 0,4) stehen Zone B und C beidbeinig mit höchstens
30 kg, Zone A einbeinig. Bei einer mittleren Kurve (kritische Last 0,6) stehen jetzt alle drei
Zonen einbeinig, Zone B mit 15 kg entlastet. Der Grund unter den Knöpfen nennt 30 kg statt 60.

**Die Regeln.**

- Schmerz auf einbeinig ohne Gewicht: 2,5 kg entlastet, die Übung bleibt einbeinig.
- Schmerz an der tiefsten Entlastung: beidbeinig + 27,5 kg.
- Zu leicht auf beidbeinig: erst mehr Rucksack bis 30 kg, dann einbeinig. Knapp zu leicht mit
  25 kg wird einbeinig, 22,5 kg entlastet.
- Beidbeinig + 30 kg über 310 s bei 120 s Ziel wird einbeinig + 20 kg. Die Meldung sagt „Eine
  Stufe schwerer — jetzt „Einbeinig + 20 kg"".
- Der Wechsel an der Naht ist lastgleich. Deshalb geht die Regel auf der neuen Sprosse einen
  Schritt weiter (`stufeWechseln`). Ohne diesen Schritt hätte Schmerz an der Naht nichts
  geändert.

**Die Karte.** Einbeinig zeigt wieder die Knöpfe Waage / Ohne Gewicht / Rucksack und bei
Entlastung „Waage zeigt 62,5 kg" mit Stepper. Beidbeinig zeigt den Rucksack, „+" ist bei 30 kg
gesperrt. Die Lastachse beginnt mit Waage bei beidbeinig ohne Gewicht, die Gewichtsachse bei
25 kg entlastet. Ohne Waage ist nirgends eine Entlastung zu sehen. Der Übungstext beschreibt,
wo die Waage liegt, und bei „Geräte" steht, wofür die Waage beim Wadenheben da ist.

**Der Umzug (Fassung 21).** Nur mit angehakter Waage:

| Stand in Build c (80 kg) | Last | Neu |
|---|---|---|
| Beidbeinig + 45 kg | 0,781 | Einbeinig, 17,5 kg entlastet (Waage zeigt 62,5 kg) |
| Beidbeinig + 60 kg | 0,875 | Einbeinig, 10 kg entlastet |
| Beidbeinig + 30 kg oder weniger | — | bleibt |
| Einbeinig, beidbeinig gemerkt + 45 kg | — | einbeinig bleibt, beidbeinig gemerkt 30 kg |

Umgerechnet wird auf dieselbe Last: Entlastung = (Rucksack − Körpergewicht) / 2, aufs
2,5-kg-Raster. Ein angefangener Lauf mit beidbeinigen Blöcken über 30 kg steht danach auf
30 kg. Gemessene Sätze bleiben, wie sie sind. Ohne Waage ändert sich nichts, ein zweiter
Durchlauf auch nicht.

Wer direkt aus Build b kommt (Fassung 19) und eine Waage hat, behält seine Entlastung, soweit
sie heute geht. 7,5 kg entlastet bleibt. 30 kg entlastet liegt unter der Naht (25 kg) und wird
beidbeinig + 20 kg, wie in Build c.

**Geprüft.** 590 Prüfungen. Neu dazu:

- Spanne je Sprosse bei 60, 80, 81 und 100 kg
- Naht und Umrechnung in beide Richtungen
- Namen, Schmerz, zu leicht, Regel 2a
- Ausführung je Zone bei zwei Kurven
- Achse mit Waage
- Karte mit und ohne Waage
- Umzug aus Fassung 19 und 20

Im Browser mit einem Stand aus Fassung 20 angesehen (Waage angehakt, beide Seiten beidbeinig
+ 45 kg): Umzug auf einbeinig, 17,5 kg entlastet. Die Karte zeigt „Waage zeigt 62,5 kg",
beidbeinig steht bei 30 kg mit gesperrtem „+". Testdaten gelöscht. Build `2026-09-25 a`.

---

### 2026-09-24 c — Wadenheben: Rucksack auch beidbeinig, die Waage geht

**Befund (Luke).** „man kann doch die beidbeinige variante auch mit dem rücksack zuladen und so
lückenlos skalieren bis man stark genug für einbeinig ist. Dann ist doch eine personenwaage gar
nicht nötig." Er hat recht (P53).

**Die Leiter.** Weiterhin zwei Sprossen, Beidbeinig (50 %) und Einbeinig (100 %). Beide tragen
den Rucksack, in 2,5-kg-Schritten bis drei Viertel des Körpergewichts, höchstens 60 kg. Die
Entlastung mit der Waage fällt beim Wadenheben weg (`extraMin`, `extraMinGeraet`,
`extraMinAnteil` gestrichen). Jede Sprosse merkt sich ihr eigenes Gewicht (`kgJeSprosse`,
`kgKey`: beidbeinig liegt unter `wade|links#0`, einbeinig weiter unter `wade|links`).

**Die Kurve wählt die Ausführung.** Für die Mitte der Zone rechnet die App die nötige Last
(`beidPerKurve`). Reicht beidbeinig mit vollem Rucksack, steht beidbeinig vorn, sonst
einbeinig. Liegt die Last im Rest zwischen beiden, gewinnt die nähere Seite. Ohne Kurve gilt
wie bisher der letzte beidbeinige Halt. Der Grund unter den Knöpfen heißt jetzt „Zone A braucht
mehr, als beidbeinig mit 60 kg geht. Deshalb steht einbeinig vorn." Bei einer mittleren Kurve
(kritische Last 0,6) sind das beidbeinig in Zone B und C, einbeinig in Zone A. Bei Lukes Kurve
steht in allen drei Zonen einbeinig.

**Die Regeln.** Zu leicht auf beidbeinig: erst mehr Rucksack, erst am Deckel einbeinig.
Beidbeinig + 60 kg über 310 s bei 120 s Ziel wird einbeinig + 47,5 kg. Die Meldung nennt die
Stufe und das neue Gewicht („Eine Stufe schwerer — jetzt „Einbeinig + 47,5 kg""), keine
Differenz zweier Sprossen. Schmerz auf einbeinig ohne Gewicht: beidbeinig + 60 kg. Das
beidbeinige Gewicht bleibt beim Wechsel gemerkt. Regel 4 sagt „Diese Stufe trägt über 150
Sekunden" statt „Die oberste Stufe", weil das jetzt auch beidbeinig passiert.

**Die Karte.** Beidbeinig und einbeinig zeigen denselben Rucksack-Stepper. Die Anzeige
„Waage zeigt …" und die Knöpfe Waage / Ohne Gewicht / Rucksack sind weg. Die Lastachse beginnt
bei 0 kg, auch wenn unter „Geräte" eine Waage angehakt ist.

**Der Umzug (Fassung 20).** Aus einbeinig mit Entlastung wird beidbeinig mit Rucksack, auf
dieselbe Last: Körpergewicht minus zweimal Entlastung, aufs 2,5-kg-Raster.

| Stand in Build b (80 kg) | Last | Neu |
|---|---|---|
| Einbeinig, 30 kg entlastet (früher 63 % — Waage) | 0,625 | Beidbeinig + 20 kg |
| Einbeinig, 7,5 kg entlastet | 0,906 | Beidbeinig + 60 kg (0,875) |
| Einbeinig, 2,5 kg entlastet | 0,969 | Einbeinig ohne Gewicht |
| Beidbeinig mit Entlastung | — | Beidbeinig ohne Gewicht |

Liegt die Last im Rest, gewinnt die nähere Seite. Gemessene Sätze bleiben, wie sie sind, mit
ihrer Last und ihrem Etikett. Ein angefangener Lauf verliert die Entlastung. Ein zweiter
Durchlauf ändert nichts.

**Nebenbei.** Stand eine Seite einbeinig und die andere beidbeinig, las der beidbeinige Block
das Gewicht der weiter fortgeschrittenen Seite. Die hatte für beidbeinig nichts gemerkt, und
auf der Karte stand 0 kg. Er liest jetzt die Seite, die beidbeinig ein Gewicht hat.

**Die Handtuchrolle** behält ihre fünf Sprossen mit Waage, bis Luke entscheidet. Nur deshalb
gibt es den Haken „Waage" unter „Geräte" noch.

**Geprüft.** 546 Prüfungen. Neu dazu:

- Gewichtsspanne beider Sprossen bei 60, 80 und 100 kg
- Umrechnung beim Sprossenwechsel
- getrennte Felder je Sprosse
- Schmerz, Regel 2a und zu leicht auf beidbeinig
- Ausführung je Zone bei drei Kurven
- Umzug aus Fassung 18 und 19
- Karte ohne Waage

Im Browser mit einem Stand aus Fassung 19 angesehen: Umzug auf beidbeinig + 20 kg, Karte
beidbeinig mit Rucksack, Wechsel auf einbeinig mit zwei Steppern. Testdaten gelöscht. Build
`2026-09-24 c`.

---

### 2026-09-24 b — Wadenheben: einbeinig als Grundform, Waage und Rucksack stufenlos · Regel 2a rechnet wieder

**Befund (Luke).** Einbeinig ist die Kernvariante und die erste Stufe. Darunter liegt eine
leichtere Form mit Minusgewicht über die Waage, darüber nur noch Zusatzgewicht (P52). Beidbeinig
bleibt, „meinetwegen auch mit Waage". Dazu sein Profil-Screenshot: Die Punkte mit Gewicht lagen
rechts außerhalb der Zeichnung. „zumindest muss alles weiter nach links rücken".

**Die Leiter.** Zwei Sprossen: **Beidbeinig** (50 %) und **Einbeinig** (100 %). An der
einbeinigen geht das Gewicht in beide Richtungen, in Schritten von 2,5 kg:

- **Rucksack**: bis drei Viertel des Körpergewichts, höchstens 60 kg (`extraAnteil`,
  `zusatzDeckel`). Der allgemeine Deckel von einem halben Körpergewicht bleibt für alle anderen
  Übungen.
- **Entlastet**: nur mit Waage (`extraMinGeraet`), höchstens ein halbes Körpergewicht
  (`extraMinAnteil`), bei 80 kg also 40 kg, bei 60 kg nur 30 kg. Ohne Waage endet die Spanne
  bei 0 kg.

Neue Nutzer beginnen einbeinig mit 0 kg (`startStufe`, `standOf`). Wo vorher „|| 0" stand,
rechneten Planer und Regeln bei einem Neuling auf zwei verschiedenen Sprossen: Der Block lief
einbeinig, die Regel nach der Messung stufte von beidbeinig aus.

**Die Karte.** Mit Entlastung steht im Stepper „Waage zeigt 60 kg" statt „20 kg entlastet".
Das ist die Zahl, die der Nutzer auf der Anzeige einstellt. Plus heißt mehr auf der Waage, also
weniger Entlastung. Die drei Knöpfe Waage / Ohne Gewicht / Rucksack wechseln die Richtung, der
Betrag bleibt. Der Name im Verlauf lautet „Einbeinig, 20 kg entlastet" oder
„Einbeinig + 20 kg". Die Stufe steht vorn, sonst wüsste niemand, welche von beiden gemeint ist.

**Die Zeichnung.** Die Lastachse reicht jetzt immer über alle Messpunkte und die jetzige Last
(`gewichtAchse`). Wo das Gewicht die Leiter ist, reicht sie außerdem über die ganze Spanne, die
sich zu Hause umhängen lässt. Unter der Achse stehen neben den Sprossennummern Gewichtsmarken
mit Vorzeichen, bei der Wade −20, +20, +40, +60. Die Legende sagt dazu: „Unter der Zeichnung
stehen die Stufen und, mit Vorzeichen, das Gewicht auf Stufe 2 in kg". Übungen, bei denen das
Gewicht nur ein Aufsatz ist, bleiben so breit wie bisher, solange niemand darauf misst.

**Der Umzug (Fassung 19).** Aus alten Daten wird:

| Alt | Neu |
|---|---|
| 40 % — Waage | Einbeinig, entlastet |
| 50 % — beidbeinig | Beidbeinig |
| 63 % — Waage | Einbeinig, entlastet |
| 79 % — Waage | Einbeinig, entlastet |
| 100 % — einbeinig | Einbeinig |

Jeder Satz behält seine Last exakt. Umgeschrieben werden nur die Sprossennummer und auf den
Waage-Sprossen das Gewicht, zum Beispiel 63 % bei 80 kg → −29,6 kg. Das alte Etikett bleibt im
Verlauf stehen, wie es gemessen wurde. Kein Punkt wird verworfen. Der Leiterstand zieht mit, die
Entlastung dort wird aufs 2,5-kg-Raster gerundet. Ein unterbrochener Lauf zieht ebenfalls um.

**Die Handtuchrolle** bleibt auf ihren fünf Sprossen, bis Luke für sie entscheidet.

**Regel 2a rechnete seit Build `2026-09-18 i` mit einer Last, die es nicht gab.** Die
Nachsteuerung nach einem Satz weit neben dem Ziel las die Last am Satz (`first.load`). Dort
steht sie nicht, sie steht an der Sitzung. Die Ziellast war deshalb `NaN`, mit zwei Folgen:

- Die Schleife brach nie ab und stufte immer um den größten Sprung von drei Sprossen, auch
  wo eine gereicht hätte.
- Auf der obersten Sprosse schrieb sie `NaN` ins Gewicht. Gemeldet wurde „schwerer",
  gespeichert waren 0 kg.

Die Regel liest die Last jetzt an der Sitzung, notfalls aus Stufe und Gewicht gerechnet.

**Beobachtet, nicht geändert.** Liegt die Last unter der kritischen Leistung, sagt die Kurve
„unbegrenzt". Die Gewichtswahl behält dann die vorige Zielzeit. Bei Lukes Kurve (kritische
Leistung ungefähr einbeinig, 20 kg entlastet) bleibt die Karte bei 22,5 kg Entlastung auf
76 s stehen.

**Geprüft.** 504 Prüfungen, davon ein neuer Block zur Wade:

- Leiter und Neuling
- Gewichtsspanne je Körpergewicht und Waage
- Deckel und Namen
- Regeln: zu leicht, Schmerz mit und ohne Waage, beidbeinig zu leicht
- Regel 2a an vier Zeitpaaren
- Umzug von Fassung 18, zweimal hintereinander
- Zeichnung
- Karte

Im Browser mit einem Stand aus Fassung 18 angesehen: Umzug, Zeichnung mit allen Punkten,
„Waage zeigt 60 kg", − auf 57,5, Wechsel auf Rucksack mit 22,5 kg und 66 s. Testdaten
gelöscht. Build `2026-09-24 b`.

---

### 2026-09-24 a — Zwei Menüs bis zum Timer · oben trägt das Gewicht die Zone

**Befund (Luke).** Drei Menüs, bis der Timer läuft, und zwei davon wiederholen sich (P51). Seine
Vorgabe: „am anfang 1 menü um das workout zusammen zustellen dann das zweite menü um die
variation zu wählen … fertig". Auf Rückfrage hat er festgelegt:

- Menü 1 ist der Plan plus eine Häkchenliste.
- Menü 2 kommt einmal je Übung und zeigt beide Seiten.
- Als Extras bleiben Gewicht − / +, Satzzahl − / +, das Übungsbild und Auslassen / Beenden.

**Menü 1 — Heute.** Oben steht die Reihenfolge in einer Zeile, zum Beispiel „Ausfallschritt
rechts → Seitstütz links → …", darunter „Workout starten". Die Dauer steht als „rund 55 min" im
Untertitel. Darunter folgt eine Häkchenliste aller Übungen, die die App vorbelegt.

Gesperrte Übungen sind grau und nicht anklickbar. Den Grund nennt die Zeile:

- „morgen" oder „in 2 Tagen"
- „Gerät fehlt"
- „nicht zusammen mit …"

Anklickbar bleiben zwei Fälle:

- **Offene Gegenseite** („rechts offen").
- **Selbst gewählte Zone gesperrt** („Zone B in 2 Tagen"). Der Haken stellt die Zone dann
  zurück auf automatisch.

Jeder Haken baut den Plan sofort neu, an Ort und Stelle. Die Karte „Bis zum vollständigen
Profil" steht jetzt im Profil-Tab. Am Pausentag steht die Liste unter „Heute nicht" und
„Trotzdem trainieren".

**Menü 2 — die Karte vor der Übung.** Eine Karte je Übung, beide Seiten darauf:

- Oben die Zonenknöpfe (vorgewählt), bei der Wade einmal Einbeinig / Beidbeinig.
- Dann je Seite eine Zeile mit Seite und vorhergesagter Haltezeit. Darunter:
  - Variante, nur wenn es mehr als eine gibt
  - Gewicht − / +, nur wo Gewicht getragen wird
  - Sätze − / +
- Unten das Übungsbild und „Übung starten".
- Dazu die Ausstiege:
  - „Übung auslassen" nimmt beide Seiten heraus.
  - „Workout beenden"
  - „Später weitermachen", sobald etwas gemessen ist.

Die zweite Seite derselben Übung bekommt keine Zonenwahl mehr. Die gilt für die Übung, nicht
für die Seite.

**Entfallen** sind:

- Ausführungstext
- Gewichtshinweise
- Abgleich „Beide Seiten gleich"
- „trotzdem messen"
- Erklärzeilen
- Stufen-Knöpfe − / +
- Zonen- und Ausführungswahl auf Heute
- Hilfebox zur Dauer

Im Quelltext sind damit `zonenWahlBinden`, `bloeckeText`, `ZAHLWORT` und `zweiteSeiteGrund`
weg.

**Links → rechts ohne neue Karte.** Nach dem letzten Satz links steht wie seit `2026-09-21 c`
ein Knopf ohne Uhr, jetzt beschriftet mit „Weiter mit rechts". Er startet direkt den Countdown
für rechts. Die Pause dazwischen bestimmt weiter der Nutzer, `planDauer` rechnet sie
unverändert mit 3 Minuten. Für die direkte Folge muss die zweite Seite auf der Karte
eingestellt worden sein (`eingestellt`). Fehlt diese Marke, kommt wie bisher ihre Karte.

**Der Weg zum Timer:** „Workout starten" → „Übung starten" → Timer. Zwei Taps.

**Oben trägt das Gewicht die Zone** (P50): `gewichtFolgt`, `gewichtRechnet` und `zielKg` sind
neu. `stufenFuerSeite` fasst den Seitenfilter zusammen, den Planer und Karte vorher getrennt
geführt hatten. Im Browser nachgeprüft:

- Wade, Zone B: 32,5 kg, 73 s und 4 Sätze auf beiden Seiten.
- Gewicht + nur links: links 35 kg und 65 s, rechts unverändert.

**Offen.** Wer von Hand Übungen dazu anhakt, kann über den Deckel von 70 Minuten kommen. Die
App zeigt dann zum Beispiel „rund 75 min", sperrt aber nicht. Der Deckel gilt für ihren eigenen
Vorschlag.

**Geprüft.** 424 Prüfungen, darunter ein neuer Block zu beiden Menüs: Liste, Sperrgründe, Karte
mit zwei Seiten, „Weiter mit rechts" und der direkte Start. Im Browser auf Handybreite
angesehen:

- Liste mit „rechts offen" und „morgen"
- Profil-Tab
- Karte mit beiden Seiten
- Auslassen
- Wechsel links → rechts

Testdaten gelöscht. Build `2026-09-24 a`.

---

### 2026-09-21 i — Eine Sitzung unter falschem Etikett verbog die halbe Kurve

Luke hat am 21.09. um 07:08 den Ausfallschritt links in der **Wandstellung** trainiert —
vorderer Fuß mit den Zehen an der Wand, hinterer Fuß zwei Fußlängen dahinter —, um in Zone C
zu landen. Die App zeigte zu diesem Zeitpunkt noch die alte Leiter, und die Sitzung wurde
unter „knapp über den Boden" gespeichert: Sprosse 8, Last 0,90. Gehalten hat er 179,2 s bei
einem Ziel von 91 s, also fast das Doppelte.

Die Zeit ist echt. Falsch ist nur, welcher Stellung sie zugeschrieben war — und das reicht,
um die linke Kurve unbrauchbar zu machen. Nachgerechnet mit dem Fit dieser App:

| der Punkt steht auf | CP | W' | rmse | Kurve gilt von–bis | Ziel Sprosse 8 |
|---|---|---|---|---|---|
| Last 0,90 (falsch) | 0,669 | 35,2 | 0,241 | 0,90–1,25 | **152 s** |
| Last 0,41 (Sprosse 2) | 0,000 | 79,0 | 0,090 | 0,41–1,25 | 88 s |
| Punkt ganz raus | 0,000 | 82,0 | 0,090 | 1,00–1,25 | 91 s |

Bei 0,90 rutscht CP über die halbe Leiter. Der Geltungsbereich schrumpft auf 0,90–1,25,
**sieben von zehn Sprossen bekommen links gar keine Zielzeit mehr**, und ausgerechnet
Sprosse 8 — die Stellung, die er an diesem Morgen nicht trainiert hat — bekäme künftig 152 s
statt 91 s vorgesetzt. Auf ihrer richtigen Sprosse fügt sich dieselbe Messung dagegen ohne
Bruch ein: rmse 0,090, exakt so gut wie die Kurve ohne sie, und sie trägt von 0,41 bis 1,25,
also über neun Sprossen. Der Punkt ist damit nicht nur repariert, er ist der wertvollste der
Reihe — er ist der einzige am unteren Ende.

Eingebaut als `LUNGE_WAND_MESS_AT` (Fassung 18), die über die workout-ID genau diese eine
Sitzung trifft und sonst nichts. Kein `ausKurve`, kein Umrechnen der Haltezeit: korrigiert
wird allein die Sprossennummer und der Lastfaktor, der an ihr hängt. Die Stellung hat Luke
selbst beschrieben; erschlossen habe ich sie nicht.

**Die Last 0,41 ist gerechnet, nicht gemessen.** Sie ist der Leiterwert von Sprosse 2. Was
die Messung dazu sagt, ist weniger, als man hoffen würde — siehe unten.

#### Was 179,2 s über die Wandstellung aussagen — und was nicht

Die Zeit sollte die vier gerechneten Wand-Sprossen kalibrieren. Sie tut es nur grob, und der
Grund dafür ist selbst der interessantere Befund. Vier Wege, aus den vorhandenen Punkten auf
die tatsächliche Last zu schließen, geben vier Antworten:

| Anker | ergibt |
|---|---|
| Kurve links, wie die App sie am 21.09. hatte (1,00 / 1,25) | 0,458 |
| Kniestand links 1,25, frisch, 1:1 übertragen | 0,500 |
| Kniestand rechts 0,55, frisch, 1:1 übertragen | 0,252 |
| Kniestand links 0,55, ermüdet (untere Schranke) | 0,216 |

Spanne 0,22 bis 0,50. Der gerechnete Wert 0,41 liegt darin, wird also nicht widerlegt — aber
auch nicht bestätigt. Als schwaches Indiz dafür, dass er eher richtig als falsch liegt: Setzt
man die Last, bei der sich alle drei linken Punkte am besten zu einer Kurve fügen, landet das
rmse-Minimum bei 0,46 (0,41 → 0,090; 0,46 → 0,073; 0,30 → 0,214). Drei Punkte, davon einer
ein Platzhalter — das trägt kein Urteil, nur eine Richtung.

#### Der Befund darunter: die Kniestand-Lastfaktoren trennen nicht

Der Grund für die breite Spanne steht in den Daten selbst. Rechts, beide Sätze frisch und
erster Satz:

- 15.09., Kniestand über 20 cm, Last 0,55 → **82,0 s**
- 16.09., hinterer Fuß erhöht, Last 1,25 → **81,9 s**

Mehr als die doppelte Last, praktisch identische Haltezeit. Die Leiter behauptet zwischen
diesen beiden Sprossen einen Faktor 2,3; sein Bein misst keinen Unterschied. Links dasselbe
Muster in schwächerer Form (t × L steigt von 38,8 über 74,9 auf 89,6).

Das hat eine plausible mechanische Erklärung: Im aufgebrochenen Kniestand ist das vordere
Knie **per Definition** immer rechtwinklig — über alle vier Kniestand-Sprossen hinweg. Was
sich zwischen ihnen ändert, ist nur, wie viel Gewicht das hintere Bein abnimmt. Der Hebel am
vorderen Knie, der die Haltezeit bestimmt, bleibt gleich. Die Spreizung 0,55 → 1,25 ist
gegriffen, und sie ist offenbar zu weit gegriffen.

Das betrifft nicht nur die Kalibrierung der Wand-Sprossen, sondern die Zonensteuerung der
ganzen Kniestand-Hälfte: Wenn Sprosse 5 und Sprosse 10 dasselbe kosten, führt ein Sprossen-
wechsel dort nicht in eine andere Zone. **Offen** — zu klären ist es nur durch Messung: einen
Tag Sprosse 5 links frisch, einen Tag Sprosse 8 links frisch, beide bis zum Versagen. Bis
dahin bleiben die Faktoren stehen, wie sie sind; sie zu ändern, ohne zu wissen wohin, machte
es nicht besser.

#### Nebenbefund, nicht behoben

Der älteste Punkt der linken Reihe (14.09., „beide Hände abgestützt, flach") trägt die Last
1,00 — ein Platzhalter aus der Fassung vor den Lastfaktoren. Er zieht die Kurve, aber kaum:
mit ihm 193/144/88 s für die Sprossen 2/5/8, ohne ihn 198/147/90 s. Unter fünf Prozent.
Verworfen wird er nicht — die Entscheidung darüber gehört Luke, und dringend ist sie nicht.

---

### 2026-09-21 h — Die Ausfallschritt-Leiter läuft über den Fußabstand

**Was Luke gemeldet hat.** Er hat beim Training den vorderen Fuß an die Wand gestellt, so dass
die Kniescheibe beim Absenken vor der Wand schwebt, und den hinteren Fuß zwei Fußlängen von der
Wand weg. Sein Urteil über die alte Leiter: „hinteres Knie bis Stuhlsitz" macht keinen Sinn, und
„bis Hocker" auch nicht, weil die meisten Menschen keinen so langen Unterschenkel haben.

**Warum die Wand mehr ändert als die Tiefe.** Die alte Leiter stufte über eine Ablage unter dem
hinteren Knie. Die legt die Tiefe fest, aber nicht den Knievorschub — und beim Knie entscheidet
genau der über die Last. Steht die Kniescheibe über dem Knöchel, trägt vor allem das Gesäß;
steht sie eine Fußlänge davor, kippt die Last ins Knie. Beides hieß in der alten Leiter
„knapp über den Boden".

**Zwei Bewegungen, nicht eine.** Lukes bestehende Messpunkte stammen aus dem *aufgebrochenen
Kniestand*: vorderes Knie bei 90°, Rumpf aufrecht, hinteres Knie frei schwebend knapp über dem
Boden oder über seinem Stepper (dessen Höhe er noch nachmisst). Die neue Sprosse ist ein
*stehender* Ausfallschritt an der Wand. Nachgerechnet (Modell `scratchpad/lunge6.py`, H = 1,80 m)
sind das nicht zwei Tiefen derselben Übung: Steht die Kniescheibe eine Fußlänge vor der Ferse an
der Wand und ist das vordere Knie bei 90°, liegt die Hüfte auf 67 cm — und das hintere Knie kommt
dann bei *keinem* Fußabstand unter 23 cm über den Boden. Ein hinteres Knie knapp über dem Boden
bei 90° vorn verlangt ein nahezu senkrechtes Schienbein, also fast keinen Knievorschub. Lukes
Punkte stammen damit aus einer knieärmeren, hüftlastigeren Stellung als der neuen. Er hat
entschieden, sie nicht zu verwerfen; sie bleiben in der Kurve und tragen ihr gespeichertes `load`
mit, kein `ausKurve` gesetzt.

**Was gebaut ist.** Zehn Sprossen, und die Leiter trägt beide Stellungen nebeneinander. Vorn
vier stehende an der Wand, gestuft über den Abstand des hinteren Fußes in eigenen Fußlängen
(1,5 / 2 / 2,5 / 3). Dahinter vier im Kniestand über Lukes Stepper, der drei Höhen hat: 3 Stufen
= 20 cm, 2 Stufen = 15 cm, 1 Stufe = 10 cm, dann der Boden — höher heißt weniger tief heißt
leichter. Oben die beiden bisherigen mit erhöhtem hinteren Fuß. `EX_VERSION` 16 → 17.

**Die gemessenen Lasten sind unverändert.** 0,55 / 0,72 / 0,90 / 1,05 / 1,25 stehen alle noch da,
nur unter einer neuen Nummer — die alten Ablagen sind durch die Stepperhöhen ersetzt, die ihnen
am nächsten kommen (Kiste ~20 cm → 3 Stufen, dickes Kissen ~12 cm → 1 Stufe). Neu dazwischen ist
allein 0,63 für die mittlere Stepperhöhe, interpoliert.

**Warum die Sprossennummern umziehen mussten.** Die gemessenen Sprossen sind um drei nach hinten
gerückt. `ankerpunkt()` gibt die Nummer des jüngsten Punktes zurück, und `zielNeu()` baut daraus
die Obergrenze `anker.level + MAX_SPRUNG`. MAX_SPRUNG ist 3 — genau die Verschiebung. Ohne Umzug
käme, wer zuletzt „knapp über den Boden" gehalten hat, nur noch bis zur neuen Sprosse 7 statt 10
und bekäme im nächsten Workout eine zu leichte. `LUNGE_UMZUG` zieht deshalb `levels` *und* die
`level`-Nummer in den gespeicherten Sätzen um; die Last wird dabei nicht angefasst, und es wird
kein `ausKurve` gesetzt.

**Die vier Wand-Sprossen sind gerechnet, nicht gemessen.** Sie kommen aus dem vorderen Kniewinkel,
den die Wandstellung erzwingt (56° / 64° / 72° / 82°, Modell `scratchpad/lunge4.py`, H = 1,80 m),
linear darauf skaliert und über einen Faktor 0,65 an die Kniestand-Sprossen angeschlossen. Der
Faktor ist die Begründung, warum der Stand bei gleicher Beugung leichter ist: im aufgebrochenen
Kniestand schwebt das hintere Knie und der hintere Fuß trägt nur über die Zehen, im Stand trägt
das hintere Bein über den Ballen mit — gerechnet liegen dort 62 bis 67 % des Körpergewichts vorn.
Lukes gestrige Sitzung (2 Fußlängen) landet damit auf Sprosse 2, genau wo er sie vermutet hat.

**Was das Modell nicht trägt.** Die Gewichtsverteilung auf die Füße trennt die Wand-Sprossen
nicht — sie bleibt über die ganze Leiter bei 62 bis 67 % vorn. Eine Personenwaage unter dem
vorderen Fuß, wie beim Wadenheben, hilft hier also nicht. Was die Sprossen trennt, ist der
Kniewinkel.

**Offen.** Ob der Übergang zwischen den beiden Stellungen an der richtigen Stelle sitzt —
0,52 (3 Fußlängen, stehend) auf 0,55 (Kniestand über 20 cm) ist der kleinste Schritt der Leiter
mit 6 %, und er überspringt einen Stellungswechsel. Das entscheidet erst die Messung.

### 2026-09-21 g — Das Zonen-Etikett sagt jetzt, was der Block wirklich hält

**Befund (Luke).** „Beim Ruderzug steht Zone A 30–60 Sekunden als Richtwert für den ersten Satz,
aber 90 Sekunden — das widerspricht sich."

**Vorweg zur Zahl:** Zone A ist in der App **20–60 s**, nicht 30–60. Eine „30" kommt in keinem
Zone-A-Zusammenhang im Code vor; der Knopf ist mit „20–60 s" beschriftet. An der Sache ändert das
nichts — der Widerspruch war echt.

**Zwei Zahlen unter einem Namen.** `bl.target` trug bis hierher zwei Bedeutungen. Es geht als
**Zonen-Sollwert** in die Blockplanung (Zonenmitte: A = 40 s, B = 75 s, C = 120 s) und kommt als
**Vorhersage für die tatsächlich gewählte Sprosse** wieder heraus (`predictTime` auf der
Kraftkurve). Beides wurde als „Richtwert erster Satz" neben die Zonen-Plakette gerendert.

**Warum die Sprosse die Zone verfehlt.** Die Blockplanung wählt nur unter den Sprossen, die im
**gemessenen Bereich** der Kurve liegen. Beim Schrägzug standen nach dem Verwerfen der alten Punkte
(Build `f`) genau zwei Punkte in der Kurve — 0,502 bei 130 s und 0,598 bei 100 s. Zwischen diesen
beiden Sprossen liegt kein Halt bei 40 Sekunden. Die App nahm die am wenigsten schlechte und ließ
das Etikett „Zone A" stehen. Das ist kein Schrägzug-Sonderfall: Es trifft jede Übung, deren Leiter
im gemessenen Bereich grob ist, und es trifft auch die automatische Zonenwahl — nicht nur die,
die Luke selbst setzt.

**Was ausdrücklich nicht geändert wurde: die Zahl.** Der naheliegende Griff wäre, die 90 auf das
Zonenfenster zu klemmen. Das hätte die Selbstkorrektur abgeschaltet: `applyRules` misst den ersten
Satz gegen `target` und steigt eine Sprosse hoch oder runter, wenn er um mehr als 20 % danebenliegt.
Bei 90 s liegt die Verfehlungsschwelle bei 72 s; eine geklemmte 60 würde sie auf 48 s schieben —
also genau die Korrektur lahmlegen, die die Sprosse überhaupt erst in Richtung Zone zieht. Dazu
rechnet `planDauer` die Workoutlänge aus `target`; eine geschönte Zahl hätte Lukes 30–70-Minuten-
Deckel verzerrt. Die Erwartung bleibt die Zahl, die zählt.

**Geändert ist stattdessen dreierlei.**

1. **Der Block trägt jetzt beide Zonen.** `zone` bleibt die angeforderte (das Rotationsziel, das
   die Messpunkte über die Zonen verteilt), neu daneben steht `zoneErwartet` — die Zone, in der
   die gewählte Sprosse laut Kurve landet. Die Plakette auf Workoutkarte und Intro-Karte zeigt die
   **erwartete**. Wo beide auseinanderfallen, steht darunter ein Satz: „Geplant war Zone A (40 s) —
   dazwischen hat diese Leiter keine Sprosse."
2. **Die Zahl sagt, was sie ist.** Statt „Richtwert erster Satz" steht dort „erwartete Haltezeit",
   sobald die Vorhersage aus der Kurve kommt und nicht aus dem Zonen-Sollwert.
3. **Die Satzzahl folgt der Erwartung, nicht dem Etikett.** Fünf Sätze à 100 s sind kein
   Zone-A-Block, sondern 12 Minuten Ausdauer. Der Block fällt damit von 5 auf 3 Sätze — das ist
   die Vorgabe für Zone C, und sie hält den Deckel ein, den Luke gesetzt hat („lass den Deckel so,
   dass man 30–70 Minuten trainiert").

Dasselbe gilt bei jeder Änderung von Hand: Wechselt Luke Sprosse oder Zusatzgewicht, zieht
`zielNeu()` die erwartete Zone mit — und eine von Hand gesetzte Satzzahl bleibt dabei stehen.

**Zwei Nachzügler aus Build `f`, im selben Zug repariert.** Beide betreffen Sitzungen, die dort
mit `ausKurve: "alte Skala"` aus der Kurve genommen wurden:

- `ankerpunkt()` las sie weiter mit. Beim Schrägzug hieß das nach dem Faktorwechsel eine bis zu
  **2,6-fach zu schwere** Sprosse im ersten Workout danach.
- Der Export markierte sie weiter als `in_kurve: 1`, obwohl `points()` sie überspringt.

**Ohne Fassungssprung.** `EX_VERSION` bleibt 16. Ein Sprung würde jeden von Hand umbenannten
Stufennamen und jeden bearbeiteten Lastfaktor verwerfen; hier war keine Datenwanderung nötig.
Pläne, die in einem unterbrochenen Workout überleben, kennen `zoneErwartet` noch nicht und
bekommen beim Laden die angeforderte Zone eingetragen.

**Geprüft.** 372 Prüfungen, davon 26 neu für diesen Weg — vorher hatte die Testreihe **keine**
einzige Prüfung auf `predictTime → target`. Lukes Fall in der laufenden App nachgestellt: Block
plant Zone A (Soll 40 s), erwartet Zone C (100 s), fällt auf 3 Sätze, Karte trägt beide Zonen.

### 2026-09-21 f — Die Arbeitsposition ist der Lastfaktor · alte Punkte verworfen

**Befund (Luke), auf die Rückfrage aus `e`.** „Nein, ich habe die Zugposition mit Ellenbogen 45°
neben dem Körper gemessen, also genau das, was du vorgegeben hattest."

**Die Spaltenüberschrift hat mich in die Irre geführt.** In der Excel heißt die zweite Spalte
„Pull up position". Ich habe sie als den oberen Umkehrpunkt gelesen — das, was der Hinweistext
ausdrücklich nicht will („nicht oben am Griff kleben") — und deshalb in Build `e` das **Mittel**
aus Hang und dieser Spalte eingebaut. Falsch: Die Spalte **ist** die Arbeitsposition, Ellenbogen
rechtwinklig, Oberarme rund 45° vom Rumpf. Genau die Stellung, die das Arbeitsblatt zu messen
verlangt hatte.

Damit ist der Lastfaktor keine Schätzung mehr, sondern ein Messwert:

| Nr. | Stellung | Hang | Arbeit | Faktor |
|---|---|---|---|---|
| 1 | 2 Füße vor · stehend | 24 kg | 6 kg | **0,072** |
| 2 | 3 Füße vor · stehend | 30 kg | 16 kg | **0,191** |
| 3 | 4 Füße vor · stehend | 36 kg | 24 kg | **0,287** |
| 4 | 5 Füße vor · stehend | 44 kg | 36 kg | **0,431** |
| 5 | 1 Fuß vor der Aufhängung · Knie 90° | 47 kg | 42 kg | **0,502** |
| 6 | Füße unter der Aufhängung · Knie 90° | 52 kg | 50 kg | **0,598** |
| 7 | Schultern unter der Aufhängung · Knie 90° | 54 kg | 60 kg | **0,718** |
| 8 | Schultern unter der Aufhängung · gestreckt | 62 kg | 70 kg | **0,837** |

**Die Leiter wird dadurch besser, nicht schlechter.** Die vier oberen Sprünge liegen bei 16, 19,
20 und 17 % — die gleichmäßigste Leiter der App. Unten ist sie grob (+165 %, +50 %, +50 %), und
Sprosse 1 trägt nur 6 kg. Das ist kein Fehler, sondern der Einstieg: Wer dort anfängt, springt
schnell durch, ab Sprosse 5 wird es fein.

**Wie weit die App bisher danebenlag.** Die alte Rechnung `0,746 · cos θ` traf den **Hang** auf
1 bis 4 %. Gegen die Arbeitsposition gerechnet, war sie unten fast doppelt zu hoch und oben zu
niedrig:

| Stellung | App bisher | gemessen | Fehler |
|---|---|---|---|
| 3 Füße vor | 0,370 | 0,191 | **1,94-fach zu hoch** |
| 4 Füße vor | 0,450 | 0,287 | 1,57-fach zu hoch |
| 5 Füße vor | 0,520 | 0,431 | 1,21-fach zu hoch |
| Füße unter der Aufhängung | 0,620 | 0,598 | 1,04-fach zu hoch |
| Schultern unter, 90° | 0,660 | 0,718 | 8 % zu **niedrig** |
| Schultern unter, gestreckt | 0,750 | 0,837 | 10 % zu **niedrig** |

Der Fehler ist keine Konstante, sondern dreht auf halber Leiter das Vorzeichen. Genau deshalb war
Umrechnen keine Option.

**Die alten Messpunkte sind verworfen** (Lukes Entscheidung). `ROW_SKALA_AT = 16`: Jede
Schrägzug-Sitzung, die vor dem Faktorwechsel gespeichert wurde, bekommt beim Laden das Feld
`ausKurve: "alte Skala"`; `points()` überspringt sie. **Im Verlauf bleiben die Blöcke stehen** —
abgeblendet, mit dem Grund unter dem Sprossennamen und einer erweiterten Legende. Ein still
verschwundener Trainingstag ist schlimmer als ein erklärter.

Drei Dinge waren dabei wichtig. Erstens **verschiebt dieser Umzug keine Sprosse**: Die acht
Stellungen bleiben, nur ihre Last wird neu beziffert — eine Umzugstabelle wäre hier falsch
gewesen. Zweitens trägt das Feld den **Grund als Text**, nicht ein blankes `true`, damit ein
späterer Skalenwechsel nicht dieselbe Markierung überschreibt. Drittens wird, wer schon auf
Fassung 16 gemessen hat, **nicht ein zweites Mal** entwertet.

**Nicht `fresh: false` missbraucht.** Es gibt bereits ein Feld, das eine Sitzung aus der Kurve
nimmt — aber es bedeutet „vorermüdete zweite Seite" und schreibt genau das in den Verlauf. Zwei
verschiedene Gründe unter einem Flag hätten die Anzeige zur Lüge gemacht.

**Zwei geschätzte Winkel aus den Bildtexten entfernt.** „Rund 60°" und „rund 40°" standen in den
Bildunterschriften der Sprossen 3 und 4. Beide waren meine Rechnung — die Schulterhöhe wurde bei
der Messung nie erhoben. Ersetzt durch das, was auf dem Foto zu sehen ist.

**Bestätigt (Luke, 21.09.):** Die fünf Fotos liegen richtig (Sprosse 1 bis 5). Die Ringhöhe bleibt
fest bei 85 cm — er verschiebt den Körper gegen die Aufhängung, nicht die Ringe.

**Geprüft.** 344 Prüfungen, davon zehn neu für die Verwerfung. Im Browser gegengeprüft: Ein Gerät
auf Fassung 15 mit zwei alten Schrägzug-Messungen zeigt „2 Messungen · 0 davon in der Kurve", beide
Zeilen abgeblendet mit „alte Skala" unter dem Sprossennamen, Sprosse unverändert.

### 2026-09-21 e — Die Schrägzug-Leiter ist gemessen · Fotos je Sprosse

> **Die Faktorentabelle in diesem Eintrag ist durch `f` ersetzt.** Sie enthält das Mittel aus
> beiden Messspalten; richtig ist die Arbeitsspalte allein. Der Eintrag bleibt stehen, weil der
> Rest (Fotos je Sprosse, Umzugskette, Gedränge-Befund) unverändert gilt.

**Befund (Luke).** „Ich habe jetzt mit dem Tindeq die Messreihe gemacht, hier die Ergebnisse in
der Excel und die passenden Fotos dazu. Leg die Bilder gerne schon mal als Prototyp in der App an,
ich liefere sie irgendwann in guter Qualität einmal nach."

**(a) Acht gemessene Sprossen ersetzen sieben gerechnete.** Gemessen am 21.09.2026 bei **83,6 kg**
Körpergewicht und **85 cm** Ringhöhe, mit dem Tindeq an den Ringen. Luke hat je Stellung **zwei**
Werte genommen: im Hang (Arme gestreckt) und oben (Zugposition). Er hat dabei auch die Benennung
geändert — gezählt wird in **Fußlängen**, und die vier liegenden Stellungen messen nicht mehr die
Ringhöhe, sondern die Lage des Körpers gegen die Aufhängung.

| Nr. | Stellung | Hang | oben | **Faktor (Mitte)** |
|---|---|---|---|---|
| 1 | 2 Füße vor · stehend | 24 kg | 6 kg | **0,179** |
| 2 | 3 Füße vor · stehend | 30 kg | 16 kg | **0,275** |
| 3 | 4 Füße vor · stehend | 36 kg | 24 kg | **0,359** |
| 4 | 5 Füße vor · stehend | 44 kg | 36 kg | **0,478** |
| 5 | 1 Fuß vor der Aufhängung · Knie 90° | 47 kg | 42 kg | **0,532** |
| 6 | Füße unter der Aufhängung · Knie 90° | 52 kg | 50 kg | **0,610** |
| 7 | Schultern unter der Aufhängung · Knie 90° | 54 kg | 60 kg | **0,682** |
| 8 | Schultern unter der Aufhängung · gestreckt | 62 kg | 70 kg | **0,789** |

**Warum das Mittel und nicht eine der beiden Spalten.** Der Hinweis der Übung schreibt seit jeher
die Mitte vor: „Ellenbogen etwa rechtwinklig — nicht mit gestreckten Armen hängen und nicht oben am
Griff kleben." Gemessen sind die beiden **Enden** dieser Bewegung; der Lastfaktor ist deshalb ihr
Mittel. Das ist eine Schätzung, kein Messwert — über den Zugweg läuft die Last nicht exakt linear.
Eine einzige Kontrollmessung in der Arbeitsposition auf Sprosse 4 oder 5 würde das klären.

**Die Spanne ist der Preis der Armstellung.** Unten 4-fach (24 → 6 kg), oben 4 %. Ab Sprosse 7
kippt das Vorzeichen: dort ist die Zugposition **schwerer** als der Hang, weil der Schwerpunkt über
die Ringebene gehoben wird, statt unter ihr zu pendeln. Praktisch heißt das: Auf den unteren
Sprossen ist die Ellenbogenstellung die halbe Übung, und wer dort ungenau hält, bekommt Streuung in
die Kurve, die nach Tagesform aussieht und keine ist.

> **Überholt am 21.09.2026, abends.** Dieser Absatz macht aus einem Quotienten eine Regel und
> benennt die falsche Ursache. Korrektur im Nachtrag zu Punkt 27.

**Die alte Rechnung war gut, der Bezugspunkt war falsch.** `0,746 · cos θ` trifft die **Hang**-Spalte
auf 1 bis 4 % genau (0,370 gegen 0,359; 0,450 gegen 0,431; 0,520 gegen 0,526; 0,620 gegen 0,622;
0,750 gegen 0,742). Der Kosinus stimmte also — gerechnet war nur der Hang, gehalten wird die Mitte.
Deshalb liegen die neuen Faktoren trotzdem deutlich tiefer, auf der leichtesten Sprosse um 38 %.

**Die Gedränge-Warnung aus `d` hat sich bestätigt und aufgelöst.** Im Hang liegen 4 → 5 nur 7 % und
6 → 7 nur 4 % auseinander — dort wären es tatsächlich keine getrennten Sprossen. Im Mittel sind die
kleinsten Sprünge 11 % und 12 %, weil die Zugposition oben stärker spreizt als der Hang. Die Leiter
trägt: unten grob (+54 %, +31 %, +33 %), oben fein (11–16 %). Das ist der Verlauf, den eine
Progression haben soll.

**Der zweite Umzug.** `EX_VERSION` steigt auf **15**. `ROW_MESS_UMZUG` schiebt die gespeicherte
Stufe auf die neue Leiter: 0–4 bleiben, 5 → 6, 6 → 7. Die alte Sprosse 5 („Flach · Aufhängung auf
Schulterhöhe") und 6 („Flach · Aufhängung tief") heißen jetzt „Füße unter der Aufhängung" und
„Schultern unter der Aufhängung"; die neue Sprosse 1 (2 Füße vor) ist unten dazugekommen, deshalb
verschiebt sich oben alles um eins. `rowUmziehen()` nimmt die Tabelle jetzt als Argument, damit ein
Gerät, das noch auf Fassung 13 steht, **beide** Umzüge nacheinander fährt — erst `ROW_UMZUG`, dann
`ROW_MESS_UMZUG`. Im Browser geprüft: Fassung 13 mit `row: 5` landet auf Sprosse 7 („Schultern unter
der Aufhängung · gestreckt"), die anderen Übungen bleiben unberührt.

**Was die Messung nicht trägt.** Die 85 cm Ringhöhe sind Teil der Messung. Hängt Luke die Ringe um,
stimmen die vier liegenden Sprossen nicht mehr — das steht jetzt auch im Hinweistext der Übung.

**(b) Fotos je Sprosse — eine neue Bauart für `UEBUNGSBILDER`.** Bisher hatte jede Übung **ein**
Bild. Beim Schrägzug **ist** die Stellung die Sprosse, ein einzelnes Bild wäre für sieben von acht
Sprossen falsch. `UEBUNGSBILDER.row` hat deshalb `stufen: {0: …, 1: …}` statt eines Bildes, und
`uebungsbild(key, level)` bekommt die Stufe übergeben und wählt aus. Alle anderen Übungen laufen
unverändert über die alte Bauart weiter.

Fünf Fotos von Luke, zugeschnitten und auf 560 px Breite gerechnet (je rund 34 KB, als
Data-URI eingebettet wie alle Bilder — die App bleibt eine Datei). Sie liegen auf den Sprossen 1
bis 5. **Die Sprossen 6, 7 und 8 zeigen kein Bild**, statt das Foto der Nachbarsprosse zu borgen:
Bei dieser Übung wäre das nicht ungefähr richtig, sondern eine andere Übung. Die Bildunterschrift
nennt die Herkunft und den Prototyp-Status.

Auf Sprosse 5 steht eine rote Warnzeile unter dem Bild: Das Foto zeigt den Hang mit gestreckten
Armen, gehalten wird aber mit rechtwinkligen Ellenbogen. Lieber die Abweichung benennen als ein
Bild zeigen, das dem Hinweistext widerspricht.

**Geprüft.** 334 Prüfungen im Testlauf, davon neu: die acht gemessenen Faktoren, die Namen in
Fußlängen, kein Sprung unter 10 %, die Umzugskette über beide Tabellen, die Bilder je Sprosse, das
Fehlen der Bilder auf 6–8, und die Sperre gegen die temporale Todeszone bei `ROW_MESS_AT`. Im
Browser gegengeprüft: Das Foto wechselt mit der Sprosse, verschwindet ab Sprosse 6, das
Zusatzgewicht erscheint weiter ab Sprosse 5.

**Was offen bleibt.** Messpunkte von **vor** heute tragen ihre Last eingefroren auf der alten Skala,
die eine andere Bezugsposition hatte (Hang statt Mitte) — sie sind systematisch zu schwer beziffert.
Stehen lassen, umrechnen oder verwerfen ist Lukes Entscheidung; der Anhang des Messprotokolls
beziffert die drei Wege.

### 2026-09-21 d — Neue Schrägzug-Leiter · „Für später unterbrechen" · offene Gegenseite

Drei Befunde aus einem Satz von Luke: „Ich hatte beim Schrägzug gestern eine neue Leiter angegeben,
diese taucht in der App nicht auf. Ich habe das Workout abgebrochen … nachdem ich das Workout
abbreche wäre es schön wenn ab dem Schrägzug das Workout noch angezeigt werden würde, z. B. wenn
ich am Abend weiter trainieren will. Jetzt steht ein völlig neues Workout da und die rechte Seite
wird nicht nachgeholt."

**(a) Die Leiter stand nur im Papier.** Sie war als Teil 4 Punkt 26 notiert, weil die Faktoren
fehlten — in der App lagen weiter die sechs alten Winkelnamen. Jetzt stehen Lukes sieben
Stellungen drin, mit **vorläufigen** Faktoren:

| Nr. | Stellung | Faktor | Herkunft |
|---|---|---|---|
| 1 | 3 Schritte vor | 0,370 | `0,746 · cos 60,3°` |
| 2 | 4 Schritte vor | 0,450 | `0,746 · cos 52,9°` |
| 3 | 5 Schritte vor | 0,520 | `0,746 · cos 45,8°` |
| 4 | 6 Schritte vor | 0,580 | `0,746 · cos 39,0°` |
| 5 | Flach · Aufhängung auf Schulterhöhe · Knie 90° | 0,620 | unter 0,66, leicht aufgerichtet |
| 6 | Flach · Aufhängung tief · Knie 90° | 0,660 | Winter-Segmentmassen, Punkt 26 |
| 7 | Flach · Beine gestreckt | 0,750 | k selbst, gerundet |

Ein Schritt macht rund 7° flacher; das ist die Annahme hinter den vier Winkeln. Alle sieben
Sprossen hängen weiter am Gerät `zug`, nicht an `ringe` — sonst verschwände die Übung für jeden,
der den Ringe-Haken nicht gesetzt hat, und der Zug-Haken sagt ausdrücklich „ohne das entfällt die
Übung ganz". Die Ringe stehen in den Sprossennamen als „Aufhängung", damit die Leiter auch am
Schlingentrainer oder an der Stange lesbar bleibt.

**Der Leiterumzug.** `EX_VERSION` steigt auf 14, und `rowUmziehen()` schiebt die gespeicherte
Stufe auf die nächstgelegene neue Last: alt 1–3 → neu 1, alt 4 → neu 3, alt 5 → neu 5, alt 6 →
neu 7. Die drei untersten alten Sprossen fallen zusammen, weil die neue Leiter bei 0,370 anfängt
statt bei 0,155 — wer dort stand, findet die erste Sprosse schwerer vor als die, die er zuletzt
gehalten hat. Ein Leitersturz wäre das falsche Werkzeug gewesen: er hätte auch die sieben anderen
Übungen auf null gesetzt.

**Zwei Warnungen, die bleiben.** Erstens drängen sich die Sprossen 4 bis 6 innerhalb von 14 % —
`cos θ` ist nahe der Waagerechten flach, derselbe Befund wie bei P20. Es kann sein, dass das keine
drei Sprossen sind, sondern eine. Zweitens frieren Messpunkte ihre Last beim Speichern ein: Was
jetzt auf diesen Zahlen gemessen wird, behält sie dauerhaft, und die Kurve bekäme einen Knick,
sobald die Tindeq-Messung die Faktoren ersetzt. Punkt 26 bleibt offen, bis gemessen ist.

**(b) „Für später unterbrechen" — der dritte Weg.** Bisher gab es an den Abbruchstellen nur
„Workout hier beenden und speichern" und „ganz verwerfen". Beenden ist semantisch *fertig*: es
schreibt die Sitzungen und löscht Plan und Lauf — danach steht am Abend ein neu gerechnetes
Workout da. `workoutUnterbrechen()` schreibt **nichts** und lässt alles stehen, wo es ist: den halb
gelaufenen Block, die Reihenfolge, die eingestellten Varianten. `RUN` wird gelöst (sonst zeigt die
Startseite weiter den Tagesplan, weil die Lauf-Karte auf `!RUN` prüft), der Lauf lebt in `S.lauf`
weiter, und die Karte „Workout läuft noch · Weitermachen" führt zurück. Der Knopf steht an allen
drei Stellen: vor einer Übung, in der Satzpause, am Blockende.

**Die Grenze ist Mitternacht** (Lukes Entscheidung). `renderHome` bietet „Weitermachen" nur an,
solange `L.tag === today()`; danach bleibt die Karte, aber nur noch zum Abschließen. Eine Messung,
die zwölf Stunden alt ist, gehört nicht in dieselbe Sitzung.

**(c) Die offene Gegenseite hat Vorrang vor der Sperre.** Lukes Workout war Ausfallschritt links,
Wade links, Liegestütz, Schrägzug, Ausfallschritt rechts, Wade rechts. Nach dem Abbruch war links
gemessen, rechts nie — und am nächsten Tag sperrte die Erholungsregel beide Übungen, weil die
**Gruppe** trainiert war. Die rechte Seite fiel still unter den Tisch.

`offeneSeite(ex)` findet den Fall: jüngster Trainingstag der Übung, welche Seiten dort gemessen
wurden, und ob genau eine fehlt. Ein Block ohne Seite ist der beidbeinige Halt und zählt für beide
Kurven, lässt also nichts offen. Das Fenster ist so lang wie die Sperre selbst — danach ist die
Übung ohnehin wieder dran, und `startSeite()` stellt die schwächer gemessene Seite von sich aus
nach vorn. Der Vorrang ist ein Nachholfenster, keine Dauerregel.

Vier Stellen mussten mitziehen, sonst hätten Teile der App einander widersprochen:
`heuteDran()` lässt die Übung durch; `gate()` auch, sonst stünde „Pausentag — heute nicht", während
der Plan schon einen Nachholblock hält; `auswahlHeute()` zählt den Nachholblock als **eine** Seite
und stellt ihn vor die Wartezeit-Liste; `zoneFuer()` erbt Zone und Zielzeit der Gegenseite und
setzt `fest`, damit die Lückenlogik sie nicht wegdreht — sonst stünde auf der Karte Zone A und im
Block Zone B. Zwei Seiten in zwei Zonen ließen sich nicht nebeneinanderlegen, und genau dafür wird
nachgeholt. Der Grund steht sichtbar auf der Plankarte („Nachgeholt: Ausfallschritt · rechts …")
und in der Häkchenliste („rechts ist noch offen — wird heute nachgeholt" statt „heute schon
trainiert").

Die gemessene Seite bleibt in der Pause: Der Plan baut **einen** Block, nicht zwei.

### 2026-09-21 c — Die normale Pause zwischen links und rechts ist zurück

**Befund (Luke).** „Bau wieder die normale Pause zwischen links/rechts Wadenheben ein."

**Zurückgebaut.** Zwischen zwei Blöcken steht wieder ausnahmslos die Blockpause (3 Minuten) —
auch zwischen den beiden Seiten derselben Übung. Damit fällt alles, was auf der kurzen Pause
aufsaß: der Durchlauf aus `2026-09-21 b` samt Ansage „Seite wechseln" und Notausgang „Erst noch
einstellen", und `blockPauseS()` selbst. `planDauer` rechnet für jeden Blockwechsel wieder mit
`PAUSE_BLOCK_S`.

**Was bewusst stehen bleibt — und warum das kein halber Rückbau ist.** `seitenParallel` hatte drei
Wirkungen; nur eine davon war die Pause. Die anderen beiden bleiben: Beide Seiten stehen
nebeneinander im vorderen Teil des Workouts, und beide sind `fresh`, werden also gemessen. Das
ist die physiologische Aussage („eine Wade ermüdet die andere nicht"), und die ist von der
Pausenlänge unabhängig — eine längere Pause macht die zweite Seite nicht schlechter messbar,
sondern höchstens besser. Läge sie dagegen wieder am Ende des Workouts, wäre sie vom übrigen
Training vorermüdet und als Messpunkt wertlos.

**Das ist die eigentliche Lehre aus P49, einmal in die andere Richtung gelesen.** Eine Eigenschaft
wie `seitenParallel` bündelt leicht mehrere Folgen — Reihenfolge, Messbarkeit, Pause. Wird eine
davon zurückgenommen, müssen die anderen einzeln geprüft werden, statt das Flag pauschal
abzuschalten. Ein pauschales `seitenParallel = false` hätte hier die Messung der zweiten Seite
mit erledigt, ohne dass jemand danach gefragt hätte.

**Folge für die Zeit.** Das Wadenpaar in Zone A kostet wieder rund 18 statt 15 Minuten; der
Deckel von 30–70 Minuten bleibt unberührt.

**Texte nachgezogen.** Die beiden Hilfetexte, die seit `2026-09-21 a` eine Ausnahme von den
„3 Minuten zwischen zwei Blöcken" nannten, nennen sie nicht mehr.

**Geprüft.** 272 Prüfungen (Block 22 entfernt, die Pausentests umgestellt), dazu im Browser: nach
dem letzten Satz links wieder der Weiter-Knopf ohne Uhr, „Weiter" legt den linken Block ab und
führt auf die Karte für rechts; Dauer des Paares 18 Minuten.

---

### 2026-09-21 b — Zwei Seiten laufen auch zwei Seiten lang

**Befund (Luke).** „Mach es noch so, dass das isometrische Wadenheben links und rechts direkt
hintereinander läuft."

**Was gemeint war.** Im *Plan* standen die beiden Seiten seit `2026-09-21 a` schon nebeneinander.
Gemeint war der *Ablauf*: Letzter Satz links → Knopf „Weiter" → die volle Karte vor der Übung →
Knopf „Los". Drei Tipps für einen Fußwechsel, und dazwischen eine Karte, auf der für die zweite
Seite nichts mehr zu entscheiden ist — Zone, Sprosse und Ausführung stehen bereits.

**Die Änderung.** Nach dem letzten Satz eines Blocks sieht die App auf den Block danach. Sagt
`blockPauseS()` für dieses Paar die **Satzpause** voraus (20 s statt 180 s, also: dieselbe Übung
mit `seitenParallel`), läuft die Pause durch und die zweite Seite beginnt von selbst.

**Warum kein eigenes Kriterium.** Der Ablauf fragt genau die Funktion, die auch die Workoutzeit
rechnet. Damit können Ablauf und Zeitangabe nicht auseinanderlaufen: Was der Plan mit 20 s
veranschlagt, läuft auch als 20 s; was er mit 180 s veranschlagt, bekommt weiter die Karte. Eine
zweite, eigenständig formulierte Bedingung wäre genau die Stelle, an der beide über die Zeit
auseinanderdriften — derselbe Fehlertyp wie in P27, wo Anzeige und Rechnung zwei Quellen hatten.

**Die Pausenkarte sagt an.** Statt „Sekunden bis zum nächsten Satz" steht dort „Sekunden bis zur
anderen Seite", darunter *Seite wechseln* mit Name und Sprosse des nächsten Blocks und dem
Hinweis, dass diese Seite eben nicht mitgearbeitet hat und deshalb genauso gemessen wird. Das
Schmerz-Häkchen bleibt setzbar; die Uhr hält dafür nicht an.

**Ein Notausgang bleibt.** Der Knopf „Erst noch einstellen" hält die Uhr an und führt auf die
gewohnte Karte, falls doch Sprosse oder Zone geändert werden sollen. Ohne ihn wäre die Karte für
die zweite Seite unerreichbar geworden — eine Einbahnstraße im Namen der Bequemlichkeit.

**Grenzen.** Der Durchlauf greift nur nach dem *letzten* Satz eines Blocks, nur wenn danach
überhaupt noch ein Block kommt, und nur bei derselben Übung mit `seitenParallel`. Der letzte Block
des Workouts endet weiter mit „Weiter"; eine andere Übung bekommt weiter die volle Blockpause.

> **Zurückgenommen mit `2026-09-21 c`.** Luke wollte die normale Pause zwischen den Seiten
> zurück; damit entfällt die Bedingung, auf der dieser Durchlauf saß. Der Eintrag bleibt stehen,
> weil die Begründung weiter trägt, falls die kurze Pause je wiederkommt.

**Geprüft.** 286 Prüfungen (16 neu), dazu im Browser durchgespielt: Pausenkarte ohne „Weiter";
nach Ablauf der Uhr ist der linke Block abgelegt und rechts läuft Satz 1 von 4; der letzte Satz
rechts zeigt wieder den normalen Weiter-Knopf; „Erst noch einstellen" landet auf „Übung 2 von 2".

---

### 2026-09-21 a — Beidbeinig ist eine Wahl, keine Sackgasse

**Befund (Luke).** „In der ISO-App wird mir heute isometrisches Wadenheben beidbeinig in Zone A
empfohlen, das kann nicht sein, da ich beidbeinig das letzte Mal mehr als 3 Minuten halten
konnte. Baue die Funktion ein, beidbeiniges Wadenheben in einbeiniges zu ändern; hier muss sich
dann auch Ablauf des Workouts und Workoutzeit verändern. Man kann allerdings isometrisches
Wadenheben links/rechts direkt hintereinander trainieren und auch messen, ohne Vorermüdung."

**Was falsch lief.** Siehe P49: Drei unabhängige Wege — Kurve, Messfenster, Ankerrechnung —
führten alle auf dieselbe beidbeinige Sprosse zurück, weil ein Block ohne Seite nur dort landen
durfte. Die Übung war damit dauerhaft auf 50 % festgenagelt, während die Karte eine Zone-A-Vorgabe
trug, die diese Sprosse nie erreicht.

**Die Änderung.**

1. **`beidJetzt(ex)` entscheidet in drei Stufen**, in dieser Reihenfolge: die Wahl von Hand für
   heute → der gemerkte Leiterstand → die Messung. Die Messung schiebt dabei **nur nach oben**
   (beidbeinig → einbeinig), nie zurück. Der alte Rumpf der Funktion heißt jetzt `beidGemerkt()`
   und ist unverändert die Leiterstand-Stufe.
2. **Der Maßstab ist gemessen, nicht modelliert.** `beidZuLang(ex, zone)` fragt: War der letzte
   beidbeinige Halt länger als die Obergrenze der heutigen Zone? Mit nur einer Last gibt es keine
   Kurve, auf die sich ein Modellargument stützen könnte — jede modellgestützte Begründung wäre
   an dieser Stelle zirkulär. Die gemessene Zeit ist zugleich Lukes eigenes Argument.
3. **Die Ausführung steht als Knopfpaar da**, auf der Karte vor der Übung und in der
   Plan-Vorschau: *Beidbeinig · ein Halt* / *Einbeinig · links und rechts*. Der Knopf, den die App
   gewählt hat, ist bereits markiert; darunter steht der Grund im Klartext („Beidbeinig hast du
   zuletzt 250 s gehalten — Zone B geht bis 90 s …"). Nochmal auf die eigene Wahl tippen heißt:
   zurück zur Automatik. Dasselbe Muster wie bei der Zonenwahl aus `2026-09-19 h`.
4. **Die Wahl gilt für heute**, mit eigener Marke `AW_V` neben `ZW_V` — eine spätere
   Bedeutungsänderung der einen soll die andere nicht mitreißen. Morgen rechnet die App wieder
   selbst, aus dem, was tatsächlich gemessen wurde.
5. **`seitenParallel` auf beiden Wadenübungen.** Eine Wade ermüdet die andere nicht. Die zweite
   Seite geht deshalb nicht mehr ans Ende des Workouts, sondern direkt hinter die erste, und
   **beide werden gemessen** statt eine gemessen und eine nur trainiert. Das verdoppelt die
   Datenpunkte je Sitzung für diese Übung — die Frage, die in Teil 4 Punkt 6 für den Seitstütz
   offen ist, ist hier physiologisch entschieden.
6. **Die Dauer rechnet die Ausnahme mit.** *(Mit `2026-09-21 c` zurückgenommen — `blockPauseS`
   gibt es nicht mehr, `planDauer` rechnet wieder durchgehend mit der Blockpause.)*
   `blockPauseS(vorher, jetzt)` lieferte zwischen zwei
   Blöcken derselben Übung mit `seitenParallel` die Satzpause (20 s) statt der Blockpause
   (3 min); `planDauer` las sie. Aus 15 Minuten wurden 12. Die beiden Hilfetexte, die bisher
   pauschal „3 Minuten zwischen zwei Blöcken" behaupteten, nennen die Ausnahme.
7. **`ausfUmbauen(i)` baut den laufenden Plan an Ort und Stelle um.** Aus einem Block werden zwei
   oder umgekehrt; was davor liegt, bleibt unangetastet, die Zählung („Übung 1 von 2") und die
   Dauer ziehen mit. Neu zu planen wäre einfacher gewesen, hätte aber den laufenden Lauf
   verworfen — samt allem, was schon gemessen ist. Angeboten wird die Umstellung nur, solange von
   dieser Übung noch kein Block gelaufen ist; sonst entwertete sie eine bereits gemessene Seite.
8. **Zwei Gegenfilter.** Ein Block *mit* Seite landet nie mehr auf der beidbeinigen Sprosse, und
   dieselbe Regel steht jetzt auch in der Variantenliste der Karte: Ein Seitenblock sieht dort nur
   die einbeinigen Sprossen, ein Block ohne Seite nur die beidbeinige — und damit gar keine Liste,
   weil es dort nichts zu wählen gibt. Die Ausführung wird eine Zeile höher umgestellt.
9. **Der Vorermüdungs-Hinweis ist bei beiden Wadenübungen weg.** Er behauptete eine
   Vorermüdung, die es dort nicht gibt.

**Eine Setzung, die Lukes Veto braucht.** `seitenParallel` steht auch auf dem Wadenheben mit
Handtuchrolle. Genannt hatte Luke nur das normale Wadenheben; es ist dieselbe Bewegung mit
derselben Physiologie, deshalb dieselbe Flagge.

**Geprüft.** 270 Zusicherungen, darunter 39 neue für Flagge, Sprossenzuordnung, Entscheidungsfolge,
Tagesmarke, Karte, Pause und Dauer. Am Gerät nachgestellt bei 375 × 812 mit Lukes Fall (244 s
beidbeinig): Der Plan zeigt zwei Blöcke links/rechts, beide „wird gemessen", 12 Minuten; das
Umschalten funktioniert in beide Richtungen in der Vorschau und mitten im laufenden Workout; die
Variantenliste bietet einem Seitenblock nur die vier einbeinigen Sprossen an.

### 2026-09-19 h — Die Zone wird vorgegeben, nicht gefragt

**Befund (Luke).** „Bei den Übungen steht jetzt ›Zonenpause‹, je nachdem ob ich Zone A, B
oder C angewählt habe. Nimm dem User diese Entscheidung bitte ab: Es wird automatisch die
Zone trainiert, die zur Verfügung steht oder die in letzter Zeit am wenigsten trainiert
wurde. Die Zonenauswahl ist vorgegeben, kann aber manuell angepasst werden."

**Was falsch lief.** Die Zonenzeile hatte vier Knöpfe: A, B, C und „Rotation entscheidet".
Vorgewählt war der vierte. Damit sahen die drei Zonen aus wie eine offene Frage — und wer
eine antippte, hatte ohne es zu merken eine **feste** Wahl getroffen. Eine feste Wahl prüft
das Tor anders: Statt „ist irgendeine Zone erholt?" gilt dann „ist genau diese erholt?".
Übungen, deren Muskelgruppe in der gewählten Zone noch pausierte, verschwanden mit dem
Vermerk „Zonenpause — noch 2 Tage" aus dem Workout, obwohl eine andere Zone frei gewesen
wäre. Die Rotation hätte das nie getan; sie nimmt unter den erholten Zonen die, die am
längsten zurückliegt.

**Die Änderung.**

1. **Der vierte Knopf ist weg.** Es stehen drei Zonen da, und die, die heute ohnehin
   trainiert würde, steht bereits ausgewählt. Die Antwort ist sichtbar, bevor gefragt wird.
2. **Ein Tipp auf die eigene Wahl nimmt sie zurück.** Das ist der Weg zurück zur Automatik,
   den vorher der vierte Knopf war.
3. **`rotationsZone(gruppe)`** ist die Regel als eine Funktion: erholte Zonen zuerst,
   darunter die älteste; ist keine erholt, entscheidet allein das Alter. `zoneFuer` und die
   Zeile lesen jetzt dieselbe Quelle — vorher stand die Rechnung nur in `zoneFuer`, und die
   Zeile wusste von ihr nichts.
4. **Ohne Gruppe zählt die Rotation über alle Übungen.** Die Zeile auf der Workoutkarte und
   auf dem Pausentag zeigt damit die Zone, die insgesamt am längsten zurückliegt. Je Übung
   kann das abweichen — die Erholung zählt je Muskelgruppe —, und die Zonenpille am Block
   sagt dann, was dort wirklich läuft.
5. **Eine gespeicherte Wahl von gestern Abend wird verworfen.** Unter der alten Oberfläche
   war ein Tipp auf „Zone C" möglicherweise gar keine Entscheidung, sondern ein Versehen.
   Beides ist im Speicher nicht zu unterscheiden, also trägt die Wahl ab jetzt eine Marke
   (`ZW_V = 2`); was sie nicht hat, fällt beim Laden weg. Nach der Wirkung, nicht nach der
   Fassungsnummer — Lukes Telefon steht schon auf Fassung 13.
6. **Eine falsche Behauptung nebenbei korrigiert.** Der Satz unter der Zeile endete immer mit
   „und ist erholt" — auch dann, wenn gar keine Zone erholt war und die Rotation nur noch die
   älteste nehmen konnte. Jetzt steht dort in diesem Fall „— erholt ist heute keine."

**Was bleibt.** Wer von Hand eine pausierende Zone wählt, bekommt weiterhin die Zonenpause.
Das ist jetzt aber eine Ansage und kein Nebeneffekt: Er hat die Zone selbst gesetzt, der Text
sagt das, und „Trotzdem trainieren" steht darunter.

**Ein Fehler, den nur der Browser zeigt.** Die Marke stand zuerst weiter unten in der Datei
als die Zeile `let S = load();`. Funktionen werden vorgezogen, `const` nicht — beim Start lag
die Marke noch in der temporalen Totzone, `normalize` warf, `load()` fing es ab und lieferte
einen leeren Speicher zurück. Die App startete wie frisch installiert, obwohl alle Daten im
Browser standen. Im Test lief die Datei vorher komplett durch, dort war nichts zu sehen. Eine
Prüfung auf die Reihenfolge im Quelltext steht jetzt in der Testdatei.

**Geprüft.** 231 Zusicherungen, darunter 20 neue für Rotation, Vorgabe, Rücknahme, Tor und
Migration. Am Gerät nachgestellt bei 375 × 812: Vorgabe steht markiert, eine pausierende Zone
von Hand erzeugt den Pausentag, derselbe Knopf noch einmal bringt das Workout zurück, eine
Übung läuft in Zone B, während die anderen in Zone C bleiben.

### 2026-09-19 g — Beidbeinig ist eine Sprosse, keine dritte Kurve

**Befund (Luke).** Im Verlauf standen für das isometrische Wadenheben drei Zeilen: links,
rechts und beidbeinig. Richtig sind zwei. Der beidbeinige Halt ist keine eigene Kraftkurve,
sondern **eine Sprosse in beiden Leitern** — Sprosse 2 von 5 heißt „50 % des Körpergewichts,
beidbeinig, ohne Waage" und ist genau der Zwischenschritt zwischen 40 % auf der Waage und
63 % auf der Waage. Wer sie hält, misst beide Waden auf einmal.

**Was falsch lief.** Eine Sitzung ohne Seite wurde unter dem Schlüssel `wade` abgelegt, eine
einseitige unter `wade|links` beziehungsweise `wade|rechts`. Drei Schlüssel, drei Kurven. Die
beidbeinige Messung floss damit in **keine** der beiden Kurven ein, die die App später zur
Vorhersage benutzt — der Einstieg ins Wadenheben erzeugte lauter Punkte, die nirgends ankamen.
Der Leiterstand hing am selben dritten Schlüssel: Nur `gemerkteStufe` las ihn überhaupt, alles
andere sah eine Leiter, auf der noch nie jemand gestanden hatte.

**Die Regel jetzt.** Die Sprosse entscheidet, nicht der Eintrag. Eine Sitzung ohne Seite auf
einer Sprosse mit `beid:true` zählt für links **und** für rechts; gespeichert wird sie
weiterhin genau einmal. Alles andere ohne Seite bleibt, was es war: Altdaten aus der Zeit vor
der Seitentrennung, die für keine Kurve zählen — jetzt auch so benannt.

**Was daraus folgt:**

1. **Zwei Kurven statt drei.** Die Auswahl in Profil und Verlauf kennt nur noch links und
   rechts. Eine Zeile „beidbeinig" gibt es nicht mehr; der Punkt steht stattdessen in beiden.
2. **Die alten Leiterstände sind umgezogen.** `S.levels["wade"]`, `S.extra["wade"]` und
   `S.streak["wade"]` wandern beim Laden auf beide Seitenschlüssel — bei der Stufe gewinnt die
   höhere, damit eine bereits erklommene Seite nicht zurückfällt. Der Umzug hängt **nicht** an
   der Fassungsnummer, sondern daran, ob die Zeile noch da ist; auf dem Telefon steht die
   Fassung längst auf 13.
3. **Die Regeln laufen je Seite.** Ein beidbeiniger Satz durchläuft die Progressionsregeln
   zweimal, einmal pro Leiter. Kommt beide Male dasselbe heraus — der Normalfall —, steht die
   Meldung einmal da und trägt den Namen „· beidbeinig". Gehen die Folgen auseinander, bekommt
   jede Seite ihre eigene Meldung; was auf beiden Seiten wortgleich herauskäme, steht trotzdem
   nur einmal da.
4. **Der Schmerz bekommt eine Seite.** Wird bei einem beidbeinigen Halt „wegen Schmerzen
   abgebrochen" angekreuzt, fragt die App: links, rechts oder beide. Vorbelegt ist „beide" —
   die vorsichtigere Antwort und das bisherige Verhalten. Nur die genannte Seite führt die
   Schmerzserie und wird zurückgestuft. Der Schmerzwert 7+ am Ende des Workouts trifft dagegen
   weiterhin beide Leitern: Es war eine Belastung für beide.
5. **Nachtragen kann sich nicht mehr widersprechen.** Bisher standen Stufe und Seite frei
   nebeneinander, und „100 % — einbeinig" ließ sich mit „beidbeinig" kombinieren. Jetzt
   entscheidet die Stufe: Auf einer beidbeinigen Sprosse zeigt das Seitenfeld gesperrt
   „beidbeinig — zählt für beide Seiten", auf jeder anderen links oder rechts.
6. **Ein Block ohne Seite steht nie auf einer einbeinigen Sprosse.** Die Vorhersage durfte
   einen seitenlosen Block bisher auf „63 % — Waage" stellen: eine einbeinige Sprosse,
   beidbeinig gehalten, ohne Seite gespeichert. Sie wählt jetzt nur unter den beidbeinigen
   Sprossen. Von der Leiter herunter kommt man über die Regeln — steht danach eine einbeinige
   Sprosse gemerkt, plant die App wieder links und rechts getrennt.
7. **Eine Seite auf der beidbeinigen Sprosse hält die ganze Übung dort.** Maßgeblich ist nicht
   mehr die höhere der beiden Stufen, sondern die niedrigere Frage: Steht *irgendeine* Seite
   auf der beidbeinigen Sprosse, wird beidbeinig geplant. Der Fall ist nicht theoretisch — wer
   von „63 % — Waage" zurückgestuft wird, landet genau dort, und mit der höheren Stufe als
   Maßstab hätte die App ihm direkt nach einer Rückstufung wegen Schmerz 50 % **einbeinig**
   vorgelegt. Also mehr Last, nicht weniger.
8. **Der Export benennt es.** Die Spalte `seite` trägt bei einem beidbeinigen Halt das Wort
   „beidbeinig" statt eines leeren Feldes — sonst wäre er von echten Altdaten ohne Seite nicht
   zu unterscheiden. Die Zeile bleibt eine; die Zahl der Kurven im Exporthinweis zählt sie für
   beide Seiten.

**Was der Punkt nicht kann.** Im beidbeinigen Stand kann die stärkere Wade unbemerkt mehr als
die Hälfte übernehmen. Der Messwert schmeichelt dann der schwächeren Seite. Genau deshalb
bleibt er als „beidbeinig" sichtbar markiert und verschwindet nicht hinter einer Seitenangabe,
die er nicht hat. Dasselbe gilt für das Schmerzhäkchen: Bricht der Halt wegen der linken Wade
ab, ist auch der rechte Wert abgeschnitten — er steht deshalb in beiden Kurven als
Schmerzpunkt, nicht nur in der linken.

**Betrifft auch die Handtuchrolle.** Sie trägt dieselbe Leiter mit derselben beidbeinigen
Sprosse und hat alles davon mitbekommen.

**Alte Sitzungen ohne Seite auf einbeinigen Sprossen** bleiben, wo sie sind: in einer eigenen
Zeile „ohne Seite (Altdaten)". Sie in eine der Kurven zu schieben, hieße Daten zu erfinden.

**Geprüft:** 209 Prüfungen, davon 26 neu für diesen Umbau — Kurvenzuordnung, Umzug der
Leiterstände, Regeln je Seite, Schmerzzuordnung, Sprossenschranke. Dazu ein vollständiger
beidbeiniger Durchlauf im Browser: ein Block, eine gespeicherte Sitzung, drei Messpunkte in
jeder der beiden Kurven, getrennte Meldungen für links (Schmerz) und rechts (unter Vorgabe).

### 2026-09-19 f — Drei Ursachen für einen Sprung

Auf Lukes Zuruf mit Bildschirmaufnahme: *„bei schnellem klicken der gewichtsanzeige springt der
Bildschirm und zoomed rein"*. Die Aufnahme zeigt es zweifelsfrei: bei 2,2 s, 5,0 s und 8,4 s ist
die Seite hineingezoomt, dazwischen wieder normal.

**Drei Ursachen, die nichts miteinander zu tun haben — und alle drei mussten weg.**

**1 · Der Sprung nach oben.** Jeder Tipp auf Plus oder Minus zeichnet die Karte neu, und das läuft
über `show("home")`. Dessen letzte Zeile ist `window.scrollTo(0,0)`. Wer also unten bei den Sätzen
steht und das Gewicht hochtippt, landet nach jedem einzelnen Tipp oben — und der zweite Tipp trifft
etwas anderes als der erste. Das Werkzeug dagegen gab es längst: `renderAmOrt()` merkt sich die
Lage einer Ankerzeile, unterdrückt den Sprung und holt die Seite um genau die Verschiebung zurück.
Es hing nur an `render2()` fest. Jetzt ist der Kern als `amOrt(anker, zeichnen)` herausgelöst, die
Karte trägt einen Anker, und **alle vier** Neuzeichnungen im Block-Vorspann gehen darüber:
Gewicht, Seitenwahl, Variante, Sätze, Zonenwahl und die Häkchen.

**2 · Der Zoom beim Hineintippen.** Safari auf dem iPhone zoomt in jedes Eingabefeld, dessen
Schrift kleiner als 16px ist. Der Fließtext der App hat 15px, und die Felder erbten ihn. Ein Pixel
Unterschied, und das Gewichtsfeld reißt beim Antippen den halben Bildschirm auf. Die Felder tragen
jetzt ihre eigene Größe — das Datumsfeld unter „Messpunkt nachtragen" gleich mit, das war vorher
gar nicht erfasst und sah auch anders aus als der Rest.

**3 · Der Doppeltipp.** Zwei schnelle Tipps auf dieselbe Stelle deutet iOS als Doppeltipp und
zoomt. Genau das passiert beim Hochtippen von 10 auf 16 kg. `touch-action:manipulation` auf dem
Body nimmt dem Browser diese eine Geste; Wischen und Aufziehen mit zwei Fingern bleiben.

**Warum das zusammen auffiel:** Einzeln ist jede der drei harmlos. Erst die Kombination — nach
oben springen, dann hineinzoomen, dann nochmal — macht das Feld unbedienbar. Nur eine der drei zu
reparieren hätte den Fehler halb stehen gelassen und wie ein Fix ausgesehen.

Keine `EX_VERSION`-Änderung: hier ändert sich nur, wie die Seite auf Berührung reagiert, keine
gespeicherte Größe. Die Erholungszeiten bleiben stehen. 159 Verhaltensprüfungen, alle bestanden.

### 2026-09-19 e — Ein neuer Haken hat eine Übung stillgelegt

Auf Lukes Zuruf: *„ich finde jetzt keine menü wo ich die variation und das gewicht einstellen
kann"*.

**Er hatte recht, und die Ursache lag in *b*.** Der Läufer hat seit der Gürtel-Umstellung
`extraGeraet:["band","gewicht"]` — zwei Haken, die BEIDE sitzen müssen. „Band oder Gürtel zum
Einhängen" ist dabei ein **neuer** Haken; vorher hing das Gewicht in der Hand und brauchte nur
„Kettlebell". Wer, wie Luke, die Kettlebell angehakt hatte, hatte das Band nie angehakt, weil es
den Haken nicht gab. `extraBereich` fällt damit auf `{lo:0, hi:0}`.

**Und dann verschwand alles auf einmal.** Die Variantenzeile erscheint erst ab zwei Sprossen — der
Läufer hat seit *b* nur noch eine. Die Seitenwahl und das Gewichtsfeld hängen an der Spanne. Alle
drei Bedingungen waren plötzlich falsch, und die Karte vor der Übung hatte **nichts** mehr zu
bedienen: kein Feld, kein Knopf, kein Hinweis, warum. Die Übung war still stillgelegt.

Zwei Sachen waren also kaputt, und beide sind repariert:

**1. Eine Anforderung in zwei Haken zu zerlegen, darf den alten nicht verfallen lassen.** Wer
„Kettlebell" angehakt hatte, hat gesagt: *bei dieser Übung kann ich Gewicht anhängen*. Der Gürtel
ist der neue Aufhängepunkt für dasselbe Gewicht, kein zweiter Wunsch. `normalize` setzt `band`
deshalb auf `true`, **wenn darüber noch nie entschieden wurde** — ein abgewähltes Band bleibt
abgewählt, denn der Haken schreibt `false`, nicht `undefined`, und ohne Kettlebell wird auch kein
Band erfunden. Bewusst nicht an `EX_VERSION` gehängt: Lukes Handy stand beim Zuruf schon auf
Fassung 13, eine versionsgebundene Migration hätte ihn nicht mehr erreicht.

**2. Eine Karte ohne Bedienelemente muss sagen, warum.** Ein Satz, wo vorher Leere war: *„Nichts
einzustellen: Gewicht am Gürtel braucht Band oder Gürtel zum Einhängen — unter ‚Mehr → Geräte'
anhaken."* Dafür `extraFehltGeraet(ex)`. Der Satz erscheint nur, wenn die Übung eine Sprosse hat,
keine Gewichtsspanne, **und** ein Gerät der Grund dafür ist.

**Was daraus zu lernen ist:** Eine Leiter aus einer Sprosse hat keine Reserve. Bei sieben Sprossen
blendet ein fehlendes Gerät eine Stufe aus und der Rest trägt weiter; bei einer Sprosse plus
Gewicht nimmt derselbe fehlende Haken die ganze Bedienung mit. Jede künftige Verschärfung von
`extraGeraet` gehört gegen bestehende Haken geprüft, nicht nur gegen die Voreinstellung.

147 Verhaltensprüfungen, alle bestanden (vorher 138). Lukes Stand im Browser nachgestellt —
Fassung 10, Kettlebell ohne Band: Nach dem Laden steht `band` auf `true`, die Spanne wieder auf
−32…+32, und die drei Knöpfe sind da. Mit ausdrücklich abgewähltem Band steht stattdessen der
eine Satz auf der Karte.

### 2026-09-19 d — Drei Knöpfe statt eines Vorzeichens

Auf Lukes Zuruf, mit zwei Handy-Bildern der Variantenliste: *„die auswahl steht immern och so da,
3 auswahl brauchen wir gewicht mit Gurt gleiche Seite, ohne gewicht, gewicxht mit gurt Gegenseite
und dazu wie schon vorhanden die gewichtsauswahl. ändere das"*.

**Zwei Befunde in einer Nachricht.** Der erste: Die Bilder zeigen *2026-09-18 i* — vier Varianten,
dazu der Satz „nicht am Gürtel". Auf dem Handy lag die alte Fassung, weil vier Commits
unveröffentlicht auf `main` standen. Gepusht; GitHub Pages liefert seither *2026-09-19 c*. Der
zweite Befund steht unabhängig davon und ist der eigentliche: Die Bedienung, die seit *b* drin ist,
taugt nicht.

**Ein vorzeichenbehaftetes Zahlenfeld ist keine Bedienung.** Seit *b* trägt das Gewicht die ganze
Leiter, von −32 bis +32 kg, und das Vorzeichen sagt die Seite: minus heißt Gürtel auf der
Standbeinseite, also Entlastung. Das ist die richtige *Rechnung* — der Hebel geht nach außen 0,31 m
und nach innen 0,13 m, das sind zwei verschiedene Faktoren, und ein einziges vorzeichenbehaftetes
Feld bildet beide sauber ab. Als *Frage an den Nutzer* ist es Unsinn. „−10 kg" liest niemand als
„die Kettlebell hängt auf der anderen Seite", und ein Erklärsatz daneben macht es nicht besser
(dieselbe Regel wie im Profil: braucht eine Zahl einen Absatz, ist die Zahl an der Stelle falsch).

**Eine Frage, in zwei beantwortbare zerlegt.** Drei Knöpfe für die Seite — *Standbeinseite
(leichter)* · *Ohne Gewicht (Körpergewicht)* · *Freie Seite (schwerer)* — und darunter ein Feld,
das nur noch den Betrag trägt. Betrag null **ist** „Ohne Gewicht", dann entfällt das Feld ganz.
Intern bleibt das Vorzeichen die Wahrheit: `S.extra` speichert weiter −10 statt „10, Seite B", die
Rechnung hängt daran, und der Verlauf schreibt unverändert „10 kg am Gürtel der Standbeinseite".
Geändert ist ausschließlich, wie gefragt wird.

Bedingung für die drei Knöpfe ist `extraSeiten && lo < 0 && hi > 0` — sie erscheinen also nur, wo
das Gewicht wirklich in beide Richtungen geht. Ohne Gürtel und Gewicht ist die Spanne null, dann
gibt es weder Knöpfe noch Feld und die Übung bleibt ohne Zusatzlast nutzbar. Beim Seitenwechsel
bleibt der Betrag stehen: Wer von *freie Seite* auf *Standbeinseite* wechselt, hängt dieselbe
Kettlebell um und tippt sie nicht neu ein. Der Ausflug auf „Ohne Gewicht" merkt sich den letzten
Betrag für die Sitzung (`letzteLast`, bewusst nicht gespeichert — eine Bequemlichkeit, kein
Zustand).

**Nicht vergessen:** `EX_VERSION` musste auf 13, weil `S.ex` gespeichert wird und die neuen
Kurznamen sonst kleben bleiben. Das setzt wie jede Übungsumstellung die Erholungszeiten auf 7/7/7
zurück. Wer von Fassung 10 kommt — Lukes Handy — durchläuft weiterhin beide Umzüge der Reihe nach
(Band, dann freies Gewicht) und behält seine Punkte lastwahrend.

**Offen geblieben:** Lukes Worte waren „gleiche Seite" und „Gegenseite". Gemeint sein kann beides,
je nachdem, worauf sich „gleich" bezieht — auf das Standbein oder auf das gehobene. Die Knöpfe
nennen deshalb die Seite direkt (*Standbeinseite* / *Freie Seite*) statt seine Kurzform zu
übernehmen; damit ist die Frage gegenstandslos statt geraten.

138 Verhaltensprüfungen, alle bestanden (vorher 112). In der laufenden App gegengeprüft: Knöpfe
erscheinen nur mit Gerät, Betrag überlebt den Seitenwechsel und den Ausflug auf „Ohne Gewicht",
Last fällt auf der Standbeinseite und steigt auf der freien, beide Seiten gehen mit, und der
Zonenwechsel lässt die eingestellten 10 kg stehen.

### 2026-09-19 c — Die Zone hängt jetzt an der Übung, nicht am Tag

Auf Lukes Zuruf: *„gib mir die möglichkeit bei allen Übungen die zonenwahl A, B, C selber
einzustellen"*.

**Was heute Vormittag gebaut wurde, war eine Wahl für den ganzen Tag.** Ein Knopfpaar oben auf der
Workoutkarte, und die gesetzte Zone galt für alle sechs Blöcke. Das ist nicht, was er will: Die
Wade darf heute in Zone A stehen, während die Fersenbrücke in Zone C hält.

**Es brauchte keine neue Ebene.** Erholung und Zonenalter zählen seit dem 18.09. **je
Muskelgruppe** (`zoneAge(gr)`, `restOf`, `zoneLastDay(gr)`), und jede der acht Übungen hat ihre
eigene Gruppe — geprüft, acht Übungen, acht Gruppen. *Je Übung* und *je Gruppe* sind hier also
dasselbe, und die Wahl konnte einfach dort andocken, wo die Rotation ohnehin schon entscheidet.
`S.zoneWahl` heißt jetzt `{tag, zone, je:{gruppe: "A"|"B"|"C"}}`; eine Einzelwahl schlägt die Wahl
für alle, eine Wahl für alle löscht die Einzelwahlen. Beides gilt nur für **heute** und wird
bewusst nicht erinnert. Eine gespeicherte Wahl aus der Vormittagsfassung (ohne `je`) gilt
unverändert für jede Gruppe weiter.

**Wo die Knöpfe stehen:** auf jeder Blockkarte des Workouts, auf der Karte unmittelbar vor der
Übung, und in *Was heute dabei ist* bei jeder Übung, die dort noch keine Zeile hat. Doppelt steht
keine Zeile. Weggelassen wird sie, wo sie heute nichts ändern kann: bei fehlendem Gerät, bei
abgehaktem Häkchen und wenn die Gruppe wegen des **Tages** pausiert — eine andere Zone hilft dann
nicht. Bei einer **Zonen**pause steht sie sehr wohl da; das ist gerade der Ausweg.

Bei der zweiten Seite einer Übung fehlt die Zeile, und das ist Absicht: Beide Seiten teilen sich
eine Muskelgruppe. Stünden sie in verschiedenen Zonen, wären nach dem Workout **zwei** Zonen dieser
Gruppe als trainiert gezählt und beide eine Woche gesperrt. Eine Umstellung während des Workouts
zieht die noch ausstehende zweite Seite deshalb mit; eine bereits gemessene bleibt unangetastet.

🔴 **Dabei ist ein Fehler in meinem eigenen Umbau von heute Vormittag herausgekommen.** Der Planer
trifft die Zielzone, indem er unter den **Sprossen** die passende sucht (`mkBlock`, Minimum von
`|log(t/ziel)|`) — das Zusatzgewicht liest er dabei nur aus dem Speicher. Der aufrechte Läufer hat
seit heute Vormittag aber **eine** Sprosse. Damit konnte der Planer dort überhaupt nichts mehr
bewegen: Auf der Karte stand eine Zone, die der Block nie angelaufen wäre — das Gewicht blieb auf
dem Stand des letzten Workouts, ob A, B oder C gefordert war. Die Zonenrotation war für diese
Übung seit heute Vormittag wirkungslos, die neue Zonenwahl wäre es geblieben.

**Behoben, indem bei einer Leiter aus einer Sprosse das Gewicht selbst die Leiter wird.** Aus der
Kurve wird die Last zurückgerechnet, die genau die Zielzeit hält (`lastZuZeit`: `t = W'/(L−CP)`
umgestellt zu `L = CP + W'/t`), aus der Last das Gewicht (`kgFuerLast`: `last = load·(1+k·kg/bw)`
umgestellt zu `kg = bw·(ziel/load − 1)/k`, mit `k = extra_k_negativ`, wenn das Verhältnis negativ
ist). Gerastert auf 2 kg, geklemmt auf −32…+32 kg und auf höchstens ein halbes Körpergewicht.
An einer gemessenen Kurve durchgerechnet läuft der Block damit die drei Zonen tatsächlich an —
je höher die Zone, desto mehr Gewicht, und die drei Gewichte liegen auseinander.
Bei Leitern mit mehr als einer Sprosse bleibt alles unverändert: dort sind die Abstände fest, das
Gewicht sitzt nur obendrauf, und nachgesteuert wird nach der Messung durch Regel 2a.

**Der Nutzer schlägt diese Rechnung.** Luke, unmittelbar danach: *„rechne die haltezeiten noch
nicht vor ich teste sie heute bzw. mit +10kg, deswegen ja auch die freie zonenwahl"*. Damit war der
Zweig oben in seiner ersten Fassung falsch herum: Er hätte bei jedem Planlauf das Gewicht aus der
Zielzone gerechnet — also auch die 10 kg überschrieben, die Luke von Hand einstellt. Die freie
Zonenwahl hätte ihm genau das Gewicht genommen, das sie freigeben sollte. Seit jetzt merkt sich die
App unter `S.extraWahl` je Übungsseite, dass das Gewicht **heute von Hand** gesetzt wurde, und
lässt es dann stehen; auf der Karte steht dazu ein Satz. Tagesgebunden wie die Zonenwahl, weil
morgen die Messung von heute in der Kurve steht. Regel 2a schreibt nach einem gemessenen Satz
weiterhin ungehindert — der Merker sperrt die Zonenrechnung, nicht die Korrektur aus Daten.
Bei zweiseitigen Übungen trägt der Seitenabgleich ihn mit, sonst rechnete der Planer der zweiten
Seite das Gewicht wieder weg.

**Zweiter Fund aus demselben Umbau:** Die Einstiegsphase spreizt die drei ersten Messungen über die
**Sprossen** (`[0,2 · 0,5 · 0,75]` der Leiterlänge). Bei einer Leiter aus einer Sprosse wären das
dreimal dieselbe Last — drei Punkte übereinander, und `fit()` gibt keine Kurve zurück, weil es
mindestens zwei verschiedene Lasten braucht. Der Einstieg beginnt jetzt bei 0 kg, danach rechnet
die Ankerschätzung aus dem jeweils jüngsten Punkt weiter. Durchgerechnet ergibt das drei
verschiedene, ansteigende Gewichte über die drei Einstiegs-Workouts (Zone C, B, A), alle aus Lukes
Kettlebells baubar — und daraus entsteht eine Kurve.

**Geprüft:** 112 Zusicherungen gegen die App-Logik, darunter die Umkehrungen (`lastZuZeit` kehrt
`predictTime` um, `kgFuerLast` kehrt `lastVon` um, beide auf drei Nachkommastellen), die
Vorrangregeln der Einzelwahl, die Rückwärtsverträglichkeit der alten Wahl ohne `je`, der Nachweis,
dass eine mehrsprossige Leiter kein Gewicht gesetzt bekommt, und der Vorrang des von Hand
gesetzten Gewichts über alle drei Zonen und beide Seiten. Dazu ein Durchlauf in der App: Zone auf
einer Blockkarte gesetzt → nur dieser Block und seine zweite Seite wechseln, die anderen vier
bleiben stehen; Zone auf der Karte vor der Übung gesetzt → der laufende Lauf bleibt an seiner
Stelle, die Satzzahl folgt der Zone; 10 kg eingestellt, dann durch alle drei Zonen geschaltet →
die 10 kg bleiben stehen, auf beiden Seiten.

**Dateien:** `prototyp/index.html`

### 2026-09-19 b — Der Gürtel trägt, er zieht nicht: eine Sprosse und ein frei wählbares Gewicht

Auf Lukes Widerspruch vom selben Tag, zweimal nachgeschärft: *„nein das band ist kein hebel
sondern lediglich dafür da wie ein kürtel das gewicht seitlich an der hüfte zu befestigen"* —
und, auf die Rückfrage, ob der Hebel damit kürzer werde: *„Nein der hebel ist sogar länger, je
nach armlänge hält die kettlebell am langen arm eher unter der hüfte (der arm tendiert zur
Körpermitte) mit dem gürtel häng das gewicht grob außen am trochanter major"*.

**Der Aufbau von heute Vormittag war falsch verstanden.** Ich hatte das Band als Zugelement
gebaut — als etwas, das an der Hüfte *zieht*. Luke meint das Gegenteil: Das Band **trägt**. Es
befestigt die Kettlebell wie ein Gürtel seitlich an der Hüfte, mehr nicht. Damit ist der Fall
mechanisch derselbe wie beim Gewicht in der Hand und nicht der, den die Vormittagsrechnung
durchgerechnet hat. Die Datei heißt jetzt `laeufer-band-VERWORFEN.mjs` und trägt den Grund im
Kopf; die Rechnung darin bleibt stehen, weil sie dokumentiert, warum ein waagerechter Zug nicht
standardisierbar wäre.

**Warum ein hängendes Gewicht überhaupt standardisierbar ist.** Sein Moment um die Standbeinhüfte
ist Masse × waagerechter Abstand — und dieser Abstand ändert sich nicht, wenn der ganze Körper zur
Seite wandert. Die Ausweichbewegung, die einen waagerechten Zug unbrauchbar macht, ist hier ohne
Wirkung. Der Gürtel ist dabei sogar **besser** als die Hand: Wo der Arm abspreizen oder sich an
den Oberschenkel legen kann — beides sieht die App nicht —, sitzt der Gürtel fest.

**Der Hebel ist gemessen, nicht geschätzt.** Luke: *„der abstand beträgt grob 22cm vom bauchnabel
zur mitte des gewichts (10kg kettlebell)"*. Vom Nabel aus, also von der Körpermitte:

| | Hebel um die Standbeinhüfte | `extraK` |
|---|---|---|
| Gewicht auf der **freien** Seite | 0,22 + 0,09 = **0,31 m** | 3,44 |
| Gewicht auf der **Standbein**seite | 0,22 − 0,09 = **0,13 m** | 1,44 |

Die 0,09 m sind die halbe Hüftbreite, dieselbe Zahl, mit der das Grundmoment gerechnet wird.
Beide Richtungen liegen damit auf **einer** Formel: `last = 0,090 · (1 + k · kg / Körpergewicht)`,
mit `k = 3,44` außen und `k = 1,44` innen.

**Die Leiter ist abgeschafft.** Auf Lukes Zuruf *„sorg direkt für freie gewichtsabstufungen …
oder direkt in 2kg abschnitten einstellbar für maximale feinjustierung"*: Der aufrechte Läufer hat
nur noch **eine** Sprosse — ohne Zusatzlast, `load 0,090` — und darüber ein frei wählbares Gewicht
von **−32 bis +32 kg in 2-kg-Schritten**. Minus heißt: am Gürtel der Standbeinseite, das entlastet.
Die Anzeige nennt die Seite im Klartext („20 kg am Gürtel der Standbeinseite") statt ein
Minuszeichen zu zeigen.

Die erreichbare Spanne wächst dadurch von ×3,62 auf **×5,60** (Last 0,038 bis 0,214). Die
Sprossenabstände, die bisher die Tore 2 und 3 prüfen mussten, gibt es nicht mehr — der Abstand
beträgt jetzt überall 2 kg, das sind je nach Stelle ×1,02 bis ×1,10.

**Was Luke damit wirklich bauen kann.** Aus 6 · 8 · 10 · 16 · 24 · 32 kg lassen sich kombinieren:
0, 6, 8, 10, 14, 16, 18, 22, 24, 26, 30, 32 kg. **Nicht** baubar sind 2, 4, 12, 20 und 28 kg. Die
App lässt sie trotzdem einstellen — sie kennt den Gerätebestand nicht, und eine zweite 2-kg-Hantel
oder eine Kurzhantelscheibe schließt die Lücken sofort.

**Der Umzug erhält die Last, nicht die Kilos.** Gespeicherte Stufen der Fassung 11 werden in die
Last umgerechnet, die sie hatten, und daraus das Gewicht bestimmt, das dieselbe Last ergibt. Die
Rasterung auf 2 kg kostet dabei unter 2 % — an keiner alten Sprosse entsteht ein Sprung, der nicht
trainiert wäre. Lukes Stand (Stufe 5 von 5, kein Zusatzgewicht) landet auf **Stufe 1 + 30 kg**,
Last 0,206 wie vorher. Gemessene Punkte sind ohnehin nicht betroffen, sie tragen ihre Last als Zahl.

**Die Zugwaage ist wieder weg.** Sie war nur nötig, solange das Band Kraft erzeugen sollte. Ein
Gürtel trägt ein bekanntes Gewicht — gezogen wird daran nicht. In der Geräteliste heißt der Eintrag
jetzt „Band oder Gürtel zum Einhängen".

🔴 **Offen und nicht nebensächlich:** Der Hebel der **Hand** (0,29 m) war nie gemessen, sondern
angenommen — und Luke sagt selbst, der Arm tendiere zur Körpermitte. Läge der echte Wert bei etwa
0,26 m, wären alle bisher gespeicherten Handstufen rund 11 % zu hoch gerechnet, und seine
bestehende Kurve bekäme genau dort eine Stufe. Eine Kontrollmessung (Nabel → Mitte der Kettlebell
bei hängendem Arm) klärt das in einer Minute.

🔴 **Nebenwirkung des Versionssprungs, unverändert:** `EX_VERSION` steht auf 12, und die bestehende
Mechanik setzt dabei wieder die **Erholungszeiten** unter *Mehr → Erholung* auf 7/7/7 zurück. Das
ist dieselbe offene Stelle wie bei der Fassung 11 — eine geänderte Übungsleiter hat mit Pausentagen
nichts zu tun.

**Im Export:** neue Spalte `zusatz_seite` (`innen` · `aussen` · leer) direkt hinter `zusatz_kg` —
ohne sie ist ein negatives Gewicht von außen nicht deutbar, weil dieselbe Zahl außen belastet und
innen entlastet, mit verschiedenen Hebeln. In der Übungsbeschreibung stehen zusätzlich
`extra_k_negativ` und die Grenzen des Gewichtsfelds.

**Dateien:** `prototyp/index.html` · `scratchpad/laeufer-guertel.mjs` (neu) ·
`scratchpad/laeufer-band-VERWORFEN.mjs`

### 2026-09-19 a — Band statt Kettlebell, Zone von Hand, und kurze Sätze verbrauchen nichts mehr

> **Punkt 1 dieses Eintrags ist am selben Tag überholt worden** — siehe *2026-09-19 b*.
> Das Band zieht nicht, es trägt; die drei Bandstufen und die Zugwaage gibt es nicht mehr.
> **Punkt 2 ist am selben Tag erweitert worden** — siehe *2026-09-19 c*: Die Zone wird nicht mehr
> für den ganzen Tag gesetzt, sondern je Übung. Punkt 3 gilt unverändert.

Auf Lukes Zuruf vom 19.09.2026, drei Punkte in einem: *„ich würde empfehlen, das Gewicht hier
nicht … in die Hand zu nehmen, sondern mit einem Resistance Band seitlich an der Hüfte zu
befestigen … gib mir entweder die Möglichkeit, die Zone selbstständig zu wählen … und sorgt
dafür, dass kurze Haltezeiten unter 20 Sekunden nicht gezählt werden."*

**1 · Der aufrechte Läufer hängt jetzt am Band.** Die drei Handstufen (16 kg, 32 kg) sind weg.
An ihrer Stelle stehen drei Bandstufen: **12 · 26 · 48 kg Zug am Gürtel der freien Hüfte**, das
Band unter dem Standfuß durch. Damit fällt die Griffkraft als Mitentscheider raus — Lukes Grund.

Dabei ist eine Sache herausgekommen, die gegen die wörtliche Fassung des Vorschlags spricht und
deshalb hier steht (`scratchpad/laeufer-band-VERWORFEN.mjs`): **Ein waagerechter Zug auf Hüfthöhe erzeugt um
die Standbeinhüfte fast kein Moment.** Sein Hebel ist nicht der seitliche Abstand, sondern der
senkrechte Abstand vom Hüftgelenk — also der Gürtelsitz, rund 5 cm gegen die 29 cm der Hand. Was
darüber hinaus passiert, hängt daran, **wie** der Körper den Zug ausgleicht: Weicht der Rumpf aus
und der Standfuß bleibt, kommt ein Hebel von 0,92 m dazu; rückt stattdessen der Standfuß zum
Anker und der Rumpf bleibt, kommt null dazu. Beides erfüllt das Gleichgewicht, beides ist von
außen kaum zu unterscheiden, und die App sieht keines von beiden. Ein waagerechtes Band
standardisiert die Übung also nicht — es macht die Last von Fußstellung und Rumpfneigung abhängig.

Ein **senkrechter** Zug hat das Problem nicht: Er verhält sich wie ein hängendes Gewicht, fester
Hebel, unabhängig davon, wohin der Körper als Ganzes ausweicht. Deshalb läuft das Band unter dem
Standfuß durch. Wirksamer Hebel 0,194 m (Gürtelpunkt 0,21 m seitlich, 0,92 m hoch, 12,9° Schräge),
`extraK` von 3,22 auf **2,15**.

**Was dabei nicht geht:** Die unterste Sprosse — 24 kg Gegengewicht auf der **Standbein**seite —
lässt sich nicht an den Gürtel verlegen. Sie braucht einen Hebel auf der falschen Seite des
Hüftgelenks, und den liefert nur der ausgestreckte Arm (−0,11 m). Am Gürtel wären es −0,03 m,
und dieselbe Entlastung bräuchte dort **88 kg**. Diese eine Stufe bleibt in der Hand. Wer sie
nicht braucht, braucht die Kettlebell nicht mehr.

**Was dabei neu gebraucht wird:** eine **Zugwaage** (Kofferwaage, rund 10 €). Die Kraft eines
Bandes hängt an der Dehnung — ohne einmal abgelesene Zahl ist der Zug kein Messwert, sondern ein
Gefühl, und dann trägt die ganze Lastachse nichts mehr. Das steht als eigenes Gerät in der Liste.
🔴 Offen: ob Luke das in der Praxis mitmacht oder ob eine feste Bandlänge mit Markierung reicht.

Die Leiter hat damit fünf statt vier Sprossen — 0,057 · 0,090 · 0,119 · 0,153 · 0,206. Alle drei
Tore bestanden (Spanne ×3,62, Abstände ×1,58 / ×1,32 / ×1,28 / ×1,35). Die oberste Sprosse trifft
die alte oberste auf drei Stellen genau. Gespeicherte Stufen ziehen automatisch um (alt 2 → neu 3,
alt 3 → neu 4); die gemessenen Punkte sind nicht betroffen, sie tragen ihre Last als Zahl.

**2 · Die Zone lässt sich von Hand setzen.** Drei Knöpfe auf der Workoutkarte und auf dem
Pausentag: **Zone A · B · C · Rotation**. Die Wahl gilt nur für den Tag. Ist sie gesetzt, prüft
das Tor auch nur diese eine Zone — sonst wäre es keine Wahl, sondern eine Anzeige. Und die
Lückenlogik in `mkBlock` darf eine gesetzte Zone nicht mehr wegdrehen.

**3 · Kurze Sätze verbrauchen die Zone nicht mehr.** Der Teil, den Luke schon eingebaut glaubte,
war es zur Hälfte: `MESS_MIN = 20` gab es seit jeher, und `points()`, der Export und Regel 2a
halten sich daran — ein Halt unter 20 s geht **nicht** in die Kurve. Verbraucht hat er trotzdem:
`blockSichern()` speichert jeden Block mit mindestens einem Satz, `zoneLastDay()` las ihn, und
damit war die Zone eine Woche zu. Genau das war Lukes Sperre. Jetzt zählt eine Einheit ohne einen
einzigen messbaren Satz weder für die Zone noch für den Trainingstag noch für die Einstiegsphase.
Sie bleibt in Historie und Export stehen — sie ist passiert, sie hat nur nichts gemessen.

**Eine Ausnahme:** Ein **Schmerz**abbruch zählt immer, auch nach acht Sekunden. Wer wegen Schmerz
aufhört, hat kein Nicht-Ereignis hinter sich, und ein zweiter Anlauf am selben Tag ist das Letzte,
was die App dann freigeben sollte.

**Nebenwirkung des Versionssprungs:** `EX_VERSION` steht auf 11. Die bestehende Mechanik setzt
dabei auch die **Erholungszeiten** unter *Mehr → Erholung* auf 7/7/7 zurück. Das ist nicht neu und
nicht von dieser Änderung gewollt — wer dort eigene Werte stehen hatte, trägt sie einmal nach.
🔴 Offen, ob das so bleiben soll: Eine geänderte Übungsleiter hat mit Pausentagen nichts zu tun.

**Dateien:** `prototyp/index.html` · `scratchpad/laeufer-band-VERWORFEN.mjs`

### 2026-09-18 i — Der Hinweis kommt sofort, und der Sprung wird gerechnet

Auf Lukes Zuruf: *„die Ansage muss schon nach einem unbrauchbaren Durchgang kommen … nichts ist
frustrierender als beim Onboarding ‚durchzufallen' und dann direkt noch zweimal mit der gleichen
Stufe und ähnlichem Ergebnis konfrontiert zu werden."*

**1 · Sofort statt nach drei Versuchen.** Der Hinweis auf die fehlende Zwischensprosse kam bisher
erst nach dem dritten unbrauchbaren Durchgang. Jetzt kommt er beim ersten. Und er rät nicht mehr
(„meistens ist das ein Gerät"), sondern **sieht nach**: Eine neue Funktion zählt die Sprossen
zwischen alter und neuer Stufe, die an einem nicht angehakten Gerät hängen, und benennt es. Beim
zweiten Mal in Folge *ohne* fehlendes Gerät sagt die App stattdessen klar, dass der Sprung nicht
gereicht hat und die Übung hier nichts für die Kurve hergibt.

**2 · Die Sprungweite kommt aus der Rechnung, nicht aus einer Faustzahl.** Wie weit danebengelegen
wurde, lässt sich beziffern — mit genau der Formel, mit der die App den Einstieg ohne Kurve rechnet:

```
Ziellast / Istlast = ANNAHME_CP + (1 − ANNAHME_CP) · gehalten / Zielzeit
```

| Halt | gegen Zone C (120 s) | gegen Zone B (75 s) | gegen Zone A (35 s) |
|---|---|---|---|
| 300 s | ×1,75 | ×2,50 | ×4,79 |
| 20 s | ÷1,71 | ÷1,58 | ÷1,27 |

Tor 2 verlangt von einer Leiter nur **1,25 je Sprosse**. Eine einzige Sprosse war damit
rechnerisch zu wenig — man wäre mit hoher Wahrscheinlichkeit wieder außerhalb des Fensters
herausgekommen, und genau das soll niemand zweimal erleben. Es wird jetzt so weit gestuft, bis die
Ziellast erreicht ist, höchstens aber `MAX_SPRUNG` Schritte weit.

Die Richtung ist in beiden Fällen die sichere: Über 300 s ist die Zeit nur eine **untere** Schranke,
der Sprung fällt also eher zu klein aus als zu groß. Unter 20 s ist die Zeit dagegen echt — dort ist
nur das Modell im Kurzzeitbereich unsicher, und Erleichtern ist die harmlose Richtung.

**3 · Am Leiterende wird das Zusatzgewicht ausgerechnet, nicht ertastet.** `last = load · (1 +
extraK · kg / bw)` lässt sich nach `kg` auflösen. Sich in 2,5-kg-Schritten heranzutasten wäre dort
nicht vorsichtig, sondern nur langsam: Wer über dem Fenster hält, bräuchte ein Dutzend Durchgänge
bis zur passenden Last — und jeder einzelne davon wäre wieder unbrauchbar.

*Dabei ist ein neues Problem entstanden und gleich mit behoben worden:* Ungedeckelt liefert die
Formel Zahlen wie **348 kg** (300 s einbeinig gegen Zone A). Die Rechnung ist nicht falsch, sie sagt
nur, dass die Übung dort nicht mehr erreichbar ist. Die App deckelt jetzt bei einem halben
Körpergewicht — Weste plus Rucksack, das Ende des Tragbaren — und sagt bei Überschreitung die
gerechnete Zahl **und** den Befund: in dieser Zone nicht erreichbar, andere Übung wählen.

**4 · Textkorrektur.** `fmt()` rundet auf ganze Sekunden, wodurch bei 300,1 s der Satz „300 s — mehr
als 300 Sekunden" dastand. Jetzt mit Nachkommastelle, und die Grenze wird als Fenster benannt statt
als Vergleich.

Geprüft: alle acht Übungen in beide Richtungen, die Fenstergrenzen (300,0 zählt, 300,1 nicht; 20,0
zählt, 19,9 nicht) und Lukes echte Werte (244,2 s zählt weiter, 95 s unverändert).

### 2026-09-18 h — Messfenster 20–300 s, Standbeinregel, Startabsturz behoben

Auf Lukes Zuruf: *„mach es so, dass Werte über 300 Sekunden und unter 20 Sekunden nicht gezählt
werden, die Übungen sind dann zu leicht oder zu schwer. Beim einbeinigen isometrischen Wadenheben
muss das Zusatzgewicht zwingend auf der Standbeinseite gehalten werden, da sonst der Gluteus der
limitierende Faktor wird anstatt die Wade."*

**1 · Das Messfenster** (P48). Ein erster Satz unter 20 oder über 300 Sekunden zählt nicht mehr für
die Kurve — gefiltert an allen drei Stellen, die Messwerte lesen: `points()` für den Fit,
`ankerpunkt()` für die erste Stufenwahl ohne Kurve, und die Exportspalte `in_kurve`.

Verworfen heißt aber nicht folgenlos. Eine neue **Regel 2a** zieht die Konsequenz, die der Wert
tatsächlich trägt: über 300 s eine Stufe hoch, unter 20 s eine zurück. Ohne das bekäme man beim
nächsten Mal dieselbe Stufe und dieselbe unbrauchbare Messung. Nach **drei** unbrauchbaren
Durchgängen in Folge sagt die App, dass dieser Leiter ein Zwischenschritt fehlt — meistens ein
Gerät, das unter *Mehr → Geräte* noch nicht angehakt ist.

Dazu zwei Stellen, an denen es sichtbar wird: Die Uhr warnt **ab 300 Sekunden** im Satz selbst
(vorher stand dort „jede Sekunde mehr zieht die Kurve hoch" — ab hier stimmt das nicht mehr, und
wer es nicht erfährt, steht umsonst weiter). Und die Verlaufstabelle markiert solche Werte mit
einem Kreuz samt Fußnote.

*Beim Prüfen gefunden und mit behoben:* Regel 4 (»die Kurve trägt auf dieser Stufe über 150
Sekunden«) lief bisher **nach** Regel 2a noch einmal durch. Nach einem 14-Sekunden-Halt sah man
dadurch erst „eine Stufe zurück" und unmittelbar danach „eine Stufe höher" — zwei Meldungen, die
sich aufheben, und am Ende stand man wieder auf der Stufe, die man gerade nicht halten konnte.
Regel 4 rechnet auf der Kurve, und die enthält diesen Wert ohnehin nicht; sie greift beim nächsten
Durchgang unverändert.

**2 · Die Standbeinregel** an beiden Wadenhebern. Zusatzgewicht in der Hand gehört zwingend auf die
**Standbeinseite**. Hängt es auf der freien Seite, zieht es das Becken dorthin, die Hüfte muss
dagegenhalten — und dann begrenzt der Gluteus den Halt, nicht die Wade. Die Zeit misst dann das
falsche Gelenk. Rucksack und Weste sitzen mittig, erzeugen dieses Moment gar nicht erst und sind
hier die ruhigere Wahl, weil auch kein Griff vorzeitig aufgibt; die Rechnung ist in beiden Fällen
exakt. `EX_VERSION` von 9 auf 10, damit der Text die bestehende Installation erreicht —
`LEVELS_RESET_AT` bleibt bei 7, Stufen und Messreihen bleiben erhalten.

**3 · Der Startabsturz** (P47), gefunden beim Prüfen von Punkt 1 und ein Fehler der ausgelieferten
Fassung, nicht der neuen: `geraetDa` ist jetzt eine Funktionsdeklaration statt einer
`const`-Pfeilfunktion und wird damit nach oben gezogen.

### 2026-09-18 g — Eine Übung abbrechen, ohne das Workout zu beenden

Auf Lukes Zuruf: *„gib mir die Möglichkeit, während ich Übungen mache, z. B. nach dem ersten oder
zweiten Satz die Übung abzubrechen, ohne das ganze Workout zu beenden."* (P45)

- **Vor dem ersten Satz:** „Diese Übung heute auslassen" auf der Ankündigungskarte des Blocks.
- **Zwischen zwei Sätzen:** „Übung hier beenden und zur nächsten" auf der Pausenkarte.
- Beide Knöpfe **fehlen dort, wo sie bedeutungslos wären** — im letzten Block und nach dem letzten
  Satz eines Blocks, wo „Weiter" ohnehin weiterschaltet.

**Warum das nichts kostet, und warum es auf dem Knopf steht:** Für Kurve und Stufenregel zählt
ausschließlich der **erste** Satz jedes Blocks (`applyRules` und `ankerpunkt` lesen `sets[0]`).
Sätze 2 und 3 sind Trainingsumfang. Ein Abbruch nach Satz 1 oder 2 verliert also keine Messung,
sondern Volumen — und der Hinweistext unter dem Knopf sagt genau das, statt den Nutzer raten zu
lassen.

**Die eigentliche Arbeit steckt in `seiteNachruecken()`.** Wird die erste Seite einer zweiseitigen
Übung **ohne einen einzigen Satz** übersprungen, ist die zweite Seite nicht mehr vorermüdet. Sie
trägt aber noch den Vermerk `fresh:false`, und der schließt sie von der Stufenregel aus. Ohne diese
Reparatur hätte der neue Knopf eine stille Datenlücke eingeführt. Jetzt rückt die zweite Seite auf
„frisch" nach und wird normal gewertet.

### 2026-09-18 f — Fix: `warnKarte()` war verlorengegangen

**Vier Übungen ließen sich zwei Fassungen lang nicht starten** (P44): Fersenbrücke, aufrechter
Läufer und beide Wadenheben — alle Übungen mit einem noch nicht beantworteten Warnhinweis. Der
Knopf „Workout starten" reagierte einfach nicht.

- `warnKarte()` und `bloeckeOhne()` sind unverändert aus der letzten Fassung zurückgeholt worden,
  die sie noch hatte.
- Neu dazu: ein **rotes Fehlerband** am unteren Rand, das bei jeder unbehandelten Ausnahme
  erscheint (`window.onerror` und `unhandledrejection`). Es steht als Erstes im Skript, hängt von
  nichts ab und verschwindet auf Antippen. Ohne es ist eine Ausnahme im Klick-Handler von
  „der Knopf geht nicht" nicht zu unterscheiden.
- Als stehende Prüfung nach jedem größeren Umbau notiert: die Funktionsnamen einer älteren Fassung
  gegen die aktuelle stellen und für jeden verschwundenen Namen nachsehen, ob er noch aufgerufen
  wird. `node --check` kann das nicht sehen.

### 2026-09-18 e — Wadenheben: getrennte Tage, beidbeiniger Einstieg, zwei Gelenkwinkel

Drei Befunde von Luke aus dem Training, alle drei in einer Fassung.

- **Die beiden Wadenheben gehören auf verschiedene Tage** (P42) — die Muskulatur überlappt zu
  stark, sonst misst die zweite die Ermüdung der ersten. Umgesetzt über ein neues Feld `nichtMit`
  je Übung, das unabhängig von der Muskelgruppe greift.
- **Der Einstieg ist beidbeinig** (P43). In Zone C (90–150 s) ist eine einbeinige Übung kein
  realistischer Anfang. Die 50-%-Sprosse ist jetzt als `beid:true` gekennzeichnet: sie kostet
  **einen** Block statt zwei, heißt in der Anzeige „beidbeinig" statt „links", und der Planer
  fragt die Blockzahl aus der aktuellen Stufe ab, statt sie aus der Übung abzuleiten.
- **Die beiden Übungen halten in verschiedenen Gelenkwinkeln.** Das Wadenheben ohne Handtuch wird
  in der **Mittelstellung** gehalten — die Fußsohle waagerecht, Ferse und Vorfuß auf einer Höhe,
  nicht absinken und nicht hochdrücken. Das Wadenheben mit Handtuchrolle wird **ganz oben**
  gehalten, das Sprunggelenk nah an seiner Grenze. Wie die Fersenhöhe ist das eine
  **Ausführungsvorschrift, keine Sprosse**: es steht in den Erklärtexten, nicht in der Leiter.

**Der Preis steht als Kommentar im Code:** Die beiden Übungen unterscheiden sich damit in **zwei**
Dingen gleichzeitig — Handtuchrolle *und* Gelenkwinkel. Kommen verschiedene Kurven heraus, lässt
sich nicht mehr sagen, welcher der beiden Unterschiede sie getrennt hat. Die ursprüngliche Frage
(„sind das eine Übung oder zwei?") ist damit keine saubere Messung mehr, sondern eine bewusste
Setzung. Lukes Entscheidung, im Code datiert.

**Migration:** `EX_VERSION` von 8 auf 9. Bestehende gespeicherte Übungen werden durch die neuen
Vorgaben ersetzt, Stufen und Messreihen bleiben (`LEVELS_RESET_AT` unverändert bei 7).

**Was sich an der Workoutlänge dadurch nicht geändert hat:** Der Fußtag ist von vier Blöcken
(rund 36 min) auf einen (rund 7 min) gefallen. Der Rotationstag steht unverändert bei sechs
Blöcken und rund 55 Minuten, weil in ihm gar keine Wadenübung vorkommt. Die Stellschrauben dafür
sind der Blockdeckel und die Blockpause — siehe P46.

### 2026-09-18 d — Die geschätzte Dauer steht auf dem Startbildschirm

Auf Lukes Zuruf, direkt nach Fassung `c`: *„oder besser zeig mir die ungefähre Workoutlänge direkt
am Startbildschirm."* Die Zahl steht jetzt beim Workout selbst („Sechs Blöcke · rund 55 min"),
nicht nur in der Erklärung unter „Mehr". Sie wird aus dem tatsächlich geplanten Workout gerechnet,
nicht aus einem Durchschnitt.

### 2026-09-18 c — Zeitrahmen einer Sitzung unter „Länge einer Sitzung"

Auf Lukes Zuruf: *„gib mir unter ‚Mehr' bei Länge einer Sitzung eine ungefähre Zeitangabe, wie
lange das Workout dauert bei 3 Minuten Pause zwischen zwei Übungen, nur als grober Estimate."*

Die Rechnung ist aufgeschlüsselt ausgewiesen, weil ein Bestandteil davon **geraten** ist (P46):
Haltezeit und die 20-Sekunden-Pausen innerhalb eines Blocks kennt die App genau, die Pause
zwischen zwei Blöcken misst sie **nicht** — dort stehen 180 Sekunden als Annahme.

### 2026-09-18 b — Wadenheben ohne Waage: beidbeinig und einbeinig freigeschaltet

Ausgelöst durch Lukes Meldung, dass beim Wadenheben mit Handtuchrolle das Gerät „Personenwaage"
als fehlend angezeigt wird — und durch seinen Hinweis, dass er dafür eine **analoge** Waage
braucht, weil Digitalwaagen sich unter Dauerlast abschalten (P40).

- Die Kennung „Waage" ist von den beiden Sprossen entfernt worden, die keine brauchen: 50 %
  (beidbeinig) und 100 % (einbeinig). Beide messen den eigenen Körper, nicht die Anzeige.
- Damit sind beide Übungen ohne jedes Gerät benutzbar. Vorher war die **ganze Übung** unsichtbar,
  weil keine einzige Sprosse ohne Gerät lief (P41).

**Migration:** `EX_VERSION` von 7 auf 8, damit die korrigierten Sprossen auch bei bestehenden
Installationen ankommen.

### 2026-09-18 a — Der Bildschirm springt nicht mehr zurück, und Erklärungen klappen auf

Zwei Meldungen von Luke am selben Vormittag, beide dieselbe Ursache (P39): `render()` zeichnet
fest verdrahtet den Startbildschirm.

- **Ein Gerät im Mehr-Bildschirm abhaken** warf zurück auf die Startseite. Jetzt wird der aktive
  Bildschirm neu gezeichnet, und die Scrollposition bleibt stehen.
- **Die Erklärung der Blöcke ist ein Aufklappmenü geworden** — auf Lukes Vorschlag: *„das spart
  Platz und man liest es nicht immer wieder."*
- Dazu ein Kommentar an der **BUILD-Kennung**, dass sie bei jedem Push hochzuzählen ist. Anlass:
  zwei Änderungen dieses Tages sind mit dem Stempel des Vortags ausgeliefert worden, siehe den
  Nachtrag unten.

### 2026-09-17 c (nachgetragen, ausgeliefert am 18.09.) — Drei neue Übungen

Diese beiden Änderungen sind am 18.09. gepusht worden, tragen aber noch den **Stempel des
Vortags**, weil die BUILD-Kennung nicht hochgezählt wurde. Sie stehen deshalb hier und nicht
weiter oben. Genau dieser Fehler hat zu dem Kommentar in Fassung `18-a` geführt.

- **Drei neue Übungen** in der Übungsliste: **Aufrechter Läufer** (Hüfte, Abspreizer),
  **Isometrisches Wadenheben** (Wade) und **Isometrisches Wadenheben mit Handtuchrolle**
  (Fußsohle). Dazu zwei neue Gerätekennungen, `gewicht` und `waage`. Alle drei sind Lukes eigene
  Übungsentwürfe und stehen als Versuch im Prototyp, nicht als gesichertes Programm.
- **Die Auswahlliste springt nicht mehr nach oben**, wenn man eine Übung anhakt (P39, erste
  Fundstelle).

**Zur Einordnung der drei Übungen:** Beim aufrechten Läufer hat die Nachrechnung ergeben, dass die
drei geplanten **Kniehöhen keine Stufen sind** — das Abspreizmoment ändert sich um weniger als
10 %, die höchste Kniestellung ist sogar die leichteste. Die Lastachse ist deshalb das Gewicht in
der Hand (Faktor 3,22 je Kilo), die Kniehöhe ist Ausführungsvorschrift. Beim Wadenheben trägt die
Leiter nur, weil eine Waage unter dem Vorfuß die Teilentlastung messbar macht; gerätefrei gibt es
genau zwei Haltepunkte im Verhältnis 2,00, und die passen nicht gemeinsam ins Haltezeitfenster.

### 2026-09-17 c — Die Zahlen raus aus dem Profil

Zwei Zurücknahmen von Fassung `b`, beide auf Lukes Zuruf:

- **Die Warnkarte zur Streuung** — *„Zu viel Text und unverständlich für den Endverbraucher."*
- **Die ganze Kennzahlen-Karte** — *„alle erstmal nichtssagend und unverständlich."* Damit sind
  CP, W′, die Vier-Minuten-Last und die Streuung aus der Oberfläche verschwunden.

**Das Modell rechnet unverändert weiter.** Die Zahlen stehen nach wie vor im Export und treiben
Zielzeiten und Stufenvorschläge — es ist eine reine Anzeigeentscheidung, und sie ist richtig
getroffen: Diese Zahlen sind Werkzeuge des Modells, keine Trainingsrückmeldung.

Das Profil zeigt jetzt die Zeichnung mit Stufenlegende, den Zonenstand und die Tabelle, was auf
jeder Stufe herauskommt. Mitgefallen sind `predictLoad`, `streuung()` und `STREU_MIN` — ohne die
Karte hätte sie nichts mehr aufgerufen.

**Was das Profil dadurch nicht mehr meldet:** dass eine Kurve ihre eigenen Messpunkte verfehlt. Die
Streuung war der einzige Hinweis darauf. Offener Punkt 20 in Teil 4.

### 2026-09-17 b — Das Profil sagt, was es weiß, und was nicht

Ausgelöst durch ein Bildschirmfoto von Lukes eigenem Profil.

- **CP zeigt einen Strich statt einer erfundenen Null** (P35). Klebt das Optimum am Rand des
  Suchbereichs, ist die Zahl der Rand und keine Messung. Die App sagt jetzt, was fehlen würde:
  eine Messung auf einer deutlich leichteren Stufe. Die CP-Asymptote verschwindet dann auch aus
  der Zeichnung — sie hatte die Lastachse bis fast auf null gestreckt.
- **Die Streuung wird nicht mehr bei 35 % gedeckelt** und ist zweiseitig angegeben (P36). Aus
  „±35 %" wurde „+51 / −34 %". Ab 25 % erklärt eine Karte die zwei möglichen Ursachen.
- **Dauerlast-Schwelle und Last für 4 Minuten werden auseinandergehalten.** Luke fragte nach dem
  Unterschied, und die App sagte ihn nirgends: Die Vier-Minuten-Last ist ein Punkt *auf* der Kurve
  und liegt nah an den Messungen; die Dauerlast-Schwelle ist das *Ende* der Kurve und wird weit
  darüber hinaus verlängert.
- **Die Startseite hängt an der Übung statt am Workout** (P37). Es startet die Seite mit den
  wenigsten frischen Messpunkten — ein Rückstand gleicht sich damit von selbst aus.
- **Legende unter der Zeichnung** mit Stufennummer und Name; die Tabelle darunter trägt dieselben
  Nummern (P38).
- **Zwei Erklärtexte raus:** der Maximalkraft-Kasten und „Der Ausfallschritt hat zwei Kurven"
  (P3, P38).
- **Der Verlauf sagt jetzt „3 Messungen · 2 davon in der Kurve"** statt „3 Satzblocks". Damit steht
  die Antwort auf Lukes Frage in der App, statt sich aus zwei Zahlen an zwei Orten zu ergeben.
- **`points()` stürzt nicht mehr über eine Sitzung ohne `sets`-Feld.** Der ungeschützte Zugriff
  hätte bei Import- oder Fremddaten das **ganze** Profil leer gelassen.

**220 Unit-Tests grün** (vorher 199). Beim Regenerieren der Testscheiben fiel auf, dass die
extrahierte Kopie im Scratchpad veraltet war — die 199 vom Vortag liefen gegen den Stand davor.

### 2026-09-17 a — Zwei neue Übungen, und was dafür vorher repariert werden musste

**Reparaturen am Bestand — sie gingen vor den neuen Übungen, sonst hätte man auf dem Scheinfit
aufgebaut.**

- **Kein Fit mehr aus einer einzigen Last** (P30). Der Scheinfit mit CP ≈ 0 und der *besseren*
  Gütezahl kann nicht mehr entstehen.
- **Die Kurve merkt sich ihren Geltungsbereich.** Vorschläge, Zielzeiten und die Stufentabelle
  halten sich daran; außerhalb des gemessenen Lastbereichs zeichnet das Profil **gestrichelt** statt
  durchgezogen und schreibt „außerhalb" statt einer Zahl. Keine Extrapolation mehr über Lasten, bei
  denen nie gemessen wurde.
- **Zonen werden nur noch gefordert, wo die Leiter sie treffen kann** (P31). Damit verschwindet die
  Karte *Bis zum vollständigen Profil*, sobald das Erreichbare belegt ist.
- **Progression an beiden Leiterenden** (P32): oben über Zusatzgewicht, unten erst Gewicht abbauen.
  Bei Schmerz ohne leichtere Stufe sagt die App das ausdrücklich.

**Neu:**

- **Schrägzug** am Schlingentrainer (Gruppe *Zug*, einseitig nein). Sechs Sprossen über Fußabstand
  und Gurtlänge, Lastspanne 4,65×. Das Lastgesetz ist dasselbe wie beim Seitstütz — siehe P20.
- **Fersenbrücke** (Gruppe *Hüftstreckung*, zweiseitig). Sieben Sprossen über den Fersenabstand,
  Lastspanne 3,24×. Die unteren vier sind von einem zweiten, unabhängigen Segmentmodell bestätigt;
  **Faszienrolle und Ringe sind geschätzt** und in der App als solche gekennzeichnet.
- **Geräte** unter *Mehr → Setup*: Zugpunkt (voreingestellt an), Faszienrolle, Ringe. Sprossen, die
  ein fehlendes Gerät brauchen, erscheinen nicht in der Auswahl — die Nummerierung bleibt aber
  stehen (P33).
- **Kontrollposition.** Eine Leiter, deren verfügbare Sprossen weniger als Faktor 1,25 auseinander
  liegen, bekommt **keine Kurve**, sondern einen Verlauf der Haltezeit über die Wochen. Die Klasse
  wird aus der Lastspanne abgeleitet, nicht deklariert — sie ändert sich also mit, wenn ein Gerät
  dazukommt oder wegfällt. Damit ist der Fall abgedeckt, den Luke als „schlecht skalierbare
  Position" beschrieben hat; eine benannte Instanz dafür gibt es noch nicht (siehe Teil 4).
- **Rotation statt acht Blöcken** (P34): Deckel bei sechs, Reihenfolge nach Wartezeit der Kurve.
  Dazu die Karte *Heute selbst zusammenstellen* — dieselbe Liste wie der dauerhafte Filter, aber die
  Häkchen gelten nur für diese eine Sitzung und setzen den Deckel außer Kraft. Zurückgestellte
  Übungen stehen mit Begründung auf der Startseite.
- **Sicherheitsfrage für eine neue Region.** Der Red-Flag-Fragebogen lief nur im Onboarding; eine
  später hinzugekommene Übung löste ihn nicht aus. Vor der ersten Fersenbrücke wird die Frage zur
  proximalen Hamstring-Sehne nachgeholt — Sitzschmerz und ausstrahlende Beschwerden sind dort der
  klassische Verwechslungsfall mit etwas, das ärztlich abgeklärt gehört.
- **`extra_k` im JSON-Export.** Ohne diesen Faktor war die Spalte `last` extern nicht nachrechenbar
  — beim Seitstütz hätte jeder externe Konsument garantiert falsch gerechnet.

**Migration:** Die Übungsversion bleibt bei 7. Die neuen Übungen kommen über den regulären Zweig
durch; Stufenanpassungen, Erholungswerte, gewählte Stufen und Zusatzgewichte bleiben erhalten. Das
war keine Selbstverständlichkeit: Ein Versionssprung hätte alles davon zurückgesetzt, während Luke
täglich mit dieser Fassung trainiert.

**199 Unit-Tests grün** (vorher 127), alle gegen echte Codescheiben aus `index.html`.

### 2026-09-16 g — Einstellungen an die Übung

- **Karte vor jeder Übung** mit Variante, Zusatzgewicht, Satzzahl und dem Knopf *Übung starten*.
  Die Startseite ist auf eine Übersichtszeile je Block geschrumpft.
- **Beide Seiten gleich** bei zweiseitigen Übungen, mit Häkchen zum Entkoppeln. Die Zielzeit
  bleibt je Seite getrennt.
- **Satzzahl je Zone** (A 5, B 4, C 3), im Bereich 1 bis 6 änderbar.
- **Workout vorzeitig beenden** aus jeder Satzpause, der laufende Satz wird gespeichert.
- **Export Schemaversion 2**: neue Spalte `saetze_geplant`.
- Nach einer Unterbrechung führt *Weitermachen* auf die Übungskarte statt direkt in die Uhr.

### 2026-09-16 f — Datenexport

- **Messreihen exportieren** unter *Mehr*: eine Zeile je gehaltenem Satz, als CSV (Excel,
  Numbers) und als JSON (zusätzlich mit Zonengrenzen und allen Stufenleitern).
- Variantenname und Lastfaktor stehen **so in der Zeile, wie sie zum Messzeitpunkt galten**.
  Stufenindizes taugen dafür nicht — die Leitern haben sich seit dem 14.09. zweimal geändert.
- `in_kurve = 1` markiert die Zeilen, aus denen sich CP und W' fitten lassen.
- Das Körpergewicht wandert ab jetzt in jede Session, damit Kilogramm später deutbar bleiben.
- Beschreibung des Formats: **`ISO-Coach_Datenformat.md`**, Beispieldatei
  `ISO-Coach_Messreihen_BEISPIEL.csv`.

### 2026-09-16 c/d — Datenverlust behoben

- **Der laufende Lauf wird nach jedem Satz gesichert** und lässt sich fortsetzen (P28).
- Abbruch betrifft nur den Satz, nicht den ganzen Lauf.
- „Heute" führt während eines Laufs zurück ins Workout.
- **Messpunkt nachtragen** unter *Mehr*.

### 2026-09-16 b — Zusatzgewicht

- **Eingebackene Kilo-Stufen raus** (Ausfallschritt „+ Zusatzlast", Seitstütz „+5/10/15 kg"),
  Zusatzgewicht stattdessen als eigene Achse in Kilogramm (P27).
- **Körpergewicht** unter *Mehr → Messung*, Voreinstellung 80 kg.
- Die Stufenkarten stehen jetzt direkt unter der Workoutkarte, die Übungsauswahl darunter.
- Der Stufenname im Verlauf trägt das Gewicht mit („… + 10 kg"), sonst stünde dieselbe Stufe
  zweimal bei verschiedenen Lasten.

### 2026-09-16 a — Stufenwahl ohne Kurve

- **Anker statt leichtester Stufe** (P26). Der Deadlock im Onboarding ist behoben.
- Gespreizte Startstufen (20 / 50 / 75 % der Leiter), wenn noch gar nichts gemessen ist.
- Sprünge ohne Kurve auf drei Stufen gedeckelt; die Karte nennt die ungedeckelte Rechnung dazu.
- Die Blockkarte weist aus, **woher** der Vorschlag kommt: Kurve, Anker oder Startstufe.

### 2026-09-15 d — Übungen einzeln wählbar

- Neue Karte **„Was heute dabei ist"**: Häkchen je Übung, bleibt stehen. Wer heute nur den
  Seitstütz machen will, nimmt die anderen beiden raus.
- **Erholung wieder je Muskelgruppe** statt je Workout (P24). Tagesabstand, Zonenrotation und
  Einstiegsphase werden alle drei je Gruppe gezählt.
- Ein Workout kann damit **mehrere Zonen** enthalten — je Gruppe die, die dort am längsten
  zurückliegt.
- Der Plan baut sich aus dem, was übrig bleibt: erst die frisch gemessenen Seiten, einseitige
  Übungen in der Mitte, dann die zweiten Seiten.
- **Zweite Seite optional mitmessen** (P25).

### 2026-09-15 b — Seitstütz als dritte Übung

- Neue Übung **Seitstütz**, zweiseitig, eigene Muskelgruppe (`core`). Zehn Stufen von
  „Knie abgelegt, Unterarm auf Arbeitsplatte" bis „Standard + 15 kg an der Hüfte".
- Die Leiter ist **gerechnet, nicht geschätzt**: Biegemoment an der Taille aus einem
  Segmentmodell, jede Schräglage skaliert mit cos θ. Daraus P20 (Fuß-Erhöhung ist keine
  Progression) und die Beschränkung auf Unterarm-Höhe als Regression.
- Einstiegsmessung hängt jetzt an der **Kurve** statt am Workout, und zählt fehlende Zonen
  statt Messungen (P22).
- **Fünf Blöcke** statt drei; Liegestütz exakt in der Mitte (P23).
- Gespeicherte Stufen überleben das Update: `LEVELS_RESET_AT` trennt „Leiter geändert" von
  „Übung dazugekommen". Vorher hätte jede neue Übung die Stufen aller anderen zurückgesetzt.

### 2026-09-15 a — Messablauf, zweite Runde

- Vorlauf nur noch vor Satz 1 eines Blocks (P19).
- Reaktionsabzug je Satz, Voreinstellung 2 s, einstellbar 0–4 unter *Mehr → Messung*. Die
  Satzkarte zeigt Rohwert und Abzug getrennt (P17).
- Vorgabe für Satz 3 aus dem beobachteten statt dem angenommenen Abfall (P18).

### 2026-09-14 h — Neue Stufenleitern

- Liegestütz über Handhöhe und Fußabstand in Fußlängen; schwere Stufen über erhöhte Füße
  (P14, P15).
- Ausfallschritt über die Tiefe, normiert über einen Gegenstand bekannter Höhe unter dem
  hinteren Knie. Keine Handstütze mehr (P14).
- Lastfaktoren sind jetzt geschätzte **Körpergewichtsanteile** statt reiner Verhältniszahlen.
  Damit hat die Lastachse einen Bezugspunkt.
- Zusatzlast als Feinjustierung in beiden Übungshinweisen benannt (P4).
- Datenschema 5.

### 2026-09-14 g — Messablauf

- Pause und Schmerzfrage laufen gleichzeitig; die 20 Sekunden starten sofort und ungebremst
  (P16).
- Drei Sekunden Vorlauf vor jedem Satz.
- Stufenwahl zusätzlich über Minus/Plus — ein Auswahlfeld ist am Boden liegend schlecht zu
  treffen.

### 2026-09-14 f — Seitentrennung und Blockstruktur

- Ausfallschritt in **zwei Kurven** getrennt (links / rechts). Profil und Verlauf haben einen
  Kurvenwähler.
- Workout aus **drei Blöcken**; Liegestütz als Erholungspuffer in der Mitte.
- Startseite wechselt je Workout.
- Zweite Seite wird aufgezeichnet, als *vorermüdet* geführt und aus dem Fit gehalten; im
  Verlauf abgeblendet sichtbar.
- Stufe und Fehlserien hängen jetzt an der Kurve statt am Nutzer (P9).
- Erholung und Zonenrotation auf Workout-Ebene, da ein Workout ohnehin beide Muskelgruppen
  enthält.
- Datenschema 4. Ausfallschritt-Daten ohne Seitenangabe bekommen einen eigenen Eintrag, statt
  auf eine Seite geraten zu werden.

### 2026-09-14 e — Stufenwahl durch den Nutzer

- Auswahlfeld je Block, mit der erwarteten Haltezeit je Stufe, sobald eine Kurve existiert.
- Anzeige, in welcher Zone die gewählte Stufe voraussichtlich landet.
- Gespeichert wird die **erreichte** Zone, nicht die geplante (P10).
- Rückstufungsregel greift bei selbst gewählter Stufe nicht mehr (P11).

### 2026-09-14 d — Erholung nach Fachvorgabe

- Zonenpause 7 Tage, Mindestabstand 2 Tage, Onboarding-Abstand 24 Stunden.
- Muskelgruppenfeld je Übung; überschneidungsfreie Übungen dürfen am selben Tag.
- Pausentag mit Begründung, Termin und bewusstem Übergehen.

### 2026-09-14 c — Ein Test pro Tag

- Vorher plante die App drei Onboarding-Tests ohne Tagesabstand und rotierte die Zonen nach
  **Reihenfolge statt nach Datum**. Beides erlaubte drei Zonen an einem Tag.
- Zonenrotation auf Kalendertage umgestellt.
- Gespeicherter Plan verfällt über Nacht.

### 2026-09-13/14 a–b — Erste Fassung und Installierbarkeit

- Kurve, gespreiztes Onboarding über drei Stufen, Zonenrotation, Progressionsregeln 1–4,
  Schmerzerfassung, Sicherheitsregel, Red-Flag-Abfrage, Timer mit Tonsignalen,
  20-Sekunden-Pausen, Export/Import.
- Acht Stufen statt fünf (P4).
- Als Web-App installierbar gemacht; eigene https-Adresse über GitHub Pages.

---

## Teil 3 — Was der Prototyp bewusst nicht kann

| | Warum |
|---|---|
| Lastfaktoren lernen | Braucht Population und Server (P2) |
| CMF | Numerisch nicht tragfähig (P3) |
| Timer bei gesperrtem Display | Kann eine Webseite nicht (P13) |
| Konten, Sync, Push, Videos, Bezahlung | Außerhalb des MVP |
| Trainingsfortschritt vom Modell trennen | Verbesserung über die Zeit und falsch geschätzte Variantenlast sind aus Haltezeiten allein nicht trennbar — **offen, auch für die echte App** |
| Schmerzabbrüche als untere Schranke | Werden derzeit als volle Messung behandelt, obwohl sie nur sagen „mindestens so lange" — **offen** |

---

## Teil 3b — Was die Achillessehnen-Recherche vom 17.09.2026 für den Bau ändert

Belege und Vorbehalte vollständig in `Recherche_Achillessehne-Schmerz.md`. Hier nur, was den
Prototyp betrifft.

**A1 · Isometrie darf nicht als Schmerzmittel angekündigt werden.**
Modul 4 der niederländischen Leitlinie, wörtlich: „isometric exercises on average have no direct
analgesic effect in patients with Achilles tendinopathy". Isometrie steht dort als **verträglichere
Einstiegsform ab Schmerz 5/10**. Im einzigen RCT zur Ansatzform war die Isometrie zudem in **beiden**
Armen identisch, also nie die geprüfte Größe. → Texte, die „Schmerz weg durch Halten" nahelegen,
gehören raus. Das betrifft Onboarding und Zonenetiketten, nicht die Rechenlogik.

**A2 · Sehnensteifigkeit gehört weder ins Versprechen noch in die Anzeige.**
Keine Leitlinie nennt sie als Ziel; für die Verlaufsmessung existiert laut systematischem Review
„no established measurement instrument". Anzuzeigen ist **Belastbarkeit** und **Dauer der
Morgensteifigkeit** — beides belegt und ohne Erklärkasten verständlich.

**A3 · Der Dorsalextensionswinkel ist die eine Einstellung, um die die Zwei-Formen-Logik gebaut
gehört.** Drei Stellungen: *frei* (Mittelportion, sicher zugeordnet) · *begrenzt* (Ansatz, „beides",
unsicher — Ferse nie unter Bodenniveau, **und Absenken nie mit Kniebeugung kombiniert**) ·
*entlastet* (zusätzlich 12 mm Fersenerhöhung).
**Die Voreinstellung muss „begrenzt" sein.** Grund: In der einzigen Studie zur Selbstzuordnung
trafen nur **82 %** die richtige Region (Kappa 0,67) — und das bei bereits sportmedizinisch
vorselektierten Patienten. Für App-Erstnutzer ist das die Obergrenze. Der Fehler in Richtung
„zu vorsichtig" kostet etwas Reiz, der Fehler in die andere Richtung liefert genau das Protokoll,
das in der Literatur 32 % Zufriedenheit erzeugt hat.

**A4 · Ein dritter Pfad „beides" wird gebraucht.** 8,5 % der Betroffenen haben beide Formen. Zur
kombinierten Form gibt es außer der Häufigkeit keine Evidenz — sie muss auf den sicheren Pfad.

**A5 · Die Zuordnungsabfrage gibt eine Wahrscheinlichkeit aus, keine Diagnose.** Grenze: distale
2 cm. Zwei Zusatzfragen erhöhen mechanistisch die Trennschärfe, sind aber **unvalidiert** und
dürfen nur als Hinweis erscheinen: „Drückt die Fersenkappe auf genau die Stelle?" und „Wird es
bergauf oder beim Absenken unter Stufenniveau schlimmer?"

**A6 · Zwei getrennte Red-Flag-Pfade.** Ansatzform → Rheumatologie (Spondyloarthritis-Verdacht);
Mittelportion → Internist (familiäre Hypercholesterinämie). Zusätzlich: deutlich **medialer**
Punktschmerz am Ansatz (Plantaris-Verdacht, bei 48 % der operierten Ansatzfälle) und **sichtbare,
druckschmerzhafte Schwellung vor der Sehne** — ab etwa 4 mm Bursaerguss greift die
„bleib in Plantarflexion"-Regel möglicherweise nicht mehr.

**A7 · Das Programm darf nicht bei der Isometrie enden.** Die einzige Protokollstufe mit reiner
Isometrie ist eine **Einstiegsstufe mit Weiterpflicht** (mindestens 2 Wochen, dann isotonisch).
Eine App, die dauerhaft nur hält, steht neben der Evidenz.

**A8 · Steuergröße ist der Schmerz, nicht die Last.** Progression nur bei Schmerz < 5/10 während,
eine Stunde danach **und** am nächsten Morgen. Der Schmerz *während* der Übung ist von der Last
unabhängig (4,5 / 4,5 / 4,6 NRS bei 6RM / 10RM / 14RM, p = 0,942) — leicht anzufangen macht es
nicht angenehmer.

**A9 · Offen, aber teuer, wenn es niemand entscheidet:** Der **Fersenkeil** ist als einzelne,
unbetreute Maßnahme besser belegt als jedes Übungsprogramm im Selbststudium (91 % Adhärenz gegen
60 %, 9,6 VISA-A-Punkte besser, KI 1,8–17,4). Gehört er ins Produkt — als Hinweis, als Bestellung,
als Teil des Ansatz-Pfads? Das ist eine Produktentscheidung, keine technische.

---

## Teil 4 — Offene Fachentscheidungen

1. **Wochenpause je Zone** — 7 Tage sind gesetzt, die Literatur trägt sie nicht (P6). Seit
   Build 2026-09-25 c fällt sie als Sperre, sobald der Formtrend einer Gruppe steht (P57).
2. **Zonenetiketten** — „Hypertrophie", „Sehnenkonditionierung", %-MVC-Angaben (P7).
3. **24-Stunden-Schmerzcheck** — gestrichen, aber fachlich die vorgesehene Steuergröße (P12).
4. **Lücke nach dem Onboarding** — drei Einstiegs-Workouts an drei Tagen, danach vier Pausentage,
   weil die Zonenpause eine Woche beträgt. Alternative: die Einstiegstests gleich im
   Mo/Mi/Fr-Raster.
5. **Stufenlisten** — die Höhen- und Abstandsangaben sind geschätzt und gehören nachgemessen.
   Die Lastfaktoren ebenso.
6. **Zweite Seite beim Seitstütz** — sie wird wie beim Ausfallschritt aufgezeichnet, aber nicht
   in die Kurve gelegt. Beim Ausfallschritt ist der Grund zwingend (dasselbe Bein war eben
   hinteres Bein, ~60/40 Überlappung). Beim Seitstütz arbeitet die andere Rumpfseite mit
   eigenen Muskeln — der einzige Einwand ist die allgemeine Ermüdung am Ende der Sitzung. Wird
   sie mitgemessen, verdoppeln sich die Datenpunkte je Seite. Seit 2026-09-15 d als Häkchen
   je Block entscheidbar, Voreinstellung unverändert (P25). Die Grundsatzfrage bleibt offen.
7. **Knie- und Fußversion trennen?** — siehe P21.
8. **Die Annahme `c = 0,5`** in der Ankerrechnung (P26) ist gesetzt. Sie ließe sich später aus
   den Fits vieler Nutzer schätzen — dann wäre sie ein gelernter Startwert statt einer Setzung.
9. **Die Lastfaktoren des Ausfallschritts** sind Lukes Vorschlag und nie nachgemessen. Die
   Ankerrechnung erbt jeden Fehler darin unmittelbar.
10. **Stufenbezeichnungen des Ausfallschritts** — Lukes Hinweis vom 16.09.: „Knie bis zum
    Stuhlsitz … ist auch unklar die Variation". Zurückgestellt, aber offen.
11. **`extraK` je Übung** (1 bzw. 2) ist gerechnet, nicht gemessen — wie die Leiter selbst (P20).
12. **Satzzahl je Zone** — 5 in Zone A und 3 in Zone C sind Lukes Vorgabe („5 für Kraft,
    3 für Ausdauer"). Die **4 in Zone B ist interpoliert**, nicht gesetzt. Offen ist außerdem,
    ob die Satzzahl die Zielzeit beeinflussen sollte: fünf Sätze ermüden stärker als drei, der
    beobachtete Abfall über die Sätze ist aber genau die Größe, aus der `W'` geschätzt wird. Fünf
    Messpunkte je Block liefern dafür mehr Stützstellen als drei — die Frage ist, ob der Gewinn
    an Kurvenqualität die zusätzliche Belastung wert ist.
13. **Satzzahl merken oder jedes Mal neu vorgeben?** — Derzeit setzt jeder Block die Zahl aus
    seiner Zone; eine Änderung gilt nur für dieses Workout. Wer dauerhaft anders trainieren will,
    stellt jedes Mal nach.
14. **Lange Muskellänge** — nach isometrischer Belastung bei langer Muskellänge waren nach 24 h
   noch −20 % Kraft messbar gegenüber −7 % bei optimaler Länge, obwohl die lange Position 28 %
   *weniger* Kraft erzeugte. Das betrifft die tiefen Stufen beider Übungen.

15. **Hebt die Faszienrolle die Last — oder nur den Stabilisierungsaufwand?** Bei gleicher
    Fersenhöhe ist das Hüftmoment dasselbe. Dagegen steht, dass die Rolle wegrollt und aktiv
    gehalten werden muss — das könnte echte Kniebeugerarbeit sein. **Das ist genau das Muster, an
    dem beim Seitstütz die Fußerhöhung als Scheinsprosse aufgeflogen ist** (P20). Rolle und Ringe
    stehen deshalb mit ausdrücklich geschätztem Lastfaktor in der Leiter und gehören als **erstes**
    nachgemessen. Zeigt sich, dass die Rolle die Last nicht hebt, wird sie zur Variante derselben
    Sprosse statt zu einer eigenen.
16. **Die Kontrollposition hat noch keine Instanz.** Beide Übungen, die Luke als „schlecht
    skalierbar" genannt hat, haben sich als vollwertige Leitern herausgestellt — der Schrägzug über
    Fußabstand und Gurtlänge, die Fersenbrücke über den Fersenabstand. Der Mechanismus ist gebaut
    (er repariert nebenbei P30), aber es lohnt sich zu prüfen, ob das eigentliche Problem nicht
    „nicht skalierbar" heißt, sondern „ich habe den Gegenstand dafür nicht" — dann ist es ein
    Ausrüstungsproblem, und dafür ist die Gerätekennung die Antwort (P33), nicht eine eigene
    Datenklasse.
17. **Der Deckel von sechs Blöcken ist gesetzt, nicht hergeleitet.** Er kommt aus der ISO-Stunde
    (08:30–09:30) und aus der Rechnung in P34, nicht aus einer fachlichen Größe. Wie viele
    Haltezeiten in einer Sitzung noch sinnvoll messbar sind, bevor die allgemeine Ermüdung die
    späteren Blöcke verzerrt, ist offen — dieselbe Frage wie bei der zweiten Seite (Punkt 6).
18. **Acht Kurven brauchen acht Erklärungen.** Die Startseite zeigt je Kurve eine Zeile „n von 3
    Zonen". Bei fünf Übungen sind das bereits acht Zeilen — die Profil- und Fortschrittsanzeigen
    brauchen dann eine Zusammenfassung statt einer Liste.
19. **Der einarmige Schrägzug ist keine Seite, sondern eine andere Übung.** Last pro Arm rund 1,4
    Körpergewichte plus ein Drehmoment um die Längsachse, das der Rumpf gegenhält. Das ist der
    P21-Fall und bräuchte eine eigene Kurve, nicht eine Sprosse derselben Leiter.

20. **Eine schlechte Kurve wird nicht mehr gemeldet.** Seit Fassung `c` zeigt das Profil keine
    Streuung mehr (P36) — und sie war der einzige Hinweis darauf, dass eine Kurve ihre eigenen
    Messpunkte verfehlt. Konkret an Lukes Daten: Zwei Haltezeiten von je 82 s bei Last 0,55 und
    1,25 erzeugen eine Kurve, die bei 0,55 **124 s** vorhersagt. Diese 124 s stehen heute
    unkommentiert in der Stufentabelle, und die App setzt sie als Richtwert.
    Die Frage ist also nicht beantwortet, sondern verschoben: **Was soll die App tun, wenn die
    Messungen nicht zu einer Kurve passen?** Drei Wege stehen offen — ein einzelner kurzer Satz
    statt eines Kastens; eine stille Konsequenz (die Vorhersagen verschwinden, wie sie es außerhalb
    des gemessenen Bereichs schon tun); oder bewusst gar nichts.
    **Fachlich** hängt daran mehr als eine Anzeige, denn hinter einer schlechten Kurve stecken zwei
    ganz verschiedene Dinge: ein Messfehler, der sich mit der nächsten Messung von selbst
    herauswaschen wird — oder Lastfaktoren, die für diesen Nutzer nicht stimmen und es nicht tun.
    Das Erste soll die App aussitzen, das Zweite muss sie irgendwann sagen.

21. **Die beiden Wadenheben unterscheiden sich seit dem 18.09. in zwei Dingen gleichzeitig** —
    Handtuchrolle *und* Gelenkwinkel (Mittelstellung gegen Endstellung). Fachlich ist das Lukes
    Entscheidung und gut begründet: die beiden ergänzen sich dadurch über den Bewegungsumfang.
    Als **Messung** ist damit aber die Frage verloren, für die das Paar ursprünglich gebaut wurde —
    „ist das eine Übung oder sind es zwei?". Wer sie beantworten will, braucht eine dritte
    Variante oder einen Tag, an dem beide im selben Winkel gehalten werden.

22. **Der beidbeinige Einstieg misst zweimal denselben Halt.** Steht eine Übung auf einer
    `beid`-Sprosse, sind „links" und „rechts" derselbe Halt — einmal frisch, einmal vorermüdet.
    Der Prototyp plant deshalb nur noch **einen** Block. Offen ist, ob der zweite, vorermüdete
    Halt trotzdem aufgezeichnet gehört: Er misst etwas Echtes (die Erholung zwischen den Sätzen),
    aber er ist keine zweite Seite. Dieselbe Frage wie bei Punkt 6, nur von der anderen Seite her.

23. **Das Wadenheben ist nicht gerätefrei — jedenfalls nicht auf den mittleren Stufen.** Die
    Abdeckungsmatrix führt es auf Platz 3 mit +18,6 Prozentpunkten und zählt es zu den kostenlosen
    Ankerübungen. Die drei mittleren Sprossen brauchen aber eine Personenwaage, und seit Lukes
    Befund vom 18.09. eine **analoge** (P40). Gerätefrei bleiben die 50-%- und die 100-%-Sprosse —
    zwei Haltepunkte im Verhältnis 2,00, die nicht gemeinsam ins Haltezeitfenster passen. Die
    Übung ist damit benutzbar, aber ohne Waage nicht als Leiter. Ob sie den Platz unter den
    kostenlosen Ankerübungen behält, ist eine Produktentscheidung.

24. **Es fehlt das Feld, das über die Gültigkeit jedes Messwerts entscheidet.** Warum ein Halt
    geendet hat — *Muskel · Form · Gleichgewicht · abgebrochen* — steht nirgends, und ohne diese
    Angabe ist nicht unterscheidbar, ob eine Haltezeit eine Maximalzeit war oder nur der Punkt, an
    dem jemand die Lust verloren hat. Das Messfenster aus `2026-09-18 h` fängt davon nur die
    offensichtlichen Fälle (P48). Die Frage ist keine Fleißarbeit: Sie entscheidet, ob eine Kurve
    aus vier Punkten aus vier Messungen besteht oder aus zwei Messungen und zwei Schranken.

25. **Das beidbeinige Wadenheben ist als Sprosse zu leicht — nicht knapp, sondern deutlich.**
    244,2 Sekunden auf der 50-%-Sprosse, und laut Luke wären über 300 möglich gewesen. Rechnet man
    aus diesem einen Punkt zurück, welche Last 150 Sekunden ergäbe, liegt die Antwort über den
    gesamten plausiblen CP-Bereich hinweg zwischen 0,53 und 0,75 — **immer über 0,50 und immer
    unter 1,00**. Die Sprosse liegt also in jedem Szenario unterhalb des Fensters. Der Grund ist
    trivial, sobald man ihn ausspricht: Beidbeinig auf dem Vorfuß stehen belastet jede Wade mit
    etwa dem halben Körpergewicht — ungefähr das, was Gehen ohnehin tut, und der Soleus ist der
    Muskel, der genau das den ganzen Tag macht. Das ist Stehen, kein Trainingsreiz. Ohne Waage
    routet die App deshalb in **allen drei Zonen** zur einbeinigen Sprosse; Zone A bräuchte dort
    Last 1,99, also rund 79 kg Zusatzgewicht. Gerätefrei ist die Wade für Zone A damit nicht
    erreichbar — was Punkt 23 von der Messseite her bestätigt.

    **Erledigt mit Build `2026-09-21 a`.** Die App hing auf der beidbeinigen Sprosse fest, weil
    ein Block ohne Seite ausschließlich dort landen durfte: eine einzige Last, keine Kurve, und
    das Messfenster reicht bis 300 s — also griff auch die Rückstufungsregel nicht. Jetzt
    entscheidet `beidJetzt()` in drei Stufen: die Wahl von Hand für heute, sonst der Leiterstand,
    sonst die Messung. Die Messung schiebt nur nach oben. Die Ausführung steht als Knopfpaar auf
    der Karte vor der Übung und in der Plan-Vorschau, der passende Knopf ist bereits markiert.

26. **Die Schrägzug-Leiter wird ersetzt, die Faktoren sind noch nicht gemessen.** Luke hat am
    21.09.2026 sieben Sprossen festgelegt: 3 / 4 / 5 / 6 Schritte vor (aufrecht, Ringe fest),
    dann flach mit Ringen auf Schulterhöhe und Knien 90°, Ringe niedrig mit Knien 90°, und flach
    mit gestreckten Beinen. Die alten sechs Faktoren gelten dafür nicht mehr. Vorgerechnet sind
    zwei: die gestreckte Waagerechte trägt **0,746** des Körpergewichts (Obergrenze der Übung),
    die gebeugte Variante nach Winters Segmentmassen **0,66** — angewinkelte Knie nehmen nur rund
    11 % ab, nicht ein Drittel. Die vier „Schritte vor" brauchen je eine Bandmaßmessung
    (`sin θ = Schulterhöhe / (0,80 · Körpergröße)`, `Faktor = 0,746 · cos θ`); Luke misst sie mit
    dem Tindeq direkt an den Ringen. **Warnung für die Leiterprüfung:** Der Kosinus ist nahe der
    Waagerechten flach, die Sprossen 4 bis 7 könnten sich innerhalb von 13 % drängen — derselbe
    Befund, der schon die Sprosse mit erhöhten Füßen gekippt hat (P20). Und weil `points()` die
    **gespeicherte** Last eines Satzes liest, repariert eine spätere Faktorkorrektur keine alten
    Messpunkte: gemessen wird vor dem ersten Training auf der neuen Leiter, nicht danach.
    Arbeitsblatt: `ISO-Coach_Schraegzug-Messung_2026-09-21.md`.

    **Teilweise erledigt mit Build `2026-09-21 d`.** Die Leiter stand in der App, mit
    vorläufigen Faktoren.

    **Erledigt mit Build `2026-09-21 e`.** Luke hat am 21.09.2026 mit dem Tindeq gemessen: acht
    Stellungen, je zwei Werte (Hang und Zugposition), bei 83,6 kg und 85 cm Ringhöhe. Die acht
    gemessenen Faktoren stehen in der App (0,179 bis 0,789), die Tabelle im Änderungsprotokoll,
    die Rechnung im Messprotokoll. Beide Warnungen sind beantwortet: Die vorgerechneten Faktoren
    trafen die Hang-Spalte auf 1 bis 4 % — der Kosinus stimmte, nur der Bezugspunkt war der
    Hang statt der Mitte. Und die Gedrängtheit war im Hang real (7 % und 4 %), löst sich aber
    im Mittel auf (kleinster Sprung 11 %), weil die Zugposition oben stärker spreizt. Was
    daraus neu aufgeht, steht als Punkt 27.

    **Nachtrag zu Build `2026-09-21 f`.** Der Absatz darüber beschreibt den Stand von `e` und ist
    an zwei Stellen überholt: Die Faktoren 0,179 bis 0,789 waren das *Mittel* aus beiden Spalten
    und stehen nicht mehr in der App. Gültig ist die Arbeitsspalte allein — 0,072 bis 0,837. Und
    die Gedrängtheit löst sich nicht „im Mittel" auf, sondern in der Arbeitsposition: die vier
    oberen Sprünge liegen dort zwischen 16 und 20 %. Siehe Punkt 27.

27. **Die Bezugsposition des Schrägzugs, und was mit den alten Messpunkten geschieht.**
    ~~Offen.~~ **Erledigt am 21.09.2026 mit Build `f`.** Beides hat Luke am selben Abend
    entschieden. Erstens: Die gemessene Spalte „Pull up position" **ist** die Arbeitsposition
    (Ellenbogen rechtwinklig, Oberarme rund 45° vom Rumpf) — der Lastfaktor ist damit ein
    Messwert, keine Schätzung, und die in Build `e` kurzzeitig eingebaute Mittelung entfällt.
    Eine Kontrollmessung erübrigt sich. Zweitens: Messpunkte von vor dem Faktorwechsel werden
    **verworfen**, nicht umgerechnet — der Skalenfehler dreht auf halber Leiter das Vorzeichen,
    eine Umrechnung hätte auf einer unbeweisbaren Gleichsetzung alter und neuer Sprossen beruht.
    Umgesetzt über `ROW_SKALA_AT = 16`; die Blöcke bleiben im Verlauf sichtbar, abgeblendet und
    mit Grund. Offen bleibt allein die Lieferung: **Fotos für die Sprossen 6, 7 und 8**. Bis
    dahin zeigt die App dort bewusst kein Bild.

    **Nachtrag vom selben Abend — die Hang-Spalte war zweimal falsch gedeutet.** In diesem
    Protokoll (Build `e`, Abschnitt „Die Spanne ist der Preis der Armstellung") und im
    Messprotokoll stand, unten koste die Armhaltung „das Vierfache", und wer so hänge, trainiere
    das Vierfache dessen, was die App verbuche. Die Division 24 ÷ 6 stimmt, beide Schlüsse nicht.
    Erstens gilt das Vierfache für **eine** Stellung, die leichteste: 4,00 · 1,88 · 1,50 · 1,22 ·
    1,12 · 1,04 · 0,90 · 0,89 — ab Sprosse 7 dreht das Vorzeichen. Zweitens stimmte die Richtung
    gegenüber der App nicht: Die App speichert nur die Arbeitsspalte (`load: 0.072 … 0.837`) und
    rechnet Last × Zeit. Wer im Hang hält, hängt passiv und hält länger — die App bucht die
    leichte Last **und** die lange Zeit, schreibt also zu **viel** gut, nicht zu wenig. Was die
    Spalte tatsächlich zeigt: Der Hang trifft auf den unteren Sprossen die Arbeitslast einer
    anderen Sprosse, auf das Kilo genau (Hang „2 Füße vor" = 24 kg = Arbeit „4 Füße vor"; Hang
    „4 Füße vor" = 36 kg = Arbeit „5 Füße vor"). Der hint der Übung („nicht mit gestreckten Armen
    hängen") ist damit eine **Messvorschrift**, keine Warnung vor Überlastung — inhaltlich
    unverändert richtig, nur anders begründet. Die Ursache hat Luke am selben Abend bestätigt: Die
    Hang-Spalte wurde auf **derselben Fußmarke** gemessen, nur mit langen Armen — die längere
    Strecke Schulter–Griff kippt den Körper flacher, und flacher ist schwerer. Offen bleibt allein,
    warum das Vorzeichen ab Sprosse 7 kippt; der Winkel allein erklärt das nicht. Details im
    Messprotokoll, Abschnitt 2.
28. **Die Form entscheidet statt fester Regeln (Vorbild Grip Gains).** Luke, 25.09.: Das
    Training macht erst besser, bis sich nach einigen Einheiten oder Wochen so viel Ermüdung
    ansammelt, dass die Ergebnisse schlechter werden. Das ist funktionelles Überziehen und darf
    weiterlaufen, bis ein deutliches Tal erreicht ist. Dann wird die Menge für einige Zeit
    gesenkt, bis die höhere Form zurückkommt. Das soll das Ziel der App sein, „anstatt
    abiträre regeln zu definieren wann und wie oft jemand trainieren darf".

    **Was Grip Gains öffentlich preisgibt** (Recherche 25.09., Formeln nicht veröffentlicht):
    - Jede Session wird gegen die Vorhersage der eigenen Kurve aus Haltezeit gegen Gewicht
      gelesen. Das ist dasselbe Modell wie hier.
      [FAQ](https://gripgains.ca/resources/faq)
    - Graphen gibt es erst nach 30 Sessions je Greifer.
      [FAQ](https://gripgains.ca/resources/faq),
      [Blog](https://gripgains.ca/resources/blog/post/the-data-behind-the-grind)
    - Die Tagesempfehlung von GG Pro (hart, geplante Session oder Ruhe) kommt aus dem Session
      Trend: Wie lagen die letzten Sessions gegen den eigenen Trend? Feste Zyklen gibt es
      nicht. [GG Pro](https://handofgod.shop/products/grip-gains-pro)
    - Aus den Nutzerdaten: 2,5 bis 3,5 Trainingstage pro Woche gehen mit etwa dem Vierfachen an
      Zuwachs einher gegenüber unter 1,5. Wie gleichmäßig die Tage liegen, spielt keine Rolle,
      nur ihre Zahl. Das ist ein Vergleich über Nutzer, keine Ursache.
      [Blog](https://gripgains.ca/resources/blog/post/the-data-behind-the-grind)
    - Die FAQ (gelesen 25.09., [FAQ](https://gripgains.ca/resources/faq)):
      - Der Performance Trend von 0 bis 100 ist ein Oszillator gegen die eigene Geschichte.
        50 ist das eigene übliche Niveau, über 70 bis 80 das obere Ende der eigenen Spanne.
        Er ist geglättet, damit eine einzelne Ausreißer-Session ihn nicht kippt.
      - Der Kurzfrist-Trend sagt, ob die letzten Sessions über oder unter dem eigenen jüngsten
        Niveau liegen. Er sagt die nächste Session am besten voraus (Auswertung über 480
        Athleten). Eine Erholungs- oder Übertrainingsanzeige ist er ausdrücklich nicht.
      - Jede Session wird geloggt, auch früh beendete; die sagen etwas. Sessions bis zum
        völligen Versagen fallen aus dem langen Trend.
      - Die Zonen werden normiert: 10 % weniger Ausdauer zählen wie 10 % weniger Kraft.
      - Entscheiden soll der Mensch. Eine Kennzahl aus Leistung und Umfang zeigt an, wann
        entlastet werden sollte.
      - 100 gute Sessions im Jahr schlagen 200 halbe. Es gibt einen Krankheitstag-Modus.
      - Statt einer Gesamtzahl gibt es mehrere Anzeigen. Die Periodisierung ergibt sich aus
        dem Trend, nicht aus einem Plan.

    **Entschieden (Luke, 25.09.), gebaut in Build 2026-09-25 c (P57):**
    - *Signal:* nur der erste Satz jedes Blocks, gegen die Kurve aus den Tagen davor, in Kraft
      umgerechnet.
    - *Fenster:* kurz drei Trainingstage, lang 30 Tage.
    - *Tal:* Formwert 0 bis 100 als Rang gegen die eigene Geschichte, Tal unter 20, es hält bis
      50. Unter 50 zu bleiben ist erlaubt, dort baut sich Kraft auf. Die Schwellen 20 und 50
      sind meine Setzung.
    - *Menge senken:* entscheidet der Nutzer. Die App spricht im Tal nur eine Empfehlung aus
      (Premium).
    - *Aufbau begrenzen:* einmal am Tag bleibt, die Rotation wählt weiter die Zone. Die
      Zonenruhe fällt als Sperre.
    - *Ab wann:* 30 Tage nach dem ersten Messtag der Übung.

    **Entschieden (Luke, 25.09.), gebaut in Build 2026-09-25 d:**
    - *Premium* ist entschieden; was es ist, steht im Marketing-Kanon. Die Übungen selbst
      bleiben ohne Schranke; hinter Premium liegt nur die Empfehlung aus dem Trend.
    - *Ohne Premium* bleibt die Zonenruhe (Luke: „ja wie du sagst").

---

*Kein Medizinprodukt. Keine Diagnose, keine Therapieempfehlung.*
