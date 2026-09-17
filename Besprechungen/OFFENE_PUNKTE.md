# Offene Punkte und Querverbindungen — Übersicht über alles

**Diese Datei ist ein laufendes Dokument, keine einmalige Notiz.** Sie ist das
Gegenstück zu `MULTIPLAYER.md`:

| Datei | Frage, die sie beantwortet |
|---|---|
| `MULTIPLAYER.md` | Was bedeutet das für das **Netzwerk**? |
| **diese Datei** | Was ist **sonst** noch offen, und was hängt womit zusammen? |

Wer neu dazukommt (oder nach Wochen zurückkommt) soll hier in fünf Minuten sehen:
*Was gibt es schon? Was fehlt noch? Was muss später zusammengeführt werden?*

- **Letzte Aktualisierung:** 2026-09-12

---

## 1. Regel für alles, was ab jetzt gebaut wird

> **Jede neue Mechanik bekommt einen Eintrag in Abschnitt 5** — auch wenn nichts
> offen bleibt. Lieber ein "nichts offen" zu viel als eine vergessene Verbindung.

Drei Fragen pro neuem Feature:

1. **Braucht es etwas, das es noch nicht gibt?** (Speichern, Menü, Währung …)
   → Abschnitt 4
2. **Berührt es eine andere Mechanik?** → Abschnitt 3
3. **Ist eine Design-Frage offen geblieben?** → Abschnitt 2

---

## 1b. Leitsatz fürs Balancing (entschieden 13.09.)

> **Eine Runde soll allein ungefähr so lang dauern wie zu fünft.**

Kein Spielinhalt soll dadurch verschwinden, dass mehr Leute mitspielen. Fünf
Spieler sollen dasselbe Spiel erleben wie einer — nicht dasselbe Spiel in einem
Fünftel der Zeit.

**Was daraus folgt, und das ist der unbequeme Teil:** Eine feste Zahl kann immer
nur für **eine** Spielerzahl richtig sein. `Vorrat = 50` ist entweder für einen
Spieler passend oder für fünf, nie für beide. Mehr Stellschrauben lösen das
nicht — es braucht **Regeln**, also Werte, die sich aus der Spielerzahl ergeben.

Umgesetzt ist das bisher an zwei Stellen, beide mit einem einstellbaren Faktor
(`1.0` = voll skalieren, `0.0` = abgeschaltet):

| Wo | Feld | Warum |
|---|---|---|
| `AOreNode` | `AmountScalePerExtraPlayer` | Der Erzvorrat ist Spielinhalt. Fest wäre er zu fünft in einem Fünftel der Zeit leer. |
| `ARaidDirector` | `MaxAliveScalePerExtraPlayer` | Eine feste Decke von 20 heißt 20 Gegner für einen — oder vier pro Kopf zu fünft. Mehr Leute machten es also **leichter**. |

Die Bedrohung selbst skaliert schon von allein: sie ist **eine** Zahl für alle,
steigt zu fünft also fünfmal so schnell, und die Wellen eskalieren entsprechend.
Das war halb Absicht, halb Glück — bitte nicht "aufräumen".

Gezählt werden nur Spieler **mit Pawn**, an genau einer Stelle
(`Source\NPCs\PlayerCount.h`). Wer auf seine Wiederbelebung wartet, treibt die
Zahlen nicht hoch.

**Gemessen wird am Ende jedes Überfalls** — eine Zeile im Log mit Dauer, Erz,
Wellen, Angreifern und Spielerzahl. Ohne Messung ist Balancing Raten: rund
fünfzehn Werte wirken auf dieselbe Frage, und man merkt sonst nur "fühlt sich
anders an".

*Noch nicht erfasst:* Spielertode. Wäre die nächste sinnvolle Zahl.

---

## 2. Offene Entscheidungen

Bewusst noch nicht entschieden. Nicht in eine Richtung vorbauen, bevor das geklärt ist.

