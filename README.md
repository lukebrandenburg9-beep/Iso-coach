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
Umbauten trotzdem einmal über **Einstellungen → Sichern (Datei)** sichern.

**Ausnahme Build `2026-10-25 a`:** Er konnte gespeicherte Daten nicht laden
und zeigte eine leere App mit Onboarding (Papier P100). Gelöscht wurde dabei nichts, solange man
nicht weitergemacht hat. Build `2026-10-27 a` lädt sie wieder. Schlägt das Laden künftig fehl,
legt die App den Stand vorher unter `isocoach.v1.gerettet` ab; „Alles löschen“ entfernt auch ihn.

**Das Repository ist öffentlich**, weil GitHub Pages ohne bezahltes Konto nur so funktioniert.
Unbedenklich: In den Dateien stehen keine persönlichen Daten — die Messwerte entstehen erst
beim Benutzen und bleiben im Browser des jeweiligen Nutzers. Das Handoff mit den Zugängen liegt
eine Ebene höher und ist nicht Teil des Repositorys.

---

## Was sich geändert hat und warum

[ISO-Coach_Probleme-und-Aenderungen.md](ISO-Coach_Probleme-und-Aenderungen.md) liegt neben
`index.html` und wird mit jedem Build nachgezogen:

- **Teil 1:** Probleme, die den Aufwand gegenüber dem Handoff verschieben, je mit Preis und
  Hinweisen für den Nachbau.
- **Teil 2:** jede Fassung mit Befund, Änderung und dem, was geprüft wurde.
- **Teil 3 und 4:** was der Prototyp bewusst nicht kann, und offene Fachentscheidungen.

---

## Was drinsteckt

Der Prototyp setzt Abschnitt 4 des Handoffs um — er ist kein Timer mit Deko, sondern die
Rechenlogik:

- **Die Kurve** `t = W'/(P − CP)`, gefittet über eine eindimensionale Suche, ältere Messpunkte
  werden mit 3 % je neuerem Punkt abgewertet. **Sie rechnet mit den letzten 30 Punkten** je
  Kurve, ältere bleiben im Export. Im Profil wird ein älterer Punkt so blass, wie
  wenig er noch zählt. Ein Punkt mit Gewicht bekommt einen Pfeil von der Linie seiner Sprosse,
  die kg-Zahl steht klein und grau daneben, wo Platz ist. Die Zeichnung beginnt links, wo die
  Kurve oben aus dem Bild läuft. Eine senkrechte Linie steht an der Obergrenze der Kraftausdauer
  (180 s). Stufen links davon sind zu leicht, sie fallen von der Achse und stehen in der Liste
  darunter ausgegraut. Die aktuelle Stufe bleibt immer im Bild. Bei Übungen mit linker und
  rechter Seite liegen beide Kurven in einer Zeichnung: links gestrichelt mit Quadraten, rechts
  durchgezogen mit Kreisen, ein beidbeiniger Halt als Raute. Die 180-s-Linie steht an der
  schwächeren Seite, die Tabelle hat je eine Spalte für links und rechts. Haltungen wie bei der
  Hyperextension bleiben getrennt. Unter dem Feld „Kurve“ lassen sich beide Seiten an- und
  abhaken. Sitzt das Gewicht links und rechts verschieden (Läufer), stehen zwei Zeichnungen
  untereinander, jede mit ihren Gewichtsmarken.
- **Gespreiztes Onboarding** über drei Stufen (leicht / mittel / schwer) statt einer. Der
  Einstieg hängt an der **Kurve**, nicht am Workout: Eine Übung, die später dazukommt, holt ihre
  fehlenden Zonen nach. Gezählt werden dabei leere Zonen, nicht Messungen — zwei gespreizte
  Punkte sind mehr wert als fünf aus derselben Zone.
- **Die Stufe wählst du**, nicht die App. Sie schlägt eine vor und zeigt je Stufe die erwartete
  Haltezeit sowie die Zone, in der sie voraussichtlich landet — entscheiden tust du. Der
  Messpunkt zählt so, wie er herauskommt: Er wird der Zone zugeordnet, in der er tatsächlich
  liegt, und den Richtwert zu verfehlen ist bei selbst gewählter Stufe kein Fehlschlag.
- **Zonenrotation:** Die App fordert die Zone an, die am längsten zurückliegt — **und erholt
  ist**. Ohne Spreizung wird die Kurve wertlos, ohne Erholung die Messung. Zonen, die eine Übung
  mit ihren Sprossen nach den eigenen Messungen nicht erreicht, überspringt die Rotation;
  erreicht sie nur eine, kommt die Übung einmal pro Woche dran.
