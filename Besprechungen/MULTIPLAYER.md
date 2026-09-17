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
- **Letzte Aktualisierung:** 2026-09-12

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

### 2026-09-15 — Spieler-Profil (Lieblingssichten, Hinweis-Zähler)

`UPlayerProfileSubsystem` (ein **GameInstanceSubsystem**) besitzt die Datei
`Saved/SaveGames/SpielerProfil.sav` und schreibt sie bei jeder Änderung.

**Auswirkung auf Multiplayer:**

- **Unkritisch, und zwar bauartbedingt.** Ein GameInstanceSubsystem lebt auf
  **jeder Maschine einzeln**. Jeder Spieler hat damit automatisch sein eigenes
  Profil auf seinem eigenen Rechner — genau das, was persönliche Einstellungen
  brauchen. Nichts zu replizieren, nichts abzugleichen.
- **Auf einem Dedicated Server** läuft das Subsystem ebenfalls, hat dort aber
  niemanden, für den es etwas speichern könnte. Das ist harmlos (es schreibt nur,
  wenn jemand etwas ändert), sollte beim Umbau aber bewusst so bleiben: der
  Server darf **nicht** anfangen, Profile für Clients zu führen.
- **Eine Grenze, die vor dem Splitscreen fällt:** es gibt **ein Profil pro
  Maschine**, nicht pro Spieler (`ProfileUserIndex` steht fest auf 0). Bei zwei
  lokalen Spielern — Splitscreen, oder *Number of Players = 2* im Editor —
  teilen sich beide eine Datei, halten aber je ihren eigenen Stand im Speicher.
  Wer zuletzt schreibt, gewinnt: die Lieblingssichten des einen überschreiben die
  des anderen. **Nicht durch klügeres Zusammenrechnen beim Schreiben lösbar** —
  die Werte gehören zwei Personen und müssen getrennt bleiben. Der Weg ist ein
  eigener Benutzerplatz je lokalem Spieler; die Umstellung ist **eine** Stelle,
  weil alles Laden und Schreiben durch das Subsystem geht. Für den Netzwerk-Fall
  (jeder an seinem eigenen Rechner) ist nichts zu tun.
- **Die eigentliche Gefahr ist inhaltlich, nicht technisch:** Weltzustand darf
  hier nicht hinein. Sonst hat jeder Client seine eigene Wahrheit über etwas, das
  allen gehört. Der Kauf im Lean Shop ist genau dieser Fall und bleibt deshalb
  draußen — er gehört zu **M2** (repliziert) und in ein Weltspeichern, zusammen
  mit den Lagerbeständen. Begründung im Kopf von `PlayerProfileSave.h`.

### 2026-09-13 — Design-Frage: Trefferzonen (D6) — der teuerste Netzwerkpunkt bisher

**Noch nicht entschieden** (siehe `OFFENE_PUNKTE.md`, D6). Der Netzwerkteil
gehört trotzdem jetzt notiert, weil er die Entscheidung mitbestimmt — und weil
er teurer ist, als er von außen aussieht.

Denis will Kopfschuss = tot statt Kugelschwamm. Solange **nur das Geschütz**
schießt, ist das netzwerktechnisch harmlos: der Turm steht auf dem Server, sucht
serverseitig sein Ziel und zieht serverseitig seinen Line Trace. Kein Client
redet mit. **Erst die Spielerwaffe macht daraus ein Problem.**

| # | Punkt | Was zu tun ist |
|---|---|---|
| **N-10** | **Trefferzonen + Ping = Lag Compensation** | Ein Client sieht den Gegner dort, wo er vor einer halben Ping-Zeit war. Zielt er auf den Kopf und schickt den Schuss zum Server, steht der Gegner dort längst woanders — **der Kopfschuss geht ins Leere, obwohl der Spieler getroffen hat.** Bei großen Trefferflächen fällt das kaum auf, bei einem Kopf sofort. Die übliche Antwort heißt **Rewind**: der Server merkt sich die Hitbox-Positionen der letzten Sekunden und spult sie um die Latenz des Schützen zurück, bevor er prüft. Das ist **deutlich mehr Apparat** als „`ApplyDamage` nur mit `HasAuthority()`" und gehört in die Entscheidung eingerechnet. |

**Was sich an bestehenden Punkten ändert:**

- **N-2 bleibt richtig, wird aber größer.** Der Trace darf weiterhin nur
  serverseitig zählen — aber *„wo war der Kopf im Moment des Schusses"* ist dann
  eine Frage, die der Server überhaupt beantworten können muss.
- **Die Signatur von `ApplyDamage` ändert sich** (ein Parameter für die Zone).
  Das ist **keine neue Baustelle**: es bleibt bei *einer* Stelle mit
  `HasAuthority()`, sie bekommt nur ein Argument mehr. Genau dafür war K5 da.
- **Der Notify-Punkt vom 12.09. gilt unverändert:** Der Anim-Notify trennt Effekt
  und Schaden. Mündungsblitz auf jedem Client, Trefferprüfung nur auf dem Server.

