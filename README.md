# Environment Engineering for AI Agents

> Context Engineering fragt: Was steht im Kontext-Fenster?  
> Environment Engineering fragt: In welcher Umgebung operiert der Agent — und was findet er, wenn er sucht?

---

## Das Konzept

Ein AI-Agent sucht. Er greped, öffnet Dateien, sammelt Kontext. Das ist seine Grundfunktion —
kein Harness, kein Framework nötig.

Die entscheidende Variable ist die **Umgebung** in der er sucht. Ist sie stumm, bricht er
Invarianten, dupliziert Logik, stellt Fragen die schon beantwortet wurden. Ist sie engineered,
findet er Bedeutung statt nur Code — und jede Session baut auf der letzten auf.

Environment Engineering bedeutet: die Umgebung so gestalten dass ein suchender Agent
automatisch das Richtige findet, ohne dass man es ihm jedes Mal neu erklären muss.

---

## Drei Schichten

```
CLAUDE.md          → Kognitive Schicht   — wie der Agent suchen soll
Semantic Hooks     → Strukturschicht     — wo die Nahtstellen im Code sind
XP-Datenbank       → Wissensschicht      — was bereits erschlossen wurde
```

Keine Schicht ersetzt die andere. Zusammen entstehen Pfade durch die Codebasis — maschinenlesbar,
greppbar, persistent über Sessions hinweg.

---

## Warum nicht einfach alles in CLAUDE.md schreiben?

Das klassische Muster: alles in ein Markdown-File. Architektur, Konventionen, Fallstricke,
was letzte Woche schiefgelaufen ist.

Das Problem: **ein MD-File ist immer vollständig im Kontext.**

Wenn 300 Zeilen in CLAUDE.md stehen, lädt der Agent alle 300 Zeilen — jede Session, jede Aufgabe.
Er bekommt alles auf einmal. Aber woher soll er wissen was davon jetzt gerade wichtig ist,
wenn man es nicht explizit markiert hat? Die kritische Information steht vielleicht in der Mitte —
zwischen Konventionen die er gerade nicht braucht.

**Die XP-Datenbank löst das anders:**

```json
{"id": "auth:token-flow", "note": "JWT in Redis gecacht, nicht DB. null wirft 500.", ...}
{"id": "canvas:world-to-screen", "note": "Koordinaten-Transform — nicht anfassen ohne Tests.", ...}
{"id": "ipc:write-batching", "note": "source.write() ist ein IPC-Call nach Rust. Batching zwingend.", ...}
```

Der Agent bekommt nur was er sucht. Greped er nach `token`, erscheint `auth:token-flow`.
Greped er nach `source.write`, erscheint `ipc:write-batching`. Die Datenbank surfaced
genau dann was relevant ist — demand-driven, nicht bulk-loaded.

Das setzt voraus dass die Slug-Struktur (`domain:konzept`) so gewählt wird dass ein natürlicher
grep-Treffer den richtigen Eintrag zurückgibt. Das ist das Handwerk hinter dem System.

---

## Die kognitive Schicht: CLAUDE.md als Navigationsprotokoll

Die Datenbank und die Hooks sind inerte Dateien bis der Agent weiß dass er sie benutzen soll.
Das kommt aus CLAUDE.md — **nicht als Dokumentation, sondern als Verhaltensprotokoll:**

```
Vor dem ersten File-Lookup:
  grep -i "<aufgaben-keyword>" .xp/db.jsonl
  → bereits erschlossenes Wissen, nicht neu herleiten

Wenn @xp:slug im Code auftaucht:
  grep "slug" .xp/db.jsonl
  → Wissens-Infusion für dieses Konzept

Bevor du Code einfügst:
  grep "@boundary\|@invariant" <zieldatei>
  → Schranken prüfen bevor du anfasst
```

Diese Instruktionen stehen **im globalen `~/.claude/CLAUDE.md` und im Projekt-CLAUDE.md**.
Beide. Denn das globale File steuert das generelle Suchverhalten des Agenten —
das Projekt-File verankert es im konkreten Repo-Kontext.