- **Drei Zonen, drei Farben:** Maximalkraft (20–60 s, rot), Hypertrophie / Kraft (60–120 s, blau),
  Kraftausdauer (120–180 s, gelb). Bis Build `2026-10-22 e` waren es 60–90 s und 90–150 s. Auf *Start* steht unter jeder angehakten Übung je Seite ein
  Kuchen der letzten 30 Einheiten — gezählt nach der Zone, in der der erste Satz gelandet ist —
  und daneben die drei Zonen zum Antippen. Der Knopf der Zone, die heute läuft, ist voll in ihrer
  Farbe gefüllt. Die Zone, die die App empfiehlt, bleibt orange umrandet, auch wenn man eine
  andere antippt; ein Tipp darauf führt zurück zur Empfehlung. Eine Zone, die noch ruht, lässt
  sich trotzdem wählen;
  die App sagt dann, wann sie erholt wäre. Was die Übung nicht erreicht, ist ausgegraut.
- **Satzkurve je Seite.** Auf der Karte vor jeder Übung steht unter der Kopfzeile eine kleine
  Kurve: Haltezeit je Satz, vom ersten bis zum letzten, dazu die Gesamtzeit mit Pausen. Satz *n*
  hält `a + (1−a)·q^(n−1)` mal so lange wie Satz 1. Am Anfang rechnet die App mit dem Studienwert
  (nach 20 s Pause ist rund ein Drittel der Reserve zurück, Chorley 2022), danach mit den eigenen
  Messreihen — erst über alle Übungen, dann je Übung und Seite. Der Timer gibt ab Satz 2 genau
  diese Zahlen vor; wer länger hält als geplant, bekommt einen längeren nächsten Satz. Dieselbe
  Rechnung steckt in der Dauer auf *Start*.
- **Erholung:** Dieselbe Muskelgruppe **dreimal pro Woche**, jedes Energiesystem — also jede
  Zone — **einmal pro Woche**. Beides zusammen ist genau eine Rotation: drei Zonen, drei
  Einheiten, ein Mo/Mi/Fr-Rhythmus. Umgesetzt als zwei Schranken: mindestens zwei Tage zwischen
  zwei Einheiten derselben Muskelgruppe, sieben Tage bis dieselbe Zone wiederkommt.
  Während der drei Einstiegstests genügen **24 Stunden** — sonst zöge sich das Onboarding über
  drei Wochen. Übungen **ohne Überschneidung** (Liegestütz, Ausfallschritt, Seitstütz, Schrägzug,
  Fersenbrücke, Hip Hinge, Hyperextension) dürfen am selben Tag getestet werden; die App sagt am Pausentag, welche Gruppe noch frei ist. Der Pausentag
  nennt Grund und Termin und lässt sich bewusst übergehen. Werte unter *Einstellungen → Erholung*.
  Dort steht auch **„Ohne Pause"**: Ein Haken je Übung hebt Tagessperre und Zonenruhe auf, die
  Übung läuft dann so oft am Tag, wie du willst. Nach einem Schmerzabbruch ist für den Tag
  trotzdem Schluss. Die zweite Einheit am Tag zählt in der Kurve als frisch.
- **Höchstens sechs Blöcke** je Workout. Fünf Übungen, davon drei zweiseitig, wären acht Blöcke
  — in der Kraftausdauer über eine Stunde. Die App nimmt deshalb die freigegebenen Übungen in der
  Reihenfolge, wie lange ihre **Kurve** nicht mehr gemessen wurde, und stellt den Rest auf die
  nächste Sitzung zurück; dort stehen sie oben. Wer selbst entscheiden will, benutzt auf der
  Startseite *Heute selbst zusammenstellen* — das gilt nur für diese eine Sitzung und hebt den
  Deckel auf.
- **Kurzeinheit (20 Min).** Ein Häkchen in der Plankarte über *Weiter*, gültig bis
  Mitternacht. Die App kürzt dann so, dass die Einheit mit allen Pausen höchstens 20 Minuten
  dauert: erst die Satzzahl, für alle Übungen gleich, vom Zonen-Standard bis auf einen Satz, dann
  ganze Übungen, links und rechts immer zusammen. Zuerst fallen Übungen weg, die in den letzten
  7 Tagen trainiert wurden, zuletzt die heute selbst angehakten und der Nachholblock. Zwischen den
  Blöcken läuft eine Minute Pause als Countdown auf der nächsten Karte, gestartet wird per Tipp.
  Die Kurve nimmt wie immer nur den ersten Satz. Der Export markiert solche Sätze in der Spalte
  `kurzeinheit`.
- **Nachholblock zum Abwählen.** Wer die offene Seite nicht mehr nachholen will, nimmt das Häkchen
  der Zeile ab. Bis Mitternacht lässt es sich wieder setzen, danach plant die App die Übung normal.
