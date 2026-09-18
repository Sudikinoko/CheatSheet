# CheatSheet

Gemeinsame Dokumente, Notizen und Werkzeuge rund um unser Spiel.

## Was liegt hier

| Ordner | Inhalt |
|---|---|
| `Besprechungen/` | Besprechungsnotizen, `OFFENE_PUNKTE.md` (was ist offen, was hängt womit zusammen) und `MULTIPLAYER.md` (was bedeutet das fürs Netzwerk) |
| `lernbegleiter-skill/` | Claude-Code-Skill: Lernmodus für Unity/C# |
| `lernbegleiter-unreal-skill/` | Claude-Code-Skill: Lernmodus für Unreal Engine 5 + C++ |
| `gitBash/` | Git-Spickzettel |

Wer neu dazukommt, fängt am besten bei `Besprechungen/OFFENE_PUNKTE.md` an — dort
steht in fünf Minuten, was es schon gibt und was noch fehlt.

## Empfohlenes Werkzeug: `grill-me`

Ein Claude-Code-Skill, der **beim Planen** hilft — nicht beim Programmieren.

Du beschreibst, was du bauen willst, und Claude fragt dich aus: Runde für Runde,
immer nur die Fragen, die gerade dran sind, jede mit einem Vorschlag als Antwort.
Erst wenn nichts mehr offen ist, wird gebaut. Der Skill schreibt selbst keinen Code
und keine Dateien — er ist reines Gespräch, meist 30–45 Minuten.

**Wofür er sich lohnt:** eine neue Mechanik durchdenken, bevor Code entsteht; eine
Struktur-Entscheidung treffen; eine vage Idee scharf bekommen. Also überall dort, wo
ein Denkfehler sonst erst nach 200 Zeilen auffällt.

**Wofür nicht:** wenn du schon genau weißt, was du tust — dann kostet er nur Zeit.

### Installieren

Er liegt nicht in diesem Repo, damit ihr immer die aktuelle Fassung habt. **Beide**
Ordner werden gebraucht — `grill-me` ist nur der Startknopf, der Inhalt steckt in
`grilling`. Fehlt der zweite, zeigt der Befehl ins Leere.

**Windows (PowerShell):**

```powershell
git clone --depth 1 https://github.com/mattpocock/skills.git $env:TEMP\mp-skills; Copy-Item -Recurse "$env:TEMP\mp-skills\skills\productivity\grill-me","$env:TEMP\mp-skills\skills\productivity\grilling" "$env:USERPROFILE\.claude\skills\"; Remove-Item -Recurse -Force $env:TEMP\mp-skills
```

**Git Bash, Linux, macOS:**

```bash
git clone --depth 1 https://github.com/mattpocock/skills.git /tmp/mp-skills && cp -r /tmp/mp-skills/skills/productivity/grill-me /tmp/mp-skills/skills/productivity/grilling ~/.claude/skills/ && rm -rf /tmp/mp-skills
```

Danach Claude Code neu starten und `/grill-me` aufrufen. Zielordner ist unter Windows
`C:\Users\<name>\.claude\skills\`, sonst `~/.claude/skills/`.

Herkunft: [github.com/mattpocock/skills](https://github.com/mattpocock/skills) von
Matt Pocock, MIT-Lizenz.
