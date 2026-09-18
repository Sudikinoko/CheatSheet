# Steuerung — alle Tasten im Überblick

Projekt: `Workspaces\Unreal\NPCs` (Unreal Engine 5.8)

Diese Datei ist für Menschen, nicht für den Compiler. Wer mitspielt oder
mittestet, soll hier in einer Minute sehen, was welche Taste tut.

- **Letzte Aktualisierung:** 2026-09-18

---

## Bewegung

| Taste | Was passiert |
|---|---|
| **W A S D** | Laufen |
| **Maus** | Umsehen |
| **Leertaste** | Springen |

Diese vier kommen aus den Input-Mapping-Contexts (`IMC_Default`,
`IMC_MouseLook`), nicht aus dem C++-Code. Umbelegen geht im Editor an den
`IA_*`-Assets.

---

## Rohstoffe

| Taste | Was passiert |
|---|---|
| **E** *(halten)* | Erz abbauen. Loslassen bricht ab |

Der Rucksack hat ein Gewichtslimit. Ist er voll, muss erst abgeliefert werden —
dafür einfach in die Basis laufen, das passiert von allein.

---

## Bauen

| Taste | Was passiert |
|---|---|
| **V** | Baumodus an und aus |
| **Tab** | Nächster Bauplan |
| **Q** | Voriger Bauplan |
| **Linke Maustaste** | Gebäude setzen |
| **Esc** | Baumodus abbrechen |

Im Baumodus schwebt eine Vorschau an der Zielstelle, dazu ein Kasten:
**grün heißt baubar, rot nicht**. Warum es rot ist, steht im Output Log unter
`LogBauen` — zu steil, zu weit, belegt, Material fehlt, oder außerhalb des
begehbaren Geländes.

Bezahlt wird aus den **Lagern**, nicht aus dem Rucksack.

Außerhalb des Baumodus tut die linke Maustaste nichts — sie bleibt für anderes
frei.

---

## Werkstation

| Taste | Was passiert |
|---|---|
| **B** | Bestellen |
| **P** | Fertiges abholen |
| **U** | Station ausbauen (kostet Material) |

Wirkt nur in Reichweite einer Station.

---

## Laden (Lean Shop)

| Taste | Was passiert |
|---|---|
| **G** | Kaufen |

Der Shop ist für **Komfort-Sachen** wie den Kamera-Roboter. Verteidigung gibt es
dort bewusst nicht — Geschütze baut man selbst, siehe *Bauen*.

---

## Kamera-Roboter

| Taste | Was passiert |
|---|---|
| **K** *(halten)* | Einstellmodus: Perspektive wechseln |
| **Mausrad** *(bei gehaltenem K)* | Zwischen Ego, Schulter und Draufsicht fahren |
| **1 2 3 4** *(bei gehaltenem K)* | Lieblingssicht abrufen, bzw. speichern |

Muss erst im Laden gekauft werden. Vorher passiert bei K nichts.

---

## Für Entwickler: wo die Tasten wohnen

Jede Mechanik bringt ihre Tasten selbst mit, als Eigenschaft einer Komponente.
Alle sind im Editor umbelegbar, ohne den Code anzufassen.

| Mechanik | Datei |
|---|---|
| Bauen | `Source\NPCs\Bauen\BuildPlayerInputComponent.h` |
| Werkstation | `Source\NPCs\Crafting\StationPlayerInputComponent.h` |
| Laden | `Source\NPCs\Shop\ShopPlayerInputComponent.h` |
| Abbauen | `Source\NPCs\Rohstoffe\MiningComponent.h` |
| Kamera-Roboter | `Source\NPCs\Kamera\CameraRobotComponent.h` |
| Bewegung, Umsehen, Springen | Input-Mapping-Contexts im Content, nicht im Code |

### Vorsicht bei neuen Tasten

**Unreal warnt nicht vor Doppelbelegungen.** Hängen zwei Mechaniken auf
derselben Taste am selben Spieler, feuern beim Druck **beide** — die Engine
arbeitet alle Bindungen einer Input-Komponente ab und markiert die Taste erst
danach als verbraucht (`PlayerInput.cpp`, Zeile 1804).

In diesem Projekt ist das schon dreimal passiert:

- **B** war für den Baumodus gedacht und gehört der Bestellung an der Station
- **Mausrad** kollidiert mit dem Zoom des Kamera-Roboters
- **F1–F4** sind Unreals eingebaute Ansichts-Umschalter (`viewmode wireframe`
  und Geschwister, `Engine\Config\BaseInput.ini` Zeile 39-42) — aktiv in allen
  Nicht-Shipping-Builds, also genau beim Testen im Editor

Deshalb: **vor einer neuen Taste erst in diese Datei sehen**, dann in
`BaseInput.ini`. Die Bau-Eingabe meldet eine Doppelbelegung inzwischen selbst im
Log (`LogBauen`), aber nur für ihre eigenen Tasten.

Belegt sind derzeit: **W A S D, Leertaste, Maus, Mausrad, E, B, P, U, G, K, V,
Q, Tab, linke Maustaste, Esc, 1 2 3 4**.