| # | Frage | Wer / wann | Warum sie wartet |
|---|---|---|---|
| D1 | **Projektstruktur:** ein gemeinsames Projekt oder getrennte pro Mechanik? | Denis + Kumpel | Betrifft den Kumpel direkt (Multiplayer). Empfehlung der KI: **ein** Projekt, Mechaniken als Ordner, je eine Testszene, ein `_Core` für Geteiltes. |
| ~~D2~~ | ~~**Kampf und Perspektive:** Top-Down nur zum Bauen?~~ | **Neu entschieden 16.09.** | **Gekämpft wird in ALLEN Sichten** — Ego, Third-Person und Top-Down. Denis hat die Fassung vom 12.09. („Kampf zieht die Sicht zurück") ausdrücklich verworfen, mit zwei Gründen: es gibt **noch gar kein Spieler-Kampfsystem**, und eine Sicht zu sperren, in der später gekämpft werden soll, ist verkehrt herum gedacht. Alle Anker stehen deshalb auf `bCombatAllowed = true`; `SetCombatActive()` wird vom `ARaidDirector` weiter gerufen, bewirkt aber nichts. Die Mechanik bleibt für den Fall stehen, dass sich eine einzelne Sicht später als untauglich erweist — dann genügt ein `false` an dem einen Anker. Das Top-Down-Zielen bleibt eine Frage **an das Kampfsystem**, wenn es gebaut wird, nicht an die Kamera. |
| D3 | **Währung:** bleibt es beim Bezahlen mit Material aus den Lagern, oder kommt eine eigene Währung? | offen | Betrifft nur die Bezahlstelle (`TryBuy`, `TryUpgrade`). Die Kostenlisten bleiben in beiden Fällen gleich. |
| D4 | **Guide aus "Nimm mich mit":** feste Figur in der Welt oder Menü/Overlay? | offen | Siehe `Konzepte\NimmMichMit_Konzept.md`. |
| D5 | **Multiplayer-Grundsatzfragen** (Dedicated/Listen Server, Spielerzahl, Job als UObject oder Struct) | Kumpel | Stehen ausführlich in `MULTIPLAYER.md`, Abschnitt 2. |
| D6 | **Tödlichkeit: entscheidet die Trefferzone statt des Lebensbalkens?** Kopfschuss — oder ein anderes tödliches Organ — tötet sofort, statt zehnmal auf einen Kugelschwamm zu schießen. | Denis — **bevor die Spielerwaffe gebaut wird** | Wunsch von Denis, 13.09. Die Entscheidung ändert `UHealthComponent::ApplyDamage` (heute **zwei** Aufrufer — jetzt billig, mit jeder weiteren Waffe teurer) und die Kollisionseinrichtung **jedes** Ziels. Ausführlich in Abschnitt 5, Eintrag vom 13.09. Fürs Geschütz (Meilenstein 8–11) ändert sie **nichts** — das zielt auf den Rumpf. |

---

## 3. Querverbindungen — was hängt womit zusammen

Das Wichtigste dieser Datei: Stellen, an denen zwei Mechaniken sich später treffen.

| Von | Nach | Was passieren muss |
|---|---|---|
| **Kamera (Top-Down)** | **Bauen** | Der Zeigermodus liefert einen sichtbaren Mauszeiger und kamerarelative Bewegung — das Bauen selbst liegt noch in `Unity\Bauen V1` und ist nicht portiert. Der Zeiger hat aktuell **nichts zum Anklicken**. |
| **Kamera (Hinweis)** | **"Nimm mich mit"** | `UPlayerHintComponent::ShouldShow(Thema)` wird heute von einem Zähler beantwortet. Sobald das Tagebuch (Schicht A) existiert, beantwortet es dieselbe Funktion — **eine Funktion umstellen**, sonst nichts. Siehe `NimmMichMit_Konzept.md`, "Erster Anwendungsfall". |
| **Lean Shop** | **alles Käufliche** | Der Shop ist von Anfang an als **Liste** von Angeboten gebaut. Neue Ware = ein neues Data Asset, kein Code. Der Kamera-Roboter ist das erste Angebot. |
| **Freischaltungen** | **Speichern (Welt)** | `UUnlockComponent` hält, was der Spieler besitzt. Ein Kauf ist nach dem Neustart weg — und das bleibt vorerst **mit Absicht so**: bezahlt wird aus den Lagern, und die Lagerbestände speichert niemand. Käme der Kauf ins Spieler-Profil, hätte man nach einem Neustart das Material **und** die Ware. Der Kauf gehört deshalb ins Weltspeichern, zusammen mit den Beständen. |
| **Lieblingssichten + Hinweis-Zähler** | ~~Speichern~~ **erledigt 15.09.** | Liegen jetzt im Spieler-Profil (`UPlayerProfileSubsystem`, Datei `Saved/SaveGames/SpielerProfil.sav`). Geschrieben wird sofort bei jeder Änderung. **Das Profil nimmt bewusst nur Einstellungen auf, keinen Weltzustand** — die Begründung steht in `PlayerProfileSave.h`. |
| **Lagerzugriff** | **Basis-Zone / Missionen** | `UStorageAccessComponent` beantwortet allein "habe ich Zugriff aufs Lagernetz?". Dort kommt später die Basis-Zone hinein — Anzeige und Handwerkslogik bleiben unangetastet. |
| **Gebäudeschilder** | **Optionsmenü** | Die Schilder kippen jetzt zur Kamera, damit sie auch von oben lesbar sind. Ein- und ausschalten soll später im Menü möglich sein. |
| **Alle Spieler-Tasten** | **Enhanced Input** | B / P / U / G / K / E / 1-4 sind **direkt gebunden** (Platzhalter, ohne Input-Assets). Die Umstellung auf Input-Actions samt Gamepad und Umbelegen lohnt sich **einmal für alle Tasten gemeinsam**. |
| **Alle Spieler-Tasten** | **Respawn / Pawn-Wechsel** | **Im Spieltest am 13.09. gefunden, und es ist ein echter Blocker fürs Netzwerk:** Alle Bedienungen binden ihre Tasten in `BeginPlay`. Bekommt der Spieler einen **neuen Pawn** (echter Respawn über `GameMode::RestartPlayer`), ist der `PlayerController` zu dem Zeitpunkt noch nicht da — im Log: *"kein PlayerController - keine Tasten gebunden"*. Danach war **keine einzige Taste** mehr belegt: nicht abbauen, nicht bestellen, nicht abholen, nicht ausbauen, nicht kaufen, nicht die Kamera umschalten. Der Respawn steht deshalb wieder auf **Versetzen** (derselbe Pawn wird geheilt und versetzt). Für Multiplayer wird ein echter Pawn-Wechsel gebraucht — **also muss die Input-Umstellung davor passieren, nicht danach.** |
| **Trefferzonen (D6)** | **Spielerwaffe** | Die Zonenfrage wird erst mit der Waffe real — das Geschütz zielt auf die Mitte der Bounding Box, also den Rumpf. Entschieden sein muss sie aber **vorher**, sonst wird `ApplyDamage` zweimal umgebaut. |
| **Trefferzonen (D6)** | **Kollision an allen Zielen** | Ein Trefferzonen-System braucht einen **eigenen Trace-Kanal**: die Character-Kapsel muss ihn ignorieren, das Skeletal Mesh ihn blocken. Das betrifft `BP_Angreifer`, `BP_CraftPlayer`, `BP_Geschuetz` und später jedes Gebäude — nicht nur eine Klasse. |

---

## 4. Fundamente, die noch fehlen

Dinge, auf die mehrere Mechaniken warten. Reihenfolge ist bewusst keine gesetzt.

| Fundament | Wer wartet darauf | Anmerkung |
|---|---|---|
| **Speichersystem** | Freischaltungen, Lieblingssichten, Hinweis-Zähler, "Nimm mich mit" (Schicht A), Spielstand allgemein | Hängt an der Spieler-Identität → hängt an D1 und am Multiplayer. Deshalb **nicht** als Einzelspieler-Lösung vorbauen. |
| **Optionsmenü** | Gebäudeschilder ein/aus, später Tastenbelegung, Empfindlichkeiten | Gibt es noch gar nicht. |
| ~~**Kampfsystem**~~ | — | **Nicht mehr offen.** Konzept und Fundament stehen: `Unreal\NPCs\plan-rohstoff-raid-unreal.md`. Gebaut sind Lebenspunkte, Seiten und **eine** Schadensfunktion. Offen bleiben Spielerwaffe und Geschütz — beides steckt im selben Plan. |
| **Bauen in Unreal** | Kamera-Zeigermodus, Top-Down-Übersicht | Liegt als `Bauen V1` in Unity, nicht portiert. |
| **Netzwerk** | alles mit Zustand | Eigene Datei: `MULTIPLAYER.md`. |

---

## 5. Laufende Liste — was wann dazukam

*(Neueste oben. Format: Datum — Mechanik — was offen bleibt)*

### 2026-09-17 — Wandvermeidung an die Tastrichtung angepasst

Die Untergrenze, die verhindert, dass die Kamera beim Ausweichen in den Spieler
rutscht, rechnete immer mit dem Kapsel**radius** (34). Seit die Kamera ab „Hoch"
senkrecht über dem Spieler steht, ist dort aber die **Halbhöhe** (96) das Maß —
die Grenze lag 62 cm zu tief. Jetzt wird zwischen beiden nach der Tastrichtung
gemischt. Betrifft auch die schrägen Anker: Third-Person 46 → 64, Mittel 46 → 80.

**Offen, bewusst noch nicht gebaut — erst spielen, dann entscheiden:** Alle vier
Weitsichten stehen seit dem 16.09. auf **derselben senkrechten Linie**. Stellt
sich der Spieler unter ein Dach, trifft der Suchstrahl dieses Dach bei allen
vieren an praktisch derselben Stelle — „Hoch", „Top-Down", „Übersicht" und
„Fernsicht" klemmen dann auf dieselbe Höhe, und die Höhenleiter
(850/1400/2200/3500) ist drinnen nicht mehr unterscheidbar. Vorher konnte das
nicht passieren, weil „Hoch" schräg hinter dem Spieler stand.

Im Level `Lvl_Wiese` gibt es derzeit nichts, unter das man sich stellen kann —
der Fehler ist also **nicht nachweisbar**, und nach der Projektregel „erst
messen, dann ändern" wird er deshalb nicht auf Verdacht behoben. Sobald es
Gebäude mit Dach gibt: nachsehen, ob es stört. Denkbare Lösung wäre, dass die
Weitsichten Dächer über dem Spieler beim Ausweichen ignorieren.

### 2026-09-16 — Kamera: siebter Anker, mittige Sicht, Fadenkreuz, Kampf überall

**Siebter Anker „Fernsicht"** (`t = 6`, 3500 cm). **Ab „Hoch" (`t = 3`) steht der
Spieler mittig im Bild** — die Kamera steht dort senkrecht über ihm und schaut
gerade hinunter. Die vier äußeren Anker unterscheiden sich nur noch in der Höhe
(850, 1400, 2200, 3500). Top-Down hat dafür seinen Versatz nach vorn verloren,
also die freie Baufläche im unteren Bilddrittel — bewusst, auf Denis' Wunsch.

**„Mittel" (`t = 2`) ist jetzt Blickmodus statt Zeigermodus**, mit
`PitchFollow = 0.5` und erlaubtem Kampf. Damit reicht die spielbare Sicht eine
Stufe weiter hinaus: Maus dreht den Blick, Fadenkreuz da, kein Cursor. Der
Zeigermodus beginnt erst zwischen „Mittel" und „Hoch" (`t ≈ 2,5`).

**Das Fadenkreuz hängt wieder am Steuerungsmodus**, nicht an einer eigenen
Grenze. Eine zweite Stelle, die „bis hierhin kann man zielen" festlegt, wäre
früher oder später auseinandergelaufen. Wer es länger sichtbar haben will,
stellt den betreffenden Anker auf Blickmodus.

**Offen / verbunden:** Der Übergang „Mittel" → „Hoch" kippt jetzt in einem Zug
von −20° auf −89° und fühlt sich laut Denis „ein bisschen komisch" an — bewusst
so gelassen, bis jemandem etwas Besseres einfällt. Der naheliegende Hebel wäre
ein steilerer Winkel bei „Mittel", damit der Sprung kleiner wird. Siehe auch
**D2** oben: gekämpft wird jetzt in allen Sichten.

### 2026-09-15 — Spieler-Profil + sechster Kamera-Anker

**Spieler-Profil** (`Source/NPCs/Profil/`): Lieblingssichten und Hinweis-Zähler
überleben jetzt das Spielende. `UPlayerProfileSubsystem` besitzt die Datei,
`UPlayerProfileSave` ist der Behälter. Geschrieben wird **sofort bei jeder
Änderung**, nicht beim Beenden — im Editor endet ein Spiel mit Stop, und ein
Absturz kündigt sich nicht an.

**Sechster Anker "Uebersicht"** (`t = 5`): höher als Top-Down und genau über dem
Spieler, damit er mittig im Bild sitzt. Bricht bewusst die 13-Grad-Regel der
anderen weiten Anker — Top-Down bleibt die Bau-Sicht, die Übersicht ist der Blick
aufs Ganze.

**Offen / verbunden:** Der **Kauf** bleibt draußen (siehe Abschnitt oben,
Dublier-Lücke über die Lagerbestände). Taste **4** springt weiterhin auf Top-Down,
nicht auf die neue Übersicht — bewusst nicht ungefragt umgestellt. Die Anker-Werte
(Höhe 2200) sind ein erster Vorschlag und im Details-Panel veränderbar; **Achtung:**
wer das Anker-Array im Blueprint anfasst, erbt danach nicht mehr vom C++-Code.

### 2026-09-13 — Design-Frage aufgemacht: Trefferzonen statt Kugelschwamm (D6)
Denis will kein Spiel, in dem man tausend Schuss in einen Gegner leert. **Ein
Kopfschuss soll töten**, ebenso ein Treffer auf ein anderes tödliches Organ. Zäh
soll es nur werden, wenn man die **falsche Waffe** benutzt.

Davon stand bisher **nirgends** etwas — weder in den Plänen noch hier. Das ist
ein neuer Gedanke, kein vergessener. Deshalb als **D6** aufgenommen und
**nicht** gebaut.

**Was schon in die richtige Richtung gebaut ist:**

- **K5 hat vorgesorgt.** Es gibt genau eine Schadensfunktion,
  `UHealthComponent::ApplyDamage(Menge, Verursacher)`. Alles Tödliche geht durch
  diese eine Stelle — Geschütz, Angreifer, später die Spielerwaffe.
- **Der Schuss ist bereits ein Line Trace** (`UTurretComponent`, Feuern). Ein
  `FHitResult` enthält von Haus aus `BoneName` — die Information *„was wurde
  getroffen"* liegt also schon vor und wird heute nur weggeworfen.

**Was fehlt — drei Dinge, und keins davon geht von allein:**

1. **Der Strahl trifft heute die Kapsel, nicht den Körper.** Bei einem
   Unreal-Character blockt die Collision-Kapsel den Kanal `Visibility`; der Trace
   endet an einem Zylinder um den ganzen NPC und `BoneName` bleibt leer. Es
   braucht einen **eigenen Trace-Kanal**, den die Kapsel ignoriert und das
   Skeletal Mesh blockt.
2. **Jedes Ziel braucht ein brauchbares Physics Asset**, dessen Bodies den Zonen
   entsprechen. Beim Geschütz ist das geplant (fünf Boxen, `T01_Geschuetz_Import.md`),
   bei den NPCs bisher überhaupt nicht bedacht.
3. **`ApplyDamage` braucht einen Parameter mehr** — Zone oder fertiger
   Multiplikator. Genau der Umbau, den K5 vermeiden wollte. **Heute mit zwei
   Aufrufern billig**, nach der Spielerwaffe teuer.

**Was Denis noch entscheiden muss:**

- **Gilt es auch für ihn selbst?** Wenn ein Kopfschuss tötet, tötet er auch den
  Spieler. Und: bekommen **Nahkampf**treffer Zonen, oder nur Schüsse? Die
  Angreifer schlagen heute zu, sie schießen nicht.
- **Haben die Angreifer überhaupt Organe?** „Tödliches Organ" heißt bei einer
  Maschine eher **Schwachstelle** — Kern, Kühler, Sensor. Das ist eine
  Entscheidung über die Gegner, nicht über den Code.
- **Bleiben Lebenspunkte?** Vermutlich ja: für Treffer außerhalb der Zonen und
  für Gebäude, die gar keine Zonen haben.
- **„Falsche Waffe"** setzt Waffen- **und** Panzerungsarten voraus. Eigenes
  Thema, **nicht** zusammen mit D6 vorbauen.

**Berührt Abschnitt 1b:** Tödlichkeit ist Balancing. Heute hat ein Angreifer
50 LP und das Geschütz 12 Schaden pro Schuss — mit einer Sofort-tot-Zone
verschiebt sich dieses Verhältnis komplett.

**Netzwerkseite:** siehe `MULTIPLAYER.md`, Eintrag vom 13.09. (**N-10**). Kurz:
solange nur das Geschütz schießt, harmlos; mit einer Spielerwaffe wird daraus
Lag Compensation.

### 2026-09-13 — Erster vollständiger Spieltest des Kreislaufs
Der ganze Ablauf ist im Spiel bestätigt: abbauen → Bedrohung steigt → Wellen →
Angreifer erreicht und schlägt → Spieler stirbt → Erz fällt als Bündel → ein
Angreifer hebt es auf und trägt es vom Feld → Erz verloren. Dazu Ablieferung,
Überfall-Ende und die Auswertungszeile.

Schön zu sehen: **drei** Angreifer rannten gleichzeitig zum selben Bündel,
**einer** bekam es, die anderen gingen sauber auf *Untätig*. Kein doppeltes Erz.

**Im Test gefunden und behoben:** Angreifer blieben stehen, wenn die Wegfindung
sie vor dem Ziel absetzte und der Spieler stillstand (kein Grund zum Nachsteuern);
die Ablieferung nahm das *nächste* Lager statt des nächsten, das das Material
auch annimmt; und das oben beschriebene Tasten-Problem beim Pawn-Wechsel.

**Noch ungetestet:** das Zurückerobern eines Bündels durch den Spieler, und
alles rund ums Geschütz (noch nicht importiert).

### 2026-09-12 — Rohstoffe, Kampf und Überfall (Meilenstein 1–7)
Plan: `Unreal\NPCs\plan-rohstoff-raid-unreal.md`, Einrichtung:
`Unreal\NPCs\Rohstoff_Raid_Einrichtung.md`. Erz abbauen erzeugt Bedrohung,
Angreifer wollen es zurück, das Geschütz (noch nicht gebaut) verteidigt.

**Das beantwortet D2** — *Kampf und Perspektive*: die **einfache Variante** ist
gewählt (Denis, 12.09.). Der Kampf zieht die Sicht auf Third-Person zurück, die
Zeiger-Sichten bleiben fürs Bauen. `UCameraRobotComponent::SetCombatActive()`
hat damit einen Aufrufer — den `ARaidDirector`. Ein echtes Top-Down-Kampfsystem
kommt später als eigenes Thema.

**Das füllt außerdem das Fundament "Kampfsystem"** aus Abschnitt 4, das bisher
mit *"noch kein Konzept vorhanden"* geführt wurde.

**Offen / verbunden:**

- **Freischaltungen am Pawn** — der gekaufte Kamera-Roboter geht beim Respawn
  verloren. Es gibt eine Überbrückung, die es verdeckt. Der eigentliche Punkt
  steht als **N-1** in `MULTIPLAYER.md` und ist dort auch die Lösung
  (`APlayerState`). **Bitte vor dem Netzwerkbau ansehen — es ist schon jetzt
  falsch, nicht erst später.**
- **Wächst der Erzbrocken nach?** Heute ist er irgendwann leer. Nachwachsen,
  mehrere Brocken oder ein wanderndes Vorkommen sind alles gute Antworten — es
  hängt daran, wie lang eine Partie werden soll. Der Code ist darauf vorbereitet:
  später entstehende Brocken melden sich über `AOreNode::OnAnyNodeRegistered`.
- **Der Spieler kann sich nicht wehren.** Ausdrücklich für später gewollt. Der
  Bau ist vorbereitet: eine Spielerwaffe ruft dieselbe `ApplyDamage` und braucht
  am Rest nichts zu ändern.
- **Gebäude sind nicht zerstörbar** (Meilenstein 13, optional). Billiger als
  gedacht — der Handwerks-Code verkraftet ein verschwindendes Lager bereits.
  Was fehlt, ist Zielwechsel-Logik beim Angreifer, nicht die Lebenspunkte.
- **Taste E reiht sich in die Platzhalter ein** (halten = abbauen, antippen =
  Bündel aufheben). Gehört zur gemeinsamen Umstellung auf Enhanced Input weiter
  oben — und ist der **erste Fall, der Halten braucht**.
- **Speichern fehlt** — betrifft jetzt auch das später gekaufte Geschütz, genau
  wie den Kamera-Roboter.
- **Der Bewegungs-Controller heißt noch `ANpcCraftWorkerController`**, obwohl ihn
  inzwischen Handwerks-NPC *und* Angreifer benutzen (über `INpcMoveListener`).
  Umbenennen ist ein eigener Schritt: es könnte Blueprint-Verweise abreißen, und
  eine Suche in binären `.uasset` ist nicht beweisend.
- **Balancing ist ungetestet.** Reichweiten, Schaden, Wellengröße, Abbautempo
  stehen auf Startwerten. Das entscheidet sich beim Spielen, nicht am Plan.

### 2026-09-11 — Kamera-Roboter / Perspektivwechsel + Lean Shop
Plan: `Unreal\NPCs\plan-kamera-roboter-unreal.md`. Perspektivwechsel Ego /
Third-Person / Top-Down über Ankerliste und einen durchlaufenden Wert `t`, gekauft im
**Lean Shop**. Wichtig: es ist **kein reiner Kamera-Wechsel** — in Top-Down wechselt
auch die Steuerung (Bewegung relativ zur Kamera, sichtbarer Mauszeiger).

**Offen / verbunden:** D2 (Kampf), Zeiger hat nichts zum Anklicken (Bauen fehlt),
Speichern fehlt für Kauf und Lieblingssichten, Hinweis wartet auf "Nimm mich mit",
Schilder warten aufs Optionsmenü.

### 2026-09-11 — Diese Datei angelegt
Ausgangsstand aus `MULTIPLAYER.md`, den Projekt-Plänen und den Konzeptdateien
zusammengetragen.