- **Dein Workout.** *Weiter* auf *Start* führt auf eine Liste der angehakten Übungen in der
  Reihenfolge des Plans. Dort stellst du für jede Übung schon vorher Sprosse, Zusatzgewicht und
  Sätze ein, je Seite eine Zeile mit der Zielzeit im Kopf. Oben springt die Dauer mit. Die Karte vor
  jeder Übung ist damit vorbelegt und lässt sich weiter ändern. Die Wahl gilt bis Mitternacht und
  bleibt stehen, wenn du auf *Start* etwas an- oder abhakst. Mit der Kurzeinheit verfallen nur die
  gewählten Sätze.
- **Gewicht mit Schieber und Tasten.** Zusatzgewicht und Körpergewicht wählst du überall gleich, in
  *Dein Workout*, im Nachtrag, im Onboarding und unter *Einstellungen → Messung*: oben der Wert,
  darunter ein Schieber mit − und + an der Seite. Der Schieber rastet auf vollen Fünfern, für
  schnelle Sprünge um 5 und 10 kg, die Tasten gehen 0,5 kg. Planer und Regeln schlagen weiter im
  Raster der Übung vor, meist 2,5 kg.
- **Progressionsregeln 1–4** über zwei Bahnen: Leiter und Zusatzgewicht. Am oberen Ende der
  Leiter kommt Gewicht dazu statt gar nichts, am unteren geht erst Gewicht weg, bevor die Stufe
  fällt. Zweimal Schmerzabbruch stuft zurück — und wenn nichts Leichteres mehr da ist, sagt die
  App das ausdrücklich, statt zu schweigen.
- **Die Kurve gilt nur, wo gemessen wurde.** Sie merkt sich die kleinste und größte Last, aus der
  sie stammt. Außerhalb zeichnet das Profil gestrichelt und schreibt „außerhalb" statt einer Zahl
  — keine Vorhersage über Lasten, bei denen du nie warst.
- **Vorläufige Sprossen stehen in Rot.** Solange eine Sprosse nicht gemessen ist, steht unter
  der Auswahl auf der Karte „Sprossenwert noch vorläufig, muss noch gemessen werden", in der
  Auswahl selbst und im Profil das Kürzel „· vorläufig". Das Rot verschwindet, sobald die Sprosse
  in der Liste der nachgemessenen Sprossen steht (sie wächst mit jedem Build) oder auf diesem
  Gerät ein gültiger Halt darauf liegt: frisch, in der Kurve, 20 bis 300 s, egal welche Seite.
- **Kontrollposition.** Liegen die verfügbaren Stufen einer Übung weniger als Faktor 1,25
  auseinander, gibt es **keine Kurve**, sondern einen Verlauf der Haltezeit über die Wochen. Zwei
  zu eng beieinander liegende Lasten ergeben rechnerisch keine Kurve — sie ergeben eine, die
  perfekt aussieht und falsch ist.
- **Schmerz** als Häkchen pro Satz plus ein Ampelwert am Sessionende.
- **Sicherheitsregel:** dreimal in Folge Ampelwert 6 oder höher → Hinweis auf ärztliche Abklärung.
- Red-Flag-Abfrage im Onboarding, Timer mit Tonsignalen, 20-Sekunden-Pausen, Export/Import.
- **Selbsttest zur Kniebeugung im Onboarding**, freiwillig. Die Antwort sagt, ab welcher Sprosse der
  Ausfallschritt Gewicht bekommt (siehe unten).
- **Sicherheitsfrage, wenn eine Übung eine neue Körperregion öffnet.** Der Fragebogen läuft sonst
  nur im Onboarding — vor der ersten Fersenbrücke wird die Frage zur proximalen Hamstring-Sehne
  nachgeholt, vor dem ersten Hip Hinge die nach Ausstrahlen ins Bein, Taubheit und plötzlichem
  Beginn.

**Bewusst nicht drin:** CMF. Die Begründung stand bis zum 17.09. auch in der App und ist dort
wieder raus — sie erklärt ein Verfahren, das der Nutzer nie zu sehen bekommt. Nachzulesen als
P3 in [ISO-Coach_Probleme-und-Aenderungen.md](ISO-Coach_Probleme-und-Aenderungen.md).

---

## Was du an Gerät brauchst

Unter **Einstellungen → Geräte** hakst du an, was du hast. Die meisten Übungen kommen mit Möbeln
aus — Arbeitsplatte, Tisch, Stuhl, Hocker, Stepper, Kiste, Kissen, Sofa. Gerät braucht es hierfür:

| Gerät | wofür | voreingestellt |
|---|---|---|
| Etwas zum Daranziehen | Schrägzug, **die ganze Übung** | an — eine stabile Tischkante zählt mit |
| Faszienrolle | Fersenbrücke, die zwei obersten Stufen beidbeinig und einbeinig (Knie 90° und langer Hebel 170°), dazu der ganze Pfad fürs innere Knie | aus |
| Kettlebell oder Kurzhantel | Aufrechter Läufer, Zusatzgewicht | aus |
| Personenwaage mit Zeiger (analog) | Isometrisches und endgradiges Wadenheben, einbeinig mit Entlastung | aus |
| Spanngurt mit Türanker | Hyperextension, die zwei leichten Stufen im Türrahmen. Ohne Gurt beginnt die Leiter am Stepper | aus |