Ohne diese Prompts: die Infrastruktur existiert, wird aber nie benutzt.

---

## Die Strukturschicht: Semantic Hooks

Maschinenlesbare Architektur-Tags direkt im Code — an den Stellen wo ein Agent
ohne Hinweis scheitert oder Grenzen bricht:

```typescript
/**
 * @invariant:no-ipc-calls  @xp:input-layer-separation
 */

// @boundary:rust-ipc-write source.write — Batched IPC call to Rust, requires batching
source.write(shapesSnapshot)
```

Drei Hooks — nicht mehr. `@invariant` setzt ein Schreibverbot. `@boundary` markiert echte
Systemgrenzen (IPC, fetch, DB). `@xp` verbindet das Symbol mit dem XP-DB-Eintrag.

Der Agent greped nach `source.write` — er bekommt `@boundary` in derselben Zeile.
Greped er nach `input-layer-separation` — er findet `@xp` und schlägt sofort in db.jsonl nach.

---

## Die Wissensschicht: XP-Datenbank (`.xp/db.jsonl`)

Was jede Session lernt das nicht im Code steht — implizite Verträge, Namens-Diskrepanzen,
Fallen, semantische Rollen — wird am Ende eingetragen. Der nächste Agent greped zuerst.

Das Erschließen einer Codebasis wird mit jeder Session billiger. Was einmal verstanden wurde
muss nicht neu hergeleitet werden. Das Projekt akkumuliert XP.

---

## Die Skills: Setup und Pflege der Infrastruktur

Die Skills in diesem Repo sind nicht das System — sie bauen es auf und halten es aktuell.

| Skill | Funktion |
|-------|---------|
| `/xp-setup` | `.xp/db.jsonl` anlegen + CLAUDE.md erweitern — einmalig pro Repo |
| `/xp-scan` | Codebasis autonom durchscannen + alle bedeutsamen Symbole eintragen — einmalig für bestehende Repos |
| `/xp-update` | Session-Wissen einschreiben — am Ende jeder Session |
| `/semantic-hooks` | Kritische Module mit `@invariant`, `@boundary`, `@xp` annotieren |
| `/semantic-hooks-review` | Hooks auf Konsistenz und Veralterung prüfen — nach Refactoring |
| `/improve-claude-file` | CLAUDE.md schärfen wenn sie aufgebläht ist |

---

## Einrichten

### 1. Globales CLAUDE.md erweitern

```bash
cat CLAUDE_global_template.md >> ~/.claude/CLAUDE.md
```

[`CLAUDE_global_template.md`](CLAUDE_global_template.md) enthält beide Abschnitte
(XP-Datenbank + Semantische Hooks) in der exakten Form die der Agent braucht.

### 2. Skills installieren

```bash
cp -r semantic-hooks ~/.claude/skills/
cp -r semantic-hooks-review ~/.claude/skills/
cp -r improve-claude-file ~/.claude/skills/
cp -r xp-system/xp-setup ~/.claude/skills/
cp -r xp-system/xp-update ~/.claude/skills/
cp -r xp-system/xp-scan ~/.claude/skills/
```

### 3. Im Projekt

```
/xp-setup            → Datenbank anlegen, Projekt-CLAUDE.md erweitern
/semantic-hooks      → kritische Module annotieren
/improve-claude-file → Projekt-CLAUDE.md schärfen
```

Dann nach jeder Session: `/xp-update` — nach jedem Refactoring: `/semantic-hooks-review`.

---

## Migration bestehender Codebases

| Datei | Was sie tut |
|-------|------------|
| [`MIGRATE_semantic-hooks_symbol-mirroring.md`](MIGRATE_semantic-hooks_symbol-mirroring.md) | Findet grep-blinde Inline-Hooks und fügt Symbolnamen ein |
| [`MIGRATE_xp_session-log-analyse.md`](MIGRATE_xp_session-log-analyse.md) | Analysiert vergangene Sessions → generiert XP-Einträge |
