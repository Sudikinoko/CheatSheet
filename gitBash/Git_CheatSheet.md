# Git Bash CheatSheet

**zum Ordner navigieren**
```
cd /c/Users/Denis/Workspaces/Pong
```

**Status abfragen**
```
git status
```

**Alle Dateien hinzufügen**
```
git add *
git add .
```

**Eine Datei entfernen**
```
git rm Assets/Editor/HubForceResolve.cs
```

**Anschließend committen**
```
git commit -m "genaue Erklärung was sich geändert hat"
```
Die Flag `-m` steht für „message" (Nachricht)

**Beispiele für aussagekräftige Commit-Messages:**
```
git commit -m "Spieler-Bewegung hinzugefügt"
git commit -m "Bug in Kollisionserkennung behoben"
git commit -m "UI-Design aktualisiert"
git commit -m "Kommentare im Code ergänzt"
```

**Zum Abschluss hochladen**
```
git push
```

**Änderungen abrufen**
```
git pull
```

**Grafische Oberfläche öffnen**
```
gitk&
```

**Text einfügen (kopiert)**
```
⇧ Shift + Einfg
```

**Ein Projekt klonen**
```
git clone https://github.com/Sudikinoko/CheatSheet.git
```

**In einen Branch wechseln** (Branch = Abzweigung)
```
git checkout [Commit-Hash oder Branch-Name einfügen]
```

Beispiel:
```
git checkout 420c29d1987342
```

**Zurück zum Master**
```
git switch master
```