Stufen, die ein fehlendes Gerät brauchen, erscheinen nicht in der Auswahl — aber die
**Nummerierung bleibt stehen**. Sonst verschöbe sich die gespeicherte Stufe deiner alten
Messungen, sobald du ein Gerät anhakst. Eine Übung, von der keine einzige Stufe ausführbar ist,
taucht im Workout gar nicht erst auf.

---

## Stufen und Lastfaktoren

In der App lassen sie sich seit Build `2026-10-22 a` nicht mehr einstellen. Die Werte sollen nie
von Hand gesetzt, sondern statistisch aus den Nutzerdaten ermittelt werden (Handoff 4.2). Bis
dahin stehen sie im Code in `DEFAULT_EX` und gelten für alle. Ein früher von Hand geänderter Wert
fällt beim nächsten Laden auf den Build zurück; gespeicherte Messpunkte behalten ihren Faktor.

Die Lastfaktoren sind Verhältniszahlen, keine Kilogramm — die leichteste Stufe ist 1,00, alles
andere bezieht sich darauf. Sie sind **grob geschätzte Startwerte** und liegen in deinem
Fachbereich.

Ein Punkt, der sich beim Testen gezeigt hat und den du kennen solltest: **Wie fein die Stufen
abgestuft sind, entscheidet, welche Zonen überhaupt trainierbar sind.** Mit den ursprünglich
fünf groben Stufen war Hypertrophie / Kraft (60–90 s) für einen durchschnittlichen Nutzer gar nicht
erreichbar — der Sprung von Stufe zu Stufe übersprang sie. Deshalb jetzt acht Stufen mit
einem Lastverhältnis von etwa 1,15 zwischen benachbarten Stufen. Wenn du die Stufen im Code änderst,
prüfe unter **Profil → „Was du auf jeder Stufe schaffst"**, ob noch alle drei Zonen vorkommen.

Der Ausfallschritt ist mein Vorschlag und nicht abgestimmt — Tiefe und Unterstützungsgrad als
Progression. Der Liegestütz folgt der Stufenfolge aus dem Handoff, nur feiner unterteilt.

**Der Ausfallschritt hat seit Build `2026-10-23 c` zwei Stellungen in einer Leiter** (Papier P96).
Unter jeder Tiefe des Kniestands steht der enge Kniestand derselben Tiefe, bei dem das hintere Knie
am tiefsten Punkt in einer Linie mit den Zehen des vorderen Fußes steht. Er trägt vorerst dieselbe
Last, bis die ersten Messungen da sind. Welche Stellung gilt, sagt das Kästchen „Es zieht am hinteren
Bein in der Hüfte oder im Knie“ in „Dein Workout“, je Seite. Die oberste Sprosse ist der hintere Fuß
mit dem Spann auf Hantelbank oder Sofa. Ihre Höhe trägst du an der Sprosse ein, die Last folgt aus
ihr: 0,90 × (1 + 0,18 × Höhe / 42 cm), also 1,06 bei 42 cm. Die alte Sprosse mit dem Fuß auf dem
Stepper ist raus, ihre Messungen bleiben mit 1,25 in der Kurve.

**Beim Ausfallschritt kommt Gewicht erst nach der Tiefe**, seit Build `2026-10-25 a` (Papier P98).
Gesteuert wird das über den Selbsttest im Onboarding: „Geh in einen Ausfallschritt und senk das
hintere Knie langsam ab. Kommst du so tief, dass das Knie knapp über dem Boden schwebt, ohne dass es
dabei mehr als leicht zieht?“ Bleibt er offen, fragt die Karte vor dem ersten Ausfallschritt einmal
nach. Ändern lässt sich die Antwort unter *Einstellungen → Ausfallschritt*, ohne Antwort gilt der
Normalfall.

- **„Ja, ohne Probleme“:** Gewicht gibt es nur auf den drei obersten Sprossen, „Knie knapp über den
  Boden“ normal und eng und auf der Bank.
- **„Nur mit Schmerzen oder Ziehen“ oder „Nein, ich komme nicht so tief“:** Gewicht gibt es schon auf
  allen Stepperhöhen. Die App plant je Seite nur bis zu einer Tiefengrenze, anfangs 1 Stufe (10 cm),
  und legt dort Gewicht auf, statt tiefer zu gehen. Wählst du in der Liste eine tiefere Sprosse, wird
  sie die Grenze. Geht eine Seite wegen Schmerz auf eine Kniestand-Sprosse zurück, wird diese die
  Grenze. Eine neue Antwort im Selbsttest löscht die Grenzen.

