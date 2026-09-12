# Multiplayer — Stand, offene Punkte, Übergabe

**Diese Datei ist ein laufendes Dokument, keine einmalige Notiz.**
Jedes Mal, wenn an der Mechanik etwas Neues gebaut wird, kommt hier ein Eintrag dazu:
*Was ist neu, und was bedeutet es für Multiplayer?* So wächst die Liste mit dem Projekt
mit, statt am Ende rekonstruiert werden zu müssen.

> **Schwesterdatei:** `OFFENE_PUNKTE.md` — dort steht alles, was **nicht** Netzwerk ist
> (offene Design-Entscheidungen, Querverbindungen zwischen Mechaniken, fehlende
> Fundamente wie Speichern und Optionsmenü). Diese Datei hier bleibt beim Netzwerk.

- **Projekt:** `C:\Users\Denis\Workspaces\Unreal\NPCs`, Unreal Engine 5.8
- **Level:** `/Game/Wiese/Lvl_Wiese`
- **Code:** `Source\NPCs\Crafting\`
- **Letzte Aktualisierung:** 2026-09-11

---

## 1. Kurzfassung für den Einstieg

Die Handwerks-Mechanik läuft vollständig — **im Einzelspieler**. Es gibt aktuell
**keinen einzigen Netzwerk-Code** im Projekt (kein `Replicated`, kein Server-RPC,
keine `GetLifetimeReplicatedProps`). Das ist kein Versehen: es wurde bewusst erst die
Mechanik fertiggebaut.

Die **Struktur** ist aber schon multiplayer-tauglich angelegt (siehe Abschnitt 3).
Was fehlt, sind im Wesentlichen **drei Stellen** (Abschnitt 4) — Leitungen um eine
Logik herum, die an der richtigen Stelle sitzt.

---

## 2. Offene Entscheidungen (vor dem Umbau zu klären)

Diese Fragen gehören demjenigen, der Multiplayer aufsetzt. Sie sind **bewusst noch
nicht entschieden**, damit nichts in die falsche Richtung vorgebaut wird.

| # | Frage | Warum sie zuerst beantwortet gehört |
|---|---|---|
| E1 | **Dedicated Server oder Listen Server?** | Bei Listen Server ist ein Spieler gleichzeitig Server — Code läuft dort doppelt (Server- und Client-Pfad). Bei Dedicated sind die Rollen sauber getrennt. Beeinflusst jede `HasAuthority()`-Abfrage. |
| E2 | **Bleibt der Auftrag ein `UObject` oder wird er ein replizierter Struct?** | Siehe M1. Empfehlung: Struct. |
| E3 | **Wie viele Spieler gleichzeitig?** | Bei 2–4 reicht einfache Replikation. Ab ~16 lohnt sich Nachdenken über Relevanz/Update-Raten der Lager-Anzeigen. |
| E4 | **Sollen NPCs auf Clients simuliert werden oder nur Serverzustand anzeigen?** | Empfehlung: nur anzeigen. Bewegung repliziert Unreal schon selbst (siehe A4). |

---

## 3. Was bereits multiplayer-freundlich ist (nicht umbauen!)

Diese Entscheidungen wurden im Einzelspieler getroffen und passen zufällig oder
absichtlich schon zum Netzwerkbetrieb. **Bitte beim Umbau erhalten.**

| # | Was | Warum es hilft |
|---|---|---|
| A1 | **Logik sitzt in ActorComponents** (`UMaterialStorageComponent`, `UCraftingStationComponent`) | Components auf replizierten Actors sind der normale Ort für replizierten Zustand. Kein Umbau der Ablage nötig. |
| A2 | **Klare Rollenverteilung**: Station entscheidet, NPC führt aus, UI liest nur | Genau die Trennung, die Server-Autorität verlangt. Der teure Multiplayer-Fehler — Clients entscheiden Spielregeln — ist hier gar nicht erst angelegt. |
| A3 | **Fortschritt wird aus `CraftStartTime` gerechnet, nicht pro Frame gezählt** (`CraftingJob.cpp`, `GetProgress()`) | Lehrbuchmuster für Multiplayer: Server repliziert **einen** Zeitstempel, jeder Client rechnet den Balken selbst aus. Kein Dauerstrom von Fortschrittswerten nötig. |
| A4 | **Der NPC repliziert bereits von allein** | `ANpcCraftWorker` erbt von `ACharacter` → `APawn` setzt im Konstruktor `bReplicates = true` und `SetReplicatingMovement(true)`. Der laufende NPC ist für Clients ohne Zutun sichtbar. |
| A5 | **Statische Listen sind nach Welt gefiltert** (`GetAllStorages`, `GetAllStations`) | `Storage->GetWorld() == World` verhindert, dass sich Server- und Client-Welt im selben Prozess vermischen (PIE mit mehreren Spielern). |
| A6 | **Timer statt Tick für Dauern** (Sammel-Dauer, Craft-Dauer) | Läuft serverseitig ab, ohne dass Clients mitrechnen müssen. |

---

## 4. Zu erledigen (der eigentliche Umbau)

### M1 — `UCraftingJob` ist ein `UObject` → repliziert nicht von allein
**Datei:** `Crafting/CraftingJob.h:27`, erzeugt in `CraftingStationComponent.cpp:240`

`UObject`s werden in Unreal **nicht** automatisch repliziert. Sie bräuchten
`IsSupportedForNetworking()` plus Registrierung als Subobjekt — machbar, aber umständlich.

**Empfehlung:** Job-Daten in einen `USTRUCT` umwandeln, der als repliziertes Feld in
`UCraftingStationComponent` liegt.

> **Hintergrund zur ursprünglichen Entscheidung:** `UObject` wurde bewusst gewählt, damit
> Station, NPC und UI auf **dasselbe** Objekt zeigen — ein Struct wird bei jeder Weitergabe
> kopiert, und der NPC hätte seinen Fortschritt in seine eigene Kopie geschrieben.
> Im Multiplayer entfällt dieser Grund: der **Server** ist ohnehin die einzige Instanz,
> die schreibt. Der NPC ruft dann `Station->SetJobState(...)` statt selbst einen Zeiger zu
> halten, und der Struct repliziert zu den Clients. Für Einzelspieler war die alte
> Entscheidung richtig, für Multiplayer ist es die neue.

### M2 — Zustand braucht `Replicated`-Markierungen
**Dateien:** `MaterialStorageComponent.h:84` (`Slots`), `CraftingStationComponent.h:183` (`CurrentJob`)

Beides sind reine `UPROPERTY()` ohne Replikation. Nötig:
- `UPROPERTY(Replicated)` bzw. `ReplicatedUsing=` für UI-Reaktionen
- `GetLifetimeReplicatedProps()` in beiden Komponenten
- `SetIsReplicatedByDefault(true)` im jeweiligen Konstruktor

Geändert wird der Bestand an zwei Stellen — nur dort muss Autorität geprüft werden:
`MaterialStorageComponent.cpp:191` (`TryTake`) und `:223` (`TryAdd`).

### M3 — Spieler-Input ruft die Station direkt auf
**Datei:** `StationPlayerInputComponent.cpp:89` und `:101`

```cpp
return Station->TryStartJob(Station->AvailableRecipes[RecipeIndex]);   // Zeile 89
return Station->TryPickup();                                          // Zeile 101
```

Auf einem Client würde das nur lokal passieren und beim Server nie ankommen.
Nötig: `UFUNCTION(Server, Reliable)` als Zwischenschritt, die Prüfung (Reichweite!)
**serverseitig wiederholen** — ein Client darf nie bestimmen, dass er nah genug steht.

Die Aufteilung ist dafür schon vorbereitet: `Order()` / `Pickup()` enthalten keine
Handwerkslogik, sie fragen nur die Station. Der RPC kommt sauber davor.

---

## 5. Regel für alles, was ab jetzt gebaut wird

> **Jede neue Mechanik bekommt hier einen Eintrag in Abschnitt 6** — auch wenn sie
> multiplayer-neutral ist. Lieber ein "keine Auswirkung" zu viel als eine vergessene
> Stelle.

Drei Fragen pro neuem Feature:
1. **Hält es Zustand?** → muss vermutlich repliziert werden
2. **Löst der Spieler es aus?** → braucht vermutlich einen Server-RPC
3. **Zeigt es nur etwas an?** → unkritisch, liest replizierten Zustand

---

## 6. Laufende Liste — neue Punkte

*(Neueste oben. Format: Datum — Feature — Auswirkung)*

### 2026-09-12 — Kamera-Roboter / Perspektivwechsel + Lean Shop

Stufenloser Wechsel Ego / Third-Person / Top-Down ueber eine Ankerliste und einen
durchlaufenden Wert `t` (Taste **K**, Mausrad, Lieblingssichten auf **1-4**).
Gekauft wird der Roboter im **Lean Shop** (Taste **G**), bezahlt aus den Lagern.

**Auswirkung auf Multiplayer:**

- **Die Kamera selbst ist rein lokal** — Pose, `t`, Zustand. Kein Replikationsbedarf;
  jeder Spieler schaut, wohin er will. Die Mesh-Sichtbarkeit wird nur fuer den
  eigenen Spieler umgeschaltet und ist ebenfalls unkritisch.
- **Achtung, die eine Ausnahme:** der Steuerungsmodus schaltet
  `bUseControllerRotationYaw` und `bOrientRotationToMovement` am CharacterMovement
  um. Das ist **kein** reiner Anzeigewert — der Server simuliert die Drehung
  derselben Figur mit. Werden die Schalter nur auf dem Client gesetzt, laufen
  Server- und Client-Drehung auseinander (die Figur "zappelt" fuer die anderen).
  Beim Umbau: Schema-Wechsel als **Server-RPC** mit Replikation an alle, nicht rein
  lokal. Das ist die einzige Stelle der Kamera-Mechanik, die das betrifft.
- **Der Besitz ist Zustand pro Spieler.** `UUnlockComponent::OwnedFeatures` muss
  **repliziert** werden → gehoert zu **M2**.
- **Kaufen ist ein Spielerbefehl** → **Server-RPC**, und die **Reichweitenpruefung
  muss serverseitig wiederholt** werden — dasselbe Muster wie Bestellen, Abholen und
  Ausbauen → gehoert zu **M3**. Ein Client darf nicht behaupten duerfen, er stehe
  vor dem Shop.
- `PayFromStorages` veraendert mehrere Lager auf einmal und muss zwingend auf dem
  **Server** laufen — derselbe Punkt wie beim Stationsausbau.
- **Lieblingssichten und Hinweis-Zaehler sind persoenliche Einstellungen** — pro
  Spieler, rein lokal, kein Replikationsbedarf. Sie gehoeren aber in das spaetere
  **Speichern pro Spieler**, nicht in den Weltzustand.
- **Die Tasten binden sich nur einmal, in `BeginPlay`, und nur wenn dann schon ein
  `PlayerController` da ist.** Im Einzelspieler ist er das immer — das Muster laeuft
  seit dem Stationsausbau fehlerfrei. **Auf einem Client kann der Controller aber
  spaeter eintreffen** als `BeginPlay` der Komponente; dann bindet sie nichts mehr
  nach, und die Mechanik ist fuer diese Runde still tot (nur eine `Verbose`-Zeile im
  Log). Betrifft **alle drei** Platzhalter-Bedienungen gleichermassen:
  `UStationPlayerInputComponent`, `UShopPlayerInputComponent`,
  `UCameraRobotComponent`. Beim Umbau deshalb gemeinsam loesen — nachbinden, sobald
  ein Controller da ist (`ReceiveControllerChangedDelegate`). Steht ohnehin an,
  wenn das Projekt von den Platzhalter-Bindungen auf Enhanced-Input-Assets umstellt.

### 2026-09-11 — Lokale Rohstoff-Anzeige + Lagerzugriff

Neue `UStorageAccessComponent` am Spieler beantwortet die Frage *"habe ich gerade
Zugriff auf das Lagernetz?"* und zeigt die Gesamtbestaende als Bildschirm-Anzeige
(`WBP_RohstoffHud`). Modus aktuell **Immer**; vorbereitet ist **InLagerReichweite**,
spaeter kommt dort die Basis-Zone hinein.

**Auswirkung auf Multiplayer:**

- Die **Anzeige selbst ist rein clientseitig** und liest nur — kein Replikationsbedarf.
- **Aber:** sie summiert ueber alle Lager. Damit ein Client richtige Zahlen sieht,
  muessen die **Lagerbestaende repliziert** sein → haengt an **M2**. Ohne das sieht
  jeder Client nur seine eigene, veraltete Sicht.
- `HasStorageAccess()` ist **pro Spieler verschieden** (haengt an der Position). Die
  Frage sitzt deshalb am Pawn und nicht global — das ist im Multiplayer genau richtig
  und sollte so bleiben.
- **Wichtig fuer spaeter:** sobald der Lagerzugriff auch das *Bestellen* steuert, muss
  die Pruefung **serverseitig wiederholt** werden — wie die Reichweitenpruefung bei
  **M3**. Ein Client darf nicht selbst behaupten duerfen, er sei in der Basis.

### 2026-09-11 — Stationsausbau

Die Station laesst sich im Spiel auf die naechste Stufe ausbauen (Taste **U**).
Die Kosten stehen als flache Tabelle `UpgradeCosts` an der Station; bezahlt wird
aus den Lagern, nach dem Prinzip **ganz oder gar nicht** ueber mehrere Lager hinweg
(`UMaterialStorageComponent::PayFromStorages`).

**Auswirkung auf Multiplayer:**

- `CurrentLevel` ist Zustand und muss **repliziert** werden → gehoert zu **M2**.
  `UpgradeCosts` dagegen ist reine Konfiguration und aendert sich im Spiel nie —
  die kann unrepliziert bleiben.
- `UStationPlayerInputComponent::Upgrade()` ruft `TryUpgrade()` direkt auf →
  braucht einen **Server-RPC**, genau wie Bestellen und Abholen → gehoert zu **M3**.
  Wichtig: die Reichweitenpruefung muss **serverseitig wiederholt** werden, sonst
  koennte ein Client von ueberall ausbauen.
- `PayFromStorages` veraendert **mehrere Lager in einem Rutsch**. Das muss zwingend
  auf dem Server laufen, sonst laufen die Bestaende der Spieler auseinander. Die
  Zurueckbuchung bei Fehlschlag ist dort ebenfalls Server-Sache.

### 2026-09-11 — Ausgangsstand aufgenommen
Mechanik vollständig im Einzelspieler: Bestellung (Taste B), Materialsuche im
nächstgelegenen Lager mit ausreichendem Bestand, Fertigung mit Stufen-Multiplikator,
Abholung (Taste P), Welt-Anzeigen mit Status/Fortschritt/Füllstand.
**Auswirkung:** M1–M3 oben.

<!-- Neue Einträge hier einfügen -->
