# Besprechung: Mechaniken zu einem Spiel zusammenführen

**Ursprünglich:** 2026-09-07 (Unity-Stand)
**Auf Unreal umgeschrieben:** 2026-09-18 — nach dem Engine-Wechsel zu UE5.
**Teilnehmer:** Denis, [Kumpel]
**Anlass:** Denis baut Mechaniken einzeln. Bevor es mehr werden: klären, wie daraus
*ein* Spiel wird und wie wir zusammenarbeiten.

> **Hinweis zum Umschreiben:** Die Struktur-Empfehlungen sind von Unity nach Unreal
> übersetzt. Was am 7.9. offen war, ist weiterhin offen — jetzt eben als
> Unreal-Frage. Es wurde nichts neu entschieden.

---

## ⭐ Die wichtigsten Punkte (Kurzfassung)

1. **Ein Unreal-Projekt statt vieler.** Mechaniken als Ordner unter `Source/NPCs/`
   + eigene Test-Level, ein gemeinsamer Kern für Geteiltes, ein Integration-Level.
   **Das ist bereits so umgesetzt** — siehe Abschnitt 1.
2. **Kern zuerst festlegen — wer besitzt Character/Kamera/Input?** Für Multiplayer
   muss der Character ein replizierter Actor sein. Kumpel gibt die Character-
   Architektur vor, *bevor* Denis darauf aufbaut. Aktuell steht alles auf dem
   **First-Person-Template**.
3. **Multiplayer betrifft die Mechaniken direkt.** Crafting-/Lager-Zustand muss
   repliziert werden. Unreal bringt Netzwerk-Replikation eingebaut mit (server-
   autoritativ) — die Frage ist weniger *welche Lösung*, sondern *wer ist Autorität*
   und *was wird repliziert*.
4. **Gemeinsames Git-Repo + Regeln.** Branch pro Person, exakt gleiche Engine-Version,
   nicht beide gleichzeitig am selben Level/Blueprint, LFS für Binärdateien.
5. **Gleiche Engine-Version und gleiche Plugins in allen Projekten.**
6. **Reihenfolge:** Denis macht erst die aktuelle Mechanik fertig, *dann* wird
   konsolidiert — nicht vorher, nicht nebenbei.
7. **Idee "Nimm mich mit":** Guide-NPC, der beim Spielstart den aktuellen Stand
   erklärt. Hängt am Speichersystem + Spieler-Identität → im gemeinsamen Projekt
   bauen. Konzept: `Workspaces/Konzepte/NimmMichMit_Konzept.md`.

---

## 1. Projektstruktur — Stand: weitgehend schon da

`Workspaces\Unreal\NPCs` (UE **5.8**, C++-Modul `NPCs`) enthält bereits mehrere
Mechaniken nebeneinander:

```
NPCs/
  Source/NPCs/
    Crafting/         NPC-Arbeitsmechanik (aus Unity portiert)
    Kamera/           Perspektivwechsel Ego/Third/TopDown
    Kampf/            Health etc.
    Rohstoffe/        Rohstoff-Raid
    Shop/             Lean Shop
    Hinweise/         Spieler-Hinweise
    Profil/
    Variant_Horror/   Reste des Templates
    Variant_Shooter/  Reste des Templates
  Content/
    FirstPerson/Lvl_FirstPerson.umap
    Wiese/Lvl_Wiese.umap
```

**Damit ist die Kernfrage vom 7.9. ("ein Projekt oder viele?") faktisch beantwortet:
ein Projekt, Mechaniken als Ordner.** Was in Unity noch bevorstand, ist in Unreal
schon passiert.

**Offen für die Besprechung:**
- **Wird `Unreal\NPCs` die Basis des gemeinsamen Spiels, oder frisch aufsetzen?**
  Dagegen spricht: Der Projektname passt nicht mehr (es ist längst mehr als NPCs),
  und die Template-Reste (`Variant_Horror`, `Variant_Shooter`) liegen noch drin.
  Dafür spricht: Es läuft, und alles neu aufzusetzen kostet Zeit ohne Gegenwert.
- **Gemeinsamer Kern-Ordner** (`Source/NPCs/Core/`) für Character, Kamera, Input,
  geteilte DataAssets und GameplayTags — gibt es noch nicht, alles hängt am Template.
- Später Mechaniken als **Plugins** auslagern? (jetzt Overkill)
- Projekt/Modul umbenennen, Template-Reste rauswerfen — ja oder nein?

## 2. Der gemeinsame Kern — Besitz und Verantwortung

- **Character:** Bewegung, Kamera, Interaktion. Für Multiplayer ggf. ganz anders
  (repliziert, Server- vs. Client-Autorität). → **Kumpel gibt Struktur vor.**
  Aktuell: First-Person-Template-Character.
- **Input:** ein gemeinsames **Enhanced Input**-Setup (Input Mapping Context +
  Input Actions) im Kern, nicht pro Mechanik.
- **Gemeinsame Daten-Typen:** Material, Rezept, Item — als **DataAsset** im Kern,
  damit Crafting, Shop und Rohstoffe dieselben benutzen.
- **GameplayTags statt Tag-Strings** früh gemeinsam festlegen.

## 3. Multiplayer-Auswirkungen (Input vom Kumpel nötig)

- **Unreal bringt Replikation mit.** Kein Mirror/FishNet/Photon nötig wie in Unity.
  Offen bleibt trotzdem: Dedicated Server oder Listen Server?