Standard ist Suitcase, seit Build `2026-10-26 a` (Papier P99): ein Gewicht, Kurzhantel oder
Kettlebell, in der Hand gegenüber dem vorderen Bein. Steht der linke Fuß vorn, hältst du es rechts.
Nach Bill Hartman hilft diese Hand dem vorderen Bein beim Absenken in die Innenrotation. Kurzhanteln in
beiden Händen, Front Rack oder eine Gewichtsweste gehen auch. Eingetragen wird das Gesamtgewicht.

Die Seitstütz-Leiter ist **gerechnet**, nicht geschätzt: Das Biegemoment an der Taille
(Segmentmodell, Schnitt L4/L5) fällt in jeder Schräglage mit cos(Neigungswinkel). Zwei Folgen,
die dem üblichen Rat widersprechen:

- **Unterarm erhöhen** ist die saubere Regression — der Hebel bleibt, nur der Winkel dreht sich.
  Messbar in Zentimetern.
- **Füße erhöhen ist keine Progression.** Es verschiebt Last auf die Stützschulter (die im
  flachen Seitstütz schon rund 70 % des Körpergewichts trägt); das Rumpfmoment sinkt dabei sogar
  leicht. Die Stufe steht deshalb bewusst nicht in der Leiter.
- **An der Wand** stehst du auf den Füßen, der Unterarm liegt flach an der Wand, der Ellenbogen auf
  Schulterhöhe. Der Fußabstand stellt die Neigung ein: 2 Fußlängen tragen 0,13, 3 Fußlängen 0,31.
  Beides ist mit demselben Gesetz gerechnet, für jede Körpergröße gleich, und vorläufig, bis es
  gemessen ist (Papier P92). Die Knie-Sprossen auf Möbeln sind seit Build `2026-10-22 g` raus: Mit
  abgelegten Knien liegt die Schulter zu tief für eine erhöhte Auflage.

Oberhalb des Standards endet die Leiter. Zusatzlast ginge rechnerisch nur nah an der Taille
(10 kg an der Hüfte sind +25 %, am Brustkorb +3 %), aber der Seitstütz bleibt bewusst eine reine
Körpergewichtsübung. Die App bietet bei ihm kein Gewicht an.

**Der Schrägzug folgt demselben Gesetz.** Last = 0,746 · cos θ, wobei θ der Körperwinkel über der
Waagerechten ist — gegen vier publizierte Kraftmessungen am Gurt geprüft, größte Abweichung 5,1 %.
Die Stufen sind deshalb **Winkelmarken**, getroffen über Fußabstand (trägt bis etwa 37°) und
Gurtlänge (übernimmt ab dort). Und damit zum zweiten Mal: **Füße erhöhen ist keine Progression.**
Es dreht dich nur zur Waagerechten, und dort hat cos θ sein Maximum — darüber hinaus fällt die
Last wieder. Oberhalb geht es nur mit Zusatzgewicht weiter.

Der **einarmige** Schrägzug ist keine Stufe davon, sondern eine andere Übung: rund 1,4
Körpergewichte pro Arm plus ein Drehmoment um die Längsachse, das der Rumpf gegenhält.

Die **Fersenbrücke** skaliert über zwei Dinge: die Höhe der Unterlage unter der Ferse (Stuhlsitz,
Hocker, Kiste, Boden, Faszienrolle) und den Kniewinkel. Es gibt zwei Stellungen, **Knie 90° und
langer Hebel 170°**, und jede Unterlage in beiden: erst 90°, dann 170°. Langer Hebel 170° heißt: Das
Bein ist fast gestreckt, das Knie bleibt leicht gebeugt und wird nicht durchgedrückt. Seit Build
`2026-10-25 a` steht er an der Stelle von Knie 135° (Papier P98). Er ist etwas schwerer als 90° auf
derselben Unterlage und leichter als 90° auf der nächsten: 90° auf dieser Unterlage × (90° auf der
nächsten / 90° auf dieser) hoch 0,75. Stuhl 90° und Hocker 90° tragen ihre alten, gerechneten Werte,
alle übrigen Sprossen sind **Platzhalter** und stehen in der App rot als vorläufig, bis gemessene
Haltezeiten da sind. Messungen auf den alten 135°-Sprossen bleiben mit Last und Namen in der
Kurve. Ein Segmentmodell sagt, dass die Höhe bei gleichem Winkel kaum etwas ändert und der Winkel
fast alles. Die Messungen entscheiden, ob die Leiter so bleibt.

Unter den Sitzbeinen steht ein **Yogablock** (22,5 × 12 × 7,5 cm), damit das Becken nicht
unbemerkt absinkt. Abgesunken wird die Oberschenkelrückseite länger und stärker, und derselbe Halt
auf derselben Sprosse würde etwas anderes messen. Die Karte sagt je Sprosse, wie der Block steht.
Bei 90°: Stuhl aufrecht, Hocker hochkant auf einem 5 cm dicken Buch, Kiste hochkant, Boden flach,
Rolle hochkant. Beim langen Hebel liegt das Becken tiefer: Stuhl hochkant, Hocker flach, auf Kiste,
Boden und Rolle steht keiner. Leichte Berührung wird korrigiert, Druck auf dem Block beendet den Halt.

