# Lernbegleiter (Claude Code Skill)

Ein Skill, der Claude in einen geduldigen Lernbegleiter verwandelt:
Schritt für Schritt, neuen Code Zeile für Zeile erklären, Clean Code und OOP von
Anfang an, Design Patterns nur wenn sie zum Problem passen, Gelerntes merken und
gezielt wiederholen, kleine "Tipp"-Hinweise und strukturelle Verbesserungen mit
Begründung.

## Installieren

**Pro Person (global):** Ordner nach `~/.claude/skills/lernbegleiter/` kopieren
(unter Windows `C:\Users\<name>\.claude\skills\lernbegleiter\`).

**Pro Projekt (mit im Repo):** Ordner nach `<projekt>/.claude/skills/lernbegleiter/`
kopieren und einchecken – dann hat das ganze Team den gleichen Lernmodus.

Danach greift der Skill automatisch, sobald an einem Lernprojekt gearbeitet wird,
oder per `/lernbegleiter`.

## Zwei Dateien pro Lernprojekt

Der Skill ist selbst­ständig – er braucht **kein** Claude-Memory. Er legt im
Projekt-Repo zwei Dateien an und pflegt sie:

| Datei | Zweck |
|---|---|
| `LERNFORTSCHRITT.md` | Ziel, Design-Entscheidungen, Stand, Plan, nächster Schritt, offene Punkte. Nach jedem laufenden Schritt aktualisiert. |
| `GELERNT.md` | Liste der schon verstandenen Konzepte (mit Datum) + Log "Zuletzt wiederholt". Verhindert, dass Bekanntes nochmal erklärt wird. |

Beide gehören mit ins Git und wandern mit dem Code.

## Anpassen

Der erste Abschnitt der `SKILL.md` ("Konfiguration") wird pro Projekt geklärt:
Name/Anrede, Sprache der Erklärungen, Stack, Vorkenntnisse. Die Beispiele in der
`SKILL.md` sind Unity/C#, gelten aber sinngemäß für jede objektorientierte Sprache.
