## XP-Datenbank

Wenn `.xp/db.jsonl` im aktuellen Repo-Root existiert:

**Vor dem ersten File-Lookup**: `grep -i "<aufgaben-keyword>" .xp/db.jsonl`
Nur bei Treffer: Eintrag als gesichertes Wissen behandeln — nicht neu herleiten.

**Wenn `@xp:domain:slug` im grep-Output erscheint**: sofort `grep "domain:slug" .xp/db.jsonl` ausführen. Der Marker erscheint automatisch wenn du nach dem Symbol-Namen grepped — er ist die Wissens-Infusion für dieses Konzept.

Neues nicht-offensichtliches Wissen erschlossen → `/xp-update` aufrufen.

---

## Semantische Hooks

Wenn `@domain:`-Annotationen im Projekt existieren (Guard: `grep -rl "@domain:" . --include="*.ts" --include="*.js" 2>/dev/null | head -1`):

**Bevor du ein Modul suchst**: `grep -r "@domain:<bereich>" src/` — findet den Owner direkt, statt Dateinamen zu raten.

**Bevor du Code in eine Datei schreibst**: `grep "@boundary\|@invariant" <zieldatei>` — welche Schranken gelten, bevor du einfügst.

**Um alle Consumer eines Moduls zu finden**: `grep -r "@depends:<modulname>"` — alle Abhängigen auf einen Schlag, ohne Files zu öffnen.
