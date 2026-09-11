# Offene Punkte und Querverbindungen — Übersicht über alles

**Diese Datei ist ein laufendes Dokument, keine einmalige Notiz.** Sie ist das
Gegenstück zu `MULTIPLAYER.md`:

| Datei | Frage, die sie beantwortet |
|---|---|
| `MULTIPLAYER.md` | Was bedeutet das für das **Netzwerk**? |
| **diese Datei** | Was ist **sonst** noch offen, und was hängt womit zusammen? |

Wer neu dazukommt (oder nach Wochen zurückkommt) soll hier in fünf Minuten sehen:
*Was gibt es schon? Was fehlt noch? Was muss später zusammengeführt werden?*

- **Letzte Aktualisierung:** 2026-09-11

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
| D2 | **Kampf und Perspektive:** Top-Down nur zum Bauen? Kampf zieht die Sicht zurück? Oder alles in allen Perspektiven? | Denis, wenn das Kampfsystem ansteht | "Alles in allen Perspektiven" wäre das Wunschbild, braucht aber ein **zweites Kampfsystem** (Zielen zum Mauszeiger, Höhenunterschiede, andere Sichtlinien). Im Kamera-Code liegt dafür nur ein Haken bereit (`bCombatAllowed`). |
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
| **Kampfsystem** | D2, Kamera-Haken `bCombatAllowed` | Noch kein Konzept vorhanden. |
| **Bauen in Unreal** | Kamera-Zeigermodus, Top-Down-Übersicht | Liegt als `Bauen V1` in Unity, nicht portiert. |
| **Netzwerk** | alles mit Zustand | Eigene Datei: `MULTIPLAYER.md`. |

---

## 5. Laufende Liste — was wann dazukam

*(Neueste oben. Format: Datum — Mechanik — was offen bleibt)*

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