**Die Reihenfolge, die daraus folgt:** Trefferzonen lohnt es sich **vor** dem
Netzwerkumbau zu *entscheiden*, aber **nach** ihm zu *bauen* — sonst entsteht das
Rewind zweimal.

### 2026-09-13 — Design-Entscheidung: Sterben ist im Mehrspieler ein TEAM-Zustand

**Das ist keine Technikfrage, sondern eine Design-Vorgabe von Denis. Sie gehört
gelesen, bevor am Respawn gebaut wird.**

Im **Einzelspieler** bleibt es wie jetzt: sterben, kurz warten, in der Basis
aufwachen.

Im **Mehrspieler** soll daraus ein gemeinsamer Kampf werden:

| Zustand | Dauer | Was gilt |
|---|---|---|
| **Am Boden** | ~20 s | Mitspieler können den Gefallenen wiederbeleben |
| **Richtig tot** | ~60 s | Niemand hat ihn rechtzeitig erreicht — er kommt erst nach Ablauf zurück |

Die Zahlen sind Startwerte, keine Festlegung.

**Warum das wichtig ist:** Erst damit wird die bestehende Abbruchbedingung
spielbar. Der Überfall endet zugunsten der Angreifer, wenn *alle Spieler tot
sind und die Gegner das Erz haben* — heute ist „alle tot" bei sofortigem
Respawn praktisch nie wahr. Mit einem Am-Boden-Zustand entsteht genau die
Situation, die die Regel meint: **entweder kommt das ganze Team zurück, oder
keiner — und keiner heißt auch keine Rohstoffe.**

**Was daran hängt:**

- **Ein gemeinsamer Wiederbelebungspunkt** statt individuellem Respawn am
  nächsten PlayerStart.
- **N-8 wird dadurch schärfer:** `AllPlayersInBase()` zählt einen Spieler ohne
  Pawn absichtlich als *nicht in der Basis*. Mit einem Am-Boden-Zustand muss
  unterschieden werden zwischen „liegt draußen und wartet auf Hilfe" und „ist
  weg und kommt in 60 s wieder" — das eine hält den Überfall offen, das andere
  entscheidet ihn.
- **Der Tasten-Blocker trifft direkt hierauf.** Ein gemeinsamer
  Wiederbelebungspunkt heißt Pawn-Wechsel, und der macht derzeit alle
  Bedienungen tot (siehe `OFFENE_PUNKTE.md`, Abschnitt 3). Die Umstellung auf
  Enhanced Input muss **davor** passieren.

### 2026-09-12 — Rohstoffe, Kampf und Überfall (Meilenstein 1–7)