- **Autorität:** Wer besitzt den Crafting-/Job-Zustand? Vermutlich der Server.
- **Zu replizieren bei der Crafting-Mechanik:**
  - Job-Status (Phase, Fortschritt, Restzeit)
  - Lager-Bestände (mehrere Spieler greifen zu)
  - NPC-Position/Pfad (oder nur Ziel replizieren, Bewegung lokal?)
- **Konsequenz für Denis:** Mechaniken so bauen, dass die Zustandsänderung an *einer*
  klaren Stelle passiert — dann ist "nur der Server darf das" später eine kleine
  Änderung statt eines Umbaus.

## 4. Zusammenarbeit / Git

- **Ein gemeinsames Repo fürs Spiel** (getrennt vom CheatSheet).
  **Offen:** Das Unreal-Projekt liegt aktuell unter
  `github.com/Corer91/NPCs_Arbeitsmechanik_Unreal` — Denis' privatem Konto. Wie wird
  daraus ein gemeinsames Repo? (Kollaborateur einladen / Organisation / neues Repo)
- **Branch-Strategie:** `main` stabil, jede Person auf `feature/...`, Merge abgesprochen.
- **Engine-Version exakt gleich:** aktuell **5.8**. Wechsel nur gemeinsam.
- **`.gitignore`:** `Binaries/`, `Intermediate/`, `Saved/`, `DerivedDataCache/` raus;
  `Source/`, `Content/`, `Config/`, `.uproject` rein.
- **`.uasset` und `.umap` sind Binärdateien** — Git kann sie **nicht** mergen.
  Schlimmer als Unitys YAML-Szenen: Bearbeiten zwei Leute dasselbe Blueprint,
  gewinnt genau einer, der andere verliert seine Arbeit.
  → **Regel: nie gleichzeitig am selben Blueprint/Level.**
  → **Offen:** Git LFS einrichten? (`.gitattributes` liegt schon vor, LFS aber noch
  nicht aktiv.) Und: File Locking über die Source-Control-Anbindung im Editor nutzen?
- **Logik nach C++, Verdrahtung nach Blueprint.** Was in C++ steht, ist mergebar;
  was im Blueprint steckt, nicht. Das ist hier kein Stil-, sondern ein Teamthema.

## 5. Rendering

- Gleiche Engine-Version und gleiche Projekt-Settings genügen weitgehend; die
  Unity-Falle "URP vs. Built-in → pinke Materialien" gibt es so nicht.
- **Offen:** Lumen/Nanite an oder aus? Betrifft, wie Assets gebaut werden, und muss
  bei beiden gleich sein.

## 6. Blueprint-Konventionen

- **Verdrahtung innerhalb eines Actors** gehört ins Blueprint.
- **Referenzen auf andere Actors im Level** nicht hart verdrahten, sondern über
  GameplayTags, `GetAllActorsWithTag` oder ein Subsystem holen.
- Namens- und Ordnerkonvention für `Content/` festlegen — **offen**.

## 7. Mechanik-Idee: "Nimm mich mit"

Unverändert gültig (engine-unabhängig): Guide-NPC, der den Spieler bei jedem
Spielstart abholt und den aktuellen Stand erklärt — Hürde beim Wiedereinstieg nach
längerer Pause wegnehmen. Gestaffelt nach Abwesenheitsdauer.

- Zwei Schichten: **A** ein "Tagebuch" (letzter Login, Fortschritt, offene Ziele),
  **B** der Guide + Dialog-UI.
- Schicht A hängt am **Speichersystem** und an der **Spieler-Identität** — im
  Multiplayer pro Spieler, vermutlich serverseitig.
- **Vorschlag:** ganze Mechanik im gemeinsamen Projekt bauen, sobald Speichern und
  Spieler-Identität stehen. Höchstens Schicht B (Dialog-UI) vorab üben.
- Volles Konzept: `Workspaces/Konzepte/NimmMichMit_Konzept.md`

## 8. Lernbegleiter im gemeinsamen Repo

- Jede Person, die lernt, bekommt einen eigenen Ordner `lernen/<kürzel>/` mit
  `LERNFORTSCHRITT.md` (+ `GELERNT.md`). Denis' Kürzel: `pfeifi`.
- Ein Ordner entsteht erst, wenn jemand den Lernmodus tatsächlich benutzt — niemand
  bekommt einen auf Vorrat.
- Der Weitergabe-Skill kommt nach `<repo>/.claude/skills/`. Die Unreal-Fassung heißt
  `lernbegleiter-unreal`, die Unity-Fassung liegt in `CheatSheet/lernbegleiter-skill/`.

---

## Stand von Denis' Mechaniken (Kontext für den Kumpel)

- **Unreal `NPCs` (UE 5.8)** — der aktive Stand. Crafting aus Unity portiert,
  dazu Kamera-Perspektivwechsel, Kampf, Rohstoffe, Shop, Hinweise.
  Multiplayer bewusst noch nicht drin.
- **Unity `NPCs_Arbeitsmechanik`** — Vorlage der Crafting-Mechanik, läuft im Editor
  (Milestones 1–6). Milestone 7 (sichtbares Feedback) offen. Wird durch die
  Unreal-Portierung abgelöst.
- **Unity `Bauen V1 Stein an Stein`** — Baumechanik + erster Player-Controller.
  **Offen: Wird die Baumechanik nach Unreal portiert, oder fällt sie weg?**

---

## Von [Kumpel] (hier ergänzen)

-

## Ergebnisse / Beschlüsse (nach der Besprechung ausfüllen)

-

## To-dos

- [ ]
