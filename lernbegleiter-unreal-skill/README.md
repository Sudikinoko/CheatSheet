# Lernbegleiter Unreal (Claude Code Skill)

Ein Skill, der Claude in einen geduldigen Lernbegleiter für **Unreal Engine 5 + C++**
verwandelt: Schritt für Schritt, neuen Code Zeile für Zeile erklären, Unreal-
Konventionen von Anfang an, Blueprint-oder-C++ jedes Mal begründen, Gelerntes merken
und gezielt wiederholen, kleine "Tipp"-Hinweise.

**Die wichtigste Regel:** Die lernende Person klickt selbst im Editor. Claude sagt,
wo es hingeht und warum — auch wenn Claude den Editor über ein MCP-Plugin
fernsteuern könnte. Wer nicht selbst klickt, lernt den Editor nicht kennen.

**Die zweitwichtigste:** Code, den Claude selbst geschrieben hat, lässt er von einem
**anderen Modell** gegenlesen, bevor ein Schritt als fertig gilt. Dasselbe Modell
wiederholt seinen eigenen Denkfehler und bestätigt sich selbst. Der Skill bringt das
als Arbeitsregel mit — ohne Hook, ohne Einrichtung.

Das Gegenstück für Unity/C# liegt in `../lernbegleiter-skill/`.

## Installieren

**Pro Person (global):** Ordner nach `~/.claude/skills/lernbegleiter-unreal/` kopieren
(unter Windows `C:\Users\<name>\.claude\skills\lernbegleiter-unreal\`).

**Pro Projekt (mit im Repo):** Ordner nach
`<projekt>/.claude/skills/lernbegleiter-unreal/` kopieren und einchecken – dann hat
das ganze Team den gleichen Lernmodus.

Danach greift der Skill automatisch in Unreal-Lernprojekten, oder per
`/lernbegleiter-unreal`.

## Zwei Dateien – pro Person, nicht pro Projekt

Der Skill ist selbst­ständig – er braucht **kein** Claude-Memory. Er legt im
Projekt-Repo für jede lernende Person einen eigenen Ordner an:

```
lernen/
  <kürzel>/GELERNT.md
  <kürzel>/LERNFORTSCHRITT.md
```

| Datei | Zweck |
|---|---|
| `LERNFORTSCHRITT.md` | Persönliches Ziel, eigener Stand, Plan, nächster Schritt, offene Punkte. |
| `GELERNT.md` | Liste der schon verstandenen Konzepte (mit Datum) + Log "Zuletzt wiederholt". Verhindert, dass Bekanntes nochmal erklärt wird. |

**Warum pro Person?** `GELERNT.md` steuert, was noch erklärt wird und was nicht. Eine
gemeinsame Datei hieße: Trägt eine erfahrene Person ihre Konzepte ein, hält der Skill
sie bei allen anderen für bekannt und erklärt sie nicht mehr – genau verkehrt für
Leute, die neu dazukommen. Getrennte Ordner vermeiden nebenbei Merge-Konflikte.

Projektweite **Design-Entscheidungen** gehören nicht in diese persönlichen Dateien,
sondern in die gemeinsame Projektdokumentation.

## Umsteiger aus Unity

Der Skill enthält eine Übersetzungstabelle (GameObject → Actor, Prefab → Blueprint,
ScriptableObject → Data Asset …). Wer **nicht** aus Unity kommt, überspringt diesen
Abschnitt – eine Übersetzung ohne Ausgangssprache hilft niemandem. Die Vorkenntnisse
werden beim ersten Einsatz abgefragt.

## Anpassen

Der erste Abschnitt der `SKILL.md` ("Konfiguration") wird pro Person geklärt:
Name/Anrede, Kürzel für den Lernordner, Sprache der Erklärungen, Vorkenntnisse,
Engine-Version. Ist der Editor auf eine andere Sprache eingestellt, benennt der Skill
die Menüs in dieser Sprache.