Plan: `Unreal\NPCs\plan-rohstoff-raid-unreal.md`. Neuer Kreislauf: Erz abbauen →
Bedrohung steigt → Angreifer-Wellen → Erz heimbringen oder verlieren. Neue Ordner
`Source\NPCs\Kampf\` und `Source\NPCs\Rohstoffe\`.

> **Der wichtigste Punkt zuerst, weil er eine Architektur-Entscheidung ist und
> nicht nur eine Leitung:** siehe N-1 unten (Freischaltungen am Pawn).

**Was bewusst schon richtig gebaut ist (bitte erhalten):**

- **Genau EINE Schadensfunktion**: `UHealthComponent::ApplyDamage(Menge, Verursacher)`.
  Geschütz, Angreifer und die spätere Spielerwaffe rufen alle dieselbe. Für den
  Umbau heißt das: **ein** `HasAuthority()` an **einer** Stelle, nicht fünf.
- **Einer entscheidet, die anderen fragen** (wie A2): `AOreNode::TryMine()` gibt
  zurück, wie viel *wirklich* herauskam — der Spieler rechnet nichts aus.
  `ADroppedOre::TakeAll()` gibt seinen Inhalt genau **einmal** heraus.
- **Dauern über Timer und Zeitstempel, nie pro Frame** (wie A3/A6): Abbautempo,
  Denk-Takt, Schlag-Takt, Wellenabstand. `UMiningComponent::GetUnitProgress()`
  rechnet aus dem Startzeitpunkt — dasselbe Muster wie der Craft-Fortschritt.
- **Alle statischen Listen sind nach Welt gefiltert** (wie A5): `ActiveTargets`,
  `ActiveNodes`, `ActiveRaiders`, `ActiveBundles`.
- **Der Anim-Notify des Geschützes wird Effekt und Schaden trennen** — der Notify
  läuft später auf jedem Client, der Schaden darf dort nicht entstehen.

**Zu erledigen:**

| # | Punkt | Was zu tun ist |
|---|---|---|
| **N-1** | **`UUnlockComponent` hängt am Spieler-PAWN.** | **Architektur, nicht Leitung — und schon im Einzelspieler kaputt.** Wird der Pawn beim Respawn zerstört (`ERespawnMode::NeuErstellen`), ist der gekaufte Kamera-Roboter weg: Taste K tut danach nichts mehr, ohne jede Fehlermeldung. Es gibt derzeit eine **Überbrückung** (`UPlayerRespawnComponent::CarryOverUnlocks` kopiert den Besitz auf den neuen Pawn), die genau das verdeckt. Richtig gehört Besitz pro Spieler an den **`APlayerState`** — der überlebt den Pawn-Wechsel und ist der übliche Ort für replizierten Zustand pro Spieler. **Die Überbrückung dann ersatzlos löschen.** |
| **N-2** | Lebenspunkte sind Zustand | `UHealthComponent::Health` und `bIsDead` brauchen `Replicated` → gehört zu **M2**. Der Line Trace des Geschützes und `ApplyDamage` dürfen **nur** serverseitig zählen. |
| **N-3** | Abbauen und Aufheben rufen direkt auf | `UMiningComponent` ruft `TryMine()` / `TakeAll()` direkt → **Server-RPC**, genau wie M3. Die Reichweitenprüfung muss **serverseitig wiederholt** werden, sonst baut ein Client von überall ab. |
| **N-4** | Der Rucksack ist Zustand pro Spieler | `UCarriedOreComponent` (Material + Menge) muss repliziert werden. Betrifft auch die spätere Anzeige. |
| **N-5** | Das fallengelassene Bündel ist **Welt**zustand | `ADroppedOre` muss repliziert werden. Wer es nimmt, entscheidet der **Server** — der `bTaken`-Merker schützt heute nur gegen zwei Zugriffe auf **einem** Rechner. |
| **N-6** | Die Bedrohung ist **eine Zahl für alle** | So entschieden (Denis, 12.09.). `ARaidDirector` gehört damit ganz auf den Server: Zählen, Schwellen, Spawnen. Clients brauchen davon höchstens eine Anzeige. |
| **N-7** | **Eine Spielregel hängt an der Spielerzahl** | `ARaiderNpc::CanFightWhileCarrying()` im Modus *Automatisch*: allein darf ein Träger kämpfen, zu mehreren nicht. Das ist so gewollt — aber es heißt, dass die Regel **auf dem Server** ausgewertet werden muss und sich **mitten in der Partie ändern kann**, wenn jemand beitritt oder geht. Der einzige Ort, der das entscheidet, ist diese eine Funktion. |
| **N-8** | Der Überfall endet über eine Frage an alle Spieler | `AllPlayersInBase()` und `AnyPlayerCarriesOre()` gehen über alle `PlayerController`. Serverseitig unproblematisch, aber: ein Spieler **ohne Pawn** (mitten im Respawn) zählt absichtlich als *nicht* in der Basis — sonst endete der Überfall im Moment eines Todes. Bitte so lassen. |
| **N-9** | `SetCombatActive()` läuft pro Spieler | Reine Sichtsache, darf auf dem Client passieren. Der **Beginn** des Überfalls muss die Clients aber erreichen (Multicast oder replizierter Zustand am Director). |

**Offene Entscheidung, die dazugehört:** Der Respawn kennt zwei Modi
(`ERespawnMode`). *Versetzen* legt nichts fest, *NeuErstellen* ist der übliche
Unreal-Weg und der, den der Netzwerkbetrieb braucht. Vorbelegt ist
**NeuErstellen**. Sobald N-1 gelöst ist, kann *Versetzen* ersatzlos weg.

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

### 2026-09-17 — Startprüfung + getrennter Laufradius beim Angreifer

Neu: `UStartupCheckSubsystem` (prüft beim Spielstart stille Fehleinstellungen)
und `ARaiderNpc::PathAcceptanceRadius`, der die Weg-Genauigkeit von der
Schlagweite `AttackRange` trennt.

**Auswirkung auf Multiplayer:**

- **Die Startprüfung gehört auf den Server.** Sie ist ein WorldSubsystem und
  liefe im Netzwerkspiel auf **jedem Client noch einmal** — dieselben Meldungen
  mehrfach, und auf Clients teils falsch, weil dort nicht alle Actors repliziert
  sind (ein Actor, der beim Client fehlt, sieht aus wie „nicht vorhanden", nicht
  wie „falsch eingestellt"). Vor dem Netzwerkbau ein `HasAuthority()` davor.
  Gehört zu **M2**.
- **`PathAcceptanceRadius` ist reine Konfiguration** und ändert sich im Spiel
  nie — muss **nicht** repliziert werden, genau wie `AttackRange`.
- **Die Bewegung selbst bleibt Server-Sache.** `ChaseTarget()` und der Abbruch
  des Laufs beim Wechsel in den Angriffszustand laufen über den AIController;
  im Netzwerkspiel denkt und läuft der NPC auf dem Server, Clients sehen nur das
  Ergebnis. Das ist bereits der Plan (**N4**) und ändert sich hierdurch nicht.
- **Die Ankunftsprüfung ist ein Präzedenzfall fürs Netzwerk**: Der NPC kam an
  einer Wegecke „an", weil eine Gameplay-Distanz als Weg-Toleranz benutzt wurde.
  Dieselbe Verwechslung im Netzwerk wäre teurer — eine Reichweitenprüfung, die
  serverseitig wiederholt werden muss, darf nie identisch mit einem
  Wegfindungs-Parameter sein.

<!-- Neue Einträge hier einfügen -->