**Die Faszienrolle ist geschätzt:** Ob die Rolle die Last wirklich hebt oder nur den
Stabilisierungsaufwand, ist offen. Bei gleicher Fersenhöhe ist das Hüftmoment dasselbe. Das ist
genau das Muster, an dem beim Seitstütz die Fußerhöhung als Scheinstufe aufgeflogen ist. Die
Ringe sind seit dem 15.10. raus, sie waren zu viel Gerät.

Vor den zehn einbeinigen Sprossen steht eine **beidbeinige Rampe** auf denselben Unterlagen und
Winkeln, von beidbeinig Stuhl 90° bis beidbeinig Rolle 170°. Wer neu anfängt, klettert sie ganz
hinauf und steigt erst dann auf einbeinig Stuhl um. Alle beidbeinigen Sprossen liegen unter
einbeinig Stuhl, ihre Lasten sind **Platzhalter**: Einbeinig belastet ein Bein mehr als doppelt so
stark wie beidbeinig, die reine Gewichtsrechnung (halbe Last) unterschätzt das. Die echten Werte
kommen aus gemessenen Haltezeiten. Auf der Karte lässt sich beidbeinig oder einbeinig wählen; die
App wählt vor.

**Der Pfad fürs innere Knie.** Auf der Karte der Fersenbrücke steht je Bein ein Häkchen
„Probleme mediales Knie / Baker-Zyste", dieselben Häkchen stehen unter *Einstellungen*. Ein angehaktes Bein
bekommt eine eigene Leiter auf der Faszienrolle mit eigener Kurve. Sie hat zwei Sprossen, beide mit
der Ferse auf der Rolle: erst beidbeinig, dann einbeinig. Vorfuß und Fußstellung sind seit dem 16.10.
raus (P83). Der Vorfuß geht auf die Wade, und die Fußstellung macht in der Brücke keinen Unterschied.
Weiter geht es nach zwei Halten in Folge ohne Schmerz, zurück nach zwei Schmerzabbrüchen in Folge.
Solange der Pfad beidbeinig läuft, ist das gesunde Bein dort mit drin und bekommt keinen eigenen
Trainingsblock. Ohne Faszienrolle wird das angehakte Bein nicht trainiert. Der Kniewinkel (vorläufig
90°) ist noch offen und steht in der App rot. Der Yogablock steht dort hochkant, mit derselben
Berührungsregel.

Bei der **Hyperextension** beginnt die Leiter seit Build `2026-10-23 d` am **Gurt im Türrahmen**
(Papier P97). Der Türanker klemmt oben in der Zarge, die Schlaufe liegt vorn knapp unter den
Hüftknochen und läuft rechtwinklig zum Körper. Man lehnt sich gestreckt hinein, bis der Körper 45°
geneigt ist. Am Gurt gibt es zwei Sprossen: den Oberkörper in Verlängerung der Beine und den
Oberkörper waagerecht. Danach kommt der **Stepper** mit Kissen: Die Kante liegt auf Höhe des
Bauchnabels, die Füße stehen hinten an der Wand, und der Oberkörper wird frei und waagerecht
gehalten. Am Stepper bleiben zwei Armhaltungen. Die Last ist das Moment des Oberkörpers um den
Drehpunkt, bezogen auf die frühere Sprosse „Arme in Y“ am Stepper = 1,00:

| Sprosse | Last | Herkunft |
|---|---|---|
| Gurt · Oberkörper 45° in Verlängerung der Beine | 0,44 | Platzhalter, der waagerechte Gurt mal sin 45° |
| Gurt · Oberkörper waagerecht | 0,62 | Platzhalter, eine Torstufe (1,25) unter dem Stepper |
| Stepper · Hände in der Hüfte | 0,78 | gerechnet, Segmentmodell von de Leva (1996) |
| Stepper · Hände vor der Brust | 0,85, mit Gewicht mehr | gerechnet, Armhaltung angenommen |

Der Gurt ist nach Lukes Erfahrung wesentlich leichter als der Stepper. Das Segmentmodell sagt das
Gegenteil: Am Gurt hält der Rücken den ganzen Oberkörper über der Hüfte, am Stepper nur den über
dem Nabel. Was stimmt, zeigt die Messung, bis dahin stehen beide Gurt-Sprossen rot. Ohne Spanngurt
beginnt die Leiter am Stepper. Zeigefinger, Arme in V, T und Y sind raus, ihre Messungen bleiben
mit Last und Namen in der Kurve.

