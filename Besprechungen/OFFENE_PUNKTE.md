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

## 2. Offene Entscheidungen

Bewusst noch nicht entschieden. Nicht in eine Richtung vorbauen, bevor das geklärt ist.

| # | Frage | Wer / wann | Warum sie wartet |
|---|---|---|---|
| D1 | **Projektstruktur:** ein gemeinsames Projekt oder getrennte pro Mechanik? | Denis + Kumpel | Betrifft den Kumpel direkt (Multiplayer). Empfehlung der KI: **ein** Projekt, Mechaniken als Ordner, je eine Testszene, ein `_Core` für Geteiltes. |
| ~~D2~~ | ~~**Kampf und Perspektive:** Top-Down nur zum Bauen?~~ | **Entschieden 12.09.** | **Einfache Variante:** der Kampf zieht die Sicht auf Third-Person zurück, die Zeiger-Sichten bleiben fürs Bauen. `SetCombatActive()` wird vom `ARaidDirector` gerufen. Ein echtes Top-Down-Kampfsystem (Zielen zum Mauszeiger, Höhenunterschiede) kommt später als **eigenes Thema** — dann wird D2 wieder aufgemacht. |
| D3 | **Währung:** bleibt es beim Bezahlen mit Material aus den Lagern, oder kommt eine eigene Währung? | offen | Betrifft nur die Bezahlstelle (`TryBuy`, `TryUpgrade`). Die Kostenlisten bleiben in beiden Fällen gleich. |
| D4 | **Guide aus "Nimm mich mit":** feste Figur in der Welt oder Menü/Overlay? | offen | Siehe `Konzepte\NimmMichMit_Konzept.md`. |
| D5 | **Multiplayer-Grundsatzfragen** (Dedicated/Listen Server, Spielerzahl, Job als UObject oder Struct) | Kumpel | Stehen ausführlich in `MULTIPLAYER.md`, Abschnitt 2. |

---

## 3. Querverbindungen — was hängt womit zusammen

Das Wichtigste dieser Datei: Stellen, an denen zwei Mechaniken sich später treffen.

| Von | Nach | Was passieren muss |
|---|---|---|
| **Kamera (Top-Down)** | **Bauen** | Der Zeigermodus liefert einen sichtbaren Mauszeiger und kamerarelative Bewegung — das Bauen selbst liegt noch in `Unity\Bauen V1` und ist nicht portiert. Der Zeiger hat aktuell **nichts zum Anklicken**. |
| **Kamera (Hinweis)** | **"Nimm mich mit"** | `UPlayerHintComponent::ShouldShow(Thema)` wird heute von einem Zähler beantwortet. Sobald das Tagebuch (Schicht A) existiert, beantwortet es dieselbe Funktion — **eine Funktion umstellen**, sonst nichts. Siehe `NimmMichMit_Konzept.md`, "Erster Anwendungsfall". |
| **Lean Shop** | **alles Käufliche** | Der Shop ist von Anfang an als **Liste** von Angeboten gebaut. Neue Ware = ein neues Data Asset, kein Code. Der Kamera-Roboter ist das erste Angebot. |
| **Freischaltungen** | **Speichern** | `UUnlockComponent` hält, was der Spieler besitzt. Ohne Speichersystem ist ein Kauf nach dem Neustart weg. |
| **Lieblingssichten + Hinweis-Zähler** | **Speichern** | Persönliche Einstellungen des Spielers, kein Weltzustand. Gehören ins Spieler-Profil, nicht in den Levelstand. |
| **Lagerzugriff** | **Basis-Zone / Missionen** | `UStorageAccessComponent` beantwortet allein "habe ich Zugriff aufs Lagernetz?". Dort kommt später die Basis-Zone hinein — Anzeige und Handwerkslogik bleiben unangetastet. |
| **Gebäudeschilder** | **Optionsmenü** | Die Schilder kippen jetzt zur Kamera, damit sie auch von oben lesbar sind. Ein- und ausschalten soll später im Menü möglich sein. |
| **Alle Spieler-Tasten** | **Enhanced Input** | B / P / U / G / K / 1-4 sind **direkt gebunden** (Platzhalter, ohne Input-Assets). Die Umstellung auf Input-Actions samt Gamepad und Umbelegen lohnt sich **einmal für alle Tasten gemeinsam**. |

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
