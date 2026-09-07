# Besprechung: Mechaniken zu einem Spiel zusammenführen

**Datum:** 2026-09-07
**Teilnehmer:** Denis, [Kumpel]
**Anlass:** Denis baut Mechaniken einzeln (je eigenes Unity-Projekt). Bevor es mehr
werden: klären, wie daraus *ein* Spiel wird und wie wir zusammenarbeiten.

---

## ⭐ Die wichtigsten Punkte (Kurzfassung)

1. **Ein Unity-Projekt statt vieler.** Mechaniken als Ordner + eigene Test-Szenen,
   ein `_Core` für Geteiltes, eine Integration-Szene. Getrennte Projekte = später
   Merge-Hölle.
2. **`_Core` zuerst festlegen — wer besitzt Player/Kamera/Input?** Für Multiplayer
   muss der Player evtl. ein Netzwerk-Objekt sein. Kumpel gibt die Player-Architektur
   vor, *bevor* Denis einen Wegwerf-Player baut.
3. **Multiplayer betrifft die Mechaniken direkt.** NPC-/Job-/Lager-Zustand muss
   synchronisiert werden. Welche Netcode-Lösung? Wer ist Autorität (Server)?
4. **Gemeinsames Git-Repo + Regeln.** Branch pro Person, gleiche Unity-Version,
   nicht beide gleichzeitig an derselben Szene, `.gitignore` + Meta-Dateien sauber.
5. **Gleiche Render-Pipeline + gleiches Input-System in allen Projekten.** Sonst
   pinke Materialien und kaputter Input beim Zusammenführen.
6. **Reihenfolge:** Denis macht erst die NPC-Mechanik fertig, *dann* wird
   konsolidiert — nicht vorher, nicht nebenbei.

---

## 1. Projektstruktur

**Empfehlung:** ein Projekt, so aufgebaut:

```
MeinSpiel/
  Assets/
    _Core/          EINMAL: Player, Kamera, Input, gemeinsame ScriptableObject-Typen, Tags/Layers
    Crafting/       NPC-Arbeitsmechanik (Scripts + Prefabs)
    Bauen/          Baumechanik (aus "Bauen V1")
    ...
    Scenes/
      Test_Crafting.unity     isoliertes Testen dieser Mechanik
      Test_Bauen.unity
      Integration.unity       alles zusammen -> "funktioniert das große Ganze?"
```

**Warum nicht mehrere Projekte:**
- GUID-/Referenzbruch beim Zusammenkopieren (Prefabs zeigen ins Leere, Materialien pink)
- Doppeltes Fundament driftet auseinander (schon passiert: `PlayerController` von
  "Bauen V1" liegt jetzt auch im NPC-Projekt)
- Projekt-Settings (Input, Pipeline, Layer, Physik) überall minimal anders
- "Alles zusammen testen" geht nur in *einem* Projekt

**Offen für die Besprechung:**
- Nehmen wir eins von Denis' bestehenden Projekten als Basis oder frisch aufsetzen?
- Später mal Mechaniken als lokale UPM-Packages? (jetzt Overkill)

## 2. `_Core` — Besitz und Verantwortung

- **Player:** Bewegung, Kamera, Interaktion. Für Multiplayer ggf. ganz anders
  (Netzwerk-Objekt, Client-Authority vs. Server). → **Kumpel gibt Struktur vor.**
- **Input:** ein gemeinsames Input-Actions-Asset in `_Core`. Alle Projekte stehen
  auf "Input System Package (New)".
- **Gemeinsame Daten-Typen:** z. B. `MaterialType`, Ressourcen, Item-Definitionen —
  gehören in `_Core`, damit Crafting und Bauen dieselben benutzen.
- **Tags/Layers:** früh gemeinsam festlegen (z. B. `Player`, `Interactable`).

## 3. Multiplayer-Auswirkungen (Input vom Kumpel nötig)

- **Welche Netcode-Lösung?** (Netcode for GameObjects / Mirror / FishNet / Photon …)
- **Autorität:** Wer besitzt den NPC-/Job-Zustand? Vermutlich der Server.
- **Zu synchronisieren bei der NPC-Mechanik:**
  - `CraftingJob`-Status (Phase, Fortschritt, Restzeit) — steht im Plan schon als
    offener Punkt
  - Lager-Bestände (shared state, mehrere Spieler greifen zu)
  - NPC-Position/Pfad (oder nur Ziel synchronisieren, Rest lokal?)
- **Konsequenz für Denis:** Mechaniken so bauen, dass die Zustandsänderung an *einer*
  klaren Stelle passiert (macht der Code aktuell schon: nur `NpcCraftWorker` schreibt
  den Job-Status).

## 4. Zusammenarbeit / Git

- **Ein gemeinsames Repo fürs Spiel** (getrennt vom CheatSheet).
- **Branch-Strategie:** `main` stabil, jede*r arbeitet auf `feature/...`-Branches,
  Merge über Pull Requests oder abgesprochen.
- **Gleiche Unity-Version exakt:** aktuell `6000.3.22f1`. Version-Wechsel nur gemeinsam.
- **`.gitignore` für Unity:** `Library/`, `Temp/`, `Logs/`, `obj/` raus; `Assets/`,
  `ProjectSettings/`, `Packages/` rein.
- **Meta-Dateien immer mitcommitten.** Sonst brechen Referenzen beim anderen.
- **Szenen & Prefabs sind YAML → schlecht mergebar.** Regel: nicht beide gleichzeitig
  an derselben Szene/Prefab. Optional "Smart Merge" (UnityYAMLMerge) in Git einrichten.
- **Wie tauschen wir aus?** Vermutlich über GitHub (wie beim CheatSheet).

## 5. Render Pipeline

- Alle Projekte auf **dieselbe Pipeline** (prüfen: URP oder Built-in?).
- Unterschiedliche Pipeline = pinke Materialien beim Zusammenführen, alles neu zuweisen.

## 6. Prefab-Konventionen

- **Alles als Prefab.** Verdrahtung innerhalb Objekt + Kinder steckt im Prefab.
- **Externe Szenen-Referenzen** (z. B. „welcher Player") nicht als serialisiertes
  Feld, sondern per Tag-Lookup (`FindWithTag`) oder Locator/Service.
- Namens- und Ordnerkonvention festlegen.

---

## Stand von Denis' Mechaniken (Kontext für den Kumpel)

- **NPCs_Arbeitsmechanik** (Unity 6000.3.22f1): generische Crafting-Mechanik.
  Spieler beauftragt an einer Station ein Item → NPC holt Material aus Lagern
  (NavMesh) → craftet über Zeit → Spieler holt ab. Datenmodell über ScriptableObjects
  (Material, Rezept, Stationslevel), Zustandsautomat am NPC. Läuft im Editor.
  Offen: 2 Weltraum-Anzeigen, Platzhalter-Animationen. Multiplayer bewusst noch nicht drin.
- **Bauen V1 Stein an Stein**: Baumechanik + erster Player-Controller (First Person,
  neues Input System).

---

## Von [Kumpel] (hier ergänzen)

-

## Ergebnisse / Beschlüsse (nach der Besprechung ausfüllen)

-

## To-dos

- [ ]