Die Übung hat **zwei Haltungen** mit je eigener Kurve, **gestreckt** mit langem Rücken
und **rund** mit eingerolltem Rücken. Pro Workout läuft nur eine; auf der Karte vor der Übung
lässt sie sich wechseln. Vorgewählt ist gestreckt, und das passt zur Mehrheit: Rund 90 % der
Ischialgien gehen von der Bandscheibe aus (Koes 2007), und dann ziehen Sitzen und Vorbeugen ins
Bein. Beim engen Wirbelkanal ist es umgekehrt, dort ziehen Stehen und Gehen ins Bein (Inufusa
1996). Genau diese zwei Sätze stehen beim ersten Mal unter den Knöpfen.

Der **Hip Hinge** belastet die Sehne am Sitzbein, das obere Ende der Oberschenkelrückseite. Er
ergänzt die Fersenbrücke, die am unteren Ende arbeitet, hat eine eigene Erholungsgruppe und darf
am selben Tag laufen. Die Bewegung kommt aus der Hüfte, der Rücken bleibt neutral, im Stehen
sind die Knie leicht gebeugt und bleiben es. Gerechnet wird mit dem Hüftmoment des Oberkörpers je
arbeitendem Bein, stehend beidbeinig ist 0,50:

| Sprosse | Last | Herkunft |
|---|---|---|
| Sitzend, aus der Hüfte nach vorn gekippt | 0,32 | Platzhalter |
| Sitzend, Gesäß knapp über der Sitzfläche | 0,40 | Platzhalter |
| Stehend, Körpergewicht | 0,50 | gerechnet |
| Stehend, Gewicht in beiden Händen | ab 0,63, mit Mindestgewicht | gerechnet |
| Camporini, Bein hinten, ganze Sohle auf dem Keil | 0,65 | Gewichtsanteil hinten, einmal gemessen |
| Camporini, beide Füße flach, Gewicht in der Hand gegenüber | 0,65 | wie Camporini, nur Premium |
| Camporini, beide Füße flach, Gewicht in derselben Hand | 0,65 | wie Camporini, nur Premium |
| Camporini, ganze Sohle auf dem Keil, Gewicht in der Hand gegenüber | 0,65 | wie Camporini, nur Premium |
| Camporini, Zehen auf dem Keil, Ferse unten, Gewicht in der Hand gegenüber | 0,65 | wie Camporini, nur Premium |
| B-Stance, Bein vorn, hinterer Fuß nur auf den Zehen | 0,75 | angenommen |

Im Sitzen ist das Hüftmoment bei gleicher Neigung dasselbe wie im Stehen. Leichter ist es, weil
das stark gebeugte Knie die Oberschenkelrückseite entspannt, und das rechnet kein Hebelmodell —
darum Platzhalter. Beim **Camporini** steht der hintere Fuß mit der ganzen Sohle auf einem Keil,
die Ferse höher als der Vorfuß, und das hintere Bein trägt den größeren Teil des Gewichts. Beim
**B-Stance** ist es umgekehrt, das vordere trägt. „Links" heißt deshalb: Das linke Bein arbeitet
— beim Camporini steht es hinten, beim B-Stance vorn. Ein Keil ist kein Gerät, ein Buch tut es
auch.

**Die Camporini-Varianten** nach Bill Hartman stehen seit Build `2026-10-25 a` in der Leiter (Papier
P98). Das arbeitende Bein steht etwa einen halben Fuß zurückversetzt. „Gegenüber“ heißt: in der Hand
der Seite, deren Fuß vorn steht. Alle fünf Camporini-Sprossen tragen etwa dieselbe Hüftlast, die
Leiter ist an dieser Stelle flach. Sie sind deshalb eine Stufe: Je Seite gilt die, die du in *Dein
Workout* gewählt hast, Planer und Regeln wechseln nicht zwischen ihnen, und das Gewicht teilen sich
die fünf. Die vier Varianten gehören zu Premium. Ohne Premium stehen sie sichtbar, aber gesperrt in
der Auswahl, mit dem Zusatz „· im Medical Fitness System freigeschaltet“, und der Planer nimmt sie
nicht.

Zwei Dinge sind offen und stehen deshalb alle zehn Sprossen rot. **Erstens** liegen zwei
Abstände nach Rechnung unter dem Mindestabstand ×1,25: stehend mit Gewicht zum Camporini und
Camporini zum B-Stance. **Zweitens** misst die Waage Gewicht, nicht Hebel. Das Hüftmoment eines
Beins ist seine Bodenkraft mal dem Abstand vom Fuß zur Hüfte, und beim Hinge liegt der hintere
Fuß näher an der Hüfte. Im Modell trägt beim Camporini ab etwa 15 cm Fußabstand das vordere Bein
das größere Hüftmoment, obwohl das hintere mehr Gewicht trägt. Die Messung mit zwei Waagen
schreibt deshalb den Fußabstand mit. Die Haltetiefe muss in allen Sprossen gleich sein: 45° statt
60° Rumpfneigung sind schon ×1,22. Am einfachsten mit dem Rücken zur Wand, die Fersen eine feste
Zahl Fußlängen davor, das Gesäß knapp vor der Wand.

