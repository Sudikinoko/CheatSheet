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

## Zwei Dateien – pro Person, nicht pro Projekt

Der Skill ist selbst­ständig – er braucht **kein** Claude-Memory. Er legt im
Projekt-Repo für jede lernende Person einen eigenen Ordner an und pflegt darin zwei
Dateien:

```
lernen/
  <kürzel>/GELERNT.md
  <kürzel>/LERNFORTSCHRITT.md
```

| Datei | Zweck |
|---|---|
| `LERNFORTSCHRITT.md` | Persönliches Ziel, eigener Stand, Plan, nächster Schritt, offene Punkte. Nach jedem laufenden Schritt aktualisiert. |
| `GELERNT.md` | Liste der schon verstandenen Konzepte (mit Datum) + Log "Zuletzt wiederholt". Verhindert, dass Bekanntes nochmal erklärt wird. |

**Warum pro Person?** `GELERNT.md` steuert, was noch erklärt wird und was nicht. Eine
gemeinsame Datei hieße: Trägt eine erfahrene Person ihre Konzepte ein, hält der Skill
sie bei allen anderen für bekannt und erklärt sie nicht mehr – genau verkehrt für
Leute, die neu dazukommen. Getrennte Ordner vermeiden nebenbei Merge-Konflikte.

Die Ordner gehören mit ins Git und wandern mit dem Code. Der Skill liest und schreibt
nur im Ordner der Person, mit der er gerade arbeitet.

Projektweite **Design-Entscheidungen** gehören nicht in diese persönlichen Dateien,
sondern in die gemeinsame Projektdokumentation.

## Anpassen

Der erste Abschnitt der `SKILL.md` ("Konfiguration") wird pro Person geklärt:
Name/Anrede, Kürzel für den Lernordner, Sprache der Erklärungen, Stack,
Vorkenntnisse. Die Beispiele in der `SKILL.md` sind Unity/C#, gelten aber sinngemäß
für jede objektorientierte Sprache.
