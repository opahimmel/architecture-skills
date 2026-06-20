# XP-System

Agenten lesen dieselbe Codebase immer wieder neu — welche Zusammenhänge gelten, welche Fallen existieren, wo man anfangen muss. Das XP-System macht nicht-offensichtliche Insights greppable, sodass ein Agent gezielt abruft statt alles zu laden.

## Wie es funktioniert

**Schicht 1 — Datenbank** (`.xp/db.jsonl` im Repo-Root)
Eine Zeile pro Insight, grep-kompatibel. Agent grepped seinen Aufgaben-Keyword und bekommt nur die relevanten Zeilen — nicht das gesamte File.

**Schicht 2 — Marker im Code** (`@xp:slug`)
Direkt über dem relevanten Block. Wenn ein Agent den Code liest, sieht er den Marker und weiß: dazu gibt es einen Eintrag. Zweite Entdeckungsroute vom Code aus.

## Installation

### 1. Skills installieren

Beide Ordner nach `~/.claude/skills/` kopieren:

```
xp-setup/  →  ~/.claude/skills/xp-setup/
xp-update/ →  ~/.claude/skills/xp-update/
```

### 2. Globale CLAUDE.md erweitern

In `~/.claude/CLAUDE.md` folgenden Abschnitt einfügen:

```markdown
## XP-Datenbank

Wenn `.xp/db.jsonl` im aktuellen Repo-Root existiert:

**Vor dem ersten File-Lookup**: `grep -i "<aufgaben-keyword>" .xp/db.jsonl`
Nur bei Treffer: den Eintrag lesen und als gesichertes Wissen behandeln — nicht neu herleiten.

**Beim Code-Lesen**: `@xp:slug`-Marker im Kommentar → sofort `grep "slug" .xp/db.jsonl` ausführen.

Neues nicht-offensichtliches Wissen erschlossen → `/xp-update` aufrufen.
```

Ohne diesen Abschnitt wissen Agenten nicht, dass die Datenbank existiert und wie sie sie nutzen sollen.

## Nutzung

| Kommando | Wann |
|----------|------|
| `/xp-setup` | Einmalig pro Repo — legt `.xp/db.jsonl` und Projekt-CLAUDE.md-Abschnitt an |
| `/xp-update` | Nach jeder Session — schreibt neue Insights direkt, zeigt dann was eingetragen wurde |

## Datenbankformat

```json
{"id": "auth:token-refresh", "note": "TokenStore zuerst prüfen, nicht AuthService — der Store hält den tatsächlichen State.", "files": ["src/auth.ts"], "file_hash": "abc123", "updated": "2026-06-11"}
```

`id` ist namespaced: `domain:slug`. `files` und `file_hash` verankern den Eintrag im Code.

## Marker-Konvention

```js
// @xp:auth:token-refresh — refreshToken
function refreshToken() { ... }
```

Markerformat: `@xp:<id>` — die `id` ist namespaced (`domain:slug`), gefolgt von einem Dash und dem Symbolnamen im Kommentar. Marker-`id` und Datenbank-`id` sind identisch.