**Zusatzgewicht ist eine eigene Achse, keine Leitersprosse.** Die Leiter bleibt reines
Körpergewicht; Kilogramm kommen ab einer Stufe je Übung dazu (Liegestütz ab „Boden", Ausfallschritt
je nach Selbsttest, siehe oben, Fersenbrücke nur auf der obersten Sprosse „Langer Hebel 170° ·
Faszienrolle“). Der Seitstütz bleibt reines Körpergewicht,
seine Leiter endet bei „Standard". Wie stark ein Kilo wirkt, ist übungsabhängig: Bei Liegestütz
und Ausfallschritt wächst die Last mit `X/BW`. Beim Schrägzug sind es `1,3·X/BW` (Rucksack hoch auf den
Schulterblättern; rutscht er aufs Becken, wirkt dasselbe Gewicht nur zu drei Vierteln), bei der
Fersenbrücke `1,4·X/BW` (Gewicht auf dem Becken, dicht am Hüftgelenk). Bei der Hyperextension
kommt Gewicht nur auf der obersten Sprosse dazu, mit beiden Armen eng vor der Brust: `2,12·X/BW`,
bezogen auf diese Sprosse (30 cm Hebel vom Nabel bis zur Mitte des Brustbeins). Bis Build
`2026-10-23 d` lag Y darunter, und ohne Gewicht war die Sprosse leichter als Y. Die App begann
deshalb mit einem Mindestgewicht, 7,5 kg bei 80 kg Körpergewicht. Jetzt liegt „Hände in der Hüfte“
darunter, und die ist leichter als die Brust ohne Gewicht: Das Mindestgewicht ist 0 kg. Die Regel
bleibt stehen und greift wieder, sobald eine Messung die Brust leichter zeigt. Für die Last mit
Gewicht braucht die App das Körpergewicht: *Einstellungen → Messung*.
Beim Hip Hinge hängt das Gewicht in den Händen senkrecht unter den Schultern: `2,12·X/BW` (52 cm
von der Hüfte zur Schulter). Dort gibt es ein Mindestgewicht, denn „Stehend, Gewicht in beiden
Händen" ist ohne Gewicht dieselbe Stellung wie „Stehend, Körpergewicht": das kleinste Gewicht, mit
dem die Sprosse mindestens ×1,25 schwerer ist, 10 kg bei 80 kg Körpergewicht. Die fünf
Camporini-Sprossen teilen sich ein Gewicht, der B-Stance hat ein eigenes. Getragen wird es in der Hand
der Seite, deren Fuß hinten steht, bei den Varianten mit „gegenüber“ in der anderen.
Beim aufrechten Läufer trägt das Gewicht die ganze Leiter: Die einzige Sprosse heißt bei 0 kg
„Ohne Zusatzlast" und mit Kettlebell „Mit Zusatzgewicht". Wo die Kettlebell sitzt — Front Rack
`2,33·X/BW` oder Suitcase `3,22·X/BW` —, wählst du je Seite, denn links und rechts brauchen oft
einen anderen Carry. Die Kettlebell sitzt immer auf der Seite des freien Beins: Steht das
rechte Bein, heißen die Knöpfe „Front Rack · vor der Schulter – links" und „Suitcase · in der
Hand – links". Wechselst du, bleibt die Last gleich und die App rechnet das Kilo um.

---

## Was der Prototyp nicht kann

- **Die Lastfaktoren selbst lernen.** Das braucht mehrere hundert Nutzer und einen Server
  (Handoff 4.2). Ein einzelner Nutzer kann seine eigene Lastskala nicht bestimmen — bei drei
  Messungen und zwei Modellparametern bleibt dafür rechnerisch zu wenig übrig.
- **Zuverlässig bei gesperrtem Display weiterlaufen.** Eine Webseite darf das nicht. Genau
  darum geht es in Abschnitt 6.1 des Handoffs, und genau deshalb wird die echte App
  wahrscheinlich nativ.
- **Die eigene Satzkurve gegen den Durchschnitt aller Nutzer legen.** Das braucht einen Server
  und eine ausdrückliche Einwilligung. Der Export enthält die Kennzahlen dafür schon.
- Konten, Sync, Push, Videos, Bezahlung — alles außerhalb des MVP.

---

## Datensicherung

**Einstellungen → Sichern (Datei)** legt eine JSON-Datei ab. Das ist die einzige Sicherung, die es gibt:
Leerst du die Browserdaten oder wechselst das Gerät, sind die Messwerte weg. Vor jedem
Browser-Aufräumen einmal sichern.

---

*Kein Medizinprodukt. Keine Diagnose, keine Therapieempfehlung.
Prototyp vom 13.09.2026, installierbar seit 14.09.2026 · Luke Brandenburg*
